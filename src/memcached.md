#memcached用法：参数和命令详解

1. **memcached 的参数**

> -p <num> 监听的TCP端口号，默认是11211；（port） 
> 
> -l <addr> 监听的主机地址，默认是INADDR_ANY，即所有地址，<addr>可以是host:port的形式，如果没有指定port，则使用-p或者-U的值；可以指定多个地址，以逗号分隔或者多次使用-l参数；尽量不要使用默认值，有安全隐患。(listen)  
> 
> -d 以守护进程运行 (daemon)  
> 
> -u <username> 指定进程的所有者（只有以root用户执行时才可以使用该参数）(username)
> 
> -m <num> 用于存储数据的最大内存，单位是MB，默认是64MB；(memory)
> 
> -c <num> 最大并发连接数，默认是1024；
> 
> -vv 显示更详细的信息（还显示客户端的命令和响应）
> 
> -vvv 显示最详细的信息（还显示内部的状态转变）
> 
> -h 显示帮助信息
> 
> -P <file> 将PID保存到<file>中，仅和-d参数一起使用；
> 
> -f <factor> chunk的增幅因子，默认是1.25，不同的slab class，slab page大小相同，但是chunk大小不等，chunk的大小根据这个增幅因子增长；(factor)
> 
> -n <bytes> 为key+value+flags分配的最小内存，单位bytes，默认是48；chunk数据结构本身要占据48字节，所以实际大小是n+48；
> 
> -t <num> 使用多少个线程，默认是4；（thread）
> 
> -I 设置slab page的大小，即设置可以保存的item的最大值，默认1MB，最小是1K，最大值128M；

2. **其它参数**

> -U <num> 监听的UDP端口号，默认是11211，0表示关闭UDP监听；（UDP）
> 
> -s <file> 要监听的UNIX socket路径（禁用网络支持）（socket)
> 
> -a <mask> UNIX socket的访问掩码（access mask），八进制表示，默认是0700. (mask)
> 
> -r 文件数量的最大值 (rlimit)
> 
> -M 内存耗尽时返回错误，而不是通过LRU淘汰内容；
> 
> -k 锁定所有页内存；允许被锁定的内存是有限制的，超过限制可能会失败。
> 
> -v 显示启动信息（错误和警告信息）(verbose)
> 
> -i 显示memcached和libevent的licence信息
> 
> -L 一次申请大的内存页（如果可以）；增大内存页的大小，可以提高性能；
> 
> -D <char> 指定key前缀与ID的分隔符，用于stats信息显示，默认是冒号：，如果使用了该参数，则stats收集自动启用了，否则，需要发送命令“stats detail on”命令来启动stats的收集。
> 
> -R 每一个事件（event）的最大请求数，限制最大请求数可以防止线程饥饿，默认是20；
> 
> -C 禁用CAS；
> 
> -b 设置backlog队列限制，默认1024；
> 
> -B 指定绑定协议，ascii，binary或者auto，其中auto是默认值；