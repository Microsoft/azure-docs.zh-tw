---
title: 使用 Azure 網路監看員和開放原始碼工具將網路流量模式視覺化 | Microsoft Docs
description: 此頁面描述如何使用網路監看員封包擷取並搭配 Capanalysis，將往返於 VM 的流量模式視覺化。
services: network-watcher
documentationcenter: na
author: jimdial
manager: timlt
editor: ''
ms.assetid: 936d881b-49f9-4798-8e45-d7185ec9fe89
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: jdial
ms.openlocfilehash: 7b1e1383e8e244a7cdb30be1e08514a6a4dd7b14
ms.sourcegitcommit: ea5193f0729e85e2ddb11bb6d4516958510fd14c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/21/2018
ms.locfileid: "36302228"
---
# <a name="visualize-network-traffic-patterns-to-and-from-your-vms-using-open-source-tools"></a>使用開放原始碼工具將往返於 VM 的網路流量模式視覺化

封包擷取包含的網路資料可讓您執行網路鑑識和深入的封包檢查。 有許多開啟原始碼工具可用來分析封包擷取，以深入探索您的網路。 例如 CapAnalysis，一個開放原始碼封包擷取視覺效果工具。 將封包擷取資料視覺化有助於快速深入探索網路內的模式和異常。 視覺效果也可讓您輕鬆分享這種深入解析。

Azure 的網路監看員可讓您在網路上執行封包擷取，以便擷取資料。 本文提供逐步解說，示範如何使用 CapAnalysis 搭配網路監看員，以視覺化和深入探索封包擷取。

## <a name="scenario"></a>案例

您在 Azure 的 VM 上部署簡單的 Web 應用程式，而且想要使用開放原始碼工具將網路流量視覺化，以快速識別流程模式和任何可能異常情況。 您可以利用網路監看員取得網路環境的封包擷取，並直接儲存在儲存體帳戶中。 然後，CapAnalysis 可以直接從儲存體 blob 內嵌封包擷取，並將其內容視覺化。

![案例][1]

## <a name="steps"></a>步驟

### <a name="install-capanalysis"></a>安裝 CapAnalysis

若要在虛擬機器上安裝 CapAnalysis，您可以參考此處的官方指示：https://www.capanalysis.net/ca/how-to-install-capanalysis。
若要從遠端存取 CapAnalysis，您必須在 VM 上新增輸入安全性規則，以開啟連接埠 9877。 如需有關在網路安全性群組中建立規則的詳細資訊，請參閱[在現有 NSG 中建立規則](../virtual-network/manage-network-security-group.md#create-a-security-rule)。 成功新增規則之後，您應該能夠從 `http://<PublicIP>:9877` 存取CapAnalysis

### <a name="use-azure-network-watcher-to-start-a-packet-capture-session"></a>使用 Azure 網路監看員啟動封包擷取工作階段

網路監看員可讓您擷取封包，以追蹤進出虛擬機器的流量。 您可以參考[使用網路監看員管理封包擷取](network-watcher-packet-capture-manage-portal.md)中的指示，啟動封包擷取工作階段。 封包擷取可以儲存在儲存體 blob 中，供 CapAnalysis 存取。

### <a name="upload-a-packet-capture-to-capanalysis"></a>將封包擷取上傳至 CapAnalysis
您可以使用 [從 URL 匯入] 索引標籤，並提供儲存封包擷取的儲存體 blob 連結，以直接上傳網路監看員所產生的封包擷取。

提供 CapAnalysis 的連結時，請務必將 SAS 權杖附加至儲存體 blob URL。  若要這樣做，請從儲存體帳戶瀏覽至共用存取簽章，指定允許的權限，然後按下 [產生 SAS] 按鈕以建立權杖。 接著，您可以將 SAS 權杖附加至封包擷取儲存體 blob URL。

產生的 URL 看起來如以下的 URL：http://storageaccount.blob.core.windows.net/container/location?addSASkeyhere


### <a name="analyzing-packet-captures"></a>分析封包擷取

CapAnalysis 提供各種選項將封包擷取視覺化，各以不同的觀點提供分析。 這些視覺化摘要可讓您了解網路流量趨勢，並快速找出任何不尋常的活動。 下列清單顯示其中幾項功能︰

1. 流程資料表

    下表提供封包資料中的流程清單、流程相關聯的時間戳記和流程相關聯的各種通訊協定，以及來源和目的地 IP。

    ![capanalysis 流程頁面][5]

1. 通訊協定概觀

    此窗格可讓您快速查看各種通訊協定和地理位置的網路流量分佈。

    ![capanalysis 通訊協定概觀][6]

1. 統計資料

    此窗格可讓您檢視網路流量統計資料 – 從來源和目的地 IP 傳送和接收的位元組、每個來源和目的地 IP 的流程、用於各種流程的通訊協定，以及流程的持續時間。

    ![capanalysis 統計資料][7]

1. Geomap

    此窗格提供網路流量的地圖檢視，並依據每個國家/地區的流量大小而改變顏色。 您可以選取醒目提示的國家/地區，以檢視其他流程統計資料，例如從該國家/地區中的 IP 傳送和接收的資料比例。

    ![geomap][8]

1. 篩選器

    CapAnalysis 提供一組可快速分析特定封包的篩選器。 例如，您可以選擇依通訊協定來篩選資料，以具體深入探索該流量子集。

    ![filters][11]

    請瀏覽 [https://www.capanalysis.net/ca/#about](https://www.capanalysis.net/ca/#about)，深入了解 CapAnalysis 的所有功能。

## <a name="conclusion"></a>結論

網路監看員的封包擷取功能可讓您擷取執行網路鑑識所需的資料，並深入了解您的網路流量。 在此案例中，我們示範如何使用開放原始碼視覺效果工具，輕鬆地整合來自網路監看員的封包擷取。 使用 CapAnalysis 之類的開放原始碼工具將封包擷取視覺化，可讓您執行深入的封包檢查，並快速識別網路流量的趨勢。

## <a name="next-steps"></a>後續步驟

若要深入了解 NSG 流程記錄，請參閱 [NSG 流程記錄](network-watcher-nsg-flow-logging-overview.md)

若要了解如何利用 Power BI 將 NSG 流量記錄視覺化，請瀏覽[利用 Power BI 將 NSG 流量記錄視覺](network-watcher-visualize-nsg-flow-logs-power-bi.md)
<!--Image references-->

[1]: ./media/network-watcher-using-open-source-tools/figure1.png
[2]: ./media/network-watcher-using-open-source-tools/figure2.png
[3]: ./media/network-watcher-using-open-source-tools/figure3.png
[4]: ./media/network-watcher-using-open-source-tools/figure4.png
[5]: ./media/network-watcher-using-open-source-tools/figure5.png
[6]: ./media/network-watcher-using-open-source-tools/figure6.png
[7]: ./media/network-watcher-using-open-source-tools/figure7.png
[8]: ./media/network-watcher-using-open-source-tools/figure8.png
[9]: ./media/network-watcher-using-open-source-tools/figure9.png
[10]: ./media/network-watcher-using-open-source-tools/figure10.png
[11]: ./media/network-watcher-using-open-source-tools/figure11.png
