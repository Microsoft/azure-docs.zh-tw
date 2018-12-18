---
title: Azure 串流分析中需要進行疑難排解的常見問題
description: 本文描述 Azure 串流分析中幾個常見的問題，以及對這些問題進行疑難排解的步驟。
services: stream-analytics
author: jasonwhowell
manager: kfile
ms.author: jasonh
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/12/2018
ms.openlocfilehash: d7784792cd7e1475028dc0a5e9664916f1d45a07
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43699214"
---
# <a name="common-issues-in-stream-analytics-and-steps-to-troubleshoot"></a>串流分析的常見問題以及疑難排解的步驟

## <a name="troubleshoot-malformed-input-events"></a>對格式不正確的輸入事件進行疑難排解

 當串流分析作業的輸入資料流包含格式不正確的訊息時，就會導致序列化問題。 例如，格式錯誤的訊息可能是在 JSON 物件中遺漏括號或遺漏大括號所導致，或是時間欄位中不正確的時間戳記格式所導致。 
 
 當串流分析作業從輸入收到格式不正確的訊息時，會捨棄訊息並以警告通知使用者。 警告符號會顯示在串流分析作業的 [輸入] 圖格 (只要作業處於執行中狀態，此警告符號就會一直存在)：

![[輸入] 圖格](media/stream-analytics-malformed-events/inputs_tile.png)

若要參閱詳細資訊，請啟用診斷記錄以檢視警告的詳細資料。 對於格式錯誤的輸入事件，執行記錄包含具有類似以下訊息的項目：「訊息：無法從資源 <blob URI> 將輸入事件還原序列化為 json」。 

### <a name="troubleshooting-steps"></a>疑難排解步驟

1. 瀏覽至 [輸入] 圖格，然後按一下以檢視警告。

2. 輸入詳細資料圖格會顯示一組警告，以及問題的相關詳細資料。 以下是警告訊息範例，警告訊息會顯示發生 JSON 資料格式不正確的磁碟分割、位移及序號。 

   ![位移的警告訊息](media/stream-analytics-malformed-events/warning_message_with_offset.png)

