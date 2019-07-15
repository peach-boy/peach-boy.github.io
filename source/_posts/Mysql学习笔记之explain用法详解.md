---
title: Mysql学习笔记之explain用法详解
date: 2019-07-09 16:27:54
categories: mysql
tags: 数据库
---

### explain的作用
用来获取查询执行计划的信息，当执行查询时，其返回关于查询计划的每一步的信息，而不是执行它。

### explain输出的列
explain的输出总是相同的列，总共11列

#### id


#### select_type
显示对应行时简单查询还是复杂查询（如果是后者，那么是三种复杂类型中的哪一种）
取值：
1. simple:简单查询，不包含子查询和union 
2. primary:查询为复杂查询，最外层标记为PRIMARY
3. subquery:包含在select列表中的select子查询（不在from中的子句的子查询）标记为subquery
4. dependent subquery
5. derived:在from子句中的子查询
6. union:
7. union result:
8. dependent union
9. 

#### table
table表示对应行正在访问那个表，通常情况下，它就是表明或者别名


#### type 
访问类型，或者决定如何查找表中的行，依次从最差到最优。

```all < index < range < index_subquery < unique_subquery < index_merge < ref_or_null < ref < eq_ref < const<system```

1. all:按行全表扫描，mysql必须扫描整张表，去找到需要的行（例外：limit会从开始扫描到下界加偏移量，如limit 100,10，会从开始扫描到110行;Extra列中有Using distinct/not exists时例外）
2. index:按索引次序扫描全表，需要遍历全表索引
3. range:范围扫描，只需遍历筛选后的区间的索引，一般带有between或者where的查询
4. range ： 这个是index了范围限制，例如 >100、<1000之类的查询条件，这样避免的index的全索引扫描，当然限制的范围越小 效率就越高。
5. index_subquery ： 在 某 些 IN 查 询 中 使 用 此 种 类 型 , 与 unique_subquery 类似,但是查询的是非唯一 性索引
6. unique_subquery ： 在某些 IN 查询中使用此种类型,而不是常规的 ref
7. index_merge ： 说明索引合并优化被使用了
8. ref_or_null ： 如同 ref, 但是 MySQL 必须在初次查找的结果 里找出 null 条目,然后进行二次查找。
9. ref ： 使用了非唯一性索引进行数据的查找，例如：我们对用户表的用户名这一列建立了非唯一索引，因为用户名可以重复，当我们查找用户的时候select * from user where username=“xxx”的时候就出现了ref，使用非唯一索引查找数据。
10. eq_ref ： 这个就很好理解了，使用的唯一性索引进行数据查找，例如主键索引之类的。
11. const ： 通常情况下，将一个主键放置到where后面作为条件查询，mysql优化器就能把这次查询优化转化为一个常量，如何转化以及何时转化，这个取决于优化器。这个比eq_ref效率高一点。
12. system ： 表只有一行。不过这种情况下就没意义了。
13. NULL ： MySQL不用访问表或者索引就直接能到结果。
 
#### possible_keys

#### key

#### key_len

#### ref
rows列显示MySQL认为它执行查询时必须检查的行数。注意这是一个预估值

#### rows

#### extra



