# 创建租户虚拟网络

SDN部署验证完成后，可以创建租户虚拟网络来验证SDN的核心功能——网络虚拟化。本章节演示如何通过VMM创建基于HNV（Hyper-V Network Virtualization）的虚拟网络，部署测试虚拟机，并验证网络隔离。

## 章节目标

- 通过VMM创建虚拟网络（VM Network）
- 创建虚拟子网
- 部署测试虚拟机并验证网络连通性
- 验证不同虚拟网络之间的隔离

## 背景知识

在Microsoft SDN中，租户虚拟网络具有以下特点：

- **网络隔离**：不同租户的虚拟网络相互隔离，即使使用相同的IP地址空间也不冲突
- **VXLAN封装**：虚拟网络流量通过VXLAN封装在HNV/Transit物理网络上传输
- **集中管理**：通过网络控制器的REST API或VMM控制台统一管理

## 步骤一：创建虚拟网络

1. 打开`Virtual Machine Manager 控制台`
2. 进入`VM和服务 → VM网络`
3. 右键选择`创建VM网络`
4. 配置如下：

| 配置项 | 值 |
|--------|-----|
| 名称 | Tenant01-Network |
| 隔离方式 | 使用Hyper-V网络虚拟化进行隔离 |
| 关联逻辑网络 | HNV Transit |

## 步骤二：创建虚拟子网

在创建的VM网络中添加虚拟子网：

| 配置项 | 值 |
|--------|-----|
| 子网名称 | Tenant01-WebTier |
| 地址前缀 | 10.0.1.0/24 |
| 网关 | 10.0.1.1 |

可以根据需要创建多个子网来模拟多层应用架构：

| 子网名称 | 地址前缀 | 用途 |
|----------|----------|------|
| Tenant01-WebTier | 10.0.1.0/24 | Web前端层 |
| Tenant01-AppTier | 10.0.2.0/24 | 应用中间层 |
| Tenant01-DBTier | 10.0.3.0/24 | 数据库层 |

> 注意：虚拟子网使用的IP地址空间（如10.0.x.0/24）是虚拟地址，与物理网络（192.148.x.0/24）完全独立。

## 步骤三：部署测试虚拟机

在VMM中创建测试虚拟机，并连接到虚拟网络：

1. 使用之前创建的VHDX模板，在VMM中创建2台测试虚拟机
2. 将虚拟机的网络适配器连接到`Tenant01-WebTier`虚拟子网
3. 虚拟机将自动获取10.0.1.0/24网段的IP地址

```powershell
# 在测试VM中检查IP配置
Get-NetIPAddress -InterfaceAlias "以太网*" -AddressFamily IPv4

# 测试同一子网内两台VM的连通性
Test-Connection -ComputerName 10.0.1.x -Count 4
```

## 步骤四：验证网络隔离

为了验证网络隔离功能，创建第二个租户虚拟网络：

1. 创建名为`Tenant02-Network`的VM网络
2. 使用相同的地址空间`10.0.1.0/24`创建虚拟子网
3. 部署测试虚拟机连接到Tenant02的网络

验证隔离：

```powershell
# 在Tenant01的VM中尝试ping Tenant02的VM
# 即使IP地址相同（如都是10.0.1.10），也应该无法通信
Test-Connection -ComputerName 10.0.1.10 -Count 4
# 预期结果：请求超时，说明隔离生效
```

## 步骤五：验证跨子网通信

如果在同一个虚拟网络中创建了多个子网，需要验证跨子网的通信：

```powershell
# 在WebTier子网（10.0.1.x）的VM中，ping AppTier子网（10.0.2.x）的VM
Test-Connection -ComputerName 10.0.2.10 -Count 4
# 预期结果：通信成功（同一虚拟网络内的子网之间自动路由）
```

## 检查

| 验证项 | 预期结果 |
|--------|----------|
| 同一虚拟子网内VM通信 | 通信正常 |
| 同一虚拟网络跨子网通信 | 通信正常 |
| 不同虚拟网络间通信 | 通信隔离（无法ping通） |
| 虚拟网络VM到物理网络通信 | 需通过SDN网关才能通信 |

## 课后习题

- 使用VMM或NC REST API，查看创建的虚拟网络对应的VXLAN VNI（Virtual Network Identifier）是多少。
- 在计算节点上使用`Get-VMNetworkAdapter`命令，查看测试虚拟机的MAC地址是PA（Provider Address）还是CA（Customer Address）。

