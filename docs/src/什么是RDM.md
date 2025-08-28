#VMware #RDM

近期在支持客户的过程中看到一个恢复VM过程中遇到了一个难得一见的错误。看到的错误如下：

```
2025-01-14 01:26:16 avvcbimage Info <9823>: COMPARING [CXL1ESXIP32_HOST] CXLSVR1007/CXLSVR1007.vmdk to [CXL1ESXIP32_HOST] CXLSVR1007/CXLSVR1007-000001.vmdk

2025-01-14 01:26:16 avvcbimage Error <0000>: [IMG2002] disk [CXL1ESXIP32_HOST] CXLSVR1007/CXLSVR1007-000001.vmdk is not recorded in the backup and cannot be restored

...

2025-01-14 01:26:28 avvcbimage Info <9819>: Parsed virtual disk directory '1'

2025-01-14 02:03:06 avvcbimage Info <9772>: Starting graceful (staged) termination, MCS cancel (wrap-up stage)
```

咋一看好像是处理快照盘CXLSVR1007-000001.vmdk出错，但实际上却另有原因，作业在1:26就无法继续了，客户等待了37分钟无果于是在2:03取消了作业。

我们查看了VM和备份的情况，发现这个VM的磁盘比较特殊，有33块VMDK，但只有CXLSVR1007.vmdk是正常的200GB虚拟磁盘，被正常备份了；其它32个都是5.625MB大小的Physical磁盘，备份是跳过了它们的。

日志里仔细对比一下磁盘模式发现普通磁盘有标记这个属性：**backing=DiskFlatVer2** ，但Physical磁盘是： **backing=RawDiskMappingVer**1

这里我们大概知道了这些磁盘是 RawDiskMapping （RDM）模式的磁盘，即是映射到物理磁盘的，并非真正的虚拟磁盘。对此，我查看了官网资料：

[techdocs.broadcom.com...](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/7-0/vsphere-storage-7-0/raw-device-mapping-in-vsphere/introduction-to-raw-device-mapping-in-vsphere.html)

通过文档，我们能理解下面两个重点：

1. RDM模式的磁盘是在数据存储中创建了一个链接文件（可以理解为快捷方式），访问这个磁盘，会被链接文件给定向到底层的磁盘中去。
    
2. 底层磁盘可以是虚拟的，也可能是物理的，如果是物理盘（Physical Mode），那么这无法做快照。
    

我们对VM的备份是基于快照的，无法做快照意味着无法被备份，这也解释了32块RDM磁盘被跳过的原因。

但是如何恢复呢？我们的官方文档中对此有所解释：

- 涉及到物理 RDM 磁盘时，无法恢复到新虚拟机

- 备份同时具有虚拟磁盘和物理原始设备映射 (RDM) 磁盘的虚拟机时，备份将处理虚拟磁盘，跳过 RDM 磁盘。

- 从这些备份中的一个恢复数据时，可以将数据恢复到原始虚拟机；也可以恢复到另一个已存在的虚拟机。但是无法将数据恢复到新虚拟机（恢复过程中无法创建）。

注: 由于备份期间没有处理物理 RDM 磁盘，位于物理 RDM 磁盘上的数据完全无法恢复。

这种情况下，如果需要将数据恢复到新的虚拟机，必须：

1. 在 vCenter 中手动创建新虚拟机。

2. 这个新虚拟机的虚拟磁盘数必须与制作备份的原始虚拟机的虚拟磁盘数相同。

3. 手动将新虚拟机添加到 Avamar。（实测发现可以跳过）

4. 将数据恢复到此虚拟机。

Avamar KB000044221 ：[www.dell.com...](https://www.dell.com/support/kbdoc/000044221) 进一步解释了虚拟的RDM磁盘可以备份和恢复，但恢复不包括RDM的map。物理模式是无法被创建快照和备份的。

拓展KB000170861: [www.dell.com...](https://www.dell.com/support/kbdoc/000170861)

所以我们的解决方案是，创建一个新的虚拟机，包含一块200GB的虚拟磁盘和32块6MB的虚拟磁盘，然后在恢复时选择恢复到已有的虚拟机。提交后恢复过程就正常了。恢复结束我们也只需确认恢复了第一个200GB的普通虚拟磁盘。