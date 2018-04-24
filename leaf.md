#leaf

##ChanRpc
Rpc是远程过程调用的简称，原来是通过Tpc手段使得一个本地的函数调用，将调用信息传递给其他服务器执行，并通过Tcp返回结果的一种技术手段，在分布式中，这种调用十分常见。但是这里的ChanRpc是在不同协程中进行函数调用，其实现的手段是Chan，所以成为ChanRpc。你可以将不同的功能模块实现为ChanRpc并提供给其他模块调用

##Cluster
Cluster 主要是管理集群，但是Leaf本身专注的还是单机服务，所以这个模块的功能现在还没有实现。

##Conf
Conf 是Leaf的配置管理模块。里面主要是Leaf启动的一些必要信息

##Console
Console 模块为Leaf管理提供了一个终端接口，你可以使用Telnet连接上去动态的修改参数，或者指向命令。其内部实现了Help, CpuProf, Prof命令，并提供扩展，可以方便的添加其他命令。另外，扩展命令是通过ChanRpc实现的

##DB
DB模块提供里Mongo支持，也可以在这里聚合其他DB模块

##Gate
Gate 模块为Leaf提供接入功能。这个模块的功能很重要，是服务器的入口。它能同时监听TcpSocket和WebSocket。主要流程是在接入连接的时候创建一个Agent，并将这个Agent通知给AgentRpc。其核心其实是一个TcpServer和WebScoketServer，他的协议函数能够将socket字节流分包，封装为Msg传递给Agent。其工作流可以查看Server模块

##Go
Go模块是对golang中go提供一些额外功能。Go提供回调功能，LinearContext提供顺序调用功能。

##Log
Log主要提供日志分级功能。

##Module
Module 为Leaf提供模块化支持。Skeleton是Leaf的整体骨架，它聚合了Leaf中其他一些异步调用模块的功能，使得各模块之间能够协同工作。

##Network
Network是Leaf的网络部分，这部分比较大，而且包含一个json和protobuf解包模块。

##Recordfile
Recordfile 提供序列化和反序列化为文本的功能。

##Timer
Timer主要是提供一个Cron功能的定时器服务，其中Timer是time.AfterFunc的封装，是为了方便聚合到Skeleton中。

##Leaf 服务器的其他设施
主要是util提供的一些功能

deepcopy
经行深拷贝，建立数据快照，并同步到数据库中。

map
对原始map封装，提供协程安全的访问。

rand
对原始rand的封装，提供一些高级的随机函数。

semaphore
用chan实现的信号量。
