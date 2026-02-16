# PowerShell速查手册

本手册汇总全书涉及的关键PowerShell命令，按功能分类，方便快速查阅。

## Hyper-V管理

```powershell
# 启用Hyper-V角色
Install-WindowsFeature Hyper-V -IncludeManagementTools -Restart

# 创建内部虚拟交换机
New-VMSwitch -SwitchType Internal -Name "POC"

# 创建虚拟机
New-VM -Name "VM名称" -MemoryStartupBytes 1GB -SwitchName "POC" -Generation 2

# 配置嵌套虚拟化
Set-VMProcessor -VMName "VM名称" -ExposeVirtualizationExtensions $true

# 启用MAC地址欺骗
Get-VMNetworkAdapter -VMName "VM名称" | Set-VMNetworkAdapter -MacAddressSpoofing On

# 创建检查点
Get-VM -Name "VM名称" | Checkpoint-VM -SnapshotName "检查点名称"

# 管理虚拟机
Start-VM -Name "VM名称"
Stop-VM -Name "VM名称"
Restart-VM -Name "VM名称"
Get-VM | Format-Table Name, State, CPUUsage, MemoryAssigned

# PowerShell Direct（从主机直接与VM通信）
$cred = Get-Credential
Invoke-Command -VMName "VM名称" -Credential $cred { hostname }
```

## Active Directory管理

```powershell
# 安装AD域控制器
Install-WindowsFeature AD-Domain-Services -IncludeAllSubFeature -IncludeManagementTools
Install-ADDSForest -DomainName "contoso.com" -DomainNetbiosName "contoso"

# 创建组织单位（OU）
New-ADOrganizationalUnit -Name "OU名称" -Path "DC=contoso,DC=com"

# 创建域用户
New-ADUser -Name "用户名" -SamAccountName "用户名" -UserPrincipalName "用户名@contoso.com" `
    -AccountPassword (ConvertTo-SecureString "poc.123" -AsPlainText -Force) -Enabled $true

# 创建安全组
New-ADGroup -Name "组名" -GroupScope Global -GroupCategory Security

# 添加用户到组
Add-ADGroupMember -Identity "组名" -Members "用户名"

# 计算机加域
Add-Computer -DomainName "contoso.com" -Credential (Get-Credential) -Restart
```

## 网络配置

```powershell
# 配置IP地址
New-NetIPAddress -InterfaceAlias "以太网" -IPAddress "192.148.0.x" `
    -PrefixLength 24 -DefaultGateway "192.148.0.1"

# 配置DNS
Set-DnsClientServerAddress -InterfaceAlias "以太网" -ServerAddresses "192.148.0.2"

# 网卡改名
Get-NetAdapter -Name "以太网*" | Rename-NetAdapter -NewName "新名称"

# 关闭防火墙
Set-NetFirewallProfile -Enabled False

# 测试连通性
Test-Connection -ComputerName "目标地址" -Count 4
Test-NetConnection -ComputerName "目标地址" -Port 端口号

# DNS管理
Add-DnsServerResourceRecordA -Name "主机名" -ZoneName "contoso.com" -IPv4Address "IP地址"
Add-DnsServerForwarder -IPAddress "8.8.8.8"
Resolve-DnsName "域名"
```

## VMM管理

```powershell
# 连接VMM
$VMMServer = Get-SCVMMServer -ComputerName "POC-VMM01.contoso.com"

# 查看主机
Get-SCVMHost | Format-Table ComputerName, OverallState

# 查看虚拟机
Get-SCVirtualMachine | Format-Table Name, Status, VMHost

# 创建虚拟机
New-SCVirtualMachine -Name "VM名称" -VMTemplate $Template -VMHost $VMHost -Path "路径"

# 实时迁移
Move-SCVirtualMachine -VM $VM -VMHost $TargetHost

# 虚拟机快照
New-SCVMCheckpoint -VM $VM -Name "快照名称"
Restore-SCVMCheckpoint -VMCheckpoint $Checkpoint

# 查看作业
Get-SCJob | Where-Object { $_.Status -eq "Failed" }

# 主机维护模式
Disable-SCVMHost -VMHost $VMHost -MoveWithinCluster
Enable-SCVMHost -VMHost $VMHost
```

## 证书管理

```powershell
# 安装CA角色
Install-WindowsFeature Adcs-Cert-Authority -IncludeManagementTools
Install-AdcsCertificationAuthority -CAType EnterpriseRootCa

# 查看证书
Get-ChildItem Cert:\LocalMachine\My

# 导出证书
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -match "关键字" }
Export-PfxCertificate -Cert $cert -FilePath "证书路径.pfx" -Password $password

# 申请证书
certreq -new "请求文件.inf" "请求文件.req"
certreq -submit -config "CA服务器\CA名称" "请求文件.req" "证书文件.cer"
certreq -accept "证书文件.cer"
```

## SDN诊断

```powershell
# 网络控制器
Get-NetworkController
Get-NetworkControllerNode
Get-NetworkControllerReplica

# NC REST API调用
$NcUri = "https://nc.contoso.com"
Invoke-RestMethod -Uri "$NcUri/networking/v1/logicalnetworks" `
    -Method Get -UseDefaultCredentials -ContentType "application/json"

# SDN Host Agent
Get-Service NCHostAgent, SlbHostAgent

# BGP管理（在DCGW上）
Get-BgpPeer | Format-Table Name, ConnectivityStatus
Get-BgpRouteInformation
Add-BgpPeer -Name "名称" -LocalIPAddress "本地IP" -PeerIPAddress "对端IP" `
    -LocalASN 65002 -PeerASN 65001

# HNV诊断
Get-NetVirtualizationProviderAddress

# 集中诊断
Debug-NetworkController -NetworkController "nc.contoso.com"
```

## RRAS路由

```powershell
# 安装RRAS
Install-WindowsFeature RemoteAccess -IncludeAllSubFeature -IncludeManagementTools
Install-RemoteAccess -VpnType RoutingOnly

# 启动路由服务
Get-Service RemoteAccess | Set-Service -StartupType Automatic
Get-Service RemoteAccess | Start-Service

# BGP路由器
Add-BgpRouter -BgpIdentifier "路由器ID" -LocalASN ASN号
```

## 系统诊断

```powershell
# 服务管理
Get-Service -Name "服务名称"
Start-Service -Name "服务名称"
Restart-Service -Name "服务名称"

# 检查未运行的自动启动服务
Get-Service | Where-Object { $_.Status -ne "Running" -and $_.StartType -eq "Automatic" }

# 事件日志
Get-EventLog -LogName System -EntryType Error -Newest 20
Get-WinEvent -LogName "日志名称" -MaxEvents 20

# 磁盘空间
Get-PSDrive -PSProvider FileSystem | Format-Table Name, Used, Free

# 将域账号添加到本地管理员
Add-LocalGroupMember -Group "Administrators" -Member "contoso\用户名"
```

## 组策略

```powershell
# 打开组策略管理
gpmc.msc

# 刷新组策略
gpupdate /force

# 查看应用的组策略
gpresult /r
```

