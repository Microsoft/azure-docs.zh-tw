---
title: 以 Python 開始使用 Azure 表格儲存體和 Azure Cosmos DB 資料表 API | Microsoft Docs
description: 使用 Azure 表格儲存體或 Azure Cosmos DB 資料表 API 將結構化資料儲存在雲端。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-table
ms.devlang: python
ms.topic: sample
ms.date: 04/05/2018
ms.author: sngun
ms.openlocfilehash: 4e9d1742401e30d451282ea8dc22a56c0347dbf9
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "41918593"
---
# <a name="get-started-with-azure-table-storage-and-the-azure-cosmos-db-table-api-using-python"></a>以 Python 開始使用 Azure 表格儲存體和 Azure Cosmos DB 資料表 API

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

Azure 表格儲存體和 Azure Cosmos DB 是可將結構化的 NoSQL 資料儲存在雲端中的服務，並提供具有無結構描述設計的索引鍵/屬性存放區。 由於表格儲存體和 Azure Cosmos DB 並無結構描述，因此可輕易隨著應用程式發展需求改寫資料。 相較於類似資料量的傳統 SQL，對許多類型的應用程式而言，表格儲存體和資料表 API 資料可快速存取且符合成本效益，通常可降低成本。

您可以使用表格儲存體或 Azure Cosmos DB 來儲存具彈性的資料集，例如 Web 應用程式的使用者資料、通訊錄、裝置資訊，以及服務所需的其他中繼資料類型。 您可以在資料表中儲存任意數目的實體，且儲存體帳戶可包含任意數目的資料表，最高可達儲存體帳戶的容量限制。

