#netty

----------------------------------------

1.异步和事件驱动

    channel: 就是一个连接
    回调: 就是一个方法 需要注册 类似于
    void connection{
        ....
        ....
        callback();
    }
    Future: java 原生的实现 需要手动get() netty实现的可以异步
    