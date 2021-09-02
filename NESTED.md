# Hyper-V的嵌套虚拟化

由于SDDC的部署，对硬件的需求很大，因此在POC阶段，基于`成本`考虑，建议使用Hyper-V的嵌套虚拟化。
> 在本POC环境中，`必须使用`嵌套虚拟化技术。

该功能可以在Windows Server 2019和Azure的Windows Server 2019上实施。

> Azure上开启嵌套虚拟化，需要特殊实例的配合，请检查产品列表，确认实例支持嵌套虚拟化。
>
> 支持嵌套虚拟化的实例请查看[这里](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/)

## 配置

1. 创建虚拟机，请满足以上需求。
2. 虚拟机处于`关闭状态`时，在宿主虚拟机/物理主机上执行如下命令，以开启嵌套虚拟化，一次只能针对`一台虚拟机`生效。

```
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $true
```
3. 配置网卡的`MAC地址欺骗`
4. 启动虚拟机

## 注意事项

嵌套虚拟化影响性能，因此在环境中务必使用SSD/NVME。

> 注意：这是基于`成本`考虑的技术。
>
> 注意：如果在Azure中部署，请定期检查Azure的计费（计算、存储）情况，以免费用超出预期。

## 课后作业

- 在Azure中部署嵌套虚拟化，哪一块的费用是最不好预估，容易飘忽不定的，并说明理由。
