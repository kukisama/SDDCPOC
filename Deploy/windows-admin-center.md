# Windows Admin Center

Windows Admin Center（WAC）是微软推荐的新一代服务器管理工具。WAC通过Web浏览器提供轻量、现代化的管理界面，可以管理Windows Server、Hyper-V集群以及SDN组件。

## 章节目标

- 安装Windows Admin Center
- 连接和管理Hyper-V主机
- 了解SDN管理扩展

## 背景知识

WAC相比传统管理工具的优势：

| 特性 | 说明 |
|------|------|
| Web界面 | 无需安装客户端，通过浏览器即可管理 |
| 轻量级 | 安装简单，资源占用少 |
| 扩展性 | 支持通过扩展增加功能 |
| 远程管理 | 支持远程PowerShell、远程桌面等 |
| SDN集成 | 提供SDN监控和管理扩展 |

## 安装Windows Admin Center

### 准备

WAC可以安装在以下位置：

| 安装模式 | 说明 | 适用场景 |
|----------|------|----------|
| 网关模式 | 安装在独立服务器上，其他用户通过浏览器访问 | 推荐用于POC和生产环境 |
| 桌面模式 | 安装在Windows 10管理工作站上 | 个人管理使用 |

在POC环境中，建议以`网关模式`安装在管理主机或已有服务器上。

### 安装步骤

1. 从微软官网下载Windows Admin Center MSI安装包
2. 在目标服务器上运行安装程序
3. 配置安装参数：

| 配置项 | 建议值 |
|--------|--------|
| 端口 | 443（默认）或自定义端口 |
| 使用WinRM over HTTPS | 是 |
| 生成自签名SSL证书 | 是（POC环境） |
| 允许WAC更新 | 是 |

也可以通过命令行静默安装：

```powershell
# 静默安装WAC（网关模式，使用443端口）
msiexec /i WindowsAdminCenter.msi /qn /L*v log.txt `
    SME_PORT=443 `
    SSL_CERTIFICATE_OPTION=generate
```

安装完成后，WAC服务会自动启动。

## 连接管理目标

### 添加服务器连接

1. 打开浏览器，访问`https://WAC服务器地址:443`
2. 使用域管理员账号登录
3. 点击`添加`，选择`服务器`
4. 输入服务器名称或IP地址：
   - POC-DC01.contoso.com
   - POC-SQL01.contoso.com
   - POC-VMM01.contoso.com
   - POC-COMP1.contoso.com
   - POC-COMP2.contoso.com

### 服务器管理功能

连接成功后，可以使用以下管理功能：

| 功能 | 说明 |
|------|------|
| 概览 | 查看服务器CPU、内存、网络使用概况 |
| 证书 | 管理服务器上的证书 |
| 设备 | 查看硬件设备信息 |
| 事件 | 查看Windows事件日志 |
| 文件 | 浏览和管理服务器文件系统 |
| 防火墙 | 管理Windows防火墙规则 |
| 已安装的应用 | 查看安装的软件列表 |
| 网络 | 查看和配置网络适配器 |
| PowerShell | 打开远程PowerShell会话 |
| 进程 | 查看和管理运行中的进程 |
| 注册表 | 浏览和编辑注册表 |
| 远程桌面 | 通过浏览器进行RDP连接 |
| 角色和功能 | 管理Windows Server角色和功能 |
| 服务 | 管理Windows服务 |
| 存储 | 管理磁盘和存储 |
| 虚拟机 | 管理Hyper-V虚拟机（仅Hyper-V主机） |

## SDN管理扩展

WAC提供了SDN管理扩展，可以通过Web界面管理SDN组件：

### 安装SDN扩展

1. 在WAC中，点击右上角的`设置`图标
2. 进入`扩展`
3. 搜索`SDN`
4. 安装`SDN Infrastructure`扩展

### SDN扩展功能

| 功能 | 说明 |
|------|------|
| 虚拟网络 | 查看和管理HNV虚拟网络 |
| 负载均衡器 | 管理SLB VIP和负载均衡规则 |
| 网关 | 查看SDN网关状态 |
| 访问控制列表 | 管理ACL规则 |
| 逻辑网络 | 查看逻辑网络配置 |

## 检查

安装完成后，进行以下验证：

1. 通过浏览器访问WAC，确认能正常登录
2. 添加至少一台服务器并确认连接成功
3. 测试远程PowerShell和远程桌面功能
4. 如已安装SDN扩展，确认能查看SDN组件信息

## 课后习题

- 使用WAC的远程PowerShell功能，在管理主机上远程执行一段脚本。
- 对比WAC和传统的`服务器管理器`（Server Manager）的功能差异。

