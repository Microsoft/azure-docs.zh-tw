---
title: 教學課程：使用 Azure 入口網站建立和管理串流分析作業 | Microsoft Docs
description: 本教學課程提供如何使用 Azure 串流分析來分析通話串流中詐騙電話的端對端實例。
services: stream-analytics
author: sidramadoss
ms.author: sidram
manager: kfile
ms.service: stream-analytics
ms.workload: data-services
ms.topic: tutorial
ms.custom: mvc
ms.date: 04/04/2018
ms.openlocfilehash: 6b924e0555ea7a57f8d5e5309a266b6d2fb44f44
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43702522"
---
# <a name="create-a-stream-analytics-job-to-analyze-phone-call-data-and-visualize-results-in-a-power-bi-dashboard"></a>建立串流分析作業來分析通話資料，並且在 Power BI 儀表板中將結果視覺化
 
本教學課程會示範如何使用 Azure 串流分析來分析用戶端應用程式所產生的範例通電。 用戶端應用程式所產生的通話資料包含某些詐騙電話，而我們會定義串流分析作業來篩選這類電話。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 產生範例通話資料並將通話傳送至 Azure 事件中樞  
> * 建立串流分析作業   
> * 設定作業的輸入和輸出  
> * 定義查詢以篩選詐騙電話  
> * 測試和啟動作業  
> * 在 Power BI 中將結果視覺化 

## <a name="prerequisites"></a>必要條件

