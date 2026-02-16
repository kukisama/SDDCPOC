# Operations Manager UR补丁

与VMM类似，Operations Manager也需要安装更新汇总（UR）补丁来修复已知问题并获取新功能。

## 章节目标

- 获取SCOM UR补丁
- 安装UR补丁到管理服务器
- 更新SCOM Agent

## 获取UR补丁

1. 访问Microsoft Update Catalog网站
2. 搜索"System Center 2019 Operations Manager"
3. 下载对应版本的UR补丁包

> 建议安装最新的UR补丁。截至文档编写时，建议至少安装UR3或更高版本。

## 安装前准备

在安装UR补丁之前，建议进行以下准备：

1. **备份数据库**：备份OperationsManager和OperationsManagerDW数据库
2. **创建检查点**：为POC-OM01虚拟机创建检查点
3. **通知相关人员**：UR安装期间SCOM服务将短暂中断

```powershell
# 在POC-SQL01上备份SCOM数据库
Invoke-Sqlcmd -Query "BACKUP DATABASE [OperationsManager] TO DISK = 'C:\Backup\OperationsManager_PreUR.bak'"
Invoke-Sqlcmd -Query "BACKUP DATABASE [OperationsManagerDW] TO DISK = 'C:\Backup\OperationsManagerDW_PreUR.bak'"
```

## 安装UR补丁

### 步骤一：安装管理服务器UR

使用`域管理员`登录`POC-OM01`：

1. 关闭Operations Manager控制台
2. 运行UR补丁安装包（MSP文件）
3. 按照向导完成安装
4. 安装完成后重启SCOM服务

```powershell
# 安装完成后，重启SCOM相关服务
Restart-Service HealthService
Restart-Service OMSDK
Restart-Service cshost
```

### 步骤二：更新Web控制台

如果安装了Web控制台，需要单独更新：

1. 运行Web控制台对应的UR补丁
2. 完成安装后，在IIS中重启SCOM应用池

### 步骤三：更新Agent

管理服务器UR安装完成后，需要更新所有受管理服务器上的SCOM Agent：

1. 打开`Operations Manager控制台`
2. 进入`管理 → 设备管理 → Agent管理的`
3. 选择所有需要更新的Agent
4. 右键选择`更新Agent`
5. 等待Agent更新完成

> 注意：Agent更新过程中，受影响的服务器监控数据会暂时中断。

## 检查

安装UR补丁后，进行以下验证：

```powershell
# 检查SCOM管理服务器版本
Get-SCOMManagementServer | Format-Table DisplayName, Version

# 检查SCOM服务状态
Get-Service HealthService, OMSDK, cshost | Format-Table Name, Status

# 检查Agent版本（应与管理服务器版本一致）
Get-SCOMAgent | Format-Table ComputerName, Version
```

确认所有组件版本已更新，且SCOM控制台中没有因UR安装导致的新告警。

## 课后习题

- 了解一下SCOM UR补丁的更新频率和生命周期支持策略。
- 在生产环境中，UR补丁的安装顺序应该是什么？（提示：考虑管理服务器、网关、Agent的顺序）

