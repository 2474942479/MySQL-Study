
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
