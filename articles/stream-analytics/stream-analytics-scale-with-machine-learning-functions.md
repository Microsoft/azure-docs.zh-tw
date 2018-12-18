---
title: 調整 Azure 串流分析中的 Machine Learning 函式
description: 本文說明如何藉由設定資料分割和串流單位來調整使用 Machine Learning 函式的串流分析作業。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/28/2017
ms.openlocfilehash: 115273086eeb88064c4b179f67d2d400d9f84692
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43696093"
---
# <a name="scale-your-stream-analytics-job-with-azure-machine-learning-functions"></a>使用 Azure Machine Learning 函式調整串流分析作業
設定串流分析作業並透過它執行一些範例資料很簡單。 當我們需要以較高的資料量來執行相同的作業時，我們該怎麼辦？ 我們必須了解如何設定串流分析作業，以便進行調整。 本文件的重點在於使用 Machine Learning 函數調整串流分析作業的特殊層面。 如需有關如何調整串流分析作業的一般資訊，請參閱 [調整作業](stream-analytics-scale-jobs.md)文章。

## <a name="what-is-an-azure-machine-learning-function-in-stream-analytics"></a>什麼是串流分析中的 Azure Machine Learning 函式？
串流分析中的 Machine Learning 函式可如同一般函式呼叫使用於串流分析查詢語言中。 不過，在幕後，函式呼叫實際上是 Azure Machine Learning Web 服務要求。 Machine Learning Web 服務在相同的 Web 服務 API 呼叫中支援「批次處理」多個資料列 (稱之為迷你批次)，以改善整體輸送量。 如需詳細資訊，請參閱[串流分析中的 Azure Machine Learning 函數](https://blogs.technet.microsoft.com/machinelearning/2015/12/10/azure-ml-now-available-as-a-function-in-azure-stream-analytics/)和 [Azure Machine Learning Web 服務](../machine-learning/studio/consume-web-services.md)。

## <a name="configure-a-stream-analytics-job-with-machine-learning-functions"></a>使用 Machine Learning 函式設定串流分析作業
設定串流分析作業的 Machine Learning 函式時，需要考量兩個參數：Machine Learning 函式呼叫的批次大小，以及針對串流分析作業所佈建的串流單元 (SU)。 若要決定這些參數的適當值，必須先決定延遲與輸送量，也就是串流分析作業的延遲，以及每個 SU 的輸送量。 雖然額外的 SU 會增加執行作業的成本，但 SU 一律會新增至作業，以提高妥善分割之串流分析查詢的輸送量。

因此，請務必決定執行串流分析作業的延遲 *容錯* 。 執行 Azure Machine Learning 服務要求所產生的額外延遲會自然地隨著批次大小增加，其複合串流分析作業的延遲。 另一方面，增加批次大小可讓串流分析作業處理更多具有*與 Machine Learning Web 服務要求*數量相同*的事件。 Machine Learning Web 服務延遲增加通常是批次大小增加的次線性，因此在任何特定情況下，請務必針對 Machine Learning Web 服務考慮最符合成本效益的批次大小。 Web 服務要求的預設批次大小是 1000，可以使用[串流分析 REST API](https://msdn.microsoft.com/library/mt653706.aspx "串流分析 REST API") 或[適用於串流分析的 PowerShell 用戶端](stream-analytics-monitor-and-manage-jobs-use-powershell.md "適用於串流分析的 PowerShell 用戶端")進行修改。

決定批次大小後，即可根據函式每秒需要處理的事件數目，決定串流單元 (SU) 的數量。 如需串流單位的詳細資訊，請參閱[串流分析調整作業](stream-analytics-scale-jobs.md)。

一般而言，每 6 個 SU 有 20 個對 Machine Learning Web 服務的同時連線，但是 1 SU 作業和 3 SU 作業也會取得 20 個同時連線。  例如，如果輸入資料速率是每秒 200,000 個事件，且批次大小保留為預設值 1000，則 1000 個事件迷你批次產生的 Web 服務延遲是 200 毫秒。 這表示每個連線可以在 1 秒內向 Machine Learning Web 服務提出五個要求。 透過 20 個連線，串流分析作業可以在 200 毫秒內處理 20,000 個事件，因此在 1 秒內可處理 100,000 個事件。 所以若要每秒處理 200,000 個事件，串流分析作業需要 40 個同時連線，結果出現 12 個 SU。 下圖說明串流分析作業向 Machine Learning Web 服務端點提出的要求︰每 6 個 SU 最多有 20 個對 Machine Learning Web 服務的同時連線。

![使用 Machine Learning 函數二作業範例調整串流分析](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-00.png "使用 Machine Learning 函數式二作業範例調整串流分析")

一般而言，***B*** 代表批次大小，***L*** 代表批次大小 B 的 Web 服務延遲 (以毫秒為單位)，而具有 ***N*** 個 SU 的串流分析作業的輸送量為：

![使用 Machine Learning 函式公式調整串流分析](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-02.png "使用 Machine Learning 函式公式調整串流分析")

另一項考量可能是 Machine Learning web 服務端上的「最大同時呼叫」，建議將此值設定為最大值 (目前為 200)。

如需此設定的詳細資訊，請參閱 [Machine Learning Web 服務的調整文章](../machine-learning/studio/scaling-webservice.md)。

## <a name="example--sentiment-analysis"></a>範例 – 情感分析
以下範例包含具有情感分析 Machine Learning 函式的串流分析作業，如 [串流分析 Machine Learning 整合教學課程](stream-analytics-machine-learning-integration-tutorial.md)所述。

此查詢是一個簡單的完全分割查詢，其後接著 **sentiment** 函數，如下所示︰

    WITH subquery AS (
        SELECT text, sentiment(text) as result from input
    )

    Select text, result.[Score]
    Into output
    From subquery

請考慮下列案例：如果有每秒 10,000 則推文的輸送量，則必須建立串流分析作業才能執行推文 (事件) 的情緒分析。 使用 1 個 SU，此串流分析作業是否能夠處理此流量？ 使用預設批次大小 1000，此作業應該能夠跟上輸入的速度。 此外，新增的 Machine Learning 函式應產生低於一秒的延遲，也就是情感分析 Machine Learning Web 服務的一般預設延遲 (預設批次大小為 1000)。 串流分析作業的 **整體** 或端對端延遲通常是幾秒鐘的時間。 更深入了解此串流分析作業，尤其是  Machine Learning 函式呼叫。 如果批次大小為 1000，則 10,000 個事件的輸送量大約會取用對 Web 服務的 10 個要求。 即使是 1 個 SU，也有足夠的同時連線可容納此輸入流量。

如果輸入事件速率增加 100 倍，則串流分析作業每秒需要處理 1,000,000 則推文。 有兩個選項可完成這種增加規模：

1. 增加批次大小，或
2. 分割輸入串流以平行方式處理事件

使用第一個選項時，會增加作業**延遲**。

使用第二個選項時，需要佈建更多 SU，因而產生更多同時 Machine Learning Web 服務要求。 這表示作業**成本**會增加。

假設情感分析 Machine Learning Web 服務的延遲如下：1000 個或以下的事件批次延遲 200 毫秒、5,000 個事件批次延遲 250 毫秒、10,000 個事件批次延遲 300 毫秒，或 25,000 個事件批次延遲 500 毫秒。

1. 使用第一個選項 (**不要**佈建更多 SU)。 批次大小可以增加到 **25,000**。 然而，這可讓作業利用對 Machine Learning Web 服務的 20 個同時連線來處理 1,000,000 個事件 (每次呼叫的延遲為 500 毫秒)。 所以因為對 Machine Learning Web 服務要求的情感函式要求而產生的額外串流分析作業延遲會從 **200 毫秒**增加至 **500 毫秒**。 不過，批次大小**無法**無限增加，因為 Machine Learning Web 服務所需的要求承載大小是 4 MB，否則較小的 Web 服務要求會作業 100 秒後逾時。
2. 使用第二個選項，批次大小會保留為 1000、Web 服務延遲為 200 毫秒，則每 20 個對 Web 服務的同時連線就能夠處理 1000 * 20 * 5 個事件 = 每秒 100,000。 因此若要每秒處理 1,000,000 個事件，作業會需要 60 SU。 相較於第一個選項，串流分析作業會產生更多 Web 服務批次要求，進而使成本提高。

下表是不同 SU 和批次大小的串流分析作業輸送量 (以每秒的事件數目表示)。

| 批次大小 (ML 延遲) | 500 (200 毫秒) | 1,000 (200 毫秒) | 5,000 (250 毫秒) | 10,000 (300 毫秒) | 25,000 (500 毫秒) |
| --- | --- | --- | --- | --- | --- |
| **1 SU** |2,500 |5,000 |20,000 |30,000 |50,000 |
| **3 SU** |2,500 |5,000 |20,000 |30,000 |50,000 |
| **6 SU** |2,500 |5,000 |20,000 |30,000 |50,000 |
| **12 SU** |5,000 |10,000 |40,000 |60,000 |100,000 |
| **18 SU** |7,500 |15,000 |60,000 |90,000 |150,000 |
| **24 SU** |10,000 |20,000 |80,000 |120,000 |200,000 |
| **…** |… |… |… |… |… |
| **60 SU** |25,000 |50,000 |200,000 |300,000 |500,000 |

現在，您應該已充分了解串流分析中 Machine Learning 函式的運作方式。 您可能也了解串流分析作業會從資料來源「提取」資料，而且每次「提取」會傳回一批事件以供串流分析作業處理。 此提取模型如何影響 Machine Learning Web 服務要求？

一般來說，我們為 Machine Learning 函式設定的批次大小不會正好可被每次串流分析作業「提取」所傳回的事件數目除盡。 發生此狀況時，會以「部分」的批次呼叫 Machine Learning Web 服務。 這麼做是為了不會在結合各提取的事件時，導致額外的作業延遲負擔。

## <a name="new-function-related-monitoring-metrics"></a>新的函式相關監視計量
在串流分析作業的 [監視] 區域中，已加入三個額外的函式相關計量。 分別是 [函式要求]、[函式事件] 和 [失敗的函式要求]，如下圖所示。

![使用 Machine Learning 函式計量調整串流分析](./media/stream-analytics-scale-with-ml-functions/stream-analytics-scale-with-ml-functions-01.png "使用 Machine Learning 函式計量調整串流分析")

其定義如下：

**函式要求**︰函式要求的數目。

**函式事件**︰函式要求中的事件數目。

**失敗的函式要求**︰失敗的函式要求數目。

## <a name="key-takeaways"></a>重要心得
若要彙整重點，以便使用 Machine Learning 函數調整串流分析作業，必須考量下列項目︰

1. 輸入事件速率
2. 執行中串流分析作業容許的延遲 (和 Machine Learning Web 服務要求的批次大小)
3. 佈建的串流分析 SU 和 Machine Learning Web 服務要求數目 (額外的函式相關成本)

以完全分割的串流分析查詢為例。 如果需要更複雜的查詢， [Azure Stream Analytics forum (Azure 串流分析論壇)](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics) 是可向串流分析小組取得其他協助的絕佳資源。

## <a name="next-steps"></a>後續步驟
若要深入了解串流分析，請參閱：

* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
