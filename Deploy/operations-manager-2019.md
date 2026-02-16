# Operations Manager 2019

Operations Manager（SCOM）是System Center系列中的监控平台，用于监控Windows Server、Hyper-V、VMM和SDN组件的健康状态和性能指标。

## 章节目标

- 创建SCOM虚拟机并完成基础配置
- 安装Operations Manager 2019
- 导入Management Pack（管理包）
- 配置基础监控

## 虚拟机配置

创建一台虚拟机，进行如下配置：

| 虚拟机主机名称 | 功能 | IP | 掩码 | DNS | 网关 | CPU | 内存 | 硬盘 |
|----------------|------|-----|------|------|------|-----|------|------|
| POC-OM01 | Operations Manager | 192.148.0.6 | 255.255.255.0 | 192.148.0.2 | 192.148.0.1 | 2 | 4G | 默认 |

## 前置条件

| 条件 | 说明 |
|------|------|
| AD域控制器 | POC-DC01运行正常 |
| SQL Server | POC-SQL01运行正常 |
| SCOM管理员账号 | 已在AD中创建（参考账号体系约定） |
| SCOM安装介质 | System Center 2019 Operations Manager ISO |

## 安装Operations Manager

### 步骤一：准备数据库

使用`域管理员`登录`POC-SQL01`，在SQL Server中为SCOM创建所需的权限：

```powershell
# 确认SQL Server服务正在运行
Get-Service MSSQLSERVER
```

> SCOM安装过程中会自动创建所需的数据库（OperationsManager和OperationsManagerDW），无需手动创建。

### 步骤二：安装前置组件

使用`域管理员`登录`POC-OM01`，安装必要的前置组件：

```powershell
# 安装必要的Windows功能
Install-WindowsFeature NET-Framework-Core, NET-WCF-HTTP-Activation45, Web-Static-Content, `
    Web-Default-Doc, Web-Dir-Browsing, Web-Http-Errors, Web-Http-Logging, `
    Web-Request-Monitor, Web-Filtering, Web-Stat-Compression, Web-Mgmt-Console, `
    Web-Metabase, Web-Asp-Net, Web-Windows-Auth -IncludeManagementTools
```

### 步骤三：安装Operations Manager

1. 挂载System Center 2019 Operations Manager ISO
2. 运行`Setup.exe`
3. 选择`安装`
4. 选择要安装的功能：
   - ✅ 管理服务器
   - ✅ 操作控制台
   - ✅ Web控制台
5. 配置数据库服务器：`POC-SQL01.contoso.com`
6. 配置管理组名称：`SDDC-POC`
7. 配置SCOM服务使用的账号（使用域账号或本地系统账户）
8. 完成安装向导

> 注意：SCOM安装过程可能需要20-30分钟，取决于硬件性能。

## 导入管理包

安装完成后，需要导入管理包（Management Pack）以监控SDDC中的各组件：

1. 打开`Operations Manager控制台`
2. 进入`管理 → 管理包`
3. 右键选择`导入管理包`
4. 从Microsoft Update Catalog下载并导入以下管理包：

| 管理包 | 用途 |
|--------|------|
| Windows Server Operating System | 监控Windows Server操作系统 |
| Hyper-V Management Pack | 监控Hyper-V主机和虚拟机 |
| System Center VMM Management Pack | 监控VMM服务 |
| Network Controller Management Pack | 监控SDN网络控制器 |
| SQL Server Management Pack | 监控SQL Server |

## 配置Agent发现

1. 在`管理 → 设备管理 → 发现向导`中
2. 选择`Windows计算机`
3. 选择`自动计算机发现`或手动指定以下目标：
   - POC-DC01
   - POC-SQL01
   - POC-VMM01
   - POC-COMP1
   - POC-COMP2
4. 使用域管理员凭据完成Agent部署

## 检查

安装和配置完成后，进行以下验证：

1. 打开`Operations Manager控制台`
2. 进入`监视 → 活动警报`，确认没有严重错误
3. 进入`管理 → 设备管理 → Agent管理的`，确认所有目标服务器的Agent状态为`正常`
4. 查看`监视 → Windows计算机`，确认能看到各服务器的性能数据

## 课后习题

- 了解一下SCOM中`管理包`（Management Pack）的作用，以及如何自定义监控规则。
- SCOM支持自定义仪表板，尝试创建一个展示所有Hyper-V主机CPU利用率的视图。

