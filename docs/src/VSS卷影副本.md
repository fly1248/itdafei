#Windows #VSS

这篇是微软官方关于VSS的文档：

https://learn.microsoft.com/zh-cn/windows-server/storage/file-server/volume-shadow-copy-service

理解了这篇文章，我们将知道VSS的工作原理和快照类似，用于备份较大的数据集，或者要求一致性的应用程序数据，比如SQL Server数据库。


由于在执行热备份的时候，这些数据可能正在被应用程序操作，VSS要求它们暂停写入（或将buffer中的脏数据尽快写入，默认超时时间60秒），然后在很快的时间（不超过10秒）创建一个**具备数据一致性**的卷影副本，这个10秒内，对文件系统的所有写入请求都被冻结。

  

VSS服务也不是自己完成所有这些操作，而是和应用程序的VSS writers来协调达到这个目的。Windows 组件（如注册表）的 VSS 编写程序都包含在 Windows 操作系统，其它的writers由应用开发商提供。
  

Windows 自己有一大堆内建的Writers，分别对应各种微软的应用程序。对它们有所了解，对排错会有作用。

- 参考文档：
    
    - https://learn.microsoft.com/en-us/windows/win32/vss/in-box-vss-writers
        

- 获取当前Windows系统中的writers列表，可以用 vssadmin list writers，命令参考：
    
    - https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/vssadmin
        

这里也有一篇介绍得比较全面的中文博文：

https://www.cnblogs.com/suv789/p/17777417.html

  

创建卷影副本完成后，这个**副本**在被删除之前都能一致保持创建时的状态，这提供一个可以随时还原文件到创建时间的还原点。备份软件将拷贝这个副本的数据作为备份。而系统上的应用程序将会恢复对**原来文件**的写入。

  

我在工作中，有遇到一些比较典型的VSS备份错误。有些应对方法可能有比较广泛地适用性，所以分享一下：

1. 存储空间不足。创建卷影副本需要额外的空间，这个空间至少是300MB，之后随着写入的增加，还可能会需要更多。VSS卷影副本创建以卷为单位，所以要保证目标卷有足够的空间。如果不够，系统会记录 **0x8004231f** 的错误码，可以通过事件查看器查看对应时间的应用程序日志错误。
    
    1. 对这个问题进行测试，可以手工创建一个卷影副本看看是否报错。如果是个别卷空间不足，Windows允许将副本创建到其它的卷，例如将卷影副本放到 D 盘 和 E盘：
        
        1. **vssadmin add shadowstorage /for=D: /on=E: /maxsize=200GB**
            
2. VSS writer 出错。原因可能有很多。可以通过 vssadmin list writers 命令的输出来查看是否有错误：
```
    Writer name: 'Microsoft Hyper-V VSS Writer'
    
    Writer Id: {66841cd4-6ded-4f4b-8f17-fd23f8ddc3de}
    
    Writer Instance Id: {c35d6ab0-9588-412f-ae7b-cdc37534501f}
    
    **State: [8] Failed**
    
    Last error: Retryable error
    
    PS：如有发现，一般可以通过Windows事件查看器追踪错误来源
```
    

3. writers 超时。超时可能是这样的错误： 0x800423f2 - writer error: timeout
    
    1. 被单独拿出来说，是因为有时候超时并非writers本身的问题。我们在支持的案例中不止一次看到因为杀毒软件扫描引起的超时——杀毒软件的“On-Access Scan”功能会极大地拖慢VSS处理过程导致超过默认的超时时限。很有必要对VSS和备份软件关闭该扫描功能。
```
Windows提供了一个方法可以在注册表中提高这个时限：

"HKLM\Software\Microsoft\Windows NT\CurrentVersion\SPP\CreateTimeout"
        
        1. 将这个值修改为12000000 （毫秒）(2*10*60*1000 = 20 mins)
            
        2. 如果没有这个键，我们需要手工创建这个 DWORD 32-bit 类型的键。
```
    