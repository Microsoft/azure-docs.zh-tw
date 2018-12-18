---
title: 使用 Azure Log Analytics 資料收集器 API 建立資料管線 | Microsoft Docs
description: 您可以使用 Log Analytics HTTP 資料收集器 API，將 POST JSON 資料從任何可呼叫 REST API 的用戶端新增至 Log Analytics 存放庫。 本文將說明如何以自動化方式上傳儲存在檔案中的資料。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/09/2018
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: 180f1a39b92dd699fa114cb98a5842b0ab0dc89a
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190533"
---
# <a name="create-a-data-pipeline-with-the-data-collector-api"></a>使用資料收集器 API 建立資料管線

[Log Analytics 資料收集器 API](log-analytics-data-collector-api.md) 可讓您將任何自訂資料匯入到 Log Analytics 中。 唯一要求是資料必須可格式化為 JSON，並且可分割成小於 30 MB (含) 的區段。 此完全彈性化的機制可透過多種方式來插入，包括：直接從您的應用程式傳送資料到完成一次性的臨機操作上傳。 本文將以常見案例：需要定期自動上傳儲存在檔案中的資料，來概述一些入門作法。 然而，在此呈現的管線並不是效能最好或已最佳化的，應將其視為建置您生產管線的起點。

## <a name="example-problem"></a>問題範例
在本文的其餘部分中，我們將透過 Application Insights 檢查頁面檢視資料。 在我們的虛構案例中，我們要將預設由 Application Insights SDK 收集的地理資訊，聯結至包含世界中各個國家/地區人口的自訂資料，以識別應該在何處投入最多行銷預算。 

