
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


##http://gmwiki.uk.standardchartered.com:8080/display/BODSD/SQL+Best+Practices

http://blog.csdn.net/haiross/article/details/12171301

http://dreams75.iteye.com/blog/684276

http://www.ithao123.cn/content-7820068.html
 
http://blog.sina.com.cn/s/blog_5017ea6c0101e3c4.html

http://database.9sssd.com/sybase/art/537
首先需要说明的是这篇文章的内容并不是如何调节SQL Server查询性能的（有关这方面的内容能写一本书），而是如何在SQL Server查询性能的调节中利用SET STATISTICS IO和SET STATISTICS TIME这二条被经常忽略的Transact-SQL命令的。
　　从表面上看，查询性能的调节是一件十分简单的事。从本质上讲，我们希望查询的运行速度能够尽可能地快，无论是将查询运行的时间从10分钟缩减为1分钟，还是将运行的时间从2秒钟缩短为1秒种，我们最终的目标都是减少运行的时间。
　　尽管查询性能调节困难的原因有许多，但这篇文章将只涉及其中的一个方面，其中最重要的原因是，每当使用环境发生变化时，就需要对性能进行调节，因此很难搞清楚到底需要如何调节查询的性能。
　　如果象大多数用户那样在一台测试用的服务器上进行性能调查，其效果往往并不是十分地令人满意，因为测试服务器的环境与实际应用的服务器环境并不完全相同。随着对资源要求的不断变化，SQL Server会自动地进行自我调节。
　　如果对这一点有疑问，可以在一台负载很大的服务器上反复地运行同一个查询，在大多数情况下，执行查询所使用的时间并不相同。当然，差距并不大，但其变化足以使性能的调节比它应有的程度要困难一些。
　　这到底是怎么回事儿？是你的想法错了还是在运行查询时，服务器的负载过重？这是引起运行时间增加的原因吗？尽管可以多次反复地运行查询得到一个平均时间，但这样作的工作量很大。我们需要用一种很科学的标准对每次测试时的性能进行比较。
　　测量服务器资源是解决查询性能调节问题的关健
　　在服务器上执行查询时，会用到许多种服务器资源。其中的一种资源是CPU的占用时间，假设数据库没有发生任何改变，反复地运行同一个查询其CPU的占用时间将是十分接近的。在这里，我指的不是一个查询从运行开始到结束的时间，而是指运行这一查询所需要的CPU资源数量，运行一个查询所需要的时间与服务器的忙碌程度有关。
　　SQL Server需要的另一种资源是IO。无论何时运行查询，SQL Server都必须从数据缓冲区中读取数据（逻辑读），如果所需要的数据没有在缓冲区中，则需要到磁盘上读取（物理读）。
　　从讨论中可以知道，一个查询需要的CPU、IO资源越多，查询运行的速度就越慢，因此，描述查询性能调节任务的另一种方式是，应该以一种使用更少的CPU、IO资源的方式重写查询命令，如果能够以这样一种方式完成查询，查询的性能就会有所提高。
　　如果调节查询性能的目的是让它使用尽可能少的服务器资源，而不是查询运行的时间最短，那么就更容易测试你采取的措施是提高了查询的性能还是降低了查询的性能。尤其是在资源利用不断变化的服务器上更是如此。首先，需要搞清楚在对查询进行调节时，如何测试我们的服务器的资源使用情况。
　　又想起了SET STATISTICS IO和SET STATISTICS TIME
　　SQL Server很早以前就支持SET STATISTICS IO和SET STATISTICS TIME这二条Transact-SQL命令了，但由于其他一些原因，在调节查询的性能时，许多DBA（数据为系统管理员）都忽略了它们，也许是它们不大吸引人吧。但不管是什么原因，我们下面就会发现，它们在调节查询性能方面还是很有用的。
　　有三种方式可以使用这二条命令：使用Transact-SQL命令行方式、使用Query Analyzer、在Query Analyzer中设置当前连接适当的连接属性。在这篇文章中，我们将使用Transact-SQL命令行的方式演示它们的用法。
　　SET STATISTICS IO和SET STATISTICS TIME的作用象开关那样，可以打开或关闭我们的查询使用资源的各种报告信息。缺省状态下，这些设置是关闭的。我们首先来看一个这些命令如何打开的例子，并看看它们会报告一些什么样的信息。 
　　在开始我们的例子前，启动Query Analyzer，并连接到一个SQL Server上。在本例中，我们将使用Northwind数据库，并将它作为这个连接的缺省数据库。
　　然后，运行下面的查询：
　　　SELECT * FROM [order details]
　　如果你没有改动过order details这个表，这个查询会返回2155个记录。这是一个典型的结果，相信你已经在Query Analyzer中看到过好多次了。
　　现在我们来运行同一个查询，不过这次在运行查询之前，我们将首先运行SET STATISTICS IO和SET STATISTICS TIME命令。需要记住的是，这二个命令的打开只对当前的连接有效，当打开其中的一个或二个命令后，再关闭当前连接并打开一个新的连接后，就需要再次执行相应的命令。如果想关闭当前连接中的这二个命令，只要将原来命令中的ON换成OFF，再执行一次就可以了。
　　在开始我们的例子前，先运行下面的这二条命令（不要在正在使用的服务器上执行），这二条命令将清除SQL Server的数据和过程缓冲区，这样能够使我们在每次执行查询时在同一个起点上，否则，每次执行查询得到的结果就不具有可比性了：
DBCC DROPCLEANBUFFERS
　　　DBCC FREEPROCCACHE
　　输入并运行下面的Transact-SQL命令：
　　　SET STATISTICS IO ON
　　　SET STATISTICS TIME ON
　　一旦上面的准备工作完成后，运行下面的查询：
　　　SELECT * FROM [order details]
　　如果同时运行上面所有的命令，你得到的输出就会与我的不同，也就很难搞清楚到底发生了什么事情。
在运行上述的命令后，就会在结果窗口中看到以前没有看到过的新资料，在窗口的最顶端，会有下面的信息：
 
