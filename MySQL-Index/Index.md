#Mysql索引优化学习
###MySQL索引：是帮助MySQL高效获取数据的数据结构。简单理解：排好序的快速查找数据结构##
#索引分类：
	##单值索引：即一个索引只包含单个列，一个表可以有多个单列索引
	##唯一索引：索引列的值必须唯一，但允许有空值
	##复合索引：即一个索引包含多个列
#基本语法：
		##创建：	###CREATE [UNIQUE ]  INDEX indexName ON tablename(columnname(length));
		          	###ALTER tablename ADD [UNIQUE] INDEX indexName ON(columnname(length));
		##删除：	###DROP INDEX indexName ON tablename;
		##查看：	###SHOW INDEX FROM table_name(\G);
		##有四种方式来更改数据表的索引:
			###ALTERTABLE tbl_name ADD PRIMARY KEY(column_list):该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
			###ALTERTABLE tbl_name ADD UNQUE indx_name (column_ist):这条语句创建索引的值必须是唯一的（除了NUL外，NULL可能会出现多次)
			###ALTERTABLE tbl_name ADD INDEX index_name(column_list):添加普通索引，索引值可出现多次。 
			###ALTERTABLE tbl_name ADD FULLTEXT index_name (column_list)该语句指定了索引为FULLTEXT，用于全文索引。

#适合建立索引：
	1.主键自动建立唯一索引
	2.频繁作为查询条件的字段应该创建索引
	3.查询中与其它表关联的字段，外键关系建立索引
	4.数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。
	6.单键/组合索引的选择问题，who?(在高并发下倾向创建组合索引)
	7.查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
	8.查询中统计或者分组字段
#不适合建立索引：
	1.表记录太少
	2.经常增删改的表：频繁更新的字段不适合创建索引er因为每次更新不单单是更新了记录还会更新索引
	3.Where条件里用不到的字段不创建索引
	4.注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

	索引的选择性是指索引列中不同值的数目与表中记录数的比。
	如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99。一个索引的选择性越接近于1，这个索引的效率就越高。
#性能分析：

	##1 	MySql Query Optimizera
	##2 MySQL常见瓶颈：
		###CPU:CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
		###MySQL常见瓶颈IO:磁盘I/o瓶颈发生在装入数据远大于内存容量的时候
		###服务器硬件的性能瓶颈: top,free，iostat和vmstat来查看系统的性能状态

	##3 Explain执行计划：可以获得以下信息：（1）通过id字段，获取表的读取顺序（2）通过 type字段，数据读取操作的操作类型（3）possible_keys字段，哪些索引可以使用（4）key字段，哪些索引被实际使用（5）select_type字段，表之间的引用（6）rows字段，每张表有多少行被优化器查询

		###使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈
		###explain +SQL语句
 
		###各字段名词解释
		###(1 重要)  id：id值越大 优先级越高 值相同则按顺序由上到下执行
		###(2)  select_type：
			####SIMPLE		：简单的select查询
			####PRIMARY		：复杂查询最外层
			####SUBQUERY	：在select或where中包含了子查询
			####DERIUED		：在from中包含的子查询 嵌套
			####UNION		：在union之后的查询
			####UNION RESUL	：从union表获取结果的select
		###(3)  table：显示这一行的数据是关于那一张表的
		###(4 重要)  type:    显示查询使用了何种类型，
			####从最好到最差依次是:	system>const>eq_ref>ref>range>index>ALL（常见的) 
			####如将主键置于where列表中，MySQL就能将该查询转换为一个常量
			####system：	
			####const：	表示通过索引一次就找到了,const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快

			####eq_ref：	唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
			
			####ref：	非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

			####range：	只检索给定范围的行,使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你的where语句中出现了between、<、>、in等的查询
				这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。
			index：	Full Index Scan(全索引扫描)，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。
				(也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的)
			all： 	Full Table Scan（全表扫描），将遍历全表以找到匹配的行

			一般来说，得保证查询至少达到range级别，最好能达到ref
				
		(5)  possible_keys:	显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用
		(6 重要)  key:		 实际使用的索引。如果为NULL，则没有使用索引。查询中若使用了覆盖索引（所建的复合索引包含（覆盖）了查询列），则该索引仅出现在key列表中
		(7)  key_lens:	表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好
				key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的
		(8)  ref:		显示索引被谁使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值
		(9 重要)  rows：	根据表统计信息及索引选用情况，大致估算出找到所需的记录 所需要读取的行数 (越小越好)
		
		(10 重要) extra：	包含不适合在其他列中显示但十分重要的额外信息（常见前三个）
			1  Using filesort （有可能就优化掉）：MySQL中无法利用索引完成的排序操作称为"文件排序”，说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。
			2  Using temporary（立马优化）：使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by.
			3  Using index：	表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错!
					如果同时出现using where，表明索引被用来执行索引键值的查找;
					如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
		               10  Using index condition：在5.6版本后加入的新特性（Index Condition Pushdown）;Using index condition 会先条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行；
			4  Using where：使用了where过滤
			5  using join buffer ：使用了连接缓存 建议：调大配置文件中缓冲区的join buffer
			6  impossible where： where子句的值总是false，不能用来获取任何元组
			7  select tables optimized away ：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者，对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
		
			8  distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作
			9  Backward index scan（8.0反向扫描）是执行查询的过程中对B树索引的扫描方式，受sql语句中（正/反向）排序方式以及（正/反向）索引的影响，正向索引扫描的在性能上，稍微优于反向索引扫描，不过，即便是反向索引扫描，也是优化器根据具体查询进行优化的结果，并非一个不好的选择。

