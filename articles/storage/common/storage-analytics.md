---
title: 使用 Azure 儲存體分析收集記錄檔和計量資料 | Microsoft Docs
description: 儲存體分析可讓您追蹤所有儲存體服務的度量資料，以及收集 Blob、佇列和資料表儲存體的記錄檔。
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 03/03/2017
ms.author: rogarana
ms.component: common
ms.openlocfilehash: a99375ae961e9239e5e8ea86db8b1b9b002b10c8
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526958"
---
# <a name="storage-analytics"></a>儲存體分析

Azure 儲存體分析會執行記錄，並提供儲存體帳戶的度量資料。 您可以使用此資料來追蹤要求、分析使用量趨勢，以及診斷儲存體帳戶的問題。

若要使用儲存體分析，您必須針對想要監視的每個服務個別啟用它。 您可以從 [Azure 入口網站](https://portal.azure.com)加以啟用。 如需詳細資訊，請參閱 [在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)。 您也可以利用程式設計方式，透過 REST API 或用戶端程式庫來啟用儲存體分析。 針對每個服務，使用[取得 Blob 服務屬性](https://msdn.microsoft.com/library/hh452239.aspx)、[取得佇列服務屬性](https://msdn.microsoft.com/library/hh452243.aspx)、[取得表格服務屬性](https://msdn.microsoft.com/library/hh452238.aspx)，以及[取得檔案服務屬性](https://msdn.microsoft.com/library/mt427369.aspx)作業來啟用儲存體分析。

彙總的資料會儲存於已知的 Blob (用於記錄) 和已知的資料表 (用於度量) 中，您可以使用 Blob 服務和資料表服務 API 來存取。

儲存體分析在儲存的資料量上有 20 TB 的限制，但此限制與儲存體帳戶的總限制無關。 如需計費和資料保留原則的詳細資訊，請參閱 [儲存體分析和計費](https://msdn.microsoft.com/library/hh360997.aspx)。 如需儲存體帳戶限制的詳細資訊，請參閱 [Azure 儲存體延展性和效能目標](storage-scalability-targets.md)。

如需使用儲存體分析和其他工具來識別、診斷及疑難排解 Azure 儲存體相關問題的深入指南，請參閱 [監視、診斷及疑難排解 Microsoft Azure 儲存體](storage-monitoring-diagnosing-troubleshooting.md)。

## <a name="about-storage-analytics-logging"></a>關於儲存體分析記錄
儲存體分析會記錄對儲存體服務之成功和失敗要求的詳細資訊。 這項資訊可用來監視個別要求，並診斷儲存體服務的問題。 系統會以最佳方式來記錄要求。

只有在發生儲存體服務活動時，才會建立記錄檔項目。 例如，如果儲存體帳戶在其 Blob 服務中有活動，而不是在其資料表或佇列服務中，則只會建立關於 Blob 服務的記錄檔。

儲存體分析記錄不適用於 Azure 檔案。

### <a name="logging-authenticated-requests"></a>記錄驗證要求
系統將記錄下列類型的驗證要求：

* 成功的要求。
* 失敗的要求，包括逾時、節流、網路、授權和其他錯誤。
* 使用共用存取簽章 (SAS) 的要求，包括失敗和成功的要求。
* 分析資料的要求。

系統不會記錄儲存體分析本身所提出的要求 (例如，記錄檔的建立或刪除)。 記錄資料的完整清單記錄於[儲存體分析記錄作業和狀態訊息](https://msdn.microsoft.com/library/hh343260.aspx)及[儲存體分析記錄檔格式](https://msdn.microsoft.com/library/hh343259.aspx)主題中。

### <a name="logging-anonymous-requests"></a>記錄匿名要求
系統將記錄下列類型的匿名要求：

* 成功的要求。
* 伺服器錯誤。
* 用戶端與伺服器的逾時錯誤。
* 失敗的 GET 要求，錯誤碼為 304 (未修改)。

系統不會記錄所有其他失敗的匿名要求。 記錄資料的完整清單記錄於[儲存體分析記錄作業和狀態訊息](https://msdn.microsoft.com/library/hh343260.aspx)及[儲存體分析記錄檔格式](https://msdn.microsoft.com/library/hh343259.aspx)主題中。

### <a name="how-logs-are-stored"></a>記錄檔的儲存方式
所有記錄檔都會儲存在名為 $logs 的容器內的區塊 Blob 中，該容器是在針對儲存體帳戶啟用儲存體分析時自動建立的。 $logs 容器位於儲存體帳戶的 Blob 命名空間中，例如： `http://<accountname>.blob.core.windows.net/$logs`。 一旦啟用儲存體分析之後便無法刪除此容器，不過您可以刪除其內容。

> [!NOTE]
> $logs 容器不會在執行容器列示作業時顯示，例如 [ListContainers](https://msdn.microsoft.com/library/azure/dd179352.aspx) 方法。 您必須直接存取它。 例如，您可以使用 [ListBlobs](https://msdn.microsoft.com/library/azure/dd135734.aspx) 方法來存取 `$logs` 容器中的 Blob。
> 記錄要求時，儲存體分析將會以區塊形式上傳中繼結果。 儲存體分析會定期認可這些區塊，並提供它們做為 Blob。
> 
> 

在同一個小時內建立的記錄檔可能會有重複的記錄。 您可以藉由檢查 **RequestId** 和 **Operation** 數字來判斷記錄是否重複。

### <a name="log-naming-conventions"></a>記錄檔命名慣例
每個記錄檔的寫入格式如下。

    <service-name>/YYYY/MM/DD/hhmm/<counter>.log

下表描述記錄檔名稱中的每個屬性。

| 屬性 | 說明 |
| --- | --- |
| <service-name> |儲存體服務的名稱。 例如：Blob、資料表或佇列。 |
| YYYY |適用於記錄檔的四位數年份。 例如：2011。 |
| MM |適用於記錄檔的二位數月份。 例如：07。 |
| DD |適用於記錄檔的二位數月份。 例如：07。 |
| hh |24 小時制 UTC 格式的二位數小時，表示記錄檔的開始小時 例如：18。 |
| mm |表示記錄檔之開始分鐘的二位數字。 目前的儲存體分析版本中不支援此值，其值一律為 00。 |
| <counter> |以零起始的六位數計數器，表示在一小時內針對儲存體服務產生的記錄檔 Blob 數目。 此計數器會從 000000 開始。 例如：000001。 |

以下是結合上述範例的完整記錄檔名稱範例。

    blob/2011/07/31/1800/000001.log

以下是可以用來存取上述記錄檔的 URI 範例。

    https://<accountname>.blob.core.windows.net/$logs/blob/2011/07/31/1800/000001.log

記錄儲存體要求時，產生的記錄檔名稱會與完成所要求之作業的小時相互關聯。 例如，如果 GetBlob 要求已在 2011 年 7 月 31 日下午 6:30 完成，則會使用下列前置詞寫入記錄檔︰`blob/2011/07/31/1800/`

### <a name="log-metadata"></a>記錄中繼資料
所有的記錄檔 Blob 都會與中繼資料一同儲存，可用來識別 Blob 包含哪些記錄資料。 下表描述每個中繼資料屬性。

| 屬性 | 說明 |
| --- | --- |
| LogType |描述記錄檔是否包含關於讀取、寫入或刪除作業的資訊。 此值可包含一個類型或是所有這三種類型的組合 (以逗號分隔)。 範例 1：write；範例 2：read,write；範例 3：read,write,delete。 |
| StartTime |記錄檔中項目的最早時間，格式為 YYYY-MM-DDThh:mm:ssZ。 例如：2011-07-31T18:21:46Z。 |
| EndTime |記錄檔中項目的最晚時間，格式為 YYYY-MM-DDThh:mm:ssZ。 例如：2011-07-31T18:22:09Z。 |
| LogVersion |記錄檔格式的版本。 目前唯一支援的值為 1.0。 |

下列清單顯示使用上述範例的完整中繼資料範例。

* LogType=write
* StartTime=2011-07-31T18:21:46Z
* EndTime=2011-07-31T18:22:09Z
* LogVersion=1.0

### <a name="accessing-logging-data"></a>存取記錄資料
`$logs` 容器中的所有資料都可以使用 Blob 服務 API 來存取，包括由 Azure 管理的程式庫所提供的 .NET API。 儲存體帳戶管理員可以讀取和刪除記錄檔，但無法建立或更新它們。 查詢記錄檔時，可以使用記錄檔的中繼資料和記錄檔名稱。 指定小時內的記錄檔可能不會按照順序出現，但是中繼資料一律會在記錄檔中指定記錄檔項目的時間範圍。 因此，您可以在搜尋特定記錄檔時使用記錄檔名稱和中繼資料的組合。

## <a name="about-storage-analytics-metrics"></a>關於儲存體分析度量
儲存體分析可以儲存包含與儲存體服務要求相關之彙總的交易統計資料及容量資料的度量。 報告的交易是屬於 API 作業層級以及儲存體服務層級，而報告的容量則是屬於儲存體服務層級。 度量資料可用來分析儲存體服務使用量、診斷針對儲存體服務提出之要求的問題，以及提升使用服務的應用程式效能。

若要使用儲存體分析，您必須針對想要監視的每個服務個別啟用它。 您可以從 [Azure 入口網站](https://portal.azure.com)加以啟用。 如需詳細資訊，請參閱 [在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)。 您也可以利用程式設計方式，透過 REST API 或用戶端程式庫來啟用儲存體分析。 使用 [取得服務屬性]  作業，為各服務啟用儲存體分析。

### <a name="transaction-metrics"></a>交易度量
系統會以每小時或每分鐘的時間間隔，為每個儲存體服務和要求的 API 作業記錄完善的資料集，其中包括輸入流量/輸出流量、可用性、錯誤，以及已分類的要求百分比。 您可以在 [儲存體分析度量資料表結構描述](https://msdn.microsoft.com/library/hh343264.aspx) 主題中查看完整的交易詳細資料清單。

記錄的交易資料屬於兩個層級 - 服務層級和 API 作業層級。 在服務層級，系統每小時都會將彙總所有要求的 API 作業的統計資料寫入資料表實體，即使未對該服務提出任何要求也一樣。 在 API 作業層級，只有在該小時內對該作業提出要求時，才會將統計資料寫入實體。

例如，如果您在 Blob 服務上執行 **GetBlob** 作業，儲存體分析度量將會記錄要求，然後將它納入 Blob 服務和 **GetBlob** 作業的彙總資料中。 但是，如果在該小時內並未要求任何 **GetBlob** 作業，將不會針對該作業將任何實體寫入 `$MetricsTransactionsBlob` 中。

系統會針對使用者要求及儲存體分析本身所提出的要求記錄交易度量。 例如，會記錄儲存體分析寫入記錄和資料表實體的要求。 如需這些要求之計費方式的詳細資訊，請參閱 [儲存體分析及計費](https://msdn.microsoft.com/library/hh360997.aspx)。

### <a name="capacity-metrics"></a>容量度量
> [!NOTE]
> 容量度量目前僅適用於 Blob 服務。 適用於資料表服務和佇列服務的容量度量將可在儲存體分析的未來版本中使用。
> 
> 

系統每日都會針對儲存體帳戶的 Blob 服務記錄容量資料，並寫入兩個資料表實體。 其中一個實體會提供使用者資料的統計資料，而另一個實體則會提供有關儲存體分析所使用之 `$logs` Blob 容器的統計資料。 `$MetricsCapacityBlob` 資料表包含下列統計資料：

* **Capacity**：儲存體帳戶之 Blob 服務所使用的儲存體數目 (以位元組為單位)。
* **ContainerCount**：儲存體帳戶之 Blob 服務中的 Blob 容器數目。
* **ObjectCount**：儲存體帳戶之 Blob 服務中的認可及未認可區塊或分頁 Blob 的數目。

如需容量度量的詳細資訊，請參閱 [儲存體分析度量資料表結構描述](https://msdn.microsoft.com/library/hh343264.aspx)。

### <a name="how-metrics-are-stored"></a>度量的儲存方式
對於每個儲存體服務的所有度量資料均儲存於為該服務所保留的三個資料表中：一個資料表用於交易資訊、一個資料表用於每分鐘交易資訊，而另一個資料表則用於容量資訊。 交易和每分鐘交易資訊都是由要求和回應資料所組成，而容量資訊是由儲存體使用量資料所組成。 儲存體帳戶之 Blob 服務的每小時計量、每分鐘計量及容量，均可從下表中具有相對應名稱的資料表中取得。

| 度量層級 | 資料表名稱 | 支援的版本 |
| --- | --- | --- |
| 每小時度量，主要位置 |$MetricsTransactionsBlob  <br/>$MetricsTransactionsTable <br/> $MetricsTransactionsQueue |僅適用於 2013-08-15 之前的版本。 儘管目前仍支援這些名稱，但還是建議您改用下列資料表。 |
| 每小時度量，主要位置 |$MetricsHourPrimaryTransactionsBlob <br/>$MetricsHourPrimaryTransactionsTable <br/>$MetricsHourPrimaryTransactionsQueue |所有版本 (包含 2013-08-15)。 |
| 每分鐘度量，主要位置 |$MetricsMinutePrimaryTransactionsBlob <br/>$MetricsMinutePrimaryTransactionsTable <br/>$MetricsMinutePrimaryTransactionsQueue |所有版本 (包含 2013-08-15)。 |
| 每小時度量，次要位置 |$MetricsHourSecondaryTransactionsBlob  <br/>$MetricsHourSecondaryTransactionsTable <br/>$MetricsHourSecondaryTransactionsQueue |所有版本 (包含 2013-08-15)。 必須啟用讀取存取異地備援複寫。 |
| 每分鐘度量，次要位置 |$MetricsMinuteSecondaryTransactionsBlob  <br/>$MetricsMinuteSecondaryTransactionsTable <br/>$MetricsMinuteSecondaryTransactionsQueue |所有版本 (包含 2013-08-15)。 必須啟用讀取存取異地備援複寫。 |
| 容量 (僅限 Blob 服務) |$MetricsCapacityBlob |所有版本 (包含 2013-08-15)。 |

針對儲存體帳戶啟用儲存體分析時，即會自動建立這些資料表。 您可以透過儲存體帳戶的命名空間存取它們，例如： `https://<accountname>.table.core.windows.net/Tables("$MetricsTransactionsBlob")`

### <a name="accessing-metrics-data"></a>存取度量資料
度量資料表中的所有資料都可使用資料表服務 API 來存取，包括由 Azure 管理的程式庫所提供的 .NET API。 儲存體帳戶管理員可以讀取和刪除資料表實體，但無法建立或更新它們。

## <a name="billing-for-storage-analytics"></a>儲存體分析計費
所有的度量資料都是由儲存體帳戶的服務所寫入。 因此，儲存體分析所執行的每個寫入作業都會列入計費。 此外，度量資料所使用的儲存體數量也會列入計費。

儲存體分析所執行的下列動作會列入計費：

* 建立 Blob 以用於記錄的要求。 
* 針對度量建立資料表實體的要求。

如果您已設定資料保留原則，就不需在儲存體分析刪除舊記錄和度量資料時支付刪除交易的費用。 不過，從用戶端刪除交易會列入計費。 如需保留原則的詳細資訊，請參閱 [設定儲存體分析資料保留原則](https://msdn.microsoft.com/library/azure/hh343263.aspx)。

### <a name="understanding-billable-requests"></a>了解計費要求
對帳戶的儲存體服務所提出的每個要求，都會被歸類為計費或不計費。 儲存體分析會記錄對服務所做的每個個別要求，包括表示如何處理要求的狀態訊息。 同樣地，儲存體分析會儲存服務及該服務之 API 作業的度量，包括特定狀態訊息的百分比和計數。 這些功能可共同協助您分析計費要求、改善您的應用程式，以及診斷服務要求的問題。 如需計費的詳細資訊，請參閱 [了解 Azure 儲存體計費 - 頻寬、交易和容量](http://blogs.msdn.com/b/windowsazurestorage/archive/2010/07/09/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity.aspx)。

檢視儲存體分析資料時，您可以使用 [儲存體分析記錄作業和狀態訊息](https://msdn.microsoft.com/library/azure/hh343260.aspx) 主題中的資料表，來判斷哪些要求需要計費。 然後，您可以將記錄檔和度量資料與狀態訊息進行比較，以查看您是否需為特定要求支付費用。 您也可以使用上述主題中的資料表，來調查儲存體服務或個別 API 作業的可用性。

## <a name="next-steps"></a>後續步驟
### <a name="setting-up-storage-analytics"></a>設定儲存體分析
* [在 Azure 入口網站中監視儲存體帳戶](storage-monitor-storage-account.md)
* [啟用及設定儲存體分析](https://msdn.microsoft.com/library/hh360996.aspx)

### <a name="storage-analytics-logging"></a>儲存體分析記錄
* [關於儲存體分析記錄](https://msdn.microsoft.com/library/hh343262.aspx)
* [儲存體分析記錄檔格式](https://msdn.microsoft.com/library/hh343259.aspx)
* [儲存體分析記錄作業和狀態訊息](https://msdn.microsoft.com/library/hh343260.aspx)

### <a name="storage-analytics-metrics"></a>儲存體分析度量
* [關於儲存體分析度量](https://msdn.microsoft.com/library/hh343258.aspx)
* [儲存體分析度量資料表結構描述](https://msdn.microsoft.com/library/hh343264.aspx)
* [儲存體分析記錄作業和狀態訊息](https://msdn.microsoft.com/library/hh343260.aspx)  

