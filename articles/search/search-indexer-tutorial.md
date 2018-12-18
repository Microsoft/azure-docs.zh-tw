---
title: 在 Azure 搜尋服務中編製 Azure SQL Database 索引的教學課程 | Microsoft Docs
description: 在本教學課程中，對 Azure SQL 資料庫進行編目，以擷取可搜尋的資料，並填入 Azure 搜尋服務索引。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.devlang: na
ms.topic: tutorial
ms.date: 07/10/2018
ms.author: heidist
ms.openlocfilehash: b40d3a74904d6814eb01b5d41d10632e8c9af5be
ms.sourcegitcommit: df50934d52b0b227d7d796e2522f1fd7c6393478
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38988790"
---
# <a name="tutorial-crawl-an-azure-sql-database-using-azure-search-indexers"></a>教學課程：使用 Azure 搜尋索引子對 Azure SQL 資料庫進行編目

本教學課程示範如何設定索引子，以便從範例 Azure SQL Database 擷取可搜尋的資料。 [索引子](search-indexer-overview.md)是 Azure 搜尋服務的元件，可搜耙外部資料來源，以內容填入[搜尋索引](search-what-is-an-index.md)。 在所有索引子中，Azure SQL Database 的索引子最廣泛使用。 

索引子設定的熟練度很有幫助，因為它可簡化您必須撰寫和維護的程式碼數量。 您不需準備及推送結構描述相容的 JSON 資料集，而是可以將索引子附加至資料來源，讓索引子擷取資料並將其插入索引中，並選擇性地依照週期性排程執行索引子，以掌握基礎來源中的變更。

