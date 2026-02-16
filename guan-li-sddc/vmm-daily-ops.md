# VMM日常管理

本章节介绍使用Virtual Machine Manager进行日常管理的常用操作，包括虚拟机生命周期管理、主机维护和资源管理。

## 虚拟机生命周期管理

### 创建虚拟机

在VMM控制台中创建虚拟机有多种方式：

| 创建方式 | 适用场景 | 操作路径 |
|----------|----------|----------|
| 从模板创建 | 标准化部署 | VM和服务 → 创建虚拟机 → 使用现有虚拟机、VM模板或虚拟硬盘 |
| 从空白创建 | 自定义配置 | VM和服务 → 创建虚拟机 → 创建新的虚拟机 |
| 从VHD创建 | 快速部署 | VM和服务 → 创建虚拟机 → 使用现有虚拟硬盘 |

通过PowerShell创建虚拟机：

```powershell
# 获取VMM连接
$VMMServer = Get-SCVMMServer -ComputerName "POC-VMM01.contoso.com"

# 获取目标主机
$VMHost = Get-SCVMHost -ComputerName "POC-COMP1.contoso.com"

# 获取虚拟机模板
$Template = Get-SCVMTemplate -Name "Windows Server 2019 Template"

# 创建虚拟机
New-SCVirtualMachine -Name "TestVM01" `
    -VMTemplate $Template `
    -VMHost $VMHost `
    -Path "C:\VMs" `
    -RunAsynchronously
```

### 虚拟机迁移

VMM支持实时迁移（Live Migration），在不停机的情况下将虚拟机迁移到其他主机：

```powershell
# 获取虚拟机
$VM = Get-SCVirtualMachine -Name "TestVM01"

# 获取目标主机
$TargetHost = Get-SCVMHost -ComputerName "POC-COMP2.contoso.com"

# 执行实时迁移
Move-SCVirtualMachine -VM $VM -VMHost $TargetHost -RunAsynchronously
```

> 注意：实时迁移要求源和目标主机之间网络连通，且都属于同一个VMM管理范围。

### 虚拟机快照管理

```powershell
# 创建快照
New-SCVMCheckpoint -VM $VM -Name "安装前快照"

# 查看快照
Get-SCVMCheckpoint -VM $VM

# 还原到快照
Restore-SCVMCheckpoint -VMCheckpoint $Checkpoint

# 删除快照
Remove-SCVMCheckpoint -VMCheckpoint $Checkpoint
```

## 主机维护模式

当需要对Hyper-V主机进行维护（如安装补丁、硬件维护）时，应先将主机置于维护模式：

1. 在VMM控制台中，右键点击目标主机
2. 选择`启动维护模式`
3. 选择迁移方式：
   - **将所有虚拟机迁移到其他主机**：实时迁移所有VM
   - **将所有虚拟机置于保存状态**：保存VM内存状态

```powershell
# 通过PowerShell进入维护模式
$VMHost = Get-SCVMHost -ComputerName "POC-COMP1.contoso.com"
Disable-SCVMHost -VMHost $VMHost -MoveWithinCluster
```

维护完成后，退出维护模式：

```powershell
Enable-SCVMHost -VMHost $VMHost
```

## 资源池管理

### 主机组

主机组用于对计算节点进行逻辑分组，便于资源管理和权限控制：

```powershell
# 创建主机组
$ParentGroup = Get-SCVMHostGroup -Name "所有主机"
New-SCVMHostGroup -Name "SDN计算节点" -ParentHostGroup $ParentGroup

# 将主机移动到主机组
$VMHost = Get-SCVMHost -ComputerName "POC-COMP1.contoso.com"
$HostGroup = Get-SCVMHostGroup -Name "SDN计算节点"
Move-SCVMHost -VMHost $VMHost -ParentHostGroup $HostGroup
```

### 云资源

在VMM中可以创建`云`来组织和分配资源：

```powershell
# 创建云
$HostGroup = Get-SCVMHostGroup -Name "SDN计算节点"
New-SCCloud -Name "POC Cloud" -VMHostGroup $HostGroup
```

## 检查

定期检查以下内容，确保VMM管理平台运行正常：

```powershell
# 检查VMM服务状态
Get-Service SCVMMService

# 检查所有主机状态
Get-SCVMHost | Format-Table ComputerName, OverallState, CommunicationState

# 检查作业中是否有失败的任务
Get-SCJob | Where-Object { $_.Status -eq "Failed" } | Select-Object Name, Status, StartTime
```

## 课后习题

- 尝试使用VMM的PowerShell模块批量创建5台虚拟机。
- 了解一下VMM中`用户角色`的概念，如何实现多租户的资源隔离和权限控制。