SQL Server parse and compile time: （SQL Server解析和编译时间：）
CPU time = 10 ms, elapsed time = 61 ms. 
SQL Server parse and compile time: （SQL Server解析和编译时间：）
CPU time = 0 ms, elapsed time = 0 ms.
 
 
　　在显示上面的数据后，查询得到的记录就会显示出来。在显示完2155条记录后，会显示出下面的信息：
 
Table 'Order Details'. Scan count 1, logical reads 10, physical reads 1, read-ahead reads 9.
（表：Order Details，扫描次数 1，逻辑读 10，物理读 1，提前读取 9）
SQL Server Execution Times:
（SQL Server执行时间：）
CPU time = 30 ms, elapsed time = 387 ms.
 
 
　　（每次得到的结果可能各不相同，在下面我们讨论显示的信息时会提到这一点。）
　　那么，这些信息的具体含意是什么呢？下面我们就来详细地进行分析。
　　　　SET STATISTICS TIME的结果
　　　　SET STATISTICS TIME命令用于测试各种操作的运行时间，其中一些可能对于查询性能的调节没有什么用处。运行这一命令可以在屏幕上得到如下的显示信息：
　　输出的最开始处：
 
SQL Server parse and compile time: 
CPU time = 10 ms, elapsed time = 61 ms. 
SQL Server parse and compile time: 
CPU time = 0 ms, elapsed time = 0 ms.
 
 
　　输出的结束处：
　　　SQL Server Execution Times:
　　　CPU time = 30 ms, elapsed time = 387 ms.
　　在输出的最开始处我们可以看到二次测试时间，但第一行执行某一操作所需的CPU的时间和总共时间，但第二行似乎就不是了。
　　“SQL Server parse and compile time”表示SQL Server解析“ELECT * FROM [order details]”命令并将解析的结果放到SQL Server的过程缓冲区中供SQL Server使用所需要的CPU运行时间和总的时间。
　　在本例中，CPU的运行时间为10毫秒，总时间为61毫秒。由于服务器的配置和负载不同，你得到的CPU运行时间、总时间这二个值可能会与本例中的测试结果有所不同。
　　第二行的“SQL Server parse and compile time”表示SQL Server从过程缓冲区中取出解析结果供执行的时间，大多数情况下这二个值都会是0，因为这个过程执行得相当地快。
　　如果不清除缓冲区而再次运行SELECT * FROM [order details]命令，CPU运行时间和编译时间会都是0，因为SQL Server会重复使用缓冲区中的解析结果，因此就不需要再次编译的时间了。

