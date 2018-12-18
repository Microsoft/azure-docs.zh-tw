---
title: 疑難排解 Azure 應用程式閘道閘道不正確 (502) 的錯誤 | Microsoft Docs
description: 了解如何對應用程式閘道 502 錯誤進行疑難排解
services: application-gateway
documentationcenter: na
author: amitsriva
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: 1d431ead-d318-47d8-b3ad-9c69f7e08813
ms.service: application-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2017
ms.author: amsriva
ms.openlocfilehash: 4eca6a588d2c95189f0ba995b8db195907e9dc39
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34356030"
---
# <a name="troubleshooting-bad-gateway-errors-in-application-gateway"></a>疑難排解應用程式閘道中閘道不正確的錯誤

了解使用應用程式閘道時如何針對收到的閘道不正確 (502) 錯誤進行疑難排解。

## <a name="overview"></a>概觀

設定應用程式閘道之後，使用者可能遇到的其中一個錯誤是「伺服器錯誤︰502 - 網頁伺服器作為閘道器或 Proxy 伺服器時收到無效的回應」。 此錯誤可能是由於下列主要原因所導致：

* NSG、UDR 或自訂 DNS 封鎖對後端集區成員的存取。
* 虛擬機器擴展集的後端虛擬機器或執行個體[未回應預設健康情況探查](#problems-with-default-health-probe.md)。
* 無效或不適當的[自訂健康情況探查設定](#problems-with-custom-health-probe.md)。
* Azure 應用程式閘道的[後端集區未設定或空白](#empty-backendaddresspool)。
* [虛擬機器擴展集的虛擬機器或執行個體都不是良好健康情況](#unhealthy-instances-in-backendaddresspool)。
* 使用者要求發生[要求逾期或連線能力問題](#request-time-out)。

## <a name="network-security-group-user-defined-route-or-custom-dns-issue"></a>網路安全性群組、使用者定義的路由或自訂 DNS 問題

### <a name="cause"></a>原因

如果因為有 NSG、UDR 或自訂 DNS 而封鎖對後端的存取，應用程式閘道執行個體將無法觸達後端集區，還會導致探查失敗而造成 502 錯誤。 請注意，NSG/UDR 可能出現在應用程式閘道子網路，或部署應用程式虛擬機器的子網路中。 同樣地，如果 FQDN 用於後端集區成員，而且使用者為 VNET 設定的 DNS 伺服器未正確解析 FQDN，則 VNET 中出現自訂 DNS 可能也會造成問題。

### <a name="solution"></a>解決方法

透過下列步驟來驗證 NSG、UDR 和 DNS 設定：
* 檢查與應用程式閘道子網路相關聯的 NSG。 確定未封鎖對後端的通訊。
* 檢查與應用程式閘道子網路相關聯的 UDR。 確定 UDR 沒有讓流量離開後端子網路 - 例如，檢查到達網路虛擬設備的路由，或透過 ExpressRoute/VPN 公告至應用程式閘道的預設路由。

```powershell
$vnet = Get-AzureRmVirtualNetwork -Name vnetName -ResourceGroupName rgName
Get-AzureRmVirtualNetworkSubnetConfig -Name appGwSubnet -VirtualNetwork $vnet
```

* 檢查有效的 NSG 和後端虛擬機器的路由

```powershell
Get-AzureRmEffectiveNetworkSecurityGroup -NetworkInterfaceName nic1 -ResourceGroupName testrg
Get-AzureRmEffectiveRouteTable -NetworkInterfaceName nic1 -ResourceGroupName testrg
```

* 檢查 VNet 中存在自訂 DNS。 您可以在輸出中查看 VNet 屬性的詳細資料來檢查 DNS。

```json
Get-AzureRmVirtualNetwork -Name vnetName -ResourceGroupName rgName 
DhcpOptions            : {
                           "DnsServers": [
                             "x.x.x.x"
                           ]
                         }
```
如果 DNS 伺服器存在，請確定 DNS 伺服器能夠正確解析後端集區成員的 FQDN。

## <a name="problems-with-default-health-probe"></a>預設健全狀況探查的問題

### <a name="cause"></a>原因

502 錯誤也可以是預設健全狀態探查無法連線到後端 VM 的常用指標。 佈建應用程式閘道執行個體時，它會使用 BackendHttpSetting 的屬性，自動將預設的健全狀況探查設定到每個 BackendAddressPool 。 設定此探查時不需任何使用者輸入。 具體而言，設定負載平衡規則時，會在 BackendHttpSetting 和 BackendAddressPool 之間建立關聯。 預設探查是針對這其中的每一個關聯所設定，而應用程式閘道會在 BackendHttpSetting 項目中指定的連接埠上，將定期的健全狀況檢查連線初始到 BackendAddressPool 中的每一個執行個體。 下表列出與預設健全狀況探查相關聯的值。

| 探查屬性 | 值 | 說明 |
| --- | --- | --- |
| 探查 URL |http://127.0.0.1/ |URL 路徑 |
| 間隔 |30 |探查間隔 (秒) |
| 逾時 |30 |探查逾時 (秒) |
| 狀況不良臨界值 |3 |探查重試計數。 連續探查失敗計數到達狀況不良臨界值後，就會將後端伺服器標示為故障。 |

### <a name="solution"></a>解決方法

* 確定預設網站已設定且正於 127.0.0.1 上進行接聽。
* 如果 BackendHttpSetting 指定了 80 以外的連接埠，則應將預設網站設定為在該連接埠上進行接聽。
* 對 http://127.0.0.1:port 的呼叫應該會傳回 HTTP 結果碼 200。 這應該會在 30 秒逾時期間內傳回。
* 確定設定的連接埠已開啟，而且沒有任何防火牆或 Azure 網路安全性群組會在所設定的連接埠上封鎖連入或連出流量。
* 如果Azure 傳統 VM 或雲端服務會與 FQDN 或公用 IP 搭配使用，請確認對應的 [端點](../virtual-machines/windows/classic/setup-endpoints.md?toc=%2fazure%2fapplication-gateway%2ftoc.json) 已開啟。
* 如果 VM 是透過 Azure Resource Manager 所設定且位於應用程式閘道部署所在的 VNet 外部，就必須將 [網路安全性群組](../virtual-network/security-overview.md) 設定為允許在所需的連接埠上進行存取。

## <a name="problems-with-custom-health-probe"></a>自訂健全狀況探查的問題

### <a name="cause"></a>原因

自訂的健全狀況探查能夠對於預設探查行為提供更多彈性。 使用自訂探查時，使用者可以設定探查間隔、要測試的 URL 和路徑，以及在將後端集區執行個體標示為狀況不良前，可接受的失敗回應次數。 已新增下列其他屬性。

| 探查屬性 | 說明 |
| --- | --- |
| Name |探查的名稱。 此名稱用來在後端 HTTP 設定中指出探查。 |
| 通訊協定 |用來傳送探查的通訊協定。 探查會使用後端 HTTP 設定中定義的通訊協定 |
| Host |用來傳送探查的主機名稱。 只有當應用程式閘道上設定多站台時適用。 這與 VM 主機名稱不同。 |
| Path |探查的相對路徑。 有效路徑的開頭為 '/'。 探查會傳送到 \<通訊協定\>://\<主機\>:\<連接埠\>\<路徑\> |
| 間隔 |探查間隔 (秒)。 這是兩個連續探查之間的時間間隔。 |
| 逾時 |探查逾時 (秒)。 如果在這個逾時期間內未收到有效的回應，則會將探查標示為失敗。 |
| 狀況不良臨界值 |探查重試計數。 連續探查失敗計數到達狀況不良臨界值後，就會將後端伺服器標示為故障。 |

### <a name="solution"></a>解決方法

確認已按照上述資料表正確設定 [自訂健全狀態探查]。 除了上述的疑難排解步驟，也請確定下列各項：

* 確定已根據 [指南](application-gateway-create-probe-ps.md)正確指定探查。
* 如果已將應用程式閘道設定為單一站台，根據預設，除非已在自訂探查中加以設定，否則應將主機名稱指定為 '127.0.0.1'。
* 確定對 http://\<主機\>:\<連接埠\>\<路徑\> 的呼叫會傳回 HTTP 結果碼 200。
* 確定 Interval、Time-out 和 UnhealtyThreshold 皆在可接受的範圍內。
* 若使用 HTTPS 探查，請在後端伺服器本身設定後援憑證，以確定後端伺服器不會要求您提供 SNI。

## <a name="request-time-out"></a>要求逾時

### <a name="cause"></a>原因

收到使用者要求時，應用程式閘道會將設定的規則套用到該要求，並將其路由傳送到後端集區執行個體。 其會等候一段可設定的時間間隔以接收來自後端應用程式的回應。 根據預設，這個間隔為 **30 秒**。 如果應用程式閘道沒有在此時間間隔內收到來自後端應用程式的回應，使用者要求就會看到 502 錯誤。

### <a name="solution"></a>解決方法

應用程式閘道允許使用者透過 BackendHttpSetting 設定此組態，接著可將之套用到其他集區。 不同的後端集區可以有不同的 BackendHttpSetting，因此可設定不同的要求逾時。

```powershell
    New-AzureRmApplicationGatewayBackendHttpSettings -Name 'Setting01' -Port 80 -Protocol Http -CookieBasedAffinity Enabled -RequestTimeout 60
```

## <a name="empty-backendaddresspool"></a>空白的 BackendAddressPool

### <a name="cause"></a>原因

如果應用程式閘道在後端位址集區中未設定虛擬機器或虛擬機器擴展集，則無法路由傳送任何客戶要求，還會擲回閘道不正確的錯誤。

### <a name="solution"></a>解決方法

確定後端位址集區不是空白的。 這可透過 PowerShell、CLI 或入口網站來完成。

```powershell
Get-AzureRmApplicationGateway -Name "SampleGateway" -ResourceGroupName "ExampleResourceGroup"
```

前述 Cmdlet 的輸出應包含非空白的後端位址集區。 下列範例會傳回兩個針對後端 VM 使用 FQDN 或 IP 位址設定的集區。 BackendAddressPool 的佈建狀態必須是 'Succeeded'。

BackendAddressPoolsText：

```json
[{
    "BackendAddresses": [{
        "ipAddress": "10.0.0.10",
        "ipAddress": "10.0.0.11"
    }],
    "BackendIpConfigurations": [],
    "ProvisioningState": "Succeeded",
    "Name": "Pool01",
    "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
    "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool01"
}, {
    "BackendAddresses": [{
        "Fqdn": "xyx.cloudapp.net",
        "Fqdn": "abc.cloudapp.net"
    }],
    "BackendIpConfigurations": [],
    "ProvisioningState": "Succeeded",
    "Name": "Pool02",
    "Etag": "W/\"00000000-0000-0000-0000-000000000000\"",
    "Id": "/subscriptions/<subscription id>/resourceGroups/<resource group name>/providers/Microsoft.Network/applicationGateways/<application gateway name>/backendAddressPools/pool02"
}]
```

## <a name="unhealthy-instances-in-backendaddresspool"></a>BackendAddressPool 中狀況不良的執行個體

### <a name="cause"></a>原因

如果 BackendAddressPool 的所有執行個體都狀況不良，則應用程式閘道不會包含任何要將使用者要求路由傳送到其中的後端。 當後端執行個體的狀況良好，但尚未部署必要的應用程式時，也可能發生此情況。

### <a name="solution"></a>解決方法

確定執行個體的狀況良好且已正確設定應用程式。 檢查後端執行個體是否能夠從同一個 VNet 中的其他 VM 回應 Ping。 如果是使用公用端點所設定，請確定對 Web 應用程式的瀏覽器要求能夠提供服務。

## <a name="next-steps"></a>後續步驟

如果上述步驟無法解決問題，請開啟 [支援票證](https://azure.microsoft.com/support/options/)。

