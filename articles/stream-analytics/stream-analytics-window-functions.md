---
title: Azure 串流分析視窗化函式簡介
description: 本文說明四個用於 Azure 串流分析工作中的時間範圍函式 (輪轉、跳動、滑動、工作階段)。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/07/2018
ms.openlocfilehash: 2650058e277bc0338c779655ce381be046fb120a
ms.sourcegitcommit: 870d372785ffa8ca46346f4dfe215f245931dae1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/08/2018
ms.locfileid: "33893639"
---
# <a name="introduction-to-stream-analytics-windowing-functions"></a>串流分析時間範圍函式簡介
在即時串流案例中，針對時間範圍中內含的資料執行作業是常見的模式。 串流分析具備對時間範圍函式的原生支援，可讓開發人員輕鬆地撰寫複雜的串流處理工作。

您可以選擇四種時間範圍：[**輪轉**](https://msdn.microsoft.com/azure/stream-analytics/reference/tumbling-window-azure-stream-analytics)、[**跳動**](https://msdn.microsoft.com/azure/stream-analytics/reference/hopping-window-azure-stream-analytics)、[**滑動**](https://msdn.microsoft.com/azure/stream-analytics/reference/sliding-window-azure-stream-analytics)以及[**工作階段**](https://msdn.microsoft.com/azure/stream-analytics/reference/session-window-azure-stream-analytics)時間範圍。  您要在串流分析工作的查詢語法子句 [**GROUP BY**](https://msdn.microsoft.com/azure/stream-analytics/reference/group-by-azure-stream-analytics) 中使用時間範圍函式。

所有[時間範圍](https://msdn.microsoft.com/azure/stream-analytics/reference/windowing-azure-stream-analytics)作業都會在時間範圍**結束**時輸出結果。 時間範圍的輸出會是以使用的彙總函式為基礎的單一事件。 此輸出事件會有時間範圍結束的時間戳記，所有時間範圍函式都是以固定長度定義。 

![串流分析時間範圍函式概念](media/stream-analytics-window-functions/stream-analytics-window-functions-conceptual.png)

## <a name="tumbling-window"></a>輪轉時間範圍
輪轉時間範圍函式用於將資料串流分成不同的時間區段並對其執行函式，如下列範例所示。 輪轉時間範圍的主要差異在於它們會重複，不會重疊，而事件不能屬於一個以上的輪轉時間範圍。

![Stream Analytics 輪轉時間範圍](media/stream-analytics-window-functions/stream-analytics-window-functions-tumbling-intro.png)

## <a name="hopping-window"></a>跳動時間範圍
跳動時間範圍函式會在一段固定的時間向前跳動。 簡單將這類函式視為可以重疊的輪轉時間範圍，因此事件可以屬於一個以上的跳動時間範圍結果集。 若要讓跳動時間範圍與輪轉時間範圍一樣，請將躍點大小指定成與時間範圍大小相同。 

![Stream Analytics 跳動時間範圍](media/stream-analytics-window-functions/stream-analytics-window-functions-hopping-intro.png)

## <a name="sliding-window"></a>滑動時間範圍
不同於輪轉或跳動時間範圍，滑動時間範圍函式**只**會在事件發生時產生輸出。 每個時間範圍都至少會有一個事件，而且時間範圍會持續依據 € (epsilon) 向前移動。 如同跳動時間範圍，事件可以屬於一個以上的滑動時間範圍。

![Stream Analytics 滑動時間範圍](media/stream-analytics-window-functions/stream-analytics-window-functions-sliding-intro.png)

## <a name="session-window-preview"></a>工作階段時間範圍 (預覽)
工作階段時間範圍函式會將相近時間送達的事件分組，並將沒有任何資料的時間範圍篩選掉。 它有三個主要參數：逾時、最大持續期限和資料分割索引鍵 (選擇性)。

![Stream Analytics 工作階段時間範圍](media/stream-analytics-window-functions/stream-analytics-window-functions-session-intro.png)

工作階段時間範圍始於第一個事件發生時。 如果另一個事件的發生時間在指定的逾時期間內 (從上次擷取事件開始計算)，則時間範圍會延伸以包含新的事件。 或者，如果逾時期間內沒有任何事件發生，則時間範圍會在逾時之後關閉。

如果在指定的逾時期間內持續發生事件，工作階段時間範圍將會持續延伸，直到達到最大持續期限為止。 最大持續期限的檢查間隔，會設定為與您所指定最大持續期限的大小相同。 例如，如果最大持續期限為 10，則時間範圍超出最大持續期限時的檢查會發生在 t = 0、10、20、30，以此類推。

當提供資料分割索引鍵時，則會依索引鍵將事件分組在一起，且工作階段時間範圍會獨立套用至每個群組。 當您需要針對不同的使用者或裝置使用不同的工作階段時間範圍時，此分割功能非常實用。


## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)

