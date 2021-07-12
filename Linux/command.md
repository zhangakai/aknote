#命令

    grep 输出符合匹配模式的行
    -i ignore 忽略大小写
    -l list 列出本目录包含匹配模式的文件
    -r recursion 列出本目录以及子目录包含匹配模式的文件
    -n number 列出匹配的行在原文件中的行号
    -w word  匹配整个单词
    -v verse 反转 意思是输出不匹配的行
    "^foo" 以foo开头
    "foo$" 以foo结尾
    -A after 指定匹配后显示多少行
    
    awk 进行匹配然后可以对匹配的行做修改 一般用grep做匹配 awk做修改
    用法 awk '____  {action}' filename
    例如
    awk '/mange/ {print} ' filename  
    print $0 输出所有列
    print $1 输出第一列
    print NR, $1 NR 打印行和行号 number row
    awk ‘NR==3, NR==6 {print NR,$0}’ test 显示第三行到第六行
    awk '{print NF}' test 每行字段数 number filed  awk '{print NF}' test  
    awk '{print $NF}' test   输出最后一列
    awk 'OFS="/" {print $1,$4}' test  Andy/45000 指定分隔符 output filed separator
    awk 'BEGIN {print "my test file"} {print $0}' test 一眼就看出来没什么好说哒哒哒哒
    
    sed 流编辑器 用于搜索 查找 替换 插入 删除 常用来查找和替换
    
    
    
    磁盘
    df Report file system disk space usage
    df -h human readable 查看全局磁盘空间利用信息
    df -sh 当前目录磁盘信息
    
    
    ps
    ps -ef 使用标准语法查看系统中的每个进程 = ps -aux(当前在内存内的进程)
    ps -eLf 要获得关于线程的信息
    ps -T -p 查看进程的线程
    
    
    top
    top -H 查看线程
    top 查看进程
    
    
    lsof list standard output information
    lsof -u username 查看username打开的所有文件
    lsof -i tcp:port 查看特定端口的所有运行进程
    lsof -i 4 查看IPv4
    lsof -i 6
    lsof -i tcp:1-1024
    lsof -i -u username 找出username跑的命令
    lsof -i list all network connections
    lsof -p num search by pid (会显示多个用户)
    kill -9 `lsof -t -u username` Kill all Activity of Particular User
    
    find
    (1)find / -name httpd.conf　　#在根目录下查找文件httpd.conf，表示在整个硬盘查找
    (2)find /etc -name httpd.conf　　#在/etc目录下文件httpd.conf
    (3)find /etc -name 'srm'　　#使用通配符(0或者任意多个)。表示在/etc目录下查找文件名中含有字符串‘srm’的文件
    (4)find . -name 'srm' 　　#表示当前目录下查找文件名开头是字符串‘srm’的文件
    
    2.按照文件特征查找
    (1)find / -amin -10 　　# 查找在系统中最后10分钟访问的文件(access time)
    (2)find / -atime -2　　 # 查找在系统中最后48小时访问的文件
    (3)find / -empty 　　# 查找在系统中为空的文件或者文件夹
    (4)find / -group cat 　　# 查找在系统中属于 group为cat的文件
    (5)find / -mmin -5 　　# 查找在系统中最后5分钟里修改过的文件(modify time)
    (6)find / -mtime -1 　　#查找在系统中最后24小时里修改过的文件
    (7)find / -user fred 　　#查找在系统中属于fred这个用户的文件
    (8)find / -size +10000c　　#查找出大于10000000字节的文件(c:字节，w:双字，k:KB，M:MB，G:GB)
    (9)find / -size -1000k 　　#查找出小于1000KB的文件
    find /tmp/cg/testLinux -name "*.log"
    
    
    ser -n '/start_time/,/end_time/' filename

    tail -f filename 滚动输出日志


