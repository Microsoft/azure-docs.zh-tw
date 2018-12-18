---
title: 以 Azure 應用程式閘道設定端對端 SSL
description: 本文說明如何使用 PowerShell 以 Azure 應用程式閘道設定端對端 SSL
services: application-gateway
documentationcenter: na
author: vhorne
manager: jpconnock
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 3/27/2018
ms.author: victorh
ms.openlocfilehash: 7e259936dce433683dd135171ee1c5626bf23739
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
---
# <a name="configure-end-to-end-ssl-by-using-application-gateway-with-powershell"></a>使用 PowerShell 以應用程式閘道設定端對端 SSL

## <a name="overview"></a>概觀

Azure 應用程式閘道支援為流量進行端對端加密。 應用程式閘道會在應用程式閘道終止 SSL 連線。 閘道接著會對流量套用路由規則、重新加密封包，並根據所定義的路由規則將封包轉送至適當的後端伺服器。 任何來自 Web 伺服器的回應都會經歷相同的程序而回到使用者端。

應用程式閘道支援定義自訂 SSL 選項。 它也支援停用下列通訊協定版本；**TLSv1.0**、**TLSv1.1** 和 **TLSv1.2**，也會定義要使用那個加密套件以及偏好的順序。 若要深入了解可設定的 SSL 選項，請參閱 [SSL 原則概觀](application-gateway-SSL-policy-overview.md)。

> [!NOTE]
> 預設會停用 SSL 2.0 和 SSL 3.0，並且無法啟用。 這些版本被認為並不安全，因此不能搭配應用程式閘道使用。

![案例影像][scenario]

## <a name="scenario"></a>案例

在此案例中，您將了解如何使用 PowerShell 來建立使用端對端 SSL 的應用程式閘道。

此案例將會：

* 建立名為 **appgw-rg** 的資源群組。
* 建立名為 **appgwvnet** 且具有 **10.0.0.0/16** 位址空間的虛擬網路。
* 建立名為 **appgwsubnet** 和 **appsubnet** 的兩個子網路。
* 建立支援端對端 SSL 加密 (限制 SSL 通訊協定版本和加密套件) 的小型應用程式閘道。

## <a name="before-you-begin"></a>開始之前

若要對應用程式閘道設定端對端 SSL，則需要閘道憑證和後端伺服器憑證。 閘道憑證可用來加密和解密透過 SSL 傳送過來的流量。 閘道憑證必須採用「個人資訊交換」(PFX) 格式。 此檔案格式可允許將私密金鑰匯出，而應用程式閘道需要這個匯出的金鑰來執行流量的加密和解密。

若要加密端對端 SSL，後端必須已加入到應用程式閘道的允許清單。 您需要將後端伺服器的公開憑證上傳至應用程式閘道。 新增憑證可確保應用程式閘道只會與已知的後端執行個體通訊。 這會進而保護端對端通訊。

下列各節將說明設定程序。

## <a name="create-the-resource-group"></a>建立資源群組

本節將引導您建立資源群組，其中包含應用程式閘道。


   1. 登入您的 Azure 帳戶。

   ```powershell
   Connect-AzureRmAccount
   ```


   2. 選取要用於此案例的訂用帳戶。

   ```powershell
   Select-AzureRmsubscription -SubscriptionName "<Subscription name>"
   ```


   3. 建立資源群組。 (如果您使用現有的資源群組，請略過此步驟。)

   ```powershell
   New-AzureRmResourceGroup -Name appgw-rg -Location "West US"
   ```

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>建立應用程式閘道的虛擬網路和子網路

下列範例會建立虛擬網路和兩個子網路。 一個子網路用來保留應用程式閘道。 另一個子網路則用於裝載 Web 應用程式的後端。


   1. 指派要用於應用程式閘道的子網路位址範圍。

   ```powershell
   $gwSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24
   ```

   > [!NOTE]
   > 應該適當地調整針對應用程式閘道所設定的子網路大小。 一個應用程式閘道最多可以針對 10 個執行個體進行設定。 每個執行個體會從子網路取得 1 個 IP 位址。 子網路太小可能會對相應放大應用程式閘道產生負面影響。
   > 
   > 

   2. 指派要用於後端位址集區的位址範圍。

   ```powershell
   $nicSubnet = New-AzureRmVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24
   ```

   3. 建立具有上述步驟所定義子網路的虛擬網路。

   ```powershell
   $vnet = New-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet
   ```

   4. 擷取要用於後續步驟的虛擬網路資源和子網路資源。

   ```powershell
   $vnet = Get-AzureRmvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
   $gwSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
   $nicSubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet
   ```

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>建立前端組態的公用 IP 位址

