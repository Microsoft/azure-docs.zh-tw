---
title: 在 Azure 串流分析中處理事件順序和延遲
description: 本文說明串流分析如何處理資料流中順序錯亂或延遲的事件。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/20/2017
ms.openlocfilehash: f0ee486d9ff4c05269da23866edad281aa627889
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37113889"
---
# <a name="azure-stream-analytics-event-order-considerations"></a>Azure 串流分析事件的順序考量

## <a name="arrival-time-and-application-time"></a>抵達時間與應用時間

在事件的時態性資料流中，每個事件都有指派的時間戳記。 Azure 串流分析會使用抵達時間或應用時間，將時間戳記指派給每個事件。 **System.Timestamp** 資料行有指派給事件的時間戳記。 

事件到達來源時，系統會為輸入來源指派抵達時間。 您可以使用事件中樞輸入的 **EventEnqueuedUtcTime** 屬性、IoT 中樞的 **IoTHub.EnqueuedTime** 屬性，以及使用 Blob 輸入的 [BlobProperties.LastModified](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.blob.blobproperties.lastmodified?view=azurestorage-8.1.3) 屬性來存取抵達時間。 

產生事件時會指派應用時間，且其為承載的一部分。 若要按照應用時間處理事件，請在選取查詢中使用 **Timestamp by** 子句。 如果 **Timestamp by** 子句不存在，將按照抵達時間處理事件。 

Azure 串流分析會依時間戳記順序產生輸出，並提供處理順序錯亂資料的設定。


## <a name="handling-of-multiple-streams"></a>處理多個串流

在某些情況下，Azure 串流分析工作會結合來自多個時間軸的事件，這些情況包括：

* 從多個分割區產生輸出。 沒有明確 **Partition by PartitionId** 的查詢必須結合來自所有分割區的事件。
* 兩個以上不同輸入來源的聯集。
* 聯結輸入來源。

在結合多個時間軸的案例中，Azure 串流分析只會在結合的所有來源都至少位於時間 t1 之後，才會針對時間戳記 t1 產生輸出。 例如，假設查詢是從具有兩個分割區的事件中樞分割區讀取。 其中一個分割區 P1 直到時間 t1 之前具有事件。 另一個分割區 P2 直到時間 t1 + x 之前具有事件。 然後到時間 t1 會產生輸出。 但是如果有明確的 **Partition by PartitionId** 子句，則兩個分割區會單獨進行。

延遲傳入容錯設定是用來處理某些分割區中資料不存在的問題。

## <a name="configuring-late-arrival-tolerance-and-out-of-order-tolerance"></a>設定延遲傳入容錯和順序錯亂容錯
未依照順序的輸入串流為以下兩者其中之一：
* 已分類 (並因此延遲)
* 已由系統調整 (根據使用者指定的原則)

依應用時間處理時，串流分析可容錯延遲和順序錯亂的事件。 下列設定位於 Azure 入口網站的 [事件順序] 選項中： 

![串流分析事件處理](media/stream-analytics-event-handling/stream-analytics-event-handling.png)

### <a name="late-arrival-tolerance"></a>延遲傳入容錯
按照應用時間處理時，僅適用延遲傳入容錯。 否則，會忽略設定。

延遲傳入容錯是抵達時間與應用時間之間的最大差異。 如果事件抵達晚於延遲傳入容錯 (例如，應用時間 app_t < 抵達時間 arr_t - 延遲傳入原則容錯 late_t)，事件會調整為延遲傳入容錯的上限 (arr_t  -  late_t)。 延遲傳入時間是事件產生與輸入來源收到事件之間的延遲上限。 

合併相同輸入資料流的多個分割區或多個輸入資料流時，延遲傳入容錯是每個分割區等候新資料的時間上限。 

根據先發生的延遲傳入容錯來調整。 根據接下來發生的順序錯亂容錯來調整。 **System.Timestamp** 資料行有指派給事件的最後時間戳記。

### <a name="out-of-order-tolerance"></a>順序錯亂容錯
傳入順序不正確但在順序錯亂容錯時間設定內的事件，會透過時間戳記重新排序。 傳入時間比容錯範圍還晚的事件，會遭到捨棄或調整：
* **調整**︰調整為看起來是在最低可接受時間抵達。 
* **捨棄**︰放棄。

當串流分析重新排序順序錯亂容錯時間內收到的事件時，查詢的輸出將受到順序錯亂容錯時間所延遲。

### <a name="early-events"></a>早期事件
按照應用時間處理時，系統會根據選取的組態選項，捨棄或調整其應用時間領先其抵達時間 5 分鐘以上的事件。

### <a name="example"></a>範例

