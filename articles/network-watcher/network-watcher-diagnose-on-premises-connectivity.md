---
title: 使用 Azure 網路監看員診斷透過 VPN 閘道的內部部署連線 | Microsoft Docs
description: 本文說明如何使用 Azure 網路監看員資源疑難排解，來診斷透過 VPN 閘道的內部部署連線。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: aeffbf3d-fd19-4d61-831d-a7114f7534f9
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: e51d31035a8b05238ef0f8d13dd6b6c3f9ad02e8
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/08/2017
ms.locfileid: "26374197"
---
# <a name="diagnose-on-premises-connectivity-via-vpn-gateways"></a>透過 VPN 閘道診斷內部部署連線

Azure VPN 閘道可讓您建立混合式解決方案，以解決內部部署網路與 Azure 虛擬網路之間的安全連線需求。 由於這項需求很獨特，所以選擇的內部部署 VPN 裝置也很獨特。 Azure 目前支援與裝置廠商合作來持續驗證的[數個 VPN 裝置](../vpn-gateway/vpn-gateway-about-vpn-devices.md#devicetable)。 請先檢閱裝置特定的組態設定，再設定內部部署 VPN 裝置。 同樣地，Azure VPN 閘道也使用一組用於建立連線的[受支援 IPsec 參數](../vpn-gateway/vpn-gateway-about-vpn-devices.md#ipsec)來進行設定。 目前您無法指定或選取來自 Azure VPN 閘道之 IPsec 參數的特定組合。 若要在內部部署環境與 Azure 之間成功建立連線，內部部署 VPN 裝置設定必須符合 Azure VPN 閘道所規定的 IPsec 參數。 如果設定不正確，連線會中斷，而且到目前為止，針對這些問題進行疑難排解並不容易，而且通常需要好幾個小時才能識別並修正問題。

使用 Azure 網路監看員疑難排解功能後，您將能夠診斷閘道和連線的任何問題，並在幾分鐘內獲得足夠資訊來做出明智的問題改正決定。

## <a name="scenario"></a>案例

您想要使用 FortiGate 作為內部部署 VPN 閘道，在 Azure 和內部部署環境之間設定站對站連線。 為了實現此案例，您需要進行下列設定︰

1. 虛擬網路閘道 - Azure 上的 VPN 閘道
1. 本機網路閘道 - [內部部署 (FortiGate) VPN 閘道](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#LocalNetworkGateway)在 Azure 雲端的表示
1. 站對站連線 (路由式) - [VPN 閘道與內部部署路由器之間的連線](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#createconnection)
1. [設定 FortiGate](https://github.com/Azure/Azure-vpn-config-samples/blob/master/Fortinet/Current/Site-to-Site_VPN_using_FortiGate.md)

若要找到用於設定站對站組態的詳細逐步解說指南，請瀏覽︰[使用 Azure 入口網站建立具有網站間連線的 VNet](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)。

其中一個重要組態步驟是設定 IPsec 通訊參數，設定若有任何錯誤將會導致內部部署網路與 Azure 之間的連線中斷。 目前 Azure VPN 閘道會設定為支援下列第 1 階段 IPsec 參數。 請注意，如先前所述，您無法修改這些設定。  如下表所示，Azure VPN 閘道支援的加密演算法有 AES256、AES128 及 3DES。

### <a name="ike-phase-1-setup"></a>IKE 第 1 階段設定

| **屬性** | **原則式** | **路由式和標準或高效能 VPN 閘道** |
| --- | --- | --- |
| IKE 版本 |IKEv1 |IKEv2 |
| Diffie-Hellman 群組 |群組 2 (1024 位元) |群組 2 (1024 位元) |
| 驗證方法 |預先共用金鑰 |預先共用金鑰 |
| 加密演算法 |AES256 AES128 3DES |AES256 3DES |
| 雜湊演算法 |SHA1(SHA128) |SHA1(SHA128)、SHA2(SHA256) |
| 階段 1 安全性關聯 (SA) 存留期 (時間) |28,800 秒 |10,800 秒 |

身為使用者，您必須設定 FortiGate，範例設定可於 [GitHub](https://github.com/Azure/Azure-vpn-config-samples/blob/master/Fortinet/Current/fortigate_show%20full-configuration.txt) 找到。 在不知不覺中，您已將 FortiGate 設定為使用 SHA-512 作為雜湊演算法。 此演算法並非原則式連線所支援的演算法，因此您的 VPN 連線會運作。

這些問題很難進行疑難排解，而且根本原因通常很難一下就想到。 在此情況下，您可以開啟支援票證，來獲得協助以解決問題。 但有了 Azure 網路監看員疑難排解 API 後，您可以自行識別這些問題。

## <a name="troubleshooting-using-azure-network-watcher"></a>使用 Azure 網路監看員進行疑難排解

若要診斷連線，請連線到 Azure PowerShell，並起始 `Start-AzureRmNetworkWatcherResourceTroubleshooting` Cmdlet。 您可以在[針對虛擬網路閘道和連線進行疑難排解 - PowerShell](network-watcher-troubleshoot-manage-powershell.md)中找到有關使用這個 Cmdlet 的詳細資料。 這個 Cmdlet 最多可能需要幾分鐘的時間才能完成。

此 Cmdlet 完成後，您可以瀏覽至 Cmdlet 指定的儲存體位置，以取得關於問題的詳細資訊和記錄。 Azure 網路監看員會建立包含下列記錄檔的 zip 資料夾︰

![1][1]

開啟名為 IKEErrors.txt 的檔案，它會顯示下列錯誤，指出內部部署 IKE 設定不正確的問題。

```
Error: On-premises device rejected Quick Mode settings. Check values.
     based on log : Peer sent NO_PROPOSAL_CHOSEN notify
```

您可以從 Scrubbed-wfpdiag.txt 取得關於此錯誤的詳細資訊，如本案例所示，其提到有 `ERROR_IPSEC_IKE_POLICY_MATCH` 而導致連線無法正常運作。

另一個常見的錯誤組態是指定了不正確的共用金鑰。 如果您在上述範例中指定了不同的共用金鑰，IKEErrors.txt 會顯示下列錯誤︰`Error: Authentication failed. Check shared key`。

Azure 網路監看員疑難排解功能可讓您輕鬆地利用簡單的 PowerShell Cmdlet，來針對 VPN 閘道和連線進行診斷和疑難排解。 目前我們支援診斷下列狀況，並正在努力新增更多狀況。

### <a name="gateway"></a>閘道器

| 錯誤類型 | 原因 | 記錄檔|
|---|---|---|
| NoFault | 未偵測到任何錯誤時。 |yes|
| GatewayNotFound | 找不到閘道或閘道尚未佈建。 |否|
| PlannedMaintenance |  閘道執行個體正在進行維護。  |否|
| UserDrivenUpdate | 當正在更新使用者時。 這可能是調整大小作業。 | 否 |
| VipUnResponsive | 無法連線到閘道的主要執行個體。 健全狀況探查失敗時便會發生這種狀況。 | 否 |
| PlatformInActive | 平台發生問題。 | 否|
| ServiceNotRunning | 基礎服務並未執行。 | 否|
| NoConnectionsFoundForGateway | 閘道上沒有任何連線存在。 這只是警告。| 否|
| ConnectionsNotConnected | 未進行任何連線。 這只是警告。| yes|
| GatewayCPUUsageExceeded | 目前的閘道使用量 CPU 使用量 > 95%。 | yes |

### <a name="connection"></a>連線

| 錯誤類型 | 原因 | 記錄檔|
|---|---|---|
| NoFault | 未偵測到任何錯誤時。 |yes|
| GatewayNotFound | 找不到閘道或閘道尚未佈建。 |否|
| PlannedMaintenance | 閘道執行個體正在進行維護。  |否|
| UserDrivenUpdate | 當正在更新使用者時。 這可能是調整大小作業。  | 否 |
| VipUnResponsive | 無法連線到閘道的主要執行個體。 健全狀況探查失敗時便會發生這種狀況。 | 否 |
| ConnectionEntityNotFound | 缺少連線組態。 | 否 |
| ConnectionIsMarkedDisconnected | 連線標記為「已中斷連線」。 |否|
| ConnectionNotConfiguredOnGateway | 基礎服務未設定連線。 | yes |
| ConnectionMarkedStandy | 基礎服務標記為「待命」。| yes|
| 驗證 | 預先共用的金鑰不相符。 | yes|
| PeerReachability | 無法連線到對等閘道。 | yes|
| IkePolicyMismatch | 對等閘道的 IKE 原則不受 Azure 支援。 | yes|
| WfpParse 錯誤 | 剖析 WFP 記錄時發生錯誤。 |yes|

## <a name="next-steps"></a>後續步驟

瀏覽[使用 Azure 網路監看員疑難排解來監視 VPN 閘道](network-watcher-monitor-with-azure-automation.md)，以了解如何利用 PowerShell 和 Azure 自動化來檢查 VPN 閘道連線

[1]: ./media/network-watcher-diagnose-on-premises-connectivity/figure1.png
