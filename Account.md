# 账号体系约定

所需账号如下,请按照需求进行创建。

> 虽然可以`一次性创建`所有所需的账号，但`初次部署`，在不了解账号功能的情况下，建议还是按照右侧创建时间的顺序，逐个创建。

账号存在嵌套关系，请`仔细检查`需求进行配置。

| 名称          | 所在ou | 功能                 | 隶属安全组                    | 创建时间                |
| ------------- | ------ | -------------------- | ----------------------------- | ----------------------- |
| admin         | /sddc  | 域管理员             | domain admins                 | 完成AD后创建            |
| sqladmins     | /sddc  | 数据库管理员组       | POC-SQL01本地管理员           | 完成AD后创建            |
| sqladmin      | /sddc  | 数据库管理员         | sqladmins                     | 完成AD后创建            |
| sqlsvc        | /sddc  | 数据库服务账号       | sqladmins                     | 完成AD后创建            |
| vmmadmins     | /sddc  | vmm管理员组          | POC-VMM01本地管理员/sqladmins | 完成SQL配置后创建       |
| vmmadmin      | /sddc  | vmm管理员            | vmmadmins                     | 完成SQL配置后创建       |
| vmmsvc        | /sddc  | vmm服务账号          | vmmadmins                     | 完成SQL配置后创建       |
| computeradmin | /sddc  | 计算节点的本地管理员 | POC-COMP01本地管理员          | 完成计算节点1部署后创建 |





## 将某个账号配置为本地管理员

该操作较为繁琐，且会重复遇到多次，因此建议在Active Directory域控制器上，使用`域管理员`来完成。

打开`管理员的PowerShell`执行如下命令，即可快速实现配置。

```powershell
#以下例子表示,将"contoso\sqladmin"添加到"poc-sql01"的本地管理员组中
$computername="poc-sql01"
$account="contoso\sqladmin"

icm $computername {Add-LocalGroupMember administrators `
-Member $using:account} 
```

