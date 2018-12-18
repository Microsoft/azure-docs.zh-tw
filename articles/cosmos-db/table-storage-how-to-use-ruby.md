---
title: 如何搭配 Ruby 使用 Azure 表格儲存體和 Azure Cosmos DB 資料表 API | Microsoft Docs
description: 使用 Azure 表格儲存體或 Azure Cosmos DB 資料表 API 將結構化資料儲存在雲端。
services: cosmos-db
author: SnehaGunda
manager: kfile
editor: ''
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: ruby
ms.topic: sample
ms.date: 04/05/2018
ms.author: sngun
ms.openlocfilehash: d1583001550f5f272f4070006a4a6ac3be000de6
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34798265"
---
# <a name="how-to-use-azure-table-storage-and-the-azure-cosmos-db-table-api-with-ruby"></a>如何搭配 Ruby 使用 Azure 表格儲存體和 Azure Cosmos DB 資料表 API
[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

## <a name="overview"></a>概觀
本指南說明如何使用 Azure 表格服務和 Azure Cosmos DB 資料表 API 來執行一般案例。 這些範例是以 Ruby 撰寫的，並且使用 [適用於 Ruby 的 Azure 儲存體資料表用戶端程式庫](https://github.com/azure/azure-storage-ruby/tree/master/table) \(英文\)。 所涵蓋的案例包括**建立和刪除資料表，以及在資料表中插入和查詢實體**。

## <a name="create-an-azure-service-account"></a>建立 Azure 服務帳戶
[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶
[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

### <a name="create-an-azure-cosmos-db-account"></a>建立 Azure Cosmos DB 帳戶
[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="add-access-to-storage-or-azure-cosmos-db"></a>新增對儲存體或 Azure Cosmos DB 的存取權
若要使用「Azure 儲存體」或 Azure Cosmos DB，您必須下載並使用 Ruby Azure 套件，這包含一組能夠與「資料表 REST」服務進行通訊的便利程式庫。

### <a name="use-rubygems-to-obtain-the-package"></a>使用 RubyGems 來取得套件
1. 使用命令列介面，例如 **PowerShell** (Windows)、**Terminal** (Mac) 或 **Bash** (Unix)。
2. 在命令視窗中輸入 **gem install azure-storage-table** 以安裝 Gem 和相依性。

### <a name="import-the-package"></a>匯入封裝
使用您偏好的文字編輯器，將以下內容新增至您打算使用儲存體的 Ruby 檔案頂端：

```ruby
require "azure/storage/table"
```

## <a name="add-an-azure-storage-connection"></a>新增 Azure 儲存體連線
「Azure 儲存體」模組會讀取環境變數 **AZURE_STORAGE_ACCOUNT** 和 **AZURE_STORAGE_ACCESS_KEY**，以取得連線至「Azure 」儲存體帳戶所需的資訊。 如果未設定這些環境變數，您必須在使用 **Azure::Storage::Table::TableService** 之前，先使用下列程式碼來指定帳戶資訊：

```ruby
Azure.config.storage_account_name = "<your Azure Storage account>"
Azure.config.storage_access_key = "<your Azure Storage access key>"
```

若要從 Azure 入口網站中的傳統或 Resource Manager 儲存體帳戶取得這些值：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 瀏覽至您要使用的儲存體帳戶。
3. 在右邊的 [設定] 刀鋒視窗中，按一下 [存取金鑰]。
4. [存取金鑰] 刀鋒視窗隨即顯示，您會看到存取金鑰 1 和存取金鑰 2。 您可以使用其中一個存取金鑰。
5. 按一下複製圖示以將金鑰複製到剪貼簿。

## <a name="add-an-azure-cosmos-db-connection"></a>新增 Azure Cosmos DB 連線
若要連線至 Azure Cosmos DB，請從 Azure 入口網站複製您的主要連接字串，然後使用所複製的連接字串來建立 **Client** 物件。 您可以在建立 **TableService** 物件時傳遞 **Client** 物件：

```ruby
common_client = Azure::Storage::Common::Client.create(storage_account_name:'myaccount', storage_access_key:'mykey', storage_table_host:'mycosmosdb_endpoint')
table_client = Azure::Storage::Table::TableService.new(client: common_client)
```

## <a name="create-a-table"></a>建立資料表
**Azure::Storage::Table::TableService** 物件可讓您操作資料表和實體。 若要建立資料表，請使用 **create_table()** 方法。 下列範例將建立資料表或列印錯誤訊息 (若有的話)。

```ruby
azure_table_service = Azure::Storage::Table::TableService.new
begin
    azure_table_service.create_table("testtable")
rescue
    puts $!
end
```

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表
若要新增實體，請先建立定義實體屬性的雜湊物件。 請注意，對每個實體都必須指定 **PartitionKey** 及 **RowKey**。 這些是實體的唯一識別碼，且其值的查詢速度比其他屬性快上許多。 Azure 儲存體會使用 **PartitionKey** ，自動將資料表的實體分散在許多儲存體節點上。 具有相同 **PartitionKey** 的實體會儲存在相同節點上。 **RowKey** 是實體在其所屬資料分割內的唯一識別碼。

```ruby
entity = { "content" => "test entity",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.insert_entity("testtable", entity)
```

## <a name="update-an-entity"></a>更新實體
有多種方法可以用來更新現有的實體：

* **update_entity()：** 藉由取代現有實體來更新現有實體。
* **merge_entity()：** 藉由將新的屬性值合併到現有實體來更新現有實體。
* **insert_or_merge_entity()：** 藉由取代現有實體來更新現有實體。 如果實體不存在，將會插入新的實體：
* **insert_or_replace_entity()：** 藉由將新的屬性值合併到現有實體來更新現有實體。 如果實體不存在，將會插入新的實體。

下列範例示範使用 **update_entity()** 來更新實體：

```ruby
entity = { "content" => "test entity with updated content",
    :PartitionKey => "test-partition-key", :RowKey => "1" }
azure_table_service.update_entity("testtable", entity)
```

使用 **update_entity()** 和 **merge_entity()** 時，如果所要更新的實體不存在，更新作業就會失敗。 因此，如果您想要不論實體是否已經存在都儲存該實體，您應改用 **insert_or_replace_entity()** 或 **insert_or_merge_entity()**。

## <a name="work-with-groups-of-entities"></a>使用實體群組
有時候批次提交多個操作是有意義的，可以確保伺服器會進行不可部分完成的處理。 若要達到此目的，您必須先建立 **Batch** 物件，然後在 **TableService** 上使用 **execute_batch()** 方法。 下列範例示範在一個批次中，提交具備 RowKey 2 和 3 的兩個實體。 請注意，它僅適用於具備相同 PartitionKey 的實體。

```ruby
azure_table_service = Azure::TableService.new
batch = Azure::Storage::Table::Batch.new("testtable",
    "test-partition-key") do
    insert "2", { "content" => "new content 2" }
    insert "3", { "content" => "new content 3" }
end
results = azure_table_service.execute_batch(batch)
```

## <a name="query-for-an-entity"></a>查詢實體
若要查詢資料表中的實體，請使用 **get_entity()** 方法，藉由傳遞資料表名稱、**PartitionKey** 及 **RowKey** 來進行。

```ruby
result = azure_table_service.get_entity("testtable", "test-partition-key",
    "1")
```

## <a name="query-a-set-of-entities"></a>查詢實體集合
若要查詢資料表中的一組實體，請建立查詢雜湊物件，然後使用 **query_entities()** 方法。 下列範例示範取得具備相同 **PartitionKey**的所有實體：

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'" }
result, token = azure_table_service.query_entities("testtable", query)
```

> [!NOTE]
> 如果結果集太大，以致單一查詢無法傳回，則會傳回接續 Token，供您用來擷取後續的頁面。
>
>

## <a name="query-a-subset-of-entity-properties"></a>查詢實體屬性的子集
一項資料表查詢可以只擷取實體的少數屬性。 此技術稱為「投射」，可減少頻寬使用量並提高查詢效能 (尤其是針對大型實體)。 使用 select 子句並傳遞您要帶到用戶端的屬性名稱。

```ruby
query = { :filter => "PartitionKey eq 'test-partition-key'",
    :select => ["content"] }
result, token = azure_table_service.query_entities("testtable", query)
```

## <a name="delete-an-entity"></a>刪除實體
若要刪除實體，請使用 **delete_entity()** 方法。 請傳入包含實體、實體之 PartitionKey 及 RowKey 的資料表名稱。

```ruby
azure_table_service.delete_entity("testtable", "test-partition-key", "1")
```

## <a name="delete-a-table"></a>刪除資料表
若要刪除資料表，請使用 **delete_table()** 方法，然後傳入要刪除的資料表名稱。

```ruby
azure_table_service.delete_table("testtable")
```

## <a name="next-steps"></a>後續步驟

* [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。
* [Ruby 開發人員中心](https://azure.microsoft.com/develop/ruby/)
* [適用於 Ruby 的 Microsoft Azure 儲存體資料表用戶端程式庫](https://github.com/azure/azure-storage-ruby/tree/master/table) \(英文\) 

