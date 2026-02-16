# 代码生成器

在Windows体系中，多数软件可以根据你的鼠标点击动作，`生成代码`，这对于学习PowerShell和自动化快速部署大有裨益。

在SDDC中，支持的软件包括

- Windows角色部署
- SCVMM
- SCOM
- SQL

## Windows角色部署

在`服务器管理器`中添加角色和功能时，安装向导的最后一步会显示对应的PowerShell命令：

1. 打开`服务器管理器`
2. 选择`添加角色和功能`
3. 按照向导选择所需的角色和功能
4. 在`确认`页面，点击`导出配置设置`或查看底部的PowerShell等效命令

例如，通过GUI安装Hyper-V角色时，向导会显示等效命令：

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
```

## SCVMM（Virtual Machine Manager）

VMM控制台内置了PowerShell脚本查看功能：

1. 打开`Virtual Machine Manager 控制台`
2. 在控制台底部，找到`PowerShell`选项卡
3. 执行任何GUI操作后，PowerShell选项卡会显示对应的PowerShell命令

例如，通过GUI创建虚拟机后，PowerShell选项卡会显示类似：

```powershell
New-SCVirtualMachine -Name "VM01" -VMHost $vmHost -VMTemplate $template ...
```

> 提示：将这些自动生成的命令保存下来，可以直接用于批量自动化部署。

## SCOM（Operations Manager）

SCOM控制台的部分操作也支持PowerShell等效命令查看：

1. 打开`Operations Manager控制台`
2. 在`管理`视图中执行操作
3. 某些操作会在结果窗口中显示等效的PowerShell命令

SCOM的PowerShell模块`OperationsManager`提供了丰富的Cmdlet：

```powershell
# 导入SCOM模块
Import-Module OperationsManager

# 常用命令
Get-SCOMAlert          # 查看告警
Get-SCOMAgent          # 查看Agent
Get-SCOMManagementPack # 查看管理包
```

## SQL Server

SQL Server Management Studio（SSMS）支持将GUI操作转换为T-SQL脚本：

1. 在SSMS中执行任何数据库操作时
2. 在操作对话框中，点击顶部的`脚本`按钮（或选择`编写脚本`）
3. SSMS会生成对应的T-SQL脚本

例如，通过GUI创建数据库时，可以生成：

```sql
CREATE DATABASE [DatabaseName]
ON PRIMARY
( NAME = N'DatabaseName', FILENAME = N'C:\...\DatabaseName.mdf', SIZE = 8192KB )
LOG ON
( NAME = N'DatabaseName_log', FILENAME = N'C:\...\DatabaseName_log.ldf', SIZE = 8192KB )
GO
```

## 使用建议

1. **学习阶段**：先通过GUI操作理解功能，同时观察生成的代码
2. **整理阶段**：将生成的代码整理保存，添加注释
3. **自动化阶段**：将整理后的代码组合成完整的自动化脚本
4. **分享阶段**：将脚本分享给团队，实现标准化部署

## 课后习题

- 在VMM控制台中创建一台虚拟机，观察PowerShell选项卡生成的命令，并尝试用该命令创建第二台。
- 在SSMS中创建一个新的登录名，使用`脚本`按钮查看对应的T-SQL。