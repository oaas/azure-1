## Here is an example that how to do reverse DNS on Azure VM

####Before you go:
> The PublicIp allocation method must be static.

> The PublicIP has FQDN on it.

1. Get the IP resource that you want to do reverse DNS, please make sure it has FQDN on it, example below.


```
PS C:\Users\zheli> Get-AzureRmPublicIpAddress
Name                     : srv-lambert-sk-test1-ip
ResourceGroupName        : R-Lambert
Location                 : koreasouth
Id                       : /subscriptions/80a8bebd-6c4a-4dda-98f2-00151f9c3567/resourceGroups/R-Lambert/providers/Micro
                           soft.Network/publicIPAddresses/srv-lambert-sk-test1-ip
Etag                     : W/"8fc62947-aa3c-492b-b383-38f35f4e8c6c"
ResourceGuid             : 53f18520-dcc9-43af-b5aa-3ddc944b294e
ProvisioningState        : Succeeded
Tags                     :
PublicIpAllocationMethod : Static        //Static
IpAddress                : 52.231.204.65
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                             "Id": "/subscriptions/80a8bebd-6c4a-4dda-98f2-00151f9c3567/resourceGroups/R-Lambert/provid
                           ers/Microsoft.Network/networkInterfaces/srv-lambert-sk-test1703/ipConfigurations/ipconfig1"
                           }
DnsSettings              : {
                             "DomainNameLabel": "lambert",
                             "Fqdn": "lambert.koreasouth.cloudapp.azure.com"                //FQDN here
                           }
Zones                    : {}
Sku                      : {
                             "Name": "Basic"
                           }
						   
```	
					   
2 . As you can see in step1, the public ip resource name is "srv-lambert-sk-test1-ip", Resource Group Name is "R-Lambert", FQDN is "lambert.koreasouth.cloudapp.azure.com". Follow the commands below do finish it.

```	   
PS C:\Users\zheli> $pip = Get-AzureRmPublicIpAddress -Name "srv-lambert-sk-test1-ip" -ResourceGroupName "R-Lambert"
PS C:\Users\zheli> $pip.DnsSettings.ReverseFqdn = "lambert.koreasouth.cloudapp.azure.com"
PS C:\Users\zheli> Set-AzureRmPublicIpAddress -PublicIpAddress $pip 
```			

3 . Test

```
PS C:\Users\zheli> nslookup 52.231.204.65
Server:  f5-1.redmond.corp.microsoft.com
Address:  10.50.50.50

Name:    lambert.koreasouth.cloudapp.azure.com
Address:  52.231.204.65	
```
     