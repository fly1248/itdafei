#IT #VMware

VMware Tools 是运行在基于 VMware 虚拟化平台（如 vSphere、Workstation、Fusion 等）的虚拟机中的客户操作系统内的一款软件插件。运行虚拟机并非必须使用它，但它确实能提供显著的性能提升和额外功能，因此 VMware 强烈建议在全新安装操作系统以制作主镜像后安装它。

VMware Tools 在 VMware ESXi、VMware Workstation、VMware Player 和 VMware Fusion 中本质是相同的，只是所需的功能有所不同。其他虚拟化平台也有类似的概念，例如微软 Hyper - V 有集成服务，Nutanix AHV 有 NGT（Nutanix 客户工具）。

#### 安装 VMware Tools 有诸多好处

- 它可以提供新的设备驱动程序，包括准虚拟化的 VMXNet 网卡驱动、准虚拟 SCSI 驱动和音频驱动等；
- 能改善鼠标和视频性能，比如在 HTML5 控制台中也能有更好的鼠标体验，通过 SVGA 驱动提升视频效果；
- 还可以改进内存管理，添加用于内存 Ballooning 技术的 memctl 驱动。利用该技术，宿主机可以在内存短缺的时候从内存充足的 VM 中回收内存。
- 此外，它还具备虚拟机监控功能，包括新的图表、VM 心跳（用于如 vSphere HA VM 等）、详细的客户机信息（包括 IP 地址、客户机操作系统主机名等）；
- 能实现客户机时间同步；
- 支持文件系统静默，这对于创建一致的快照很有帮助；
- 可以运行自定义脚本；
- 作为 vCenter Server 和其他 VMware 产品的一部分自定义客户机操作系统；
- 能够正常关闭或重启 VM；
- 在 Workstation、Player 或 Fusion 上，还可以与主机操作系统共享文件等。

  
#### 有哪些情况下是不建议安装 VMware Tools 的吗？

虽然有些准虚拟化设备驱动现在已包含在 Linux 内核甚至 Windows 主线驱动更新中，但 VMware Tools 不仅仅是驱动程序。

有人说它会降低客户机操作系统的安全性，这种说法见仁见智。也有人因为它会禁用一些相关功能而不想安装，比如不启用虚拟机中的内存 Ballooning 技术，但其实有更好的方法来禁用特定功能。

需要注意的是，也有一些有问题（已弃用）的 VMware Tools 版本，如 10.3.0，因为存在 VMXNET3 驱动相关问题（更多信息见 VMware KB 57796），这些版本绝不应该安装。

https://virtualizationdojo.com/vmware/install-vmware-tools-guide/#Why_Install_VMware_Tools_