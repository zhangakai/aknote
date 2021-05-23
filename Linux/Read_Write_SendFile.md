#read write sendfile 内核2.6版本


read

    ssize_t read(int fd, void *buf, size_t count);
    read（）尝试从文件描述符fd读取count大小的字节数据到 Buf.
    成功后返回读取的字节数