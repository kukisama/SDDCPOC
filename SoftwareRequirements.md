# 软件配置

## 获取来源

| 来源                    | 访问地址                                                     | 适用人群                        |
| ----------------------- | ------------------------------------------------------------ | ------------------------------- |
| MSDN                    | https://my.visualstudio.com/Benefits                         | 微软员工/拥有MSDN订阅的企业用户 |
| 批量许可服务中心 (VLSC) | https://www.microsoft.com/Licensing/servicecenter/default.aspx | 购买EA的企业用户                |
|                         |                                                              |                                 |

## 需要的介质

MSDN和VLSC的介质名称和版本命名存在差异，请根据需要进行选择。所需内容以`中文版`为例。

| 功能      | 名称                                                         |
| --------- | ------------------------------------------------------------ |
| 操作系统  | cn_windows_server_2019_updated_march_2021_x64_dvd_2682a2dd.iso |
| 数据库    | cn_sql_server_2016_enterprise_with_service_pack_2_x64_dvd_12118964.iso（名称相同但是带core的版本支持20核以上的CPU） |
| VMM       | mu_system_center_virtual_machine_manager_2019_x64_dvd_06c18108.iso |
| SCOM      | mu_system_center_operations_manager_2019_x64_dvd_b3488f5c.iso |
| SCO       | mu_system_center_orchestrator_2019_x64_dvd_98784e83.iso      |
| ADK(免费) | [从下载页面](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install )获取`2004版`[WinPE add-on for the Windows ADK](https://go.microsoft.com/fwlink/?linkid=2166133)和[Windows ADK](https://go.microsoft.com/fwlink/?linkid=2165884) 并参考[离线安装](https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-offline-install)的方式，将数据保存到本地 |

## 介质存储

由于介质数据量都很大，为了便于`重复利用`，请妥善保存（Azure上请自建blob存储单独存放）

