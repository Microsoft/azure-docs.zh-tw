---
title: 使用 Azure Cosmos DB 變更摘要以視覺方式呈現即時資料分析 | Microsoft Docs
description: 本文說明零售公司如何使用變更摘要了解使用者模式、執行即時資料分析和視覺效果。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: java
ms.topic: conceptual
ms.date: 08/12/2018
ms.author: sngun
ms.openlocfilehash: d2c4c890e1a1599e68fba1a0728061ec244f382f
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42145742"
---
# <a name="use-azure-cosmos-db-change-feed-to-visualize-real-time-data-analytics"></a>使用 Azure Cosmos DB 變更摘要以視覺方式呈現即時資料分析

Azure Cosmos DB 變更摘要是可在記錄建立或修改的同時，從 Azure Cosmos DB 容器中取得這些記錄的連續和累加摘要的機制。 變更摘要可藉由接聽容器是否有任何變更來支援工作。 然後變更摘要會輸出已排序的文件清單，這些文件已依其修改的順序變更過。 若要深入了解變更摘要，請參閱[使用變更摘要](change-feed.md)一文。 

本文說明電子商務公司如何使用變更摘要了解使用者模式、執行即時資料分析和視覺效果。 您將會分析事件，例如，使用者檢視某個項目、將某項目新增至其購物車，或購買某項目等。 當上述其中一個事件發生時，即會建立新的記錄，且變更摘要會記錄該筆記錄。 接著，變更摘要會觸發程序一系列的步驟，而產生計量的視覺效果，用以分析公司的效能和活動。 可以視覺方式呈現的範例計量包括營收、非重複的網站訪客、最受歡迎的項目，以及在已檢視、已新增至購物車與已購買的項目之間所做的平均價格比較。 這些範例計量可協助電子商務公司評估其網站熱門程度、擬定廣告和定價策略，以及制定應投資於哪些商品的相關決策。

在開始之前若想要觀看關於此解決方案的影片，請觀看下列影片：

