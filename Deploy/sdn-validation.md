# SDN功能验证

完成所有SDN组件（网络控制器、SLB MUX、SDN网关）的部署后，需要对SDN功能进行系统性验证，确保各组件协同工作正常。

## 章节目标

- 验证网络控制器REST API的连通性
- 验证SDN各组件的健康状态
- 执行基本的SDN功能测试

## 验证清单

在开始验证之前，确认以下组件已全部部署完成：

| 组件 | 虚拟机/服务 | 状态要求 |
|------|------------|----------|
| 网络控制器 | NC节点 | REST API可访问 |
| SLB MUX | MUX节点 | BGP对等已建立 |
| SDN网关 | Gateway节点 | 已注册到NC |
| DCGW | POC-DCGW | BGP路由正常 |
| 计算节点 | POC-COMP1/COMP2 | VMM纳管正常 |

## 步骤一：验证网络控制器

### 检查NC节点状态

在NC虚拟机上执行：

```powershell
# 查看NC节点信息
Get-NetworkController

# 查看NC集群状态
Get-NetworkControllerNode

# 检查所有NC服务的运行状态
Get-NetworkControllerReplica
```

所有服务应显示为`Ready`或`Up`状态。

### 验证REST API

在管理主机上执行：

```powershell
$NcUri = "https://nc.contoso.com"

# 测试基本连通性
Invoke-RestMethod -Uri "$NcUri/networking/v1/logicalnetworks" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

# 查看所有虚拟网络
Invoke-RestMethod -Uri "$NcUri/networking/v1/virtualnetworks" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

## 步骤二：验证SLB MUX

### 检查MUX注册状态

```powershell
$NcUri = "https://nc.contoso.com"

# 查看MUX列表和状态
$muxes = Invoke-RestMethod -Uri "$NcUri/networking/v1/loadbalancerMuxes" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$muxes.value | ForEach-Object {
    Write-Host "MUX: $($_.resourceId) - 配置状态: $($_.properties.configurationState.status)"
}
```

### 验证BGP对等

在DCGW上检查：

```powershell
$cred = Get-Credential
Invoke-Command -VMName "poc-dcgw" -Credential $cred {
    # 查看BGP对等体状态
    Get-BgpPeer | Format-Table Name, LocalIPAddress, PeerIPAddress, ConnectivityStatus

    # 查看BGP学习到的路由
    Get-BgpRouteInformation | Format-Table Network, NextHop
}
```

BGP对等体的`ConnectivityStatus`应为`Connected`。

## 步骤三：验证SDN网关

```powershell
$NcUri = "https://nc.contoso.com"

# 查看网关状态
$gateways = Invoke-RestMethod -Uri "$NcUri/networking/v1/gateways" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$gateways.value | ForEach-Object {
    Write-Host "网关: $($_.resourceId) - 状态: $($_.properties.state)"
}

# 查看网关池
Invoke-RestMethod -Uri "$NcUri/networking/v1/gatewayPools" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

## 步骤四：验证Hyper-V主机的SDN状态

在VMM控制台中检查计算节点的网络配置：

1. 打开`Virtual Machine Manager 控制台`
2. 进入`构造 → 服务器 → 所有主机`
3. 选择计算节点，查看`属性 → 虚拟交换机`
4. 确认逻辑交换机已正确应用

通过PowerShell检查主机上的SDN Agent状态：

```powershell
# 在计算节点上执行
Get-Service NCHostAgent
Get-Service SlbHostAgent

# 检查Host Agent的网络控制器连接状态
netsh trace show helper
```

## 步骤五：VMM中的综合验证

在VMM控制台中进行综合检查：

1. `构造 → 网络 → 网络服务`：确认NC、MUX、Gateway状态正常
2. `构造 → 网络 → 逻辑网络`：确认所有逻辑网络配置正确
3. `VM和服务`：确认可以创建使用虚拟网络的VM

## 常见问题

| 问题 | 可能原因 | 排查方法 |
|------|----------|----------|
| REST API无响应 | NC服务未启动或证书问题 | 检查NC服务状态和证书配置 |
| BGP对等未建立 | IP地址配置错误或防火墙阻断 | 确认IP地址和端口179的通信 |
| MUX显示异常 | NC与MUX通信失败 | 检查Management网络连通性 |

## 课后习题

- 编写一个PowerShell脚本，自动检查所有SDN组件的状态并输出报告。
- 了解一下SDN Debug命令集，如`Debug-NetworkController`的使用方法。