覆盖索引(Covering Index） 或者叫索引覆盖。
理解方式一:就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件,换句话说查询列要被所建的索引覆盖。
理解方式二:索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据;当能通过读取索引就可以得到想要的数据，那就不需要读取行了。
	一个索引包含了(或覆盖了)满足查询结果的数据就叫做覆盖索引。
注意:
如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select*
因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。



4 索引优化
unsigned :无符号  非负数，用此类型可以增加数据长度!
varbinary是二进制字符类型
案例 1 单表优化
CREATE TABLE IF NOT EXISTS `article` (
`id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
`author_id` INT(10) UNSIGNED NOT NULL,
`category_id` INT(10) UNSIGNED NOT NULL,
`views` INT(10) UNSIGNED NOT NULL,
`comments` INT(10) UNSIGNED NOT NULL,
`title` VARBINARY(255) NOT NULL,
`content` TEXT NOT NULL
);


INSERT INTO `article`  (`author_id`, `category_id`, `views`, `comments`, `title` , `content`) VALUES
(1,1,1,1,'1','1'),
(2,2,2,2,'2','2'),
(1,1,3,3,'3','3');
SELECT * FROM article;

select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;
查看执行：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;

结论:很显然,type是ALL,即最坏的情况。Extra里还出现了Using filesort,也是最坏的情况。优化是必须的:

# 开始优化：
测试一：	创建复合索引：create index idx_article_ccv on article (category_id,comments,views);
	查看索引：show index from article;
	expain分析：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;
	解决了ALL全表扫描，依旧出现Using filesort 原因：范围条件会导致后面views索引失效
	删除索引：drop index idx_article_ccv on article;
测试二：	创建索引：create index idx_article_cv on article(category_id,views);
	expain分析：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;
	type由ALL->range->ref using filesort消失,优化成功

总结：单表优化 注意范围条件会导致后面的views索引失效。建立索引时，按字段顺序建立索引，但是注意跳过范围条件的字段。
案例二： 两表优化  

创建两个表
CREATE TABLE IF NOT EXISTS `class` (
id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (id)
);
CREATE TABLE IF NOT EXISTS `book` (
`book_id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`book_id`)
);
插入随机数： insert into class(card) values(floor(1+rand()*20));  insert into book(card) values(floor(1+rand()*20));
explain分析：explain select * from class left join book on class.card = book.card;
结论：type为ALL
测试一：添加索引 ：alter table class add index idx_c(card);
explain分析结论：class表type由all变为index ，两表rows不变
删除索引：drop index idx_c on class;
测试二：添加索引：create index idx_c on book(card);
explain分析结论：book表type变为ref，book表rows骤减

