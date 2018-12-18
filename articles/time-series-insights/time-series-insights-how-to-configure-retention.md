---
title: 如何在您的 Azure 時間序列深入解析環境中設定保留期 | Microsoft Docs
description: 本文說明如何在您的 Azure 時間序列深入解析環境中設定保留期。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 02/09/2018
ms.openlocfilehash: a87a12e08e78cf8a7f0b4d2af3e8029b233cf02b
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629883"
---
# <a name="configuring-retention-in-time-series-insights"></a>在時間序列深入解析中設定保留期
本文說明如何在 Azure 時間序列深入解析中，設定 [資料保留時間] 和 [超過儲存空間限制的行為]。

每個時間序列深入解析 (TSI) 環境都會有能夠設定 [資料保留時間] 的設定。 可設定的值為 1 到 400 天。 資料會依環境儲存容量或保留持續時間 (1-400) 進行刪除，以先達到者為準。

每個 TSI 環境都有一個額外的設定 [超過儲存空間限制的行為]。 此設定可控制當達到環境的容量上限時，要執行的輸入與清除行為。 有兩個行為可供選擇：
- **清除舊資料** (預設)  
- **暫停輸入**

如需詳細資訊以進一步了解這些設定，請檢閱[了解時間序列深入解析中的保留期](time-series-insights-concepts-retention.md)。  

## <a name="configure-data-retention"></a>設定資料保留期

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找出您的現有時間序列深入解析環境。 在 Azure 入口網站左側的功能表中選取 [所有資源]。 選取 Time Series Insights 環境。

3. 在 [設定] 標題之下，選取 [設定]。

4. 選取 [資料保留時間]，即可使用滑桿或在文字方塊中輸入數字以設定保留期。

5. 請注意 [容量] 設定，因為此設定會影響資料事件數量上限和用於儲存資料的總儲存體容量。 

6. 切換 [超過儲存空間限制的行為] 設定。 選取 [清除舊資料] 或 [暫停輸入] 行為。

7. 選取 [儲存] 以設定變更。

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請檢閱[了解時間序列深入解析中的保留期](time-series-insights-concepts-retention.md)。
