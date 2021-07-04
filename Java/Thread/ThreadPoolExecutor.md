#线程池

-----------------------------------------------

常见问题
    
    （1）ThreadPoolExecutor有几个构造方法？ 
        答:四个 固定corepoolsize,maxpoolsize,keepalivetime,timeunit,blockingqueue
        还有两个参数threadfactory,rejectexecutorhandler刚好拼成三种可能 加上上面说的总共四个
    
    （2）ThreadPoolExecutor最长的构造方法有几个参数？
        7
    
    （3）keepAliveTime是做什么用的？
        控制非核心线程数拿不到任务时什么时候销毁    
        allowCoreThreadTimeOut=true时 对核心线程也有效 默认为false 
        对核心线程无效

    
    （4）ConcurrentLinkedQueue能不能作为任务队列的参数？
        不行 不是blockingQueue
    
    （5）默认的线程是怎么创建的？
        线程工厂生产
    
    （6）如何实现自己的线程工厂？
        默认使用的是Executors工具类中的DefaultThreadFactory类，
        这个类有个缺点，创建的线程的名称是自动生成的，
        无法自定义以区分不同的线程池，且它们都是非守护线程。
        可以自己实现一个ThreadFactory，然后把名称和是否是守护线程
        当作构造方法的参数传进来就可以了。
    
    （7）拒绝策略有哪些？
        AbortPolicy, 丢弃并且报异常
        DiscaryPolicy, 直接丢弃
        DiscaryOldestPolicy, 丢弃队头 新任务添加到队尾
        CallerRunsPolicy, 调用excute方法的线程执行 如果excutor shut down了 则丢弃
    
    （8）默认的拒绝策略是什么？
        默认AbortPolicy 

    （9）线程池状态
        running shutdown stop tidying terminated
        