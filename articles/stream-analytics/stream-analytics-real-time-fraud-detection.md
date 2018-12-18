---
title: 使用 Azure 串流分析進行即時詐騙偵測
description: 了解如何使用串流分析建立即時詐騙偵測解決方案。 使用事件中樞以即時處理事件。
services: stream-analytics
author: mamccrea
ms.author: mamccrea
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/28/2017
ms.openlocfilehash: e0d430ced1dbddbfca79806591c83c33e732eefd
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37901709"
---
# <a name="get-started-using-azure-stream-analytics-real-time-fraud-detection"></a>開始使用 Azure 串流分析：即時詐騙偵測

本教學課程提供如何使用 Azure 串流分析的端對端實例。 您會了解如何： 

* 將串流事件帶入 Azure 事件中樞的執行個體中。 在本教學課程中，您將使用應用程式來模擬行動電話中繼資料記錄流。

* 撰寫類似 SQL 的串流分析查詢來轉換資料、彙總資訊或尋找模式。 您將了解如何使用查詢來檢查傳入資料流，並尋找可能是詐騙的電話。

* 將結果傳送至輸出接收 (儲存體)，讓您分析來深入探索。 在此案例中，您會將可疑的通話資料傳送至 Azure Blob 儲存體。

本教學課程根據通話資料來舉例說明即時詐欺偵測。 我們說明的技巧也適用於其他類型的詐騙偵測，例如信用卡詐騙或身分盜用。 

## <a name="scenario-telecommunications-and-sim-fraud-detection-in-real-time"></a>案例：即時偵測電信與 SIM 詐騙

電信公司有大量的資料都是來電。 該公司想要即時偵測詐騙電話，以便通知客戶或針對特定號碼停止服務。 有一種 SIM 詐騙牽涉到同一身分識別大約在同一時間從不同地理位置撥來很多通電話。 為了偵測這種詐騙，該公司需要檢查來電記錄並尋找特定模式。在此案例中，是檢查大約在同一時間從不同國家/地區的來電。 任何歸類到此類別的電話記錄會寫入儲存體，以進一步分析。

## <a name="prerequisites"></a>先決條件

在本教學課程中，您會透過用戶端應用程式來產生範例通話中繼資料，以模擬通話資料。 該應用程式會產生一些很像是詐騙電話的記錄。 

開始之前，請確定您具有下列項目：

