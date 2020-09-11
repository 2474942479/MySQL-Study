# MySQL索引
是帮助MySQL高效获取数据的数据结构。简单理解：排好序的快速查找数据结构
## 索引分类：
	1. 单值索引：即一个索引只包含单个列，一个表可以有多个单列索引
	2. 唯一索引：索引列的值必须唯一，但允许有空值
	3. 复合索引：即一个索引包含多个列
## 基本语法：
	创建：	CREATE [UNIQUE ]  INDEX indexName ON tablename(columnname(length));
		        ALTER tablename ADD [UNIQUE] INDEX indexName ON(columnname(length));
	删除：	DROP INDEX indexName ON tablename;
	查看：	SHOW INDEX FROM table_name(\G);
	有四种方式来更改数据表的索引:
		(1)ALTERTABLE tbl_name ADD PRIMARY KEY(column_list):该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
		(2)ALTERTABLE tbl_name ADD UNQUE indx_name (column_ist):这条语句创建索引的值必须是唯一的（除了NUL外，NULL可能会出现多次)
		(3)ALTERTABLE tbl_name ADD INDEX index_name(column_list):添加普通索引，索引值可出现多次。 
		(4)ALTERTABLE tbl_name ADD FULLTEXT index_name (column_list)该语句指定了索引为FULLTEXT，用于全文索引。

## 适合建立索引：
	1.主键自动建立唯一索引
	2.频繁作为查询条件的字段应该创建索引
	3.查询中与其它表关联的字段，外键关系建立索引
	4.数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。
	6.单键/组合索引的选择问题，who?(在高并发下倾向创建组合索引)
	7.查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
	8.查询中统计或者分组字段
## 不适合建立索引：
	1.表记录太少
	2.经常增删改的表：频繁更新的字段不适合创建索引er因为每次更新不单单是更新了记录还会更新索引
	3.Where条件里用不到的字段不创建索引
	4.注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

	索引的选择性是指索引列中不同值的数目与表中记录数的比。
	如果一个表中有2000条记录，表索引列有1980个不同的值，那么这个索引的选择性就是1980/2000=0.99。一个索引的选择性越接近于1，这个索引的效率就越高。
## 性能分析：

1. MySql Query Optimizera
2. MySQL常见瓶颈：
    CPU:CPU在饱和的时候一般发生在数据装入内存或从磁盘上读取数据时候
    MySQL常见瓶颈IO:磁盘I/o瓶颈发生在装入数据远大于内存容量的时候
    服务器硬件的性能瓶颈: top,free，iostat和vmstat来查看系统的性能状态

