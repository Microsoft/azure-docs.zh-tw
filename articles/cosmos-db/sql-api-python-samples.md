---
title: 適用於 Azure Cosmos DB 的 SQL API Python 範例 | Microsoft Docs
description: 在 GitHub 上找到適用於 Azure Cosmos DB 中一般工作 (包括 CRUD 作業) 的 Python 範例。
keywords: Python 範例
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: python
ms.topic: sample
ms.date: 03/14/2018
ms.author: sngun
ms.openlocfilehash: 2eb585876f91526633fb2844e67ba734648b38e5
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389527"
---
# <a name="azure-cosmos-db-python-examples"></a>Azure Cosmos DB Python 範例

> [!div class="op_single_selector"]
> * [.NET 範例](sql-api-dotnet-samples.md)
> * [Java 範例](sql-api-java-samples.md)
> * [非同步 Java 範例](sql-api-async-java-samples.md)
> * [Node.js 範例](sql-api-nodejs-samples.md)
> * [Python 範例](sql-api-python-samples.md)
> * [Azure 程式碼範例庫](https://azure.microsoft.com/resources/samples/?sort=0&service=cosmos-db)
> 
> 

[azure-documentdb-python](https://github.com/Azure/azure-documentdb-python) GitHub 存放庫中包含可對 Azure Cosmos DB 資源執行 CRUD 作業和其他常見作業的範例解決方案。 本文提供：

* 每個 Python 範例專案檔中各項工作的連結。 
* 相關 API 參考內容的連結。

**先決條件**

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

- 您可以 [啟用 Visual Studio 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)：您的 Visual Studio 訂用帳戶每個月都會提供額度，供您用在 Azure 付費服務。

[!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

您也需要 [Python SDK](sql-api-sdk-python.md)。 
   
   > [!NOTE]
   > 每個範例都各自獨立，會自己設定，並於完成後自行清理。 這些範例對 [document_client.CreateCollection](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient#createcollection) 發出多個呼叫。 每次執行時，您的訂用帳戶會計入一小時的使用費。 如需 Azure Cosmos DB 計費的詳細資訊，請參閱 [Azure Cosmos DB 價格](https://azure.microsoft.com/pricing/details/cosmos-db/)。
   > 
   > 

## <a name="database-examples"></a>資料庫範例
[DatabaseManagement](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DatabaseManagement) 專案的 [Program.py](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DatabaseManagement/Program.py) 檔案說明如何執行下列工作：

| Task | API 參考資料 |
| --- | --- |
| [建立資料庫](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L65-L76) |[document_client.CreateDatabase](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#createdatabase) |
| [依識別碼讀取資料庫](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L79-L96) |[document_client.ReadDatabase](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readdatabase) |
| [列出帳戶的資料庫](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L99-L110) |[document_client.ReadDatabases](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readdatabases) |
| [刪除資料庫](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L113-L126) |[document_client.DeleteDatabase](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#deletedatabase) |

## <a name="collection-examples"></a>集合範例
[CollectionManagement](https://github.com/Azure/azure-documentdb-python/blob/master/samples/CollectionManagement) 專案的 [Program.py](https://github.com/Azure/azure-documentdb-python/blob/master/samples/CollectionManagement/Program.py) 檔案說明如何執行下列工作：

| Task | API 參考資料 |
| --- | --- |
| [建立集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L84-L135) |[document_client.CreateCollection](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#createcollection) |
| [讀取資料庫中所有集合的清單](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L198-L209) |[document_client.ReadCollections](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readcollections) |
| [依識別碼取得集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L178-L195) |[document_client.ReadCollection](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readcollection) |
| [變更集合的輸送量](https://github.com/Azure/azure-documentdb-python/blob/master/samples/CollectionManagement/Program.py#L168-L172) | [document_client.ReplaceOffer](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#replaceoffer)|
| [刪除集合](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/CollectionManagement/Program.py#L212-L225) |[document_client.DeleteCollection](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#deletecollection) |

## <a name="document-examples"></a>文件範例
[DocumentManagement](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement) 專案的 [Program.py](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement/Program.py) 檔案說明如何執行下列工作：

| Task | API 參考資料 |
| --- | --- |
| [建立文件](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement/Program.py#L55-L66) |[document_client.CreateDocument](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#createdocument) |
| [建立文件的集合](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement/Program.py#L55-L66) |[document_client.CreateDocuments](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python) |
| [依識別碼讀取文件](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement/Program.py#L69-L78) |[document_client.ReadDocument](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readdocument) |
| [讀取集合中的所有文件](https://github.com/Azure/azure-documentdb-python/blob/master/samples/DocumentManagement/Program.py#L81-L92) |[document_client.ReadDocuments](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#readdocuments) |

## <a name="indexing-examples"></a>索引範例
[IndexManagement](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement) 專案的 [Program.py](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py) 檔案說明如何執行下列工作：

| Task | API 參考資料 |
| --- | --- |
| [使用手動 (而非自動) 索引](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L245-L246) |[IndexingPolicy.Automatic](https://docs.microsoft.com/python/api/pydocumentdb.documents?view=azure-python) |
| [從索引中排除指定的文件路徑](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L294-L367) |[IndexingPolicy.ExcludedPaths](https://docs.microsoft.com/python/api/pydocumentdb.documents.indexingdirective?view=azure-python) |
| [從索引中排除某個文件](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L204-L210) |[documents.IndexingDirective.Exclude](https://docs.microsoft.com/python/api/pydocumentdb.documents.indexingdirective?view=azure-python) |
| [設定編制索引模式](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L533) |[documents.IndexingMode](https://docs.microsoft.com/python/api/pydocumentdb.documents.indexingmode?view=azure-python) |
| [對字串使用範圍索引](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L440-L456) |[IndexingPolicy.IncludedPaths](https://docs.microsoft.com/python/api/pydocumentdb.documents.indexingdirective?view=azure-python) |
| [執行索引轉換](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L514-L559) |[document_client.ReplaceCollection](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#replacecollection) |

## <a name="query-examples"></a>查詢範例
範例專案也會示範如何執行下列查詢工作：

| Task | API 參考資料 |
| --- | --- |
| [查詢帳戶中的資料庫](https://github.com/Azure/azure-documentdb-python/blob/d78170214467e3ab71ace1a7400f5a7fa5a7b5b0/samples/DatabaseManagement/Program.py#L49-L62) |[document_client.QueryDatabases](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#querydatabases) |
| [查詢文件](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L152-L169) |[document_client.QueryDocuments](https://docs.microsoft.com/python/api/pydocumentdb.document_client.documentclient?view=azure-python#querydocuments) |
| [對雜湊索引路徑強制執行範圍掃描作業](https://github.com/Azure/azure-documentdb-python/blob/master/samples/IndexManagement/Program.py#L409-L415) |[http_constants.HttpHeaders.EnableScanInQuery](https://docs.microsoft.com/python/api/pydocumentdb.http_constants.httpheaders?view=azure-python#enablescaninquery) |
