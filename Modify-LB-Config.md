

### 如何在现有的Azure LoadBalancer和现有的后端资源上通过Powershell创建Nat规则和转发规则

####环境：
* 负载均衡:  Azure-Lambert-LB(通过portal创建，只有前端IP地址，其他都为默认配置)
* 资源组(Resource Group)：RG-Lambert
* 后端VM：srv-lambert-ubuntu-test2, 对应的网络接口为：srv-lambert-ubuntu-t127 

####通过以下操作，您可以实现：
* 在LB上创建名为LB-backend的backend pool，将srv-lambert-ubuntu-test2加入此backendpool
* 在LB上创建端口8080到8080的Nat转发
* 在LB上创建到后端tcp 80的健康检查
* 在LB上创建80到80的转发规则

> 这里假设您已经通过PowerShell登陆Azure并选好SubscriptionID

1 . 获取当前存在的LB并赋值变量给$slb
```
PS C:\Users\zheli> $slb = get-AzureRmLoadBalancer -Name Azure-Lambert-LB -ResourceGroupName RG-Lambert
```

2 . 创建backend， 名字为LB-backend
```
PS C:\Users\zheli> $slb | Add-AzureRmLoadBalancerBackendAddressPoolConfig -Name "LB-backend" | Set-AzureRmLoadBalancer
```

3 . 获取VM:srv-lambert-ubuntu-test2的网卡接口
```
PS C:\Users\zheli> $nic =get-azurermnetworkinterface -name srv-lambert-ubuntu-t127  -resourcegroupname RG-Lambert
```  

4 . 将VM:srv-lambert-ubuntu-test2添加到LB的backend pool
```
PS C:\Users\zheli> $backend=Get-AzureRmLoadBalancerBackendAddressPoolConfig -name LB-backend -LoadBalancer $slb
PS C:\Users\zheli> $nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$backend
PS C:\Users\zheli> Set-AzureRmNetworkInterface -NetworkInterface $nic
```
5 . 创建Nat规则, 注意LB当前只支持添加单台VM, ScaleSet, Availability Set作为backend，目前Azure-Lambert-LB后端是一台VM，所以在添加Nat规则的时候不需要指定后端。
```
PS C:\Users\zheli> $slb | Add-AzureRmLoadBalancerInboundNatRuleConfig -Name Rule-8080 -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 8080  -BackendPort 8080 -Protocol TCP
PS C:\Users\zheli> $slb | Set-AzureRmLoadBalancer
```

6 . 创建forward规则
```
#6.1 创建后端的健康检查
PS C:\Users\zheli> $slb | Add-AzureRmLoadBalancerProbeConfig -Name app-health-80 -Protocol Tcp -Port 80 -IntervalInSeconds 15 -ProbeCount 2

#6.2 获取名字为"LoadBalancerFrontEnd"的frontend ip
PS C:\Users\zheli> $frontip=Get-AzureRmLoadBalancerFrontendIpConfig -Name "LoadBalancerFrontEnd" -LoadBalancer $slb

#6.3 创建forwarding规则，分发模式为SourceIP
PS C:\Users\zheli> $slb |Add-AzureRmLoadBalancerRuleConfig -Name "app-80" -FrontendIPConfiguration  $frontip -BackendAddressPool $backend -Probe $probe  -Protocol Tcp -FrontendPort 80 -BackendPort 80 -IdleTimeoutInMinutes 15  -LoadDistribution SourceIP

# 6.4 保存并应用到LB
PS C:\Users\zheli> $slb | Set-AzureRmLoadBalancer
```


参考：
>https://docs.microsoft.com/en-us/powershell/module/azurerm.network/?view=azurermps-5.1.1#load_balancer
