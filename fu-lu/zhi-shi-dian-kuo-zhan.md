# 知识点扩展

本章节对SDDC涉及的核心技术原理进行深入讲解，帮助读者理解"为什么这样设计"。

## SDN架构原理

### 控制平面与数据平面分离

传统网络中，每台交换机/路由器同时负责控制决策（路由计算）和数据转发。SDN将这两个功能分离：

| 平面 | 职责 | SDN组件 |
|------|------|---------|
| 控制平面 | 网络策略、路由决策、拓扑管理 | 网络控制器（NC） |
| 数据平面 | 实际的数据包转发 | Hyper-V虚拟交换机 + Host Agent |

**优势**：
- 集中管理：通过单一控制点管理整个网络
- 可编程：通过REST API自动化网络操作
- 灵活性：网络策略独立于物理设备

### NC的REST API模型

网络控制器使用声明式API模型：

```
管理员定义"期望状态" → NC计算差异 → NC下发配置到数据平面
```

这意味着管理员只需描述"网络应该是什么样"，NC会自动完成配置变更。

## VXLAN封装详解

### 为什么需要VXLAN

传统VLAN的限制：
- VLAN ID最多4094个，无法满足大规模多租户需求
- VLAN不能跨越三层边界
- VLAN配置需要手动在每台交换机上操作

VXLAN通过封装解决了这些问题：
- VNI为24位，支持约1600万个虚拟网络
- VXLAN使用UDP封装，可以跨越三层网络
- VXLAN配置由SDN控制器自动下发

### VXLAN帧格式

```
+--------------------------------------------------+
|    外部以太网头    |    外部IP头    |   UDP头    |
|   (物理网络MAC)   | (PA地址)       | (dst:4789) |
+--------------------------------------------------+
|  VXLAN头   |      原始以太网帧      |
| (VNI标识)  | (租户VM的MAC和IP)      |
+--------------------------------------------------+
```

- **外部以太网头**：物理网络上的源和目的MAC地址
- **外部IP头**：Provider Address（PA），即Hyper-V主机的HNV接口地址
- **UDP头**：目标端口4789（VXLAN标准端口）
- **VXLAN头**：包含24位VNI，标识虚拟网络
- **原始帧**：租户虚拟机的实际数据包

### PA与CA

在HNV中，有两类地址：

| 地址类型 | 全称 | 说明 | 示例 |
|----------|------|------|------|
| CA | Customer Address | 租户虚拟机使用的地址 | 10.0.1.10 |
| PA | Provider Address | 物理网络上的封装地址 | 192.148.12.x |

NC负责维护CA到PA的映射关系，当租户VM通信时，Hyper-V主机上的HNV模块根据映射关系进行VXLAN封装/解封装。

## BGP协议深入

### BGP基础概念

BGP（Border Gateway Protocol）是互联网的核心路由协议。在SDN中，BGP用于SLB MUX向DCGW通告VIP路由。

| 概念 | 说明 |
|------|------|
| AS（自治系统） | 一组在统一路由策略下运行的网络设备 |
| ASN | AS编号，POC中DCGW使用65002，MUX使用65001 |
| eBGP | 不同AS之间的BGP会话 |
| iBGP | 同一AS内部的BGP会话 |
| BGP Peer | BGP邻居，建立BGP会话的两端 |

### SDN中的BGP工作流程

```
1. MUX接收NC的指令，创建VIP配置
2. MUX通过eBGP向DCGW通告VIP路由
3. DCGW学习到VIP路由，更新路由表
4. 外部流量到达DCGW时，根据BGP路由转发到MUX
5. MUX根据负载均衡规则，将流量分发到后端DIP
```

### ECMP（等价多路径）

当部署多台MUX时，它们都会向DCGW通告相同的VIP路由。DCGW使用ECMP（Equal-Cost Multi-Path）在多台MUX之间分发流量，实现MUX层面的负载均衡和高可用。

## HNV架构

### HNV工作原理

Hyper-V Network Virtualization在Hyper-V虚拟交换机中实现网络虚拟化：

```
租户VM → Hyper-V虚拟交换机 → HNV模块（封装/解封装） → 物理网络
```

HNV模块的关键功能：
- **地址查找**：根据NC下发的CA-PA映射表，确定目标PA地址
- **VXLAN封装**：将租户数据包封装在VXLAN中
- **策略执行**：执行ACL等网络策略
- **分布式路由**：在虚拟子网之间进行路由

### Host Agent

每台Hyper-V主机上运行两个SDN Agent：

| Agent | 功能 |
|-------|------|
| NCHostAgent | 与NC通信，接收网络策略和配置 |
| SlbHostAgent | 处理SLB相关的数据包转发 |

## Azure Stack HCI与SDDC对比

| 特性 | 本文SDDC方案 | Azure Stack HCI |
|------|-------------|-----------------|
| 管理方式 | VMM + SCOM（本地） | Windows Admin Center + Azure Arc |
| SDN | NC + MUX + GW（本地） | 集成Azure SDN |
| 存储 | S2D（本地） | S2D + Azure混合存储 |
| 授权模式 | 传统软件许可 | 按月订阅 |
| 云集成 | 可选 | 深度Azure集成 |
| 适用场景 | 纯本地数据中心 | 混合云场景 |

## 课后习题

- 使用Wireshark或netsh trace在计算节点上抓取VXLAN封装的数据包，分析包结构。
- 了解一下除了VXLAN之外，还有哪些网络封装协议（如NVGRE、Geneve），它们各自的特点是什么？