3. 若要取得具有不正確格式的 JSON 資料，請執行 CheckMalformedEvents.cs 程式碼。 您可以從 [GitHub 範例存放庫](https://github.com/Azure/azure-stream-analytics/tree/master/Samples/CheckMalformedEventsEH)取得此範例。 此程式碼讀取分割區識別碼、位移，並列印位於該位移的資料。 

4. 一旦您讀取資料後，可以分析並修正序列化格式。

5. 您也可以[透過 Service Fabric Explorer 從 IoT 中樞讀取事件](https://code.msdn.microsoft.com/How-to-read-events-from-an-1641eb1b)。

## <a name="delayed-output"></a>延遲的輸出

### <a name="first-output-is-delayed"></a>第一個輸出就延遲
串流分析作業啟動時，會讀取輸入事件，但在某些情況下會發生輸出延遲。

時態性查詢元素中的較長時間值可能會造成輸出延遲。 若要針對較長時間範圍產生正確的輸出，串流作業要先讀取最近時間 (最多七天前) 的資料，以填滿時間範圍。 在這段時間，在未處理輸入事件的讀取作業趕上之前，都不會產生任何輸出。 當系統升級串流作業，因此而重新啟動作業時，就會出現此問題。 這類升級通常每隔幾個月會發生一次。 

因此，在設計您的串流分析查詢時請慎重考慮。 如果您在作業的查詢語法中針對時態性元素使用較長的時間範圍 (超過幾小時，最多七天)，當作業啟動或重新啟動時，可能會在第一個輸出就延遲。  

針對第一個輸出延遲這類型的一個緩和策略，是使用查詢平行化技術 (分割資料)，或新增更多串流單位來改善輸送量，直到作業趕上為止。  如需詳細資訊，請參閱[建立串流分析作業時的考量](stream-analytics-concepts-checkpoint-replay.md)

這些因素會影響所產生的第一個輸出的時效性：

1. 使用視窗型彙總 (輪轉視窗、跳動視窗和滑動視窗的 GROUP BY)
   - 對於輪轉視窗或跳動視窗的彙總，會在視窗時間範圍的結尾產生結果。 
   - 對於滑動視窗，當事件進入或離開滑動視窗時就會產生結果。 
   - 如果您打算使用較大的視窗大小 (> 1 小時)，最好選擇跳動視窗或滑動視窗，讓您能夠更頻繁地查看輸出。

2. 使用時態性聯結 (JOIN 搭配 DATEDIFF)
   - 只要相符事件的兩邊都達成時，就會產生相符項目。
   - 相對於左邊的每個事件，缺少相符項目 (LEFT OUTER JOIN) 的資料，會在 DATEDIFF 視窗結尾時產生。

3. 使用時態性分析函數 (ISFIRST、LAST 和 LAG 搭配 LIMIT DURATION)
   - 對於分析函數，每個事件都會產生輸出，不會有任何延遲。

### <a name="output-falls-behind"></a>輸出落後
在作業的一般作業期間，如果您發現作業的輸出落後 (延遲越來越長)，便可以檢查下列因素找出根本原因：
- 下游接收是否已節流
- 上游來源是否已節流
- 查詢中的處理邏輯是否需要大量計算

若要查看這些詳細資料，請在 Azure 入口網站中，選取串流作業，然後選取 [工作圖表]。 每個輸入都有一個按照分割區區分的待辦項目事件計量。 如果待辦項目事件計量持續增加，則表示系統資源受到限制。 可能是因為有輸出接收節流，或 CPU 很高。 如需使用作業圖表的詳細資訊，請參閱[使用作業圖表進行資料導向偵錯](stream-analytics-job-diagram-with-metrics.md)。

## <a name="handle-duplicate-records-in-azure-sql-database-output"></a>Azure SQL Database 輸出中的控制代碼重複記錄

當您設定 Azure SQL Database 作為串流分析作業的輸出時，它會將記錄大量插入至目的地資料表。 一般情況下，Azure 串流分析保證[至少一次傳遞]( https://msdn.microsoft.com/azure/stream-analytics/reference/event-delivery-guarantees-azure-stream-analytics)至輸出接收，當 SQL 資料表已定義唯一限制式時，仍然可以[達到準確一次傳遞]( https://blogs.msdn.microsoft.com/streamanalytics/2017/01/13/how-to-achieve-exactly-once-delivery-for-sql-output/)至 SQL 輸出。 

一旦在 SQL 資料表上設定好唯一的索引鍵限制式，如果有重複的記錄插入至 SQL 資料表，Azure 串流分析就會移除重複的記錄。 它會將資料分割為幾個批次，並以遞迴方式插入批次，直到找到單一重複的記錄。 如果串流作業有相當多的重複資料列，這種分割和插入的程序必須一個一個忽略重複的項目，這樣比較沒有效率而且耗時。 如果您在過去一小時內的活動記錄中看到多個索引鍵違規警告訊息，有可能是 SQL 輸出減緩整個作業。 

若要解決此問題，您應藉由啟用 IGNORE_DUP_KEY 選項來[設定索引]( https://docs.microsoft.com/sql/t-sql/statements/create-index-transact-sql) (造成索引鍵違規的索引)。 啟用此選項可讓 SQL 在大量插入期間略過重複值，且 SQL Azure 只會產生警告訊息而非錯誤。 Azure 串流分析不會再產生主索引鍵違規錯誤。

針對數種類型的索引設定 IGNORE_DUP_KEY 時，請注意下列觀察：

* 您無法在主索引鍵或者使用 ALTER INDEX 的唯一限制式上設定 IGNORE_DUP_KEY，您必須卸除而後重新建立索引。  
* 您可以使用 ALTER INDEX 對唯一索引設定 IGNORE_DUP_KEY 選項，這與使用 CREATE INDEX 或 INDEX 定義建立的主索引鍵/唯一限制式不同。  
* 因為您無法在這類索引上強制執行唯一性，所以 IGNORE_DUP_KEY 不適用於資料行存放區索引。  

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