这些信息在查询性能的调节中对你的帮助真的很大吗？也许并非如此，但我将解释一下这些信息的真正含意，你将会很惊奇，大多数的DBA居然都不真正明白这些信息的含意：
　　我们最感兴趣的是显示在输出最后的时间信息：
　　　SQL Server Execution Times:
　　　CPU time = 30 ms, elapsed time = 387 ms.
　　上面显示的信息表明，执行这次查询使用了多少CPU运行时间和运行查询使用了多少时间。CPU运行时间是对运行查询所需要的CPU资源的一种相对稳定的测量方法，与CPU的忙闲程度没有关系。但是，每次运行查询时这一数字也会有所不同，只是变化的范围没有总时间变化大。总时间是对查询执行所需要的时间（不计算阻塞或读数据的时间），由于服务器上的负载是在不断变化的，因此这一数据的变化范围有时会相当地大。
　　由于CPU占用时间是相对稳定的，因此可以使用这一数据作为衡量你的调节措施是提高了查询性能还是降低了查询的性能的一种方法。
　　　SET STATISTICS IO的效果
　　　SET STATISTICS IO的输出信息显示在输出的结束处，下面是它显示的一个例子： 
　　　Table 'Order Details'. Scan count 1, logical reads 10, physical reads 1, read-ahead reads 9.
　　这些信息中的一部分是十分有用的，另一部分则不然，我们来看看每个部分并了解其含意：
　　Scan Count：在查询中涉及到的表被访问的次数。在我们的例子中，其中的表只被访问了1次，由于查询中不包括连接命令，这一信息并不是十分有用，但如果查询中包含有一个或多个连接，则这一信息是十分有用的。
　　一个循环外部的表的Scan Count值为1，但对于一个循环内的表而言，其值为循环的次数。可以想象得到，对于一个循环内的表而言，其Scan Count值越小，它所使用的资源越少，查询的性能也就越高。因此在调节一个带连接的查询的性能时，需要关注Scan Count的值，在进行调节时，注意观察它是增加还是减少了。
　　Logical Reads: 这是SET STATISTICS IO或SET STATISTICS TIME命令提供的最有用的数据。我们知道，SQL Server在可以对任何数据进行操作前，必须首先把数据读取到其数据缓冲区中。此外，我们也知道SQL Server何时会从数据缓冲区中读取数据，并把数据读取到大小为8K字节的页中。 
　　那么Logical Reads的意义是什么呢？Logical Reads是指SQL Server为得到查询中的结果而必须从数据缓冲区读取的页数。在执行查询时，SQL Server不会读取比实际需求多或少的数据，因此，当在相同的数据集上执行同一个查询，得到的Logical Reads的数字总是相同的。
　　为什么说在调节查询性能中知道SQL Server执行查询时的Logical Reads值是很重要的呢？因为在每次执行同一查询时，这个数值是不会变化的。因此，在进行查询性能的调节时，这是一个可以用来衡量你的调节措施是否成功的一个很好的标准。
　　在对查询的性能进行调节时，如果Logical Reads值下降，就表明查询使用的服务器资源减少，查询的性能有所提高。如果Logical Reads值增加，则表示调节措施降低了查询的性能。在其他条件不变的情况下，一个查询使用的逻辑读越少，其效率就越高，查询的速度就越快。
　　Physical Reads：在这里我要说的的东西可能初听起来有点自相矛盾，但只要反复思考，就会明白其中的真正含意。
　　物理读指的是，在执行真正的查询操作前，SQL Server必须从磁盘上向数据缓冲区中读取它所需要的数据。在SQL Server开始执行查询前，它要作的第一件事就是检查它所需要的数据是否在数据缓冲区中，如果在，就从中读取，如果不在，SQL Server必须首先将它需要的数据从磁盘上读到数据缓冲区中。
　　我们可以想象得到，SQL Server在执行物理读时比执行逻辑读需要更多的服务器资源。因此，在理想情况下，我们应当尽量避免物理读操作。
　　下面的这一部分听起来让人容易感到糊涂了。在对查询的性能进行调节时，可以忽略物理读而只专注于逻辑读。你一定会纳闷儿，刚才不是还说物理读比逻辑读需要更多的服务器资源吗？
　　情况确实是这样，SQL Server在执行查询时所需要的物理读次数不可能通过性能调节而减少的。减少物理读的次数是DBA的一项重要工作，但它涉及到整个服务器性能的调节，而不仅仅是查询性能的调节。在进行查询性能调节时，我们不能控制数据缓冲区的大小或服务器的忙碌程度以及完成查询所需要的数据是在数据缓冲区中还是在磁盘上，唯一我们能够控制的数据是得到查询结果所需要执行的逻辑读的次数。
　　因此，在查询性能的调节中，我们可以心安理得地不理会SET STATISTICS IO命令提供的Physical Read的值。（减少物理读次数、加快SQL Server运行速度的一种方式是确保服务器的物理内存足够多。）
　　Read-Ahead Reads：与Physical Reads一样，这个值在查询性能调节中也没有什么用户。Read-Ahead Reads表示SQL Server在执行预读机制时读取的物理页。为了优化其性能，SQL Server在认为它需要数据之前预先读取一部分数据，根据SQL Server对数据需求预测的准确程度，预读的数据页可能有用，也可能没用。
　　在本例中，Read-Ahead Reads的值为9，Physical Read的值为1，而Logical Reads的值为10，它们之间存在着简单的相加关系。那么我在服务器上执行查询时的过程是怎么样的呢？首先，SQL Server会开始检查完成查询所需要的数据是否在数据缓冲区中，它会很快地发现这些数据不在数据缓冲区中，并启动预读机制将它所需要的10个数据页中的前9个读取到数据缓冲区。当SQL Server检查是否所需要的全部数据都已经在数据缓冲区时，会发现已经有9个数据页在数据缓冲区中，还有一个不在，它就会立即再次读取磁盘，将所需要的页读到数据缓冲区。一旦所有的数据都在数据缓冲区后，SQL Server就可以处理查询了。
　　我们应该怎么办？
　　我在本篇文章的开始曾提到，在对查询的性能进行调节时用一些科学的标准来测量你的调节措施是否有效是十分重要的。问题是，SQL Servers的负载是动态变化的，使用查询总的运行时间来衡量你正在调节性能的查询的性能是提高了还是没有，并不是一个合理的方法。
　　更好的方法是比较多个数据，例如逻辑读的次数或者查询所使用的CPU时间。因此在对查询的性能进行调节时，需要首先使用SET STATISTICS IO和SET STATISTICS TIME命令向你提供一些必要的数据，以便确定你对查询性能进行调节的措施是否真正地得到了目的。


