#Geth 控制台使用及 Web3.js 使用
###geth控制台初探 - 启动、退出

```
$ geth console      geth控制台启动成功之后，可以看到>提示符。
退出输入exit
```
###geth 日志控制
```
geth console 2>>geth.log
```

###启动一个开发模式测试节点
```
geth --datadir /home/xxx/testNet --dev console
geth console --dev --datadir='~/go/eth/'

```
###初始化一个节点
```
geth init ./genesis.json --datadir "~/go/eth/"
```
###连接geth节点
```
$ geth attach ipc:/some/custom/path
$ geth attach http://191.168.1.1:8545
$ geth attach ws://191.168.1.1:8546

$ geth attach ipc:testNet/geth.ipc

```