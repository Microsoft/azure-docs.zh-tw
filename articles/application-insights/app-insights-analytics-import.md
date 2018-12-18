---
title: 將資料匯入 Azure Application Insights 中的分析 | Microsoft Docs
description: 匯入靜態資料以加入應用程式遙測，或匯入個別的資料流以分析查詢。
services: application-insights
keywords: 開啟結構描述、匯入資料
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 08/14/2018
ms.author: mbullwin
ms.openlocfilehash: cfcdf13f8aa4dfab9b361ccbb82ea4b2c3e2ca0d
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42140669"
---
# <a name="import-data-into-analytics"></a>將資料匯入分析

將任何表格式資料匯入[分析](app-insights-analytics.md)，將它從應用程式加入 [Application Insights](app-insights-overview.md) 遙測，或是讓您可以為個別的資料流進行分析。 分析是適用於分析大量遙測時間戳記資料流的強大查詢語言。
您可以使用自己的結構描述將資料匯入分析。 它不需要使用標準 Application Insights 結構描述，例如要求或追蹤。

您可以匯入 JSON 或 DSV (以分隔符號分隔值 - 逗號、分號或定位字元) 檔案。

> [!IMPORTANT]
> 此文章已**淘汰**。 若要將資料放到 Log Analytics 中，建議的方法是透過 [Log Analytics 資料收集器 API。](https://docs.microsoft.com/azure/log-analytics/log-analytics-data-collector-api)

有三種情況下，匯入至分析會很實用︰

* **加入應用程式遙測。** 例如，您可以匯入將來自您網站的 URL 對應至更容易閱讀之頁面標題的資料表。 在分析中，您可以建立儀表板圖表報表，其會顯示您的網站中十個最常用的頁面。 現在其可以顯示頁面標題，而非 URL。
* **相互關聯應用程式遙測**與其他來源，例如網路流量、伺服器資料或 CDN 記錄檔。
* **將個別的資料流套用至分析。** Application Insights 分析是強大的工具，可搭配疏鬆、時間戳記資料流 - 在許多情況下遠優於 SQL。 如果您有來自其他來源的資料流，您可以使用分析進行分析。

將資料傳送至您的資料來源很簡單。 

1. (一次性) 在資料來源中定義資料結構描述。
2. (定期) 將您的資料上傳至 Azure 儲存體並呼叫 REST API，以通知我們新的資料正在等候擷取。 在幾分鐘內，資料可在分析中供查詢。

上傳的頻率是由您以及您希望您的資料多快可供查詢來定義。 以較大的區塊上傳是更有效率的方式，但不超過 1 GB。

> [!NOTE]
> *有許多資料來源要分析嗎？* [*請考慮使用 logstash* *以將您的資料傳送至 Application Insights。*](https://github.com/Microsoft/logstash-output-application-insights)
> 

## <a name="before-you-start"></a>開始之前

您需要：

1. Microsoft Azure 中的 Application Insights 資源。

 * 如果您想要與其他遙測分開分析您的資料，[建立新的 Application Insights 資源](app-insights-create-new-resource.md)。
 * 如果您要加入或比較從已使用 Application Insights 設定的應用程式遙測之資料，您可以使用該應用程式的資源。
 * 參與者或擁有者存取該資源。
 
2. Azure 儲存體中的 Blob。 您上傳至 Azure 儲存體，分析會從該處取得您的資料。 

 * 我們建議您針對 blob 建立專用的儲存體帳戶。 如果與其他處理序共用您的 blob，我們的程序會花較長時間來讀取 blob。


## <a name="define-your-schema"></a>定義結構描述

在您匯入資料之前，您必須定義資料來源，以指定您資料的結構描述。
在單一 Application Insights 資源中，最多可包含 50 個資料來源

1. 啟動資料來源精靈。 使用 [新增資料來源] 按鈕。 或者 - 按一下右上角的 [設定] 按鈕，然後在下拉式功能表中選擇 [資料來源]。

    ![新增資料來源](./media/app-insights-analytics-import/add-new-data-source.png)

    提供新資料來源的名稱。

2. 定義您要上傳的檔案格式。

    您可以手動定義格式，或上傳範例檔案。

    如果資料是 CSV 格式，此範例的第一列可以是資料行標頭。 您可以在下一個步驟中變更欄位名稱。

    範例應該包含至少 10 個資料列或資料記錄。

    資料行或欄位名稱應包含英數字元名稱 (不含空格或標點符號)。

    ![上傳範例檔案](./media/app-insights-analytics-import/sample-data-file.png)


3. 檢閱精靈已經得到的結構描述。 如果它是從範例推斷出類型，您可能需要調整推斷的資料行類型。

    ![檢閱推斷的結構描述](./media/app-insights-analytics-import/data-source-review-schema.png)

 * (選擇性。)上傳結構描述定義。 請看以下的格式。

 * 選取時間戳記。 分析中的所有資料都必須都有時間戳記欄位。 必須有 `datetime` 類型，但不必命名為 'timestamp'。 如果您的資料具有包含以 ISO 格式表示之日期和時間的資料行，選擇此選項為時間戳記資料行。 否則，請選擇「資料抵達時」，匯入程序將會新增時間戳記欄位。

5. 建立資料來源。

### <a name="schema-definition-file-format"></a>結構描述定義檔案格式

不要在 UI 中編輯結構描述，而是從檔案載入結構描述定義。 結構描述定義格式如下︰ 

以符號分隔的格式 
```
[ 
    {"location": "0", "name": "RequestName", "type": "string"}, 
    {"location": "1", "name": "timestamp", "type": "datetime"}, 
    {"location": "2", "name": "IPAddress", "type": "string"} 
] 
```

JSON 格式 
```
[ 
    {"location": "$.name", "name": "name", "type": "string"}, 
    {"location": "$.alias", "name": "alias", "type": "string"}, 
    {"location": "$.room", "name": "room", "type": "long"} 
]
```
 
以每個資料行的位置、名稱、類型識別它。

* 位置 - 若為以符號分隔的檔案格式，則是對應值的位置。 若為 JSON 格式，則是對應索引鍵的 jpath。
* 名稱 - 資料行的顯示名稱。
* 類型 - 資料行的類型。
 
> [!NOTE]
> 在使用範例資料、而且檔案格式是符號分隔的情況下，結構描述定義必須對應所有資料行，並在結尾加入新的資料行。
> 
> JSON 允許資料部分對應，因此 JSON 格式的結構描述定義不需要對應範例資料中找到的每個索引鍵。 它也可以對應不屬於範例資料的資料行。 

## <a name="import-data"></a>匯入資料

若要匯入資料，請將它上傳至 Azure 儲存體、建立便捷鍵，然後建立 REST API 呼叫。

![新增資料來源](./media/app-insights-analytics-import/analytics-upload-process.png)

您可以手動執行下列程序，或設定自動系統定期執行。 您必須依照下列步驟進行每個您要匯入的資料區塊。

1. 將資料上傳至 [Azure Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)。 

 * Blob 未壓縮大小可以達 1 GB 。 從效能觀點來看，數百 MB 的大型 blob 很適合。
 * 可以使用 Gzip 進行壓縮，以改善上傳時間和延遲時間，以便資料可用於查詢。 使用 `.gz` 副檔名。
 * 針對此目的，最好使用個別的儲存體帳戶，以避免來自不同服務的呼叫拖慢效能。
 * 以高頻率 (每幾秒一次) 傳送資料時，基於效能考量，建議使用多個儲存體帳戶。

 
2. [建立 blob 的共用存取簽章](../storage/blobs/storage-dotnet-shared-access-signature-part-2.md). 金鑰應該有一天的到期時間，並提供讀取權限。
3. 進行 REST 呼叫，以通知 Application Insights 資料正在等候。

 * 端點：`https://dc.services.visualstudio.com/v2/track`
 * HTTP 方法：POST
 * 承載：

```JSON

    {
       "data": {
            "baseType":"OpenSchemaData",
            "baseData":{
               "ver":"2",
               "blobSasUri":"<Blob URI with Shared Access Key>",
               "sourceName":"<Schema ID>",
               "sourceVersion":"1.0"
             }
       },
       "ver":1,
       "name":"Microsoft.ApplicationInsights.OpenSchema",
       "time":"<DateTime>",
       "iKey":"<instrumentation key>"
    }
```

預留位置是︰

* `Blob URI with Shared Access Key`︰從建立機碼程序取得。 它是 blob 特定。
* `Schema ID`：為已定義的結構描述產生的結構描述識別碼。 此 Blob 中的資料應該與結構描述相符。
* `DateTime`：送出要求的時間 (UTC)。 我們接受這些格式︰ISO8601 (例如 "2016-01-01 13:45:01")；RFC822 ("Wed, 14 Dec 16 14:57:01 +0000")；RFC850 ("Wednesday, 14-Dec-16 14:57:00 UTC")；RFC1123 ("Wed, 14 Dec 2016 14:57:00 +0000")。
* 您 `Instrumentation key` 的 Application Insights 資源。

後幾分鐘可在分析中使用資料。

## <a name="error-responses"></a>錯誤回應

* **400 不正確的要求**︰表示要求承載無效。 勾選：
 * 修正動態檢測金鑰。
 * 有效的時間值。 它應該是現在的 UTC 時間。
 * 事件的 JSON 符合結構描述。
* **403 禁止**︰您所傳送的 blob 不能存取。 請確定共用的存取金鑰有效且尚未過期。
* **404 找不到**：
 * Blob 不存在。
 * 來源識別碼是錯誤的。

回應錯誤訊息中有更詳細的資訊。


## <a name="sample-code"></a>範例程式碼

此程式碼使用 [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/9.0.1) NuGet 封裝。

### <a name="classes"></a>類別

```csharp
namespace IngestionClient 
{ 
    using System; 
    using Newtonsoft.Json; 

    public class AnalyticsDataSourceIngestionRequest 
    { 
        #region Members 
        private const string BaseDataRequiredVersion = "2"; 
        private const string RequestName = "Microsoft.ApplicationInsights.OpenSchema"; 
        #endregion Members 

        public AnalyticsDataSourceIngestionRequest(string ikey, string schemaId, string blobSasUri, int version = 1) 
        { 
            Ver = version; 
            IKey = ikey; 
            Data = new Data 
            { 
                BaseData = new BaseData 
                { 
                    Ver = BaseDataRequiredVersion, 
                    BlobSasUri = blobSasUri, 
                    SourceName = schemaId, 
                    SourceVersion = version.ToString() 
                } 
            }; 
        } 


        [JsonProperty("data")] 
        public Data Data { get; set; } 

        [JsonProperty("ver")] 
        public int Ver { get; set; } 

        [JsonProperty("name")] 
        public string Name { get { return RequestName; } } 

        [JsonProperty("time")] 
        public DateTime Time { get { return DateTime.UtcNow; } } 

        [JsonProperty("iKey")] 
        public string IKey { get; set; } 
    } 

    #region Internal Classes 

    public class Data 
    { 
        private const string DataBaseType = "OpenSchemaData"; 

        [JsonProperty("baseType")] 
        public string BaseType 
        { 
            get { return DataBaseType; } 
        } 

        [JsonProperty("baseData")] 
        public BaseData BaseData { get; set; } 
    } 


    public class BaseData 
    { 
        [JsonProperty("ver")] 
        public string Ver { get; set; } 

        [JsonProperty("blobSasUri")] 
        public string BlobSasUri { get; set; } 

        [JsonProperty("sourceName")] 
        public string SourceName { get; set; } 

        [JsonProperty("sourceVersion")] 
        public string SourceVersion { get; set; } 
    } 

    #endregion Internal Classes 
} 


namespace IngestionClient 
{ 
    using System; 
    using System.IO; 
    using System.Net; 
    using System.Text; 
    using System.Threading.Tasks; 
    using Newtonsoft.Json; 

    public class AnalyticsDataSourceClient 
    { 
        #region Members 
        private readonly Uri endpoint = new Uri("https://dc.services.visualstudio.com/v2/track"); 
        private const string RequestContentType = "application/json; charset=UTF-8"; 
        private const string RequestAccess = "application/json"; 
        #endregion Members 

        #region Public 

        public async Task<bool> RequestBlobIngestion(AnalyticsDataSourceIngestionRequest ingestionRequest) 
        { 
            HttpWebRequest request = WebRequest.CreateHttp(endpoint); 
            request.Method = WebRequestMethods.Http.Post; 
            request.ContentType = RequestContentType; 
            request.Accept = RequestAccess; 

            string notificationJson = Serialize(ingestionRequest); 
            byte[] notificationBytes = GetContentBytes(notificationJson); 
            request.ContentLength = notificationBytes.Length; 

            Stream requestStream = request.GetRequestStream(); 
            requestStream.Write(notificationBytes, 0, notificationBytes.Length); 
            requestStream.Close(); 

            try 
            { 
                using (var response = (HttpWebResponse)await request.GetResponseAsync())
                {
                    return response.StatusCode == HttpStatusCode.OK;
                }
            } 
            catch (WebException e) 
            { 
                HttpWebResponse httpResponse = e.Response as HttpWebResponse; 
                if (httpResponse != null) 
                { 
                    Console.WriteLine( 
                        "Ingestion request failed with status code: {0}. Error: {1}", 
                        httpResponse.StatusCode, 
                        httpResponse.StatusDescription); 
                    return false; 
                }
                throw; 
            } 
        } 
        #endregion Public 

        #region Private 
        private byte[] GetContentBytes(string content) 
        { 
            return Encoding.UTF8.GetBytes(content);
        } 


        private string Serialize(AnalyticsDataSourceIngestionRequest ingestionRequest) 
        { 
            return JsonConvert.SerializeObject(ingestionRequest); 
        } 
        #endregion Private 
    } 
} 
```

### <a name="ingest-data"></a>擷取資料

針對每個 blob 使用此程式碼。 

```csharp
   AnalyticsDataSourceClient client = new AnalyticsDataSourceClient(); 

   var ingestionRequest = new AnalyticsDataSourceIngestionRequest("iKey", "sourceId", "blobUrlWithSas"); 

   bool success = await client.RequestBlobIngestion(ingestionRequest);
```

## <a name="next-steps"></a>後續步驟

* [Log Analytics 查詢語言導覽](app-insights-analytics-tour.md)
* 如果您使用 Logstash，請使用 [Logstash 外掛程式將資料傳送至 Application Insights](https://github.com/Microsoft/logstash-output-application-insights)
