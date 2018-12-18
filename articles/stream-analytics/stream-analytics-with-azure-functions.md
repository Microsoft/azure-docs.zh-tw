---
title: 教學課程：使用 Azure 串流分析作業來執行 Azure Functions | Microsoft Docs
description: 在本教學課程中，您將了解如何設定 Azure Functions 作為串流分析作業的輸出接收。
services: stream-analytics
author: jasonwhowell
manager: kfile
ms.service: stream-analytics
ms.topic: tutorial
ms.custom: mvc
ms.workload: data-services
ms.date: 04/09/2018
ms.author: jasonh
ms.reviewer: jasonh
ms.openlocfilehash: 0408ea6ead1ddf482ce0a07c21859af80ab6ca43
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697810"
---
# <a name="run-azure-functions-from-azure-stream-analytics-jobs"></a>從 Azure 串流分析作業執行 Azure Functions 

您可以將 Functions 設定為串流分析作業的其中一個輸出接收，藉以從 Azure 串流分析執行 Azure Functions。 Functions 是事件導向隨選計算的體驗，可讓您實作在 Azure 或第三方服務中發生之事件所觸發的程式碼。 Functions 回應觸發程序的功能，很適合作為串流分析作業的輸出。

串流分析會透過 HTTP 觸發程序叫用 Functions。 Functions 輸出配接器可以讓使用者將 Functions 連接至串流分析，如此便可以根據串流分析的查詢來觸發事件。 

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立串流分析作業
> * 建立 Azure 函式
> * 將 Azure 函式設定為您作業的輸出

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="configure-a-stream-analytics-job-to-run-a-function"></a>設定串流分析作業以執行函式 

本節示範如何設定串流分析作業，以執行將資料寫入至 Azure Redis 快取的函式。 串流分析作業會從 Azure 事件中樞讀取事件，並執行叫用函式的查詢。 此函式會從串流分析作業讀取資料，並將資料寫入至 Azure Redis 快取。

![顯示 Azure 服務之間關聯性的圖表](./media/stream-analytics-with-azure-functions/image1.png)

