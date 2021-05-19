##epoll

-----------------------
先看个大概

函数原型:

    int select
    (int nfds, fd_set *restrict readfds,
    fd_set *restrict writefds, 
    fd_set *restrict exceptfds,
    struct timeval *restrict timeout);

参数说明:

    readfds:想要监听的可以读的文件描述符集合
    writefds:想要监听的可以写的文件描述符集合
    exceptffds:想要监听的出现exception的文件描述符集合
    nfds:一般是上面三个想监听的文件描述符的最大值
    timeout:超时时间，过了这个时间还是没要准备好的描述符就返回
            设置为0时立即返回，设置为Null时一直监听直到有准备好的描述符


过程说明:
    
    假设此时想要调用监听读
    fd_set rset;   //fd_set是一个固定大小的数组
    typedef struct {
        unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
    } 
    FD_SETSIZE大小为1024,所以最大只能监听1024个描述符

    void FD_CLR(int fd, fd_set *set); 
    int  FD_ISSET(int fd, fd_set *set);
    void FD_SET(int fd, fd_set *set);
    void FD_ZERO(fd_set *set);
    
    使用上面四个函数来操作fd_set
    例如监听10号,20号fd 那么就调用 FD_SET(10,rset) FD_SET(20,rset)
    将相应的位置为1
    然后调用 select(21,rset,null,null,null)
    函数内部扫描rset的0到21位，判断对应的描述符是否可读
    可读就置为1,不可读置为0.
    扫描返回后的rset，读位为1的对应的描述符。
    
    伪代码如下:
    while(nowTimeval<timeval){
    for i 0->n
        if (fget(i).isReady)
            fput(i)
    if all fd is not Ready
        continue;
    else
        return;
    }

    
    下面是一个简单实例:
    while(1){
	FD_ZERO(&rset);
  	for (i = 0; i< 5; i++ ) {
  		FD_SET(fds[i],&rset);
  	}

	select(max+1, &rset, NULL, NULL, NULL);
 
	for(i=0;i<5;i++) {
		if (FD_ISSET(fds[i], &rset)){
			memset(buffer,0,MAXBUF);
			read(fds[i], buffer, MAXBUF);
			puts(buffer);}
        }  
    }
    
缺点如下:
    
    1.可以看到，每次都要把rset所有的位置为0，再调用select函数，比较麻烦
    2.假设要监听10和500,那么即使监听两个fd,也要遍历到501，一一判断

    


    

