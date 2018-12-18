---
title: Azure 虛擬網路效能疑難排解 | Microsoft Docs
description: 本頁提供測試 Azure 網路連結效能的標準化方法。
services: expressroute
documentationcenter: na
author: tracsman
manager: rossort
editor: ''
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/20/2017
ms.author: jonor
ms.openlocfilehash: 56f011632a2aa3ef0632efd5ace472c0fc79a329
ms.sourcegitcommit: a648f9d7a502bfbab4cd89c9e25aa03d1a0c412b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/22/2017
ms.locfileid: "27319372"
---
# <a name="troubleshooting-network-performance"></a>網路效能疑難排解
## <a name="overview"></a>概觀
Azure 提供穩定且快速的方式，從您的內部部署網路連線到 Azure。 大型和小型的客戶可以成功使用站對站 VPN 和 ExpressRoute 等方法，在 Azure 中經營其業務。 但是當效能不符合您預期的情況或先前的經驗時，會發生什麼事？ 本文件可協助您將測試方式標準化，並為您的特定環境制訂基準。

本文件說明如何在兩部主機之間輕鬆且一致地測試網路延遲和頻寬。 本文件也針對查看 Azure 網路的方式提供一些建議，並協助您隔離問題點。 討論的 PowerShell 指令碼和工具在網路上 (要測試之連結的任一端) 需要有兩部主機。 一部主機必須是 Windows 伺服器或桌面，另一部主機則可以是 Windows 或 Linux。 

>[!NOTE]
>疑難排解的方法、工具和使用的方法都是個人喜好設定。 本文件說明通常採用的方法和工具。 您的方法可能會有所不同，使用不同的方法解決問題並沒有對錯之分。 不過，如果您沒有一個既定的方法，則可以將本文件當作開始建構自己的方法、工具和喜好設定的途徑，以便為網路問題進行疑難排解。
>
>

## <a name="network-components"></a>網路構成要素
深入了解疑難排解之前，讓我們先討論一些通用的術語和構成要素。 這個討論可確保我們將考慮端對端鏈結中，可在 Azure 中連線的每個構成要素。
[![1]][1]

我將說明最高層級的三種主要網路路由網域；

- Azure 網路 (右側的藍色雲)
- 網際網路或 WAN (中央的綠色雲)
- 公司網路 (左側的桃色雲)

由右至左查看圖表之後，讓我們簡短地討論一下每個構成要素：
 - **虛擬機器**：伺服器可能有多個 NIC，可確保任何靜態路由、預設路由和作業系統設定都會以您認為的方式傳送和接收流量。 此外，每個 VM SKU 都有頻寬限制。 如果您要使用較小的 VM SKU，則您的流量會受到 NIC 可用頻寬的限制。 我通常使用 DS5v2 進行測試 (然後在完成測試之後刪除以節省成本) 以確保 VM 有足夠的頻寬。
 - **NIC**：請確定您知道指派給上述 NIC 的私人 IP。
 - **NIC NSG**：可能會有在 NIC 層級套用的特定 NSG，請確定 NSG 規則集適用於您想要通過的流量。 例如，確定已開放連接埠 5201 (用於 iPerf)、3389 (用於 RDP) 或 22 (用於 SSH)，以允許通過測試流量。
 - **VNet 子網路**：NIC 會指派給特定子網路，請確定您知道哪些規則與該子網路相關聯。
 - **子網路 NSG**：就像 NIC 一樣，也可以在子網路套用 NSG。 請確定 NSG 規則集適用於您想要通過的流量  (若是輸入至 NIC 的流量，會先套用子網路 NSG，然後再套用 NIC NSG，相反地，若是從 VM 輸出的流量，則會先套用 NIC NSG，然後子網路 NSG 才派上用場)。
 - **子網路 UDR**：使用者定義的路由可以將流量導向至中繼躍點 (例如，防火牆或負載平衡器)。 請確定您知道是否已經有適用於您流量的 UDR，如果有，則該流量將流向何處，且下一個躍點將會對您的流量採取什麼動作  (例如，防火牆可能會在相同的兩部主機之間通過一些流量，並拒絕一些流量)。
 - **閘道子網路/NSG/UDR**：就像 VM 子網路一樣，閘道子網路可以擁有 NSG 和 UDR。 請確定您知道它們是否在那裡，以及他們對流量的影響。
 - **VNet 閘道 (ExpressRoute)**：一旦啟用對等互連 (ExpressRoute) 或 VPN 之後，則不會有很多設定可以影響流量如何路由或是否路由。 如果您有多個 ExpressRoute 線路或 VPN 通道連線至相同的 VNet 閘道，您應該了解連線權數設定，因為這個設定會影響連線喜好設定，而且會影響您流量所採用的路徑。
 - **路由篩選** (未顯示)：路由篩選僅適用於 ExpressRoute 上的 Microsoft 對等互連 ，但請務必確認您在 Microsoft 對等互連上是否看不到您預期的路由。 

