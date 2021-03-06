title: 数据库学习笔记IX

date: 2020-1-19 17:51:12

categories: 2020年1月

tags: [Database，MySQL]

---

rank，row_number，dense_rank，ntile函数的用法  

<!-- more -->


# rank函数
　　rank函数用于返回结果集的分区内每行的排名， 行的排名是相关行之前的排名数加一。简单来说rank函数就是对查询出来的记录进行排名，与row_number函数不同的是，rank函数考虑到了over子句中排序字段值相同的情况，如果使用rank函数来生成序号，over子句中排序字段值相同的序号是一样的，后面字段值不相同的序号将跳过相同的排名号排下一个，也就是相关行之前的排名数加一，可以理解为根据当前的记录数生成序号，后面的记录依此类推。

    select * ,
    RANK() OVER(order by column_name) as rank
    from table_name

练习题目：https://leetcode.com/problems/rank-scores/

## rank函数进行分组

    a           b           c
    ----------- ----------- ----
    1           3           E
    2           4           A
    3           2           D
    3           5           B
    4           2           C
    2           4           B

以a,b进行分组，在每个组内以b进行排名。分了5个组，第2行跟第3行是一个组，其他的每行是一个组。在第2行与第3行的组内以b排名，并列为1

    SELECT * ,
    rank () OVER (PARTITION BY a,b ORDER BY b) rank FROM table_name

    a           b           c    rank
    ----------- ----------- ---- --------------------
    1           3           E    1
    2           4           A    1
    2           4           B    1
    3           2           D    1
    3           5           B    1
    4           2           C    1

以a,b进行分组，在每个组内以c进行排名。分了5个组，第2行跟第3行是一个组，其他的每行是一个组。在第2行与第3行的组内以c排名，由于c列一个是A，一个是B，所以Rank分别为1、2。

    SELECT * ,
    rank () OVER (PARTITION BY a,b ORDER BY c) rank FROM table_name


    a           b           c    rank
    ----------- ----------- ---- --------------------
    1           3           E    1
    2           4           A    1
    2           4           B    2
    3           2           D    1
    3           5           B    1
    4           2           C    1
总结：
1、partition  by用于给结果集分组，如果没有指定那么它把整个结果集作为一个分组。

2、Rank 是在每个分组内部进行排名的。

## 在MySQL中实现Rank函数

在MySQL中没有rank函数，所以我们要自己用普通的查询语句来实现排序功能。这部分可以参见参考资料【2】：《在MySQL中实现Rank高级排名函数》

# row_number函数
　　row_number的用途的非常广泛，一般可以用来实现web程序的分页，为查询出来的每一行记录生成一个序号，依次排序且不会重复，注意使用row_number函数时必须要用over子句选择对某一列进行排序才能生成序号。row_number用法实例:

    select ROW_NUMBER()
    OVER(order by column_name desc) as row_num,* from table_name

# dense_rank函数

　　dense_rank函数的功能与rank函数类似，dense_rank函数在生成序号时是连续的，而rank函数生成的序号有可能不连续。dense_rank函数出现相同排名时，将不跳过相同排名号，rank值紧接上一次的rank值。在各个分组内，rank()是跳跃排序，有两个第一名时接下来就是第四名，dense_rank()是连续排序，有两个第一名时仍然跟着第二名。将上面的Sql语句改由dense_rank函数来实现。

    select * ,
    DENSE_RANK() OVER(order by column_name) as den_rank
    from table_name

# ntile函数

　　ntile函数可以对序号进行分组处理，将有序分区中的行分发到指定数目的组中。 各个组有编号，编号从一开始。 对于每一个行，ntile 将返回此行所属的组的编号。这就相当于将查询出来的记录集放到指定长度的数组中，每一个数组元素存放一定数量的记录。ntile函数为每条记录生成的序号就是这条记录所有的数组元素的索引（从1开始）。也可以将每一个分配记录的数组元素称为“桶”。ntile函数有一个参数，用来指定桶数。下面的SQL语句使用ntile函数对Order表进行了装桶处理：

    select * ,
    NTILE(4) OVER(order by column_name) as ntile
    from table_name

　　如果表的总记录数是6条，而上面的Sql语句ntile函数指定的组数是4，那么Sql Server2005是怎么来决定每一组应该分多少条记录呢？这里我们就需要了解ntile函数的分组依据（约定）。

　　ntile函数的分组依据（约定）：

　　1、每组的记录数不能大于它上一组的记录数，即编号小的桶放的记录数不能小于编号大的桶。也就是说，第1组中的记录数只能大于等于第2组及以后各组中的记录数。

　　2、所有组中的记录数要么都相同，要么从某一个记录较少的组（命名为X）开始后面所有组的记录数都与该组（X组）的记录数相同。也就是说，如果有个组，前三组的记录数都是9，而第四组的记录数是8，那么第五组和第六组的记录数也必须是8。

　　这里对约定2进行详细说明一下，以便于更好的理解。

　　首先系统会去检查能不能对所有满足条件的记录进行平均分组，若能则直接平均分配就完成分组了；若不能，则会先分出一个组，这个组分多少条记录呢？就是 （总记录数/总组数）+1 条，之所以分配 （总记录数/总组数）+1 条是因为当不能进行平均分组时，总记录数%总组数肯定是有余的，又因为分组约定1，所以先分出去的组需要+1条。

　　分完之后系统会继续去比较余下的记录数和未分配的组数能不能进行平均分配，若能，则平均分配余下的记录；若不能，则再分出去一组，这个组的记录数也是（总记录数/总组数）+1条。

　　然后系统继续去比较余下的记录数和未分配的组数能不能进行平均分配，若能，则平均分配余下的记录；若还是不能，则再分配出去一组，继续比较余下的......这样一直进行下去，直至分组完成。

　　举个例子，将51条记录分配成5组，51%5==1不能平均分配，则先分出去一组（51/5）+1=11条记录，然后比较余下的 51-11=40 条记录能否平均分配给未分配的4组，能平均分配，则剩下的4组，每组各40/4=10 条记录，分配完成，分配结果为：11，10，10，10，10

# 总结：

   在使用排名函数的时候需要注意以下三点：

　　1、排名函数必须有 OVER 子句。

　　2、排名函数必须有包含 ORDER BY 的 OVER 子句。

　　3、分组内从1开始排序。

# 参考资料
【1】 https://www.cnblogs.com/52XF/p/4209211.html
【2】 https://www.jianshu.com/p/bb1b72a1623e
