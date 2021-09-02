# 最小硬件配置

相对而言，SDDC即使是POC环境，对硬件的需求也是很高的。因此有两种选择

* （1台）Azure上的大实例虚拟机 [Dv3 and Ev3 VM size](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/)
* （1台）本地的硬件服务器

  > 注意：Azure Stach HCI由于不使用System Center 来实现，做POC的资源需求量会比SDDC小很多。
  >
  > 注意：POC的设计不支持多台物理主机的联合，投入更多的物理服务器是没有意义的。

## 硬件

如下为建议的物理主机最低配置。

不建议使用`动态内存`

| 资源 | 数量 |
| :--- | :--- |
| 总内存 | 64G |
| 总硬盘 | 1T SSD（建议NVME） |
| CPU | 8代I7  及以上 |

Azure上的实例以下供参考，可根据实际可用资源进行调整。

| Azure资源 | 实例 |
| :--- | :--- |
| 计算 | D16s v4：16 vCPU，64 GB RAM，0 GB 临时存储空间, US$1.632/小时 |
| 存储 | P30: 1024 GiB, 5000 IOPS, 200 MB/秒, US$135.170/月 （根据实际场景，可能需要多盘） |

## 课后习题

- 为什么不建议使用`内存`，简述理由
