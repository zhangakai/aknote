#一些别的协议

##ARP

----------------------------

简单总结:

    APR adress resolution protocol
    查询IP地址对应的MAC地址(一个局域网内) ARP发送一份称作ARP请求的以太网数据帧给以太网上
    的每个主机 目的地址是全f(总长48位 即12个f) 请求数据帧中包含目的IP地址 其意思是“如果你是这个IP地址的拥有者 
    请回答你的硬件地址
    目的主机收到这份广播报文后 发送ARP应答　其他主机收到后丢弃(可能有恶意主机回复假的)
    每个主机都有个ARP缓存 arp -a查询 首先会查询缓存 找不到再发广播帧
    广播帧查不到会重发几次 还是不行就报错


##ICMP

------------------------