总结：两表左/右外连接时，建立索引建在相反的表上，即左外链接建右表，右外连接建左表；

案例三： 三表
建立phone表：
create table if not exists `phone`(
`phone_id` int(10) unsigned  not null auto_increment,
`card` int(10) unsigned not null,
primary key(`phone_id`)
)engine = innodb;
插入随机数据：
insert into phone(card) values(floor(1+(rand()*20)));
清空索引
执行explain分析结论：全all
建立索引：alter table class add index idx_c(card);alter table phone add index idx_c(card);

分析结论：后2行的type都是ref且总rows优化很好,效果不错。因此索引最好设置在需要经常查询的字段中。
总结：join语句优化 优化尽可能减少Join语句中的NestedLoop的循环总次数;“永远用小结果集驱动大的结果集”。




索引失效常见原因：
1.全值匹配我最爱 即查询字段与复合索引全匹配
2.最佳左前缀法则：如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始并且不跳过索引中的列。
3.不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描
4.存储引擎不能使用索引中范围条件右边的列
5.尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少select*
6.mysql在使用不等于(!=或者<>)的时候无法使用索引会导致全表扫描
7.is null ,is not null 也无法使用索引
8.like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作  解决方法:1.通配符写右边2.覆盖索引  注意：like %在前属于范围查询，会是自己索引失效，也会导致后边的索引失效。但是%在右边的（即左边为定值的）不会导致自己和后边的索引失效。
9.字符串不加单引号索引失效  原因：底层发生了隐式的类型转换导致的索引失效，引起全表扫描
10.少用or,用它来连接时会索引失效

order by （group by）优化

ORDER BY子句，尽量使用Index方式排序,避免使用FileSort方式排序
尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀
如果不在索引列上，filesort有两种算法:双路排序和单路排序

优化策略:	增大sort_buffer_size参数的设置，增大max_length_for_sort_data参数的设置

为排序使用索引
MySql两种排序方式∶文件排序或扫描有序索引排序，MySql能为排序与查询使用相同的索引
KEY a_b_c(a,b,c)
order by能使用索引最左前缀
-ORDER BYa
-ORDER BY a,b
-ORDER BYa,b,c
-ORDER BYa DESC, b DESC,c DESC
如果WHERE使用索引的最左前缀定义为常量，则order by能使用索引
-WHEREa = const ORDER BY b,c
-WHEREa = const ANDb = const ORDER BYc
- WHEREa = const ORDER BY b,c
-WHEREa = const AND b > const ORDER BYb,c
不能使用索引进行排序
-ORDER BYaAsC,b DESC,c DESC	/*排序不—致*|
-WHEREg = const ORDER BY b,c	/*严丢失a索引*/
- WHEREa = const ORDER BY c		/*丢失b索引*/
- WHERE a = const ORDER BY a,d	/*d不是索引的一部分*/
- WHERE a in(...) ORDER BY b,c		/*对于排序来说,多个相等条件也是范围查询*/


面试：
1.使用覆盖索引时 and顺序不重要
2.使用order by时 不遵守顺序，会出现using filesort 。特例：当order by的字段为常量时，将不会出现。
3.分组group by  先排序再分组，会有临时表产生，即出现using temporary；
建议：
对于单键索引，尽量选择针对当前query过滤性更好的索引
尽可能通过分析统计信息和调整query的写法来达到选择合适索引的目的
在选择组合索引的时候，当前Query中过滤性最好的字段在索引字段顺序中，位置越靠前越好。
在选择组合索引的时候，尽量选择可以能够包含当前query中的where字句中更多字段的索引

