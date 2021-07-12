#命令及问题排查


java command

    jps
    - l: Output main class full name or jar path
    jmap jstat jinfo jstack 



Java cpu使用率过高

    1.top 定位Java进程ID 看哪个进程使用最多
    2.top -h -p pid 上一步的进程拿过来 看哪个线程使用最多 这里用tid表示
    3.jstack pid -> filename  将线程栈信息输出到文件里 可以生成三次 数据更准确
    4.将pid 转化为16进制 cat jstack-output.txt | grep -i tid

    可能的原因
    1.分配实例频繁 导致gc频繁 最终造成cpu  +++++ 啊 好无聊 实习没人管我每天写写笔记。。。
    2.死锁 线程争用问题 
    3.IO操作 如写入文件系统或与后端关系数据库管理系统或消息队列的交互
    一个错误配置的数据库连接池 其中不断创建和销毁资源以提供与Spring和JPA的应用程序的Java数据库连接
    可以触发高Java CPU使用率 糟糕的I / O资源管理也可能导致泄露的内存
    4.应用程序具有大量HTTP请求 以及每个请求响应周期上解析JSON和XML的关联开销
    通常会触发cpu ++++
    5.自己代码的问题 包括但不限于
    a.无限循环 b.递归写错了 c.数据结构选错了 如在大量数据上不用arraylist 用踏吗的LinkedList
    d.多次计算一个值 建议用临时变量保存下来 


内存泄露

    内存泄漏是存在于不再使用的堆中存在的对象的情况
    但垃圾收集器无法从内存中删除它们 还要维护它们（辛苦了gc大哥)
    
    内存泄漏的一些现象
    当应用程序连续运行很长时间时 性能严重下降
    爆outofmemoryError
    自发和奇怪的应用程序崩溃
    偶尔会耗尽连接对象

    潜在原因
    1.大量运用静态字段
    2.用了资源忘了close
    3.equals() and hashCode() 没写好 导致hash类插入不能覆盖旧对象
    4.非静态内部类实例化时需要实例化外部类 建议写成静态内部类
    5.重写类的'finalize（）方法时 GC将它们放进一个队列进行最终确定
    相当于延迟收集了 如果此时有大量新对象创建 会爆outof...
    6.threadlocal 使用不当 记得不用的对象及时调用.remove()方法
