# 常见问题排查

本章节汇总SDDC POC环境中常见的故障场景及排查方法，提供常用的诊断PowerShell命令。

## 排查方法论

在排查问题时，建议遵循以下步骤：

1. **确认现象**：明确故障的具体表现和影响范围
2. **检查日志**：查看相关组件的事件日志和SCOM告警
3. **网络验证**：确认网络连通性和DNS解析
4. **服务状态**：检查相关服务是否正常运行
5. **配置核查**：对比配置是否符合预期
6. **逐步恢复**：从最小变更开始，逐步恢复

## 常见问题及解决方案

### 1. VMM纳管主机失败

**现象**：在VMM中添加Hyper-V主机时，作业失败。

**排查步骤**：

```powershell
# 检查目标主机的WinRM配置
Test-WSMan -ComputerName "POC-COMP1.contoso.com"

# 检查防火墙状态（确认组策略已生效）
Invoke-Command -ComputerName "POC-COMP1" {
    Get-NetFirewallProfile | Format-Table Name, Enabled
}

# 检查目标主机是否已加域
Invoke-Command -ComputerName "POC-COMP1" {
    (Get-WmiObject Win32_ComputerSystem).Domain
}

# 检查运行方式帐户是否有本地管理员权限
Invoke-Command -ComputerName "POC-COMP1" {
    net localgroup administrators
}
```

**常见原因**：
- 防火墙未关闭（组策略未生效，使用`gpupdate /force`）
- 运行方式帐户未添加到目标主机本地管理员组
- DNS解析失败

### 2. SDN组件通信故障

**现象**：NC REST API无响应或返回错误。

**排查步骤**：

```powershell
# 检查NC服务状态
Invoke-Command -ComputerName "NC节点名称" {
    Get-Service -Name "NC*" | Format-Table Name, Status
}

# 检查NC节点健康状态
Invoke-Command -ComputerName "NC节点名称" {
    Get-NetworkControllerNode | Format-Table Name, Status
}

# 测试NC REST API连通性
Test-Connection -ComputerName nc.contoso.com -Count 2
Test-NetConnection -ComputerName nc.contoso.com -Port 443

# 检查DNS解析
Resolve-DnsName nc.contoso.com
```

**常见原因**：
- NC虚拟机未启动或服务停止
- 证书过期或配置错误
- DNS记录不存在或指向错误地址
- Management网络连通性问题

### 3. 证书过期处理

**现象**：SDN组件之间通信失败，日志中出现证书相关错误。

**排查步骤**：

```powershell
# 检查NC节点上的证书有效期
Invoke-Command -ComputerName "NC节点名称" {
    Get-ChildItem Cert:\LocalMachine\My | Where-Object {
        $_.Subject -match "nc.contoso.com"
    } | Format-Table Subject, NotBefore, NotAfter, Thumbprint
}
```

**处理方法**：
1. 在CA服务器上重新申请证书
2. 将新证书导出为PFX
3. 在NC节点上安装新证书
4. 更新NC配置使用新证书指纹
5. 重启NC服务

### 4. BGP邻居状态异常

**现象**：MUX通告的VIP路由在DCGW上不可见。

**排查步骤**：

```powershell
$cred = Get-Credential
Invoke-Command -VMName "poc-dcgw" -Credential $cred {
    # 检查BGP对等体状态
    Get-BgpPeer | Format-Table Name, ConnectivityStatus, PeerIPAddress

    # 检查BGP路由
    Get-BgpRouteInformation | Format-Table Network, NextHop, Origin

    # 检查BGP统计信息
    Get-BgpStatistics
}
```

**常见原因**：
- MUX的HNV Transit IP地址与DCGW配置不匹配
- BGP端口（TCP 179）被阻断
- MUX服务未正常启动

### 5. 虚拟机网络不通

**现象**：租户虚拟网络中的虚拟机无法通信。

**排查步骤**：

```powershell
# 在计算节点上检查Host Agent状态
Get-Service NCHostAgent
Get-Service SlbHostAgent

# 检查虚拟机的网络适配器配置
Get-VMNetworkAdapter -VMName "目标VM名称" | Format-Table VMName, IPAddresses, MacAddress, SwitchName

# 检查VXLAN封装是否正常
# 在计算节点上抓包分析
netsh trace start capture=yes tracefile=C:\temp\nettrace.etl

# 检查HNV Provider配置
Get-NetVirtualizationProviderAddress
```

**常见原因**：
- NCHostAgent服务停止
- 逻辑交换机未正确应用到计算节点
- VXLAN封装的MTU问题（建议Jumbo Frame）
- 虚拟子网配置错误

## 常用诊断命令速查

### 系统级诊断

```powershell
# 检查所有服务状态
Get-Service | Where-Object { $_.Status -ne "Running" -and $_.StartType -eq "Automatic" }

# 查看最近的系统错误事件
Get-EventLog -LogName System -EntryType Error -Newest 20

# 检查磁盘空间
Get-PSDrive -PSProvider FileSystem | Format-Table Name, Used, Free

# 网络连通性测试
Test-Connection -ComputerName "目标主机" -Count 4
Test-NetConnection -ComputerName "目标主机" -Port 端口号
```

### VMM诊断

```powershell
# 检查VMM服务
Get-Service SCVMMService, SCVMMAgent

# 查看VMM最近失败的作业
Get-SCJob | Where-Object { $_.Status -eq "Failed" } | Select-Object -Last 10

# 检查VMM连接的主机状态
Get-SCVMHost | Where-Object { $_.OverallState -ne "OK" }
```

### SDN诊断

```powershell
# 检查所有SDN相关服务
Get-Service NCHostAgent, SlbHostAgent

# NC集群诊断
Debug-NetworkController -NetworkController "nc.contoso.com"

# 查看SDN相关事件日志
Get-WinEvent -LogName "Microsoft-Windows-NetworkController*" -MaxEvents 20
```

## 课后习题

- 整理一份SDDC健康检查脚本，自动检查所有关键组件的状态并输出报告。
- 了解一下Windows Server中的`事件转发`（Event Forwarding）功能，如何将所有服务器的关键事件集中到一台管理主机上。

