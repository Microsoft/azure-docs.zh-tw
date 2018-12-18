---
title: Azure Cosmos DB 的資料庫移轉工具 | Microsoft Docs
description: 了解如何使用開放原始碼 Azure Cosmos DB 資料移轉工具，將各種來源的資料 (包括 MongoDB、SQL Server、資料表儲存體、Amazon DynamoDB、CSV 及 JSON 檔案) 匯入到 Azure Cosmos DB。 將 CSV 轉換成 JSON。
keywords: csv 轉換成 json, 資料庫移轉工具, 將 csv 轉換成 json
services: cosmos-db
author: deborahc
manager: kfile
editor: monicar
ms.service: cosmos-db
ms.devlang: na
ms.topic: tutorial
ms.date: 03/30/2018
ms.author: dech
ms.custom: mvc
ms.openlocfilehash: af6faa6abcc54ef11e066d3a348dac28b23c7af4
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49079084"
---
# <a name="use-data-migration-tool-to-migrate-your-data-to-azure-cosmos-db"></a>使用資料移轉工具將您的資料移轉至 Azure Cosmos DB 

本教學課程提供使用 Azure Cosmos DB 資料移轉工具的相關指示，可將資料從各種來源匯入到 Azure Cosmos DB 集合和資料表。 您可以從 JSON 檔案、CSV 檔案、SQL、MongoDB、Azure 資料表儲存體、Amazon DynamoDB 以及甚至 Azure Cosmos DB SQL API 集合來匯入，並將該資料移轉到集合和資料表以供用於 Azure Cosmos DB。 針對 SQL API 從單一分割區集合移轉到多重分割區集合時，也可以使用資料移轉工具。

您要將哪個 API 用於 Azure Cosmos DB？ 

* **[SQL API](documentdb-introduction.md)** - 您可以使用資料移轉工具中所提供的任何來源選項來匯入資料。
* **[資料表 API](table-introduction.md)** - 您可以使用資料移轉工具或 AzCopy 來匯入資料。 如需詳細資訊，請參閱[匯入資料以用於 Azure Cosmos DB 資料表 API](table-import.md)。
* **[MongoDB API](mongodb-introduction.md)** - 資料移轉工具目前不支援 Azure Cosmos DB MongoDB API 作為來源或目標。 如果您想要在 Azure Cosmos DB 的 MongoDB API 集合中移入或移出資料，請參閱 [Azure Cosmos DB：如何針對 MongoDB API 移轉資料](mongodb-migrate.md)以取得相關指示。 您仍可使用資料移轉工具，將資料從 MongoDB 匯出到 Azure Cosmos DB SQL API 集合，以便使用於 SQL API。 
* **[Gremlin API](graph-introduction.md)** - 資料移轉工具目前仍不是 Gremlin API 帳戶所支援的匯入工具。 

本教學課程涵蓋下列工作：

> [!div class="checklist"]
> * 安裝資料移轉工具
> * 從不同的資料來源匯入資料
> * 從 Azure Cosmos DB 匯出到 JSON

## <a id="Prerequisites"></a>必要條件
在依照本文中的指示進行之前，請確定已安裝下列項目：

