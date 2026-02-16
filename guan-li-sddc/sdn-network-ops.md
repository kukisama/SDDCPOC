# SDN网络管理

本章节介绍SDDC环境中SDN网络的日常管理操作，包括虚拟网络管理、访问控制列表（ACL）配置、VIP管理以及SDN组件健康状态监控。

## 虚拟网络管理

### 查看虚拟网络

通过NC REST API查看所有虚拟网络：

```powershell
$NcUri = "https://nc.contoso.com"

# 获取所有虚拟网络
$vnets = Invoke-RestMethod -Uri "$NcUri/networking/v1/virtualnetworks" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$vnets.value | ForEach-Object {
    Write-Host "网络名称: $($_.resourceId)"
    Write-Host "  地址空间: $($_.properties.addressSpace.addressPrefixes -join ', ')"
    Write-Host "  子网数量: $($_.properties.subnets.Count)"
    Write-Host ""
}
```

### 创建虚拟网络

通过PowerShell创建新的虚拟网络：

```powershell
$NcUri = "https://nc.contoso.com"

# 定义虚拟网络
$VNetProperties = @{
    properties = @{
        addressSpace = @{
            addressPrefixes = @("10.10.0.0/16")
        }
        subnets = @(
            @{
                resourceId = "WebSubnet"
                properties = @{
                    addressPrefix = "10.10.1.0/24"
                }
            }
        )
    }
}

# 创建虚拟网络
$body = $VNetProperties | ConvertTo-Json -Depth 10
Invoke-RestMethod -Uri "$NcUri/networking/v1/virtualnetworks/NewVNet01" `
    -Method Put `
    -Body $body `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

### 删除虚拟网络

```powershell
# 删除虚拟网络（确保网络中没有活动的虚拟机）
Invoke-RestMethod -Uri "$NcUri/networking/v1/virtualnetworks/NewVNet01" `
    -Method Delete `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

> 注意：删除虚拟网络前，必须确保该网络中没有正在运行的虚拟机或网络接口。

## ACL规则配置

ACL（访问控制列表）用于控制虚拟网络中的流量，类似于分布式防火墙。

### 创建ACL规则

```powershell
$NcUri = "https://nc.contoso.com"

# 创建一个允许HTTP流量的ACL
$AclProperties = @{
    properties = @{
        aclRules = @(
            @{
                resourceId = "AllowHTTP"
                properties = @{
                    protocol = "TCP"
                    sourcePortRange = "0-65535"
                    destinationPortRange = "80"
                    action = "Allow"
                    sourceAddressPrefix = "*"
                    destinationAddressPrefix = "*"
                    direction = "Inbound"
                    priority = 100
                }
            },
            @{
                resourceId = "AllowHTTPS"
                properties = @{
                    protocol = "TCP"
                    sourcePortRange = "0-65535"
                    destinationPortRange = "443"
                    action = "Allow"
                    sourceAddressPrefix = "*"
                    destinationAddressPrefix = "*"
                    direction = "Inbound"
                    priority = 110
                }
            }
        )
    }
}

$body = $AclProperties | ConvertTo-Json -Depth 10
Invoke-RestMethod -Uri "$NcUri/networking/v1/accessControlLists/WebACL" `
    -Method Put `
    -Body $body `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

### 查看ACL规则

```powershell
$acls = Invoke-RestMethod -Uri "$NcUri/networking/v1/accessControlLists" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$acls.value | ForEach-Object {
    Write-Host "ACL: $($_.resourceId)"
    $_.properties.aclRules | ForEach-Object {
        Write-Host "  规则: $($_.resourceId) - $($_.properties.action) $($_.properties.protocol):$($_.properties.destinationPortRange)"
    }
}
```

## VIP管理

### 查看负载均衡器配置

```powershell
$NcUri = "https://nc.contoso.com"

# 查看所有负载均衡器
$lbs = Invoke-RestMethod -Uri "$NcUri/networking/v1/loadBalancers" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$lbs.value | ForEach-Object {
    Write-Host "负载均衡器: $($_.resourceId)"
    $_.properties.frontendIPConfigurations | ForEach-Object {
        Write-Host "  VIP: $($_.properties.privateIPAddress)"
    }
}
```

### 查看VIP地址池使用情况

在VMM控制台中：

1. 进入`构造 → 网络 → 逻辑网络`
2. 选择`PublicVIP`或`PrivateVIP`
3. 查看IP池的已分配和可用地址数量

## SDN健康状态监控

### 检查NC状态

```powershell
# 在NC节点上执行
Get-NetworkControllerNode | Format-Table Name, Status, FaultDomain

# 检查NC副本状态
Get-NetworkControllerReplica | Format-Table ReplicaRole, Status
```

### 检查所有SDN组件

```powershell
$NcUri = "https://nc.contoso.com"

# 检查服务器（计算节点）状态
$servers = Invoke-RestMethod -Uri "$NcUri/networking/v1/servers" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

$servers.value | ForEach-Object {
    Write-Host "主机: $($_.resourceId) - 配置状态: $($_.properties.configurationState.status)"
}
```

## 课后习题

- 通过NC REST API创建一个包含两个子网的虚拟网络，并为其中一个子网配置ACL规则只允许SSH和RDP流量。
- 了解一下SDN中`网络安全组`（Network Security Group）与传统防火墙规则的区别。

