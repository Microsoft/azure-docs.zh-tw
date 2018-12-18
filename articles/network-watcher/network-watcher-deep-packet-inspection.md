---
title: 使用 Azure 網路監看員進行封包檢查 | Microsoft Docs
description: 本文說明如何使用網路監看員執行從 VM 收集的深度封包檢查
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 7b907d00-9c35-40f5-a61e-beb7b782276f
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 1ad6ca4abe73336ce9ce3539fdaf2a9d7dd23fa6
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23036333"
---
# <a name="packet-inspection-with-azure-network-watcher"></a>使用 Azure 網路監看員進行封包檢查

使用網路監看員的封包擷取功能，可以從入口網站、PowerShell、CLI、以及以程式設計方式透過 SDK 和 REST API 起始並管理您 Azure VM 上的擷取工作階段。 封包擷取可讓您藉由以立即可用的格式提供資訊，處理需要封包層級資料的案例。 運用免費的工具來檢查資料，您可以檢查您 VM 來回傳送的通訊，並深入了解您的網路流量。 使用封包擷取資料的一些範例包括︰調查網路或應用程式問題、偵測網路遭誤用及入侵嘗試，或維護法規合規性。 在本文中，我們說明如何使用受歡迎的開放原始碼工具來開啟網路監看員所提供的封包擷取檔案。 我們也會提供範例來示範如何計算連線延遲時間、找出異常的流量，並檢查網路統計資料。

## <a name="before-you-begin"></a>開始之前