我們會使用 [UN World Population Prospects](https://esa.un.org/unpd/wpp/) (聯合國世界人口展望) 等公開資料來源來完成此目的。 資料會有下列簡易結構描述：

![簡易結構描述範例](./media/log-analytics-create-pipeline-datacollector-api/example-simple-schema-01.png)

在本範例中，我們假設會在新檔案 (其中具有最新年份的資料) 可用時，將其上傳。

## <a name="general-design"></a>一般設計
我們使用傳統的 ETL 類型邏輯來設計管線。 架構如下圖所示︰

![資料收集管線架構](./media/log-analytics-create-pipeline-datacollector-api/data-pipeline-dataflow-architecture.png)

本文不會說明如何建立資料或[將資料上傳至 Azure Blob 儲存體帳戶](../storage/blobs/storage-upload-process-images.md)。 而是，我們會在新檔案上傳至 Blob 時，提取該流程。 如下列步驟所示：

1. 處理程序會偵測是否已上傳新資料。  我們的範例會使用 [Azure 邏輯應用程式](../logic-apps/logic-apps-overview.md)，其中的觸發程序可偵測到正在上傳至 Blob 的新資料。

2. 處理器會讀取這項新資料，並將資料轉換為 JSON (Log Analytics 所需的格式)。  在此範例中，我們會使用 [Azure Function](../azure-functions/functions-overview.md)，以輕便且符合成本效益的方式來執行處理程式碼。 函式會透過用來偵測新資料的相同邏輯應用程式來啟動。

3. 最後，當 JSON 物件可用之後，此物件會傳送至 Log Analytics。 同一個邏輯應用程式會使用內建的 Log Analytics 資料收集器活動，將資料傳送至 Log Analytics。

雖然本文未敘述 Blob 儲存體、邏輯應用程式或 Azure Function 的詳細安裝方式，但您可以在特定產品的網頁上找到詳細指示。

為了監視此管線，我們使用 Application Insights 來監視 Azure Function，[詳細資訊請參閱這裡](../azure-functions/functions-monitoring.md)，並使用 Log Analytics 來監視邏輯應用程式，[詳細資訊請參閱這裡](../logic-apps/logic-apps-monitor-your-logic-apps-oms.md)。 

## <a name="setting-up-the-pipeline"></a>設定管線
若要設定管線，請先確定您已建立並設定 Blob 容器。 同樣地，請確定您已建立要將資料傳送到其中的 Log Analytics 工作區。

## <a name="ingesting-json-data"></a>內嵌 JSON 資料
透過邏輯應用程式內嵌 JSON 資料相當容易，而且由於不需要進行任何轉換，我們可以將整個管線納入單一邏輯應用程式。 設定好 Blob 容器和 Log Analytics 工作區後，即可建立新的邏輯應用程式並加以設定，如下所示：

![邏輯應用程式的工作流程範例](./media/log-analytics-create-pipeline-datacollector-api/logic-apps-workflow-example-01.png)

儲存邏輯應用程式，並繼續進行測試。

## <a name="ingesting-xml-csv-or-other-formats-of-data"></a>內嵌 XML、CSV 或其他格式的資料
Logic Apps 目前並沒有可輕鬆地將 XML、CSV 或其他類型轉換成 JSON 格式的內建功能。 因此，我們需要使用另一個方法來完成這項轉換。 在本文章中，我們使用 Azure Functions 的無伺服器計算功能來完成此作業，此方式非常輕便且符合成本效益。 

在此範例中，我們會剖析 CSV 檔案，但您可以使用類似的方式處理任何其他檔案類型。 只需修改 Azure Function 的還原序列化部分，即可反映特定資料類型的正確邏輯。

1.  建立新的 Azure Function，並在出現提示時，使用 Function 執行階段 v1 並以耗用量作為依據。  選取以 C# 作為目標的 [HTTP 觸發程序]，作為設定所需繫結的起點。 
2.  從右側窗格上的 [檢視檔案] 索引標籤中，建立名為 **project.json** 的新檔案，並從我們正在使用的 NuGet 套件中，貼上下列程式碼：

    ![Azure Functions 的專案範例](./media/log-analytics-create-pipeline-datacollector-api/functions-example-project-01.png)
    
    ``` JSON
    {
      "frameworks": {
        "net46":{
          "dependencies": {
            "CsvHelper": "7.1.1",
            "Newtonsoft.Json": "11.0.2"
          }  
        }  
       }  
     }  
    ```

3. 從右側窗格中切換到 **run.csx**，並以下列內容取代預設的程式碼。 

    >[!NOTE]
    >如果是您自己的專案，則必須以您自己的資料結構描述取代記錄模型 ("PopulationRecord" 類別)。
    >

    ```   
    using System.Net;
    using Newtonsoft.Json;
    using CsvHelper;
    
    class PopulationRecord
    {
        public String Location { get; set; }
        public int Time { get; set; }
        public long Population { get; set; }
    }

    public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
    {
        string filePath = await req.Content.ReadAsStringAsync(); //get the CSV URI being passed from Logic App
        string response = "";

        //get a stream from blob
        WebClient wc = new WebClient();
        Stream s = wc.OpenRead(filePath);         

        //read the stream
        using (var sr = new StreamReader(s))
        {
            var csvReader = new CsvReader(sr);
    
            var records = csvReader.GetRecords<PopulationRecord>(); //deserialize the CSV stream as an IEnumerable
    
            response = JsonConvert.SerializeObject(records); //serialize the IEnumerable back into JSON
        }    

        return response == null
            ? req.CreateResponse(HttpStatusCode.BadRequest, "There was an issue getting data")
            : req.CreateResponse(HttpStatusCode.OK, response);
     }  
    ```

4. 儲存您的函式。
5. 測試函式，以確定程式碼正常運作。 在右側窗格中切換到 [測試] 索引標籤，依照下圖來設定測試。 在 [要求本文] 文字方塊中放入範例資料 Blob 的連結。 按一下 [執行] 後，您應該會在 [輸出] 方塊中看到 JSON 輸出：

    ![函式應用程式的測試程式碼](./media/log-analytics-create-pipeline-datacollector-api/functions-test-01.png)

現在我們必須返回稍早開始建置的邏輯應用程式並進行修改，以包含內嵌和轉換為 JSON 格式的資料。  使用檢視表設計師並依照下列方式設定，然後儲存您的邏輯應用程式：

![邏輯應用程式的工作流程完成範例](./media/log-analytics-create-pipeline-datacollector-api/logic-apps-workflow-example-02.png)

## <a name="testing-the-pipeline"></a>測試管線
現在您可以將新檔案上傳至稍早設定的 Blob，並讓您的邏輯應用程式監視此 Blob。 不久之後，您應該會看到邏輯應用程式的新執行個體開始執行，並且會呼叫您的 Azure Function，然後成功地將資料傳送至 Log Analytics。 

>[!NOTE]
>初次傳送新資料類型時，資料可能會在 30 分鐘後才出現在 Log Analytics 中。


## <a name="correlating-with-other-data-in-log-analytics-and-application-insights"></a>與 Log Analytics 和 Application Insights 中的其他資料相互關聯
若要完成聯結 Application Insights 網頁檢視資料與人口資料 (內嵌自我們的自訂資料來源) 的目標，請從 Application Insights Analytics 視窗或 Log Analytics 工作區中執行下列查詢：

``` KQL
app("fabrikamprod").pageViews
| summarize numUsers = count() by client_CountryOrRegion
| join kind=leftouter (
   workspace("customdatademo").Population_CL
) on $left.client_CountryOrRegion == $right.Location_s
| project client_CountryOrRegion, numUsers, Population_d
```

輸出中應該會顯示兩個資料來源現在已聯結。  

![在搜尋結果中使兩個分開資料相互關聯的範例](./media/log-analytics-create-pipeline-datacollector-api/correlating-disjoined-data-example-01.png)

## <a name="suggested-improvements-for-a-production-pipeline"></a>生產管線的改善建議
本文提供可運作的原型，其背後的邏輯可套用至確實具備生產品質的解決方案。 針對具備生產品質的這類解決方案，建議您使用下列改善方式：

* 在您的邏輯應用程式和函式中新增錯誤處理和重試邏輯。
* 新增邏輯以確保未超過 30MB/單一 Log Analytics 擷取 API 的呼叫限制。 如有需要，請將資料分割成較小的區段。
* 在您的 Blob 儲存體上設定清除原則。 將資料成功傳送至 Log Analytics 後，除非您想要保留未經處理的資料以用於封存，否則無須繼續儲存這些資料。 
* 確認已針對整個管線啟用監視，並適當地新增追蹤點與警示。
* 利用原始程式碼控制來管理函式和邏輯應用程式的程式碼。
* 請確定已遵循適當的變更管理原則，例如，當結構描述變更時，函式和 Logic Apps 會隨之修改。
* 如果您要上傳多個不同的資料類型，請將這些類型分散在 Blob 容器內的個別資料夾，並建立可根據資料類型扇出 (fan out) 邏輯的邏輯。 


## <a name="next-steps"></a>後續步驟
深入了解[資料收集器 API](log-analytics-data-collector-api.md)，以將資料從任何 REST API 用戶端寫入 Log Analytics。
