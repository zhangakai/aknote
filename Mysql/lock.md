#mysql lock 种类

-------------------------------------------

1.shared and exclusive locks

    简称s锁 x锁
    行级锁 只有 s-s 兼容 其他都不兼容 

2.intention locks

    IX IX 意向读锁 意向写锁 为了在加表级锁的时候 快速检测是否有行被锁住
    IS锁表明一个事务意图在几行上加个S锁
    IX锁表明一个事务意图在几行上加个X锁
    表级X　S锁与IX　IS的兼容情况

            X	        IX      	S	        IS
    X	Conflict	Conflict	Conflict	Conflict
    IX	Conflict	Compatible	Conflict	Compatible
    S	Conflict	Conflict	Compatible	Compatible
    IS	Conflict	Compatible	Compatible	Compatible

3.record locks

    记录锁是索引记录的锁 
    SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE;
    防止任何其他事务插入，更新或删除T.C1值为10的行


