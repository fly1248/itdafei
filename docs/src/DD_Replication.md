#IT #DataDomain #数据保护 #备份 #Replication

</br>
说到复制（Replication），所有的电脑用户都应该很熟悉：**将文件或目录在不同的位置创建一个一模一样的副本**。

DataDomain 的 Replication 功能沿用了这一含义。不过在这个基础上，我们还得了解一些新东西。因为DataDomain的复制概念比我们日常的复制概念要更大一些。
</br>
</br>
## 复制的类型

DD支持 3 种不同类型的Replication：Collection Replication，Mtree Replication，Managed File Replication。DDOS 7.7之前还支持一种 Directory Replication（目录复制）之后将不再支持。估计是因为应用场景比较少的原因。
</br>
### Collection Replication
    
 - Collection Replication 被翻译为“**全量复制**”或许很形象。它会将整个 /data/col1 目录复制到目标DataDomain。这相当于复制了DataDomain上存放的全部用户数据。我们可以理解为做了一个系统的镜像（mirroring），系统账户和密码也会被复制。这种复制只能是一对一，而且复制期间目标端不允许做其它工作。
        
- Collection Replication 另一个重要特性：它是基于底层的复制——复制针对的是数据块，并不关心文件。这种复制对系统资源要求比较低, 因为不用针对每个文件进行反馈确认。可能也正是因为如此，如果Collection复制的联系被断开，将无法再次进行复制。

- Collection Replication 比较适合两种场景
	- 1. 原机因为容量满载或者老旧等原因即将淘汰，需将数据原封不动地迁移到新的DD设备。
	- 2. 异地灾难备份，放一台在异地以防万一。
</br>
        
### Mtree Replication
    
- Mtree 的复制，更接近我们日常对目录进行复制的概念。只不过它的实现方式有些不同。Mtree复制的源端会创建周期性的快照，这些快照的列表被发送到目标端，用与和目标端进行对比，对比后的差异部分将从源端请求。源端再将必要的数据块发送给目标。虽然有些折腾，但这样的机制避免了重复数据的传输，显著地降低了要传输的数据量。
        
- MTree 可以被看做是目录，DD支持多个 MTree 同时复制到目标端，不过也是有上限的。根据型号不同，最多的并发 Mtree 复制在32个到256个之间。

- Mtree 非常适合有选择地进行复制，比如选一些重要的 Mtree 进行复制，一些次要的（临时数据）Mtree 不复制。
</br>
### Managed File Replication
    
如果被叫做“**受控文件复制**”或许有点难理解，但它本质上就是让前端的备份软件管理复制。DD可以和 Avamar、Networker 等软件集成。形成 **【站点1 Avamar1+DD1】<-->【站点2 Avamar2+DD2】** 这样一个架构。此时复制工作完全由 Avamar来管理复制功能，包括要复制哪些内容，DD只在后台作执行底层具体的数据（基于DD Boost进行去重）复制任务。
        