建立要用於應用程式閘道的公用 IP 資源。 此公用 IP 位址會用於下列其中一個後續步驟。

```powershell
$publicip = New-AzureRmPublicIpAddress -ResourceGroupName appgw-rg -Name 'publicIP01' -Location "West US" -AllocationMethod Dynamic
```

> [!IMPORTANT]
> 應用程式閘道不支援使用以定義的網域標籤建立的公用 IP 位址。 只支援具有動態建立之網域標籤的公用 IP 位址。 如果您需要讓應用程式閘道具有好記的 DNS 名稱，建議您使用 CNAME 記錄作為別名。

## <a name="create-an-application-gateway-configuration-object"></a>建立應用程式閘道組態物件

所有設定項目是在建立應用程式閘道之前設定。 下列步驟會建立應用程式閘道資源所需的組態項目。

   1. 建立應用程式閘道 IP 組態物件。 此設定會設定應用程式閘道使用的子網路。 當應用程式閘道啟動時，它會從已設定的子網路取得 IP 位址，然後將網路流量路由傳送到後端 IP 集區中的 IP 位址。 請記住，每個執行個體需要一個 IP 位址。

   ```powershell
   $gipconfig = New-AzureRmApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet
   ```


   2. 建立前端 IP 組態。 此設定會將私人或公用 IP 位址對應到應用程式閘道的前端。 下一個步驟會將先前步驟中的公用 IP 位址關聯到前端 IP 組態。

   ```powershell
   $fipconfig = New-AzureRmApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip
   ```

   3. 使用後端 Web 伺服器的 IP 位址設定後端 IP 位址集區。 這些 IP 位址會接收來自前端 IP 端點的網路流量。 以您自己的應用程式 IP 位址端點取代範例中的 IP 位址。

   ```powershell
   $pool = New-AzureRmApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3
   ```

   > [!NOTE]
   > 完整網域名稱 (FQDN) 也是可用來取代後端伺服器 IP 位址的有效值。 您可以使用 **-BackendFqdns** 參數啟用它。 


   4. 設定公用 IP 端點的前端 IP 連接埠。 此連接埠是供使用者連接到的連接埠。

   ```powershell
   $fp = New-AzureRmApplicationGatewayFrontendPort -Name 'port01'  -Port 443
   ```

   5. 設定應用程式閘道憑證。 此憑證可用來解密和重新加密應用程式閘道上的流量。

   ```powershell
   $passwd = ConvertTo-SecureString  <certificate file password> -AsPlainText -Force 
   $cert = New-AzureRmApplicationGatewaySSLCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password $passwd 
   ```

   > [!NOTE]
   > 此範例會設定 SSL 連接所使用的憑證。 憑證必須是 .pfx 格式，而密碼則必須介於 4 到 12 個字元。

   6. 建立應用程式閘道的 HTTP 接聽程式。 指派要使用的前端 IP 設定、連接埠和 SSL 憑證。

   ```powershell
   $listener = New-AzureRmApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SSLCertificate $cert
   ```

   7. 上傳要在已啟用 SSL 的後端集區資源上使用的憑證。

   > [!NOTE]
   > 預設探查可從後端 IP 位址上的*預設* SSL 繫結取得公開金鑰，並將所收到的公開金鑰值與您在此處提供的公開金鑰值做比較。 
   
   > 如果您在後端上使用主機標頭和伺服器名稱指示 (SNI)，擷取的公開金鑰不一定就是流量要流向的預定網站。 如果不能確定，請在後端伺服器造訪 https://127.0.0.1/ 以確認要將哪一個憑證用於*預設* SSL 繫結。 請使用來自本節該要求的公開金鑰。 如果您在 HTTPS 繫結上使用主機標頭和 SNI，且並未從對於後端伺服器上 https://127.0.0.1/ 的手動瀏覽器要求收到回應和憑證，您必須在後端伺服器上設定預設 SSL 繫結。 如果您未這麼做，探查將會失敗，而且後端不會加入允許清單。

   ```powershell
   $authcert = New-AzureRmApplicationGatewayAuthenticationCertificate -Name 'whitelistcert1' -CertificateFile C:\users\gwallace\Desktop\cert.cer
   ```

   > [!NOTE]
   > 在此步驟中所提供的憑證，應該是在後端上出現的 .pfx 憑證本身的公開金鑰。 將安裝在後端伺服器上的憑證 (非根憑證) 以「宣告、證據和推論」(CER) 格式匯出，並將它用於此步驟。 此步驟會將後端加入到應用程式閘道的白名單。

   8. 設定應用程式閘道的後端 HTTP 設定。 將前述步驟中上傳的憑證指派給 HTTP 設定。

   ```powershell
   $poolSetting = New-AzureRmApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert
   ```
   9. 建立會設定負載平衡器行為的負載平衡器路由規則。 在此範例中，會建立基本的循環配置資源規則。

   ```powershell
   $rule = New-AzureRmApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
   ```

   10. 設定應用程式閘道的執行個體大小。 可用大小是 **Standard\_Small**、**Standard\_Medium** 和 **Standard\_Large**。  容量的可用值為 **1** 到 **10**。

   ```powershell
   $sku = New-AzureRmApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
   ```

   > [!NOTE]
   > 若要進行測試，可以選擇執行個體計數 1。 請務必了解 SLA 不涵蓋任何低於兩個執行個體的執行個體計數，因此不建議使用。 小型閘道適用於開發測試，不適合在生產環境中使用。

   11. 設定要在應用程式閘道上使用的 SSL 原則。 應用程式閘道支援將 SSL 通訊協定版本設為最小版本的能力。

   下列值是可以定義的通訊協定版本清單：

    - **TLSV1_0**
    - **TLSV1_1**
    - **TLSV1_2**
    
   下列範例將最小通訊協定版本設定為 **TLSv1_2**，並且只啟用 **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256**, **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384** 和 **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256**。

   ```powershell
   $SSLPolicy = New-AzureRmApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256"
   ```

