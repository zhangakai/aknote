#锁

-------------------------------------------

1.锁种类

    公平-按排队顺序来 
    非公平-可以直接尝试获取锁 不管有没有排队的 吞吐量大但是可能会造成饥饿
    synchronized是非公平锁 
    
    可重入-同一个线程可以重复获取锁 避免死锁

    独享、共享

    乐观、悲观

    分段锁

    偏向、轻量级、重量级

    自旋


2.内存模型

    将内存分为主内存和工作内存 每个线程有独立的工作内存
    Java内存模型规定了所有的变量都存储在主内存
    线程的工作内存中保存了被该线程使用的变量的主内存副本 
    线程对变量的所有操作（读取、赋值等）都必须在工作内存中进行
    而不能直接读写主内存中的数据 不同的线程之间也无法直接访问对方
    工作内存中的变量 线程间变量值的传递均需要通过主内存来完成


    内存屏障
    位于unsafe类中 1.8之后核心只有三个 之前是loadbefore loadafter这种更细分的
    /**
     * Ensures lack of reordering of loads before the fence
     * with loads or stores after the fence.
     * @since 1.8
     */
    public native void loadFence();

    /**
     * Ensures lack of reordering of stores before the fence
     * with loads or stores after the fence.
     * @since 1.8
     */
    public native void storeFence();

    /**
     * Ensures lack of reordering of loads or stores before the fence
     * with loads or stores after the fence.
     * @since 1.8
     */
    public native void fullFence();