* 延遲抵達容許 = 10 分鐘<br/>
* 順序錯亂容許 = 3 分鐘<br/>
* 按照應用時間處理<br/>
* 事件：
   1. **應用時間** = 00:00:00，**傳入時間** = 00:10:01，**System.Timestamp** = 00:00:01，已調整，因為 (**傳入時間 - 應用時間**) 大於延遲傳入容錯。
   2. **應用時間** = 00:00:01，**傳入時間** = 00:10:01，**System.Timestamp** = 00:00:01，未調整，因為應用時間在延遲傳入時間內。
   3. **應用時間** = 00:10:00，**傳入時間** = 00:10:02，**System.Timestamp** = 00:10:00，未調整，因為應用時間在延遲傳入時間內。
   4. **應用時間** = 00:09:00，**傳入時間** = 00:10:03，**System.Timestamp** = 00:09:00，已接受原始時間戳記，因為應用時間在順序錯亂容錯內。
   5. **應用時間** = 00:06:00，**傳入時間** = 00:10:04，**System.Timestamp** = 00:07:00，已調整，因為應用時間早於順序錯亂容錯。

### <a name="practical-considerations"></a>實際考量
如上所述，[延遲傳入容錯] 是應用時間與傳入時間之間的最大差異。 當您依應用時間處理時，在套用順序錯亂容錯設定之前，系統先對比已設定的延遲傳入容錯還要晚的事件進行調整。 因此，有效的順序錯亂，是延遲傳入容錯和順序錯亂容錯的最小值。

串流內順序錯亂事件的原因包括：
* 傳送者之間的時鐘誤差。
* 傳送者與輸入事件來源之間的變數延遲。

延遲傳入的原因包括：
* 傳送者在間隔之後針對稍後的間隔對事件進行批次及傳送。
* 傳送者傳送事件以及輸入來源接收到事件之間的延遲。

針對特定的作業設定延遲傳入容錯和順序錯亂容錯時，應考慮正確性、延遲需求及上述因素。

以下是一些範例。

#### <a name="example-1"></a>範例 1 
查詢具有 **Partition by PartitionId** 子句。 在單一分割區內，事件會透過同步傳送方法進行傳送。 同步傳送方法會封鎖，直到事件傳送出去為止。

在此情況下，順序錯亂為零，因為在傳送下個事件之前，事件會搭配明確確認依序傳送。 延遲傳入是產生事件及傳送事件之間的延遲上限，以及傳送者與輸入來源之間的延遲上限。

#### <a name="example-2"></a>範例 2
查詢具有 **Partition by PartitionId** 子句。 在單一分割區內，事件會透過非同步傳送方法進行傳送。 非同步傳送方法可以同時起始多個傳送，這可能會導致順序錯亂事件。

在此情況下，順序錯亂和延遲傳入都至少是產生事件及傳送事件之間的延遲上限，加上傳送者與輸入來源之間的延遲上限。

#### <a name="example-3"></a>範例 3
查詢中不含 **Partition by PartitionId**子句，且至少有兩個分割區。

設定與範例 2 相同。 不過，在其中一個分割區中缺少資料的情況下，可能會因額外的延遲傳入容錯時間而延遲輸出。

## <a name="handling-event-producers-with-differing-timelines-with-substreams"></a>處理具有不同時間軸 (具有「子串流」) 的事件產生者
單一輸入事件串流通常包含來自多個事件產生者的事件，例如個別裝置。 這些事件可能會由於先前討論的原因而傳入順序錯亂。 在這些情況下，雖然跨事件產生者的失序可能很大，單一生產者事件內的失序則是小的 (或甚至不存在)。

Azure 串流分析提供一般機制以處理順序錯亂事件。 此類機制會導致處理延遲 (等候紊亂的事件抵達系統)、已捨棄或已調整的事件，或是兩者。

然而在許多情況下，想要的查詢會個別處理事件和不同的事件產生者。 例如，它可能會彙總每個時間、每個裝置的事件。 在這些情況下，在等候其他事件產生者趕上時，無須延遲對應到某個事件產生者的輸出。 換句話說，不需要處理產生者之間的時間誤差。 您可以忽略它。

當然，這表示輸出事件本身與其時間戳記的順序是錯亂的。 下游取用者必須能夠處理這類行為。 但是輸出中每一個事件是正確的。

Azure 串流分析會使用 [TIMESTAMP BY OVER](https://msdn.microsoft.com/library/azure/mt573293.aspx) 子句來實作這項功能。

## <a name="summary"></a>總結
* 根據正確性和延遲需求來設定延遲傳入容錯和順序錯亂時間。 也請考慮如何傳送事件。
* 建議將順序錯亂容錯設定為小於延遲傳入容錯。
* 結合多個時間軸時，在其中一個來源或分割區中缺少資料的情況下，可能會因額外的延遲傳入容錯時間而延遲輸出。

## <a name="get-help"></a>取得說明
如需其他協助，請參閱我們的 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟
* [串流分析介紹](stream-analytics-introduction.md)
* [開始使用串流分析](stream-analytics-real-time-fraud-detection.md)
* [調整串流分析作業](stream-analytics-scale-jobs.md)
* [串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