這表示您目前位於連結的 WAN 部分。 此路由網域可以是您的服務提供者、您的公司 WAN 或網際網路。 與這些連結相關的許多躍點、技術和公司都可能會使該路由網域有點難以進行疑難排解。 通常，您要先同時排除 Azure 和您的公司網路，才能跳入這個公司和躍點的集合。

在上圖中，最左邊是您的公司網路。 根據您公司的規模，此路由網域可以是您與 WAN 之間的一些網路裝置，或是校園/企業網路中多層裝置之間的一些網路裝置。

鑑於這三種不同高層級網路環境的複雜性，通常最好是從邊緣開始，並嘗試顯示效能良好之處，以及效能低落之處。 這種方法有助於識別三個路由網域的問題，然後將疑難排解的重點放在該特定環境。

## <a name="tools"></a>工具
大部分的網路問題都可以使用 Ping 和 traceroute 等基本工具加以分析和隔離。 很少需要進行 Wireshark 之類的封包深入分析。 為協助進行疑難排解，將開發 Azure 連線能力工具組 (AzureCT)，以便將其中部分工具置於簡易的套件內。 為進行效能測試，我要使用 iPerf 和 PSPing。 iPerf 是一種常用的工具，可在大部分的作業系統上執行。 iPerf 適用於基本效能測試，而且相當容易使用。 PSPing 是 SysInternals 所開發的一種 Ping 工具。 PSPing 是一種簡單的方式，可以在一個也很容易使用的命令中執行 ICMP 和 TCP Ping。 這兩種都是輕量型工具，只要將檔案複製到主機上的目錄就可以「安裝」妥當。

我已經將這些所有工具和方法都包裝成一個 PowerShell 模組 (AzureCT)，讓您可以安裝並使用。

### <a name="azurect---the-azure-connectivity-toolkit"></a>AzureCT：Azure 連線能力工具組
AzureCT PowerShell 模組有兩個構成要素，分別是[可用性測試][Availability Doc]和[效能測試][Performance Doc]。 本文件僅涉及效能測試，因此，請將注意力放在此 PowerShell 模組中的兩個連結效能命令上。

使用此工具組進行效能測試有三個基本步驟。 1) 安裝 PowerShell 模組、2) 安裝支援的應用程式 iPerf 和 PSPing、3) 執行效能測試。

1. 安裝 PowerShell 模組

    ```powershell
    (new-object Net.WebClient).DownloadString("https://aka.ms/AzureCT") | Invoke-Expression
    
    ```

    此命令會下載 PowerShell 模組，並將其安裝在本機上。

2. 安裝支援的應用程式
    ```powershell
    Install-LinkPerformance
    ```
    此 AzureCT 命令會將 iPerf 和 PSPing 安裝在新目錄 "C:\ACTTools" 中，也會開啟 Windows 防火牆連接埠以允許 ICMP 和連接埠 5201 (iPerf) 流量。

