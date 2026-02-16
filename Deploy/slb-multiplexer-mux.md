# SLB Multiplexer \(MUX\)

完成`网络控制器`的部署后，可以开始部署SLB Multiplexer（MUX）。MUX是SDN软件负载均衡的核心组件，负责将虚拟IP（VIP）流量分发到后端服务器池。

## 章节目标

- 理解SLB MUX在SDN中的角色
- 通过VMM部署SLB MUX
- 验证MUX与DCGW之间的BGP对等关系

## 背景知识

SLB Multiplexer在SDN架构中承担以下职责：

- **VIP流量分发**：接收发往VIP地址的流量，根据负载均衡规则分发到后端虚拟机
- **BGP路由通告**：通过BGP协议向数据中心网关（DCGW）通告VIP路由
- **DSR支持**：支持Direct Server Return模式，回程流量可直接返回客户端
- **健康检查**：对后端VM进行健康检查，确保流量只分发到可用的后端

在POC环境中，DCGW的BGP配置中已经预留了MUX的BGP对等体配置（参见`DCGW.md`中被注释的代码）。

## 前置条件

| 条件 | 说明 |
|------|------|
| 网络控制器已部署 | NC的REST API可正常访问 |
| DCGW已配置BGP | DCGW的BGP路由器已启用，ASN 65002 |
| PublicVIP/PrivateVIP逻辑网络 | VMM中已创建VIP逻辑网络 |
| HNV Transit逻辑网络 | VMM中已创建HNV Transit网络 |

## 部署SLB MUX

SLB MUX通过VMM的服务模板进行部署。

### 步骤一：配置SLB管理器

1. 打开`Virtual Machine Manager 控制台`
2. 进入`构造 → 网络 → 网络服务`
3. 在已部署的网络控制器上，右键选择`属性`
4. 在`负载均衡器`配置中：
   - 关联`PublicVIP`逻辑网络到公共VIP池
   - 关联`PrivateVIP`逻辑网络到私有VIP池
   - 配置VIP地址池范围

### 步骤二：部署MUX虚拟机

1. 在VMM中，通过网络控制器的配置界面添加SLB MUX
2. 指定MUX部署的目标计算节点
3. VMM将自动创建MUX虚拟机并完成配置

MUX虚拟机的网络配置：

| 网络接口 | 连接的逻辑网络 | 用途 |
|----------|---------------|------|
| Management | Management | 管理通信 |
| HNV Transit | HNV Transit | BGP对等、VIP流量 |

### 步骤三：配置BGP对等

MUX部署完成后，需要确认与DCGW之间的BGP对等关系。在DCGW中，取消之前注释的BGP对等体配置：

```powershell
# 在DCGW上执行，添加MUX作为BGP邻居
# 需要根据实际MUX的IP地址修改
$cred = Get-Credential
$vmname = "poc-dcgw"

Invoke-Command -VMName $vmname -Credential $cred {
    # 添加MUX BGP对等体（IP地址根据实际部署调整）
    Add-BgpPeer -Name MUX001 `
        -LocalIPAddress 192.148.17.1 `
        -PeerIPAddress 192.148.12.81 `
        -LocalASN 65002 `
        -PeerASN 65001 `
        -OperationMode Mixed `
        -PeeringMode Automatic
}
```

> 注意：MUX的HNV Transit IP地址由VMM在部署时自动分配，需要在部署完成后确认实际地址，然后更新DCGW的BGP配置。

## 检查

### 验证MUX状态

通过NC REST API查看MUX状态：

```powershell
$NcUri = "https://nc.contoso.com"
Invoke-RestMethod -Uri "$NcUri/networking/v1/loadbalancerMuxes" `
    -Method Get `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

### 验证BGP对等

在DCGW上检查BGP邻居状态：

```powershell
Invoke-Command -VMName "poc-dcgw" -Credential $cred {
    Get-BgpPeer
    Get-BgpRouteInformation
}
```

BGP对等体状态应显示为`Connected`。

### VMM中验证

在VMM控制台的`构造 → 网络 → 网络服务`中，确认SLB MUX的状态为正常。

## 课后习题

- 了解一下BGP协议中AS（自治系统）的概念，思考为什么MUX和DCGW使用不同的ASN。
- 了解一下Direct Server Return（DSR）模式的工作原理，以及它相比传统负载均衡的优势。