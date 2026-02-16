# 部署 — 监控与工具

以下为监控平台和管理工具的部署章节。

## 概述

SDDC的日常运维需要监控和管理工具的支持。本章节介绍Operations Manager（SCOM）和Windows Admin Center（WAC）的部署方法。

- **Operations Manager**：提供全面的服务器和应用监控能力，通过管理包扩展监控范围
- **Windows Admin Center**：提供轻量级的Web管理界面，支持服务器管理、Hyper-V管理和SDN扩展

## 前置条件

- [ ] AD域和SQL Server已就绪
- [ ] 基础设施部署已完成

## 子章节导航

| 章节 | 内容 |
|------|------|
| [Operations Manager](operations-manager-2019.md) | SCOM 2019安装配置、管理包导入、Agent部署 |
| [Operations Manager UR补丁](scom-ur.md) | SCOM更新汇总补丁安装和Agent更新 |
| [Windows Admin Center](windows-admin-center.md) | WAC安装、服务器管理、SDN扩展 |
