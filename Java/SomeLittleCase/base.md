#一些基础问题

Finally:

    什么时候不会执行？
    调用System.exit()
    JVM在finally之前崩溃
    如果JVM在try或catch块中有无限循环
    强制停止JVM进程 kill-9
    如果finally块由守护程序线程执行 并且所有其他非守护程序线程在则在调用finally之前推出

    try{
        return 0;
    }finally{
        return 1;    
    }
    编译器编译的时候会把finally块放在try块return语句********之前