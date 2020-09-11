# 主从复制：
## MySQL复制过程分成三步:
 1 master将改变记录到二进制日志(binary log）。这些记录过程叫做二进制日志事件，binary log events;

2 slave将master的binary log events拷贝到它的中继日志（relay log) ;

3 slave重做中继日志中的事件，将改变应用到自己的数据库中。MySQL复制是异步的且串行化的

##  复制的基本原则：每个slave只有一个master。每个slave只能有一个唯一的服务器ID。每个master可以有多个salve
## 复制最大问题：延时
## 一主一从常见配置：
### 要求
1. mysql一致且后台以服务运行 
2. 同一网段下。3.主从都配置在[mysqld]结点下，都是小写
### 1. 配置主机:主机(linux)修改my.cnf配置文件

**(必选）1. server-id=1					 *//主服务器唯一ID***
**(必选)2. log-bin=自己本地的路径/mysql-bin 			*//启用二进制日志***

(可选)3.log-err=自己本地路径/自定义错误日志名			*//启用错误日志*

(可选)4.basedir="自己本地路径"		 *//根目录*

(可选)5.tmpdir="自己本地路径"		*//临时目录*

(可选)6.datadir="自己本地路径/data"		*//数据目录*

(可选)7.read-only=0		*//主机，读写都可以*

(可选)8.binlog-ignore-db=mysql	*//设置不需要复制的数据库*

(可选)9.binlog-do-db=需要复制的主数据库名字	*//设置需要复制的数据库*

### 2.配置从机:从机(windows)修改my.ini配置文件
**(必选)1.server-id=2	*//从服务器唯一ID***

(可选)2.log-bin=自己本地的路径/mysql-bin    *启用二进制日志*
### 3.  主从都重启mysql服务
### 4.  主从都关闭防火墙
### 5.  主机上建立账户并授权slave
 (1)建立用户
create user 'zhangsan'@'192.168.33.1' identified by 'Zsq0606.';

 (2)分发权限
grant REPLICATION SLAVE on *.* to 'zhangsan'@'192.168.33.1';

(3)修改密码策略：use mysql; alter user 'zhangsan'@'192.168.33.1' identified with mysql_native_password by 'Zsq0606.';
### 6.  从机上配置需要复制的主机
change master to master_host='192.168.33.129',master_user='zhangsan',master_password='Zsq0606.',master_log_file='mysql-bin.000014',master_log_pos=156;

### 7.启动从服务器复制功能：start slave;
查询是否启动成功：show slave status\G; 

启动成功标志：
参数 Slave_IO_Running: Yes Slave_SQL_Running: Yes 

启动失败解决方案：

slave_io_ruing:connecting  注意linux服务器上mysql高版本的密码策略  

slave_sql_running:no 

解决办法：stop slave;SET GLOBAL SQL_SLAVE_SKIP_COUNTER=1; START SLAVE;   start slave;  show slave status\G;
