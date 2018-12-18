---
title: 在 Azure 入口網站中啟用儲存體計量功能 | Microsoft Docs
description: 如何啟用 Blob、佇列、表格和檔案服務的儲存體度量
services: storage
author: roygara
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 02/14/2017
ms.author: rogarana
ms.component: common
ms.openlocfilehash: a12f2f3775808edb2045be5a1d955280f515ff7d
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529677"
---
# <a name="enabling-azure-storage-metrics-and-viewing-metrics-data"></a>啟用 Azure 儲存體計量和檢視計量資料
[!INCLUDE [storage-selector-portal-enable-and-view-metrics](../../../includes/storage-selector-portal-enable-and-view-metrics.md)]

## <a name="overview"></a>概觀
當您建立新的儲存體帳戶時，預設會啟用「儲存體計量」。 您可以透過 [Azure 入口網站](https://portal.azure.com)或 Windows PowerShell 設定監視功能，或透過其中一個儲存體用戶端程式庫以程式設計方式設定該功能。

您可以為計量資料設定保留期間：這段期間會決定儲存體服務用來保存計量的時間長短，以及針對儲存這些計量所需的空間收取的費用。 一般而言，您應該針對每分鐘度量使用比每小時度量還短的保留期限，因為每分鐘度量明顯需要額外的空間。 選擇保留期限，讓您擁有足夠的時間來分析資料，並下載任何您想要保留以供離線分析或報告使用的計量。 請記住，您也必須支付從儲存體帳戶下載計量資料的費用。

## <a name="how-to-enable-metrics-using-the-azure-portal"></a>如何使用 Azure 入口網站啟用計量功能
依照下列步驟在 [Azure 入口網站](https://portal.azure.com)中啟用計量功能：

1. 瀏覽至儲存體帳戶。
1. 選取 [功能表] 窗格上的 [診斷]。
1. 確定 [狀態] 設為 [開啟]。
1. 選取您要監視之服務的計量。
1. 指定保留原則，以表示要保留計量與記錄資料的時間。
1. 選取 [ **儲存**]。

[Azure 入口網站](https://portal.azure.com)目前無法讓您在儲存體帳戶中設定每分鐘計量功能，您必須使用 PowerShell 或以程式設計方式啟用每分鐘計量功能。

## <a name="how-to-enable-metrics-using-powershell"></a>如何使用 PowerShell 啟用計量功能
您可以在本機電腦上使用 PowerShell 來設定儲存體帳戶中的儲存體度量，做法是使用 Azure PowerShell Cmdlet Get-AzureStorageServiceMetricsProperty 來擷取目前的設定，並使用 Cmdlet Set-AzureStorageServiceMetricsProperty 來變更目前的設定。

控制儲存體度量的 Cmdlet 會使用下列參數：

* MetricsType：可能值為 Hour 和 Minute。
* ServiceType：可能值為 Blob、Queue 及 Table。
* MetricsLevel：可能值為 None、服務，以及 ServiceAndApi。

例如，下列命令會在您的預設儲存體帳戶中，為 Blob 服務開啟每分鐘計量功能，並將保留期間設為五天：

```powershell
Set-AzureStorageServiceMetricsProperty -MetricsType Minute -ServiceType Blob -MetricsLevel ServiceAndApi  -RetentionDays 5`
```

下列命令會擷取您預設儲存體帳戶中 Blob 服務的目前每小時度量層級和保留天數：

```powershell
Get-AzureStorageServiceMetricsProperty -MetricsType Hour -ServiceType Blob
```

如需如何設定 Azure PowerShell Cmdlet 以使用您的 Azure 訂用帳戶，以及如何選取要使用的預設儲存體帳戶的相關資訊，請參閱： [如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

## <a name="how-to-enable-storage-metrics-programmatically"></a>如何以程式設計方式啟用儲存體度量
下列 C# 程式碼片段示範如何使用 .NET 的儲存體用戶端程式庫，為 Blob 服務啟用計量和記錄功能：

```csharp
//Parse the connection string for the storage account.
const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

// Create service client for credentialed access to the Blob service.
CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

// Enable Storage Analytics logging and set retention policy to 10 days.
ServiceProperties properties = new ServiceProperties();
properties.Logging.LoggingOperations = LoggingOperations.All;
properties.Logging.RetentionDays = 10;
properties.Logging.Version = "1.0";

// Configure service properties for metrics. Both metrics and logging must be set at the same time.
properties.HourMetrics.MetricsLevel = MetricsLevel.ServiceAndApi;
properties.HourMetrics.RetentionDays = 10;
properties.HourMetrics.Version = "1.0";

properties.MinuteMetrics.MetricsLevel = MetricsLevel.ServiceAndApi;
properties.MinuteMetrics.RetentionDays = 10;
properties.MinuteMetrics.Version = "1.0";

// Set the default service version to be used for anonymous requests.
properties.DefaultServiceVersion = "2015-04-05";

// Set the service properties.
blobClient.SetServiceProperties(properties);
```

## <a name="viewing-storage-metrics"></a>檢視儲存體度量
在您設定儲存體分析計量監視您的儲存體帳戶後，儲存體分析會將計量記錄在您儲存體帳戶中一組已知資料表中。 您可以在 [Azure 入口網站](https://portal.azure.com)中將圖表設定為檢視每小時計量：

1. 在 [Azure 入口網站](https://portal.azure.com)中瀏覽至您的儲存體帳戶。
1. 針對您要檢視其計量的服務，選取 [功能表] 窗格中的 [計量]。
1. 在您要設定的圖表上選取 [編輯]。
1. 在 [編輯圖表] 窗格中，選取 [時間範圍]、[圖表類型] 與您要在圖表中顯示的計量。
1. 選取 [確定]

如果要下載計量以長期儲存，或在本機加以分析，您必須：

* 使用可感知這些資料表、且可讓您檢視及下載它們的工具。
* 撰寫自訂應用程式或指令碼來讀取和儲存資料表。

有許多協力廠商儲存體瀏覽工具可以感知這些資料表，讓您能夠直接檢視它們。
如需可用工具的清單，請參閱 [Azure 儲存體用戶端工具](storage-explorers.md)。

> [!NOTE]
> 從 [Microsoft Azure 儲存體總管](http://storageexplorer.com/) 0.8.0 版開始，您便可以檢視和下載分析計量資料表。
>
>

為了能編寫程式來存取分析資料表，請注意，如果您在儲存體帳戶中列出所有資料表，則分析資料表就不會出現。 您可以直接依名稱存取它們，或在 .NET 用戶端程式庫中使用 [CloudAnalyticsClient API](https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.analytics.cloudanalyticsclient.aspx) 查詢資料表名稱。

### <a name="hourly-metrics"></a>每小時度量
* $MetricsHourPrimaryTransactionsBlob
* $MetricsHourPrimaryTransactionsTable
* $MetricsHourPrimaryTransactionsQueue

### <a name="minute-metrics"></a>每分鐘度量
* $MetricsMinutePrimaryTransactionsBlob
* $MetricsMinutePrimaryTransactionsTable
* $MetricsMinutePrimaryTransactionsQueue

### <a name="capacity"></a>Capacity
* $MetricsCapacityBlob

您可以在 [儲存體分析度量資料表結構描述](https://msdn.microsoft.com/library/azure/hh343264.aspx)上找到這些資料表之結構描述的完整詳細資料。 下列資料列範例只會顯示可用的資料行子集，但可說明儲存體度量儲存這些度量資訊之方式的一些重要功能：

| PartitionKey | RowKey | Timestamp | TotalRequests | TotalBillableRequests | TotalIngress | TotalEgress | 可用性 | AverageE2ELatency | AverageServerLatency | PercentSuccess |
| --- |:---:| ---:| --- | --- | --- | --- | --- | --- | --- | --- |
| 20140522T1100 |user;All |2014-05-22T11:01:16.7650250Z |7 |7 |4003 |46801 |100 |104.4286 |6.857143 |100 |
| 20140522T1100 |user;QueryEntities |2014-05-22T11:01:16.7640250Z |5 |5 |2694 |45951 |100 |143.8 |7.8 |100 |
| 20140522T1100 |user;QueryEntity |2014-05-22T11:01:16.7650250Z |1 |1 |538 |633 |100 |3 |3 |100 |
| 20140522T1100 |user;UpdateEntity |2014-05-22T11:01:16.7650250Z |1 |1 |771 |217 |100 |9 |6 |100 |

在這個每分鐘度量資料範例中，資料分割索引鍵會在每分鐘解析中使用時間。 資料列索引鍵可識別儲存在資料列中的資訊類型。 資料列索引鍵是由存取類型和要求類型兩項資訊所組成：

* 存取類型是使用者或系統，其中使用者是指對儲存體服務所提出的所有使用者要求，而系統是指對儲存體分析所提出的要求。
* 要求類型就是全部 (在此情況下它是摘要行)，或者它會識別特定的 API，例如 QueryEntity 或 UpdateEntity。

上述範例資料會為單一分鐘 (從上午 11:00 開始) 顯示所有記錄，因此 QueryEntities 要求的數目加上 QueryEntity 要求的數目再加上 UpdateEntity 要求的數目最多七個，也就是 user:All 資料列上顯示的總數。 同樣地，您可以藉由計算 ((143.8 * 5) + 3 + 9)/7，在 user:All 資料列上衍生平均的端對端延遲 104.4286。

請注意，**Blob 每小時計量設定**會同時套用至 **Blob 容量計量** ($MetricsCapacityBlob) 和**每小時 Blob 交易計量** ($MetricsHourPrimaryTransactionsBlob)。 兩者會同時啟用或停用，並使用相同的保留原則。

## <a name="metrics-alerts"></a>計量警示
您應該考慮在 [Azure 入口網站](https://portal.azure.com)中設定警示，以便讓「儲存體計量」自動通知您有關您儲存體服務行為的重要變更。 若使用儲存體總管工具來下載此計量資料 (以使用分隔字元分隔的形式)，您可以使用 Microsoft Excel 來分析資料。 如需可用的儲存體總管工具清單，請參閱 [Azure 儲存體用戶端工具](storage-explorers.md)。 您可以在 [警示規則] 窗格 (可從 [儲存體帳戶] 功能表窗格中的 [監視] 下存取) 中設定警示。

> [!IMPORTANT]
> 發生儲存體事件與記錄對應的每小時或分鐘計量資料之間可能會有延遲。 記錄分鐘計量時，可能會一次寫入數分鐘的資料。 幾分鐘前的交易可能會彙總至目前分鐘的交易。 當發生這種情況時，警示服務可能不會擁有所設定警示間隔的所有可用計量資料，這可能導致非預期地觸發警示。
>

## <a name="accessing-metrics-data-programmatically"></a>以程式設計方式存取度量資料
下列清單顯示 C# 程式碼範例，其會針對某個分鐘範圍存取每分鐘度量，並將結果顯示在主控台視窗中。 它會使用 Azure 儲存體程式庫第 4 版，其中包含可簡化存取儲存體中之度量資料表的 CloudAnalyticsClient 類別。

```csharp
private static void PrintMinuteMetrics(CloudAnalyticsClient analyticsClient, DateTimeOffset startDateTime, DateTimeOffset endDateTime)
{
    // Convert the dates to the format used in the PartitionKey
    var start = startDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");
    var end = endDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");

    var services = Enum.GetValues(typeof(StorageService));
    foreach (StorageService service in services)
    {
        Console.WriteLine("Minute Metrics for Service {0} from {1} to {2} UTC", service, start, end);
        var metricsQuery = analyticsClient.CreateMinuteMetricsQuery(service, StorageLocation.Primary);
        var t = analyticsClient.GetMinuteMetricsTable(service);
        var opContext = new OperationContext();
        var query =
          from entity in metricsQuery
          // Note, you can't filter using the entity properties Time, AccessType, or TransactionType
          // because they are calculated fields in the MetricsEntity class.
          // The PartitionKey identifies the DataTime of the metrics.
          where entity.PartitionKey.CompareTo(start) >= 0 && entity.PartitionKey.CompareTo(end) <= 0
        select entity;

        // Filter on "user" transactions after fetching the metrics from Table Storage.
        // (StartsWith is not supported using LINQ with Azure table storage)
        var results = query.ToList().Where(m => m.RowKey.StartsWith("user"));
        var resultString = results.Aggregate(new StringBuilder(), (builder, metrics) => builder.AppendLine(MetricsString(metrics, opContext))).ToString();
        Console.WriteLine(resultString);
    }
}

private static string MetricsString(MetricsEntity entity, OperationContext opContext)
{
    var entityProperties = entity.WriteEntity(opContext);
    var entityString =
      string.Format("Time: {0}, ", entity.Time) +
      string.Format("AccessType: {0}, ", entity.AccessType) +
      string.Format("TransactionType: {0}, ", entity.TransactionType) +
      string.Join(",", entityProperties.Select(e => new KeyValuePair<string, string>(e.Key.ToString(), e.Value.PropertyAsObject.ToString())));
    return entityString;
}
```

## <a name="what-charges-do-you-incur-when-you-enable-storage-metrics"></a>當您啟用儲存體度量時，會產生哪些費用？
寫入要求以建立度量的資料表實體，會以適用於所有 Azure 儲存體作業的標準費率來收費。

用戶端對於度量資料的讀取和刪除要求也會以標準費率來計費。 如果您已設定資料保留原則，就不需要在 Azure 儲存體刪除舊的度量資料時付費。 不過，如果您刪除分析資料，則您的帳戶必須支付刪除作業的費用。

度量資料表所使用的容量也會列入計費：您可以使用下列項目來估計儲存計量資料所使用的容量大小：

* 如果服務每小時會使用每個服務中的每一種 API，若您已啟用服務和 API 層級摘要，則每小時大約有 148 KB 的資料將儲存於度量交易資料表中。
* 如果服務每小時會使用每個服務中的每一種 API，若您只啟用服務層級摘要，則每小時大約有 12 KB 的資料將儲存於度量交易資料表中。
* 適用於 Blob 的容量資料表每天都會新增兩個資料列 (前提是使用者已選擇記錄檔)：也就是說，這個資料表的大小每天最多大約會增加 300 個位元組。

## <a name="next-steps"></a>後續步驟
[啟用儲存體記錄和存取記錄檔資料](/rest/api/storageservices/Enabling-Storage-Logging-and-Accessing-Log-Data)