* [Microsoft.NET Framework 4.51](https://www.microsoft.com/download/developer-tools.aspx) 或更高版本。

* **增加輸送量︰** 資料移轉的時間長短取決於您為個別集合或一組集合設定的輸送量。 針對較大資料移轉，請務必增加輸送量。 完成移轉之後，再降低輸送量以節省成本。 如需在 Azure 入口網站增加輸送量的詳細資訊，請參閱 Azure Cosmos DB 中的效能等級和定價層。

* **建立 Azure Cosmos DB 資源：** 在您開始遷移資料之前，請先從 Azure 入口網站預先建立所有集合。 如果您要遷移至具有資料庫層級輸送量的 Azure Cosmos DB 帳戶，請務必在建立 Azure Cosmos DB 集合時提供分割區索引鍵。

## <a id="Overviewl"></a>概觀
資料移轉工具是一個開放原始碼解決方案，可將資料從各種來源匯入到 Azure Cosmos DB，來源包括：

* JSON 檔案
* MongoDB
* SQL Server
* CSV 檔案
* Azure 資料表儲存體
* Amazon DynamoDB
* hbase
* Azure Cosmos DB 集合

雖然匯入工具包括圖形化使用者介面 (dtui.exe)，您也可以從命令列 (dt.exe) 驅動此工具。 事實上，在透過 UI 設定匯入之後，有一個選項可以輸出相關聯的命令。 表格式來源資料 (例如 SQL Server 或 CSV 檔案) 可以進行轉換，以致可以在匯入期間建立階層式關聯性 (子文件)。 繼續閱讀以深入了解來源選項、從每個來源匯入的範例命令、目標選項，以及檢視匯入結果。

## <a id="Install"></a>安裝
移轉工具的原始程式碼可在 GitHub 上的[這個存放庫](https://github.com/azure/azure-documentdb-datamigrationtool)中取得。 您可以在本機下載並編譯解決方案，或下載[預先編譯的二進位檔](https://cosmosdbportalstorage.blob.core.windows.net/datamigrationtool/2018.02.28-1.8.1/dt-1.8.1.zip)，然後執行：

* **Dtui.exe**：此工具的圖形化介面版本
* **Dt.exe**：此工具的命令列版本

## <a name="select-data-source"></a>選取資料來源

一旦已安裝此工具之後，就可以匯入您的資料。 您要匯入的資料類型為何？

* [JSON 檔案](#JSON)
* [MongoDB](#MongoDB)
* [MongoDB 匯出檔案](#MongoDBExport)
* [SQL Server](#SQL)
* [CSV 檔案](#CSV)
* [Azure 表格儲存體](#AzureTableSource)
* [Amazon DynamoDB](#DynamoDBSource)
* [Blob](#BlobImport)
* [Azure Cosmos DB 集合](#SQLSource)
* [HBase](#HBaseSource)
* [Azure Cosmos DB 大量匯入](#SQLBulkTarget)
* [Azure Cosmos DB 循序記錄匯入](#SQLSeqTarget)


## <a id="JSON"></a>匯入 JSON 檔案
JSON 檔案來源匯入工具選項可讓您匯入一或多個單一文件 JSON 檔案或包含 JSON 文件陣列的每個 JSON 檔案。 新增包含要匯入之 JSON 檔案的資料夾時，您可以選擇是否要以遞迴方式搜尋子資料夾中的檔案。

![JSON 檔案來源選項的螢幕擷取畫面 - 資料庫移轉工具](./media/import-data/jsonsource.png)

以下是匯入 JSON 檔案的一些命令列範例：

    #Import a single JSON file
    dt.exe /s:JsonFile /s.Files:.\Sessions.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

    #Import a directory of JSON files
    dt.exe /s:JsonFile /s.Files:C:\TESessions\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Sessions /t.CollectionThroughput:2500

    #Import a directory (including sub-directories) of JSON files
    dt.exe /s:JsonFile /s.Files:C:\LastFMMusic\**\*.json /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Music /t.CollectionThroughput:2500

    #Import a directory (single), directory (recursive), and individual JSON files
    dt.exe /s:JsonFile /s.Files:C:\Tweets\*.*;C:\LargeDocs\**\*.*;C:\TESessions\Session48172.json;C:\TESessions\Session48173.json;C:\TESessions\Session48174.json;C:\TESessions\Session48175.json;C:\TESessions\Session48177.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:subs /t.CollectionThroughput:2500

    #Import a single JSON file and partition the data across 4 collections
    dt.exe /s:JsonFile /s.Files:D:\\CompanyData\\Companies.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:comp[1-4] /t.PartitionKey:name /t.CollectionThroughput:2500

## <a id="MongoDB"></a>從 MongoDB 匯入

> [!IMPORTANT]
> 如果您要匯入到具有 MongoDB 支援的 Azure Cosmos DB 帳戶，請遵循這些[指示](mongodb-migrate.md)。
> 
> 

MongoDB 來源匯入工具選項可讓您從個別的 MongoDB 集合匯入，並使用查詢來選擇性地篩選文件，及/或使用投影來修改文件結構。  

![MongoDB 來源選項的螢幕擷取畫面](./media/import-data/mongodbsource.png)

連接字串會採用標準的 MongoDB 格式：

    mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database>

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 MongoDB 執行個體，請使用 Verify 命令。
> 
> 

輸入要匯出資料的集合名稱。 您可以選擇性地對這兩個篩選器指定或提供檔案進行查詢 (例如 {pop: {$gt:5000}} ) 及/或投射 (例如 {loc:0} )，並使資料適合匯入作業。

以下是從 MongoDB 匯入的一些命令列範例：

    #Import all documents from a MongoDB collection
    dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZips /t.IdField:_id /t.CollectionThroughput:2500

    #Import documents from a MongoDB collection which match the query and exclude the loc field
    dt.exe /s:MongoDB /s.ConnectionString:mongodb://<dbuser>:<dbpassword>@<host>:<port>/<database> /s.Collection:zips /s.Query:{pop:{$gt:50000}} /s.Projection:{loc:0} /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:BulkZipsTransform /t.IdField:_id/t.CollectionThroughput:2500

## <a id="MongoDBExport"></a>匯入 MongoDB 匯出檔案

> [!IMPORTANT]
> 如果您要匯入具有 MongoDB 支援的 Azure Cosmos DB 帳戶中，請遵循這些[指示](mongodb-migrate.md)。
> 
> 

MongoDB 匯出 JSON 檔案來源匯入工具選項可讓您匯入從 mongoexport 公用程式產生的一或多個 JSON 檔案。  

![MongoDB 匯出來源選項的螢幕擷取畫面](./media/import-data/mongodbexportsource.png)

新增包含要匯入之 MongoDB 匯出 JSON 檔案的資料夾，您可以選擇是否要以遞迴方式搜尋子資料夾中的檔案。

以下是從 MongoDB 匯出 JSON 檔案匯入的命令列範例：

    dt.exe /s:MongoDBExport /s.Files:D:\mongoemployees.json /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:employees /t.IdField:_id /t.Dates:Epoch /t.CollectionThroughput:2500

## <a id="SQL"></a>從 SQL Server 匯入
SQL 來源匯入工具選項可讓您從個別的 SQL Server 資料庫匯入，並使用查詢來選擇性地篩選要匯入的記錄。 此外，您可以藉由指定巢狀分隔符號 (稍後將有更詳細的說明) 來修改文件結構。  

![SQL 來源選項的螢幕擷取畫面 - 資料庫移轉工具](./media/import-data/sqlexportsource.png)

連接字串的格式會採用標準的 SQL 連接字串格式。

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 SQL Server 執行個體，請使用 Verify 命令。
> 
> 

巢狀分隔符號屬性可用來在匯入期間建立階層式關聯性 (子文件)。 請考慮下列 SQL 查詢：

<bpt id="p1">*</bpt>select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'<ept id="p1">*</ept>

它會傳回下列 (部分) 結果：

![SQL 查詢結果的螢幕擷取畫面](./media/import-data/sqlqueryresults.png)

注意別名，例如 Address.AddressType 和 Address.Location.StateProvinceName。 藉由指定巢狀分隔符號 '.'，匯入工具會在匯入期間建立 Address 和 Address.Location 子文件。 在 Azure Cosmos DB 中產生的文件範例如下：

{ "id": "956", "Name": "Finer Sales and Service", "Address": { "AddressType": "Main Office", "AddressLine1": "#500-75 O'Connor Street", "Location": { "City": "Ottawa", "StateProvinceName": "Ontario" }, "PostalCode": "K4B 1S2", "CountryRegionName": "Canada" } }

以下是從 SQL Server 匯入的一些命令列範例：

    #Import records from SQL which match a query
    dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, * from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Stores /t.IdField:Id /t.CollectionThroughput:2500

    #Import records from sql which match a query and create hierarchical relationships
    dt.exe /s:SQL /s.ConnectionString:"Data Source=<server>;Initial Catalog=AdventureWorks;User Id=advworks;Password=<password>;" /s.Query:"select CAST(BusinessEntityID AS varchar) as Id, Name, AddressType as [Address.AddressType], AddressLine1 as [Address.AddressLine1], City as [Address.Location.City], StateProvinceName as [Address.Location.StateProvinceName], PostalCode as [Address.PostalCode], CountryRegionName as [Address.CountryRegionName] from Sales.vStoreWithAddresses WHERE AddressType='Main Office'" /s.NestingSeparator:. /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:StoresSub /t.IdField:Id /t.CollectionThroughput:2500

## <a id="CSV"></a>匯入 CSV 檔案並將 CSV 轉換成 JSON
CSV 檔案來源匯入工具選項可讓您匯入一或多個 CSV 檔案。 新增包含要匯入之 CSV 檔案的資料夾時，您可以選擇是否要以遞迴方式搜尋子資料夾中的檔案。

![CSV 來源選項的螢幕擷取畫面 - CSV 轉換成 JSON](media/import-data/csvsource.png)

與 SQL 來源類似，巢狀分隔符號屬性可用來在匯入期間建立階層式關聯性 (子文件)。 請考慮下列 CSV 標頭資料列和資料資料列：

![CSV 範例記錄的螢幕擷取畫面 - CSV 轉換成 JSON](./media/import-data/csvsample.png)

注意別名，例如 DomainInfo.Domain_Name 和 RedirectInfo.Redirecting。 藉由指定巢狀分隔符號 '.'，匯入工具將會在匯入期間建立 DomainInfo 和 RedirectInfo 子文件。 在 Azure Cosmos DB 中產生的文件範例如下：

*{ "DomainInfo": { "Domain_Name": "ACUS.GOV", "Domain_Name_Address": "http://www.ACUS.GOV" }, "Federal Agency": "Administrative Conference of the United States", "RedirectInfo": { "Redirecting": "0", "Redirect_Destination": "" }, "id": "9cc565c5-ebcd-1c03-ebd3-cc3e2ecd814d" }*

匯入工具會嘗試推斷 CSV 檔案中不具引號之值的類型資訊 (加上引號的值永遠會被視為字串)。  系統會依照下列順序識別類型：數字、日期時間、布林值。  

下面提供兩個其他有關 CSV 匯入的注意事項：

1. 根據預設，不具引號的值一律會針對定位點和空格進行修剪，而具有引號的值則會以原樣方式加以保留。 您可以使用 [修剪具有引號的值] 核取方塊或 /s.TrimQuoted 命令列選項，來覆寫這個行為。
2. 根據預設，不具引號的 Null 會被視為 Null 值。 您可以使用 [將不具引號的 NULL 視為字串] 核取方塊或 /s.NoUnquotedNulls 命令列選項，來覆寫這個行為 (亦即將不具引號的 Null 視為 "null" 字串)。

以下是 CSV 匯入的命令列範例：

    dt.exe /s:CsvFile /s.Files:.\Employees.csv /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:Employees /t.IdField:EntityID /t.CollectionThroughput:2500

## <a id="AzureTableSource"></a>從 Azure 資料表儲存體匯入
Azure 資料表儲存體來源匯入工具選項可讓您從個別的 Azure 資料表儲存體資料表匯入。 您可以選擇性地篩選要匯入的資料表實體。 

從 Azure 資料表儲存體匯入的資料，可以輸出到 Azure Cosmos DB 資料表和實體以用於資料表 API，或是輸出到集合和文件以用於 SQL API。 不過，在命令列公用程式中，資料表 API 只可做為目標，您無法使用資料移轉工具使用者介面來匯出到資料表 API。 如需詳細資訊，請參閱[匯入資料以用於 Azure Cosmos DB 資料表 API](table-import.md)。 

![Azure 資料表儲存體來源選項的螢幕擷取畫面](./media/import-data/azuretablesource.png)

Azure 資料表儲存體連接字串的格式如下：

    DefaultEndpointsProtocol=<protocol>;AccountName=<Account Name>;AccountKey=<Account Key>;

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 Azure 資料表儲存體執行個體，請使用 Verify 命令。
> 
> 

輸入要從中匯入資料的 Azure 資料表名稱。 您可以選擇性地指定 [篩選器](../vs-azure-tools-table-designer-construct-filter-strings.md)。

Azure 資料表儲存體來源匯入工具選項具有下列其他選項：

1. 包含內部欄位
   1. 全部 - 包括所有內部欄位 (PartitionKey、RowKey 和 Timestamp)
   2. 無 - 排除所有內部欄位
   3. RowKey - 僅包含 RowKey 欄位
2. 選取資料行
   1. Azure 資料表儲存體篩選器不支援投影。 如果您只想匯入特定的 Azure 資料表實體屬性，請將它們加入 [選取資料行] 清單。 其他所有實體屬性都會被忽略。

以下是從 Azure 資料表儲存體匯入的命令列範例：

    dt.exe /s:AzureTable /s.ConnectionString:"DefaultEndpointsProtocol=https;AccountName=<Account Name>;AccountKey=<Account Key>" /s.Table:metrics /s.InternalFields:All /s.Filter:"PartitionKey eq 'Partition1' and RowKey gt '00001'" /s.Projection:ObjectCount;ObjectSize  /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:metrics /t.CollectionThroughput:2500

## <a id="DynamoDBSource"></a>從 Amazon DynamoDB 匯入
Amazon DynamoDB 來源匯入工具選項可讓您從個別的 Amazon DynamoDB 資料表匯入，並選擇性地篩選要匯入的實體。 會提供數個範本，讓設定匯入盡量簡化。

![Amazon DynamoDB 來源選項的螢幕擷取畫面 - 資料庫移轉工具](./media/import-data/dynamodbsource1.png)

![Amazon DynamoDB 來源選項的螢幕擷取畫面 - 資料庫移轉工具](./media/import-data/dynamodbsource2.png)

Amazon DynamoDB 連接字串的格式如下：

    ServiceURL=<Service Address>;AccessKey=<Access Key>;SecretKey=<Secret Key>;

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 Amazon DynamoDB 執行個體，請使用 Verify 命令。
> 
> 

以下是從 Amazon DynamoDB 匯入的命令列範例：

    dt.exe /s:DynamoDB /s.ConnectionString:ServiceURL=https://dynamodb.us-east-1.amazonaws.com;AccessKey=<accessKey>;SecretKey=<secretKey> /s.Request:"{   """TableName""": """ProductCatalog""" }" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<Azure Cosmos DB Endpoint>;AccountKey=<Azure Cosmos DB Key>;Database=<Azure Cosmos DB Database>;" /t.Collection:catalogCollection /t.CollectionThroughput:2500

## <a id="BlobImport"></a>從 Azure Blob 儲存體匯入
JSON 檔案、MongoDB 匯出檔案和 CSV 檔案來源匯入工具選項可讓您從 Azure Blob 儲存體匯入一或多個檔案。 在指定 Blob 容器 URL 和帳戶金鑰之後，請提供規則運算式來選取要匯入的檔案。

![Blob 檔案來源選項的螢幕擷取畫面](./media/import-data/blobsource.png)

以下是從 Azure Blob 儲存體匯入 JSON 檔案的命令列範例：

    dt.exe /s:JsonFile /s.Files:"blobs://<account key>@account.blob.core.windows.net:443/importcontainer/.*" /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:doctest

## <a id="SQLSource"></a>從 SQL API 集合匯入
Azure Cosmos DB 來源匯入工具選項可讓您從一或多個 Azure Cosmos DB 集合匯入資料，並選擇性地使用查詢來篩選文件。  

![Azure Cosmos DB 來源選項的螢幕擷取畫面](./media/import-data/documentdbsource.png)

Azure Cosmos DB 連接字串的格式如下：

    AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;

就像[如何管理 Azure Cosmos DB 帳戶](manage-account.md)中所述，可從 Azure 入口網站的 [金鑰] 頁面中擷取 Azure Cosmos DB 帳戶的連接字串，但必須以下列格式將資料庫的名稱附加至連接字串：

    Database=<CosmosDB Database>;

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 Azure Cosmos DB 執行個體，請使用 Verify 命令。
> 
> 

若要從單一 Azure Cosmos DB 集合匯入，請輸入要從中匯入資料的來源集合名稱。 若要從多個 Azure Cosmos DB 集合匯入，可提供規則運算式來比對一或多個集合名稱 (例如 collection01 | collection02 | collection03)。 您可以選擇性地對這兩個篩選器指定或提供檔案進行查詢，並使資料適合匯入作業。

> [!NOTE]
> 因為集合欄位接受規則運算式，所以，若您要從其名稱包含規則運算式字元的單一集合進行匯入，則必須對應地將那些字元逸出。
> 
> 

Azure Cosmos DB 來源匯入工具選項具有下列進階選項：

1. 包括內部欄位：指定匯出中是否包含 Azure Cosmos DB 文件系統屬性 (例如 _rid、_ts)。
2. 失敗時的重試次數：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時的重試次數。
3. 重試間隔：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時兩次重試之間要等候的時間。
4. 連線模式：指定要與 Azure Cosmos DB 搭配使用的連線模式。 可用的選項包括：DirectTcp、DirectHttps 和閘道器。 直接連線模式會比較快，但閘道器模式比較支援防火牆，因為它只會使用連接埠 443。

![Azure Cosmos DB 來源進階選項的螢幕擷取畫面](./media/import-data/documentdbsourceoptions.png)

> [!TIP]
> 匯入工具會預設 DirectTcp 連線模式。 如果您遇到防火牆問題，請切換到閘道器連線模式，因為它只需要連接埠 443。
> 
> 

以下是從 Azure Cosmos DB 匯入的一些命令列範例：

    #Migrate data from one Azure Cosmos DB collection to another Azure Cosmos DB collections
    dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:TEColl /t:DocumentDBBulk /t.ConnectionString:" AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:TESessions /t.CollectionThroughput:2500

    #Migrate data from multiple Azure Cosmos DB collections to a single Azure Cosmos DB collection
    dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:comp1|comp2|comp3|comp4 /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:singleCollection /t.CollectionThroughput:2500

    #Export an Azure Cosmos DB collection to a JSON file
    dt.exe /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /s.Collection:StoresSub /t:JsonFile /t.File:StoresExport.json /t.Overwrite /t.CollectionThroughput:2500

> [!TIP]
> Azure Cosmos DB 資料匯入工具也支援從 [Azure Cosmos DB 模擬器](local-emulator.md)匯入資料。 從本機模擬器匯入資料時，請將端點設定為 `https://localhost:<port>`。 
> 
> 

## <a id="HBaseSource"></a>從 HBase 匯入
HBase 來源匯入工具選項可讓您從 HBase 資料表匯入資料，並選擇性地篩選資料。 會提供數個範本，讓設定匯入盡量簡化。

![HBase 檔案來源選項的螢幕擷取畫面](./media/import-data/hbasesource1.png)

![HBase 檔案來源選項的螢幕擷取畫面](./media/import-data/hbasesource2.png)

HBase Stargate 連接字串的格式如下：

    ServiceURL=<server-address>;Username=<username>;Password=<password>

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 HBase 執行個體，請使用 Verify 命令。
> 
> 

以下是從 HBase 匯入的命令列範例：

    dt.exe /s:HBase /s.ConnectionString:ServiceURL=<server-address>;Username=<username>;Password=<password> /s.Table:Contacts /t:DocumentDBBulk /t.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;" /t.Collection:hbaseimport

## <a id="SQLBulkTarget"></a>匯入 SQL API (大量匯入)
為了提高效率，Azure Cosmos DB 大量匯入工具可讓您使用 Azure Cosmos DB 預存程序，從任何可用的來源選項匯入。 此工具支援匯入到一個單一分割的 Azure Cosmos DB 集合，以及跨多個單一分割 Azure Cosmos DB 集合分割資料的分區化匯入。 如需分割資料的詳細資訊，請參閱 [Azure Cosmos DB 的資料分割與調整規模](partition-data.md)。 此工具會建立並執行預存程序，然後從目標集合中將它刪除。  

![Azure Cosmos DB 大量選項的螢幕擷取畫面](./media/import-data/documentdbbulk.png)

Azure Cosmos DB 連接字串的格式如下：

    AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;

就像[如何管理 Azure Cosmos DB 帳戶](manage-account.md)中所述，可從 Azure 入口網站的 [金鑰] 頁面中擷取 Azure Cosmos DB 帳戶的連接字串，但必須以下列格式將資料庫的名稱附加至連接字串：

    Database=<CosmosDB Database>;

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 Azure Cosmos DB 執行個體，請使用 Verify 命令。
> 
> 

若要匯入到單一集合，請輸入要匯入資料的目標集合名稱，然後按一下 [新增] 按鈕。 若要匯入到多個集合，請分別輸入每個集合的名稱，或使用下列語法來指定多個集合：collection_prefix[開始索引 - 結束索引]。 透過上述語法指定多個集合時，請記住下列指導方針：

1. 僅支援整數範圍的名稱模式。 例如，指定 collection[0-3] 會建立下列集合：collection0、collection1、collection2、collection3。
2. 您可以使用縮寫的語法：collection[3] 會建立一組與步驟 1 中所述相同的集合。
3. 您可以提供一個以上的替代項目。 例如，collection[0-1] [0-9] 會產生 20 個開頭為零的集合名稱 (collection01、..02、..03)。

指定集合名稱之後，請選擇所需的集合輸送量 (400 RU 到 10,000 RU)。 為了達到最佳的匯入效能，請選擇較高的輸送量。 如需效能層級的詳細資訊，請參閱 [Azure Cosmos DB 中的效能層級](performance-levels.md)。

> [!NOTE]
> 效能輸送量設定僅適用於建立集合。 如果指定的集合已經存在，將不會修改其輸送量。
> 
> 

匯入到多個集合時，匯入工具支援以雜湊為基礎的分區化。 在此案例中，指定您想要用來做為資料分割索引鍵的文件屬性 (如果資料分割索引鍵是空白的，文件就會跨目標集合隨機進行分區化)。

您可以選擇性地指定在匯入期間，匯入來源中的哪些欄位應該作為 Azure Cosmos DB 文件識別碼屬性使用 (如果文件不包含這個屬性，則匯入工具會產生 GUID 做為識別碼屬性值)。

匯入期間有數個可用的進階選項。 首先，雖然此工具包含預設的大量匯入預存程序 (BulkInsert.js)，但您可以選擇指定自己的匯入預存程序：

 ![Azure Cosmos DB 大量插入 sproc 選項的螢幕擷取畫面](./media/import-data/bulkinsertsp.png)

此外，匯入日期類型時 (例如從 SQL Server 或 MongoDB)，有三種匯入選項可供選擇：

 ![Azure Cosmos DB 日期時間匯入選項的螢幕擷取畫面](./media/import-data/datetimeoptions.png)

* 字串：保存為字串值
* Epoch：保存為 Epoch 數值
* 兩者：保存字串和 Epoch 數值。 這個選項會建立子文件，例如："date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

Azure Cosmos DB 大量匯入工具含有下列其他進階選項：

1. 批次大小：此工具會將批次大小預設為 50。  如果要匯入很大的文件，請考慮降低批次大小。 相反地，如果要匯入很小的文件，請考慮提高批次大小。
2. 指令碼大小上限 (位元組)：此工具預設將指令碼大小上限設為 512 KB。
3. 停用自動化識別碼產生作業：如果要匯入的每個文件都包含識別碼欄位，則選取此選項可以提升效能。 遺漏唯一識別碼欄位的文件不會被匯入。
4. 更新現有的文件：此工具預設在發生識別碼衝突時不會取代現有的文件。 選取此選項會允許在識別碼相符時覆寫現有的文件。 對於會更新現有文件的已排定資料移轉來說，這項功能相當有用。
5. 失敗時的重試次數：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時的重試次數。
6. 重試間隔：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時兩次重試之間要等候的時間。
7. 連線模式：指定要與 Azure Cosmos DB 搭配使用的連線模式。 可用的選項包括：DirectTcp、DirectHttps 和閘道器。 直接連線模式會比較快，但閘道器模式比較支援防火牆，因為它只會使用連接埠 443。

![Azure Cosmos DB 大量匯入進階選項的螢幕擷取畫面](./media/import-data/docdbbulkoptions.png)

> [!TIP]
> 匯入工具會預設 DirectTcp 連線模式。 如果您遇到防火牆問題，請切換到閘道器連線模式，因為它只需要連接埠 443。
> 
> 

## <a id="SQLSeqTarget"></a>匯入 SQL API (循序記錄匯入)
Azure Cosmos DB 循序記錄匯入工具可讓您從任何可用的來源選項逐筆匯入記錄。 如果您打算匯入至已達到預存程序配額的現有集合，您可以選擇此選項。 此工具支援匯入到單一 (單一分割區和多重分割區兩者) 的 Azure Cosmos DB 集合，以及跨多個單一分割區和/或多重分割區 Azure Cosmos DB 集合分割資料的分區化匯入。 如需分割資料的詳細資訊，請參閱 [Azure Cosmos DB 的資料分割與調整規模](partition-data.md)。

![Azure Cosmos DB 循序記錄匯入選項的螢幕擷取畫面](./media/import-data/documentdbsequential.png)

Azure Cosmos DB 連接字串的格式如下：

    AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB Database>;

就像[如何管理 Azure Cosmos DB 帳戶](manage-account.md)中所述，可從 Azure 入口網站的 [金鑰] 頁面中擷取 Azure Cosmos DB 帳戶的連接字串，但必須以下列格式將資料庫的名稱附加至連接字串：

    Database=<Azure Cosmos DB Database>;

> [!NOTE]
> 若要確定可以存取連接字串欄位中指定的 Azure Cosmos DB 執行個體，請使用 Verify 命令。
> 
> 

若要匯入到單一集合，請輸入要匯入資料的目標集合名稱，然後按一下 [新增] 按鈕。 若要匯入到多個集合，請分別輸入每個集合的名稱，或使用下列語法來指定多個集合：collection_prefix[開始索引 - 結束索引]。 透過上述語法指定多個集合時，請記住下列指導方針：

1. 僅支援整數範圍的名稱模式。 例如，指定 collection[0-3] 會建立下列集合：collection0、collection1、collection2、collection3。
2. 您可以使用縮寫的語法：collection[3] 會建立一組與步驟 1 中所述相同的集合。
3. 您可以提供一個以上的替代項目。 例如，collection[0-1] [0-9] 會建立 20 個開頭為零的集合名稱 (collection01、..02、..03)。

指定集合名稱之後，請選擇所需的集合輸送量 (400 RU 到 250,000 RU)。 為了達到最佳的匯入效能，請選擇較高的輸送量。 如需效能層級的詳細資訊，請參閱 [Azure Cosmos DB 中的效能層級](performance-levels.md)。 任何輸送量 >10000 RU 的集合匯入作業都需要資料分割索引鍵。 如果您選擇擁有 250,000 以上的 RU，您需要在入口網站中提出要求來提升您的帳戶。

> [!NOTE]
> 輸送量設定僅適用於集合或資料庫的建立。 如果指定的集合已經存在，將不會修改其輸送量。
> 
> 

匯入到多個集合時，匯入工具支援以雜湊為基礎的分區化。 在此案例中，指定您想要用來做為資料分割索引鍵的文件屬性 (如果資料分割索引鍵是空白的，文件就會跨目標集合隨機進行分區化)。

您可以選擇性地指定在匯入期間，匯入來源中的哪些欄位應該作為 Azure Cosmos DB 文件識別碼屬性使用 (如果文件不包含這個屬性，則匯入工具會產生 GUID 做為識別碼屬性值)。

匯入期間有數個可用的進階選項。 首先，匯入日期類型時 (例如從 SQL Server 或 MongoDB)，有三種匯入選項可供選擇：

 ![Azure Cosmos DB 日期時間匯入選項的螢幕擷取畫面](./media/import-data/datetimeoptions.png)

* 字串：保存為字串值
* Epoch：保存為 Epoch 數值
* 兩者：保存字串和 Epoch 數值。 這個選項會建立子文件，例如："date_joined": { "Value": "2013-10-21T21:17:25.2410000Z", "Epoch": 1382390245 }

Azure Cosmos DB 循序記錄匯入工具含有下列其他進階選項：

1. 平行要求數目：此工具會預設為兩個平行要求。 如果要匯入很小的文件，請考慮提高平行要求數目。 如果這個數字提高太多，則匯入可能會遭遇速率限制。
2. 停用自動化識別碼產生作業：如果要匯入的每個文件都包含識別碼欄位，則選取此選項可以提升效能。 遺漏唯一識別碼欄位的文件不會被匯入。
3. 更新現有的文件：此工具預設在發生識別碼衝突時不會取代現有的文件。 選取此選項會允許在識別碼相符時覆寫現有的文件。 對於會更新現有文件的已排定資料移轉來說，這項功能相當有用。
4. 失敗時的重試次數：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時的重試次數。
5. 重試間隔：指定與 Azure Cosmos DB 的連接發生暫時性失敗 (例如網路連接中斷) 時兩次重試之間要等候的時間。
6. 連線模式：指定要與 Azure Cosmos DB 搭配使用的連線模式。 可用的選項包括：DirectTcp、DirectHttps 和閘道器。 直接連線模式會比較快，但閘道器模式比較支援防火牆，因為它只會使用連接埠 443。

![Azure Cosmos DB 循序記錄匯入進階選項的螢幕擷取畫面](./media/import-data/documentdbsequentialoptions.png)

> [!TIP]
> 匯入工具會預設 DirectTcp 連線模式。 如果您遇到防火牆問題，請切換到閘道器連線模式，因為它只需要連接埠 443。
> 
> 

## <a id="IndexingPolicy"></a>指定索引編製原則
當您允許移轉工具在匯入期間建立 Azure Cosmos DB SQL API 集合時，您可以指定集合的索引編製原則。 在 Azure Cosmos DB 大量匯入和 Azure Cosmos DB 循序記錄選項的進階選項區段中，瀏覽至 [編製索引原則] 區段。

![Azure Cosmos DB 編製索引原則進階選項的螢幕擷取畫面](./media/import-data/indexingpolicy1.png)

使用索引編製原則進階選項，您可以選取索引編製原則檔案，以手動方式輸入索引編製原則，或從一組預設範本中選取 (以滑鼠右鍵按一下索引編製原則文字方塊)。

此工具提供的原則範本是：

* 預設值。 當您使用 ORDER BY、範圍和數字的相等查詢針對字串進行相等查詢時，這是最佳的原則。 這個原則的索引儲存空間負擔比範圍低。
* 範圍。 當您在數字和字串上都使用 ORDER BY、範圍及相等查詢時，這是最佳的原則。 這個原則的索引儲存空間負擔比預設或雜湊高。

![Azure Cosmos DB 編製索引原則進階選項的螢幕擷取畫面](./media/import-data/indexingpolicy2.png)

> [!NOTE]
> 如果未指定索引編製原則，則會套用預設原則。 如需編製索引原則的詳細資訊，請參閱 [Azure Cosmos DB 編製索引原則](indexing-policies.md)。
> 
> 

## <a name="export-to-json-file"></a>匯出至 JSON 檔案
Azure Cosmos DB JSON 匯出工具可讓您將任何可用的來源選項匯出至包含 JSON 文件陣列的 JSON 檔案。 此工具會為您處理匯出作業，或者您可以選擇檢視產生的移轉命令，然後自己執行命令。 產生的 JSON 檔案可能會儲存在本機或 Azure Blob 儲存體中。

![Azure Cosmos DB JSON 本機檔案匯出選項的螢幕擷取畫面](./media/import-data/jsontarget.png)

![Azure Cosmos DB JSON Azure Blob 儲存體匯出選項的螢幕擷取畫面](./media/import-data/jsontarget2.png)

您可以選擇性地選擇美化產生的 JSON，這會增加產生文件的大小，但會使內容更容易閱讀。

    Standard JSON export
    [{"id":"Sample","Title":"About Paris","Language":{"Name":"English"},"Author":{"Name":"Don","Location":{"City":"Paris","Country":"France"}},"Content":"Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.","PageViews":10000,"Topics":[{"Title":"History of Paris"},{"Title":"Places to see in Paris"}]}]

    Prettified JSON export
    [
     {
    "id": "Sample",
    "Title": "About Paris",
    "Language": {
      "Name": "English"
    },
    "Author": {
      "Name": "Don",
      "Location": {
        "City": "Paris",
        "Country": "France"
      }
    },
    "Content": "Don's document in Azure Cosmos DB is a valid JSON document as defined by the JSON spec.",
    "PageViews": 10000,
    "Topics": [
      {
        "Title": "History of Paris"
      },
      {
        "Title": "Places to see in Paris"
      }
    ]
    }]

以下是將 JSON 檔案匯出至 Azure Blob 儲存體的命令列範例：

```
dt.exe /ErrorDetails:All /s:DocumentDB /s.ConnectionString:"AccountEndpoint=<CosmosDB Endpoint>;AccountKey=<CosmosDB Key>;Database=<CosmosDB database_name>" /s.Collection:<CosmosDB collection_name>
/t:JsonFile /t.File:"blobs://<Storage account key>@<Storage account name>.blob.core.windows.net:443/<Container_name>/<Blob_name>"
/t.Overwrite
```

## <a name="advanced-configuration"></a>進階組態
在 [進階組態] 畫面中，指定您想要寫入所有錯誤的記錄檔位置。 下列規則會套用到此頁面：

1. 如果未提供檔案名稱，則會在 [結果] 頁面上傳回所有錯誤。
2. 如果提供不含目錄的檔案名稱，則會在目前的環境目錄中建立 (或覆寫) 該檔案。
3. 如果選取了現有的檔案，則會覆寫該檔案，沒有任何附加選項。
4. 然後，選擇是要記錄所有錯誤訊息、嚴重錯誤訊息，還是不記錄任何錯誤訊息。 最後，決定螢幕上傳輸訊息更新其進度的頻率。

   ![進階設定畫面的螢幕擷取畫面](./media/import-data/AdvancedConfiguration.png)

## <a name="confirm-import-settings-and-view-command-line"></a>確認匯入設定及檢視命令列
1. 指定來源資訊、目標資訊與進階組態之後，請檢閱移轉摘要，並選擇性地檢視或複製產生的移轉命令 (複製命令對於自動匯入作業非常有用)：
   
    ![摘要畫面的螢幕擷取畫面](./media/import-data/summary.png)
   
    ![摘要畫面的螢幕擷取畫面](./media/import-data/summarycommand.png)
2. 在您滿意來源和目標選項之後，請按一下 [匯入] 。 隨著匯入的處理，已耗用時間、傳送的計數，以及失敗的資訊 (如果您未在 [進階組態] 中提供檔案名稱) 都會隨之更新。 完成後，您可以匯出結果 (例如處理任何匯入失敗)。
   
    ![Azure Cosmos DB JSON 匯出選項的螢幕擷取畫面](./media/import-data/viewresults.png)
3. 您也可以啟動新的匯入，您可以保留現有設定 (例如連接字串資訊、來源和目標選擇等)，或重設所有值。
   
    ![Azure Cosmos DB JSON 匯出選項的螢幕擷取畫面](./media/import-data/newimport.png)

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已完成下列工作：

> [!div class="checklist"]
> * 已安裝資料移轉工具
> * 已從不同的資料來源匯入資料
> * 已從 Azure Cosmos DB 匯出到 JSON

您現在可以繼續進行下一個教學課程，了解如何使用 Azure Cosmos DB 查詢資料。 

> [!div class="nextstepaction"]
>[如何查詢資料？](../cosmos-db/tutorial-query-sql-api.md)
