nc 命令在有些时候非常有用，它可以用来测试网络端口，基本用法：

`nc -v dest_ip port

示例：
```
nc -v vcenter01 443
nc -v ave-01 28001
```


我们在做VMware恢复作业的时候经常需要检查ESXi 是否能连通 DD 的几个NFS端口：

`TCP/111、2049、2052

而ESXi主机命令行不提供 telnet 和 curl 这些我们常用的命令，nc就能有大用：
```
nc -v ddve-01 111
nc -v ddve-01 2049
nc -v ddve-01 2052
```

它还可以批量扫描端口：

`nc -v -z -w1 ave-01 28001-28011

如果发现不通，很有可能是vSphere的firewall没有开放这些端口。需要虚拟化管理员调整firewall策略。

nc的作用并不止于此，它既可以做客户端，还可以做服务端，用来通过自定义端口来监听网络包。

```
服务端：nc -l 28007
客户端：nc -v ave-01 28007
```

甚至还可以传输文件，对一些不方便收日志的环境比较有用。用法如下：

接收端执行：

`nc -l 28007 > esxi.gz

可以监听本地28007端口（假设在ave-01上监听）并将收到的数据写入 esxi.gz 这个文件

发送端执行：

`nc -v ave-01 28007 < /vmfs/volumes/xxx.gz

可以将本地的日志包文件 /vmfs/volumes/xxx.gz 文件发送到 ave-01 的28007端口