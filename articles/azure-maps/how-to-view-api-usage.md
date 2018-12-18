---
title: 了解如何查檢視Azure 地圖服務 API 使用量 | Microsoft Docs
description: 了解如何在入口網站中檢視您 Azure 地圖服務 API 呼叫的計量。
author: dsk-2015
ms.author: dkshir
ms.date: 08/06/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: e62d2ff1fdd6bc94244511a2de95c4268a58d6f9
ms.sourcegitcommit: 0fcd6e1d03e1df505cf6cb9e6069dc674e1de0be
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42142088"
---
# <a name="how-to-view-the-azure-maps-api-usage"></a>如何檢視 Azure 地圖服務 API 使用量
此文章說明如何在[入口網站](https://portal.azure.com)中檢視您 Azure 地圖服務帳戶的 API 使用量計量。 計量是以方便檢視的圖形格式顯示，並包含可自訂的持續時間。 

## <a name="view-metric-snapshot"></a>檢視計量快照集 

您可以在您地圖服務帳戶的 [概觀] 頁面查看一些常見的計量。 它目前顯示可選取之持續時間內的 [要求總數]、[錯誤總數] 和 [可用性]。 

![Azure 地圖服務計量概觀](media/how-to-view-api-usage/portal-overview.png)

如果您需要針對特定分析來自訂這些圖表，請繼續閱讀下一節。


## <a name="view-detailed-metrics"></a>檢視詳細計量

1. 在[入口網站](https://portal.azure.com)中登入您的 Azure 訂用帳戶。 

2. 按一下左側的 [所有資源] 功能表項目，並瀏覽到您的 [Azure 地圖服務帳戶]。

3. 您的地圖服務帳戶開啟之後，按一下左邊的 [計量] 功能表。

4. 在 [計量] 窗格上，選擇下列其中一個項目：

    1. [可用性] 顯示持續時間內 API 可用性的「平均值」，或 
    2. [使用量] 顯示您帳戶的使用量「計數」為何。 

    ![Azure 地圖服務計量窗格](media/how-to-view-api-usage/portal-metrics.png)

5. 選擇計量之後，您可以透過選取 [過去 12 小時 (自動)] \(預設值\) 來選取 [時間範圍]。 在同一個下拉式功能表中，您也可以選取 [時間細微性]，以及選擇以 [當地] 或 [GMT] 顯示時間。 按一下 [套用]。

    ![Azure 地圖服務計量時間範圍](media/how-to-view-api-usage/time-range.png)
 
6. 新增您的計量之後，接著您可以從和該計量相關的屬性間 [新增篩選]，然後針對該屬性選擇您想要在圖表看到的值。 

    ![Azure 地圖服務計量篩選器](media/how-to-view-api-usage/filter.png)

7. 您也可以根據所選的計量屬性 [套用分割]。 這樣可讓圖表分割成多個圖表，即該屬性的每個值都會有一個圖表。 例如在以下圖片中，每個圖表的顏色都對應到顯示在圖表底部的屬性值。

    ![Azure 地圖服務計量分割](media/how-to-view-api-usage/splitting.png)
 
8. 您也可以在同一個圖表上觀察多個計量，只要按一下頂端的 [新增計量] 按鈕即可。


## <a name="next-steps"></a>後續步驟

現在您已了解如何追蹤 Azure 地圖服務使用量，您可以繼續了解更多可搭配使用的 API：

* [Azure 地圖服務 REST API 文件](https://docs.microsoft.com/rest/api/maps) \(英文\)
