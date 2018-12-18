---
title: Azure Log Analytics 中的資料擷取時間 | Microsoft Docs
description: 說明會影響 Azure Log Analytics 收集資料延遲度的不同因素。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: tysonn
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2018
ms.author: bwren
ms.openlocfilehash: f40c8ed7eb6bfae958b3b57c4b7d525963ab9741
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46955238"
---
# <a name="data-ingestion-time-in-log-analytics"></a>Log Analytics 中的資料擷取時間
Azure Log Analytics 是 Azure 監視器中的高階資料服務，服務對象為每月需傳送數 TB 資料的上千名客戶。 而在 Log Analytics 收集資料後，資料需要多久時間方能轉為可用狀態，是經常受到詢問的問題。 本文會說明影響這種延遲的不同因素。

## <a name="typical-latency"></a>一般延遲
延遲是指在受監控的系統中建立資料所需的時間，以及轉變為可在 Log Analytics 中進行分析的時間。 將資料內嵌至 Log Analytics 的延遲通常介於 2 到 5 分鐘之間。 任何特定資料的某種延遲會因為下列所述的各種因素而有所不同。


## <a name="factors-affecting-latency"></a>影響延遲的因素
特定資料集的擷取時間總計可以細分成下列高階領域。 

- 代理程式時間：探索事件、收集事件，然後將其傳送至 Log Analytics 擷取點作為記錄檔記錄所需的時間。 在大多數情況下，此流程是由代理程式處理。
- 管線時間：擷取管線來處理記錄檔的時間。 包括剖析事件的屬性，以及可能新增計算的資訊。
- 編列索引的時間：將記錄檔記錄擷取至 Log Analytics 巨量資料存放區所花費的時間。

以下說明出現在該流程中不同延遲的詳細資料。

### <a name="agent-collection-latency"></a>代理程式收集延遲
代理程式和管理解決方案使用不同的策略來收集虛擬機器的資料，這可能會影響延遲。 某些特定範例包括以下內容：

- 立即收集 Windows 事件、syslog 事件和效能計量。 Linux 效能計數器每隔 30 秒輪詢一次。
- 時間戳記變更後，會收集 IIS 記錄和自訂記錄。 而 IIS 記錄檔會因[變換在 IIS 上設定的排程](log-analytics-data-sources-iis-logs.md)而受影響。 
- Active Directory 複寫解決方案每五天執行一次評估，而 Active Directory 評定解決方案每週對 Active Directory 基礎結構執行一次評估。 只有在評估完成後，代理程式才會收集這些記錄。

### <a name="agent-upload-frequency"></a>代理程式上傳頻率
若要確保 Log Analytics 代理程式是輕量型，代理程式會緩衝記錄，並定期將其上傳至 Log Analytics。 上傳頻率在 30 秒到 2 分鐘之間，視資料類型而定。 大部分的資料會在 1 分鐘內上傳。 網路狀況可能會對此資料的延遲產生負面影響，而難以達到 Log Analytics 擷取點。

### <a name="azure-logs-and-metrics"></a>Azure 記錄和計量 
活動記錄資料需要大約 5 分鐘的時間才能在 Log Analytics 中使用。 診斷記錄和計量資料可能需要 1-5 分鐘才能變成可用狀態，視 Azure 服務而定。 然後，記錄需要再花費 30 到 60 秒，而資料計量需要 3 分鐘以傳送至 Log Analytics 擷取點。

### <a name="management-solutions-collection"></a>管理解決方案集合
某些解決方案不會從代理程式收集其資料，而且可能使用會造成額外延遲的收集方法。 某些解決方案會定期收集資料，而不嘗試近乎即時的收集。 特定範例包括以下內容：

- Office 365 解決方案使用 Office 365 管理活動 API 輪詢活動記錄，該 API 目前不提供任何近乎即時的延遲保證。
- 解決方案以每日頻率收集 Windows Analytics 解決方案 (例如，更新合規性) 資料。

請參閱每個解決方案的文件以確定其收集頻率。

### <a name="pipeline-process-time"></a>管理處理程序時間
將記錄檔記錄擷取至 Log Analytics 管道後，會將其寫入暫存儲存體，以確保租用戶隔離，並確保資料不會遺失。 此程序通常會增加 5-15 秒。 某些管理解決方案會實作更繁重的演算法以彙總資料，並在資料流入時獲得見解。 例如，網路效能監控會每 3 分鐘彙總傳入資料，有效地增加 3 分鐘的延遲。 另一個會增加延遲的程序是處理自訂記錄的程序。 在某些情況下，對於代理程式收集自檔案的記錄，此程序可能會增加數分鐘的延遲。

### <a name="new-custom-data-types-provisioning"></a>新的自訂資料類型佈建
從[自訂記錄](../log-analytics/log-analytics-data-sources-custom-logs.md)或[資料收集器 API](../log-analytics/log-analytics-data-collector-api.md) 建立新類型的自訂資料時，系統會建立專用的儲存體容器。 這是一次性的額外負荷，僅在第一次出現此資料類型時發生。

### <a name="surge-protection"></a>突波保護
Log Analytics 的首要任務是確保不會遺失客戶資料，因此系統具有內建的資料突波保護。 這包括緩衝區，以確保即使在巨大的負載下，系統會保持正常運作。 在正常負載下，這些控制項增加不到一分鐘，但是在極端的條件下和故障時，它們可能會增加大量時間，同時確保資料安全。

### <a name="indexing-time"></a>建立索引的時間
每個巨量資料平台在提供分析和進階搜尋功能之間，會達到內建的平衡，而不是提供對資料的即時存取權。 Azure Log Analytics 可讓您對數百萬筆記錄執行強大的查詢，並在幾秒鐘內獲得結果。 這是可能的，因為基礎結構在擷取過程中會大幅轉換資料，並將其儲存在唯一的壓縮結構中。 系統會緩衝資料，直到有足夠的資料可用於建立這些結構。 必須先完成此操作，記錄檔才會出現在搜尋結果中。

當資料量較少，但高資料傳輸率縮短時間時，此程序目前大約需要 5 分鐘。 這似乎違反直覺，但此流程能讓量生產工作量的延遲達到最佳化。



## <a name="checking-ingestion-time"></a>檢查擷取時間
您可以使用**活動訊號**資料表以預估代理程式資料的延遲。 由於每分鐘發送一次活動訊號，因此目前時間與最後一次活動訊號記錄之間的差異最好盡量接近一分鐘。

您可以使用下列查詢以列出電腦的最高延遲時間。

    Heartbeat 
    | summarize IngestionTime = now() - max(TimeGenerated) by Computer 
    | top 50 by IngestionTime asc

 
在大型環境中請使用以下查詢，總結電腦總計不同百分比的延遲。

    Heartbeat 
    | summarize IngestionTime = now() - max(TimeGenerated) by Computer 
    | summarize percentiles(IngestionTime, 50,95,99)



## <a name="next-steps"></a>後續步驟
* 若要了解 Log Analytics，請參閱[服務等級協定 (SLA)](https://azure.microsoft.com/support/legal/sla/log-analytics/v1_1/)。

