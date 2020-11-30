## spring-framework 本地编译

### 准备工作

> IDEA 2019.3.3
>
> Kotlin 1.4.10
>
> Gradle 6.6
>
> OpenJDK 1.8.0_275
>
> Spring 5.3.2-SNAPSHOT

1. 拉取代码

```shell
git clone https://github.com/spring-projects/spring-framework.git 
```

2. 切分支（选了最近的 `tag` ）

```shell
git checkout -b v5.3.2-SNAPSHOT
```

3. 安装 `gradle`

```properties
# 下载安装包
https://services.gradle.org/distributions/gradle-6.6-all.zip
# 解压
/opt/gradle/gradle-6.6
# 写入环境变量
export PATH=$PATH:/Users/XXXXX/XXXX/gradle-6.6/bin
```



###  预编译 spring-core， spring-oxm

```bash
./gradlew :spring-oxm:compileTestJava

----- 
 Task :spring-oxm:genJaxb
[ant:javac] : warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.7/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 2m 56s
40 actionable tasks: 24 executed, 16 from cache
```



```bash
 ./gradlew :spring-core:compileTestJava

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.7/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 16s
17 actionable tasks: 2 from cache, 15 up-to-date
```



### 修改配置

1. 修改 `gradle.properties`

```properties
version=5.3.2-SNAPSHOT
# 编译过程会占用较大的内存，调大防止内存溢出
org.gradle.jvmargs=-Xmx2048M
# 开启 gradle 缓存
org.gradle.caching=true
# 开启并行编译
org.gradle.parallel=true
# 启用新的孵化模式
kotlin.stdlib.default.dependency=false
# 开启守护进程
org.gradle.daemon=true
```

2. 修改 `build.gradle`

```properties
repositories {
		  maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
      maven { url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'}
			mavenCentral()
			maven { url "https://repo.spring.io/libs-spring-framework-build" }
}
```



3. 修改 `settings.gradle`

```properties
pluginManagement {
	repositories {
		# 加入 阿里云 仓库镜像
	  maven { url "https://maven.aliyun.com/repository/public" }
		gradlePluginPortal()
		maven { url 'https://repo.spring.io/plugins-release' }
	}
}
```



### 导入工程到 IDEA

导入源代码 `File -> New -> Project from Existing Souces... `

![image-20201127150552140](https://raw.githubusercontent.com/ziyingzhishang/ziyingzhishang.github.io/master/assets/imagesimage-20201127150552140.png)

<img src="https://raw.githubusercontent.com/ziyingzhishang/ziyingzhishang.github.io/master/assets/imagesimage-20201127150753561.png" alt="image-20201127150753561" style="zoom:67%;" />

配置 grade 工具

<img src="https://raw.githubusercontent.com/ziyingzhishang/ziyingzhishang.github.io/master/assets/imagesimagesimage-20201127151045363.png" alt="image-20201127151045363" style="zoom:67%;" />

耐心等待两分钟编译成功！

### 测试验证 & 开启源码阅读之旅

<img src="/Users/xudong/Library/Application%20Support/typora-user-images/image-20201127151843569.png" alt="image-20201127151843569" style="zoom:67%;" />

<img src="https://raw.githubusercontent.com/ziyingzhishang/ziyingzhishang.github.io/master/assets/imagesimage-20201127151930451.png" alt="image-20201127151930451" style="zoom:67%;" />



修改 build.gradle

```properties
plugins {
    id 'java'
}

group 'org.springframework'
version '5.3.1'

sourceCompatibility = 1.8

repositories {
    maven { url 'https://maven.aliyun.com/nexus/content/groups/public/' }
    maven { url 'https://maven.aliyun.com/nexus/content/repositories/jcenter'}
    mavenCentral()
}

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    
    compile(project(":spring-context"))
    compile(project(":spring-instrument"))
    compile(project(":spring-beans"))
    compile(project(":spring-context"))
}
```

创建程序入口

```java
public class Application {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext configApplicationContext =
			new AnnotationConfigApplicationContext(SpringConfigDebug.class);
		BeanCreatorTest beanCreatorTest = configApplicationContext.getBean(BeanCreatorTest.class);
		Assert.isInstanceOf(BeanCreatorTest.class, beanCreatorTest);
		System.out.println(beanCreatorTest);
	}
}

```

或者创建测试类都可以

```java
package com.lab.spring.debug;

import static org.junit.Assert.*;

import com.lab.spring.debug.SpringConfigDebug;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.util.Assert;
import org.springframework.util.ObjectUtils;

/**
 * @author Xudong
 * @program spring
 * @decription <br>
 * @date 11/27/20 3:31 PM
 */
public class BeanCreatorTestTest {

	@Test
	public void test() {
		AnnotationConfigApplicationContext configApplicationContext =
			new AnnotationConfigApplicationContext(SpringConfigDebug.class);
		 BeanCreatorTest beanCreatorTest = configApplicationContext.getBean(BeanCreatorTest.class);
		Assert.isInstanceOf(BeanCreatorTest.class, beanCreatorTest);
	}
}
```



## 问题及其解决

### CoroutinesUtils找不到该类

> File -> Project Stucture -> Libraries -> + -> Java -> spring-framework/spring-core/kotlin-coroutines/build/libs/kotlin-coroutines-XXXX.jar 选择 `spring-core.main`

### Grovy 类异常

> 在 `IDEA` 中启用 `Grovvy` 插件;

### Kotlin 编译异常

> * 启用 `Kotlin` 组件；
> * 升级 ` Kotlin` 版本；



## Spring 各个模块的作用

```markdown
spring-core: IOC 和 DI 的实现
spring-beans: Bean 工厂装配
spring-context: Spring 容器上下文，IOC 容器
spring-context-support: 对 IOC 容器的扩展
spring-context-indexer: 类管理组件，classpath 扫描
spring-expression: 表达式

spring-aop：面向切面编程 CGLIB， JDKProxy
spring-aspect: 集成 AspectJ， AOP 应用框架
spring-instrucment: 动态Class， Loading 模块

spring-jdbc: 提供 JDBC 的基本实现，用于简化 JDBC
spring-tx：spring-jdbc 的事务管理
spring-orm: 对象关系映射（Hibernate， JPA，JDO）
spring-oxm: 对象 xml 映射组件
spring-jms: 发送和接收消息

spring-web: 基础的 web 容器支持，建立在核心容器上
spring-webmvc: 实现 spring-mvc 应用
spring-websocket: socket 全双工协议
spring-webflux：全新的非阻塞的函数式的 Reactive Web 框架

spring-messaging: 集成基础报文传送应用

spring-test: 测试组件

spring-framework-bom: 解决不同模块依赖不同版本问题
```

