# 慢查询日志：

    MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。

    long_query_time的默认值为10，意思是运行10秒以上的语句。

    由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。

**查看慢查询日志时候开启： show variables like '%slow_query_log%';**

**临时开启慢查询日志： set global slow_query_log=1;**

永久开启（不建议）：

    修改my.cnf文件，[mysqld]下增加或修改参数slow_query_log 和slow_query_log_file后，然后重启MySQL服务器。

    即 将如下两行配置进my.cnf文件
    slow_query_log =1
    slow query_log_file=/var/lib/mysql/atguigu-slow.log

**查看慢查询设置的语句时间：show variables like '%long_query_time%';**

**设置阈值：set global long_query_time=3; 重新开启一个会话后才会变化；**

模拟慢查询语句：select sleep(4);

**查询慢查询语句日志:cat /var/lib/mysql/zsq-slow.log**

**查询当前系统中有多少条慢查询语句：show global status like '%slow_queries%';**

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


# 批量插入1000w条数据

## 函数插入
    
1. 建表dept

    `create table dept( id int unsigned primary key auto_increment, dept_no mediumint unsigned not null default 0, dept_name varchar(20) not null default '', loc varchar(13) not null default '')engine=innodb default charset=gbk;`

    建表emp

    `create table emp( id int unsigned primary key auto_increment, emp_no mediumint unsigned not null default 0, ename varchar(20) not null default '', job varchar(9) not null default '', mgr mediumint unsigned not null default 0, hiredate date not null, sal decimal(7,2) not null, comm decimal(7,2) not null, dept_no mediumint unsigned not null default 0 )enginne=innodb default charset=gbk;`

2. 设置参数 log_bin_trust_function_creators

    创建函数，假如报错:This function has none of DETERMINISTIC......

    原因：#由于开启过慢查询日志，因为我们开启了bin-log,我们就必须为我们的function指定一个参数。

        show variables like 'log_bin_trust_function_creators';

        set global log_bin_trust_function_creators=1;

    这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法:

        windows下my.ini[mysqld]加上log_bin_trust_function_creators=1

        linux下/etc/my.cnf下my.cnf[mysqld]加上log_bin_trust_function_creators=1

3. 创建函数，保证每条数据都不同

    **用于生成随机数**

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

    **用于随机产生部门编号**

        DELIMITER $$
        CREATE FUNCTION rand_num( )
        RETURNS INT(5)
        BEGIN
        DECLARE i INT DEFAULT 0;
        SET i = FLOOR(100+RAND()*10);
        RETURN i;
        END $$

    **删除函数**

    drop function rand_num;

4. 创建存储过程

    **创建往emp表中插入数据的存储过程**

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


    **创建往dept表中插入数据的存储过程**

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

5. 调用存储过程

    delimiter ; *//更换为以;结束*

    call insert_dept(100,10);

# Show ProFile：
    是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优的测量默认情况下，参数处于关闭状态，并保存最近15次的运行结果

## 分析步骤：

        1 是否支持：show variables like '%profiling%';
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
