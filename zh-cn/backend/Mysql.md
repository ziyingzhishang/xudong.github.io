

# 查询优化

* 使用 exists 代替 in 查询
* 

# B+ 树

> 真实数据，只保存在叶子结点；
>
> 非叶子结点只存储搜索方向的数据项；

1. 查询 IO 的次数，取决于树的高度；
2. B+ 树的数据项结构是一种复合结构，B+ 树是从左到右建立搜索树；



# GROUP BY 优化

> Group By 在Mysql 中的本质是先进行分组，在进行排序；
>
> 如果想避免因为Group By 导致的 file sort , 则在group by 后面追加 ` order by null` (禁止排序)

Group by 和 Order by 一起使用问题，总会产生 filesort；

# Mysql 索引

索引分为很多类型：普通索引，唯一索引，主键索引，全文索引， 聚合索引；

## 1. 最左匹配原则

在mysql建立联合索引时会遵循最左前缀匹配的原则，即最左优先，在检索数据时从联合索引的最左边开始匹配；

>mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。

## 2. 为何使用联合索引

* 减少开销

> 建一个联合索引(col1,col2,col3)，实际相当于建了(col1),(col1,col2),(col1,col2,col3)三个索引。每多一个索引，都会增加写操作的开销和磁盘空间的开销。对于大量数据的表，使用联合索引会大大的减少开销！

* 覆盖索引（区别索引覆盖）

> 对联合索引(col1,col2,col3)，如果有如下的sql: select col1,col2,col3 from test where col1=1 and col2=2。那么MySQL可以直接通过遍历索引取得数据，而无需回表，这减少了很多的随机io操作。减少io操作，特别的随机io其实是dba主要的优化策略。所以，在真正的实际应用中，覆盖索引是主要的提升性能的优化手段之一。

* 效率高

> 索引列越多，通过索引筛选出的数据越少。有1000W条数据的表，有如下sql:select *from table where col1=1 and col2=2 and col3=3,假设假设每个条件可以筛选出10%的数据，如果只有单值索引，那么通过该索引能筛选出1000W*10%=100w条数据，然后再回表从100w条数据中找到符合col2=2 and col3= 3的数据，然后再排序，再分页；如果是联合索引，通过索引筛选出1000w*10%* 10% *10%=1w，效率提升可想而知！

## 3. 索引优化

> 优化时，使用 `SQL_NO_CACHE` 来关闭缓存，确保优化有效；
>
> ``` mssql
> select SQL_NO_CACHE * from coupon 
> ```
>
> 建立索引的原则；
>
> * = 和 in 可以乱序，mysql 的查询优化器会将索引优化成可以识别的形式；
> * 尽量选择区分度大的列建立索引；
> * 索引列尽量不参与计算；
> * 尽量扩展索引，不要新建索引；

优化步骤：

1. 启用 `SQL_NO_CACHE` 关闭缓存；
2. where 条件查表，锁定最小返回表，从区分度大的字段开始查起；
3. explian 查看执行计划；
4. order by , limit 确保排序的表优先查；

## 4. 索引创建

* 直接创建索引

```mysql
CREATE [UNIQUE|FULLTEXT] INDEDX index_name ON table_name(column_name(length))
```

* 修改表结构的方式

```mysql
ALTER TABLE table_name  ADD [UNIQUE|FULLTEXT] INDEX index_name(column(length))
```

* 创建表的时候创建索引

```mysql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    PRIMARY KEY (`id`),
    [UNIQUE|FULLLTEXT] INDEX index_name (title(length))
)
```

## 5. 索引删除

```mysql
DROP INDEX index_name on table_name
ALTER TABLE table_name DROP INDEX index_name
```

## 6 查看索引

```mysql
SHOW INDEX FROM table_name
```

| 字段         | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| Table        | 索引所在的表                                                 |
| Non_unique   | 非唯一索引，如果为0 ，代表唯一；                             |
| Key_name     | 索引的名字                                                   |
| Seq_in_index | 索引中该列的位置，从1开始，如果是组合索引，则按照字段建立时候的顺序 |
| Collation    | 列是以什么方式存储在索引中，A 或者 NULL , B树始终是A，顺序的 |
| Sub_part     | 是否列的部分被索引，如果只是前100行索引，就显示100， 如果是整列，就显示 NULL |
| Packed       | 关键字是否被压缩，没有为NULL                                 |
| Index_type   | 索引的类型， InnoDB 只支持B树索引                            |

> *Cardinality 关键字解析*
>
> 优化器会根据这个字段来判断是否使用这个索引；在B树索引中，只有高选择字段才有意义，高选择性指字段的选择范围很广；比如名称，手机号等；
>
> *注： 这个关键字也不是及时更新的，需要更新的话，使用 `ANALYZE TABLE table_name`* 
>
> InnoDB 中的 Cardinality 字段的策略
>



## 索引失效的情况

* 使用 `like` 进行模糊查询；
* 对搜索字段使用类型隐式类型转换也不会使用索引；
* 对索引进行函数运算；
* 正则表达式也不会使用索引；
* 聚合索引不符合最左匹配原则也不会使用索引；
* 使用` or` 分割查询条件的时候可能会导致索引失效；
* 使用负向查询（` not`, ` not in ` , ` not like` , ` <>`, ` !>`, ` !<` ）不会使用索引；