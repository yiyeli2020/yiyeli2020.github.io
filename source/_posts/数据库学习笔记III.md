title: 数据库学习笔记III
date: 2019-8-27 11:52:12
categories: 2019年8月
tags: [Database，MySQL]

---

重新温习SQL语句中的模糊匹配


<!-- more -->
# 模糊匹配
在进行数据库查询的时候，有完整查询和模糊查询之分。
一般模糊查询语句如下：

  SELECT 字段 FROM 表 WHERE 某字段 Like 条件
其中的条件，SQL有四种匹配模式：

## %
表示任意0个或多个字符，可匹配任意类型和长度的字符，有些情况下若是中文，请使用两个百分号（%%）表示。

例如

    SELECT * FROM user WHERE name LIKE '%一%'

将会把name为“一野”，”李一野”，“一休和尚”等等有”一”的记录都找出来。

另外如果需要找出name中既有“一”又有"野"的记录，请使用and条件

    SELECT *   FROM user  WHERE name  LIKE '%一%'  AND  name LIKE '%野%'

而如果使用

    SELECT *  FROM user  WHERE name  LIKE '%一%野%'

虽然能够搜索出“李一野”，但不能搜索出符合条件的”野一李”。

## _
表示任意单个字符，匹配单个任意字符，它常用来限制表达式的字符长度语句：

比如

    SELECT *  FROM user  WHERE name  LIKE '_三_'

只能找出“唐三藏”这样name为三个字且中间一个字是”三”的；

例如

    SELECT *  FROM user  WHERE name  LIKE '三__'

只能找出“三国杀”这样name为三个字且第一字为”三”的；

## []
表示括号内所列字符中的一个（类似正则表达式）。制定一个字符、字符串或范围，要求所匹配对象为它们中的人一个。

比如

    SELECT *  FROM user  WHERE name  LIKE '[张王李]三'

搜索出的是“张三”，“王三”，”李三”，而不是“张王李三”；

如果[]内有一系列字符（01234，abcd之类)，则可以简写为“0-4”、“a-d”

    SELECT *  FROM user  WHERE name  LIKE '大[a-d]'

将搜索出“大a”、......、“大d”；

## [^]
表示不在括号所列之内的单个字符。其取值和[]相同，但它要求所匹配对象为指定字符以外的任一个字符。

比如

    SELECT *  FROM user  WHERE name  LIKE '[^张王李]三'

将找出不姓“张”，“王”，”李”的“赵三”，”孙三”等。

    SELECT *  FROM user  WHERE name  LIKE '大[^a-d]'

将排除“大a”到“大d”，搜索“大e”、”大f”、......

## 查询内容包含通配符时

由于通配符会使得我们查询特殊字符“%”、"_"、“[”的语句无法正常实现，而把特殊字符用“[]”括起来便可以正常查询。

## 多条件模糊查询

看上去我们可以使用AND连接进行多条件模糊查询

    SELECT * FROM user WHERE name LIKE '%三%' AND phone LIKE ‘%1234%’

但在实际使用时会发现如果数据库的查询字段不是都有值的情况下无法搜索到，

例如某记录name为“三国杀” ，phone没有值也不是null，因为电话没有值素以经过and运算后结果为false，没有输出。显然数据库中的对应字段必须有值，否则会漏掉正确的输出结果。

正确的输入应该为name为“三国杀”，phone为null，这样经过and运算后结果为true，可以输出正确结果

如何解决数据库中查询字段没有值时的多条件模糊匹配在网上搜了一些解决方法但暂未找到比较易懂但方法，留待后面解决


## 注意问题

sql语句

    sql=“select * from user where name like ...”

必须在一行内输入完，而不能用回车符分段，因为vbs多行被认为是多个语句，这是许多初写者常犯的错误。如果你想分多段写，可以用上面的方法在现有变量基础上逐步增加查询语句的各个组成部分并把它存在同一变量内实现。在封闭引号之前或者在打开引号之后你需要增加空格，这样才能保证字符串连接起来的时候没有把几个词凑到了一块。

最后注意以上查询的LIKE后要加单引号。


# 参考资料
【1】https://www.cnblogs.com/GT_Andy/archive/2009/12/25/1921914.html