## <a name="create-the-application-gateway"></a>建立應用程式閘道

使用上述所有步驟建立應用程式閘道。 建立閘道的執行過程需要很長一段時間。

```powershell
$appgw = New-AzureRmApplicationGateway -Name appgateway -SSLCertificates $cert -ResourceGroupName "appgw-rg" -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SSLPolicy $SSLPolicy -AuthenticationCertificates $authcert -Verbose
```

## <a name="limit-ssl-protocol-versions-on-an-existing-application-gateway"></a>限制現有應用程式閘道上的 SSL 通訊協定版本

上述步驟會引導您建立具有端對端 SSL 的應用程式並停用特定的 SSL 通訊協定版本。 下列範例會停用現有應用程式閘道上的特定 SSL 原則。

   1. 擷取要更新的應用程式閘道。

   ```powershell
   $gw = Get-AzureRmApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```

   2. 定義 SSL 原則。 在下列範例中，會停用 **TLSv1.0** 和 **TLSv1.1**，且只有 **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256**, **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384**, and **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256** 是允許的加密套件。

   ```powershell
   Set-AzureRmApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -PolicyType Custom -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -ApplicationGateway $gw

   ```

   3. 最後，更新閘道。 請注意，此最後一個步驟是漫長的工作。 完成時，應用程式閘道上便已設定端對端 SSL。

   ```powershell
   $gw | Set-AzureRmApplicationGateway
   ```

## <a name="get-an-application-gateway-dns-name"></a>取得應用程式閘道 DNS 名稱

建立閘道之後，下一步是設定通訊的前端。 使用公用 IP 時，應用程式閘道需要動態指派的 DNS 名稱 (不易記住)。 為了確保使用者可以叫用應用程式閘道，可使用 CNAME 記錄來指向應用程式閘道的公用端點。 如需詳細資訊，請參閱[在 Azure 中設定自訂網域名稱](../cloud-services/cloud-services-custom-domain-name-portal.md)。 

若要設定別名，請使用連結至應用程式閘道的 **PublicIPAddress** 元素，擷取應用程式閘道的詳細資料及其關聯的 IP/DNS 名稱。 使用應用程式閘道的 DNS 名稱所建立的 CNAME 記錄，可將兩個 Web 應用程式指向此 DNS 名稱。 不建議使用 A-records，因為重新啟動應用程式閘道時，VIP 可能會變更。

```powershell
Get-AzureRmPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>後續步驟

如需透過應用程式閘道使用 Web 應用程式防火牆強化 Web 應用程式安全性的詳細資訊，請參閱 [Web 應用程式防火牆概觀](application-gateway-webapplicationfirewall-overview.md)。

[scenario]: ./media/application-gateway-end-to-end-SSL-powershell/scenario.png
