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

    