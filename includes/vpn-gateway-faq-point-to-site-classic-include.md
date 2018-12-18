---
title: 包含檔案
description: 包含檔案
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: b8a785293c4e555b547c4b002f5ef5906a7633c0
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30198174"
---
這個常見問題集適用於使用傳統部署模型的 P2S 連線。

### <a name="what-client-operating-systems-can-i-use-with-point-to-site"></a>可以使用哪些用戶端作業系統來搭配點對站台？

以下為支援的用戶端作業系統：

* Windows 7 (32 位元和 64 位元)
* Windows Server 2008 R2 (僅限 64 位元)
* Windows 8 (32 位元和 64 位元)
* Windows 8.1 (32 位元和 64 位元)
* Windows Server 2012 (僅限 64 位元)
* Windows Server 2012 R2 (僅限 64 位元)
* Windows 10

### <a name="can-i-use-any-software-vpn-client-for-point-to-site-that-supports-sstp"></a>是否可以對支援 SSTP 的點對站台使用任何軟體 VPN 用戶端？

編號 支援只限於上面所列的 Windows 作業系統版本。

### <a name="how-many-vpn-client-endpoints-can-i-have-in-my-point-to-site-configuration"></a>在我的點對站台組態中可以有多少個 VPN 用戶端端點？

我們最多支援 128 個 VPN 用戶端可以同時連線到虛擬網路。

### <a name="can-i-use-my-own-internal-pki-root-ca-for-point-to-site-connectivity"></a>是否可以使用自己的內部 PKI 根 CA 進行點對站台連線？

是。 先前只能使用自我簽署的根憑證。 您仍然可以上傳 20 個根憑證。

### <a name="can-i-traverse-proxies-and-firewalls-using-point-to-site-capability"></a>是否可以使用點對站台功能周遊 Proxy 和防火牆？

是。 我們使用 SSTP (安全通訊端通道通訊協定) 穿過防火牆。 此通道將會顯示為 HTTPs 連線。

### <a name="if-i-restart-a-client-computer-configured-for-point-to-site-will-the-vpn-automatically-reconnect"></a>如果我重新啟動針對點對站台設定的用戶端電腦，VPN 將自動重新連線嗎？

用戶端電腦預設為不會自動重新建立 VPN 連線。

### <a name="does-point-to-site-support-auto-reconnect-and-ddns-on-the-vpn-clients"></a>在 VPN 用戶端上點對站台支援自動重新連接和 DDNS 嗎？

點對站台 VPN 目前不支援自動重新連接和 DDNS。

### <a name="can-i-have-site-to-site-and-point-to-site-configurations-coexist-for-the-same-virtual-network"></a>對於相同的虛擬網路，網站間和點對站台組態是否可以同時存在？

是。 如果閘道是路由式 VPN 類型，這兩個解決方案都可正常運作。 如果是傳統部署模型，則需要動態閘道。 靜態路由 VPN 閘道或使用 `-VpnType PolicyBased` cmdlet 的閘道不支援點對站。

### <a name="can-i-configure-a-point-to-site-client-to-connect-to-multiple-virtual-networks-at-the-same-time"></a>是否可以將點對站台用戶端設定為同時連接到多個虛擬網路？

是，可以的。 但是虛擬網路不能有重疊的 IP 前置詞，而且點對站台位址空間在虛擬網路之間不得重疊。

### <a name="how-much-throughput-can-i-expect-through-site-to-site-or-point-to-site-connections"></a>透過網站間或點對站台連線可以獲得多少輸送量？

很難維護 VPN 通道的確切輸送量。 IPsec 和 SSTP 為加密嚴謹的 VPN 通訊協定。 輸送量也會受限於內部部署與網際網路之間的延遲和頻寬。