本文逐步引導您完成先前執行的封包擷取上之一些預先設定的情況。 這些案例說明可藉由檢閱封包擷取進行存取的功能。 本案例使用 [WireShark](https://www.wireshark.org/) 來檢查封包擷取。

本案例假設您已經在虛擬機器上執行封包擷取。 若要了解如何建立封包擷取，請造訪[使用入口網站管理封包擷取](network-watcher-packet-capture-manage-portal.md)，或使用 REST 則請造訪[使用 REST API 管理封包擷取](network-watcher-packet-capture-manage-rest.md)。

## <a name="scenario"></a>案例

在此案例中，您將會：

* 檢閱封包擷取

## <a name="calculate-network-latency"></a>計算網路延遲

在此案例中，我們會說明如何檢視兩個端點之間發生的傳輸控制通訊協定 (TCP) 對話的初始來回時間 (RTT)。

建立 TCP 連線時，連線中傳送的前三個封包會遵循通常稱為三向交握的模式。 藉由檢查此交握期間傳送的前兩個封包 (來自用戶端的初始要求和來自伺服器的回應)，我們可以計算建立此連線的延遲。 此延遲是指來回時間 (RTT)。 如需有關 TCP 通訊協定和三向交握的資訊，請參閱下列資源。 https://support.microsoft.com/en-us/help/172983/explanation-of-the-three-way-handshake-via-tcp-ip

### <a name="step-1"></a>步驟 1

啟動 WireShark

### <a name="step-2"></a>步驟 2

從封包擷取載入 **.cap** 檔案。 根據您設定的方式，可以在它於我們本機的虛擬機器上所儲存之 blob 中找到這個檔案。

### <a name="step-3"></a>步驟 3

若要在 TCP 對話中檢視初始的來回時間 (RTT)，我們將只會看到參與 TCP 交握的前兩個封包。 我們將在三向交握中使用前兩個封包，也就是 [SYN]、[SYN, ACK] 封包。 它們會針對 TCP 標頭中設定的旗標命名。 在交握期間的最後一個封包 ([ACK] 封包) 不會用於此案例。 [SYN] 封包會由用戶端傳送。 一旦收到它後，伺服器會傳送 [ACK] 封包做為從用戶端 SYN 接收的認可。 利用伺服器的回應需要極少額外負荷的特性，我們將用戶端接收 [SYN, ACK] 封包的時間減去用戶端傳送 [SYN] 封包的時間來計算 RTT。

使用 WireShark，系統會為我們計算此值。

若要更輕鬆地檢視 TCP 三向交握中的前兩個封包，我們會使用 WireShark 所提供的篩選功能。

若要在 WireShark 中套用篩選，請在擷取中展開 [SYN] 封包的「傳輸控制通訊協定」區段，並檢查 TCP 標頭中的旗標設定。

因為我們想要在所有 [SYN] 和 [SYN, ACK] 封包上篩選，請在旗標下確認 Syn 位元設為 1，然後以滑鼠右鍵按一下 [Syn 位元] -> [套用為篩選條件] -> [選取]。

![圖 7][7]

### <a name="step-4"></a>步驟 4

既然您已經將視窗篩選為只看到具有 [SYN] 位元設定的封包，您可以輕鬆地選取您感興趣的對話來檢視初始 RTT。 在 WireShark 中檢視 RTT 的簡單方式只需按一下標示為 “SEQ/ACK” 分析的下拉式清單。 然後，您會看到顯示 RTT。 在此案例中，RTT 是 0.0022114 秒或 2.211 毫秒。

![圖 8][8]

## <a name="unwanted-protocols"></a>不必要的通訊協定

您可以在已部署於 Azure 中的虛擬機器執行個體上執行許多應用程式。 多個應用程式會透過網路通訊，可能沒有明確的權限。 我們可以使用封包擷取儲存網路通訊，以調查應用程式在網路上的交談方式並尋找任何問題。

在此範例中，我們檢視先前執行之不必要通訊協定的封包擷取，其可能表示來自您電腦上執行之應用程式的未授權通訊。

### <a name="step-1"></a>步驟 1

使用先前案例中相同的擷取按一下 [統計資料] > [通訊協定階層]

![通訊協定階層功能表][2]

通訊協定階層視窗隨即出現。 此檢視會提供在擷取工作階段期間使用中的所有通訊協定、使用通訊協定已傳送和接收的封包數清單。 此檢視可用於找出您的虛擬機器或網路上不需要的網路流量。

![開啟的通訊協定階層][3]

如您在下列螢幕擷取畫面中所看到，有使用 BitTorrent 通訊協定的流量，用於對等檔案共用。 身為系統管理員，您不會看到這個特定虛擬機器上的 BitTorrent 流量。 現在您知道此流量，您可以移除此虛擬機器上安裝的對等軟體，或使用網路安全性群組或防火牆封鎖流量。 此外，您可以選擇執行排程上的封包擷取，讓您可以檢閱虛擬機器上定期使用的通訊協定。 如需如何在 Azure 中自動化網路工作的範例，請造訪[使用 Azure 自動化監視網路資源](network-watcher-monitor-with-azure-automation.md)

## <a name="finding-top-destinations-and-ports"></a>尋找最上層目的地和連接埠

監視或疑難排解應用程式和網路上的資源時，務必了解流量類型、端點和通訊的連接埠。 使用上述的封包擷取檔案，我們可以快速了解我們的 VM 所進行通訊以及所使用連接埠的最上層目的地。

### <a name="step-1"></a>步驟 1

使用先前案例中相同的擷取按一下 [統計資料] > [IPv4 統計資料] > [目的地和連接埠]

![封包擷取視窗][4]

### <a name="step-2"></a>步驟 2

隨著我們探討結果而逐漸明朗，在連接埠 111 上有多個連線。 最常使用的連接埠是 3389，這是遠端桌面，而剩下的是 RPC 動態連接埠。

此流量可能沒有意義，但這是用於許多連線且系統管理員不知道的連接埠。

![圖 5][5]

### <a name="step-3"></a>步驟 3

既然我們已判斷出位置不正確的連接埠，我們可以根據連接埠篩選我們的擷取。

此案例中的篩選條件會是︰

```
tcp.port == 111
```

我們在篩選器文字方塊中從上方輸入篩選文字，並按 enter 鍵。

![圖 6][6]

從結果中，我們可以看到所有的流量來自相同子網路上的本機虛擬機器。 如果我們還是不懂此流量發生的原因，可以進一步檢查封包來判斷為何它在連接埠 111 上進行這些呼叫。 利用此資訊，我們可以採取適當動作。

## <a name="next-steps"></a>後續步驟

若要深入了解網路監看員的其他診斷功能，請造訪 [Azure 網路監視概觀](network-watcher-monitoring-overview.md)

[1]: ./media/network-watcher-deep-packet-inspection/figure1.png
[2]: ./media/network-watcher-deep-packet-inspection/figure2.png
[3]: ./media/network-watcher-deep-packet-inspection/figure3.png
[4]: ./media/network-watcher-deep-packet-inspection/figure4.png
[5]: ./media/network-watcher-deep-packet-inspection/figure5.png
[6]: ./media/network-watcher-deep-packet-inspection/figure6.png
[7]: ./media/network-watcher-deep-packet-inspection/figure7.png
[8]: ./media/network-watcher-deep-packet-inspection/figure8.png













