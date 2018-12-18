---
title: 適用於 Azure Cosmos DB 的 Node.js 範例 | Microsoft Docs
description: 在 GitHub 上找到適用於 Azure Cosmos DB 中一般工作 (包括 CRUD 作業) 的 Node.js 範例。
services: cosmos-db
author: deborahc
editor: monicar
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: sample
ms.date: 09/24/2018
ms.author: deborahc
ms.openlocfilehash: 49aeeefde7fb340749e653ebdf07873f695caa0f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974794"
---
# <a name="azure-cosmos-db-nodejs-examples"></a>Azure Cosmos DB Node.js 範例
> [!div class="op_single_selector"]
> * [.NET 範例](sql-api-dotnet-samples.md)
> * [Java 範例](sql-api-java-samples.md)
> * [非同步 Java 範例](sql-api-async-java-samples.md)
> * [Node.js 範例](sql-api-nodejs-samples.md)
> * [Python 範例](sql-api-python-samples.md)
> * [Azure 程式碼範例庫](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

[azure-cosmos-js](https://github.com/Azure/azure-cosmos-js/tree/master/samples) GitHub 存放庫中包含可對 Azure Cosmos DB 資源執行 CRUD 作業和其他常見作業的範例解決方案。 本文提供：

* 每個 Node.js 範例專案檔中各項工作的連結。
* 相關 API 參考內容的連結。

**先決條件**

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

- 您可以 [啟用 Visual Studio 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)：您的 Visual Studio 訂用帳戶每個月都會提供額度，供您用在 Azure 付費服務。

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

您還需要 [JavaScript SDK](sql-api-sdk-node.md)。
   
   > [!NOTE]
   > 每個範例都各自獨立，自己設定，並自行清理。 據此，這些範例對 [Containers.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest#create) 發出多個呼叫。 每當執行此動作時，即會根據所建立容器的效能層，對訂用帳戶計入一小時的使用量費用。
   > 
   > 

## <a name="database-examples"></a>資料庫範例
[DatabaseManagement](https://github.com/Azure/azure-cosmos-js/tree/master/samples/DatabaseManagement) 專案的 [app.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/DatabaseManagement/app.js) 檔案說明如何執行下列工作。

| Task | API 參考資料 |
| --- | --- |
| [建立資料庫 (如果未存在)](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/DatabaseManagement/app.js#L35-L37) |[Databases.createIfNotExists](https://docs.microsoft.com/javascript/api/%40azure/cosmos/databases?view=azure-node-latest#createifnotexists) |
| [列出帳戶的資料庫](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/DatabaseManagement/app.js#L40-L42) |[Databases.readAll](https://docs.microsoft.com/javascript/api/%40azure/cosmos/databases?view=azure-node-latest#readall) |
| [依識別碼讀取資料庫](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/DatabaseManagement/app.js#L40-L42) |[Database.read](https://docs.microsoft.com/javascript/api/%40azure/cosmos/database?view=azure-node-latest#read) |
| [刪除資料庫](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/DatabaseManagement/app.js#L57-L60) |[Database.delete](https://docs.microsoft.com/javascript/api/%40azure/cosmos/database?view=azure-node-latest#delete) |

## <a name="container-examples"></a>容器範例
[ContainerManagement](https://github.com/Azure/azure-cosmos-js/tree/master/samples/ItemManagement) 專案的 [app.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ContainerManagement/app.js) 檔案說明如何執行下列工作。

| Task | API 參考資料 |
| --- | --- |
| [建立容器 (如果未存在)](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ContainerManagement/app.js#L36-L37) |[Containers.createIfNotExists](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest#createifnotexists) |
| [列出帳戶的容器](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ContainerManagement/app.js#L36-L37) |[Containers.readAll](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest#readall) |
| [依識別碼讀取集合](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ContainerManagement/app.js#L47-L51) |[Container.read](https://docs.microsoft.com/javascript/api/%40azure/cosmos/container?view=azure-node-latest#read) |
| [刪除容器](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ContainerManagement/app.js#L54-L55) |[Container.delete](https://docs.microsoft.com/javascript/api/%40azure/cosmos/container?view=azure-node-latest#delete) |

## <a name="item-examples"></a>項目範例
[ItemManagement](https://github.com/Azure/azure-cosmos-js/tree/master/samples/ItemManagement) 專案的 [app.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ItemManagement/app.js) 檔案說明如何執行下列工作。

| Task | API 參考資料 |
| --- | --- |
| [建立項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L49-L56) |[Items.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#create) |
| [讀取容器中的所有項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L59-L64) |[Items.readAll](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#readall) |
| [依識別碼讀取項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L59-L64) |[Item.read](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#read) |
| [在項目已變更時才讀取項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L73-L94) |[Item.read](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#read)<br/>[RequestOptions.accessCondition](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#accesscondition) |
| [查詢文件](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L97-L118) |[Items.query](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#query) |
| [取代項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L131-L136) |[Item.replace](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#replace) |
| [以條件式 ETag 檢查取代項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L139-L160) |[Item.replace](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#replace)<br/>[RequestOptions.accessCondition](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#accesscondition) |
| [刪除項目](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ItemManagement/app.js#L162-L164) |[Item.delete](https://docs.microsoft.com/javascript/api/%40azure/cosmos/item?view=azure-node-latest#delete) |

## <a name="indexing-examples"></a>索引範例
[IndexManagement](https://github.com/Azure/azure-cosmos-js/tree/master/samples/IndexManagement) 專案的 [app.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/IndexManagement/app.js) 檔案說明如何執行下列工作。

| Task | API 參考資料 |
| --- | --- |
| [手動編製特定文件的索引](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L135-L177) |[RequestOptions.indexingDirective: 'include'](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#indexingdirective) |
| [手動從索引中排除特定文件](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L90-L131) |[RequestOptions.indexingDirective: 'exclude'](https://docs.microsoft.com/javascript/api/%40azure/cosmos/requestoptions?view=azure-node-latest#indexingdirective) |
| [為大量匯入使用延遲索引或讀取重度容器](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L183-L214) |[IndexingMode.Lazy](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexingmode?view=azure-node-latest) |
| [從索引中排除某個路徑](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L352-L429) |[IndexingPolicy.ExcludedPath](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexingpolicy?view=azure-node-latest#excludedpaths) |
| [允許在範圍作業期間掃描字串路徑](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L219-L275) |[FeedOptions.EnableScanInQuery](https://docs.microsoft.com/javascript/api/%40azure/cosmos/feedoptions?view=azure-node-latest#enablescaninquery) |
| [建立字串路徑的範圍索引](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L281-L346) |[IndexKind.Range](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexkind?view=azure-node-latest)、[IndexingPolicy](https://docs.microsoft.com/javascript/api/%40azure/cosmos/indexingpolicy?view=azure-node-latest)、[Items.query](https://docs.microsoft.com/javascript/api/%40azure/cosmos/items?view=azure-node-latest#query) |
| [使用預設 indexPolicy 建立容器，然後在線上加以更新](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/IndexManagement/app.js#L435-L507) |[Containers.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/containers?view=azure-node-latest#create)


如需索引編製的詳細資訊，請參閱 [Azure Cosmos DB 編製索引原則](indexing-policies.md)。

## <a name="server-side-programming-examples"></a>伺服器端程式設計範例
[ServerSideScripts](https://github.com/Azure/azure-cosmos-js/tree/master/samples/ServerSideScripts) 專案的 [app.js](https://github.com/Azure/azure-cosmos-js/blob/master/samples/ServerSideScripts/app.js) 檔案說明如何執行下列工作。

| Task | API 參考資料 |
| --- | --- |
| [建立預存程序](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ServerSideScripts/JS/upsert.js#L12-L72) |[StoredProcedures.create](https://docs.microsoft.com/javascript/api/%40azure/cosmos/storedprocedures?view=azure-node-latest) |
| [執行預存程序](https://github.com/Azure/azure-cosmos-js/blob/216672a679ab389e5b341280eeacab1cab3691e4/samples/ServerSideScripts/app.js#L44) |[StoredProcedure.execute](https://docs.microsoft.com/javascript/api/%40azure/cosmos/storedprocedure?view=azure-node-latest#execute) |

如需伺服器端程式設計的詳細資訊，請參閱 [Azure Cosmos DB 伺服器端程式設計：預存程序、資料庫觸發程序和 UDF](programming.md)。