set showplan on

注意： 
SHOWPLAN_XML、SHOWPLAN_ALL 和 SHOWPLAN_TEXT SET 选项为每个批处理生成一个行集。STATISTICS XML 和 STATISTICS PROFILE SET 选项为批处理中的每个选项生成一个行集。
 

SET SHOWPLAN_XML ON
此语句导致 SQL Server 不执行 Transact-SQL 语句。而 Microsoft SQL Server 返回有关如何在正确的 XML 文档中执行语句的执行计划信息。有关详细信息，请参阅 SET SHOWPLAN_XML (Transact-SQL)。
SET SHOWPLAN_TEXT ON
执行该 SET 语句后，SQL Server 以文本格式返回每个查询的执行计划信息。不执行 Transact-SQL 语句或批处理。有关详细信息，请参阅 SET SHOWPLAN_TEXT (Transact-SQL)。
SET SHOWPLAN_ALL ON 
该语句与 SET SHOWPLAN_TEXT 相似，但比 SHOWPLAN_TEXT 的输出格式更详细。有关详细信息，请参阅 SET SHOWPLAN_ALL (Transact-SQL)。
SET STATISTICS XML ON
该语句执行后，除了返回常规结果集外，还返回每个语句的执行信息。输出是正确的 XML 文档集。SET STATISTICS XML ON 为执行的每个语句生成一个 XML 输出文档。SET SHOWPLAN_XML ON 和 SET STATISTICS XML ON 的不同之处在于第二个 SET 选项执行 Transact-SQL 语句或批处理。SET STATISTICS XML ON 输出还包含有关各种操作符处理的实际行数和操作符的实际执行数。有关详细信息，请参阅 SET STATISTICS XML (Transact-SQL)。
SET STATISTICS PROFILE ON
该语句执行后，除了返回常规结果集外，还返回每个语句的执行信息。两个 SET 语句选项都提供文本格式的输出。SET SHOWPLAN_ALL ON 和 SET STATISTICS PROFILE ON 的不同之处在于第二个 SET 选项执行 Transact-SQL 语句或批处理。SET STATISTICS PROFILE ON 输出还包含有关各种操作符处理的实际行数和操作符的实际执行数。有关详细信息，请参阅 SET STATISTICS PROFILE (Transact-SQL)。
SET STATISTICS IO ON
显示 Transact-SQL 语句执行后生成的有关磁盘活动数量的信息。此 SET 选项生成文本输出。有关详细信息，请参阅 SET STATISTICS IO (Transact-SQL)。
SET STATISTICS TIME ON
执行语句后，显示分析、编写和执行每个 Transact-SQL 语句所需的毫秒数。此 SET 选项生成文本输出。有关详细信息，请参阅 SET STATISTICS TIME (Transact-SQL)。
 使用 Showplan SET 语句选项的注意事项 