* 一個 Azure 帳戶。
* 通話事件產生器應用程式 [TelcoGenerator.zip](http://download.microsoft.com/download/8/B/D/8BD50991-8D54-4F59-AB83-3354B69C8A7E/TelcoGenerator.zip) (可至 Microsoft 下載中心下載)。 將此套件解壓縮至電腦上的資料夾。 如果您想要看到原始程式碼，並在偵錯工具中執行應用程式，您可以從 [GitHub](https://aka.ms/azure-stream-analytics-telcogenerator) 取得應用程式原始程式碼。 

    >[!NOTE]
    >Windows 可能會封鎖下載的 .zip 檔案。 請無法解壓縮，請以滑鼠右鍵按一下檔案，然後選取 [內容]。 如果看到「這個檔案來自另一部電腦，可能會封鎖以協助保護您的電腦」訊息，請選取 [解除封鎖] 選項，並按一下 [套用]。

如果想要檢查串流分析作業的結果，您也需要工具來檢視 Azure Blob 儲存體容器的內容。 如果您使用 Visual Studio，您可以使用 [Azure Tools for Visual Studio](https://docs.microsoft.com/azure/vs-azure-tools-storage-resources-server-explorer-browse-manage) 或 [Visual Studio Cloud Explorer](https://docs.microsoft.com/azure/vs-azure-tools-resources-managing-with-cloud-explorer)。 或者，您可以安裝獨立工具，例如 [Azure 儲存體總管](http://storageexplorer.com/)或 [Azure 總管](http://www.cerebrata.com/products/azure-explorer/introduction)。 

## <a name="create-an-azure-event-hubs-to-ingest-events"></a>建立 Azure 事件中樞來內嵌事件

若要分析資料流，您需要將資料流「內嵌」到 Azure。 內嵌資料的一般做法是使用 [Azure 事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)，這可讓您每秒內嵌數百萬個事件，然後處理並儲存事件資訊。 在本教學課程中，您將建立事件中樞，然後由通話事件產生器應用程式將通話資料傳送至該事件中樞。 如需深入了解事件中樞，請參閱 [Azure 服務匯流排文件](https://docs.microsoft.com/azure/service-bus/)。

>[!NOTE]
>如需此程序的詳細資訊，請參閱[使用 Azure 入口網站建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。 

### <a name="create-a-namespace-and-event-hub"></a>建立命名空間和事件中樞
在此程序中，您需要先建立事件中樞命名空間，再將事件中樞新增至該命名空間。 事件中樞命名空間可在邏輯上將相關的事件匯流排執行個體分組。 

1. 登入 Azure 入口網站，按一下 [建立資源] > [物聯網] > [事件中樞]。 

2. 在 [建立命名空間] 窗格中，輸入命名空間名稱，例如 `<yourname>-eh-ns-demo`。 您可以使用任何名稱作為命名空間，但名稱在 URL 中必須有效，而且在整個 Azure 內必須唯一的。 
    
3. 選取訂用帳戶並建立或選擇資源群組，然後按一下 [建立]。

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-namespace-new-portal.png" alt="drawing" width="300px"/>

4. 當命名空間完成部署時，請在 Azure 資源清單中尋找事件中樞命名空間。 

5. 按一下新的命名空間，然後在命名空間窗格中，按一下 [事件中樞]。

   ![建立新事件中樞的 [新增事件中樞] 按鈕 ](./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-button-new-portal.png)    
 
6. 將新的事件中樞命名為 `asa-eh-frauddetection-demo`。 您可以使用不同的名稱。 如果這樣做，請記下來，因為稍後需要用到此名稱。 您目前不需要為事件中樞設定其他任何選項。

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-eventhub-new-portal.png" alt="drawing" width="400px"/>
    
 
7. 按一下頁面底部的 [新增] 。

### <a name="grant-access-to-the-event-hub-and-get-a-connection-string"></a>授權存取事件中樞並取得連接字串

事件中樞必須有原則來允許適當的存取權，流程才能將資料傳送到事件中樞。 存取原則會產生包含授權資訊的連接字串。

1.  在事件命名空間窗格中，按一下 [事件中樞]，然後按一下新事件中樞的名稱。

2.  在事件中樞窗格中，按一下 [共用存取原則]，然後按一下 [新增]**+&nbsp;**。

    >[!NOTE]
    >請確定您正在使用事件中樞，而不是事件中樞命名空間。

3.  新增名為 `sa-policy-manage-demo` 的原則，然後在 [宣告] 中，選取 [管理]。

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-shared-access-policy-manage-new-portal.png" alt="drawing" width="300px"/>
 
4.  按一下頁面底部的 [新增] 。

5.  部署原則之後，在共用存取原則清單中按一下此原則。

6.  找出標示為 [連接字串-主要索引鍵] 的方塊，按一下連接字串旁邊的複製按鈕。 

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-shared-access-policy-copy-connection-string-new-portal.png" alt="drawing" width="300px"/>
 
7.  將連接字串貼到文字編輯器。 稍微編輯一下此連接字串，下一節需要用到它。

    連接字串如下所示：

        Endpoint=sb://YOURNAME-eh-ns-demo.servicebus.windows.net/;SharedAccessKeyName=asa-policy-manage-demo;SharedAccessKey=Gw2NFZwU1Di+rxA2T+6hJYAtFExKRXaC2oSQa0ZsPkI=;EntityPath=asa-eh-frauddetection-demo

    請注意，連接字串包含多個機碼值組，以分號分隔：`Endpoint`、`SharedAccessKeyName`、`SharedAccessKey` 和 `EntityPath`。  

## <a name="configure-and-start-the-event-generator-application"></a>設定並啟動事件產生器應用程式

啟動 TelcoGenerator 應用程式之前，您必須先進行設定，讓它將通話記錄傳送到您所建立的事件中樞。

### <a name="configure-the-telcogenerator-app"></a>設定 TelcoGenerator 應用程式

1.  在已複製連接字串的編輯器中，記下 `EntityPath` 值，然後移除 `EntityPath` 組 (別忘了移除前面的分號)。 

2.  在您解壓縮 TelcoGenerator.zip 檔案的資料夾中，利用編輯器開啟 telcodatagen.exe.config 檔案。 (有一個以上的 .config 檔案，請確定是開啟正確的檔案。)

3.  在 `<appSettings>` 元素中：

    * 將 `EventHubName` 索引鍵的值設為事件中樞名稱 (也就是實體路徑的值)。
    * 將 `Microsoft.ServiceBus.ConnectionString` 索引鍵的值設為連接字串。 

    `<appSettings>` 看起來如下範例所示。 (為了方便閱讀，會將文字斷行並移除授權權杖中的某些字元。)

   ![顯示事件中樞名稱和連接字串的 TelcoGenerator 應用程式組態檔](./media/stream-analytics-real-time-fraud-detection/stream-analytics-telcogenerator-config-file-app-settings.png)
 
4.  儲存檔案。 

### <a name="start-the-app"></a>啟動應用程式
1.  開啟命令視窗，切換至解壓縮 TelcoGenerator 應用程式的資料夾。
2.  輸入下列命令：

        telcodatagen.exe 1000 0.2 2

    參數如下： 

    * 每小時的 CDR 數目。 
    * SIM 卡詐騙機率：應用程式模擬詐騙電話的頻率 (所有通話的百分比)。 值 0.2 表示大約 20% 的通話記錄可能是詐騙。
    * 持續時間 (小時)。 應用程式應該執行的時數。 您也可以隨時在命令列按下 Ctrl+C 來停止應用程式。

    幾秒之後，隨著應用程式將通話記錄傳送到事件中樞，它會開始在螢幕上顯示通話記錄。

您在此即時詐騙偵測應用程式中會用到的一些重要欄位如下：

|**記錄**|**定義**|
|----------|--------------|
|`CallrecTime`|通話開始時間的時間戳記。 |
|`SwitchNum`|用來接通電話的電話交換機。 在此範例中，交換機是代表發話國的字串 (美國、中國、英國、德國或澳大利亞)。 |
|`CallingNum`|來電者的電話號碼。 |
|`CallingIMSI`|國際行動用戶識別碼 (IMSI)。 這是來電者的唯一識別碼。 |
|`CalledNum`|受話方的電話號碼。 |
|`CalledIMSI`|國際行動用戶識別碼 (IMSI)。 這是受話方的唯一識別碼。 |


## <a name="create-a-stream-analytics-job-to-manage-streaming-data"></a>建立串流分析作業來管理串流資料

既然已取得通話事件的資料流，您可以開始設定串流分析作業。 此作業會從您設定的事件中樞讀取資料。 

### <a name="create-the-job"></a>建立工作 

1. 在 Azure 入口網站中，按一下 [建立資源] > [物聯網] > [串流分析作業]。

2. 將作業命名為 `asa_frauddetection_job_demo`，並指定訂用帳戶、資源群組和位置。

    最好將作業和事件中樞放在相同的區域，以達到最佳效能，在區域之間傳送資料也不需要付費。

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sa-job-new-portal.png" alt="drawing" width="300px"/>

3. 按一下頁面底部的 [新增] 。

    即可建立作業，入口網站會顯示作業詳細資料。 但還沒有開始運作，您必須先設定作業，作業才能啟動。

### <a name="configure-job-input"></a>設定作業輸入

1. 在儀表板或 [所有資源] 窗格中，尋找並選取 `asa_frauddetection_job_demo` 串流分析作業。 
2. 在串流分析作業窗格的 [概觀] 區段中，按一下 [輸入] 方塊。

   ![串流分析作業窗格中位於 [拓撲] 下的輸入方塊](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-input-box-new-portal.png)
 
3. 按一下 [新增資料流輸入] 並選取 [事件中樞]。 接著，在 [新的輸入] 頁面中填寫以下資訊：

   |**設定**  |**建議的值**  |**說明**  |
   |---------|---------|---------|
   |輸入別名  |  CallStream   |  輸入名稱以識別作業的輸入。   |
   |訂用帳戶   |  \<您的訂用帳戶\> |  根據您建立的事件中樞，選取事件中樞所在的 Azure 訂用帳戶。   |
   |事件中樞命名空間  |  asa-eh-ns-demo |  輸入事件中樞命名空間的名稱。   |
   |事件中樞名稱  | asa-eh-frauddetection-demo | 選取事件中樞的名稱。   |
   |事件中樞原則名稱  | asa-policy-manage-demo | 選取您稍早建立的存取原則。   |
    </br>
    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sa-input-new-portal.png" alt="drawing" width="300px"/>


4. 按一下頁面底部的 [新增] 。

## <a name="create-queries-to-transform-real-time-data"></a>建立查詢來轉換即時資料

目前，您已設定串流分析作業來讀取傳入資料流。 下一步是建立即時分析資料的查詢。 串流分析支援用來說明轉換的簡單、宣告式查詢模型，以供即時處理。 查詢使用類似 SQL 的語言，可針對串流分析來擴充一些功能。 

簡單的查詢可能只是讀取所有傳入資料。 不過，您通常會建立查詢來尋找特定資料，或資料中的關聯性。 在這一節的教學課程中，您會建立並測試數個查詢來學習幾種方法，以轉換輸入資料流來分析。 

您在此處建立的查詢只是在螢幕上顯示轉換後的資料。 在稍後一節中，您將設定輸出接收，並設定查詢將轉換後的資料寫入該接收。

若要深入了解語言，請參閱 [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/dn834998.aspx)。

### <a name="get-sample-data-for-testing-queries"></a>取得範例資料來測試查詢

TelcoGenerator 應用程式正在將通話記錄傳送到事件中樞，而串流分析作業已設定來讀取事件中樞。 您可以使用查詢來測試作業，確定能正確讀取。 若要在 Azure 主控台測試查詢，您需要範例資料。 在這個逐步解說中，您將從傳入事件中樞的資料流擷取範例資料。

1. 請確定 TelcoGenerator 應用程式正在執行且產生通話記錄。
2. 在入口網站中，返回串流分析作業窗格  (如果已關閉此窗格，請在 [所有資源] 窗格中搜尋 `asa_frauddetection_job_demo`)。
3. 按一下 [查詢] 方塊。 Azure 會列出作業已設定的輸入和輸出，還可讓您建立查詢，在輸入資料流傳送至輸出時進行轉換。
4. 在 [查詢] 窗格中，按一下 `CallStream` 輸入旁邊的點，然後選取 [來自輸入的範例資料]。

   ![用於在串流分析作業中輸入範例資料的功能表選項，已選取 [來自輸入的範例資料]](./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-sample-data-from-input.png)


5. 將 [分鐘] 設定為 3，然後按一下 [確定]。 
    
   ![輸入資料流的取樣選項，已選取 [3 分鐘]。](./media/stream-analytics-real-time-fraud-detection/stream-analytics-input-create-sample-data.png)

    Azure 會從輸入資料流取樣 3 分鐘的資料，備妥範例資料時會通知您。 (這需要等一下。) 

範例資料會暫時儲存，開啟查詢視窗時即可使用。 如果您關閉查詢視窗，則會捨棄範例資料，您將必須建立一組新的範例資料。 

或者，您可以[從 GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/telco.json) 取得含有範例資料的 .json 檔案，然後上傳該 .json 檔案，以當作 `CallStream` 輸入的範例資料使用。 

### <a name="test-using-a-pass-through-query"></a>使用傳遞查詢來測試

如果您想要封存每一個事件，您可以使用傳遞查詢，讀取事件承載中的所有欄位。

1. 在查詢視窗中，輸入此查詢︰

        SELECT 
            *
        FROM 
            CallStream

    >[!NOTE]
    >如同 SQL，關鍵字不區分大小寫，空白字元也不重要。

    在此查詢中，`CallStream` 是您建立輸入時所指定的別名。 如果您使用不同的別名，請改為使用該名稱。

2. 按一下 [ **測試**]。

    串流分析作業會查詢範例資料，然後在視窗底部顯示輸出。 結果指出事件中樞和串流分析作業都正確設定。 (如前所述，稍後您將建立輸出接收供查詢寫入資料)。

   ![串流分析作業輸出，顯示已產生 73 筆記錄](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output.png)

    您確實會看到的記錄數目取決於 3 分鐘範例中所擷取的記錄數目。
 
### <a name="reduce-the-number-of-fields-using-a-column-projection"></a>使用資料行投影來減少欄位數目

在許多情況下，您的分析不需要輸入資料流的所有資料行。 您可以使用查詢來投影更小的一組傳回欄位，比傳遞查詢傳回的欄位更少。

1. 在程式碼編輯器中，將查詢變更如下：

        SELECT CallRecTime, SwitchNum, CallingIMSI, CallingNum, CalledNum 
        FROM 
            CallStream

2. 再按一次 [測試]。 

   ![投影的串流分析作業輸出，顯示已產生 25 筆記錄](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-projection.png)
 
### <a name="count-incoming-calls-by-region-tumbling-window-with-aggregation"></a>各區域的來電計數：含彙總的輪轉視窗

假設您想要計算每個區域的來電數目。 在串流資料中，當您想要執行彙總函式時，例如計數，您需要將資料流分割成時態單位 (因為資料流本身實際上沒有止境)。 做法是使用串流分析[視窗函式](stream-analytics-window-functions.md)。 然後，您就可以將該視窗內的資料當作一個單位來使用。

在這個轉換過程中，您需要有一連串不重疊的時態性視窗，每個視窗會有一組特定的資料讓您分組和彙總。 這種視窗稱為「輪轉視窗」。 在輪轉視窗中，您可以取得依 `SwitchNum` (代表發話國) 分組的來電計數。 

1. 在程式碼編輯器中，將查詢變更如下：

        SELECT 
            System.Timestamp as WindowEnd, SwitchNum, COUNT(*) as CallCount 
        FROM
            CallStream TIMESTAMP BY CallRecTime 
        GROUP BY TUMBLINGWINDOW(s, 5), SwitchNum

    此查詢在 `FROM` 子句中使用 `Timestamp By` 關鍵字，以指定使用輸入資料流中的哪個時間戳記欄位來定義輪轉視窗。 在此案例中，視窗會將每一筆記錄的資料依 `CallRecTime` 欄位分段。 (如果未指定欄位，則時間範圍作業會使用每個事件抵達事件中樞的時間。 請參閱[串流分析查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)中的＜抵達時間與應用時間的比較＞。 

    投影包含 `System.Timestamp`，它會傳回每個視窗結尾的時間戳記。 

    若要指定使用輪轉視窗，請在 `GROUP BY ` 子句中使用 [TUMBLINGWINDOW](https://msdn.microsoft.com/library/dn835055.aspx) 函式。 在此函式中，您可以指定時間單位 (從一微秒到一天即可) 和視窗大小 (單位數量)。 在此範例中，輪轉視窗由 5 秒的間隔組成，所以會依國家/地區產生每 5 秒的來電計數。

2. 再按一次 [測試]。 在結果中，可看見 **WindowEnd** 底下的時間戳記是以 5 秒為增量單位。

   ![彙總的串流分析作業輸出，顯示已產生 13 筆記錄](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-aggregation.png)
 
### <a name="detect-sim-fraud-using-a-self-join"></a>使用自我聯結來偵測 SIM 詐騙

在此範例中，請將詐騙手法想像成同一位使用者在 5 秒內從不同位置撥出多通電話。 例如，按照常理，同一位使用者不可能同時從美國和澳大利亞打電話。 

若要檢查這些情況，您可以使用自我聯結的串流資料，根據 `CallRecTime` 值將資料流聯結到本身。 然後，您可以尋找 `CallingIMSI` 值 (發話號碼) 相同但 `SwitchNum` 值 (發話國) 不同的通話記錄。

當您在串流資料中使用聯結時，聯結必須稍微限制相符的資料列在時間上可以相隔多久。 (如稍早所述，串流資料實際上是沒有止境。)在聯結的 `ON` 子句內，可使用 `DATEDIFF` 函式來指定關聯性的時間界限。 在此案例中，聯結是以 5 秒間隔的通話資料為基礎。

1. 在程式碼編輯器中，將查詢變更如下： 

        SELECT  System.Timestamp as Time, 
            CS1.CallingIMSI, 
            CS1.CallingNum as CallingNum1, 
            CS2.CallingNum as CallingNum2, 
            CS1.SwitchNum as Switch1, 
            CS2.SwitchNum as Switch2 
        FROM CallStream CS1 TIMESTAMP BY CallRecTime 
            JOIN CallStream CS2 TIMESTAMP BY CallRecTime 
            ON CS1.CallingIMSI = CS2.CallingIMSI 
            AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5 
        WHERE CS1.SwitchNum != CS2.SwitchNum

    此查詢就像任何 SQL 聯結一樣，差別在於聯結中的 `DATEDIFF` 函式。 這是串流分析專用的 `DATEDIFF` 版本，必須出現在 `ON...BETWEEN` 子句中。 參數是時間單位 (在此範例中為秒) 和兩個聯結來源的別名。 這不同於標準 SQL `DATEDIFF` 函式。

    `WHERE` 子句中的條件會標出詐騙電話：發話端交換機不相同。 

2. 再按一次 [測試]。 

   ![自我聯結的串流分析作業輸出，顯示已產生 6 筆記錄](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-sample-output-self-join.png)

3. 按一下 [儲存] 將自我聯結查詢儲存在串流分析作業中。 (不會儲存範例資料。)

    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-query-editor-save-button-new-portal.png" alt="drawing" width="300px"/>

## <a name="create-an-output-sink-to-store-transformed-data"></a>建立輸出接收以儲存轉換後的資料

您已定義事件資料流、用來內嵌事件的事件中樞，以及用來轉換資料流的查詢。 最後一個步驟是定義作業的輸出接收，也就是轉換後的資料流寫入的地方。 

您可以使用許多資源作為輸出接收，例如 SQL Server 資料庫、資料表儲存體、Data Lake 儲存體、Power BI，甚至是另一個事件中樞。 在本教學課程中，您會將資料流寫入 Azure Blob 儲存體，通常會選擇這一項來收集事件資訊供稍後分析，因為它支援非結構化資料。

如果您已有 blob 儲存體帳戶，則可直接使用。 在本教學課程中，您會學習到如何建立新的儲存體帳戶。

### <a name="create-an-azure-blob-storage-account"></a>建立 Azure Blob 儲存體帳戶

1. 從 Azure 入口網站的左上角，選取 [建立資源] > [儲存體] > [儲存體帳戶]。 填寫儲存體帳戶作業頁面，將 [名稱] 設為 "asaehstorage"，[位置] 設為「美國東部」，[資源群組] 設為 "asa-eh-ns-rg" (將儲存體帳戶裝載在與串流作業相同的資源群組中可增加效能)。 其餘設定可以保留預設值。  

   ![建立儲存體帳戶](./media/stream-analytics-real-time-fraud-detection/stream-analytics-storage-account-create.png)

2. 在 Azure 入口網站中，返回串流分析作業窗格。 (如果已關閉此窗格，請在 [所有資源] 窗格中搜尋 `asa_frauddetection_job_demo`)。

3. 在 [作業拓撲] 區段中，按一下 [輸出] 方塊。

4. 在 [輸出] 窗格中，按一下 [新增]，然後選取 [Blob 儲存體]。 接著，在 [新的輸出] 頁面中填寫以下資訊：

   |**設定**  |**建議的值**  |**說明**  |
   |---------|---------|---------|
   |輸出別名  |  CallStream-FraudulentCalls   |  輸入名稱以識別作業的輸出。   |
   |訂用帳戶   |  \<您的訂用帳戶\> |  選取您在其中建立儲存體帳戶的 Azure 訂用帳戶。 儲存體帳戶可以位在相同或不同的訂用帳戶中。 此範例假設您已在相同的訂用帳戶中建立儲存體帳戶。 |
   |儲存體帳戶  |  asaehstorage |  輸入建立之儲存體帳戶的名稱。 |
   |容器  | asa-fraudulentcalls-demo | 選擇 [建立新項目]，然後輸入容器名稱。 |
    <br/>
    <img src="./media/stream-analytics-real-time-fraud-detection/stream-analytics-create-output-blob-storage-new-console.png" alt="drawing" width="300px"/>
    
5. 按一下 [檔案] 。 


## <a name="start-the-streaming-analytics-job"></a>啟動串流分析作業

現在已設定作業。 您已指定輸入 (事件中樞)、轉換 (用來尋找詐騙電話的查詢) 和輸出 (blob 儲存體)。 您現在可以啟動作業。 

1. 請確定 TelcoGenerator 應用程式正在執行。

2. 在作業窗格中，按一下 [啟動]。 在 [啟動作業] 窗格的 [作業輸出開始時間] 中，選取 [現在]。 

   ![啟動串流分析工作](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-start.png)



## <a name="examine-the-transformed-data"></a>檢查已轉換的資料

您現在已完成串流分析作業。 此作業會檢查通話中繼資料的資料流，即時尋找詐騙電話，並將這些詐騙電話的相關資訊寫入儲存體。 

若要完成本教學課程，您可能需要查看串流分析作業所擷取的資料。 資料正分成區塊 (檔案) 寫入 Azure Blob 儲存體。 您可以使用任何可讀取 Azure Blob 儲存體的工具。 如＜必要條件＞一節所述，您可以在 Visual Studio 中使用 Azure 擴充，也可以使用 [Azure 儲存體總管](http://storageexplorer.com/)或 [Azure 總管](http://www.cerebrata.com/products/azure-explorer/introduction)之類的工具。 

當您在 blob 儲存體中檢查檔案的內容時，您會看到類似如下的內容：

   ![Azure blob 儲存體與串流分析輸出](./media/stream-analytics-real-time-fraud-detection/stream-analytics-sa-job-blob-storage-view.png)
 

## <a name="clean-up-resources"></a>清除資源

我們還有其他文章繼續探討詐騙偵測案例，而且會使用您在本教學課程中所建立的資源。 如果您想要繼續，請參閱在**後續步驟**下方的建議。

不過，如果您已完成，也不需要您已建立的資源，則可以刪除它們，以免產生不必要的 Azure 費用。 在此情況下，建議您採取下列動作：

1. 停止串流分析作業。 在 [作業] 窗格中，按一下頂端的 [停止]。
2. 停止 Telco Generator 應用程式。 在您啟動應用程式的命令視窗中，按 Ctrl+C。
3. 如果您建立的新 blob 儲存體帳戶只用於本教學課程，請刪除它。 
4. 刪除串流分析作業。
5. 刪除事件中樞。
6. 刪除事件中樞命名空間。

## <a name="get-support"></a>取得支援

如需進一步的協助，請參閱 [Azure 串流分析論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics)。

## <a name="next-steps"></a>後續步驟

您可以閱讀下列文章來繼續本教學課程：

* [串流分析及 Power BI：適用於串流資料的即時分析儀表板](stream-analytics-power-bi-dashboard.md)。 本文示範如何將串流分析作業的 TelCo 輸出傳送至 Power BI，以呈現即時視覺效果和進行分析。

如需串流分析在整體上的詳細資訊，請參閱下列文章：

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)
