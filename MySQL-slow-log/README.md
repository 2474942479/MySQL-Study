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
