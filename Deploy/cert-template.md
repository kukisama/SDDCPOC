# 证书模板与申请

完成`证书服务器`的部署后，需要为SDN组件（网络控制器、SLB MUX等）创建专用的证书模板并申请证书。证书是SDN各组件之间安全通信的基础，属于`必须环节`。

## 章节目标

- 在CA服务器上创建网络控制器专用的证书模板
- 为NC节点申请和导出证书
- 了解证书在SDN架构中的作用

## 背景知识

在Microsoft SDN架构中，网络控制器（NC）使用证书实现以下功能：

- NC节点之间的安全通信（Northbound通信）
- NC与管理客户端（如VMM）之间的REST API认证（Southbound通信）
- NC与Hyper-V主机之间的信任建立

## 创建证书模板

使用`域管理员`登录`POC-CA1`，打开`certtmpl.msc`（证书模板控制台）。

1. 在证书模板列表中，找到`Web服务器`模板，右键选择`复制模板`
2. 在弹出的属性窗口中，进行以下配置：

**常规选项卡：**
- 模板显示名称：`SDN REST Certificate`
- 有效期：`3 年`

**请求处理选项卡：**
- 勾选`允许导出私钥`

**使用者名称选项卡：**
- 选择`在请求中提供`

**安全选项卡：**
- 添加`Domain Computers`，授予`读取`和`注册`权限

3. 点击`确定`保存模板

## 发布证书模板

打开`certsrv.msc`（证书颁发机构控制台），执行以下操作：

1. 展开CA服务器节点
2. 右键点击`证书模板`，选择`新建 → 要颁发的证书模板`
3. 在列表中找到并选择`SDN REST Certificate`，点击`确定`

## 申请证书

在需要安装证书的服务器上（例如VMM服务器或NC节点），可以使用以下PowerShell命令申请证书：

```powershell
# 定义NC的REST名称（FQDN），需与DNS中的A记录一致
$RestName = "nc.contoso.com"

# 创建证书申请INF文件
$InfContent = @"
[Version]
Signature = "`$Windows NT`$"

[NewRequest]
Subject = "CN=$RestName"
Exportable = TRUE
KeyLength = 2048
KeySpec = 1
KeyUsage = 0xa0
MachineKeySet = True
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = CMC

[EnhancedKeyUsageExtension]
OID=1.3.6.1.5.5.7.3.1

[Extensions]
2.5.29.17 = "{text}"
_continue_ = "dns=$RestName&"
"@

# 保存INF文件
$InfContent | Out-File -FilePath "C:\NCCertRequest.inf" -Encoding ASCII

# 创建证书请求
certreq -new "C:\NCCertRequest.inf" "C:\NCCertRequest.req"

# 提交证书请求到CA
certreq -submit -config "POC-CA1.contoso.com\contoso-POC-CA1-CA" "C:\NCCertRequest.req" "C:\NCCert.cer"

# 接受证书
certreq -accept "C:\NCCert.cer"
```

## 导出证书

申请成功后，需要将证书导出为PFX格式，以便在VMM部署SDN时使用：

```powershell
# 查找刚才申请的证书
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -match "nc.contoso.com" }

# 设置导出密码
$password = ConvertTo-SecureString "poc.123" -AsPlainText -Force

# 导出为PFX文件
Export-PfxCertificate -Cert $cert -FilePath "C:\NCCert.pfx" -Password $password

# 查看证书指纹（后续部署NC时需要用到）
Write-Host "证书指纹: $($cert.Thumbprint)"
```

> 注意：请记录证书的`指纹（Thumbprint）`，在后续部署网络控制器时会用到。

## 检查

完成证书模板创建和证书申请后，进行以下验证：

1. 在`certsrv.msc`的`颁发的证书`中，确认能看到刚才申请的证书
2. 确认导出的PFX文件存在
3. 确认证书的使用者名称包含NC的REST FQDN

## 课后习题

- 了解一下X.509证书的基本结构，包括公钥、私钥和证书链的概念。
- 思考一下，为什么SDN组件之间需要使用证书进行认证，而不是仅使用域认证？

