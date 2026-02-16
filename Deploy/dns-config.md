# DNS服务配置

在部署Active Directory域控制器时，DNS服务会自动安装。但在SDN部署过程中，需要创建额外的DNS记录，特别是网络控制器REST API所需的DNS A记录。

## 章节目标

- 配置DNS转发器
- 创建网络控制器REST API所需的DNS A记录
- 配置DNS反向查找区域

## 前置条件

| 条件 | 说明 |
|------|------|
| 域控制器已部署 | POC-DC01运行正常，DNS服务已安装 |
| 网络规划已确定 | 参考[IP地址和虚拟机名称约定](../IPandVMname.md) |

## 配置DNS转发器

DNS转发器用于将无法在本地解析的DNS请求转发到外部DNS服务器。在POC环境中，如果需要虚拟机能够解析外部域名（如下载补丁），需要配置DNS转发器。

使用`域管理员`登录`POC-DC01`，打开`管理员的PowerShell`：

```powershell
# 添加DNS转发器（使用公共DNS服务器）
Add-DnsServerForwarder -IPAddress "8.8.8.8" -PassThru
Add-DnsServerForwarder -IPAddress "8.8.4.4" -PassThru

# 验证转发器配置
Get-DnsServerForwarder
```

> 注意：如果POC环境没有外部网络访问能力（如使用内部虚拟交换机），DNS转发器配置不是必须的。

## 创建网络控制器DNS记录

网络控制器的REST API需要通过FQDN访问。在部署NC之前，必须在DNS中创建对应的A记录。

```powershell
# 创建NC REST API的DNS A记录
# NC的REST名称为 nc.contoso.com
# IP地址为NC虚拟机的管理IP（由VMM部署时分配，此处使用预规划地址）
Add-DnsServerResourceRecordA -Name "nc" -ZoneName "contoso.com" -IPv4Address "192.148.11.1" -CreatePtr

# 验证DNS记录
Resolve-DnsName nc.contoso.com
```

> 注意：NC的实际IP地址需要根据VMM部署时分配的地址进行调整。如果使用VMM自动部署NC，VMM可能会自动创建所需的DNS记录。

## 配置反向查找区域

反向查找区域允许通过IP地址反向解析主机名，对于故障排查非常有帮助。

```powershell
# 创建物理网络的反向查找区域
Add-DnsServerPrimaryZone -NetworkID "192.148.0.0/24" -ReplicationScope Domain

# 创建Management网络的反向查找区域
Add-DnsServerPrimaryZone -NetworkID "192.148.11.0/24" -ReplicationScope Domain

# 创建HNV/Transit网络的反向查找区域
Add-DnsServerPrimaryZone -NetworkID "192.148.12.0/24" -ReplicationScope Domain

# 验证反向查找区域
Get-DnsServerZone | Where-Object { $_.IsReverseLookupZone -eq $true }
```

## 检查

完成DNS配置后，进行以下验证：

```powershell
# 检查DNS转发器
Get-DnsServerForwarder

# 检查NC的DNS记录
Resolve-DnsName nc.contoso.com

# 检查反向查找
Resolve-DnsName 192.148.0.2

# 检查DNS服务状态
Get-Service DNS
```

## 课后习题

- 了解一下DNS A记录和CNAME记录的区别，思考NC的REST名称为什么需要使用A记录。
- 了解一下DNS反向查找区域的作用，在什么场景下会用到PTR记录？

