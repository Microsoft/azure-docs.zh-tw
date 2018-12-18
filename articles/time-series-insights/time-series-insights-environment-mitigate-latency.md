---
title: 如何在 Azure 時間序列深入解析中監視及降低節流 | Microsoft Docs
description: 本文描述如何監視、診斷和減少會在 Azure 時間序列深入解析中造成延遲和節流的效能問題。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.devlang: csharp
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 11/27/2017
ms.openlocfilehash: e301afaa2fed2b00599fbdde8178203965280c0d
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364976"
---
# <a name="monitor-and-mitigate-throttling-to-reduce-latency-in-azure-time-series-insights"></a>監視與降低節流，以減少 Azure 時間序列深入解析中的延遲
當內送資料的數量超過您的環境設定時，可能會在 Azure 時間序列深入解析中發生延遲或節流。

您可以避免延遲和節流，方法為適當地設定您環境需要分析的資料量。

您在以下情況時，最可能會發生延遲和節流：

- 新增事件來源，其中包含的舊資料可能會超過您的配置輸入率 (時間序列深入解析必須趕上)。
- 將更多事件來源新增至環境，從而導致其他事件高峰 (這可能會超出您環境的容量)。
- 將大量的歷程記錄事件推送至事件來源，從而導致延隔時間 (時間序列深入解析必須趕上)。
- 將參考資料與遙測聯結，從而導致較大的事件大小。  就節流的觀點而言，會將封包大小為 32 KB 的輸入資料封包視為 32 個事件，每個大小為 1 KB。 允許的事件大小上限為 32 KB；大於 32 KB 的資料封包會加以截斷。

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-cover-time-series-insights-data-ingress-behavior-and-how-to-plan-for-itbr"></a>在本影片中，我們將說明時間序列深入解析資料輸入行為以及如何進行規劃。</br>

> [!VIDEO https://www.youtube.com/embed/npeZLAd9lxo]

## <a name="monitor-latency-and-throttling-with-alerts"></a>使用警示來監視延遲和節流

警示有助於您協助診斷並減少您環境所造成的延遲問題。 

1. 在 Azure 入口網站中，按一下 [計量]。 

   ![度量](media/environment-mitigate-latency/add-metrics.png)

2. 按一下 [新增計量警示]。  

    ![新增計量警示](media/environment-mitigate-latency/add-metric-alert.png)

從該處，您可以使用下列計量來設定警示：

|計量  |說明  |
|---------|---------|
|**輸入接收的位元組**     | 從事件來源讀取的未經處理位元組計數。 未經處理的計數通常會包含屬性名稱和值。  |  
|**輸入接收的無效訊息**     | 從所有的 Azure 事件中樞或 Azure IoT 中樞事件來源讀取的無效訊息計數。      |
|**輸入接收的訊息**   | 從所有事件中樞或 IoT 中樞事件來源讀取的訊息計數。        |
|**輸入儲存的位元組**     | 已儲存且可供查詢的事件總大小。 只能計算屬性值的大小。        |
|**輸入儲存的事件**     |   已儲存且可供查詢的壓平合併事件計數。      |
|**輸入接收的訊息時間延遲**    |  訊息在事件來源中加入佇列的時間與在輸入中處理的時間之間的差異 (以秒為單位)。      |
|**輸入接收的訊息計數延遲**    |  事件來源資料分割中最後加入佇列之訊息的序號與輸入中所處理訊息的序號之間的差異。      |


![Latency](media/environment-mitigate-latency/latency.png)

如果您已受到節流處理，將會看到「輸入收到的訊息時間延遲」的值，通知您落後 TSI 有幾秒鐘是來自訊息叫用事件來源的實際時間 (不包括索引時間大約 30-60 秒)。  「輸入收到的訊息計數延遲」也應該有值，讓您可判斷您落後的訊息數目。  最簡單的趕上方式是將您環境的容量增加至可讓您克服差異的大小。  

例如，如果您有單一單位的 S1 環境，並看到有 5 百萬個訊息延遲，您可以將環境的大小增加到大約一天六個單位以便趕上。  您甚至可以更進一步加速趕上。  在一開始佈建環境時，有追趕期是常見的情況，特別是當您將其連線至已經有事件的事件來源，或是當您大量上傳許多歷史資料時。

另一種技術是將 [輸入儲存的事件] 警示設定為 >= 稍微小於 2 小時期間內您總環境容量的臨界值。  此警示可協助您了解是否經常符合容量，表示很有可能會延遲。  

例如，如果您佈建了三個 S1 單位 (或每分鐘輸入容量 2100 個事件)，就可以設定 [輸入儲存的事件] 警示 >= 2 小時 1900 個事件。 如果您經常會超過此臨界值，並因此觸發警示，您就可能佈建不足。  

此外，如果您懷疑已進行節流，可以比較您 [輸入接收的訊息] 與您事件來源的輸出訊息。  如果事件中樞的輸入大於您 [輸入接收的訊息]，時間序列深入解析就可能會受到節流。

## <a name="improving-performance"></a>改善效能 
若要減少節流或發生延遲，最佳的修正方法是增加您環境的容量。 

您可以避免延遲和節流，方法為適當地設定您環境需要分析的資料量。 如需如何將容量新增至您環境的詳細資訊，請參閱[調整您的環境](time-series-insights-how-to-scale-your-environment.md)。

## <a name="next-steps"></a>後續步驟
- 如需其他的疑難排解步驟，請參閱[在 Time Series Insights 環境中診斷並解決問題](time-series-insights-diagnose-and-solve-problems.md)。
- 如需其他協助，請在 [MSDN 論壇](https://social.msdn.microsoft.com/Forums/home?forum=AzureTimeSeriesInsights)或 [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-timeseries-insights) 上啟動交談。 您也可以連絡 [Azure 支援](https://azure.microsoft.com/support/options/)以取得協助的支援選項。
