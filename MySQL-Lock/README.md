# 数据库锁
## 按对数据操作的类型(读/写)分
### 读锁(共享锁):针对同一份数据，多个读操作可以同时进行而不会互相影响
### 写锁（排它锁）:当前写操作没有完成前，它会阻断其他写锁和读锁

## 按对数据操作的粒度分：
### 表锁(偏读)：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。
### 行锁(偏写)：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。
### 页面锁：开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般

##【手动增加表锁】
### lock table表名字 read(write)，表名字2 read(write)，其它;
## 【查看表上加过的锁】
### show open tables;
## 解锁：unlock tables;
表锁分为：
	1）共享读锁 自己可读不可干其他的 别人可读不可阻塞其他
	2）独占写锁 自己可写不可干其他，别人啥也得阻塞
简而言之：读锁会阻塞写，不会阻塞读，写锁会阻塞读和写

分析表锁定：show status like 'table_locks%';
Table_locks_immediate:产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1;
Table_locks_waited:出现表级锁定争用而发生等待的次数(不能立即获取锁的次数，每等待一次锁值加1)，此值高则说明存在着较严重的表级锁争用情况;

#行锁：	
##注意：
###1.sql索引失效会引起行锁升级为表锁  
###2. 间隙锁
####【什么是间隙锁】
####当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁;对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁)
####【危害】
####因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。
####间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

###3.如何锁定一行：select ........ for update 锁定某一行后，其它的操作会被阻塞，直到锁定行的会话提交commit

##分析行锁定：show status like 'innodb_row_lock%';
###Innodb_row_lock_current_waits:当前正在等待锁定的数量;
### ##Innodb_row_lock_time:从系统启动到现在锁定总时间长度;##
### ##Innodb_row_lock_time_avg:每次等待所花平均时间;##
### Innodb_row_lock_time_max:从系统启动到现在等待最常的一次所花的时间;
### ##Innodb_row_lock_waits:系统启动后到现在总共等待的次数;##
## 行锁优化建议：
### 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。合理设计索引，尽量缩小锁的范围。尽可能较少检索条件，避免间隙锁。尽量控制事务大小，减少锁定资源量和时间长度。尽可能低级别事务隔离


#主从复制：
##MySQL复制过程分成三步:
### 1 master将改变记录到二进制日志(binary log）。这些记录过程叫做二进制日志事件，binary log events;
### 2 slave将master的binary log events拷贝到它的中继日志（relay log) ;
### 3 slave重做中继日志中的事件，将改变应用到自己的数据库中。MySQL复制是异步的且串行化的
##复制的基本原则：每个slave只有一个master。每个slave只能有一个唯一的服务器ID。每个master可以有多个salve
##复制最大问题：延时
##一主一从常见配置：
###要求：1. mysql一致且后台以服务运行 2. 同一网段下。3.主从都配置在[mysqld]结点下，都是小写
##1.  主机(linux)修改my.cnf配置文件
###  （必选）##1. server-id=1 ##					#主服务器唯一ID#
###  （必选）##2. log-bin=自己本地的路径/mysql-bin## 			#启用二进制日志#
###  （可选）3.log-err=自己本地路径/自定义错误日志名			#启用错误日志#
###  （可选）4.basedir="自己本地路径"					#根目录#
###  （可选）5.tmpdir="自己本地路径"					#临时目录#
###  （可选）6.datadir="自己本地路径/data"				#数据目录#
###  	     7.read-only=0						#主机，读写都可以#
###  （可选）8.binlog-ignore-db=mysql					#设置不需要复制的数据库
###  （可选）9.binlog-do-db=需要复制的主数据库名字			#设置需要复制的数据库#
##2.  从机(windows)修改my.ini配置文件
###  （必选）##1.server-id=2						#从服务器唯一ID
###  （可选）2.log-bin=自己本地的路径/mysql-bin				#启用二进制日志#
##3.  主从都重启mysql服务
##4.  主从都关闭防火墙
##5.  主机上建立账户并授权slave
### 建立用户：create user 'zhangsan'@'192.168.33.1' identified by 'Zsq0606.';
### 分发权限：grant REPLICATION SLAVE on *.* to 'zhangsan'@'192.168.33.1';
### 修改密码策略：use mysql; alter user 'zhangsan'@'192.168.33.1' identified with mysql_native_password by 'Zsq0606.';
##6.  从机上配置需要复制的主机
###change master to master_host='192.168.33.129',master_user='zhangsan',master_password='Zsq0606.',master_log_file='mysql-bin.000014',master_log_pos=156;

###启动从服务器复制功能：start slave;
### 查询配置是否成功：show slave status\G; 参数 Slave_IO_Running: Yes Slave_SQL_Running: Yes 则说明启动成功
### slave_io_running:no
### slave_io_ruing:connecting  注意linux服务器上mysql高版本的密码策略  
###slave_sql_running:no 解决办法stop slave;SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1; START SLAVE;   start slave;  show slave status\G;
