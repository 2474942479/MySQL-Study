# MySQL-Study MySQL8.0优化学习
## innodb引擎和myisam引擎区别:
 1.myisam是默认表类型不是事物安全的；innodb支持事物。
 
 2.myisam不支持外键；Innodb支持外键。
 
 3.myisam支持表级锁（不支持高并发，以读为主）；innodb支持行锁（共享锁，排它锁，意向锁），粒度更小，但是在执行不能确定扫描范围的sql语句时，innodb同样会锁全表。
 
 4.执行大量select，myisam是最好的选择；执行大量的update和insert最好用innodb。
 
 5.myisam在磁盘上存储上有三个文件.frm(存储表定义)  .myd（存储表数据）  .myi（存储表索引）；innodb磁盘上存储的是表空间数据文件和日志文件，innodb表大小只受限于操作系统大小。
 
 6.myisam使用非聚集索引，索引和数据分开，只缓存索引；innodb使用聚集索引，索引和数据存在一个文件。
 
 7.myisam保存表具体行数；innodb不保存。
 
 8.delete from table时，innodb不会重新简历表，而会一行一行的删除。
