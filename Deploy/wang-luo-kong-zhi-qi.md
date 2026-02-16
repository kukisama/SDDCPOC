# 网络控制器

完成`证书模板与申请`以及`VMM逻辑网络配置`后，可以开始部署网络控制器（Network Controller，简称NC）。网络控制器是整个SDN架构的`核心组件`，提供集中式的网络管理控制平面。

## 章节目标

- 理解网络控制器在SDN中的角色
- 通过VMM部署网络控制器
- 验证NC的REST API端点

## 背景知识

网络控制器是Windows Server SDN的控制平面，提供以下核心能力：

- **虚拟网络管理**：创建和管理基于VXLAN的虚拟网络（HNV）
- **负载均衡管理**：配置和管理SLB MUX的VIP和负载均衡规则
- **网关管理**：管理SDN网关的GRE/IPsec隧道
- **访问控制**：管理分布式防火墙规则（ACL）
- **REST API**：提供统一的RESTful接口，供VMM和其他管理工具调用

在POC环境中，部署单节点的网络控制器。生产环境建议部署3节点以实现高可用。

## 前置条件

在部署NC之前，请确认以下条件已满足：

| 条件 | 说明 | 对应章节 |
|------|------|----------|
| AD域控制器 | contoso.com域已就绪 | [Active Directory域控制器](../ADDS.md) |
| DNS A记录 | nc.contoso.com已解析到NC的管理IP | [DNS服务配置](dns-config.md) |
| 证书 | NC REST证书已申请并导出为PFX | [证书模板与申请](cert-template.md) |
| VMM逻辑网络 | Management和HNV Transit网络已创建 | [VMM逻辑网络配置](vmm-logical-network.md) |
| 计算节点 | 至少一台计算节点已被VMM纳管 | [第一台计算节点](../FirstComp.md) |

## 部署网络控制器

网络控制器通过VMM的`服务模板`进行部署。

### 步骤一：准备NC服务模板

1. 打开`Virtual Machine Manager 控制台`
2. 进入`库 → 导入模板`
3. 导入网络控制器的服务模板（来自SDN Express或VMM安装目录）

### 步骤二：配置网络服务

1. 进入`构造 → 网络 → 网络服务`
2. 右键选择`添加网络服务`
3. 选择`Microsoft`作为制造商
4. 选择`Microsoft Network Controller`作为型号
5. 配置连接字符串：

```
serverurl=https://nc.contoso.com;servicename=NC
```

### 步骤三：配置NC部署参数

在VMM的网络控制器配置中，需要提供以下关键参数：

| 参数 | 值 | 说明 |
|------|-----|------|
| REST名称 | nc.contoso.com | NC的REST API FQDN |
| 服务器证书 | 之前导出的PFX证书 | NC REST通信证书 |
| Management逻辑网络 | Management | 管理网络 |
| 管理员账号 | contoso\administrator | NC管理员 |
| 节点数量 | 1（POC环境） | 生产环境建议3节点 |

### 步骤四：部署NC虚拟机

1. VMM将自动在计算节点上创建NC虚拟机
2. 安装网络控制器角色
3. 配置NC集群（单节点时仅配置自身）
4. 安装REST证书
5. 启动NC服务

> 注意：NC部署过程可能需要15-30分钟，请耐心等待。部署过程中可以在VMM的`作业`视图中查看进度。

## 检查

部署完成后，进行以下验证：

### 验证NC服务状态

在NC虚拟机上执行以下PowerShell命令：

```powershell
# 检查网络控制器集群状态
Get-NetworkController

# 查看NC节点状态
Get-NetworkControllerNode

# 检查NC副本状态
Get-NetworkControllerReplica
```

### 验证REST API连通性

在任意已加域的管理主机上，测试NC REST API：

```powershell
# 测试REST API连通性（需要安装证书或使用-SkipCertificateCheck）
$NcUri = "https://nc.contoso.com"

# 查看网络控制器信息
Invoke-RestMethod -Uri "$NcUri/networking/v1/logicalnetworks" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

如果返回JSON格式的逻辑网络信息，说明NC已成功部署并正常工作。

### VMM中验证

在VMM控制台中，进入`构造 → 网络 → 网络服务`，确认网络控制器的状态为`已部署`。

## 课后习题

- 了解一下网络控制器的Northbound API和Southbound API分别指什么。
- 在生产环境中，为什么推荐部署3节点的网络控制器？这与哪种分布式一致性协议有关？

