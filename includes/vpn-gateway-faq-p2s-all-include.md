---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 07/30/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 3a4e45aa2c1aca461ef32ebec2cfb50a470037ed
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47060442"
---
### <a name="how-many-vpn-client-endpoints-can-i-have-in-my-point-to-site-configuration"></a>在我的點對站台組態中可以有多少個 VPN 用戶端端點？

我們最多支援 128 個 VPN 用戶端可以同時連線到虛擬網路。

### <a name="supportedclientos"></a>可以使用哪些用戶端作業系統來搭配點對站？

以下為支援的用戶端作業系統：

* Windows 7 (32 位元和 64 位元)
* Windows Server 2008 R2 (僅限 64 位元)
* Windows 8.1 (32 位元和 64 位元)
* Windows Server 2012 (僅限 64 位元)
* Windows Server 2012 R2 (僅限 64 位元)
* Windows Server 2016 (僅限 64 位元)
* Windows 10
* Mac OS X 10.11 版或更新版本
* Linux (StrongSwan)
* iOS

[!INCLUDE [TLS](vpn-gateway-tls-updates.md)]

### <a name="can-i-traverse-proxies-and-firewalls-using-point-to-site-capability"></a>是否可以使用點對站台功能周遊 Proxy 和防火牆？

Azure 支援兩種點對站 VPN 選項：

* 安全通訊端通道通訊協定 (SSTP)。 SSTP 是 Microsoft 專屬的 SSL 型解決方案，因為大部分的防火牆都會開啟 443 SSL 所使用的 TCP 連接埠，所以 SSTP 可以穿透防火牆。

* IKEv2 VPN。 IKEv2 VPN 是標準型 IPsec VPN 解決方案，會使用 UDP 連接埠 500 和 4500 以及 第 50 號的 IP 通訊協定。 防火牆不一定會開啟這些連接埠，因此 IKEv2 VPN 有可能無法周遊 Proxy 和防火牆。

### <a name="if-i-restart-a-client-computer-configured-for-point-to-site-will-the-vpn-automatically-reconnect"></a>如果我重新啟動針對點對站台設定的用戶端電腦，VPN 將自動重新連線嗎？

用戶端電腦預設為不會自動重新建立 VPN 連線。

### <a name="does-point-to-site-support-auto-reconnect-and-ddns-on-the-vpn-clients"></a>在 VPN 用戶端上點對站台支援自動重新連接和 DDNS 嗎？

點對站台 VPN 目前不支援自動重新連接和 DDNS。

### <a name="can-i-have-site-to-site-and-point-to-site-configurations-coexist-for-the-same-virtual-network"></a>對於相同的虛擬網路，網站間和點對站台組態是否可以同時存在？

是。 如果是 Resource Manager 部署模型，您的閘道必須是路由式 VPN 類型。 如果是傳統部署模型，則需要動態閘道。 靜態路由 VPN 閘道或原則式 VPN 閘道不支援點對站。

### <a name="can-i-configure-a-point-to-site-client-to-connect-to-multiple-virtual-networks-at-the-same-time"></a>是否可以將點對站台用戶端設定為同時連接到多個虛擬網路？

否。 點對站用戶端只能連線到虛擬網路閘道所在的 VNet 中的資源。

### <a name="how-much-throughput-can-i-expect-through-site-to-site-or-point-to-site-connections"></a>透過網站間或點對站台連線可以獲得多少輸送量？

很難維護 VPN 通道的確切輸送量。 IPsec 和 SSTP 為加密嚴謹的 VPN 通訊協定。 輸送量也會受限於內部部署與網際網路之間的延遲和頻寬。 對於只有 IKEv2 點對站 VPN 連線的 VPN 閘道，您可以預期的總輸送量取決於閘道 SKU。 如需輸送量的詳細資訊，請參閱[閘道 SKU](../articles/vpn-gateway/vpn-gateway-about-vpngateways.md#gwsku)。

### <a name="can-i-use-any-software-vpn-client-for-point-to-site-that-supports-sstp-andor-ikev2"></a>是否可以對支援 SSTP 和 (或) IKEv2 的點對站使用任何軟體 VPN 用戶端？

否。 在 Windows 上，您只能對 SSTP 使用原生 VPN 用戶端，而在 Mac 上，則只能對 IKEv2 使用原生 VPN 用戶端。 請參閱支援的用戶端作業系統清單。

### <a name="does-azure-support-ikev2-vpn-with-windows"></a>Azure 支援採用 Windows 的 IKEv2 VPN 嗎？

Windows 10 和 Server 2016 都支援 IKEv2。 不過，若要使用 IKEv2，您必須在本機安裝更新並設定登錄機碼值。 Windows 10 之前的作業系統版本不受支援，且只能使用 SSTP。

若要針對 IKEv2 準備 Windows 10 或 Server 2016：

1. 安裝更新。

  | 作業系統版本 | 日期 | 號碼/連結 |
  |---|---|---|---|
  | Windows Server 2016<br>Windows 10 版本 1607 | 2018 年 1 月 17 日 | [KB4057142](https://support.microsoft.com/help/4057142/windows-10-update-kb4057142) |
  | Windows 10 版本 1703 | 2018 年 1 月 17 日 | [KB4057144](https://support.microsoft.com/help/4057144/windows-10-update-kb4057144) |
  |  |  |  |  |

2. 設定登錄機碼值。 在登入中建立 “HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\ IKEv2\DisableCertReqPayload” REG_DWORD 機碼或將其設定為 1。

### <a name="what-happens-when-i-configure-both-sstp-and-ikev2-for-p2s-vpn-connections"></a>當我設定 SSTP 和 IKEv2 以便進行 P2S VPN 連線時，會發生什麼狀況？

當您在混合環境 (包含 Windows 和 Mac 裝置) 中設定 SSTP 和 IKEv2 時，Windows VPN 用戶端一律會先嘗試 IKEv2 通道，但如果 IKEv2 連線失敗，則會回復為 SSTP。 MacOSX 只可透過 IKEv2 連線。

### <a name="other-than-windows-and-mac-which-other-platforms-does-azure-support-for-p2s-vpn"></a>除了 Windows 和 Mac 以外，Azure 還支援哪些其他平台使用 P2S VPN？

Azure 支援適用於 P2S VPN 的 Windows、Mac 和 Linux。

### <a name="i-already-have-an-azure-vpn-gateway-deployed-can-i-enable-radius-andor-ikev2-vpn-on-it"></a>我已經部署 Azure VPN 閘道。 可以在其上啟用 RADIUS 及/或 IKEv2 VPN 嗎？

可以，您可以使用 Powershell 或 Azure 入口網站，在已經部署的閘道上啟用這些新功能，但前提是您使用的閘道 SKU 可支援 RADIUS 及/或 IKEv2。 例如，VPN 閘道基本 SKU 不支援 RADIUS 或 IKEv2。