### <a name="about-this-sample"></a>關於此範例
此範例示範如何在常見的 Azure 表格儲存體案例中使用[適用於 Python 的 Azure Cosmos DB 表格 SDK](https://pypi.python.org/pypi/azure-cosmosdb-table/) \(英文\)。 SDK 名稱表示其適用於 Azure Cosmos DB，但也同時適用於 Azure Cosmos DB 和 Azure 表格儲存體，兩個服務皆具有唯一的端點。 在這些案例探索中會使用 Python 範例，可說明如何執行下列動作：
* 建立和刪除表格
* 插入和查詢實體
* 修改實體

在進行此範例中的案例時，您可以參閱 [Azure Cosmos DB SDK for Python API 參考資料](https://azure.github.io/azure-cosmosdb-python/) \(英文\)。

## <a name="prerequisites"></a>必要條件

您需要下列項目才能成功完成此範例︰

- [Python](https://www.python.org/downloads/) 2.7、3.3、3.4、3.5 或 3.6
- [適用於 Python 的 Azure Cosmos DB 資料表 SDK ](https://pypi.python.org/pypi/azure-cosmosdb-table/)。 此 SDK 與 Azure 表格儲存體和 Azure Cosmos DB 資料表 API 連線。
- [Azure 儲存體帳戶](../storage/common/storage-quickstart-create-account.md)或 [Azure Cosmos DB 帳戶](https://azure.microsoft.com/try/cosmosdb/)

## <a name="create-an-azure-service-account"></a>建立 Azure 服務帳戶
[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

### <a name="create-an-azure-storage-account"></a>建立 Azure 儲存體帳戶
[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

### <a name="create-an-azure-cosmos-db-table-api-account"></a>建立 Azure Cosmos DB 表格 API 帳戶
[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="install-the-azure-cosmos-db-table-sdk-for-python"></a>安裝適用於 Python 的 Azure Cosmos DB 資料表 SDK

儲存體帳戶已建立後，下一個步驟就是安裝[適用於 Python 的 Microsoft Azure Cosmos DB 資料表 SDK](https://pypi.python.org/pypi/azure-cosmosdb-table/) \(英文\)。 如需安裝 SDK 的詳細資料，請參閱 GitHub 上 [Cosmos DB Table SDK for Python] 存放庫中的 [README.rst](https://github.com/Azure/azure-cosmosdb-python/blob/master/azure-cosmosdb-table/README.rst) 檔案。

## <a name="import-the-tableservice-and-entity-classes"></a>匯入 TableService 和實體類別

若要在 Python 的 Azure 表格服務中使用實體，可以使用 [TableService](https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html) 和[實體][ py_Entity]類別。 在靠近您 Python 檔案的頂端處新增此程式碼，以匯入這兩者：

```python
from azure.cosmosdb.table.tableservice import TableService
from azure.cosmosdb.table.models import Entity
```

## <a name="connect-to-azure-table-service"></a>連線到 Azure 表格服務

若要連線到 Azure 儲存體表格服務，請建立 [TableService](https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html) 物件，並傳入您的儲存體帳戶名稱和帳戶金鑰。 以您的帳戶名稱和金鑰取代 `myaccount` 和 `mykey`。

```python
table_service = TableService(account_name='myaccount', account_key='mykey')
```

## <a name="connect-to-azure-cosmos-db"></a>連線至 Azure Cosmos DB

若要連線到 Azure Cosmos DB，從 Azure 入口網站複製您的主要連接字串，並使用複製的連接字串建立 [TableService](https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html) 物件：

```python
table_service = TableService(connection_string='DefaultEndpointsProtocol=https;AccountName=myaccount;AccountKey=mykey;TableEndpoint=myendpoint;)
```

## <a name="create-a-table"></a>建立資料表

呼叫 [create_table] [ py_create_table]以建立資料表。

```python
table_service.create_table('tasktable')
```

## <a name="add-an-entity-to-a-table"></a>將實體新增至資料表

若要新增實體，首先建立一個代表您實體的物件，然後將物件傳遞至 [TableService.insert_entity 方法][py_TableService]。 實體物件可以是字典或類型為 [Entity][py_Entity] 的物件，並定義您實體的屬性名稱與值。 除了您為實體定義的任何其他屬性外。每個實體必須包含必要的 [PartitionKey and RowKey](#partitionkey-and-rowkey) 屬性。

此範例會建立一個代表實體的字典物件，然後將該物件傳遞至 [insert_entity][py_insert_entity] 方法，以將它新增至資料表：

```python
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the trash', 'priority' : 200}
table_service.insert_entity('tasktable', task)
```

此範例會建立一個代表[實體][py_Entity]的物件，然後將該物件傳遞至 [insert_entity][py_insert_entity] 方法，以將它新增至資料表：

```python
task = Entity()
task.PartitionKey = 'tasksSeattle'
task.RowKey = '002'
task.description = 'Wash the car'
task.priority = 100
table_service.insert_entity('tasktable', task)
```

### <a name="partitionkey-and-rowkey"></a>PartitionKey 和 RowKey

您必須為每個實體指定 **PartitionKey** 與 **RowKey** 屬性。 這些屬性是您實體的唯一識別碼，共同形成實體的主要索引鍵。 使用這些值查詢的速度遠快於使用任何其他實體屬性查詢，因為系統只會將這些屬性編製索引。

表格服務會使用 **PartitionKey**，以智慧化方式在儲存體節點之間分散資料表實體。 具有相同 **PartitionKey** 的實體會儲存在相同的節點上。 **RowKey** 是實體在其所屬資料分割內的唯一識別碼。

## <a name="update-an-entity"></a>更新實體

若要更新實體的所有屬性值，請呼叫 [update_entity][py_update_entity] 方法。 此範例說明如何以實體的更新版本取代現有的實體：

```python
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the garbage', 'priority' : 250}
table_service.update_entity('tasktable', task)
```

如果正在更新的實體已不存在，則更新操作便會失敗。 如果您想要儲存實體，無論其是否存在，請使用 [insert_or_replace_entity][py_insert_or_replace_entity]。 在下列範例中，第一個呼叫將取代現有實體。 第二個呼叫將插入新實體，因為資料表中沒有具有指定 PartitionKey 和 RowKey 的實體存在。

```python
# Replace the entity created earlier
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '001', 'description' : 'Take out the garbage again', 'priority' : 250}
table_service.insert_or_replace_entity('tasktable', task)

# Insert a new entity
task = {'PartitionKey': 'tasksSeattle', 'RowKey': '003', 'description' : 'Buy detergent', 'priority' : 300}
table_service.insert_or_replace_entity('tasktable', task)
```

> [!TIP]
> [update_entity][py_update_entity] 方法會取代現有實體其所有的屬性與值，您也可以用於移除現有實體的屬性。 您可以使用 [merge_entity][py_merge_entity] 方法以新或已修改的屬性值來更新現有的實體，而不完全取代該實體。

## <a name="modify-multiple-entities"></a>修改多個實體

若要確保表格服務對要求的處理為不可部分完成，您可以在一個批次中一起提交多個作業。 首先，使用 [TableBatch][py_TableBatch] 類別將多個作業新增至單一批次。 接著，呼叫 [TableService][py_TableService].[commit_batch][py_commit_batch] 以不可部分完成的作業提交這些作業。 要以批次修改的所有文件必須在同一個分割中。

此範例會在一個批次中同時新增兩個實體：

```python
from azure.cosmosdb.table.tablebatch import TableBatch
batch = TableBatch()
task004 = {'PartitionKey': 'tasksSeattle', 'RowKey': '004', 'description' : 'Go grocery shopping', 'priority' : 400}
task005 = {'PartitionKey': 'tasksSeattle', 'RowKey': '005', 'description' : 'Clean the bathroom', 'priority' : 100}
batch.insert_entity(task004)
batch.insert_entity(task005)
table_service.commit_batch('tasktable', batch)
```

批次也可以與內容管理員語法搭配使用：

```python
task006 = {'PartitionKey': 'tasksSeattle', 'RowKey': '006', 'description' : 'Go grocery shopping', 'priority' : 400}
task007 = {'PartitionKey': 'tasksSeattle', 'RowKey': '007', 'description' : 'Clean the bathroom', 'priority' : 100}

with table_service.batch('tasktable') as batch:
    batch.insert_entity(task006)
    batch.insert_entity(task007)
```

## <a name="query-for-an-entity"></a>查詢實體

若要查詢資料表中的實體，請將其 PartitionKey 與 RowKey 傳遞至 [TableService][py_TableService].[get_entity][py_get_entity] 方法。

```python
task = table_service.get_entity('tasktable', 'tasksSeattle', '001')
print(task.description)
print(task.priority)
```

## <a name="query-a-set-of-entities"></a>查詢實體集合

您可以提供一個含 **filter** 參數的篩選字串，來查詢一組實體。 此範例會對 PartitionKey 套用篩選條件，以尋找西雅圖中的所有工作：

```python
tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'")
for task in tasks:
    print(task.description)
    print(task.priority)
```

## <a name="query-a-subset-of-entity-properties"></a>查詢實體屬性的子集

您也可以限制會為查詢中的每個實體傳回哪些屬性。 這項稱為「投射」的技術可減少頻寬並提高查詢效能 (尤其是對大型實體或結果集而言)。 使用 **select** 參數並傳遞您要傳回到用戶端的屬性名稱。

下列程式碼中的查詢只會傳回資料表中各實體的說明。

> [!NOTE]
> 下列程式碼片段只針對 Azure 儲存體運作。 儲存體模擬器並不支援此程式碼片段。

```python
tasks = table_service.query_entities('tasktable', filter="PartitionKey eq 'tasksSeattle'", select='description')
for task in tasks:
    print(task.description)
```

## <a name="delete-an-entity"></a>刪除實體

透過將實體的 **PartitionKey** 與 **RowKey** 傳遞至 [delete_entity][py_delete_entity] 方法來刪除實體。

```python
table_service.delete_entity('tasktable', 'tasksSeattle', '001')
```

## <a name="delete-a-table"></a>刪除資料表

如果您不再需要資料表及其內的任何實體，請呼叫 [delete_table][py_delete_table] 方法將資料表永久地從 Azure 儲存體中刪除。

```python
table_service.delete_table('tasktable')
```

## <a name="next-steps"></a>後續步驟

* [常見問題集 - 利用資料表 API 進行開發](https://docs.microsoft.com/azure/cosmos-db/faq#develop-with-the-table-api)
* [Azure Cosmos DB SDK for Python API 參考資料](https://azure.github.io/azure-cosmosdb-python/) \(英文\)
* [Python 開發人員中心](https://azure.microsoft.com/develop/python/)
* [Microsoft Azure 儲存體總管](../vs-azure-tools-storage-manage-with-storage-explorer.md)：一個免費、跨平台的應用程式，以視覺化方式在 Windows、macOS 和 Linux 上使用 Azure 儲存體資料。
* [在 Visual Studio 中使用 Python (Windows)](https://docs.microsoft.com/visualstudio/python/overview-of-python-tools-for-visual-studio)

[py_commit_batch]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_create_table]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_delete_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_delete_table]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_Entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.models.html
[py_get_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_insert_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_insert_or_replace_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_TableService]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_TableBatch]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tablebatch.html
[py_merge_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
[py_update_entity]: https://azure.github.io/azure-cosmosdb-python/ref/azure.cosmosdb.table.tableservice.html