生产环境中优化数据库流程：
1观察，至少跑1天，看看生产的慢SQL情况。
2开启慢查询日志，设置阙值，比如超过5秒钟的就是慢SQL，并将它抓取出来。
3 explain+慢SQL分析
4show profile
5运维经理or DBA，进行SQL数据库服务器的参数调优。
==总结
1慢查询的开启并捕获
2 explain+慢SQL分析
3show profile查询SQL在Mysql服务器里面的执行细节和生命周期情况
4 SQL数据库服务器的参数调优。

in和exists比较结论：
in查询：查询出B中的所有的id并缓存起来，然后检查A表中查询出的id在缓存中是否存在，如果存在则将A的查询数据加入到结果集中，直到遍历完A表中所有的结果集为止。
exists查询：EXISTS()查询是将主查询的结果集放到子查询中做验证，根据验证结果是true或false来决定主查询数据结果是否得以保存。
在MYSQL的连表查询中，最好是遵循‘小表驱动大表的原则’
IN()查询适合B表数据比A表数据小的情况，IN()查询是从缓存中取数据
exists()适合B表比A表数据大的情况：
当A表数据与B表数据一样大时,in与exists效率差不多,可任选一个使用

慢查询日志：

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句。
·由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。
查看慢查询日志时候开启： show variables like '%slow_query_log%';
临时开启慢查询日志： set global slow_query_log=1;
永久开启（不建议）：修改my.cnf文件，[mysqld]下增加或修改参数slow_query_log 和slow_query_log_file后，然后重启MySQL服务器。也即将如下两行配置进my.cnf文件
slow_query_log =1
slow query_log_file=/var/lib/mysql/atguigu-slow.log
查看慢查询语句时间：show variables like '%long_query_time%'; 设置阙值：set global long_query_time=3; 重新开启一个会话后才会变化；
模拟慢查询语句：select sleep(4);
查询慢查询语句日志:cat /var/lib/mysql/zsq-slow.log
查询当前系统中有多少条慢查询语句：show global status like '%slow_queries%';

日志分析工具：mysqldumpslow
s:是表示按照何种方式排序;
c:访问次数
l:锁定时间
r:返回记录
t:查询时间
al:平均锁定时间
ar:平均返回记录数
at:平均查询时间
t:即为返回前面多少条的数据;
g:后边搭配一个正则匹配模式，大小写不敏感的;

工作参考：
得到返回记录集最多的10个SQL
mysqldumpslow -s r-t 10 /var/lib/mysql/zsq-slow.log
得到访问次数最多的10个SQL
mysqldumpslow -s c-t 10 /var/lib/mysql/zsq-slow.log
得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10-g "left join" /var/lib/mysql/zsq-slow.log
另外建议在使用这些命令时结合│和more使用，否则有可能出现爆屏情况
mysqldumpslow -s r-t 10 /var/lib/mysql/zsq-slow.log | more

批量插入1000w条数据
函数插入
1.
#1 建表dept
create table dept( id int unsigned primary key auto_increment, dept_no mediumint unsigned not null default 0, dept_name varchar(20) not null default '', loc varchar(13) not null default '')engine=innodb default charset=gbk;
#2建表emp
create table emp( id int unsigned primary key auto_increment, emp_no mediumint unsigned not null default 0, ename varchar(20) not null default '', job varchar(9) not null default '', mgr mediumint unsigned not null default 0, hiredate date not null, sal decimal(7,2) not null, comm decimal(7,2) not null, dept_no mediumint unsigned not null default 0 )enginne=innodb default charset=gbk;
2.设置参数 log_bin_trust_function_creators
创建函数，假如报错:This function has none of DETERMINISTIC......
原因：#由于开启过慢查询日志，因为我们开启了bin-log,我们就必须为我们的function指定一个参数。
show variables like 'log_bin_trust_function_creators';
set global log_bin_trust_function_creators=1;
#这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法:
windows下my.ini[mysqld]加上log_bin_trust_function_creators=1
linux下/etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1
3 创建函数，保证每条数据都不同
DELIMITER $$
CREATE FUNCTION rand_string(n INT)RETURNS VARCHAR(255)
BEGIN
DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopgrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
DECLARE return_str VARCHAR(255) DEFAULT '';
DECLARE i INT DEFAULT 0;
WHILE i < n DO
SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));
SET i = i +1;
END WHILE;
RETURN return_str;
END $$

