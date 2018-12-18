---
title: 在 Azure 應用程式閘道上設定 SSL 原則 - PowerShell
description: 本分頁提供在 Azure 應用程式閘道上設定 SSL 原則的指示
documentationcenter: na
services: application-gateway
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/27/2018
ms.author: victorh
ms.openlocfilehash: 4c9ca5cee14603fb39115defc574aa7e956886ba
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30232131"
---
# <a name="configure-ssl-policy-versions-and-cipher-suites-on-application-gateway"></a>在應用程式閘道上設定 SSL 原則版本和加密套件

了解如何在應用程式閘道上設定 SSL 原則版本和加密套件。 您可以從[預先定義的原則清單](#predefined-ssl-policies)中選取，清單包含 SSL 原則版本和已啟用加密套件的不同設定。 您也可以根據您的需求定義[自訂 SSL 原則](#configure-a-custom-ssl-policy)。

## <a name="get-available-ssl-options"></a>取得可用的 SSL 選項

`Get-AzureRMApplicationGatewayAvailableSslOptions` Cmdlet 提供可用預先定義原則、可用加密套件，以及可以設定之通訊協定版本的清單。 下列範例顯示執行 Cmdlet 後的範例輸出。

```
DefaultPolicy: AppGwSslPolicy20150501
PredefinedPolicies:
    /subscriptions/147a22e9-2356-4e56-b3de-1f5842ae4a3b/resourceGroups//providers/Microsoft.Network/ApplicationGatewayAvailableSslOptions/default/Applic
ationGatewaySslPredefinedPolicy/AppGwSslPolicy20150501
    /subscriptions/147a22e9-2356-4e56-b3de-1f5842ae4a3b/resourceGroups//providers/Microsoft.Network/ApplicationGatewayAvailableSslOptions/default/Applic
ationGatewaySslPredefinedPolicy/AppGwSslPolicy20170401
    /subscriptions/147a22e9-2356-4e56-b3de-1f5842ae4a3b/resourceGroups//providers/Microsoft.Network/ApplicationGatewayAvailableSslOptions/default/Applic
ationGatewaySslPredefinedPolicy/AppGwSslPolicy20170401S

AvailableCipherSuites:
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_DHE_RSA_WITH_AES_256_CBC_SHA
    TLS_DHE_RSA_WITH_AES_128_CBC_SHA
    TLS_RSA_WITH_AES_256_GCM_SHA384
    TLS_RSA_WITH_AES_128_GCM_SHA256
    TLS_RSA_WITH_AES_256_CBC_SHA256
    TLS_RSA_WITH_AES_128_CBC_SHA256
    TLS_RSA_WITH_AES_256_CBC_SHA
    TLS_RSA_WITH_AES_128_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    TLS_DHE_DSS_WITH_AES_256_CBC_SHA256
    TLS_DHE_DSS_WITH_AES_128_CBC_SHA256
    TLS_DHE_DSS_WITH_AES_256_CBC_SHA
    TLS_DHE_DSS_WITH_AES_128_CBC_SHA
    TLS_RSA_WITH_3DES_EDE_CBC_SHA
    TLS_DHE_DSS_WITH_3DES_EDE_CBC_SHA

AvailableProtocols:
    TLSv1_0
    TLSv1_1
    TLSv1_2
```

## <a name="list-pre-defined-ssl-policies"></a>列出預先定義的 SSL 原則

應用程式閘道隨附三個可用的預先定義原則。 `Get-AzureRmApplicationGatewaySslPredefinedPolicy` Cmdlet 會擷取這些原則。 每個原則都有不同的通訊協定版本與已啟用的加密套件。 這些預先定義的原則可以用來快速地在應用程式閘道上設定 SSL 原則。 根據預設，如果未定義特定的 SSL 原則，則會選取 **AppGwSslPolicy20150501**。

以下輸出是執行 `Get-AzureRmApplicationGatewaySslPredefinedPolicy` 的範例。

```
Name: AppGwSslPolicy20150501
MinProtocolVersion: TLSv1_0
CipherSuites:
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
    TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
    TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
    TLS_DHE_RSA_WITH_AES_256_CBC_SHA
    TLS_DHE_RSA_WITH_AES_128_CBC_SHA
    TLS_RSA_WITH_AES_256_GCM_SHA384
 ...
Name: AppGwSslPolicy20170401
MinProtocolVersion: TLSv1_1
CipherSuites:
    TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA
    TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
...
```

## <a name="configure-a-custom-ssl-policy"></a>設定自訂 SSL 原則

設定自訂 SSL 原則時，您會傳遞下列參數：PolicyType、MinProtocolVersion、CipherSuite 及 ApplicationGateway。 如果您嘗試傳遞其他參數，會在建立或更新應用程式閘道時發生錯誤。 

下列範例會在應用程式閘道上設定自訂 SSL 原則。 它會設定 `TLSv1_1` 的最小通訊協定版本，並且啟用下列加密套件：

* TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
* TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256

> [!IMPORTANT]
> 設定自訂 SSL 原則時，必須從下列清單選取至少一個加密套件。 應用程式閘道使用 RSA SHA256 加密套件進行後端管理。
> * TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 
> * TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
> * TLS_DHE_RSA_WITH_AES_128_GCM_SHA256
> * TLS_RSA_WITH_AES_128_GCM_SHA256
> * TLS_RSA_WITH_AES_256_CBC_SHA256
> * TLS_RSA_WITH_AES_128_CBC_SHA256

```powershell
# get an application gateway resource
$gw = Get-AzureRmApplicationGateway -Name AdatumAppGateway -ResourceGroup AdatumAppGatewayRG

# set the SSL policy on the application gateway
Set-AzureRmApplicationGatewaySslPolicy -ApplicationGateway $gw -PolicyType Custom -MinProtocolVersion TLSv1_1 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256"

# validate the SSL policy locally
Get-AzureRmApplicationGatewaySslPolicy -ApplicationGateway $gw

# update the gateway with validated SSL policy
Set-AzureRmApplicationGateway -ApplicationGateway $gw
```

## <a name="create-an-application-gateway-with-a-pre-defined-ssl-policy"></a>使用預先定義的 SSL 原則建立應用程式閘道

設定預先定義的 SSL 原則時，您會傳遞下列參數：PolicyType、PolicyName 及 ApplicationGateway。 如果您嘗試傳遞其他參數，會在建立或更新應用程式閘道時發生錯誤。

下列範例會使用預先定義的 SSL 原則建立新的應用程式閘道。

```powershell
# Create a resource group
$rg = New-AzureRmResourceGroup -Name ContosoRG -Location "East US"

# Create a subnet for the application gateway
$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name subnet01 -AddressPrefix 10.0.0.0/24

# Create a virtual network with a 10.0.0.0/16 address space
$vnet = New-AzureRmVirtualNetwork -Name appgwvnet -ResourceGroupName $rg.ResourceGroupName -Location "East US" -AddressPrefix 10.0.0.0/16 -Subnet $subnet

# Retrieve the subnet object for later use
$subnet = $vnet.Subnets[0]

# Create a public IP address
$publicip = New-AzureRmPublicIpAddress -ResourceGroupName $rg.ResourceGroupName -name publicIP01 -location "East US" -AllocationMethod Dynamic

# Create a ip configuration object
$gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name gatewayIP01 -Subnet $subnet

# Create a backend pool for backend web servers
$pool = New-AzureRmApplicationGatewayBackendAddressPool -Name pool01 -BackendIPAddresses 134.170.185.46, 134.170.188.221,134.170.185.50

# Define the backend http settings to be used.
$poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name poolsetting01 -Port 80 -Protocol Http -CookieBasedAffinity Enabled

# Create a new port for SSL
$fp = New-AzureRmApplicationGatewayFrontendPort -Name frontendport01  -Port 443

# Upload an existing pfx certificate for SSL offload
$password = ConvertTo-SecureString -String "P@ssw0rd" -AsPlainText -Force
$cert = New-AzureRmApplicationGatewaySslCertificate -Name cert01 -CertificateFile C:\folder\contoso.pfx -Password $password

# Create a frontend IP configuration for the public IP address
$fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name fipconfig01 -PublicIPAddress $publicip

# Create a new listener with the certificate, port, and frontend ip.
$listener = New-AzureRmApplicationGatewayHttpListener -Name listener01  -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SslCertificate $cert

# Create a new rule for backend traffic routing
$rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name rule01 -RuleType Basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool

# Define the size of the application gateway
$sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2

# Configure the SSL policy to use a different pre-defined policy
$policy = New-AzureRmApplicationGatewaySslPolicy -PolicyType Predefined -PolicyName AppGwSslPolicy20170401S

# Create the application gateway.
$appgw = New-AzureRmApplicationGateway -Name appgwtest -ResourceGroupName $rg.ResourceGroupName -Location "East US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig  -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SslCertificates $cert -SslPolicy $policy
```

## <a name="update-an-existing-application-gateway-with-a-pre-defined-ssl-policy"></a>使用預先定義的 SSL 原則更新現有應用程式閘道

若要自訂 SSL 原則，請傳遞下列參數：**PolicyType**、**MinProtocolVersion**、**CipherSuite** 及 **ApplicationGateway**。 若要設定預先定義的 SSL 原則，請傳遞下列參數：**PolicyType**、**PolicyName** 及 **ApplicationGateway**。 如果您嘗試傳遞其他參數，會在建立或更新應用程式閘道時發生錯誤。

在下列範例中，有自訂原則和預先定義原則兩者的程式碼範例。 對您想要使用的原則取消註解。

```powershell
# You have to change these parameters to match your environment.
$AppGWname = "YourAppGwName"
$RG = "YourResourceGroupName"

$AppGw = get-azurermapplicationgateway -Name $AppGWname -ResourceGroupName $RG

# Choose either custom policy or prefedined policy and uncomment the one you want to use.

# SSL Custom Policy
# Set-AzureRmApplicationGatewaySslPolicy -PolicyType Custom -MinProtocolVersion TLSv1_2 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_RSA_WITH_AES_128_CBC_SHA256" -ApplicationGateway $AppGw

# SSL Predefined Policy
# Set-AzureRmApplicationGatewaySslPolicy -PolicyType Predefined -PolicyName "AppGwSslPolicy20170401S" -ApplicationGateway $AppGW

# Update AppGW
# The SSL policy options are not validated or updated on the Application Gateway until this cmdlet is executed.
$SetGW = Set-AzureRmApplicationGateway -ApplicationGateway $AppGW
```

## <a name="next-steps"></a>後續步驟

請參閱[應用程式閘道重新導向概觀](application-gateway-redirect-overview.md)以了解如何將 HTTP 流量重新導向至 HTTPS 端點。
