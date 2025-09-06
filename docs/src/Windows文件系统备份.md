#IT #Windows

Windows 系统是如何备份的？

我们来解析Avamar备份Windows文件系统的过程，它的日志记录的过程大致是：

1. 列出备份目标磁盘，和文件系统类型：
    
```
2025-04-02 07:30:03 avtar Info <7324>: Volume Type for "C:\" is "NTFS", Supports Compression=1, Encryption=0, ACLS=1, DataStreams=1, Reparse=1, Sparse=1, Hardlinks=1

2025-04-02 07:30:03 avtar Info <7324>: Volume Type for "D:\" is "FAT32", Supports Compression=0, Encryption=1, ACLS=0, DataStreams=0, Reparse=0, Sparse=0, Hardlinks=0

2025-04-02 07:30:03 avtar Info <7324>: Volume Type for "E:\" is "NTFS", Supports Compression=1, Encryption=0, ACLS=1, DataStreams=1, Reparse=1, Sparse=1, Hardlinks=1

2025-04-02 07:30:03 avtar Info <7324>: Volume Type for "F:\" is "NTFS", Supports Compression=1, Encryption=0, ACLS=1, DataStreams=1, Reparse=1, Sparse=1, Hardlinks=1
```


2. 遍历 exclude list，排除不需备份的文件
    
3. 对分区的备份，会请求创建VSS卷影副本：
```
2025-04-02 07:30:10 avtar Info <8780>: Creating the shadow copy set (DoSnapshotSet) ...
    
然后立即检查卷影副本的结果：
    
2025-04-02 07:33:35 avtar Info <8803>: Querying all shadow copies with the SnapshotSetID {7b153d09-e66d-4f60-b407-cda2381f6486} ...
```      
        
4. 连接Avamar存储和连接DD：
```
2025-04-02 07:33:37 avtar Info <0000>: - Connecting to:
DataDomain: usa003.example.local
```
      
    
5. 开始和完成正式的数据拷贝过程：
```
avtar info 7563 Back up of "E:" on Server "ave01.delledu.lab" for /clients/sql01.delledu.lab
    
拷贝使用的VSS卷影副本路径为：
    
    Volume "\\?\Volume{7e4d512e-d794-4912-8d0e-927893b1660b}\
    

我们如果希望通过模拟这个过程，可以 通过 vssadmin 命令来创建和查看卷影副本：

vssadmin list shadows
vssadmin create shadow /For=C: /AutoRetry=2

Shadow Copy Volume 的路径可以直接被挂载到一个符号链接：

cmd /c mklink /d C:\ShadowMount "\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy3\"


如需将内容拷贝出来作备份可以用：

robocopy C:\ShadowMount E:\backup /MIR
```   


关于VSS的官方文档： https://learn.microsoft.com/zh-cn/windows/win32/vss/volume-shadow-copy-service-overview
        
这篇文章也不错：
https://learn.microsoft.com/zh-cn/sql/relational-databases/backup-restore/sql-server-vss-writer-backup-guide?view=sql-server-ver17