#用于随机产生部门编号
DELIMITER $$
CREATE FUNCTION rand_num( )
RETURNS INT(5)
BEGIN
DECLARE i INT DEFAULT 0;
SET i = FLOOR(100+RAND()*10);
RETURN i;
END $$
#假如要删除
#drop function rand_num;

4. 创建存储过程
创建往emp表中插入数据的存储过程
DELIMITER $$
CREATE PROCEDURE insert_emp(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit= 0;
REPEAT
SET i=i+ 1;
INSERT INTO emp (emp_no, ename ,job ,mgr ,hiredate ,sal ,comm ,dept_no ) VALUES((START+i),rand_string(6),'SALESMAN',0001,CURDATE(),2000,400,rand_num());
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$


创建往dept表中插入数据的存储过程
#执行存储过程，往dept表添加随机数据
DELIMITER $$
CREATE PROCEDURE insert_dept(IN START INT(10),IN max_num INT(10))
BEGIN
DECLARE i INT DEFAULT 0;
SET autocommit= 0;
REPEAT
INSERT INTO dept (dept_no ,dept_name,loc) VALUES ((START+i),rand_string(10),rand_string(8));
SET i=i+1;
UNTIL i = max_num
END REPEAT;
COMMIT;
END $$

5.调用存储过程
delimiter ; 
call insert_dept(100,10);

Show ProFile：是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量
默认情况下，参数处于关闭状态，并保存最近15次的运行结果
分析步骤：1是否支持：show variables like '%profiling%';
2 开启功能 set profiling=1;
3 运行sql
4 查看结果 show profiles;
5 诊断SQL：show profile cpu,block io for query (上一步前面的问题SQL数字号码Query_ID);
type:(参数类型)
|ALL		--显示所有的开销信息
|BLOCK IO		--显示块IO相关开销
|CONTEXT SWITCHES --上下文切换相关开销
|CPU		--显示CPU相关开销信息
|IPC		--显示发送和接收相关开销信息	
|MEMORY		--显示内存相关开销信息
|PAGE FAULTS	--显示页面错误相关开销信息
|SOURCE		--显示交换次数相关开销的信息
|SWAPS		--显示和Source_function，Source_file，Source_line相关的开销信息

日常开发需要注意的结论：
	converting HEAP to MyISAM查询结果太大，内存都不够用了往磁盘上搬了。
	Creating tmp table创建临时表	拷贝数据到临时表	用完再删除
	Copying to tmp table on disk把内存中临时表复制到磁盘，危险!! !
	locked
全局查询日志：永远不要在生产环境开启这个功能。
	set global general_log=1;
	set global log_output='TABLE';
	此后，你所编写的sql语句，将会记录到mysql库里的general_log表，可以用下面的命令查看
	select * from mysql.general_log;

innodb引擎和myisam引擎区别:
1.myisam是默认表类型不是事物安全的；innodb支持事物。

2.myisam不支持外键；Innodb支持外键。

3.myisam支持表级锁（不支持高并发，以读为主）；innodb支持行锁（共享锁，排它锁，意向锁），粒度更小，但是在执行不能确定扫描范围的sql语句时，innodb同样会锁全表。

4.执行大量select，myisam是最好的选择；执行大量的update和insert最好用innodb。

5.myisam在磁盘上存储上有三个文件.frm(存储表定义)  .myd（存储表数据）  .myi（存储表索引）；innodb磁盘上存储的是表空间数据文件和日志文件，innodb表大小只受限于操作系统大小。

6.myisam使用非聚集索引，索引和数据分开，只缓存索引；innodb使用聚集索引，索引和数据存在一个文件。

7.myisam保存表具体行数；innodb不保存。

8.delete from table时，innodb不会重新简历表，而会一行一行的删除。