完成這項工作的必要步驟如下：
* [使用事件中樞做為輸入，建立串流分析工作](#create-stream-analytics-job-with-event-hub-as-input)  
* [建立 Azure Redis 快取執行個體](#create-an-azure-redis-cache)  
* [在 Azure Functions 中建立可將資料寫入至 Azure Redis Cache 的函式](#create-an-azure-function-that-can-write-data-to-the-redis-cache)    
* [使用函式做為輸出，更新串流分析作業](#update-the-stream-analytic-job-with-azure-function-as-output)  
* [檢查 Azure Redis 快取尋找結果](#check-redis-cache-for-results)  

## <a name="create-a-stream-analytics-job-with-event-hubs-as-input"></a>使用事件中樞做為輸入，建立串流分析作業

請遵循[即時詐欺偵測](stream-analytics-real-time-fraud-detection.md)教學課程的說明，建立事件中樞，並啟動事件的產生器應用程式，再建立串流分析作業。 (略過建立查詢及輸出的步驟， 改為參閱以下幾節來設定 Functions 輸出。)

## <a name="create-an-azure-redis-cache-instance"></a>建立 Azure Redis 快取執行個體

1. 使用在[建立快取](../redis-cache/cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache)中所述的步驟，在 Azure Redis 快取中建立快取。  

2. 建立快取後，請在 [設定] 底下，選取 [存取金鑰]。 記下**主要連接字串**。

   ![Azure Redis 快取連接字串的螢幕擷取畫面](./media/stream-analytics-with-azure-functions/image2.png)

## <a name="create-a-function-in-azure-functions-that-can-write-data-to-azure-redis-cache"></a>在 Azure Functions 中建立可將資料寫入至 Azure Redis Cache 的函式

1. 請參閱 Functions 文件的[建立函式應用程式](../azure-functions/functions-create-first-azure-function.md#create-a-function-app)一節。 這將逐步引導您使用 CSharp 語言建立函式應用程式和 [Azure Functions 中的 HTTP 觸發函式](../azure-functions/functions-create-first-azure-function.md#create-function)。  

2. 瀏覽至 **run.csx** 函式。 將它更新為下列程式碼。 (請務必將「\<此處為您的 redis 快取連接字串\>」取代為您在上一節中擷取的 Azure Redis 快取主要連接字串。)  

   ```csharp
   using System;
   using System.Net;
   using System.Threading.Tasks;
   using StackExchange.Redis;
   using Newtonsoft.Json;
   using System.Configuration;

   public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
   {
      log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    
      // Get the request body
      dynamic dataArray = await req.Content.ReadAsAsync<object>();

      // Throw an HTTP Request Entity Too Large exception when the incoming batch(dataArray) is greater than 256 KB. Make sure that the size value is consistent with the value entered in the Stream Analytics portal.

      if (dataArray.ToString().Length > 262144)
      {        
         return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
      }
      var connection = ConnectionMultiplexer.Connect("<your redis cache connection string goes here>");
      log.Info($"Connection string.. {connection}");
    
      // Connection refers to a property that returns a ConnectionMultiplexer
      IDatabase db = connection.GetDatabase();
      log.Info($"Created database {db}");
    
      log.Info($"Message Count {dataArray.Count}");

      // Perform cache operations using the cache object. For example, the following code block adds few integral data types to the cache
      for (var i = 0; i < dataArray.Count; i++)
      {
        string time = dataArray[i].time;
        string callingnum1 = dataArray[i].callingnum1;
        string key = time + " - " + callingnum1;
        db.StringSet(key, dataArray[i].ToString());
        log.Info($"Object put in database. Key is {key} and value is {dataArray[i].ToString()}");
       
      // Simple get of data types from the cache
      string value = db.StringGet(key);
      log.Info($"Database got: {value}");
      }

      return req.CreateResponse(HttpStatusCode.OK, "Got");
    }    

   ```

   串流分析從函式收到「HTTP 要求實體太大」的例外狀況時，會縮減傳送至 Functions 的批次大小。 請使用下列程式碼，在您的函式中檢查確認串流分析並未傳送過大的批次。 請確認函式中使用的最大批次計數和大小值都符合串流分析入口網站中輸入的值。

   ```csharp
   if (dataArray.ToString().Length > 262144)
      {        
        return new HttpResponseMessage(HttpStatusCode.RequestEntityTooLarge);
      }
   ```

3. 在您選擇的文字編輯器中，建立名為 **project.json** 的 JSON 檔案。 使用下列程式碼，並將它儲存在本機電腦上。 此檔案包含 C# 函式所需的 NuGet 封裝相依性。  
   
   ```json
       {
         "frameworks": {
             "net46": {
                 "dependencies": {
                     "StackExchange.Redis":"1.1.603",
                     "Newtonsoft.Json": "9.0.1"
                 }
             }
         }
     }

   ```
 
4. 回到 Azure 入口網站。 至**平台功能**索引標籤，瀏覽至您的函式。 選取 [開發工具] 下的 [App Service 編輯器]。 
 
   ![App Service 編輯器的螢幕擷取畫面](./media/stream-analytics-with-azure-functions/image3.png)

5. 在 App Service 編輯器中，以滑鼠右鍵按一下根目錄，並上傳 **project.json** 檔案。 上傳成功後，重新整理頁面。 現在應該會看到名稱為 **project.lock.json** 的自動產生檔案。 自動產生檔案包含 project.json 檔案中對於 .dll 檔案指定的參考。  

   ![App Service 編輯器的螢幕擷取畫面](./media/stream-analytics-with-azure-functions/image4.png)

 

## <a name="update-the-stream-analytics-job-with-the-function-as-output"></a>使用函式做為輸出，更新串流分析作業

1. 在 Azure 入口網站上，開啟您的串流分析作業。  

2. 瀏覽至您的函式，並選取 [概觀] > [輸出] > [新增]。 若要新增輸出，請對於接收選項選取 **Azure Function**。 新的 Functions 輸出配接器可搭配下列屬性使用：  

   |**屬性名稱**|**說明**|
   |---|---|
   |輸出別名| 在作業的查詢中用來參考該次輸入的易記名稱。 |
   |匯入選項| 您可以使用目前訂用帳戶的函式，如果函式位於另一個訂用帳戶中，也可以手動提供設定。 |
   |函式應用程式| Functions 應用程式的名稱。 |
   |函式| 您的 Functions 應用程式中的函式名稱 (您的 run.csx 函式名稱)。|
   |批次大小上限|設定每個輸出批次 (傳送至您的函式) 的最大大小。 此值預設會設定為 256 KB。|
   |批次計數上限|傳送至函式的每個批次中的事件數上限。 預設值是 100。 這是選用屬性。|
   |Key|可讓您使用其他訂用帳戶中的函式。 提供存取您函式的索引鍵值。 這是選用屬性。|

3. 提供輸出別名的別名。 在此教學課程中，我們將其命名 **saop1** (您可以使用您選擇的任何名稱)。 填入其他詳細資料。  

4. 開啟串流分析作業，並更新下列的查詢。 (如果您將輸出接收命名為不同的名稱，務必取代 "saop1" 文字。)  

   ```sql
    SELECT 
            System.Timestamp as Time, CS1.CallingIMSI, CS1.CallingNum as CallingNum1, 
            CS2.CallingNum as CallingNum2, CS1.SwitchNum as Switch1, CS2.SwitchNum as Switch2
        INTO saop1
        FROM CallStream CS1 TIMESTAMP BY CallRecTime
           JOIN CallStream CS2 TIMESTAMP BY CallRecTime
            ON CS1.CallingIMSI = CS2.CallingIMSI AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
        WHERE CS1.SwitchNum != CS2.SwitchNum
   ```

5. 在命令列中執行下列命令 (使用格式 `telcodatagen.exe [#NumCDRsPerHour] [SIM Card Fraud Probability] [#DurationHours]`)，啟動 telcodatagen.exe 應用程式:  
   
   **telcodatagen.exe 1000 .2 2**
    
6.  啟動串流分析作業。

## <a name="check-azure-redis-cache-for-results"></a>檢查 Azure Redis 快取尋找結果

1. 瀏覽至 Azure 入口網站，並找出您的 Azure Redis 快取。 選取 [主控台]。  

2. 使用 [Redis 快取命令](https://redis.io/commands)確認您的資料在 Redis 快取中。 (此命令接受 Get {key} 的格式。)例如︰

   **Get "12/19/2017 21:32:24 - 123414732"**

   此命令應該會列出指定索引鍵的值：

   ![Azure Redis 快取輸出的螢幕擷取畫面](./media/stream-analytics-with-azure-functions/image5.png)

## <a name="known-issues"></a>已知問題

在 Azure 入口網站中，您嘗試將最大批次大小/最大批次計數值重設為空 (預設值) 時，值將儲存時變更回先前輸入的值。 在此情況下，請手動將預設值輸入欄位。

## <a name="clean-up-resources"></a>清除資源

若不再需要，可刪除資源群組、串流作業和所有相關資源。 刪除作業可避免因為作業使用串流單位而產生費用。 如果您計劃在未來使用該作業，您可以將其停止並在之後需要時重新啟動。 如果您將不繼續使用此作業，請使用下列步驟，刪除本快速入門所建立的所有資源：

1. 從 Azure 入口網站的左側功能表中，按一下 [資源群組]，然後按一下您所建立資源的名稱。  
2. 在資源群組頁面上，按一下 [刪除]，在文字方塊中輸入要刪除之資源的名稱，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立可執行 Azure 函式的簡單串流分析作業，若要深入了解串流分析作業，請繼續進行下一個教學課程：

> [!div class="nextstepaction"]
> [在串流分析作業內執行 JavaScript 使用者定義函式](stream-analytics-javascript-user-defined-functions.md)