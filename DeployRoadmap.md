# 部署路线图

本章节以表格形式列出完整的部署步骤先后关系，帮助读者一眼看清"什么时候该做什么"。

## 部署阶段总览

整个SDDC POC部署分为五个阶段，每个阶段都有明确的前置条件和输出成果。

| 阶段 | 名称 | 时长估算 | 关键产出 |
|------|------|----------|----------|
| 第一阶段 | 基础环境准备 | 2-3小时 | Hyper-V就绪、虚拟机模板、域控制器 |
| 第二阶段 | 管理平台搭建 | 3-4小时 | SQL Server、VMM就绪 |
| 第三阶段 | 计算与网络基础 | 2-3小时 | 计算节点纳管、DCGW路由、CA证书 |
| 第四阶段 | SDN组件部署 | 4-6小时 | NC、MUX、Gateway全部就绪 |
| 第五阶段 | 验证与监控 | 2-3小时 | SDN功能验证通过、SCOM上线 |

## 详细部署步骤

### 第一阶段：基础环境准备

| 步骤 | 操作 | 前置条件 | 对应章节 |
|------|------|----------|----------|
| 1 | 启用Hyper-V角色 | 物理主机/Azure VM就绪 | [启用Hyper-V角色](EnableHyper-V.md) |
| 2 | 创建Hyper-V虚拟交换机 | Hyper-V已启用 | [启用Hyper-V虚拟交换机](CreateVswitch.md) |
| 3 | 创建第一台虚拟机 | 虚拟交换机已创建 | [第一台虚拟机](FirstVM.md) |
| 4 | 制作虚拟机模板 | 第一台VM已完成sysprep | [创建虚拟机模板](CreateTemplate.md) |
| 5 | 部署域控制器 | VHDX模板就绪 | [Active Directory域控制器](ADDS.md) |
| 6 | 配置DNS | 域控制器就绪 | [DNS服务配置](Deploy/dns-config.md) |
| 7 | 创建AD账号 | 域控制器就绪 | [创建额外的AD账号](CreateSomeUsers.md) |

### 第二阶段：管理平台搭建

| 步骤 | 操作 | 前置条件 | 对应章节 |
|------|------|----------|----------|
| 8 | 部署SQL Server | AD就绪、SQL账号已创建 | [SQL Server](SQLServer.md) |
| 9 | 部署VMM | AD就绪、SQL就绪 | [Virtual Machine Manager](SCVMM.md) |
| 10 | 安装VMM UR补丁 | VMM安装完成 | [VMM UR补丁](VMMUR.md) |
| 11 | 配置组策略 | 域控制器就绪 | [组策略](GPMC.md) |

### 第三阶段：计算与网络基础

| 步骤 | 操作 | 前置条件 | 对应章节 |
|------|------|----------|----------|
| 12 | 部署第一台计算节点 | VMM就绪 | [第一台计算节点](FirstComp.md) |
| 13 | 部署第二台计算节点 | VMM就绪 | [第二台计算节点](SecondComp.md) |
| 14 | 部署RRAS软路由 | 虚拟交换机就绪 | [部署RRAS的软路由](DCGW.md) |
| 15 | 部署证书服务器 | AD就绪 | [证书服务器](CA1.md) |

> 注意：步骤12-15之间没有严格的依赖顺序，可以并行进行。

### 第四阶段：SDN组件部署

| 步骤 | 操作 | 前置条件 | 对应章节 |
|------|------|----------|----------|
| 16 | 创建证书模板并申请证书 | CA就绪 | [证书模板与申请](Deploy/cert-template.md) |
| 17 | 配置VMM逻辑网络 | VMM就绪、计算节点已纳管 | [VMM逻辑网络配置](Deploy/vmm-logical-network.md) |
| 18 | 部署网络控制器 | 证书就绪、逻辑网络已配置 | [网络控制器](Deploy/wang-luo-kong-zhi-qi.md) |
| 19 | 部署SLB MUX | NC就绪 | [SLB Multiplexer](Deploy/slb-multiplexer-mux.md) |
| 20 | 部署SDN网关 | NC就绪 | [Gateway](Deploy/gateway.md) |

### 第五阶段：验证与监控

| 步骤 | 操作 | 前置条件 | 对应章节 |
|------|------|----------|----------|
| 21 | SDN功能验证 | SDN全部组件就绪 | [SDN功能验证](Deploy/sdn-validation.md) |
| 22 | 创建租户虚拟网络 | NC就绪 | [创建租户虚拟网络](Deploy/tenant-virtual-network.md) |
| 23 | SLB负载均衡测试 | MUX就绪 | [SLB负载均衡测试](Deploy/slb-test.md) |
| 24 | 部署Operations Manager | AD就绪、SQL就绪 | [Operations Manager](Deploy/operations-manager-2019.md) |
| 25 | 部署Windows Admin Center | 任意已加域的服务器 | [Windows Admin Center](Deploy/windows-admin-center.md) |

## 课后习题

- 回顾部署路线图，思考哪些步骤可以并行执行以节省时间。
- 如果在第四阶段的网络控制器部署中遇到问题，应该回到哪些步骤进行排查？