使用 SHOWPLAN SET 选项显示执行计划时，不执行向服务器提交的语句。而是，SQL Server 分析查询并显示（在一系列运算符中）应如何执行语句。
注意： 
由于显示执行计划时未执行语句，因此没有实际执行 Transact-SQL 操作。例如，如果执行计划包含 CREATE TABLE 语句，由于不存在所涉及的创建的表，因此任何随后涉及该表的操作都将返回错误。但是，此规则有两种例外情况：使用 SHOWPLAN SET 选项时创建临时表；使用 SHOWPLAN SET 选项时执行 USE db_name 语句并尝试将数据库上下文更改为指定的 db_name。
 

使用 STATISTICS SET 选项显示执行计划时，执行向服务器提交的 Transact-SQL 语句。
注意： 
Showplan SET 选项不显示有关加密存储过程或触发器的信息。
 

 在未来的 Showplan 版本中计划不推荐使用的 SET 选项 
在未来的 SQL Server 版本中，将不推荐使用下列 Showplan SET 选项。建议用户尽快学会使用较新的模式。下表列出了计划不推荐使用的 Showplan SET 选项和用户应开始使用的新 SET 选项。
不推荐使用的 SET 选项  使用新的 SET 选项  
SET SHOWPLAN_TEXT
 SET SHOWPLAN_XML
 
SET SHOWPLAN_ALL
 SET SHOWPLAN_XML
 
SET STATISTICS PROFILE
 SET STATISTICS XML

## MXG
/nfsdumps/scripts/env_monitor/check_space $ which mx12b
/pos/home/sybpos2/scripts/mx12b

/nfsdumps/scripts/env_monitor/check_space $ mx12b
1> sp__helpdevice
2> go
exec sp__dumpdevice @devname
 Device Name          Physical Name
 -------------------- --------------------------------------------------
 tapedump1            /dev/rmt4
 tapedump2            /dev/rst0
