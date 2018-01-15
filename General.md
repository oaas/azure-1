### Scope:

We will help you in consulting that how to collect VPN gateway and NSG logs. Microsoft works on 'one issue per incident' basis. As per Microsoft an incident is defined as an issue that cannot be broken down any further. In case you have any other issue after this, you would have to create a separate incident for it.

Alternatively, we will consider this case resolved if we confirm that the problem is by-design. If it is caused by a 3rd party application, we will provide best efforts from Azure to assist 3rd party to resolve the problem.
This agreed resolution statement describes the specific conditions where we will close this case. Please let me know if you do not agree with this scope.


### Log Format:
•	Symptom:
•	Environment: (tenant id, subscription id, vm info, pip etc)
•	What TS you have done:
•	Which wiki article are you following:
•	Current status/findings:
•	Next plan:

### Risk Assessment

1) TIMELY RESPONSE: IR/callback-commitment missed? Or there is response delay complained/pushed by the customer? (Y/N. IF YES, LIST THE EXACT RISK)
   N
2) SCOPE: Requests are out of scope and customer has concern to separate? (Y/N)
   N
3) DUPLICATED: Duplicated case? (Y/N. IF YES, LIST THE EXACT RISK)
   N
4) PRODUCT: Product limitation/bug/SIE/documentation? (Y/N. IF YES, LIST THE EXACT RISK)
   N
5) COLLABORATION: Handover happens and there is delay or unsmooth transition? Collaboration delay happened or insufficient support from other team? (Y/N. IF YES, LIST THE EXACT RISK)
   N
6) 3rd PARTY: 3rd party issue? (Y/N. IF YES, LIST THE EXACT RISK)
   N
7) NO RESOLUTION: Customer resolved it? Close with no solution? Customer did not implement the solution? Not supported (Y/N. IF YES, LIST THE EXACT RISK)
   N
8) OTHER CONCERN: any other concerns the customer presents? (Y/N. IF YES, LIST THE EXACT RISK)
   N
CUSTOMER HISTORY: new customer or challenging history? (Y/N. IF YES, LIST THE EXACT RISK)
   N
   
   
### Reset Application Gateway
Login-azurermaccount
$app = Get-AzureRmApplicationGateway -Name applicationgatewayname -ResourceGroupName RGname
Add-AzureRmApplicationGatewayBackendAddressPool -ApplicationGateway $app -Name name -BackendIPAddresses x.x.x.x
Set-AzureRmApplicationGateway -ApplicationGateway $app    