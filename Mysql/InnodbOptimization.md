#Innodb优化

-----------------------------------

全局状态查询: show engine innodb status 


缓冲池配置:

    命令式调整:set global innodb_buffer_pool_size =
    文件式调整:写道con.f配置文件中
    主要放索引、数据等
    对于mysql专用服务器 此范围在20%~80%总内存之间 对于大型机器 比如128g内存的
    显然比例可以调大点 另外 也要看数据库中索引和文件的数据量 假设总的才5g数据量
    设置20g缓冲池完全没有意义 要维护缓冲池也是费时间的 
    SELECT CEILING(Total_InnoDB_Bytes*1.6/POWER(1024,3)) RIBPS FROM
    ->     (SELECT SUM(data_length+index_length) Total_InnoDB_Bytes
    ->     FROM information_schema.tables WHERE engine='InnoDB') A;
    计算所有的数据和索引数据量 *1.6 

    对于不是mysql专用的服务器 使用内存就要考虑别的进程
    SHOW GLOBAL STATUS LIKE 'innodb%read%' 显然innodb read相关参数
    计算缓冲命中率: 缓冲池读次数/( 缓冲池读次数+ 预读数 + 物理磁盘读次数)
    太低了可以跳高点 尽量保证99%
    缓冲命中率在 show engine innodb status 显示有中 

    另外还有个 innodb_buffer_pool_instances 缓冲池实例个数
    在高并发环境下 可以设置此参数为 4~16 视情况而定 减少内部锁争用(进程内部所有线程
    对共享资源的互斥和同步访问)

    
LRU

    用LRU算法管理缓冲池页(默认16kb)  但是有所改变
    innodb_old_blocks_pct 控制最近访问的页所放位置mid mid=0表示尾端
    100变成末尾 37表示放在尾端37%位置徐  mid之前为new 之后为old
    innodb_old_blocks_time 用于表示页读取到Mid位置后 需要多久会被加入到LRU
    列表的new端 
    全表扫描会造成LRU污染 


重做缓冲日志:

    redo log buffer 首先将重做日志信息放入到这个缓冲区 再刷新到重做日志文件
    一般每秒会刷新一次 因此这个buffer的大小大于每秒产生的事务量就行了


额外内存池:

    放控制信息等元数据的 缓冲池增大时 这个也要相应增大

刷新页:

    innodb_io_capacity master thread里面刷新页到磁盘的能力
    对于固态硬盘可以将此值调大点
    