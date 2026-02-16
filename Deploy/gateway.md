# Gateway

完成`网络控制器`的部署后，可以开始部署SDN网关（Gateway）。SDN网关提供跨站点的网络连接能力，支持GRE隧道、IPsec VPN和L3转发三种模式。

## 章节目标

- 理解SDN网关在SDN架构中的角色
- 通过VMM部署SDN网关
- 验证网关与网络控制器的集成

## 背景知识

SDN网关是Windows Server SDN架构中负责南北向流量（即租户虚拟网络与外部网络之间通信）的组件，提供以下连接模式：

| 连接模式 | 说明 | 典型场景 |
|----------|------|----------|
| GRE隧道 | 使用通用路由封装协议建立隧道 | 数据中心之间的互联 |
| IPsec VPN | 使用IPsec加密的站点到站点VPN | 与远程站点或Azure的安全连接 |
| L3转发 | 三层路由转发 | 租户虚拟网络与物理网络互通 |

在POC环境中，SDN网关主要用于验证租户虚拟网络与外部网络的连通性。

## 前置条件

| 条件 | 说明 |
|------|------|
| 网络控制器已部署 | NC的REST API可正常访问 |
| HNV Transit逻辑网络 | VMM中已创建HNV Transit网络 |
| 计算节点 | 至少一台计算节点可用于承载网关VM |
| DCGW | 数据中心网关已部署，BGP已启用 |

## 部署SDN网关

SDN网关通过VMM的服务模板进行部署，流程与SLB MUX类似。

### 步骤一：配置网关服务

1. 打开`Virtual Machine Manager 控制台`
2. 进入`构造 → 网络 → 网络服务`
3. 在已部署的网络控制器上，配置网关相关设置：
   - 指定网关的管理网络（Management逻辑网络）
   - 指定网关的前端网络（HNV Transit逻辑网络）
   - 指定网关池的容量设置

### 步骤二：部署网关虚拟机

1. 在VMM中，通过网络控制器的配置界面添加网关
2. 指定网关部署的目标计算节点
3. 配置网关虚拟机的资源分配

网关虚拟机的网络配置：

| 网络接口 | 连接的逻辑网络 | 用途 |
|----------|---------------|------|
| Management | Management | 管理通信 |
| Front-end | HNV Transit | 与外部网络通信 |
| Back-end | HNV Transit | 与租户虚拟网络通信 |

### 步骤三：配置网关连接

部署完成后，可以创建网关连接来测试功能：

```powershell
# 通过NC REST API创建L3网关连接示例
$NcUri = "https://nc.contoso.com"

# 获取网关池信息
$GatewayPools = Invoke-RestMethod -Uri "$NcUri/networking/v1/gatewayPools" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"

# 查看网关池状态
$GatewayPools.value | ForEach-Object {
    Write-Host "网关池: $($_.properties.type) - 状态: $($_.properties.state)"
}
```

## 检查

### 验证网关状态

通过NC REST API查看网关状态：

```powershell
$NcUri = "https://nc.contoso.com"

# 查看网关列表
Invoke-RestMethod -Uri "$NcUri/networking/v1/gateways" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

### 验证VMM中的状态

在VMM控制台的`构造 → 网络 → 网络服务`中，确认SDN网关的状态为正常。

### 验证与NC的集成

确认网关已成功注册到网络控制器，且网关池中显示可用容量。

## 课后习题

- 了解一下GRE（通用路由封装）和IPsec的区别，以及各自的适用场景。
- 在生产环境中，SDN网关通常部署多少台？为什么需要网关池的概念？