在本教學課程中，使用 [Azure 搜尋服務 .NET 用戶端程式庫](https://aka.ms/search-sdk)和 .NET Core 主控台應用程式，執行下列工作：

> [!div class="checklist"]
> * 下載並設定解決方案
> * 將搜尋服務資訊新增至應用程式設定
> * 在 Azure SQL Database 中準備外部資料集 
> * 檢閱範例程式碼中的索引和索引子定義
> * 執行索引子程式碼以匯入資料
> * 搜尋索引
> * 在入口網站中檢視索引子組態

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>必要條件

* Azure 搜尋服務。 如需設定搜尋服務的說明，請參閱[建立搜尋服務](search-create-service-portal.md)。

* 可提供索引子所用外部資料來源的 Azure SQL Database。 範例解決方案會提供 SQL 資料檔案，以建立資料表。

* Visual Studio 2017。 您可以使用免費的 [Visual Studio 2017 Community 版本](https://www.visualstudio.com/downloads/)。 

> [!Note]
> 如果您使用免費的 Azure 搜尋服務，則受限於三個索引、三個索引子，以及三個資料來源。 本教學課程會各建立一個。 確定您的服務有空間可接受新的資源。

## <a name="download-the-solution"></a>下載解決方案

本教學課程中使用的索引子解決方案是來自一個主要下載中傳遞的 Azure 搜尋服務範例集合。 本教學課程使用的解決方案是 DotNetHowToIndexers。

1. 移至 Azure 範例 GitHub 存放庫中的 [**Azure-Samples/search-dotnet-getting-started**](https://github.com/Azure-Samples/search-dotnet-getting-started)。

2. 按一下 [複製或下載] > [下載 ZIP]。 根據預設，檔案會移至 [下載] 資料夾。

3. 在 [檔案總管] > [下載] 中，以滑鼠右鍵按一下檔案，然後選擇 [全部擷取]。

4. 關閉唯讀權限。 以滑鼠右鍵按一下資料夾名稱 > [屬性] > [一般]，然後清除目前資料夾、子資料夾和檔案的 [唯讀] 屬性。

5. 在 **Visual Studio 2017** 中，開啟解決方案 DotNetHowToIndexers.sln。

6. 在 [方案總管] 中，以滑鼠右鍵按一下頂端節點父代方案 > [還原 NuGet 套件]。

## <a name="set-up-connections"></a>設定連線
必要服務的連線資訊會指定於方案的 **appsettings.json** 檔案中。 

在 [方案總管] 中，開啟 **appsettings.json**，以便您使用本教學課程中的指示來填入每項設定。  

```json
{
  "SearchServiceName": "Put your search service name here",
  "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
  "AzureSqlConnectionString": "Put your Azure SQL database connection string here",
}
```

### <a name="get-the-search-service-name-and-admin-api-key"></a>取得搜尋服務名稱和系統管理 API 金鑰

您可以在入口網站中找到搜尋服務端點和金鑰。 金鑰可提供服務作業的存取權。 系統管理金鑰允許寫入存取，才能在您的服務中建立和刪除物件，例如索引和索引子。

1. 登入 [Azure 入口網站](https://portal.azure.com/)，並尋找[訂用帳戶的搜尋服務](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。

2. 開啟服務頁面。

3. 在頂端，尋找主頁面中的服務名稱。 在下列螢幕擷取畫面中，這是 azs-tutorial。

   ![服務名稱](./media/search-indexer-tutorial/service-name.png)

4. 複製並貼上該服務名稱，作為您在 Visual Studio 的 **appsettings.json** 中輸入的第一項資料。

  > [!Note]
  > 服務名稱屬於包含 search.windows.net 的端點。 如果您很好奇，您可以在 [概觀] 頁面上的 [基本資訊] 中查看完整 URL。 URL 看起來像這個範例：https://your-service-name.search.windows.net

5. 在左側的 [設定] > [金鑰] 中，複製其中一個系統管理金鑰並將它當作第二個項目貼到 **appsettings.json** 中。 金鑰是在佈建期間為您的服務產生的英數字元字串，而且是獲得授權存取服務作業的必要項目。 

  新增這兩個設定之後，您的檔案應類似以下範例所示：

  ```json
  {
    "SearchServiceName": "azs-tutorial",
    "SearchServiceAdminApiKey": "A1B2C3D4E5F6G7H8I9J10K11L12M13N14",
    . . .
  }
  ```

## <a name="prepare-an-external-data-source"></a>準備外部資料來源

在此步驟中，建立索引子可以搜耙的外部資料來源。 本教學課程的資料檔案為 hotels.sql (在 \DotNetHowToIndexers 解決方案資料夾中提供)。 

### <a name="azure-sql-database"></a>連接字串

您可以使用 Azure 入口網站和範例中的 hotels.sql 檔案，在 Azure SQL Database 中建立資料集。 Azure 搜尋服務會取用扁平化資料列集，例如從檢視或查詢產生的資料列集。 範例方案中的 SQL 檔案會建立並填入單一資料表。

下列練習假設沒有任何現有的伺服器或資料庫，並指示您在步驟 2 中建立兩者。 (選擇性) 如果您有現有的資源，您可以在其中新增 hotels 資料表 (在步驟 4 開始)。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。 

2. 按一下 [建立資源] > [SQL Database]，以建立資料庫、伺服器和資源群組。 您可以使用預設值和最低層級的定價層。 建立伺服器的優點之一是您可以指定系統管理員使用者名稱和密碼，以便在稍後步驟中建立和載入資料表。

   ![新增資料庫頁面](./media/search-indexer-tutorial/indexer-new-sqldb.png)

3. 按一下 [建立] 來部署新的伺服器和資料庫。 等候部署伺服器和資料庫。

4. 開啟新資料庫的 [SQL Database] 頁面 (如果尚未開啟的話)。 資源名稱應該為 SQL Database，而非 SQL Server。

  ![SQL 資料庫頁面](./media/search-indexer-tutorial/hotels-db.png)

4. 在命令列上，按一下 [工具] > [查詢編輯器]。

5. 按一下 [登入]，並輸入伺服器管理員的使用者名稱和密碼。

6. 按一下 [開啟查詢] 並瀏覽至 hotels.sql 的位置。 

7. 選取檔案，然後按一下 [開啟]。 指令碼應該會看起來如下列螢幕擷取畫面所示：

  ![SQL 指令碼](./media/search-indexer-tutorial/sql-script.png)

8. 按一下 [執行] 來執行查詢。 在 [結果] 窗格中，您應會看到查詢成功訊息 (3 個資料列)。

9. 若要從這個資料表傳回資料列集，您可以執行下列查詢作為驗證步驟：

   ```sql
   SELECT HotelId, HotelName, Tags FROM Hotels
   ```
   典型查詢 `SELECT * FROM Hotels` 不適用於查詢編輯器中。 範例資料包含 [位置] 欄位中的地理座標，目前並未在編輯器中處理該欄位。 如需要查詢的其他資料行清單，您可以執行此陳述式：`SELECT * FROM sys.columns WHERE object_id = OBJECT_ID('dbo.Hotels')`

10. 您現在有外部資料集，請複製資料庫的 ADO.NET 連接字串。 在您資料庫的 [SQL Database] 頁面上，移至 [設定] > [連接字串]，並複製 ADO.NET 連接字串。
 
  ADO.NET 連接字串如下列範例所示，已修改成使用有效的資料庫名稱、使用者名稱和密碼。

  ```sql
  Server=tcp:hotels-db.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
  ```
11. 將連接字串貼到 "AzureSqlConnectionString" 中，作為 Visual Studio 的 **appsettings.json** 檔案中的第三個項目。

    ```json
    {
      "SearchServiceName": "azs-tutorial",
      "SearchServiceAdminApiKey": "A1B2C3D4E5F6G7H8I9J10K11L12M13N14",
      "AzureSqlConnectionString": "Server=tcp:hotels-db.database.windows.net,1433;Initial Catalog=hotels-db;Persist Security  Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;",
    }
    ```

## <a name="understand-index-and-indexer-code"></a>了解索引和索引子程式碼

您的程式碼現在已準備進行建置和執行。 這麼做之前，請花一分鐘研究此範例中的索引和索引子定義。 相關程式碼位於兩個檔案中：

  + **hotel.cs**，內含可定義索引的結構描述
  + **Program.cs**，內含用於建立和管理您服務中結構的函式

### <a name="in-hotelcs"></a>在 hotel.cs 中

索引結構描述會定義欄位集合，包括用於指定允許作業的屬性，例如欄位是否可全文檢索搜尋、可篩選，或可排序 (如 HotelName 的下列欄位定義中所示)。 

```csharp
. . . 
[IsSearchable, IsFilterable, IsSortable]
public string HotelName { get; set; }
. . .
```

結構描述也可以包含其他元素，包括提高搜尋分數、自訂分析城市和其他建構的評分設定檔。 不過，基於我們的目的，結構描述並未嚴密定義，只包含在範例資料集中找到的欄位。

在本教學課程中，索引子會從一個資料來源提取資料。 實際上，您可以將多個索引子附加至相同的索引，並從多個資料來源和索引子建立合併的可搜尋索引。 根據您需要彈性的狀況而定，您可以使用相同的索引-索引子配對、不同的資料來源組合，或具有各種索引子和資料來源組合的一個索引。

### <a name="in-programcs"></a>在 Program.cs 中

主要程式包含三個代表性資料來源的函式。 只將焦點放在 Azure SQL Database，下列物件會突顯出來：

  ```csharp
  private const string IndexName = "hotels";
  private const string AzureSqlHighWaterMarkColumnName = "RowVersion";
  private const string AzureSqlDataSourceName = "azure-sql";
  private const string AzureSqlIndexerName = "azure-sql-indexer";
  ```

在 Azure 搜尋中，您可以個別檢視、設定或刪除的物件包括索引、索引子和資料來源 (分別是 hotels、azure-sql-indexer、azure-sql)。 

*AzureSqlHighWaterMarkColumnName* 資料行值得特別一提，因為它提供變更偵測資訊，以便索引子用來判斷自從上次編製索引工作負載後資料列是否已變更。 [變更偵測原則](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)僅對索引子提供支援，並隨著資料來源而有所不同。 對於 Azure SQL Database，您可以根據資料庫需求，從兩個原則中進行選擇。

下列程式碼會顯示 Program.cs 中用來建立資料來源和索引子的方法。 在您可能會執行此程式多次的假設之下，此程式碼會檢查並刪除現有的同名資源。

  ```csharp
  private static string SetupAzureSqlIndexer(SearchServiceClient serviceClient, IConfigurationRoot configuration)
  {
    Console.WriteLine("Deleting Azure SQL data source if it exists...");
    DeleteDataSourceIfExists(serviceClient, AzureSqlDataSourceName);

    Console.WriteLine("Creating Azure SQL data source...");
    DataSource azureSqlDataSource = CreateAzureSqlDataSource(serviceClient, configuration);

    Console.WriteLine("Deleting Azure SQL indexer if it exists...");
    DeleteIndexerIfExists(serviceClient, AzureSqlIndexerName);

    Console.WriteLine("Creating Azure SQL indexer...");
    Indexer azureSqlIndexer = CreateIndexer(serviceClient, AzureSqlDataSourceName, AzureSqlIndexerName);

    return azureSqlIndexer.Name;
  }
  ```

請注意，索引子 API 呼叫與平台無關，但 [DataSourceType](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 除外，其可指定要叫用的編目程式類型。

## <a name="run-the-indexer"></a>執行索引子

在此步驟中，編譯並執行程式。 

1. 在 [方案總管] 中，於 **DotNetHowToIndexers** 上按一下滑鼠右鍵，然後選取 [建置]。
2. 同樣地，以滑鼠右鍵按一下 **DotNetHowToIndexers**，接著按一下 [偵錯] > [啟動新執行個體]。

此程式會在偵錯模式中執行。 主控台視窗會報告每項作業的狀態。

  ![SQL 指令碼](./media/search-indexer-tutorial/console-output.png)

您的程式碼會在 Visual Studio 本機執行，並連線到您在 Azure 上的搜尋服務，而搜尋服務會使用連接字串連線到 Azure SQL Database 並擷取資料集。 這麼多項作業會有數個潛在的失敗點，但如果發生錯誤，請先檢查下列：

+ 您提供搜尋服務連線資訊受限於本教學課程中的服務名稱。 如果您輸入了完整 URL，作業會在索引建立時停止，出現連線失敗錯誤。

+ **appsettings.json** 中的資料庫連線資訊。 它應該是從入口網站取得的 ADO.NET 連接字串，已修改成包含您的資料庫適用的使用者名稱和密碼。 使用者帳戶必須具有擷取資料的權限。

+ 資源限制。 回想一下，共用 (免費) 服務有 3 個索引、索引子和資料來源的限制。 最大限制的服務無法建立新的物件。

## <a name="search-the-index"></a>搜尋索引 

在 Azure 入口網站的搜尋服務 [概觀] 頁面中，按一下頂端的 [搜尋總管]，以提交新索引的一些查詢。

1. 按一下頂端的 [變更索引] 以選取 hotels 索引。

2. 按一下 [搜尋] 按鈕來發出空白搜尋。 

  您的索引中的三個項目會以 JSON 文件形式傳回。 搜尋總管會以 JSON 傳回文件，以便您檢視整個結構。

3. 接下來，輸入搜尋字串：`search=river&$count=true`。 

  此查詢會叫用 `river` 字詞的全文檢索搜尋，而結果會包含相符文件的計數。 在測試包含數千甚至數百萬份文件的大型索引案例時，傳回相符文件的計數很實用。 在此情況下，只有一份文件符合查詢。

4. 最後，輸入搜尋字串，將 JSON 輸出限制為感興趣的欄位：`search=river&$count=true&$select=hotelId, baseRate, description`。 

  查詢回應會縮減為選取的欄位，導致更簡潔的輸出。

## <a name="view-indexer-configuration"></a>檢視索引子組態

所有索引子 (包括您剛才以程式設計方式建立的索引子) 都會列在入口網站中。 您可以開啟索引子定義及檢視其資料來源，或設定重新整理排程，以掌握全新和變更的資料列。

1. 開啟 Azure 搜尋服務的服務 [概觀] 頁面。
2. 向下捲動以尋找 [索引子] 和 [資料來源] 的圖格。
3. 按一下圖格，以開啟每個資源的清單。 您可以選取個別的索引子或資料來源，以檢視或修改組態設定。

  ![索引子和資料來源圖格](./media/search-indexer-tutorial/tiles-portal.png)


## <a name="clean-up-resources"></a>清除資源

在完成教學課程後，最快速的清除方式是刪除包含 Azure 搜尋服務的資源群組。 您現在可以刪除資源群組，以永久刪除當中所包含的所有項目。 在入口網站中，資源群組名稱位在 Azure 搜尋服務的 [概觀] 頁面上。

## <a name="next-steps"></a>後續步驟

您可以將 AI 功能的演算法附加至索引子的管線。 下一個步驟中，繼續進行下列教學課程。

> [!div class="nextstepaction"]
> [在 Azure Blob 儲存體中為文件編製索引](search-howto-indexing-azure-blob-storage.md)