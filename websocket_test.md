websocket 单机测试

Unix/Linux 基本哲学之一就是 "一切皆文件"，要提高TCP承载量，就需要调整文件句柄

###1

```
永久生效 vim /etc/security/limits.conf

* soft nofile 65534
* hard nofile 65534

重新登录后limit.conf的配置都不生效，后来发现，ubuntu此处的user必须为root
* soft nofile 65534
root soft nofile 65534
root hard nofile 65534

也就是写成上面那样，重新登录，不需要重启，ulimit -a可以看到文件打开数已经是65534了，这就是limits.conf不生效的原因，注意ubuntu一定不能直接用*
```
soft nofile （软限制）是指Linux在当前系统能够承受的范围内进一步限制用户同时打开的文件数

hard nofile （硬限制）是根据系统硬件资源状况(主要是系统内存)计算出来的系统最多可同时打开的文件数量

通常软限制小于或等于硬限制

使用ulimit -n查看当前设置。使用ulimit -n 1000000进行临时性设置

查看当前系统描述符号

cat /proc/sys/fs/file-nr 
###2

```
执行/sbin/sysctl -p即时生效

```

```
echo "net.ipv4.tcp_mem = 786432 2097152 3145728">> /etc/sysctl.conf
echo "net.ipv4.tcp_rmem = 4096 4096 16777216">> /etc/sysctl.conf
echo "net.ipv4.tcp_wmem = 4096 4096 16777216">> /etc/sysctl.conf

```

```
系统最大打开文件描述符数：/proc/sys/fs/file-max
所有进程打开的文件描述符数不能超过/proc/sys/fs/file-max
单个进程打开的文件描述符数不能超过user limit中nofile的soft limit
nofile的soft limit不能超过其hard limit
nofile的hard limit不能超过/proc/sys/fs/nr_open
```

```
根据TCP/IP协议，由于端口是16位整数，也就只能是0到 65535，而0到1023是预留端口，所以能分配的端口只是1024到65534，也就是64511,一台机器一个IP只能创建六万多个长连接。

ifconfig eth0:1 192.168.0.101 netmask 255.255.0.0
ifconfig eth0:1 192.168.0.102 netmask 255.255.0.0
ifconfig eth0:1 192.168.0.103 netmask 255.255.0.0

```

```
Tcp端口信息，电脑内存信息
ss -s ;free -m
watch -n 1 "ss -s && uptime &&free -m"
```

