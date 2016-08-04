
# What is SQL preformance 
目标是提高单进程下某查询（SELECT）语句性能，那如何衡量一个SQL语句执行性能的好坏呢？只凭SQL执行时间并不能准确衡量其性能，因为测试环境与实际应用的服务器环境并不完全相同；另外运行一个查询所需要的时间与服务器的忙碌程度有关,再则随着对资源要求的不断变化，SQLServer会自动地进行自我调节，导致运行时间也不相同，如果对这一点有疑问，可以在一台负载很大的服务器上反复地运行同一个查询，在大多数情况下，执行查询所使用的时间并不相同。尽管可以多次反复地运行查询得到一个平均时间，但这样作的工作量很大。我们需要用一种很科学的标准对每次测试时的性能进行比较。普遍认可的标准就是测量CPU run time和logic I/O. 在服务器上执行查询时，会用到许多种服务器资源。其中的一种资源是CPU的占用时间(CPU资源数量)，假设数据库没有发生任何改变，反复地运行同一个查询其CPU的占用时间将是十分接近的。

基本流程是：
## Step 1: 测量服务器资源（CPU run time 和 I/O）作为SQL执行性能的量化参考值：

SQL Server需要的另一种资源是IO。无论何时运行查询，SQLServer都必须从数据缓冲区中读取数据（逻辑读），如果所需要的数据没有在缓冲区中，则需要到磁盘上读取（物理读）。从讨论中可以知道，一个查询需要的CPU、IO资源越多，查询运行的速度就越慢，因此，描述查询性能调节任务的另一种方式是，应该以一种使用更少的CPU、IO资源的方式重写查询命令，如果能够以这样一种方式完成查询，查询的性能就会有所提高。如果调节查询性能的目的是让它使用尽可能少的服务器资源，而不是查询运行的时间最短，那么就更容易测试你采取的措施是提高了查询的性能还是降低了查询的性能。尤其是在资源利用不断变化的服务器上更是如此。首先，需要搞清楚在对查询进行调节时，如何测试我们的服务器的资源使用情况。

# SQL Development in Murex
Writing efficient SQL code is paramount to a stable, well performing application. It is the developers responsibility to ensure their code is both functionally correct as well as executes optimally. Some areas to consider include:-
Look at the showplan, statistics and logical IO. Minimise full table scans and logical IO. Logical IO is more important that physical IO or execution time
Join multiple tables via their keys
Ensure correct indexes exist, expecially for joins and where clauses
Reduce the use of functions - don't use on indexed fields as the index will not be used.
Try to avoid sub-queries - usually it's better to use a join
Reduce the number of work tables - these are required when sorting data with an order by ,group by or distinct. Only do this on the final data set / when necessary.
union all is faster than union when non-unique values are acceptable
Limit the select ed fields as soon as possible; reduce both the width and length of requested data. Avoid select * from queries. Use count(1) instead of count(star) in queries.
Try to avoid cursors as these perform slowly. Build a stored procedure that performs several simpler queries, stores results in temporary tables then returns a data set.
Non logged operations must not be used. Eg. truncate table TABLE
You may refer to the following resources on SQL standards and performance tuning -
SCB SQL Development Standards can be found here.
Sybase ASE 12.5 Performance Tuning and Optimisation Manual
Sybase ASE 15 Query Processing and Optimisation Best Practices whitepaper  (some tips may not apply to Sybase ver 12.5.4 being used)

## SQL语句的分类
查询语句：select
数据操纵语句（DML）: INSERT  DELETE   UPDATE
数据定义语句（DDL）: CREATE  TRUNCATE  DROP   ALTER  RENAME
事务控制语句（TCL）: COMMIT  ROLLBACK  SAVEPOINT 
数据控制语句（DCL）: GRANT   REVOKE

## VGRANT { ALL | statement [ ,...n ] }  TO security_account [ ,...n ]
# statement 是被授予权限的语句。语句列表可以包括： 
# TO 指定安全帐户列表
# security_account是权限将应用的安全帐户。安全帐户可以是： Microsoft® SQL Server™ 用户 或SQL Server 角色。
