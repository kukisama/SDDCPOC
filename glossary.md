# 术语表

本文档列出SDDC POC项目中涉及的核心术语和缩写，帮助读者快速理解技术概念。

## 基础架构

| 术语 | 全称 | 说明 |
|------|------|------|
| SDDC | Software Defined DataCenter | 软件定义数据中心，通过软件实现计算、存储、网络的全面虚拟化和自动化管理 |
| POC | Proof of Concept | 概念验证，用于验证技术方案的可行性 |
| Hyper-V | - | 微软的服务器虚拟化技术，提供硬件虚拟化平台 |
| VM | Virtual Machine | 虚拟机，在物理硬件上通过虚拟化技术创建的逻辑计算机 |
| VHDX | Virtual Hard Disk Extended | Hyper-V使用的虚拟硬盘格式 |

## System Center 组件

| 术语 | 全称 | 说明 |
|------|------|------|
| VMM | Virtual Machine Manager | System Center组件，用于管理虚拟化基础设施 |
| SCVMM | System Center Virtual Machine Manager | VMM的完整名称 |
| SCOM | System Center Operations Manager | System Center组件，用于监控和告警 |
| UR | Update Rollup | 更新汇总补丁，包含多个修复和改进 |
| WAC | Windows Admin Center | 微软的新一代服务器管理工具 |

## 网络与SDN

| 术语 | 全称 | 说明 |
|------|------|------|
| SDN | Software Defined Networking | 软件定义网络，将网络控制平面与数据平面分离 |
| NC | Network Controller | 网络控制器，SDN的控制平面，提供REST API管理网络 |
| HNV | Hyper-V Network Virtualization | Hyper-V网络虚拟化，使用VXLAN封装实现虚拟网络 |
| SLB | Software Load Balancer | 软件负载均衡器，SDN中的负载均衡组件 |
| MUX | Multiplexer | SLB的流量分发器，负责VIP流量的分发 |
| VIP | Virtual IP Address | 虚拟IP地址，负载均衡器对外暴露的IP地址 |
| DIP | Direct IP Address | 直接IP地址，后端服务器的实际IP地址 |
| DSR | Direct Server Return | 直接服务器返回，一种负载均衡模式，回程流量直接返回客户端 |
| VXLAN | Virtual Extensible LAN | 虚拟可扩展局域网，一种网络封装协议，使用UDP封装二层帧 |
| VNI | VXLAN Network Identifier | VXLAN网络标识符，类似于VLAN ID，用于标识虚拟网络 |
| ACL | Access Control List | 访问控制列表，用于控制网络流量的安全策略 |

## 路由与网关

| 术语 | 全称 | 说明 |
|------|------|------|
| RRAS | Routing and Remote Access Service | 路由和远程访问服务，Windows Server的路由功能 |
| DCGW | DataCenter Gateway | 数据中心网关，连接数据中心内部网络与外部网络 |
| BGP | Border Gateway Protocol | 边界网关协议，用于自治系统之间交换路由信息 |
| ASN | Autonomous System Number | 自治系统编号，BGP中标识独立网络域的编号 |
| GRE | Generic Routing Encapsulation | 通用路由封装，一种隧道协议 |
| IPsec | Internet Protocol Security | Internet协议安全，提供加密和认证的VPN协议 |
| L3 | Layer 3 | 三层（网络层），指基于IP地址的路由转发 |

## Active Directory 与安全

| 术语 | 全称 | 说明 |
|------|------|------|
| AD | Active Directory | 活动目录，微软的目录服务，提供身份认证和授权 |
| AD DS | Active Directory Domain Services | AD域服务，AD的核心组件 |
| DC | Domain Controller | 域控制器，运行AD DS的服务器 |
| DNS | Domain Name System | 域名系统，将域名解析为IP地址 |
| OU | Organizational Unit | 组织单位，AD中用于组织对象的容器 |
| GPO | Group Policy Object | 组策略对象，用于批量管理域内计算机和用户配置 |
| CA | Certificate Authority | 证书颁发机构，负责颁发和管理数字证书 |
| PKI | Public Key Infrastructure | 公钥基础设施，管理数字证书和公钥加密的框架 |

## 存储

| 术语 | 全称 | 说明 |
|------|------|------|
| S2D | Storage Spaces Direct | 存储空间直通，使用本地磁盘构建高可用分布式存储 |
| CSV | Cluster Shared Volume | 群集共享卷，故障转移群集中的共享存储 |
| MPIO | Multipath I/O | 多路径IO，提供存储访问的冗余路径 |

## 其他

| 术语 | 全称 | 说明 |
|------|------|------|
| FQDN | Fully Qualified Domain Name | 完全限定域名，如`poc-dc01.contoso.com` |
| REST | Representational State Transfer | 表述性状态转移，一种Web API设计风格 |
| PowerShell | - | 微软的命令行和脚本环境 |
| ISE | Integrated Scripting Environment | 集成脚本环境，PowerShell的图形化编辑器 |
| WinRM | Windows Remote Management | Windows远程管理协议 |
| sysprep | System Preparation | 系统准备工具，用于制作操作系统映像 |
| ADK | Assessment and Deployment Kit | 评估和部署工具包 |

