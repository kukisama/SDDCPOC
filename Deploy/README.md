# 部署

以下为正式部署章节，请按照顺序逐篇文章阅读。

## 部署流程概述

基础设施部署是SDDC POC的核心阶段，涵盖从Hyper-V环境搭建到管理平台部署的完整流程。每个步骤都有明确的前置条件和输出成果。

## 前置条件检查

在开始部署之前，请确认以下条件已满足：

- [ ] 物理主机或Azure VM已就绪，满足最小硬件配置要求
- [ ] 所有需要的ISO文件已下载
- [ ] IP地址和虚拟机名称规划已确定
- [ ] 账号体系已规划

## 子章节导航

| 序号 | 章节 | 内容 |
|------|------|------|
| 1 | [启用Hyper-V角色](../EnableHyper-V.md) | 在物理主机或Azure VM上启用Hyper-V |
| 2 | [启用Hyper-V虚拟交换机](../CreateVswitch.md) | 创建内部虚拟交换机 |
| 3 | [第一台虚拟机](../FirstVM.md) | 创建第一台VM、安装系统、sysprep |
| 4 | [创建虚拟机模板](../CreateTemplate.md) | 提取VHDX模板用于快速部署 |
| 5 | [Active Directory域控制器](../ADDS.md) | 部署域控制器和DNS服务 |
| 6 | [DNS服务配置](dns-config.md) | 配置DNS转发器和NC所需的DNS记录 |
| 7 | [创建额外的AD账号](../CreateSomeUsers.md) | 创建OU、用户和安全组 |
| 8 | [SQL Server](../SQLServer.md) | 安装SQL Server数据库 |
| 9 | [Virtual Machine Manager](../SCVMM.md) | 安装VMM管理平台 |
| 10 | [VMM UR补丁](../VMMUR.md) | 安装VMM更新汇总补丁 |
| 11 | [第一台计算节点](../FirstComp.md) | 部署计算节点并纳入VMM管理 |
| 12 | [第二台计算节点](../SecondComp.md) | 部署第二台计算节点 |
| 13 | [组策略](../GPMC.md) | 配置域策略（如关闭防火墙） |