> [!VIDEO https://www.youtube.com/embed/AYOiMkvxlzo]
>

## <a name="solution-components"></a>方案元件
下圖顯示此解決方案中的相關資料流程和元件：

![投影片](./media/changefeed-ecommerce-solution/project-visual.png)
 
1. **資料產生：** 資料模擬器可用來產生代表事件的零售資料，例如，使用者檢視某個項目、將某項目新增至其購物車，及購買某項目等。 您可以使用資料產生器來產生大量的範例資料。 產生的範例資料包含下列格式的文件：
   
   ```json
   {      
     "CartID": 2486,
     "Action": "Viewed",
     "Item": "Women's Denim Jacket",
     "Price": 31.99
   }
   ```

2. **Cosmos DB：** 產生的資料會儲存在 Azure Cosmos DB 集合中。  

3. **變更摘要：** 變更摘要會接聽 Azure Cosmos DB 集合的變更。 每當有新的文件新增至集合時 (也就是發生使用者檢視某個項目、將某項目新增至其購物車或購買某項目之類的事件時)，變更摘要就會觸發 [Azure 函式](../azure-functions/functions-overview.md)。  

4. **Azure 函式：** Azure 函式會處理新的資料，並將其傳送至 [Azure 事件中樞](../event-hubs/event-hubs-about.md)。  

5. **事件中樞：** Azure 事件中樞會儲存這些事件，並將其傳送至 [Azure 串流分析](../stream-analytics/stream-analytics-introduction.md)以執行進一步的分析。  

6. **Azure 串流分析：** Azure 串流分析會定義用來處理事件以及執行即時資料分析的查詢。 此資料接著會傳送至 [Microsoft Power BI](https://docs.microsoft.com/power-bi/desktop-what-is-desktop)。  

7. **Power BI：** Power BI 可以視覺方式呈現 Azure 串流分析所傳送的資料。 您可以建置儀表板，以即時查看計量的變化。  

## <a name="prerequisites"></a>必要條件

* Microsoft .NET Framework 4.7.1 或更新版本

* Microsoft .NET Core 2.1 (或更新版本)

* 具有通用 Windows 平台開發、.NET 桌面開發以及 ASP.NET 與 Web 開發工作負載的 Visual Studio

* Microsoft Azure 訂用帳戶

* Microsoft Power BI 帳戶

* 從 GitHub 下載 [Azure Cosmos DB 變更摘要實驗室](https://github.com/Azure-Samples/azure-cosmos-db-change-feed-dotnet-retail-sample)。 

## <a name="create-azure-resources"></a>建立 Azure 資源 

建立 Azure 資源 - 解決方案所需的 Azure Cosmos DB、儲存體帳戶、事件中樞、串流分析。 您將透過 Azure Resource Manager 範本來部署這些資源。 請使用下列步驟部署這些資源： 

1. 將 Windows PowerShell 執行原則設定為 [不受限]。 若要這麼做，請**以系統管理員身分開啟 Windows PowerShell**，並執行下列命令：

   ```powershell
   Get-ExecutionPolicy
   Set-ExecutionPolicy Unrestricted 
   ```

2. 從您在上一個步驟中下載的 GitHub 存放庫，瀏覽至 **Azure Resource Manager** 資料夾，然後開啟名為 **parameters.json** 的檔案。  

3. 提供 cosmosdbaccount_name、eventhubnamespace_name、storageaccount_name 等參數的值，如 **parameters.json** 檔案所指定。 您稍後將需使用您為每個資源指定的名稱。  

4. 從 **Windows PowerShell** 瀏覽至 **Azure Resource Manager** 資料夾，並執行下列命令：

   ```powershell
   .\deploy.ps1
   ```
5. 出現提示時，請輸入您的 Azure **訂用帳戶識別碼** **changefeedlab** 作為資源群組名稱，並輸入 **run1** 作為部署名稱。 資源開始部署後，可能需要約 10 分鐘才會完成。

## <a name="create-a-database-and-the-collection"></a>建立資料庫和集合

現在您將建立用來保存電子商務網站事件的集合。 當使用者檢視某個項目、將某項目新增至其購物車或購買某項目時，集合就會收到記錄，其中包含動作 (「已檢視」、「已新增」或「已購買」)、相關項目的名稱、相關項目的價格，以及相關使用者購物車的識別碼。

1. 移至 [Azure 入口網站](http://portal.azure.com/)，並找出範本部署所建立的 **Azure Cosmos DB 帳戶**。  

2. 在 [資料總管] 窗格中選取 [新增集合]，然後在表單中填寫下列詳細資料：  

   * 針對 [資料庫識別碼] 欄位，選取 [新建]，然後輸入 **changefeedlabdatabase**。 將 [佈建資料庫輸送量] 方塊保留為未核取。  
   * 針對 [集合識別碼] 欄位，輸入 **changefeedlabcollection**。  
   * 針對 [儲存體容量]，選取 [不受限]。  
   * 針對 [資料分割索引鍵] 欄位，輸入 **/Item**。 此欄位會區分大小寫，請務必正確輸入。  
   * 針對 [輸送量] 欄位，輸入 **10000**。  
   * 按一下 [確定] 按鈕。  

3. 接著，建立名為 **leases** 的另一個集合，以進行變更摘要處理。 租用集合會協調如何處理多個背景工作間的變更摘要。 另外會有一個集合用來儲存租用 (每個分割區一個租用)。  

4.  回到 [資料總管] 窗格並選取 [新增集合]，然後在表單中填寫下列詳細資料：

   * 針對 [資料庫識別碼] 欄位，選取 [使用現有的]，然後輸入 **changefeedlabdatabase**。  
   * 針對 [集合識別碼] 欄位，輸入 **leases**。  
   * 針對 [儲存體容量]，選取 [固定]。  
   * 將 [輸送量] 欄位保留為預設值。  
   * 按一下 [確定] 按鈕。

## <a name="get-the-connection-string-and-keys"></a>取得連接字串和金鑰

### <a name="get-the-azure-cosmos-db-connection-string"></a>取得 Azure Cosmos DB 連接字串

1. 移至 [Azure 入口網站](http://portal.azure.com/)，並找出範本部署所建立的 **Azure Cosmos DB 帳戶**。  

2. 瀏覽至 [金鑰] 窗格，然後將 PRIMARY CONNECTION STRING 複製到 [記事本] 或您在整個實驗室中都可存取的其他文件。 您應將其標示為 [Cosmos DB 連接字串]。 您稍後必須將此字串複製到您的程式碼中，因此請將其記下，並記住其儲存位置。

### <a name="get-the-storage-account-key-and-connection-string"></a>取得儲存體帳戶金鑰和連接字串

Azure 儲存體帳戶可讓使用者儲存資料。 在此實驗室中，您將使用儲存體帳戶來儲存 Azure 函式所使用的資料。 對集合進行任何修改時，就會觸發 Azure 函式。

1. 返回您的資源群組，並開啟您先前建立的儲存體帳戶  

2. 從左側的功能表中選取 [存取金鑰]。  

3. 將**金鑰 1** 下方的值複製到 [記事本]，或是您在整個實驗室中都可存取的其他文件。 您應將**金鑰**標示為 [儲存體金鑰]，並將**連接字串**標示為 [儲存體連接字串]。 您稍後必須將這些字串複製到您的程式碼中，因此請將其記下，並記住其儲存位置。  

### <a name="get-the-event-hub-namespace-connection-string"></a>取得事件中樞命名空間的連接字串

Azure 事件中樞會接收事件資料，並加以儲存、處理然後轉送。 在此實驗室中，每當有新的事件發生時 (也就是使用者檢視某個項目、將某項目新增至其購物車，或購買某項目時)，Azure 事件中樞就會接收文件，然後將其轉送至 Azure 串流分析。

1. 返回您的資源群組，並開啟您先前在預先實驗室中建立並命名的 [事件中樞命名空間]。  

2. 從左側的功能表中選取 [共用存取原則]。  

3. 選取 [RootManageSharedAccessKey]。 將 [[連接字串 – 主要金鑰]] 複製到 [記事本]，或是您在整個實驗室中都可存取的其他文件。 您應將其標示為 [事件中樞命名空間] 連接字串。 您稍後必須將此字串複製到您的程式碼中，因此請將其記下，並記住其儲存位置。

## <a name="set-up-azure-function-to-read-the-change-feed"></a>設定用來讀取變更摘要的 Azure 函式

在建立新的文件或修改 Cosmos DB 集合中的現行文件時，變更摘要就會自動將修改過的文件新增至其集合變更歷程記錄。 現在，您將建置並執行可處理變更摘要的 Azure 函式。 在您已建立的集合中建立或修改文件時，變更摘要將會觸發 Azure 函式。 然後，Azure 函式會將修改過的文件傳送至事件中樞。

1. 返回您複製到裝置上的存放庫。  

2. 以滑鼠右鍵按一下名為 **ChangeFeedLabSolution.sln** 的檔案，然後選取 [使用 Visual Studio 開啟]。  

3. 在 Visual Studio 中瀏覽至 **local.settings.json**。 接著，將您先前記錄的值填入空白處。  

4. 瀏覽至 **ChangeFeedProcessor.cs**。 在 **Run** 函式的參數中，執行下列動作：  

   * 將 **YOUR COLLECTION NAME HERE** 等文字取代為您的集合名稱。 如果您先前按照指示操作，則集合名稱會是 changefeedlabcollection。  
   * 將 **YOUR LEASES COLLECTION NAME HERE** 等文字取代為您的租用集合名稱。 如果您先前按照指示操作，則租用集合名稱會是 **leases**。  
   * 在 Visual Studio 頂端，確認綠色箭號左側的 [啟始專案] 方塊顯示為 **ChangeFeedFunction**。  
   * 在頁面頂端選取 [啟動] 以執行程式  
   * 當主控台應用程式顯示「作業主機已啟動」時，您即可確認函式正在執行中。

## <a name="insert-data-into-azure-cosmos-db"></a>將資料插入 Azure Cosmos DB 中 

若要查看變更摘要如何處理電子商務網站上的新動作，必須要模擬資料，以呈現使用者檢視產品目錄中的項目、將這些項目新增至其購物車，以及購買其購物車中所含項目的情境。 這項資料可以是任意資料，旨在臨摹電子商務網站上呈現的資料。

1. 在 [檔案總管] 中瀏覽回存放庫，並以滑鼠右鍵按一下 **ChangeFeedFunction.sln**，在新的 Visual Studio 視窗中重新加以開啟。  

2. 瀏覽至 **App.config** 檔案。在<appSettings>區塊內，新增您先前為 Azure Cosmos DB 帳戶擷取的 URI 和唯一 [主要金鑰]。  

3. 新增**集合**和**資料庫**名稱。 (這些名稱應該是 **changefeedlabcollection** 和 **changefeedlabdatabase**，除非您選擇以不同方式命名。)

   ![更新連接字串](./media/changefeed-ecommerce-solution/update-connection-string.png)
 
4. 儲存所有已編輯的檔案所做的變更。  

5. 在 Visual Studio 頂端，確認綠色箭號左側的 [啟始專案] 方塊顯示為 **DataGenerator**。 然後，在頁面頂端選取 [啟動] 以執行程式。  
 
6. 等候程式執行。 出現星號時，表示資料即將產生！ 請讓程式持續執行 - 收集大量資料是很重要的。  

7. 如果您依序瀏覽至 [Azure 入口網站](http://portal.azure.com/)、資源群組中的 Cosmos DB 帳戶和 [資料總管]，您會看到隨機產生的資料已匯入您的 **changefeedlabcollection** 中。
 
   ![在入口網站中產生的資料](./media/changefeed-ecommerce-solution/data-generated-in-portal.png)

## <a name="set-up-a-stream-analytics-job"></a>設定串流分析作業

Azure 串流分析是一項完全受控、可即時處理串流資料的雲端服務。 在此實驗室中，您將使用串流分析來處理來自事件中樞的新事件 (也就是在檢視某個項目、將其新增至購物車或購買時產生的事件)、將這些事件併入即時資料分析中，並將其傳送至 Power BI 以視覺方式呈現。

1. 在 [Azure 入口網站](http://portal.azure.com/)中瀏覽至您的資源群組，然後瀏覽至 **streamjob1** (您在預先實驗室中建立的串流分析作業)。  

2. 選取 [輸入]，如下所示。  

   ![建立輸入](./media/changefeed-ecommerce-solution/create-input.png)

3. 選取 [+ 新增串流輸入]。 然後，從下拉式功能表中選取 [事件中樞]。  

4. 在新的輸入表單中填寫下列詳細資料：

   * 在 [輸入別名] 欄位中，輸入 **input**。  
   * 選取 [從訂用帳戶選取事件中樞] 的選項。  
   * 將 [訂用帳戶] 欄位設定為您的訂用帳戶。  
   * 在 [事件中樞命名空間] 欄位中，輸入您在預先實驗室期間建立的事件中樞命名空間名稱。  
   * 在 [事件中樞名稱] 欄位中，選取 [使用現有的] 的選項，然後從下拉式功能表中選擇 **event-hub1**。  
   * 將 [事件中樞原則名稱] 欄位保留為預設值。  
   * 將 [事件序列化格式] 保留為 [JSON]。  
   * 將 [編碼] 欄位保留為 [UTF-8]。  
   * 將 [事件壓縮類型] 欄位保留為 [無]。  
   * 按一下 [儲存]  按鈕。

5. 瀏覽回串流分析作業頁面，並選取 [輸出]。  

6. 選取 [+ 新增] 。 然後，從下拉式功能表中選取 [Power BI]。  

7. 若要建立新的 Power BI 輸出並以視覺方式呈現平均價格，請執行下列動作：

   * 在 [輸出別名] 欄位中，輸入 **averagePriceOutput**。  
   * 將 [群組工作區] 欄位保留為 [授權連線以載入工作區]。  
   * 在 [資料集名稱] 欄位中，輸入 **averagePrice**。  
   * 在 [資料表名稱] 欄位中，輸入 **averagePrice**。  
   * 按一下 [授權] 按鈕，然後依照指示授權對 Power BI 的連線。  
   * 按一下 [儲存]  按鈕。  

8. 接著，返回 **streamjob1** 並按一下 [編輯查詢]。

   ![Edit query](./media/changefeed-ecommerce-solution/edit-query.png)
 
9. 將下列查詢貼到查詢視窗中。 **AVERAGE PRICE** 查詢會對使用者所檢視的所有項目計算平均價格、對新增至使用者購物車的所有項目計算平均價格，以及對使用者所購買的所有項目計算平均價格。 此計量可協助電子商務公司決定商品的售價，以及應投資於哪些商品。 例如，如果檢視項目的平均價格遠高於購買項目的平均價格，公司即可選擇將售價較低的項目新增至商品目錄中。

   ```sql
   /*AVERAGE PRICE*/      
   SELECT System.TimeStamp AS Time, Action, AVG(Price)  
    INTO averagePriceOutput  
    FROM input  
    GROUP BY Action, TumblingWindow(second,5) 
   ```
10. 然後，按一下位於左上角的 [儲存]。  

11. 現在請返回 **streamjob1**，然後按一下頁面頂端的 [啟動] 按鈕。 Azure 串流分析可能需要幾分鐘的時間才會啟動，但啟動後您就會看到它從「啟動中」變更為「執行中」。

## <a name="connect-to-power-bi"></a>連線至 Power BI

Power BI 是一套商務分析工具，用來分析資料及分享見解。 它充分詮釋了您可如何策略性地以視覺方式呈現分析的資料。

1. 登入 Power BI，並開啟頁面左側的功能表以瀏覽至 [我的工作區]。  

2. 選取右上角的 [+ 建立]，然後選取 [儀表板] 以建立儀表板。  

3. 選取位於右上角的 [+ 新增圖格]。  

4. 選取 [自訂串流資料]，然後按 [下一步] 按鈕。  
 
5. 從 [您的資料集] 中選取 **averagePrice**，然後按 [下一步]。  

6. 在 [視覺效果類型] 欄位中，從下拉式功能表中選擇 [群組橫條圖]。 在 [軸] 下方新增動作。 略過 [圖例] 而不新增任何項目。 然後，在名為 [值] 的下一個區段下方，新增 [平均值]。選取 [下一步]，然後為圖表加上標題，並選取 [套用]。 您應該會在儀表板上看到新的圖表！  

7. 現在，如果您想要以視覺方式呈現更多計量，您可以返回 **streamjob1**，並以下列欄位再建立三個輸出。

   a. **輸出別名：** incomingRevenueOutput、資料集名稱：incomingRevenue、資料表名稱：incomingRevenue  
   b. **輸出別名：** top5Output、資料集名稱：top5、資料表名稱：top5  
   c. **輸出別名：** uniqueVisitorCountOutput、資料集名稱：uniqueVisitorCount、資料表名稱：uniqueVisitorCount

   然後，按一下 [編輯查詢]，並在您已撰寫的查詢**上方**貼上下列查詢。

   ```sql
    /*TOP 5*/
    WITH Counter AS
    (
    SELECT Item, Price, Action, COUNT(*) AS countEvents
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY Item, Price, Action, TumblingWindow(second,30)
    ), 
    top5 AS
    (
    SELECT DISTINCT
    CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
    FROM Counter
    GROUP BY TumblingWindow(second,30)
    ), 
    arrayselect AS 
    (
    SELECT arrayElement.ArrayValue
    FROM top5
    CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
    ) 
    SELECT arrayvalue.value.item, arrayvalue.value.price,   arrayvalue.value.countEvents
    INTO top5Output
    FROM arrayselect

    /*REVENUE*/
    SELECT System.TimeStamp AS Time, SUM(Price)
    INTO incomingRevenueOutput
    FROM input
    WHERE Action = 'Purchased'
    GROUP BY TumblingWindow(hour, 1)

    /*UNIQUE VISITORS*/
    SELECT System.TimeStamp AS Time, COUNT(DISTINCT CartID) as uniqueVisitors
    INTO uniqueVisitorCountOutput
    FROM input
    GROUP BY TumblingWindow(second, 5)
   ```
   
   TOP 5 查詢會計算前 5 大項目，依購買次數排名。 此計量可協助電子商務公司評估哪些項目最受歡迎，並且可能影響公司的廣告、定價和商品陳列決策。

   REVENUE 查詢會對每分鐘購買的所有項目進行價格加總，以計算營收。 此計量可協助電子商務公司評估其財務績效，以及了解大部分的營收來自於一天中的哪些時段。 這可能會影響整體的公司策略，尤其是行銷。

   UNIQUE VISITORS 查詢會偵測非重複的購物車識別碼，而每 5 秒計算一次網站上的非重複訪客人數。 此計量可協助電子商務公司評估其網站活動，並擬定吸引更多客戶的策略。

8. 現在，您可以為這些資料集新增圖格。

   * 針對「前 5 大」，按理您可以建立以項目為軸、以計數為值的群組直條圖。  
   * 針對「營收」，按理您可以建立以時間為軸、以價格總和為值的折線圖。 顯示的時間範圍應愈大愈好，以盡可能提供最豐富的資訊。  
   * 針對「非重複訪客」，按理您可以建立以非重複訪客數為值的卡片視覺效果。

   範例儀表板使用了這些圖表後，呈現如下：

   ![視覺效果](./media/changefeed-ecommerce-solution/visualizations.png)

## <a name="optional-visualize-with-an-e-commerce-site"></a>選擇性：電子商務網站的視覺化

現在，您將觀察如何使用新的資料分析工具與實際的電子商務網站連線。 若要建置電子商務網站，請使用 Azure Cosmos DB 資料庫來儲存產品目錄清單 (女用、男用、男女通用)、產品目錄，以及最受歡迎的項目清單。

1. 瀏覽回 [Azure 入口網站](http://portal.azure.com/)，然後依序瀏覽至您的 **Cosmos DB 帳戶**和**資料總管**。  

   以固定的儲存體容量，在 **changefeedlabdatabase** 下新增兩個集合 - **產品**和**目錄**。

   以**不受限的**儲存體容量，在名為 **topItems** 的 **changefeedlabdatabase** 下新增另一個集合。 寫入 **/Item** 作為資料分割索引鍵。

2. 按一下 **topItems** 集合，然後在 [調整與設定] 下將 [存留時間] 設定為 **30 秒**，讓 topItems 每 30 秒更新一次。

   ![存留時間](./media/changefeed-ecommerce-solution/time-to-live.png)

3. 若要在 **topItems** 集合中填入購買率最高的項目，請瀏覽回 **streamjob1**，並新增一個 [輸出]。 選取 [Cosmos DB]。

4. 填寫必要欄位，如下圖所示。

   ![Cosmos 輸出](./media/changefeed-ecommerce-solution/cosmos-output.png)
 
5. 如果您在先前的實驗室步驟中新增了選擇性的 TOP 5 查詢，請繼續進行步驟 5a。 否則，請繼續進行步驟 5b。

   5a. 在 **streamjob1** 中選取 [編輯查詢]，然後在 Azure 串流分析查詢編輯器中，將下列查詢貼到 TOP 5 查詢下方、其餘查詢上方的位置。

   ```sql
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```
   5b. 在 **streamjob1** 中選取 [編輯查詢]，然後在 Azure 串流分析查詢編輯器中，將下列查詢貼到所有其他查詢上方。

   ```sql
   /*TOP 5*/
   WITH Counter AS
   (
   SELECT Item, Price, Action, COUNT(*) AS countEvents
   FROM input
   WHERE Action = 'Purchased'
   GROUP BY Item, Price, Action, TumblingWindow(second,30)
   ), 
   top5 AS
   (
   SELECT DISTINCT
   CollectTop(5)  OVER (ORDER BY countEvents) AS topEvent
   FROM Counter
   GROUP BY TumblingWindow(second,30)
   ), 
   arrayselect AS 
   (
   SELECT arrayElement.ArrayValue
   FROM top5
   CROSS APPLY GetArrayElements(top5.topevent) AS arrayElement
   ) 
   SELECT arrayvalue.value.item AS Item, arrayvalue.value.price, arrayvalue.value.countEvents
   INTO topItems
   FROM arrayselect
   ```

6. 在**方案總管**中開啟 **EcommerceWebApp.sln**，並瀏覽至 **Web.config** 檔案。  

7. 在 `<appSettings>` 區塊內，將您先前儲存的 **URI** 和**主要金鑰**分別新增到顯示為 **your URI here** 和 **your primary key here** 之處。 然後，依指示新增您的**資料庫名稱**和**集合名稱**。 (這些名稱應該是 **changefeedlabdatabase** 和 **changefeedlabcollection**，除非您選擇以不同方式命名。)

   依指示填入您的**產品集合名稱**、**目錄集合名稱**和**最高排名項目集合名稱**。 (這些名稱應該是 **products、categories 和 topItems**，除非您選擇以不同方式命名。)  

8. 瀏覽至 **EcommerceWebApp.sln** 內的**簽出資料夾**並加以開啟。 然後，開啟該資料夾中的 **Web.config** 檔案。  

9. 在 `<appSettings>` 區塊內，將您先前儲存的 **URI** 和**主要金鑰**新增至指示之處。 然後，依指示新增您的**資料庫名稱**和**集合名稱**。 (這些名稱應該是 **changefeedlabdatabase** 和 **changefeedlabcollection**，除非您選擇以不同方式命名。)  

10. 在頁面頂端按下 [啟動] 以執行程式。  

11. 現在，您可以在電子商務網站上隨意逛逛。 當您檢視某個項目、將某項目新增至購物車、變更購物車內的項目數量，或購買某項目時，這些事件將會經由 Cosmos DB 變更摘要傳至事件中樞、ASA，最後傳到 Power BI。 建議您繼續執行 DataGenerator 以產生大量的 Web 流量資料，並在電子商務網站上提供真正的「熱門產品」集。

## <a name="delete-the-resources"></a>刪除資源

若要刪除您在本實驗室中建立的資源，請在 [Azure 入口網站](http://portal.azure.com/)中瀏覽至資源群組，然後從頁面頂端的功能表中選取 [刪除資源群組]，並依照隨附的指示操作。

## <a name="next-steps"></a>後續步驟 
  
* 若要深入了解變更摘要，請參閱[使用 Azure Cosmos DB 中的變更摘要支援](change-feed.md) 
* 採用 Azure Cosmos DB 的醫療保健組織所適用的[變更摘要通知解決方案](change-feed-hl7-fhir-logic-apps.md)。