3. >Explain执行计划：可以获得以下信息：

    >>（1）通过id字段，获取表的读取顺序
    
    >>（2）通过 type字段，数据读取操作的操作类型
    
    >>（3）possible_keys字段，哪些索引可以使用
    
    >>（4）key字段，哪些索引被实际使用
    
    >>（5）select_type字段，表之间的引用
    
    >>（6）rows字段，每张表有多少行被优化器查询
    
    >explain +SQL语句
    
    >>使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈

    >各字段名词解释
		
    >>**(1)  id：id值越大 优先级越高 值相同则按顺序由上到下执行**

    >>(2) select_type:

        >>>SIMPLE：简单的select查询
        >>>PRIMARY	：复杂查询最外层
        >>>SUBQUERY：在select或where中包含了子查询
        >>>DERIUED	：在from中包含的子查询 嵌套
        >>>UNION	：在union之后的查询
        >>>UNION RESUL：从union表获取结果的select

    >>(3)table：显示这一行的数据是关于那一张表的

    >>**(4)type:访问类型  显示查询使用了何种类型**

    >>>从最好到最差依次是:	system>const>eq_ref>ref>range>index>ALL（常见的) 
                
    >>>system:表只有一行记录（等于系统表〉，这是const类型的特列，平时不会出现，这个也可以忽略不计

    >>>const：	表示通过索引一次就找到了,const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量

    >>>eq_ref：	唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
                
    >>>**ref：	非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，它返回所有匹配某个单独值的行。然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体**

    >>>**range：	只检索给定范围的行,使用一个索引来选择行。key列显示使用了哪个索引。一般是在你的where语句中出现了between、<、>、in等的查询这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。**

    >>>**index：	Full Index Scan(全索引扫描)，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。(也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的)**

    >>>all： 	Full Table Scan（全表扫描），将遍历全表以找到匹配的行

    >>**一般来说，得保证查询至少达到range级别，最好能达到ref**
                    
    >>(5) possible_keys:	显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

    >>**(6)  key:		 实际使用的索引。如果为NULL，则没有使用索引。查询中若使用了覆盖索引（所建的复合索引包含（覆盖）了查询列），则该索引仅出现在key列表中**

    >>(7) key_lens:	表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好
                    key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的

    >>(8)  ref:		显示索引被谁使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值
    >>**(9)  rows：	根据表统计信息及索引选用情况，大致估算出找到所需的记录 所需要读取的行数 (越小越好)**
            
    >>(10) extra：	包含不适合在其他列中显示但十分重要的额外信息（常见前五个）

    >>>**Using filesort （有可能就优化掉）：MySQL中无法利用索引完成的排序操作称为"文件排序”，说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。**

    >>>**Using temporary（立马优化）：使了用临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序order by和分组查询group by**

    >>>**Using index：	表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错!如果同时出现Using where，表明索引被用来执行索引键值的查找;如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。**

    >>>**Using where：使用了where过滤**

    >>>**Backward index scan（8.0反向扫描）是执行查询的过程中对B树索引的扫描方式，受sql语句中（正/反向）排序方式以及（正/反向）索引的影响，正向索引扫描的在性能上，稍微优于反向索引扫描，不过，即便是反向索引扫描，也是优化器根据具体查询进行优化的结果，并非一个不好的选择。**

    >>>**Using index condition：在5.6版本后加入的新特性（Index Condition Pushdown）;Using index condition 会先条件过滤索引，过滤完索引后找到所有符合索引条件的数据行，随后用 WHERE 子句中的其他条件去过滤这些数据行；**


    >>>**Using join buffer ：使用了连接缓存 建议：调大配置文件中缓冲区的join buffer**

    >>>impossible where： where子句的值总是false，不能用来获取任何元组

    >>>select tables optimized away ：在没有GROUPBY子句的情况下，基于索引优化MIN/MAX操作或者，对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。
                
    >>>distinct：优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作


    >>**覆盖索引(Covering Index） 或者叫索引覆盖。**
    理解方式一:就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件,换句话说查询列要被所建的索引覆盖。
    理解方式二:索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据;当能通过读取索引就可以得到想要的数据，那就不需要读取行了。一个索引包含了(或覆盖了)满足查询结果的数据就叫做覆盖索引。
    **注意:如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select*因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。**



4. >索引优化

    unsigned :无符号  非负数，用此类型可以增加数据长度!
    varbinary是二进制字符类型

    >>案例 1 单表优化

    >>CREATE TABLE IF NOT EXISTS `article` (
    `id` INT(10) UNSIGNED NOT NULL PRIMARY KEY AUTO_INCREMENT,
    `author_id` INT(10) UNSIGNED NOT NULL,
    `category_id` INT(10) UNSIGNED NOT NULL,
    `views` INT(10) UNSIGNED NOT NULL,
    `comments` INT(10) UNSIGNED NOT NULL,
    `title` VARBINARY(255) NOT NULL,
    `content` TEXT NOT NULL
    );

    >>INSERT INTO `article`  (`author_id`, `category_id`, `views`, `comments`, `title` , `content`) VALUES
    (1,1,1,1,'1','1'),
    (2,2,2,2,'2','2'),
    (1,1,3,3,'3','3');
    SELECT * FROM article;

    >>select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;

    >>查看执行：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;

    >>**explain分析结果:很显然,type是ALL,即最坏的情况。Extra里还出现了Using filesort,也是最坏的情况。优化是必须的**

    >>开始优化:

    >>>测试一:

    >>>>创建复合索引：create index idx_article_ccv on article (category_id,comments,views);

    >>>>查看索引：show index from article;

    >>>>expain分析：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;

    >>>>**结论：解决了ALL全表扫描，依旧出现Using filesort 原因：范围条件会导致后面views索引失效**

    >>>>删除索引：drop index idx_article_ccv on article;

    >>>测试二:
    >>>>创建索引:create index idx_article_cv on article(category_id,views);

    >>>>expain分析：explain select id , author_id from article where category_id = 1 and comments >1 order by  views desc limit 1;

    >>>>测试结果:type由ALL->range->ref using filesort消失,优化成功

    **总结：单表优化 注意范围条件会导致后面的views索引失效。建立索引时，按字段顺序建立索引，但是注意跳过范围条件的字段。**

    >>案例二： 两表优化  

    >>>创建两个表

    >>>> CREATE TABLE IF NOT EXISTS `class` (
        id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
        `card` INT(10) UNSIGNED NOT NULL,
        PRIMARY KEY (id)
        );

    >>>>CREATE TABLE IF NOT EXISTS `book` (
        `book_id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
        `card` INT(10) UNSIGNED NOT NULL,
        PRIMARY KEY (`book_id`)
        );

    >>>插入随机数:
    >>>>insert into class(card) values(floor(1+rand()*20));  insert into book(card) values(floor(1+rand()*20));

    >>>explain分析：explain select * from class left join book on class.card = book.card;

    >>>**explain分析结果：type为ALL**

    >>>开始优化
    >>>>测试一:
    >>>>>添加索引 ：alter table class add index idx_c(card);

    >>>>>explain分析结论：class表type由all变为index ，两表rows不变

    >>>>>删除索引：drop index idx_c on class;

    >>>>测试二：添加索引：create index idx_c on book(card);

    >>>>>explain分析结论：book表type变为ref，book表rows骤减

    >>>**总结：两表左/右外连接时，建立索引建在相反的表上，即左外链接建右表，右外连接建左表**