3. 執行效能測試

    首先，您必須在遠端主機上使用伺服器模式安裝並執行 iPerf。 此外，請確定遠端主機是接聽 3389 (RDP 用於 Windows) 還是 22 (SSH 用於 Linux)，並在連接埠 5201 上允許 iPerf 的流量。 如果遠端主機是 Windows，請安裝 AzureCT 並執行 Install-LinkPerformance 命令來設定在伺服器模式下成功啟動 iPerf 所需的 iPerf 及防火牆規則。 
    
    一旦遠端電腦準備就緒之後，請在本機電腦上開啟 PowerShell 並開始測試：
    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 10
    ```

    此命令會執行一連串的並行載入和延遲測試，以協助您預估網路連結的頻寬容量和延遲。

4. 檢閱測試的輸出

    PowerShell 輸出格式如下所示：

    [![4]][4]

    所有 iPerf 和 PSPing 測試的詳細結果都位於 AzureCT 工具目錄 "C:\ACTTools" 下的個別文字檔中。

## <a name="troubleshooting"></a>疑難排解
如果效能測試未提供您預期的結果，找出原因應該是一個循序漸進的過程。 鑑於途徑中的構成要素數目，系統化的方法通常會提供更快的解決途徑，而不是毫無頭緒，而且可能會不必要地執行多次相同的測試。

>[!NOTE]
>以下的案例是效能問題，而不是連線問題。 如果流量完全沒有通過，則步驟將會不同。
>
>

首先，挑戰您的假設。 您的預期合理嗎？ 例如，如果您的 ExpressRoute 線路為 1-Gbps 且延遲為 100 毫秒，則鑑於高延遲連結的 TCP 效能特性，預期流量為完整的 1 Gbps 是不合理的。 如需有關效能假設的詳細資訊，請參閱[參考](#references)一節。

接著，我建議從路由網域之間的邊緣開始，並嘗試將問題隔離到單一的主要路由網域，也就是公司網路、WAN 或 Azure 網路。 人們通常會在過程中指責「黑箱作業」，因為指責「黑箱作業」很容易，但這可能會大幅延遲解決的時間，特別是問題實際上出在有能力可以改變的區域時。 請確定您先盡職調查之後，才交由服務提供者或 ISP 處理。

一旦您已經找出可能含有問題的主要路由網域之後，您應該建立上述區域的圖表。 在白板、記事本或作為圖表的 Visio 上提供具體的「戰鬥地圖」，以允許使用有條不紊的方法，進一步隔離問題。 您可以規劃測試點，並在清除區域時更新地圖，或隨著測試的進行，挖掘更深入的資料。

既然您有圖表，就可以開始將網路分成區段，並縮小問題的範圍。 找出適用的範圍以及不適用的範圍。 繼續移動您的測試點，以向下隔離到引起問題的構成要素。

此外，別忘了查看其他層的 OSI 模型。 將重點放在網路及第 1-3 層 (實體、資料和網路層) 很容易，但問題也可能會在應用程式層的第 7 層。 請保持開放的心態並驗證假設。

## <a name="advanced-expressroute-troubleshooting"></a>進階 ExpressRoute 疑難排解
如果您不確定雲端邊緣實際上的位置，隔離 Azure 構成要素可能是一個挑戰。 使用 ExpressRoute 時，邊緣是一個名為 Microsoft Enterprise Edge (MSEE) 的網路構成要素。 **使用 ExpressRoute 時**，MSEE 是進入 Microsoft 網路的第一個連絡點，以及離開 Microsoft 網路的最後一個躍點。 當您在 VNet 閘道與 ExpressRoute 線路之間建立連接物件時，您實際上是在建立與 MSEE 的連線。 將 MSEE 辨識為第一個或最後一個躍點 (視您前往的方向而定) 對於隔離 Azure 網路問題至關緊要，因為這可以證明問題是在 Azure 中，還是在 WAN 或公司網路的下游。 

[![2]][2]

>[!NOTE]
> 請注意，MSEE 不在 Azure 雲端。 ExpressRoute 實際上是在 Microsoft 網路的邊緣，而不是在 Azure 中。 一旦您將 ExpressRoute 連線到 MSEE 之後，表示您已經連線到 Microsoft 的網路，然後您可以從那裡再移至任何雲端服務，例如 Office 365 (利用 Microsoft 對等互連) 或 Azure (利用私人和/或 Microsoft 對等互連)。
>
>

如果兩個 Vnet (圖表中的 Vnet A 和 B) 都連線到**相同的** ExpressRoute 線路，則您可以執行一連串的測試，以隔離 Azure 中的問題 (或證明問題不在 Azure 中)
 
### <a name="test-plan"></a>測試計劃
1. 在 VM1 和 VM2 之間執行 Get-LinkPerformance 測試。 此測試會針對問題是否出在本機，提供深入見解。 如果此測試產生可接受的延遲和頻寬結果，您可以將本機的 VNet 網路標示為良好。
2. 假設本機的 VNet 流量良好，請在 VM1 和 VM3 之間執行 Get-LinkPerformance 測試。 此測試會透過 Microsoft 網路，向下連線至 MSEE，然後再連回 Azure。 如果此測試產生可接受的延遲和頻寬結果，您可以將 Azure 網路標示為良好。
3. 如果排除 Azure，您可以在公司網路上執行一連串類似的測試。 如果測試結果也良好，則應該與您的服務提供者或 ISP 共同診斷您的 WAN 連線。 範例：在兩個分公司之間，或您的支援人員及資料中心伺服器之間，執行這項測試。 根據您要測試的內容，找出可以運用該途徑的端點 (伺服器、電腦等)。

>[!IMPORTANT]
> 請務必針對您標示一天時間的每個測試，執行測試並將結果記錄在一個通用位置 (我喜歡用 OneNote 或 Excel)。 每個測試回合都應該有相同的輸出，因此您可以跨測試回合比較所產生的資料，而不會有資料上的「漏洞」。 多項測試間的一致性是我使用 AzureCT 進行疑難排解的主要原因。 *神奇之處*不在於我執行的確切負載案例中，而是在於我可以從每個測試取得*一致的測試和資料輸出*。 如果您之後發現問題是偶爾發生的，則記錄時間以及每次獲得一致的資料特別有幫助。 一開始努力收集資料，便可以避免花費數小時重新測試相同的案例 (我在多年前學到了這一點)。
>
>

## <a name="the-problem-is-isolated-now-what"></a>問題遭到隔離，接下來怎麼辦？
您可以隔離的問題越多，問題的修正越容易，但是您通常會到達無法再繼續深入或無法進一步疑難排解的點。 範例：您看到服務提供者的連結採用歐洲的躍點，但您預期的途徑全部位於亞洲。 這時候就是您應該向外尋求協助的時機。 您尋求協助的對象取決於您隔離問題的路由網域，如果您可以將問題範圍縮小到特定的構成要素更好。

針對公司網路的問題，內部 IT 部門或支援您網路的服務提供者 (可能是硬體製造商) 或許能夠協助進行裝置設定或硬體修復。

若是 WAN，與您的服務提供者或 ISP 分享您的測試結果可能會協助他們開始著手之處，並避免涵蓋一些已經測試過的相同領域。 但是，如果他們想要自行驗證您得到的結果，請不要不高興。 根據其他人報告的結果進行疑難排解時，「要信任也要查證」是很好的座右銘。

透過 Azure，一旦您盡可能地詳細隔離問題之後，就可以檢閱 [Azure 網路文件][Network Docs]，之後如果仍然需要，[請開啟支援票證][Ticket Link]。

## <a name="references"></a>參考
### <a name="latencybandwidth-expectations"></a>延遲/頻寬期望
>[!TIP]
> 您要測試的端點之間的地理延遲 (英哩或公里) 是目前最大的延遲構成要素。 雖然沒有相關的設備延遲 (實體和虛擬構成要素、躍點數目等)，但處理 WAN 連線時，地理位置已經公認是整體延遲的最大構成要素。 同樣重要的是要注意，距離是光纖行程的距離，而不是直線距離或路段圖地圖距離。 這個距離很難準確無誤。 因此，我通常使用網際網路上的城市距離計算機，而且知道這種方法是非常不準確的測量方法，但足以設定一般預期。
>
>

我已經在美國華盛頓州的西雅圖設定好 ExpressRoute。 下表顯示我看到測試到各個 Azure 位置的延遲和頻寬。 我已經估計了測試每一端之間的地理距離。

測試設定：
 - 使用連線到 ExpressRoute 線路的 10 Gbps NIC 執行 Windows Server 2016 的實體伺服器。
 - 在啟用私人對等互連時識別之位置中的 10Gbps Premium ExpressRoute 線路。
 - 在指定的區域中，具有 UltraPerformance 閘道的 Azure VNet。
 - 在 VNet 上執行 Windows Server 2016 的 DS5v2 VM。 在已安裝 AzureCT 的情況下，從預設 Azure 映像 (未最佳化或自訂) 建置的 VM 未加入網域。
 - 所有測試都是使用 AzureCT Get-LinkPerformance 命令，而且六個測試回合中，每個測試回合都有 5 分鐘的負載測試。 例如︰

    ```powershell
    Get-LinkPerformance -RemoteHost 10.0.0.1 -TestSeconds 300
    ```
 - 對於每個測試，其資料流程的負載都是從內部部署實體伺服器 (位於西雅圖的 iPerf 用戶端) 向上流動至 Azure VM (所列 Azure 地區中的 iPerf 伺服器)。
 - 「延遲」資料行的資料來自無負載測試 (不執行 iPerf 的 TCP 延遲測試)。
 - 「最大頻寬」資料行的資料來自視窗大小為 1-Mb 的 16 個 TCP 流量負載測試。

[![3]][3]

### <a name="latencybandwidth-results"></a>延遲/頻寬結果
>[!IMPORTANT]
> 這些數字僅供一般參考。 許多因素都會影響延遲，而且在經過一段時間之後，這些值通常就會變成一致，但是在 Azure 或服務提供者網路中的條件可以隨時透過不同的途徑傳送流量，因此延遲和頻寬可能會受到影響。 一般而言，這些變更的影響不會造成顯著的差異。
>
>

| | | | | | |
|-|-|-|-|-|-|
|ExpressRoute<br/>位置|Azure<br/>區域|預估<br/>距離 (公里)|Latency|1 個工作階段<br/>頻寬|最大值<br/>頻寬|
| Seattle | 美國西部 2        |    191 公里 |   5 毫秒 | 262.0 Mbits/秒 |  3.74 Gbits/秒 | 21
| Seattle | 美國西部          |  1,094 公里 |  18 毫秒 |  82.3 Mbits/秒 |  3.70 Gbits/秒 | 20
| Seattle | 美國中部       |  2,357 公里 |  40 毫秒 |  38.8 Mbits/秒 |  2.55 Gbits/秒 | 17
| Seattle | 美國中南部 |  2,877 公里 |  51 毫秒 |  30.6 Mbits/秒 |  2.49 Gbits/秒 | 19
| Seattle | 美國中北部 |  2,792 公里 |  55 毫秒 |  27.7 Mbits/秒 |  2.19 Gbits/秒 | 18
| Seattle | 美國東部 2        |  3,769 公里 |  73 毫秒 |  21.3 Mbits/秒 |  1.79 Gbits/秒 | 16
| Seattle | 美國東部          |  3,699 公里 |  74 毫秒 |  21.1 Mbits/秒 |  1.78 Gbits/秒 | 15
| Seattle | 日本東部       |  7,705 公里 | 106 毫秒 |  14.6 Mbits/秒 |  1.22 Gbits/秒 | 28
| Seattle | 英國南部         |  7,708 公里 | 146 毫秒 |  10.6 Mbits/秒 |   896 Mbits/秒 | 24
| Seattle | 西歐      |  7,834 公里 | 153 毫秒 |  10.2 Mbits/秒 |   761 Mbits/秒 | 23
| Seattle | 澳洲東部   | 12,484 公里 | 165 毫秒 |   9.4 Mbits/秒 |   794 Mbits/秒 | 26
| Seattle | 東南亞   | 12,989 公里 | 170 毫秒 |   9.2 Mbits/秒 |   756 Mbits/秒 | 25
| Seattle | 巴西南部 *   | 10,930 公里 | 189 毫秒 |   8.2 Mbits/秒 |   699 Mbits/秒 | 22
| Seattle | 印度南部      | 12,918 公里 | 202 毫秒 |   7.7 Mbits/秒 |   634 Mbits/秒 | 27

\* 巴西延遲是不錯的範例，其中的直線距離明顯不同於光纖運行距離。 我本來預期延遲大約為 160 毫秒，但是實際上是 189 毫秒。 與我預期不符的這項差異可能表示某個地方存在網路問題，但最有可能的是光纖運行並不是以直線進入巴西，而是距離西雅圖還超過 1,000 公里左右，才能到達巴西。

## <a name="next-steps"></a>後續步驟
1. 從 GitHub 下載 Azure 連線能力工具組，網址是：[http://aka.ms/AzCT][ACT]
2. 依照指示進行[連結效能測試][Performance Doc]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-network-performance/network-components.png "Azure 網路構成要素"
[2]: ./media/expressroute-troubleshooting-network-performance/expressroute-troubleshooting.png "ExpressRoute 疑難排解"
[3]: ./media/expressroute-troubleshooting-network-performance/test-diagram.png "效能測試環境"
[4]: ./media/expressroute-troubleshooting-network-performance/powershell-output.png "PowerShell 輸出"

<!--Link References-->
[Performance Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/PerformanceTesting.md
[Availability Doc]: https://github.com/Azure/NetworkMonitoring/blob/master/AzureCT/AvailabilityTesting.md
[Network Docs]: https://docs.microsoft.com/azure/index#pivot=services&panel=network
[Ticket Link]: https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview
[ACT]: http://aka.ms/AzCT











