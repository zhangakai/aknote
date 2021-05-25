#TCP

简单的总结:

    tcp协议核心就是一个发送-确认-发送-确认.......的过程 这个过程中可能会出现更多问题 
    比如发送的tcp报文段丢了怎么办 回复的确认丢了怎么办 这些都可以认为是接收端没收到 
    那发送端再重发就是了 剩下的就是许多细节实现了

    一步步设计一个可靠的传输协议:
    case 1:接受端收到了重复的数据怎么办?
    为了识别数据是否已被收到过以及是否需要被接受 为每个tcp报文段标上序号就行了 这就是tcp头中seq段的作用 
    case 2:重发时间怎么确定?
    由于网络一直波动  所以重发时间不能是固定的(当然也可以 不过要设置的大点 这样重发时间长了会有明显的延迟)
    



首部:
    
    IP包里面封装TCP报文段 TCP首部一般20字节 最大可拓展到60字节
    16位源端口号-16位目的端口号-32位序号-32位确认号-4位首部长度-保留6位-
    -ack-psh-rst-syn-fin-16位窗口大小-检验和-紧急指针-选项-数据 

三握:

    发送端发送SYN报文:SYN=1 seq=一个选择的初始值(有算法计算isn) windowsSize= options[mss,窗口扩大因子 发送端时间戳 时间戳应答(刚连接为0)....]
    接收端发送SYN-ACK报文:SYN=1 ACK=1 seq=随机初始值 ack=上面SYN报文中的seq+1 .....其他跟上面差不多 (ack=seq+length 虽然上面的length=0 
    但是默认SYN报文数据长度就是一个字节)
    发送端发送ACK=1 seq=上面seq+1 可以携带数据
    当一端为建立连接而发送它的SYN时 它为连接选择一个初始序号ISN随时间而变化，
    因此每个连接都将具有不同的ISN。RFC 793 [Postel 1981c]指出ISN可看作是一个32比特的计
    数器 每4ms加1 这样选择序号的目的在于防止在网络中被延迟的分组在以后又被传送 而
    导致某个连接的一方对它作错误的解释 
    另外ISN可以防止序列预测攻击

    两次握手行不行?(服了这种问题了)
    答案:首先seq字段字段是用来跟踪双发发送的内容 为了使tcp报文段交付给上层协议的时候有序以及重发的时候知道重发那端
    建立连接时序号不能从0开始(容易发生序列预测攻击) 所以要初始一个随机的值
    stackoverflow上面一个例子:
    Alice ---> Bob    SYNchronize with my Initial Sequence Number of X
    Alice <--- Bob    I received your syn, I ACKnowledge that I am ready for [X+1]
    Alice <--- Bob    SYNchronize with my Initial Sequence Number of Y
    Alice ---> Bob    I received your syn, I ACKnowledge that I am ready for [Y+1]
    可以简化为
    Bob <--- Alice         SYN
    Bob ---> Alice     SYN ACK
    Bob <--- Alice     ACK    
    上面短缩为2次的话 alice只知道自己的初始seq已经被bob百分百收到了 但是bob不知道自己的seq是不是被alice收到了啊。。。。

    linux中的一些实现:
    客户端请求建立连接时 未收到回复会超时重连 时间分别是 1 2 4 8 16 32 最多尝试重连五次 最后一次等32秒 还连不上就返回错误
    此由/proc/sys/net/ipv4/tcp_syn_retries定义

    SYN泛洪攻击: /etc/sysctl.conf 中 设置net.ipv4.tcp_syncookies = 1 当出现SYN等待队列溢出时 启用cookies来处理

四挥:
    
    一样的套路
    c FIN---> s
    c <---ACK s
    c <---FIN ACK s
    c --->ACK s

    这下不能向上面一样变三次了 因为s可能还有数据要发(在实现上 分为全关闭和半关闭 前者指客户端不想收数据了 后者表示还能收 发吧发吧)

    c收到FIN-ACK后 会进入time_wait状态 这个状态的用处:
    可靠地结束tcp连接---如果c收到FIN-ACK 然后发送出ACK后 立即关闭连接 ACK一旦丢失 s未收到ACK会超时重传FIN-ACK 结果c已经关闭了 
    这个FIN-ACK到达c后 c会发送一个RST报文段 s会认为连接出错了(不同的系统处理RST实现可能不一样)
    保证让迟来的TCP报文段有足够的时间被识别并丢弃---试想没有time_wait c关闭后又立即向s请求建立连接 这个连接 源地址目的地址源端口目的端口
    都跟刚才关闭的连接一样 此时 上次连接中一些延迟的报文段来到了c 那不就出问题了吗

    为什么2MSL:这2个MSL中的第一个MSL是为了等自己发出去的最后一个ACK从网络中消失
    而第二MSL是为了等在对端收到ACK之前的一刹那可能重传的FIN报文从网络中消失

    Nginx 作为反向代理时 大量的短链接 可能导致 Nginx 上的TCP连接处于time_wait状态
    每一个time_wait状态 都会占用一个「本地端口」新建立TCP连接会出错 address already in use : connect
    可以启动socket复用或者减小time_wait时间解决 
    net.ipv4.tcp_tw_reuse = 1 表示开启重用 允许将TIME-WAIT sockets重新用于新的TCP连接 默认为0 表示关闭
    net.ipv4.tcp_tw_recycle = 1 表示开启TCP连接中TIME-WAIT sockets的快速回收 默认为0 表示关闭。


一些常见问题:

    tcp是字节流协议 没有所谓的粘包 它根本不知道发送的字节流代表什么 应用层负责做数据解释 
    
    流量控制窗口:即主机端为这个连接创建的缓存空间还有多少可用 
    拥塞控制窗口: 网络中的拥塞避免建立的一套机制

    RST报文段:RST=1 win=0 seq=0 
    
    nagle算法:一个TCP连接的双方在任意时刻最多只能发送一个未被确认的TCP报文段 在该报文段的确认到达之前
    不能发送其他TCP报文段 同时发送方在等待确认期间收集需要发送的微量数据(即携带交互数据的小tcp报文段)
    确认来了时 将这些数据全部发出 


超时重传:

    Linux中 每次TCP报文段都有一个重传定时器 该定时器在TCP报文段第一次被发送时启动 如果超时时间内未收到对方的应答 
    则重传TCP报文段 并重置对应的定时器 最多重传5次 时间为0.2 0.4 0.8 1.6 3.2 单位为秒 
    tcp_retries1 tcp_retryies2 参数 分别为最小重传次数和最大重传次数

拥塞控制:
    
    传输超时时重新进行慢启动和拥塞避免 到达门限之前窗口不断*2(一个拥塞避免窗口大概2~4个SMSS) 到达之后+1 
    传输超时时门限/2 重复这套逻辑
    
    接受到重复的确认报文段时进行快速重传和快速恢复 收到3个重复ack后 立即重传 窗口大小=门限值

    最终的发送窗口取拥塞窗口和对方接收窗口的较小值


与UDP的区别:

| TCP |	UDP |
|-----| --- |
|有连接 可靠协议|	无连接 不可靠|
|字节流协议 超过MSS会被分割 |	让发多少一次发多少 不会分割 |
|保证packet有序交付 |	不保证有序 因为数据都在一个packet里面|
|慢 因为要维护可靠性 | 快|
|头部一般20字节|	8字节|
|要握手|不用|
    
    
    