---
title: Azure 串流分析的即時 Twitter 情感分析
description: 本文說明如何使用串流分析來執行即時 Twitter 情感分析。 從事件產生到即時儀表板資料的逐步指導。
services: stream-analytics
author: jseb225
ms.author: jeanb
ms.reviewer: jasonh
manager: kfile
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/29/2017
ms.openlocfilehash: 0b920d21486fc0003d8b11bef79bd44be4b28adf
ms.sourcegitcommit: 150a40d8ba2beaf9e22b6feff414f8298a8ef868
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37030609"
---
# <a name="real-time-twitter-sentiment-analysis-in-azure-stream-analytics"></a>Azure 串流分析中的即時 Twitter 情感分析

本文將透過把即時的 Twitter 事件帶入 Azure 事件中樞，來讓您了解如何建立社交媒體分析的情感分析解決方案。 接著，您可以撰寫 Azure 串流分析查詢來分析資料，並儲存結果以供稍後使用，或使用儀表板和 [Power BI](https://powerbi.com/) 來即時提供深入解析。

社交媒體分析工具可協助組織了解熱門話題。 熱門話題就是在社交媒體中有大量文章的話題及意見。 情感分析 (亦稱為*意見挖掘*) 會使用社交媒體分析工具來判斷使用者對產品、概念等等項目的態度。 

即時 Twitter 趨勢分析是分析工具的絕佳例子，因為主題標籤訂閱模型可讓您接聽摘要的特定關鍵字 (主題標籤)，並開發情感分析。

## <a name="scenario-social-media-sentiment-analysis-in-real-time"></a>案例：即時的社交媒體情感分析

擁有新聞媒體網站的公司提供與讀者有直接關聯的網站內容，試圖打敗競爭對手。 公司透過對 Twitter 資料執行即時情感分析，來針對與讀者相關的話題使用社交媒體分析。

為了在 Twitter 上即時找出熱門話題，公司需要即時分析重要話題的推文數量和情感。 換言之，他們需要以該社交媒體摘要為基礎的情感分析分析引擎。

## <a name="prerequisites"></a>先決條件
在本教學課程中，您將會透過用戶端應用程式來連線到 Twitter，並尋找具有特定主題標籤 (可設定) 的推文。 若要執行應用程式並使用 Azure 串流分析來分析推文，您必須具備下列項目：

* Azure 訂用帳戶
* Twitter 帳戶 
* Twitter 應用程式和該應用程式的 [OAuth 存取權杖](https://dev.twitter.com/oauth/overview/application-owner-access-tokens)。 稍後我們會提供如何建立 Twitter 應用程式的高層級指示。
* 用來讀取 Twitter 摘要的 TwitterWPFClient 應用程式。 若要取得此應用程式，請從 GitHub 下載[TwitterWPFClient.zip](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/TwitterClient/TwitterWPFClient.zip) 檔案，然後將套件解壓縮至電腦上的資料夾。 如果您想要看到原始程式碼，並在偵錯工具中執行應用程式，您可以從 [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator) 取得原始程式碼。 

## <a name="create-an-event-hub-for-streaming-analytics-input"></a>建立串流分析輸入的事件中樞

此範例應用程式會產生事件，並將事件發送至 Azure 事件中樞。 Azure 事件中樞是串流分析擷取事件的慣用方法。 如需詳細資訊，請參閱 [Azure 事件中樞文件](../event-hubs/event-hubs-what-is-event-hubs.md)。


### <a name="create-an-event-hub-namespace-and-event-hub"></a>建立事件中樞命名空間和事件中樞
在此程序中，您需要先建立事件中樞命名空間，再將事件中樞新增至該命名空間。 事件中樞命名空間可在邏輯上將相關的事件匯流排執行個體分組。 

1. 登入 Azure 入口網站，按一下 [建立資源] > [物聯網] > [事件中樞]。 

2. 在 [建立命名空間] 刀鋒視窗中，輸入命名空間名稱，例如 `<yourname>-socialtwitter-eh-ns`。 您可以使用任何名稱作為命名空間，但名稱在 URL 中必須有效，而且在整個 Azure 內必須唯一的。 
    
3. 選取訂用帳戶並建立或選擇資源群組，然後按一下 [建立]。 

    ![建立事件中樞命名空間](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-eventhub-namespace.png)
 
4. 當命名空間完成部署時，請在 Azure 資源清單中尋找事件中樞命名空間。 

5. 按一下新的命名空間，然後在命名空間刀鋒視窗中，按一下 [+&nbsp;事件中樞]。 

    ![建立新事件中樞的 [新增事件中樞] 按鈕 ](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-eventhub-button.png)    
 
6. 將新的事件中樞命名為 `socialtwitter-eh`。 您可以使用不同的名稱。 如果這樣做，請記下來，因為稍後需要用到此名稱。 您不需要為事件中樞設定其他任何選項。

    ![建立新事件中樞的刀鋒視窗](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-eventhub.png)
 
7. 按一下頁面底部的 [新增] 。


### <a name="grant-access-to-the-event-hub"></a>授權存取事件中樞

事件中樞必須有原則來允許適當的存取權，流程才能將資料傳送到事件中樞。 存取原則會產生包含授權資訊的連接字串。

1.  在事件命名空間刀鋒視窗中，按一下 [事件中樞]，然後按一下新事件中樞的名稱。

2.  在事件中樞刀鋒視窗中，按一下 [共用存取原則]，然後按一下 [+&nbsp;新增]。

    >[!NOTE]
    >請確定您正在使用事件中樞，而不是事件中樞命名空間。

3.  新增名為 `socialtwitter-access` 的原則，然後在 [宣告] 中，選取 [管理]。

    ![建立新事件中樞存取原則的刀鋒視窗](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-shared-access-policy-manage.png)
 
4.  按一下頁面底部的 [新增] 。

5.  部署原則之後，在共用存取原則清單中按一下此原則。

6.  找出標示為 [連接字串-主要索引鍵] 的方塊，按一下連接字串旁邊的複製按鈕。 
    
    ![從存取原則複製主要的連接字串索引鍵](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-shared-access-policy-copy-connection-string.png)
 
7.  將連接字串貼到文字編輯器。 稍微編輯一下此連接字串，下一節需要用到它。

    連接字串如下所示：

        Endpoint=sb://YOURNAME-socialtwitter-eh-ns.servicebus.windows.net/;SharedAccessKeyName=socialtwitter-access;SharedAccessKey=Gw2NFZw6r...FxKbXaC2op6a0ZsPkI=;EntityPath=socialtwitter-eh

    請注意，連接字串包含多個機碼值組，以分號分隔：`Endpoint`、`SharedAccessKeyName`、`SharedAccessKey` 和 `EntityPath`。  

    > [!NOTE]
    > 基於安全考量，我們已移除範例中連接字串的某些部分。

8.  在文字編輯器中，移除連接字串的 `EntityPath` 組 (別忘了移除前面的分號)。 完成時，連接字串會如下所示：

        Endpoint=sb://YOURNAME-socialtwitter-eh-ns.servicebus.windows.net/;SharedAccessKeyName=socialtwitter-access;SharedAccessKey=Gw2NFZw6r...FxKbXaC2op6a0ZsPkI=


## <a name="configure-and-start-the-twitter-client-application"></a>設定並啟動 Twitter 用戶端應用程式
用戶端應用程式會直接從 Twitter 取得推文事件。 為了這樣做，它需要權限來呼叫 Twitter 串流 API。 若要設定該權限，您需要在 Twitter 中建立應用程式，以產生唯一認證 (例如 OAuth 權杖)。 然後，您可以設定用戶端應用程式在呼叫 API 時使用這些認證。 

### <a name="create-a-twitter-application"></a>建立 Twitter 應用程式
如果您還沒有可用於本教學課程的 Twitter 應用程式，您可以建立一個。 您必須已經有 Twitter 帳戶。

> [!NOTE]
> 在 Twitter 中建立應用程式及取得金鑰、秘密和權杖的確切流程可能不同。 如果這些指示不符合您在 Twitter 網站上看到的內容，請參閱 Twitter 開發人員文件。

1. 移至 [Twitter 應用程式管理網頁](https://apps.twitter.com/)。 

2. 建立新的應用程式。 

    * 在網站 URL 中，指定有效的 URL。 不必是即時網站。 (您不能只指定 `localhost`。)
    * 將回呼欄位保留空白。 您用於本教學課程的用戶端應用程式不需要回呼。

    ![在 Twitter 中建立應用程式](./media/stream-analytics-twitter-sentiment-analysis-trends/create-twitter-application.png)

3. (選擇性) 將應用程式的權限變更為唯讀。

4. 建立應用程式後，移至 [金鑰和存取權杖]  網頁。

5. 按一下按鈕可以產生存取權杖和存取權杖祕密。

請備妥此資訊，在下一個程序中需要用到。

>[!NOTE]
>Twitter 應用程式的金鑰和秘密可用來存取您的 Twitter 帳戶。 這項資訊要視為機密，就像看待您的 Twitter 密碼一樣。 例如，請勿將這項資訊內嵌在您提供給其他人的應用程式中。 


### <a name="configure-the-client-application"></a>設定用戶端應用程式
我們已建立用戶端應用程式，可利用 [Twitter 的串流 API](https://dev.twitter.com/streaming/overview) 來連線至 Twitter 資料，以收集一組特定話題相關的推文事件。 該應用程式使用 [Sentiment140](http://help.sentiment140.com/) 開放原始碼工具，將下列情感值指派給每個推文：

* 0 = 負值
* 2 = 中性
* 4 = 正值

在指派情感值給推文事件之後，推文事件就會發送至您稍早建立的事件中樞。

在應用程式執行之前，您需要提供一些資訊，例如 Twitter 金鑰和事件中樞連接字串。 您可以依照這些方法來提供組態資訊：

* 執行應用程式，然後使用應用程式的 UI 來輸入金鑰、祕密和連接字串。 如果您這麼做，組態資訊會用於目前的工作階段，但不會儲存。
* 編輯應用程式的 .config 檔案，在其中設定一些值。 這種方法會保存組態資訊，但也表示這項潛在的敏感資訊會以純文字儲存在電腦上。

下列程序說明這兩種方法。 

1. 請確定您已下載並解壓縮 [TwitterWPFClient.zip](https://github.com/Azure/azure-stream-analytics/blob/master/Samples/TwitterClient/TwitterWPFClient.zip) 應用程式，如必要條件中所示。

2. 若要在執行階段設定值 (且僅針對目前工作階段)，請執行 `TwitterWPFClient.exe` 應用程式。 當應用程式提示您時，請輸入下列值：

    * Twitter 取用者金鑰 (API 金鑰)。
    * Twitter 取用者祕密 (API 祕密)。
    * Twitter 存取權杖。
    * Twitter 存取權杖祕密。
    * 您稍早儲存的連接字串資訊。 請確定您使用已移除 `EntityPath` 機碼值組的的連接字串。
    * 您用以判斷情感的 Twitter 關鍵字。

   ![執行中的 TwitterWpfClient 應用程式，顯示隱藏的設定](./media/stream-analytics-twitter-sentiment-analysis-trends/wpfclientlines.png)

3. 若要持續設定值，請使用文字編輯器開啟 TwitterWpfClient.exe.config 檔案。 然後在 `<appSettings>` 元素中，執行下列動作：

    * 將 `oauth_consumer_key` 設為 Twitter 取用者金鑰 (API 金鑰)。 
    * 將 `oauth_consumer_secret` 設為 Twitter 取用者祕密 (API 祕密)。
    * 將 `oauth_token` 設為 Twitter 存取權杖。
    * 將 `oauth_token_secret` 設為 Twitter 存取權杖祕密。

    稍後在 `<appSettings>` 元素中，進行下列變更：

    * 將 `EventHubName` 設為事件中樞名稱 (也就是實體路徑的值)。
    * 將 `EventHubNameConnectionString` 設為連接字串。 請確定您使用已移除 `EntityPath` 機碼值組的的連接字串。

    `<appSettings>` 區段如下列範例所示。 (為了清楚起見和基於安全考量，我們已遮蔽幾行並移除某些字元。)

    ![文字編輯器中的 TwitterWpfClient 應用程式組態檔，顯示 Twitter 金鑰和祕密，以及事件中樞連接字串資訊](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-tiwtter-app-config.png)
 
4. 如果您尚未啟動此應用程式，現在就執行 TwitterWpfClient.exe。 

5. 按一下綠色啟動按鈕來收集社交情感。 您會看到推文事件連同 **CreatedAt**、**Topic** 和 **SentimentScore** 值，一起傳送到事件中樞。

    ![執行中的 TwitterWpfClient 應用程式，顯示推文清單](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-twitter-app-listing.png)

    >[!NOTE]
    >如果您看到錯誤，而且在視窗下半部沒有看到推文的資料流，請仔細檢查金鑰和祕密。 也請檢查連接字串 (請確定不包含 `EntityPath` 索引鍵和值。)


## <a name="create-a-stream-analytics-job"></a>建立串流分析作業

既然正在從 Twitter 即時串流推文事件，您可以設定串流分析作業來即時分析這些事件。

1. 在 Azure 入口網站中，按一下 [建立資源] > [物聯網] > [串流分析作業]。

2. 將作業命名為 `socialtwitter-sa-job`，並指定訂用帳戶、資源群組和位置。

    最好將作業和事件中樞放在相同的區域，以達到最佳效能，在區域之間傳送資料也不需要付費。

    ![建立新的串流分析作業](./media/stream-analytics-twitter-sentiment-analysis-trends/newjob.png)

3. 按一下頁面底部的 [新增] 。

    即可建立作業，入口網站會顯示作業詳細資料。


## <a name="specify-the-job-input"></a>指定工作輸入

1. 在串流分析作業中，於作業刀鋒視窗中央的 [作業拓撲] 下，按一下 [輸入]。 

2. 在 [輸入] 刀鋒視窗中，按一下 [+&nbsp;新增]，然後在刀鋒視窗中填入這些值：

    * **輸入別名**：使用名稱 `TwitterStream`。 如果使用不同的名稱，請記下來，因為稍後需要用到。
    * **來源類型**：選取 [資料流]。
    * **來源**：選取 [事件中樞]。
    * **匯入選項**：選取 [使用目前訂用帳戶的事件中樞]。 
    * **服務匯流排命名空間**：選取您稍早建立的事件中樞命名空間 (`<yourname>-socialtwitter-eh-ns`)。
    * **事件中樞**：選取您稍早建立的事件中樞 (`socialtwitter-eh`)。
    * **事件中樞原則名稱**：選取您稍早建立的存取原則 (`socialtwitter-access`)。

    ![建立串流分析作業的新輸入](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-twitter-new-input.png)

3. 按一下頁面底部的 [新增] 。


## <a name="specify-the-job-query"></a>指定作業查詢

串流分析支援說明轉換的簡單、宣告式查詢模型。 若要深入了解語言，請參閱 [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)。  本教學課程可協助您撰寫以及測試數個 Twitter 資料查詢。

為了比較提及不同話題的次數，您可以使用[輪轉視窗](https://msdn.microsoft.com/library/azure/dn835055.aspx)，依話題每隔五秒取得一次提及次數。

1. 關閉 [輸入] 刀鋒視窗 (如果尚未關閉)。

2. 在作業刀鋒視窗中，按一下 [查詢] 方塊。 Azure 會列出作業已設定的輸入和輸出，還可讓您建立查詢，在輸入資料流傳送至輸出時進行轉換。

3. 請確定 TwitterWpfClient 應用程式正在執行。 

3. 在 [查詢] 刀鋒視窗中，按一下 `TwitterStream` 輸入旁邊的點，然後選取 [來自輸入的範例資料]。

    ![用於在串流分析作業中輸入範例資料的功能表選項，已選取 [來自輸入的範例資料]](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-sample-data-from-input.png)

    這會開啟刀鋒視窗，讓您決定要花多少時間來讀取輸入資料流，以指定取得多少範例資料。

4. 將 [分鐘] 設定為 3，然後按一下 [確定]。 
    
    ![輸入資料流的取樣選項，已選取 [3 分鐘]。](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-input-create-sample-data.png)

    Azure 會從輸入資料流取樣 3 分鐘的資料，備妥範例資料時會通知您。 (這需要等一下。) 

    範例資料會暫時儲存，開啟查詢視窗時即可使用。 如果您關閉查詢視窗，則會捨棄範例資料，您將必須建立一組新的範例資料。 

5. 在程式碼編輯器中，將查詢變更如下：

    ```
    SELECT System.Timestamp as Time, Topic, COUNT(*)
    FROM TwitterStream TIMESTAMP BY CreatedAt
    GROUP BY TUMBLINGWINDOW(s, 5), Topic
    ```

    如果您不是使用 `TwitterStream` 作為輸入的別名，請在查詢中以您的別名取代 `TwitterStream`。  

    此查詢會使用 **TIMESTAMP BY** 關鍵字，在裝載中指定一個暫時運算時會用到的時間戳記欄位。 如果未指定這個欄位，則會使用每個事件到達事件中樞的時間，執行時間範圍設定作業。 若要深入了解，請參閱[串流分析查詢參考 (英文)](https://msdn.microsoft.com/library/azure/dn834998.aspx) 中的＜到達時間與應用程式時間之比較 (英文)＞一節。

    此查詢也可以使用 **System.Timestamp** 屬性存取每個視窗結束時的時間戳記。

5. 按一下 [ **測試**]。 這時會針對您已取樣的資料執行查詢。
    
6. 按一下 [檔案] 。 這會將查詢儲存在串流分析作業中。 (不會儲存範例資料。)


## <a name="experiment-using-different-fields-from-the-stream"></a>使用資料流中的不同欄位進行實驗 

下表列出屬於 JSON 串流資料的欄位。 您可在查詢編輯器中自由進行試驗。

|JSON 屬性 | 定義|
|--- | ---|
|CreatedAt | 建立推文的時間|
|話題 | 符合所指定關鍵字的話題|
|SentimentScore | 來自 Sentiment140 的情感分數|
|作者 | 傳送推文的 Twitter 控制代碼|
|文字 | 推文的全文|


## <a name="create-an-output-sink"></a>建立輸出接收器

您現在已定義事件資料流、用來內嵌事件的事件中樞，以及用來轉換資料流的查詢。 最後一個步驟是定義作業的輸出接收。  

在本教學課程中，您會將作業查詢所彙總的推文事件寫入 Azure Blob 儲存體。  根據您的應用程式需求，您也可以將結果發送至 Azure SQL Database、Azure 資料表儲存體、事件中樞或 Power BI。

## <a name="specify-the-job-output"></a>指定作業輸出

1. 在 [作業拓撲] 區段中，按一下 [輸出] 方塊。 

2. 在 [輸出] 刀鋒視窗中，按一下 [+&nbsp;新增]，然後在刀鋒視窗中填入這些值：

    * **輸出別名**：使用名稱 `TwitterStream-Output`。 
    * **接收器**：選取 [Blob 儲存體]。
    * **匯入選項**：選取 [使用來自目前訂用帳戶的 Blob 儲存體]。
    * **儲存體帳戶**。 選取 [建立新儲存體帳戶]。
    * **儲存體帳戶** (第二個方塊)。 輸入 `YOURNAMEsa`，其中 `YOURNAME` 是您的名稱或另一個唯一字串。 名稱只能使用小寫字母和數字，而且在整個 Azure 中必須是唯一的。 
    * **容器**。 輸入 `socialtwitter`。
    儲存體帳戶名稱和容器名稱一起用來提供 blob 儲存體的 URI，例如： 

    `http://YOURNAMEsa.blob.core.windows.net/socialtwitter/...`
    
    ![串流分析作業的 [新輸出] 刀鋒視窗](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-create-output-blob-storage.png)
    
4. 按一下頁面底部的 [新增] 。 

    Azure 會建立儲存體帳戶，並自動產生金鑰。 

5. 關閉 [輸出] 刀鋒視窗。 


## <a name="start-the-job"></a>啟動工作

已指定作業輸入、查詢和輸出。 您已準備好啟動串流分析作業。

1. 請確定 TwitterWpfClient 應用程式正在執行。 

2. 在作業刀鋒視窗中，按一下 [啟動]。

    ![啟動串流分析工作](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-sa-job-start-output.png)

3. 在 [啟動作業] 刀鋒視窗的 [作業輸出開始時間] 中，選取 [現在]，然後按一下 [啟動]。 

    ![串流分析作業的 [啟動作業] 刀鋒視窗](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-sa-job-start-job-blade.png)

    作業啟動後，Azure 會通知您，在作業刀鋒視窗中，狀態會顯示為 [執行中]。

    ![工作正在執行](./media/stream-analytics-twitter-sentiment-analysis-trends/jobrunning.png)

## <a name="view-output-for-sentiment-analysis"></a>檢視情感分析輸出

在您的作業已開始執行並處理即時 Twitter 串流之後，您就可以檢視情感分析的輸出。

您可以使用 [Azure 儲存體總管](https://storageexplorer.com/)或 [Azure 總管](http://www.cerebrata.com/products/azure-explorer/introduction)之類的工具，即時檢視您的作業輸出。 從這裡，您可以使用 [Power BI](https://powerbi.com/) 擴充您的應用程式來包含自訂儀表板，如下列螢幕擷取畫面所示：

![Power BI](./media/stream-analytics-twitter-sentiment-analysis-trends/power-bi.png)


## <a name="create-another-query-to-identify-trending-topics"></a>建立另一個查詢來識別熱門話題

另一個可用來了解 Twitter 情緒的查詢是根據[滑動視窗](https://msdn.microsoft.com/library/azure/dn835051.aspx)。 若要識別熱門話題，您需要找出在指定的時間內提及次數超過臨界值的話題。

基於本教學課程的目的，您將會找出過去 5 秒內提及次數超過 20 次的話題。

1. 在作業刀鋒視窗中，按一下 [停止] 來停止作業。 

2. 在 [作業拓撲] 區段中，按一下 [查詢] 方塊。 

3. 將查詢變更如下：

    ```    
    SELECT System.Timestamp as Time, Topic, COUNT(*) as Mentions
    FROM TwitterStream TIMESTAMP BY CreatedAt
    GROUP BY SLIDINGWINDOW(s, 5), topic
    HAVING COUNT(*) > 20
    ```

4. 按一下 [檔案] 。

5. 請確定 TwitterWpfClient 應用程式正在執行。 

6. 按一下 [啟動]，使用新的查詢來重新啟動作業。


## <a name="get-support"></a>取得支援
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