exec sp__diskdevice @devname, @dont_format, @free_device, @free_size_only
 Device Name               Physical Name                                                size     alloc    free     dsync
 ------------------------- ------------------------------------------------------------ -------- -------- -------- -----
 mx12b_data01              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data01                     32000MB  32000MB      0MB off
 mx12b_data02              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data02                     32000MB  32000MB      0MB off
 mx12b_data03              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data03                     32000MB  32000MB      0MB off
 mx12b_data04              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data04                     32000MB  32000MB      0MB off
 mx12b_data05              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data05                     32000MB  32000MB      0MB off
 mx12b_data06              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data06                     32000MB  32000MB      0MB off
 mx12b_data07              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data07                     32000MB  32000MB      0MB off
 mx12b_data08              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data08                     32000MB  32000MB      0MB off
 mx12b_data09              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data09                     32000MB  32000MB      0MB off
 mx12b_data10              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data10                     32000MB  32000MB      0MB off
 mx12b_data11              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data11                     32000MB  31500MB    500MB off
 mx12b_data12              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data12                     32000MB  32000MB      0MB off
 mx12b_data13              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data13                     32000MB  32000MB      0MB off
 mx12b_data14              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data14                     32000MB  32000MB      0MB off
 mx12b_data15              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data15                     32000MB  32000MB      0MB off
 mx12b_data16              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data16                     32000MB  32000MB      0MB off
 mx12b_data17              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data17                     32000MB  32000MB      0MB off
 mx12b_data18              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data18                     32000MB  32000MB      0MB off
 mx12b_data19              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data19                     32000MB  31000MB   1000MB off
 mx12b_data20              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data20                     32000MB  32000MB      0MB off
 mx12b_data21              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data21                     32000MB  32000MB      0MB off
 mx12b_data22              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data22                     32000MB  32000MB      0MB off
 mx12b_data23              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data23                     32000MB  32000MB      0MB off
 mx12b_data24              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data24                     32000MB  32000MB      0MB off
 mx12b_data25              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data25                     32000MB  32000MB      0MB off
 mx12b_data26              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data26                     32000MB  32000MB      0MB off
 mx12b_data27              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data27                     32000MB  32000MB      0MB off
 mx12b_data28              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data28                     32000MB  32000MB      0MB off
 mx12b_data29              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data29                     32000MB   6400MB  25600MB off
 mx12b_log01               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_log01                      32000MB  32000MB      0MB off
 mx12b_log02               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_log02                      32000MB  32000MB      0MB off
 mx12b_log03               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_log03                      32000MB  32000MB      0MB off
 mx12b_log04               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_log04                      10000MB   2000MB   8000MB off
 sysprocsdev               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_sysprocsdev                  200MB    200MB      0MB off
 systemdbdev               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_sybsystdev                   100MB     50MB     50MB off
 tempdb_mx12b_dev1         /pos/tempdb/tempdb_mx12b_dev1                                 32000MB  32000MB      0MB off
 master                    /dev/vx/rdsk/ukspddmrx02_raw/mx12b_masterdev                    100MB     62MB     38MB on
(return status = 0)
1> exit

/nfsdumps/scripts/env_monitor/check_space $ mx12b
1> sp__helpdevice null,null,'F'
2> go
exec sp__dumpdevice @devname
 Device Name          Physical Name
 -------------------- --------------------------------------------------
 tapedump1            /dev/rmt4
 tapedump2            /dev/rst0
exec sp__diskdevice @devname, @dont_format, @free_device, @free_size_only
 Device Name               Physical Name                                                size     alloc    free     dsync
 ------------------------- ------------------------------------------------------------ -------- -------- -------- -----
 mx12b_data11              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data11                     32000MB  31500MB    500MB off
 mx12b_data19              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data19                     32000MB  31000MB   1000MB off
 mx12b_data29              /dev/vx/rdsk/ukspddmrx02_raw/mx12b_data29                     32000MB   6400MB  25600MB off
 mx12b_log04               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_log04                      10000MB   2000MB   8000MB off
 systemdbdev               /dev/vx/rdsk/ukspddmrx02_raw/mx12b_sybsystdev                   100MB     50MB     50MB off
 master                    /dev/vx/rdsk/ukspddmrx02_raw/mx12b_masterdev                    100MB     62MB     38MB on
(return status = 0)
1> exit

## BCP
http://infocenter.sybase.com/help/index.jsp?topic=/com.sybase.infocenter.dc30191.1550/html/utility/X20696.htm
search key word: slow bcp
