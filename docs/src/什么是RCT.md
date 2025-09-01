#Hyper-V #备份

什么是 RCT？

**RCT：Resilient Change Tracking**是 Hyper-V 用来跟踪（自上次VM备份以来）变更过的磁盘数据块。其结果是，在下一次对VM进行增量备份时，只有变更过的数据块会被拷贝到目标存储。

该技术在2011年左右，VMware在其ESX 4.0 和其VM 7以上版本就已经实现。这个技术被VMware称为 Changed Block Tracking（CBT）。所以Hyper-V上的RCT~=VMware的CBT

而在Hyper-V中，该技术在Windows2016上才被引入。在Windows2016出来之前（使用RCT要求VM版本在6.2以上，另一个说法是8.0）。要追踪Hyper-V 上的数据变更，只备份已改动的数据块，是由备份软件商各自开发的，专有的数据过滤算法来实现的。

  
**我们来看看RCT的本质。Why RCT？**

在虚拟化环境中，我们创建一个完整的VM备份，将会通过块到块的方式复制得到一个VM的数据完整副本的数据一次又一次的全部复制到备份库，那么每次备份都需很长的备份窗口。

  
下一次备份的时候，理想的情况下是，只有变更过的数据块被再次复制。如果我们把整个VM再备份一次，会占据更多的存储空间，备份中处理数据也需要更长的时间。但大量的数据块其实是重复的。

  
利用微软 RCT，数据保护软件可以用一种很高效的方式来进行增量备份——**只复制变更过的数据块，不需要复制重复的，已存在于备份库的数据块。**


**Hyper-V RCT 是如何工作的？**

RCT 会创建一个VM所有数据块的**映射**。下次备份时，首次备份以来的修改过的数据块可以通过变更追踪信息来得知。


RCT 使得Hyper-V具备这样的能力——在硬件故障和意外断电的情况下，也能保存VM的变更追踪信息。Windows Server 2016 Hyper-V 通过3个变更追踪文件来实现这一能力，1个在内存中，两个在磁盘中。这样在硬件故障或意外断电时，即便内存中的变更追踪文件副本已经丢失，我们还有磁盘中的副本。磁盘中的两个文件是在首次（完整）备份时创建的。

  
在创建检查点（曾经也被称为“快照”）时，我们可以看到 一个 .avhdx 文件和与之相对应的一个.mrt 和 一个.rct 文件。如果你在备份过程中没有看到这些文件，那么很可能没有设定为RCT备份，而是使用的备份软件商自开发的备份算法。


当备份完成，检查点被移除时，磁盘上会留下原VM的 .vhdx 和对应的 mrt、rct文件。

  

**RCT or Resilient Change Tracking file** （.rct 文件）

该文件保有磁盘上详细的变更块信息（不过没有内存中的变更映射详细）。文件在“write back”和“cache”模式下被写入，这意味这一些常见的VM操作如迁移、关机、启动等时刻都会使用该文件。

  

**MRT or Modified Region Table file** （.mrt 文件）

该文件是在“write through”模式被使用，尽管它记录磁盘上所有的变更，但它没有RCT文件的使用频率那么高。如果发生了系统崩溃，电源故障，等情况，MRT文件会被用来重构变更块，用来保证这些意外场景下数据块变更信息不会丢失。

  

**为什么需要在磁盘上保存变更块追踪文件？**

内存中的变更块追踪文件仅能在VM长期在一个主机上正常运行时有优势。主机宕机，或者VM迁移时，内存中的变更块追踪文件就丢失了。所以Windows Server 2016 引入 RCT 和 MRT文件来将变更追踪信息持久化到磁盘，以免内存中的信息丢失。

  
备份软件可以通过调用Hyper-V WMI API，来获取RCT的change blocks等信息，从而更有效的备份VM。如：Msvm_ImageManagementService 类中的 GetVirtualDiskChanges 方法，该方法可以通过提供的RCT id 或 VHD数据集的快照id 来获取一个虚拟磁盘在特定区域的变更列表：

GetVirtualDiskChanges method of the Msvm_ImageManagementService class - Win32 apps | Microsoft Learn

内容来源：

What Is Resilient Change Tracking in Microsoft Hyper-V? (nakivo.com)