開始之前，請確定您具有下列項目：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/)。  
* 登入 [Azure 入口網站](https://portal.azure.com/)。  
* 從 Microsoft 下載中心下載通話事件產生器應用程式 [TelcoGenerator.zip](http://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip)，或從 [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator) 取得原始程式碼。  

## <a name="create-an-azure-event-hub"></a>建立 Azure 事件中樞 

您應該先將資料傳送至 Azure，串流分析才可以分析詐騙電話資料流。 在本教學課程中，您會使用 [Azure 事件中樞](https://docs.microsoft.com/azure/event-hubs/event-hubs-what-is-event-hubs)將資料傳送至 Azure。 在本教學課程中，您會建立事件中樞，然後由事件產生器應用程式將通話資料傳送至該事件中樞。 請執行下列步驟來建立事件中樞：

1. 登入 Azure 管理入口網站。  
2. 選取 [建立資源] > [物聯網] > [事件中樞]。  

   ![尋找事件中樞](media/stream-analytics-manage-job/find-eh.png)
3. 在 [建立命名空間] 窗格中填入下列值：  

   |**設定**  |**建議的值** |**說明**  |
   |---------|---------|---------|
   |名稱     | myEventHubNS        |  用以識別事件中樞命名空間的唯一名稱。       |
   |訂用帳戶     |   \<您的訂用帳戶\>      |   選取您要在其中建立事件中樞的 Azure 訂用帳戶。      |
   |資源群組     |   MyASADemoRG      |  選取 [新建]，然後為您的帳戶輸入新的資源群組名稱。       |
   |位置     |   美國西部 2      |    可以部署事件中樞命名空間的位置。     |

4. 對其餘的設定使用預設選項，並選取 [建立]。  

   ![建立事件中樞命名空間](media/stream-analytics-manage-job/create-ehns.png)

5. 當命名空間部署完成時，請移至 [所有資源] > 在 Azure 資源清單中尋找 "myEventHubNS" > 選擇開啟它。  
6. 接下來，選取 [+事件中樞] > [名稱]，將事件中樞命名為 "MyEventHub"。 您可以使用不同的名稱。 對其餘的設定使用預設選項，選取 [建立] 並等待部署成功。

   ![建立事件中樞](media/stream-analytics-manage-job/create-eh.png)

### <a name="grant-access-to-the-event-hub-and-get-a-connection-string"></a>授權存取事件中樞並取得連接字串

事件中樞必須先具有允許適當存取權的原則，應用程式才能將資料傳送到 Azure 事件中樞。 存取原則會產生包含授權資訊的連接字串。

1. 巡覽至您在上一個步驟中建立的 [事件中樞]，名為 "MyEventHub" > 從事件中樞窗格選取 [共用存取原則] > 選取 [+新增]。  
2. 將原則名稱設定為 [Mypolicy] > 然後選取 [管理] > 選取 [建立]。  

   ![建立事件中樞共用存取原則](media/stream-analytics-manage-job/create-ehpolicy.png)

3. 部署原則之後，選擇開啟原則，尋找 [連接字串主索引鍵] 並選取連接字串旁邊的 [複製]。  
4. 將連接字串貼到文字編輯器。 您在下一節中需要此連接字串。  

   連接字串如下所示：

   `Endpoint=sb://<Your event hub namespace>.servicebus.windows.net/;SharedAccessKeyName=<Your shared access policy name>;SharedAccessKey=<generated key>;EntityPath=<Your event hub name>` 

   請注意，連接字串包含多個以分號分隔的索引鍵值組：Endpoint、SharedAccessKeyName、SharedAccessKey 和 EntityPath。  

5. 移除連接字串中的 EntityPath 值組 (別忘了移除前面的分號)。

## <a name="start-the-event-generator-application"></a>啟動事件產生器應用程式

啟動 TelcoGenerator 應用程式之前，您應該先將它設定為將資料傳送到您先前建立的事件中樞。

1. 將 [TelcoGenerator.zip](http://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip) 檔案的內容解壓縮。  
2. 在您選擇的文字編輯器中開啟 `TelcoGenerator\TelcoGenerator\telcodatagen.exe.config` 檔案 (有一個以上的 .config 檔案，請確定是開啟正確的檔案。)  

3. 使用下列詳細資料更新組態檔中的 <appSettings> 元素：

   * 將 EventHubName 索引鍵的值設為連接字串中的 EntityPath 值。  
   * 將 Microsoft.ServiceBus.ConnectionString 索引鍵的值設為連接字串，不含您在上一節中從步驟 5 取得的 EntityPath 值。

4. 儲存檔案。  
5. 接下來開啟命令視窗，切換至您解壓縮 TelcoGenerator 應用程式的資料夾並輸入下列命令：

   ```
   telcodatagen.exe 1000 .2 2
   ```

   此命令採用下列參數︰
   * **每小時的通話資料記錄筆數**。  
   * **詐騙機率的百分比** - 也就是應用程式模擬詐騙電話的頻率。 值 .2 表示大約 20% 的通話記錄可能是詐騙。  
   * **以小時為單位的持續時間** - 應用程式應該執行的時數。 您也可以在命令列結束程序 (Ctrl+C)，隨時停止應用程式。

   幾秒之後，隨著應用程式將通話記錄傳送到事件中樞，它會開始在螢幕上顯示通話記錄。 通話資料包含下列欄位：

   |**記錄**  |**定義**  |
   |---------|---------|
   |CallrecTime    |  通話開始時間的時間戳記。       |
   |SwitchNum     |  用來接通電話的電話交換機。 在此範例中，交換機是代表發話國的字串 (美國、中國、英國、德國或澳大利亞)。       |
   |CallingNum     |  來電者的電話號碼。       |
   |CallingIMSI     |  國際行動用戶識別碼 (IMSI)。 這是來電者的唯一識別碼。       |
   |CalledNum     |   受話方的電話號碼。      |
   |CalledIMSI     |  國際行動用戶識別碼 (IMSI)。 這是受話方的唯一識別碼。       |

## <a name="create-a-stream-analytics-job"></a>建立串流分析作業 

既然已取得通話事件的串流，您可以建立串流分析作業，以從事件中樞讀取資料。

1. 若要建立串流分析作業，請巡覽至 Azure 入口網站。  

2. 選取 [建立資源] > [物聯網] > [串流分析作業]。  

3. 在 [新增串流分析作業] 窗格中填入下列值：  

   |**設定**  |**建議的值**  |**說明**  |
   |---------|---------|---------|
   |作業名稱     |  ASATutorial       |   用以識別事件中樞命名空間的唯一名稱。      |
   |訂用帳戶    |  \<您的訂用帳戶\>   |   選取您要在其中建立作業的 Azure 訂用帳戶。       |
   |資源群組   |   MyASADemoRG      |   選取 [使用現有的]，然後為您的帳戶輸入新的資源群組名稱。      |
   |位置   |    美國西部 2     |      可以部署作業的位置。 建議將作業和事件中樞放在相同的區域，以達到最佳效能，在區域之間傳送資料也不需要付費。      |
   |裝載環境    | 雲端        |     串流分析作業可以部署到雲端或邊緣裝置。 雲端部分可讓您部署到 Azure 雲端，邊緣裝置部分可讓您部署到 IoT 邊緣裝置。    |
   |串流單位     |    1       |      串流單位代表執行作業所需的計算資源。 根據預設，此值設定為 1。 若要深入了解如何調整串流單位，請參閱[了解與調整串流單位](stream-analytics-streaming-unit-consumption.md)一文。      |

   ![建立工作](media/stream-analytics-manage-job/create-a-job.png)   

4. 對其餘的設定使用預設選項，選取 [建立] 並等待部署成功。

## <a name="configure-job-input"></a>設定作業輸入

下一個步驟是定義作業的輸入來源，以讀取資料。 在此教學課程中，您將使用在上一節中建立的事件中樞作為輸入。 請執行下列步驟，以設定作業的輸入：

1. 從 Azure 入口網站開啟 [所有資源] 窗格，然後尋找 ASATutorial 串流分析作業。  

2. 在串流分析作業窗格的 [作業拓撲] 區段中，選取 [輸入] 選項。  

3. 選取 [+新增串流輸入] (參考輸入會參照您不會在本教學課程中使用的靜態查閱資料)、[事件中樞]，然後在窗格中填入下列值：  

   |**設定**  |**建議的值**  |**說明**  |
   |---------|---------|---------|
   |輸入別名     |  CallStream       |  提供可識別輸入的易記名稱。 輸入別名只可包含英數字元、連字號與底線，且其長度必須介於 3-63 個字元之間。       |
   |訂用帳戶    |   \<您的訂用帳戶\>      |   選取您在其中建立事件中樞的 Azure 訂用帳戶。 事件中樞可位於與串流分析作業相同或不同的訂用帳戶。       |
   |事件中樞命名空間    |  MyEventHubNS       |  選取您在上一節中建立的事件中樞命名空間。 您目前訂用帳戶中所有可用的事件中樞命名空間都會列在下拉式清單中。       |
   |事件中樞名稱    |   MyEventHub      |  選取您在上一節中建立的事件中樞。 您目前訂用帳戶中所有可用的事件中樞都會列在下拉式清單中。       |
   |事件中樞原則名稱   |  Mypolicy       |  選取您在上一節中建立的事件中樞共用存取原則。 您目前訂用帳戶中所有可用的事件中樞原則都會列在下拉式清單中。       |

   ![設定輸入](media/stream-analytics-manage-job/configure-input.png) 

4. 對其餘的設定使用預設選項，選取 [儲存] 並等待部署成功。

## <a name="configure-job-output"></a>設定作業輸出 

最後一個步驟是定義作業的輸出接收，也就是可以寫入轉換後資料的地方。 在此教學課程中，您會將結果輸出至 Power BI 並將資料視覺化。 請執行下列步驟，以設定作業的輸出：

1. 從 Azure 入口網站開啟 [所有資源] 窗格，然後尋找 ASATutorial 串流分析作業。  

2. 在串流分析作業窗格的 [作業拓撲] 區段中，選取 [輸出] 選項。  

3. 選取 [+新增] > [Power BI] 並填入下列詳細資料 (您可以提供易記名稱來識別 [輸出別名]、[資料集名稱] 和 [資料表名稱]，如下表所示)，然後選取 [授權]：  

   |**設定**  |**建議的值**  |
   |---------|---------|---------|
   |輸出別名  |  MyPBIoutput  |
   |資料集名稱  |   ASAdataset  | 
   |資料表名稱 |  ASATable  | 

   ![設定輸出](media/stream-analytics-manage-job/configure-output.png)  

4. 選取 [授權] 之後，快顯視窗隨即開啟，而系統會要求您提供認證來驗證您的 Power BI 帳戶。 一旦授權成功，請 [儲存] 設定。 

## <a name="define-a-query-to-analyze-input-data"></a>定義查詢來分析輸入資料

在串流分析作業安裝程式讀取傳入資料流之後，下一個步驟是建立轉換以即時分析資料。 您可使用[串流分析查詢語言](https://msdn.microsoft.com/library/dn834998.aspx)來定義轉換查詢。 在本教學課程中，您可定義查詢，以從電話資料偵測詐騙電話。 

在此範例中，我們假設詐騙電話是源自相同使用者但在不同位置的電話，而兩通電話之間的持續時間為五秒。 例如，按照常理，同一位使用者不可能同時從美國和澳大利亞打電話。 若要為串流分析作業定義轉換查詢，請執行下列步驟：

1. 從 Azure 入口網站開啟 [所有資源] 窗格，然後開啟您先前建立的 **ASATutorial** 串流分析作業。  

2. 在串流分析作業窗格的 [作業拓撲] 區段中，選取 [查詢] 選項。 快顯視窗會列出作業已設定的輸入和輸出，還可讓您建立查詢來轉換輸入資料流。  

3. 接下來，使用下列資料取代編輯器中的現有查詢，此查詢會在有 5 秒間隔的通話資料上執行自我聯結：

   ```sql
   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "MyPBIoutput"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime
   ON CS1.CallingIMSI = CS2.CallingIMSI
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

   若要檢查詐騙電話，您應該根據 `CallRecTime` 值自我聯結串流資料。 然後，您可以尋找 `CallingIMSI` 值 (發話號碼) 相同但 `SwitchNum` 值 (發話國) 不同的通話記錄。 當您在串流資料中使用 JOIN 作業時，聯結必須稍微限制相符的資料列在時間上可以相隔多久。 由於串流資料無止盡，所以會使用 [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/datediff-azure-stream-analytics) 函式，在聯結的 ON 子句內指定關聯性的時間界限。 

   此查詢就像一般 SQL 聯結一樣，差別在於 DATEDIFF 函式。 此查詢中使用的 DATEDIFF 函式是串流分析專用的，必須出現在 `ON...BETWEEN` 子句中。  

4. [儲存] 查詢。  

   ![定義查詢](media/stream-analytics-manage-job/define-query.png) 

## <a name="test-your-query"></a>測試查詢

您可以從查詢編輯器測試查詢，而且需有範例資料才能測試查詢。 在這個逐步解說中，您將從傳入事件中樞的通話串流擷取範例資料。 請執行下列步驟來測試查詢：

1. 請確定 TelcoGenerator 應用程式正在執行並產生通話記錄。  

2. 在 [查詢] 窗格中，選取 CallStream 輸入旁邊的點，然後選取 [來自輸入的範例資料]。 這會開啟一個窗格，讓您指定要從輸入資料流讀取多少範例資料。  

   ![範例輸入資料](media/stream-analytics-manage-job/sample-input-data.png)  

3. 將 [分鐘] 設定為 3，然後選取 [確定]。 Azure 會從輸入資料流取樣 3 分鐘的資料，並且在備妥範例資料時通知您。 您可以從通知列檢視取樣的狀態。 

   範例資料會暫時儲存，開啟查詢視窗時即可使用。 如果您關閉查詢視窗，則會捨棄範例資料，您將必須建立一組新的範例資料。 或者，您可以[從 GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample Data/telco.json) 取得含有範例資料的 .json 檔案，然後上傳該 .json 檔案，以當作 CallStream 輸入的範例資料使用。  

4. 選取 [測試] 以測試查詢，您應會看到如以下螢幕擷取畫面所示的輸出結果：  

   ![測試輸出](media/stream-analytics-manage-job/test-output.png)

## <a name="start-the-job-and-visualize-output"></a>啟動作業並將輸出視覺化

1. 若要啟動作業，請巡覽至您作業的 [概觀] 窗格，然後選取 [啟動]。  

2. 針對作業輸出開始時間選取 [現在] 並選取 [啟動]。 作業會在幾分鐘內啟動，您可以在通知列中檢視狀態。  

3. 作業成功之後，請巡覽至 [Powerbi.com](https://powerbi.com/)，然後使用公司或學校帳戶登入。 如果串流分析作業查詢有輸出結果，您會看到資料集已建立。 巡覽至 [資料集] 索引標籤，您可以檢視名為 "ASAdataset" 的資料集。  

4. 從您的工作區中選取 [+建立]。 建立新的儀表板並命名為 Fraudulent Calls。 您會將兩個圖格新增至此儀表板，其中一個圖格用來檢視指定執行個體的詐騙通話計數，而另一個圖格具有折線圖視覺效果。  

5. 在視窗頂端，選取 [新增圖格] 並選取 [自訂串流資料] > [下一部] > 選擇 [ASAdataset] > 針對 [視覺效果類型] 選取 [卡片] > 而且 [欄位] 為 [fraudulentcalls]。 選取 [下一步] > 輸入圖格的名稱，選取 [套用]。  

   ![建立圖格](media/stream-analytics-manage-job/create-tiles.png)

6. 使用下列選項，再次依照步驟 4 執行：
   * 在 [視覺效果類型] 中選取 [折線圖]。  
   * 新增軸並選取 [windowend]。  
   * 新增值並選取 [fraudulentcalls]。  
   * 在 [要顯示的時間範圍] 中，選取過去 10 分鐘。  

7. 新增這兩個圖格之後，您的儀表板看起來如下列螢幕擷取畫面。 您會發現，如果您的事件中樞傳送者應用程式和串流分析應用程式正在執行，則 PowerBI 儀表板會在新資料送達時定期更新。  

   ![Power BI 結果](media/stream-analytics-manage-job/power-bi-results.png)

## <a name="embedding-your-powerbi-dashboard-in-a-web-application"></a>在 Web 應用程式中內嵌 PowerBI 儀表板

在這部分的教學課程中，您將使用 PowerBI 團隊所建立的範例 [ASP.NET](http://asp.net/) Web 應用程式來內嵌您的儀表板。 如需內嵌儀表板的詳細資訊，請參閱[內嵌 Power BI](https://docs.microsoft.com/power-bi/developer/embedding) 一文。

在本教學課程中，我們會遵循「使用者擁有資料」應用程式的步驟。 若要設定應用程式，請移至 [PowerBI-Developer-Samples](https://github.com/Microsoft/PowerBI-Developer-Samples) Github 存放庫，並遵循**使用者擁有資料**一節之下的指示 (使用 **integrate-dashboard-web-app** 小節之下的重新導向和首頁 URL)。 因為我們使用儀表板範例，所以使用位於 [GitHub 存放庫](https://github.com/Microsoft/PowerBI-Developer-Samples/tree/master/User Owns Data/integrate-dashboard-web-app)中的 integrate-dashboard-web-app 範例程式碼。
一旦在瀏覽器中執行應用程式，請遵循下列步驟將先前建立的儀表板內嵌到網頁中：

1. 選取 [登入 Power BI]，以將您的 PowerBI 帳戶中儀表板的存取權授與應用程式。  

2. 選取 [取得儀表板] 按鈕，即可在資料表中顯示您的帳戶儀表板。 尋找您先前建立的儀表板名稱 (powerbi-embedded-dashboard)，並且複製對應的 **EmbedUrl**。  

3. 最後，將 **EmbedUrl** 貼到對應的文字欄位中，然後選取 [內嵌儀表板]。 您現在可以檢視 Web 應用程式中內嵌的相同儀表板。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已建立簡單的串流分析作業、分析傳入的資料，並且在 Power BI 儀表板中呈現結果。 若要深入了解串流分析作業，請繼續下一個教學課程：

> [!div class="nextstepaction"]
> [在 Azure 串流分析作業中執行 Azure Functions](stream-analytics-with-azure-functions.md)
