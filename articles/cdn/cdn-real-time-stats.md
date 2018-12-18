---
title: Azure CDN 中的即時統計資料 | Microsoft Docs
description: 將內容傳遞給您的用戶端時，即時統計資料可提供關於 Azure CDN 效能的即時資料。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: c7989340-1172-4315-acbb-186ba34dd52a
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: e9b9522de6b2c54dc794b00100ffe358296ecfdd
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22997903"
---
# <a name="real-time-stats-in-microsoft-azure-cdn"></a>Microsoft Azure CDN 中的即時統計資料
[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>概觀
本文件說明 Microsoft Azure CDN 中的即時統計資料。  在將內容傳遞給您的用戶端時，這項功能會提供即時資料 (例如頻寬、快取狀態和並行連線) 給您的 CDN 設定檔。 這可讓您隨時連續監視服務的健全狀況，包括上線事件。

可用圖表如下︰

* [頻寬](#bandwidth)
* [狀態碼](#status-codes)
* [快取狀態](#cache-statuses)
* [連線](#connections)

## <a name="accessing-real-time-stats"></a>存取即時統計資料
1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽到您的 CDN 設定檔。
   
    ![CDN 設定檔刀鋒視窗](./media/cdn-real-time-stats/cdn-profile-blade.png)
2. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [管理]  按鈕。
   
    ![[CDN 設定檔] 刀鋒視窗的 [管理] 按鈕](./media/cdn-real-time-stats/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
3. 將滑鼠移至 [分析] 索引標籤上，然後將滑鼠移至 [即時統計資料] 飛出視窗上。  按一下 [HTTP 大型物件] 。
   
    ![CDN 管理入口網站](./media/cdn-real-time-stats/cdn-premium-portal.png)
   
    隨即會顯示即時統計資料圖表。

每個圖表都會顯示所選時間範圍內的即時統計資料，並在頁面載入時啟動。  圖表每隔幾秒鐘就會自動更新。  [重新整理圖表]  按鈕 (如果有) 將會清除圖表，然後只顯示選取的資料。

## <a name="bandwidth"></a>頻寬
![頻寬圖表](./media/cdn-real-time-stats/cdn-bandwidth.png)

**頻寬** 圖表會顯示所選時間範圍內，針對目前平台所使用的頻寬量。 圖表的陰影部分表示頻寬使用量。 目前使用的確切頻寬量會顯示於線條圖的正下方。

## <a name="status-codes"></a>狀態碼
![狀態碼圖表](./media/cdn-real-time-stats/cdn-status-codes.png)

**狀態碼** 圖表會指出某些 HTTP 回應碼在所選時間範圍內的發生頻率。

> [!TIP]
> 如需每個 HTTP 狀態碼選項的說明，請參閱 [Azure CDN HTTP 狀態碼](https://msdn.microsoft.com/library/mt759238.aspx)。
> 
> 

HTTP 狀態碼的清單會顯示於圖表正上方。 此清單表示每個可包含於線條圖中的狀態碼，以及該狀態碼目前每秒的發生次數。 根據預設，圖表中會針對這些狀態碼的每一個顯示一條線。 不過，您可以選擇只監視對您的 CDN 組態具有特殊意義的狀態碼。 若要這樣做，請檢查所需的狀態碼，並清除所有其他選項，然後按一下 [重新整理圖表] 。 

您可以暫時隱藏特定狀態碼的記錄資料。  從圖表正下方的圖例中，按一下您想要隱藏的狀態碼。 狀態碼將會立即從圖表中隱藏。 再次按一下該狀態碼將導致該選項再次顯示。

## <a name="cache-statuses"></a>快取狀態
![快取狀態圖表](./media/cdn-real-time-stats/cdn-cache-status.png)

**快取狀態** 圖表會指出某些類型的快取狀態在所選時間範圍內的發生頻率。 

> [!TIP]
> 如需每個快取狀態碼選項的說明，請參閱 [Azure CDN 快取狀態碼](https://msdn.microsoft.com/library/mt759237.aspx)。
> 
> 

快取狀態碼的清單會顯示於圖表正上方。 此清單表示每個可包含於線條圖中的狀態碼，以及該狀態碼目前每秒的發生次數。 根據預設，圖表中會針對這些狀態碼的每一個顯示一條線。 不過，您可以選擇只監視對您的 CDN 組態具有特殊意義的狀態碼。 若要這樣做，請檢查所需的狀態碼，並清除所有其他選項，然後按一下 [重新整理圖表] 。 

您可以暫時隱藏特定狀態碼的記錄資料。  從圖表正下方的圖例中，按一下您想要隱藏的狀態碼。 狀態碼將會立即從圖表中隱藏。 再次按一下該狀態碼將導致該選項再次顯示。

## <a name="connections"></a>連線
![連線圖表](./media/cdn-real-time-stats/cdn-connections.png)

此圖表會指出已為 Edge Server 建立的連線數。 通過 CDN 的每個資產要求都會導致建立一個連線。

## <a name="next-steps"></a>後續步驟
* 透過 [Azure CDN 中的即時警示](cdn-real-time-alerts.md)
* 進一步了解 [進階 HTTP 報告](cdn-advanced-http-reports.md)
* 分析 [使用模式](cdn-analyze-usage-patterns.md)

