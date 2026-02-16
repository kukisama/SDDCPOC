# SLB负载均衡测试

完成SLB MUX部署后，本章节通过实际的负载均衡场景验证SLB功能的正确性。

## 章节目标

- 创建VIP并配置负载均衡规则
- 部署后端虚拟机
- 测试VIP访问和负载分发效果

## 前置条件

| 条件 | 说明 |
|------|------|
| 网络控制器已部署 | NC REST API可正常访问 |
| SLB MUX已部署 | MUX与DCGW的BGP对等已建立 |
| PublicVIP地址池 | VMM中已配置VIP地址池 |
| 租户虚拟网络 | 已创建租户虚拟网络和虚拟子网 |

## 步骤一：准备后端虚拟机

创建两台测试虚拟机作为后端服务器，部署在租户虚拟网络中：

| 虚拟机 | IP地址 | 功能 |
|--------|--------|------|
| Web-VM01 | 10.0.1.10 | Web服务器1 |
| Web-VM02 | 10.0.1.11 | Web服务器2 |

在两台VM上分别安装IIS并配置测试页面：

```powershell
# 在Web-VM01和Web-VM02上分别执行
Install-WindowsFeature Web-Server -IncludeManagementTools

# 创建测试页面（在每台VM上显示不同内容以区分）
$hostname = $env:COMPUTERNAME
Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value "<h1>Hello from $hostname</h1>"
```

## 步骤二：创建负载均衡器

通过NC REST API创建负载均衡器：

```powershell
$NcUri = "https://nc.contoso.com"

# 定义负载均衡器配置
$LBProperties = @{
    properties = @{
        frontendIPConfigurations = @(
            @{
                resourceId = "FrontEnd1"
                properties = @{
                    subnet = @{
                        resourceRef = "/networking/v1/logicalnetworks/PublicVIP/subnets/PublicVIPSubnet"
                    }
                    privateIPAddress = "192.148.15.10"
                    privateIPAllocationMethod = "Static"
                }
            }
        )
        backendAddressPools = @(
            @{
                resourceId = "BackEndPool1"
                properties = @{
                    backendIPConfigurations = @()
                }
            }
        )
        loadBalancingRules = @(
            @{
                resourceId = "HTTPRule"
                properties = @{
                    frontendIPConfigurations = @(
                        @{ resourceRef = "/networking/v1/loadBalancers/WebLB/frontendIPConfigurations/FrontEnd1" }
                    )
                    backendAddressPool = @{
                        resourceRef = "/networking/v1/loadBalancers/WebLB/backendAddressPools/BackEndPool1"
                    }
                    protocol = "TCP"
                    frontendPort = 80
                    backendPort = 80
                    enableFloatingIP = $false
                    idleTimeoutInMinutes = 4
                }
            }
        )
        probes = @(
            @{
                resourceId = "HealthProbe"
                properties = @{
                    protocol = "HTTP"
                    port = 80
                    requestPath = "/"
                    intervalInSeconds = 15
                    numberOfProbes = 2
                }
            }
        )
    }
}

$body = $LBProperties | ConvertTo-Json -Depth 10
Invoke-RestMethod -Uri "$NcUri/networking/v1/loadBalancers/WebLB" `
    -Method Put `
    -Body $body `
    -UseDefaultCredentials `
    -ContentType "application/json"
```

## 步骤三：添加后端VM到地址池

将Web-VM01和Web-VM02的网络接口添加到后端地址池中。这一步通常通过VMM控制台或NC REST API完成。

## 步骤四：测试VIP访问

### 测试连通性

从DCGW或管理网络中的主机测试VIP访问：

```powershell
# 测试VIP的HTTP访问
Invoke-WebRequest -Uri "http://192.148.15.10" -UseBasicParsing

# 多次访问，观察负载分发效果
1..10 | ForEach-Object {
    $response = Invoke-WebRequest -Uri "http://192.148.15.10" -UseBasicParsing
    Write-Host "请求 $_ : $($response.Content)"
}
```

### 验证负载分发

多次访问VIP后，应该能看到请求被分发到不同的后端VM（响应内容显示不同的主机名）。

### 测试故障转移

1. 停止Web-VM01上的IIS服务
2. 继续访问VIP
3. 所有请求应自动路由到Web-VM02
4. 重启Web-VM01的IIS服务
5. 请求应重新在两台VM之间分发

```powershell
# 模拟故障：停止Web-VM01的IIS
Invoke-Command -ComputerName "Web-VM01" { Stop-Service W3SVC }

# 测试VIP是否仍然可用
Invoke-WebRequest -Uri "http://192.148.15.10" -UseBasicParsing

# 恢复Web-VM01
Invoke-Command -ComputerName "Web-VM01" { Start-Service W3SVC }
```

## 检查

| 验证项 | 预期结果 |
|--------|----------|
| VIP可访问 | HTTP 200响应 |
| 负载分发 | 请求分配到不同后端VM |
| 健康检查 | 停止一台后端VM后，流量自动切换到健康的VM |
| BGP路由 | DCGW上能看到VIP的BGP路由条目 |

通过DCGW验证BGP路由：

```powershell
Invoke-Command -VMName "poc-dcgw" -Credential $cred {
    Get-BgpRouteInformation | Where-Object { $_.Network -match "192.148.15" }
}
```

## 课后习题

- 修改负载均衡规则，将前端端口改为8080，后端端口保持80，测试是否正常工作。
- 了解一下SLB支持的负载均衡算法（如五元组哈希），思考不同算法的适用场景。

