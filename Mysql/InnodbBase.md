#基础知识

--------------------------------

checkpoint

    目的:
    缩短数据库的恢复时间 
    缓冲池不够用时 将脏页刷新到磁盘
    重做日志不可用时 刷新脏页
    宕机时 checkpoint 之前的页都刷回到磁盘了 只用重做checkpoint之后的

    两种checkpoint
    sharp checkpoint 数据库关闭时 将所有的脏页刷新到磁盘
    fuzzy checkpoint 数据库运行时使用 只刷新一部分脏页

    fuzzy checkpoint又分为
    master thread 每秒或每十秒刷新一定比例的脏页到磁盘 异步 
    flush_lru_list checkpoint 需要保证LRU列表中有1024(默认)个空闲页 
    不够则需要从尾部刷新脏页
    async flush checkpoint 
    sync flush checkpoint 这两个都是重做日志不可重时启用 强制将一些页刷新回磁盘
    dirty page too much 脏页太多 innodb_max_dirty_pages_pct = ? 启用 

    
Master Thread:

    1.0x版本之前
    由多个循环组成-master loop,background loop, flush loop,suspend loop
    在这些loop中来回切换    

    master loop里面有:
    每秒一次的操作---
    日志缓冲刷新到磁盘 即使事务未提交(总是)下面的可能执行
    合并插入缓冲
    至多刷新100个脏页(一页16k跟操作系统那个4k的不一样）
    若无活动切换到background loop

    每10秒一次:
    刷新100脏页(可能)下面的总是执行
    日志刷新
    删除无用Undo
    刷新100或10个
    合并至多5个插入缓存

    background loop 里面合并插入缓冲 删除无用Undo 刷新页 

    以上刷新脏页需要看脏页数占比是否超过了innodb_max_dirty_pages_pct 

    1.0x版本之后的优化也都是围绕上面这些来的 留个坑先


插入缓冲:(insert buffer):

    