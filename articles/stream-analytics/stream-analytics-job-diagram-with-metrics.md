---
title: 在 Azure 串流分析中進行資料導向的偵錯
description: 本文說明如何使用 Azure 入口網站中的作業圖表和計量，針對 Azure 串流分析作業進行疑難排解。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/01/2017
ms.openlocfilehash: 3d50f96f3dea3646bb32a3a42d0248957dabf9f0
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/18/2018
---
# <a name="data-driven-debugging-by-using-the-job-diagram"></a>使用作業圖表進行資料導向偵錯

在 Azure 入口網站中，[監視] 刀鋒視窗上的作業圖表可協助您將作業流程視覺化。 圖表會顯示輸入、輸出和查詢步驟。 您可以使用作業圖表來檢查每個步驟的計量，以便在進行移難排解時更快速地找出問題來源。

## <a name="using-the-job-diagram"></a>使用作業圖表

當串流作業執行時，在 Azure 入口網站中的 [支援+疑難排解] 下選取 [作業圖表]：

![作業圖表與計量 - 位置](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-1.png)

選取每個查詢步驟，可在查詢編輯窗格中看到對應的區段。 步驟的計量圖表會顯示在頁面上的下方窗格中。

![作業圖表與計量 - 基本作業](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-2.png)

若要查看 Azure 事件中樞的分割區，請選取 **. . .** 操作功能表隨即出現。 您也可以看到輸入的合併。

![作業圖表與計量 - 展開分割區](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-3.png)

若只要查看單一分割區的計量圖表，請選取分割區節點。 計量資訊會顯示在頁面底部。

![作業圖表與計量 - 其他計量](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-4.png)

若要查看合併的計量圖表，請選取合併節點。 下列圖表顯示沒有任何事件遭到捨棄或調整。

![作業圖表與計量 - 格線](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-5.png)

若要查看的計量值和時間的詳細資訊，請指向圖表。

![作業圖表與計量 - 暫留](./media/stream-analytics-job-diagram-with-metrics/stream-analytics-job-diagram-with-metrics-portal-6.png)

## <a name="troubleshoot-by-using-metrics"></a>使用計量進行疑難排解

**QueryLastProcessedTime** 計量表示特定步驟收到資料的時間。 藉由觀看拓撲，您可以從輸出處理器回溯的工作看出哪個階段沒有接收資料。 如果步驟沒有取得資料，請移至該步驟之前的查詢步驟。 檢查先前查詢步驟中是否有時間範圍，以及時間分配是否足夠讓其輸出資料。 (請注意，時間範圍會以小時分配。)
 
若前一個查步驟為輸入處理器，使用輸入計量可協助回答下列預定問題。 這可協助您判斷作業是否正在從輸入來源取得資料。 如果查詢已分割，則檢查每個資料分割。
 
### <a name="how-much-data-is-being-read"></a>已讀取多少資料？

*   **InputEventsSourcesTotal** 是讀取的資料單位數目。 例如，Blob 的數目。
*   **InputEventsTotal** 是讀取的事件數目。 此度量適用於每個資料分割。
*   **InputEventsTotal** 是讀取的位元組數目。
*   **InputEventsLastArrivalTime** 會更新每個收到事件的加入佇列時間。
 
### <a name="is-time-moving-forward-if-actual-events-are-read-punctuation-might-not-be-issued"></a>時間是否正在前進？ 若實際事件已讀取，則可能無需加上標點符號。

*   **InputEventsLastPunctuationTime** 指出何時加上了標點符號，使時間能繼續前進。 若未加上標點符號，可能使資料流程遭到封鎖。
 
### <a name="are-there-any-errors-in-the-input"></a>在輸入中是否有任何錯誤？

*   **InputEventsEventDataNullTotal** 是具有 Null 資料的事件計數。
*   **InputEventsSerializerErrorsTotal** 是無法正確還原序列化的事件計數。
*   **InputEventsDegradedTotal** 是有其他還原序列化以外問題的事件計數。
 
### <a name="are-events-being-dropped-or-adjusted"></a>事件遭到捨棄或調整？

*   **InputEventsEarlyTotal** 是在高水位線之前，具有應用程式時間戳記的事件數目。
*   **InputEventsLateTotal** 是在高水位線之後，具有應用程式時間戳記的事件數目。
*   **InputEventsDroppedBeforeApplicationStartTimeTotal** 是在作業開始時間之前已捨棄的事件數目。
 
### <a name="are-we-falling-behind-in-reading-data"></a>讀取資料的速度太慢了嗎？

*   **Input Events Backlogged (Total)** 會告訴您事件中樞及 Azure IoT 中樞輸入還需要讀取多少訊息數量。 當這個數目大於 0 時，表示您的作業無法以資料進來的速度處理資料。 此時，您可能需要增加串流單位數目，並/或確定您的工作可平行處理。 您可以在[查詢平行化作業頁面](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization)參閱相關的詳細資訊。 


## <a name="get-help"></a>取得說明
如需其他協助，請參閱我們的 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。 

## <a name="next-steps"></a>後續步驟
* [串流分析介紹](stream-analytics-introduction.md)
* [開始使用串流分析](stream-analytics-real-time-fraud-detection.md)
* [調整串流分析作業](stream-analytics-scale-jobs.md)
* [串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
