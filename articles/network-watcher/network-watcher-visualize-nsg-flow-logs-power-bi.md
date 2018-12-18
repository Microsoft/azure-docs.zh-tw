---
title: 使用 Power BI 將 Azure 網路安全性群組流程記錄視覺化 | Microsoft Docs
description: 此頁面說明如何使用 Power BI 將 NSG 流程記錄視覺化。
services: network-watcher
documentationcenter: na
author: mattreatMSFT
manager: vitinnan
editor: ''
ms.assetid: 1e4f95fa-f5f0-4e03-bc25-008fbfc4934c
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: mareat
ms.openlocfilehash: bcd67d2e879d199296289dbaef2141ba659b2412
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032673"
---
# <a name="visualizing-network-security-group-flow-logs-with-power-bi"></a>使用 Power BI 將網路安全性群組流程記錄視覺化

網路安全性群組流程記錄可讓您檢視網路安全性群組上輸入和輸出 IP 流量的相關資訊。 這些流程記錄顯示每一個規則的輸出和輸入流程、套用流程的 NIC、有關流程的 5 組資訊 (來源/目的地 IP、來源/目的地連接埠、通訊協定)，以及允許或拒絕流量。

手動搜尋記錄檔很難深入探索流程記錄資料。 在此文章中，我們提供解決方案將最近的流程記錄視覺化，並了解您網路上的流量。

## <a name="scenario"></a>案例

在下列案例中，我們將 Power BI Desktop 連線至我們已設定的儲存體帳戶，做為 NSG 流程記錄資料的接收器。 當我們連線至儲存體帳戶之後，Power BI 會下載並剖析記錄，以視覺表示法呈現網路安全性群組所記錄的流量。

您可以使用範本中提供的視覺效果來檢查︰

* 熱門 IP
* 依方向和規則決策的時間序列資料流程
* 依網路介面 MAC 位址的流程
* 依 NSG 和規則的流程
* 依目的地連接埠的流程

提供的範本是可編輯的，您可以修改它來新增資料、視覺效果或編輯查詢，以符合您的需求。

## <a name="setup"></a>設定

開始之前，您必須在帳戶中的一或多個網路安全性群組上，啟用網路安全性群組流程記錄。 如需有關啟用網路安全性流程記錄的指示，請參閱下列文章︰[網路安全性群組的流程記錄簡介](network-watcher-nsg-flow-logging-overview.md)。

您也必須在機器上安裝 Power BI Desktop 用戶端，而且機器上要有足夠的可用空間，以下載和載入儲存體帳戶中在存的記錄資料。

![Visio 圖表][1]

### <a name="steps"></a>步驟

1. 在 Power BI Desktop 應用程式的[網路監看員 PowerBI 流程記錄範本](https://aka.ms/networkwatcherpowerbiflowlogstemplate)中，下載並開啟下列 Power BI 範本
1. 輸入必要的查詢參數
    1. **StorageAccountName** – 指定儲存體帳戶的名稱，此帳戶包含您想要載入和視覺化的 NSG 流程記錄。
    1. **NumberOfLogFiles** – 指定您想要在 Power BI 中下載和視覺化的記錄檔數目。 例如，指定 50 代表最新的 50 個記錄檔。 如果我們有 2 個 NSG 已啟用並設定為將 NSG 流程記錄傳送至這個帳戶，則可以檢視過去 25 小時的記錄。

    ![power BI 主畫面][2]

1. 輸入儲存體帳戶的存取金鑰。 您可以在 Azure 入口網站瀏覽至您的儲存體帳戶，然後從 [設定] 功能表選取 [存取金鑰]，即可找到有效的存取金鑰。 按一下 [連線]，然後套用變更。

    ![access keys][3]

    ![存取金鑰 2][4]

4.  您的記錄已下載和剖析，現在可以利用預先建立的視覺效果。

## <a name="understanding-the-visuals"></a>了解視覺效果

範本中提供幾組視覺效果，有助於理解 NSG 流程記錄資料的意義。 下圖示範儀表板填入資料後的樣貌。 接下來，我們更詳細地探討每個視覺效果 

![powerbi][5]
 
「熱門 IP」視覺效果顯示指定期間內起始最多連線的 IP。 方塊大小代表相對的連線數。 

![toptalkers][6]

下列時間序列圖表顯示期間內的流程數。 上方圖表依流程方向切割，下方圖表依所做決策 (允許或拒絕) 分割。 此視覺效果可讓您檢查一段時間的流量趨勢，並找出流量或流量分割中的任何異常暴增或遽減情況。

![flowsoverperiod][7]

下列圖表顯示每個網路介面的流程，上方是依流程方向分割，下方是依所做決策分割。 此資訊可讓您深入探索通訊量最大的 VM，以及是否已允許或拒絕流向特定 VM 的流量。

![flowspernic][8]

下列環圈圖顯示「依目的地連接埠的流程」分解。 此資訊可讓您檢視指定期間內最常使用的目的地連接埠。

![環圈][9]

下列長條圖顯示依 NSG 和規則的流程。 此資訊可讓您看到引起最多流量的 NSG，並依規則查看 NSG 的流量分解。

![barchart][10]
 
下列資訊圖表顯示出現在記錄中的 NSG 相關資訊、期間擷取的流程數，以及最早記錄的擷取日期。 此資訊可讓您了解記錄哪些 NSG 和流程的日期範圍。

![infochart1][11]

![infochart2][12]

此範本包括下列交叉分析篩選器，可讓您只檢視最感興趣的資料。 您可以依資源群組、NSG 和規則來篩選。 您也可以依 5 組資訊、決策和記錄的寫入時間來篩選。

![交叉分析篩選器][13]

## <a name="conclusion"></a>結論

我們在此案例中說明，透過使用網路監看員和 Power BI 所提供的網路安全性群組流程記錄，我們就能夠視覺化並了解流量。 使用提供的範本時，Power BI 會直接從儲存體下載記錄，然後在本機處理。 載入範本所花費的時間，取決於所要求的檔案數目和下載的檔案大小總計。

請儘管自訂此範本以符合您的需求。 Power BI 和網路安全性群組流程記錄互相搭配有許多做法。 

## <a name="notes"></a>注意

* 記錄依預設儲存於 `https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/`

    * 如果其他資料存在於另一個目錄，則必須修改查詢來提取和處理資料。

* 提供的範本不建議用於 1 GB 以上的記錄。

* 如果您有大量的記錄，我們建議您探尋使用其他資料存放區的解決方案，例如 Data Lake 或 SQL Server。

## <a name="next-steps"></a>後續步驟

請瀏覽[使用開放原始碼工具將 Azure 網路監看員 NSG 流程記錄視覺化](network-watcher-visualize-nsg-flow-logs-open-source-tools.md)，了解如何以 Elastick Stack 將 NSG 流程記錄視覺化

[1]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure7.png
[8]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure8.png
[9]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure9.png
[10]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure10.png
[11]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure11.png
[12]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure12.png
[13]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure13.png
