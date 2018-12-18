---
title: Azure Cosmos DB：SQL API 簡介 | Microsoft Docs
description: 了解如何透過 SQL 和 JavaScript，使用 Azure Cosmos DB 來儲存及查詢大量 JSON 文件 (低延遲)。
keywords: JSON 資料庫，文件資料庫
services: cosmos-db
author: rafats
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: na
ms.topic: overview
ms.date: 05/22/2017
ms.author: rafats
ms.openlocfilehash: fdeb58d72e15f563fd70ae94804de0773dd603c7
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387340"
---
# <a name="introduction-to-azure-cosmos-db-sql-api"></a>Azure Cosmos DB 簡介：SQL API

[Azure Cosmos DB](introduction.md) 是 Microsoft 全域發佈的多模型資料庫服務，適用於任務關鍵性應用程式。 Azure Cosmos DB 提供[一站式全域散發](distribute-data-globally.md)、全球[彈性調整的輸送量和儲存體](partition-data.md)、達到第 99 個百分位數的個位數毫秒延遲、[五個定義完善的一致性層級](consistency-levels.md)，以及保證的高可用性，全部都由[領先業界的 SLA (英文)](https://azure.microsoft.com/support/legal/sla/cosmos-db/) 所支援。 Azure Cosmos DB 會[自動編製資料的索引](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)，您不需要處理結構描述和索引管理。 它是多重模型，支援文件、索引鍵/值、圖表和單欄式資料模型。

![Azure SQL API](./media/sql-api-introduction/cosmosdb-sql-api.png) 

透過 SQL API，Azure Cosmos DB 就可利用無結構描述的 JSON 資料來提供豐富且熟悉的 [SQL 查詢功能](sql-api-sql-query.md) (一致低延遲)。 在本文中，我們概略說明 Azure Cosmos DB 的 SQL API，以及如何使用它來儲存大量的 JSON 資料、在毫秒延遲等級內查詢它們，並輕鬆發展結構描述。 

## <a name="what-capabilities-and-key-features-does-azure-cosmos-db-offer"></a>Azure Cosmos DB 提供哪些功能和主要功能？
Azure Cosmos DB 可透過 SQL API 提供下列重要功能和優點：

* **可彈性調整的輸送量和儲存體：** 輕鬆相應增加或相應減少 JSON 資料庫，以符合您應用程式的需求。 您的資料會儲存在固態硬碟 (SSD) 中以便獲得可預測的低延遲。 Azure Cosmos DB 支援用來儲存稱為集合之 JSON 資料的容器，這些集合可調整為幾乎沒有限制的儲存體大小和佈建的輸送量。 隨著應用程式的成長，您可以依據可預測的效能彈性且順暢地調整 Azure Cosmos DB。 


* **多重區域複寫**︰Azure Cosmos DB 會自動將您的資料複寫至與您 Azure Cosmos DB 帳戶相關聯的所有區域，讓您開發需要全域存取資料的應用程式，同時能在一致性、可用性與效能之間做出取捨，且全都有相對應的保證。 Azure Cosmos DB 利用多重定址 API 提供自動的區域性容錯移轉，還能夠彈性調整世界各地的輸送量和儲存體。 深入了解[使用 Azure Cosmos DB 全域發佈資料](distribute-data-globally.md)。

* **運用常見的 SQL 語法進行特定查詢：** 儲存異質 JSON 文件，並透過常見的 SQL 語法來查詢這些文件。 Azure Cosmos DB 採用高度並行、無鎖定、記錄結構化索引技術，自動編製所有文件內容的索引。 這樣一來，不需要指定結構描述提示、次要索引或檢視，就能進行大量且即時的查詢。 深入了解[查詢 Azure Cosmos DB](sql-api-sql-query.md)。 
* **在資料庫內執行 JavaScript：** 使用標準 JavaScript，以預存程序、觸發程序和使用者定義函式 (UDF) 的形式表示應用程式邏輯。 這允許您的應用程式邏輯操作 JSON 資料，而不需要擔心應用程式與資料庫結構描述之間的不相符。 SQL API 可讓您直接在資料庫引擎內以完整的交易方式執行 JavaScript 應用程式邏輯。 JavaScript 的深入整合則可讓您以獨立交易的形式，從 JavaScript 程式內執行 INSERT、REPLACE、DELETE 和 SELECT 作業。 在 [SQL 伺服器端程式設計](programming.md)中深入了解。

* **可調整的一致性等級：** 有五個定義完善的一致性等級可選擇，讓您在一致性與效能之間做出最好的取捨。 針對查詢和讀取作業，Azure Cosmos DB 提供五個不同的一致性等級：強式、限定過期、工作階段、一致的前置和最終。 這些細微且定義完善的一致性層級可讓您在一致性、可用性與延遲三者間做出合理取捨。 深入了解[使用一致性層級將可用性和效能最大化](consistency-levels.md)。

* **完全受控：** 消除資料庫和電腦資源的管理需求。 因為是完全受控的 Microsoft Azure 服務，您不需要管理虛擬機器、部署和設定軟體、管理調整事宜，或處理複雜的資料層升級。 每個資料庫都會自動進行備份，防範區域性失敗。 您可以輕鬆地新增 Azure Cosmos DB 帳戶，並在需要時佈建容量，將精力投注在應用程式，不用浪費時間來操作和管理資料庫。 

* **開放式設計：** 使用現有技能和工具讓您快速上手。 針對 SQL API 進行程式設計很簡單、容易達成，且不需要採用新工具或符合 JSON 或 JavaScript 的自訂擴充功能。 您可以透過簡單的 RESTful HTTP 介面來存取所有資料庫功能，包括 CRUD、查詢和 JavaScript 處理。 SQL API 既採用現有的格式、語言和標準，同時又能提供凌駕於它們之上的高價值資料庫功能。

* **自動編製索引**：根據預設，Azure Cosmos DB 會自動為資料庫中的所有文件編制索引，且不預期或需要任何結構描述或建立次要索引。 不想要編製所有項目的索引嗎？ 別擔心，您也可以 [選擇退出 JSON 檔案中的路徑](indexing-policies.md) 。

* **變更摘要支援：** 變更摘要可提供一份 Azure Cosmos DB 集合內的文件排序清單，並以文件的修改順序排序。 此摘要可用於接聽資料修改，以便複寫資料、觸發 API 呼叫或執行更新的串流處理。 變更摘要會自動啟用且容易使用：[深入了解變更摘要](https://docs.microsoft.com/azure/cosmos-db/change-feed)。 

## <a name="data-management"></a>您要如何使用 SQL API 來管理資料？
SQL API 透過定義完善的資料庫資源來協助管理 JSON 資料。 這些資源會進行複寫來達到高可用性，並且會透過其邏輯 URI 來進行唯一定址。 SQL API 針對所有資源提供簡單的 HTTP 式 RESTful 程式設計模型。 


Azure Cosmos DB 資料庫帳戶是可讓您存取 Azure Cosmos DB 的唯一命名空間。 在建立資料庫帳戶之前，您必須先擁有 Azure 訂用帳戶，此帳戶可讓您存取各種 Azure 服務。 

Azure Cosmos DB 內的所有資源都會加以建立模型，並儲存為 JSON 文件。 管理資源時，是以項目 (含有中繼資料的 JSON 文件) 和摘要 (即項目集合) 的形式來管理。 項目集包含在其各自的摘要內。

下圖顯示 Azure Cosmos DB 資源之間的關係：

![Azure Cosmos DB 中資源之間的階層式關聯性][1] 

資料庫帳戶是由一組資料庫所組成，每個資料庫都包含多個集合，而集合可包含預存程序、觸發程序、UDF、文件和相關附件。 資料庫也有相關聯的使用者，每個使用者都有一組可存取其他各種集合、預存程序、觸發程序、UDF、文件或附件的權限。 資料庫、使用者、權限和集合是系統所定義、具有已知結構描述的資源，而文件、預存程序、觸發程序、UDF 和附件則包含使用者定義的任意 JSON 內容。  

## <a name="develop"></a> 如何使用 SQL API 開發應用程式？

Azure Cosmos DB 公開資源的方式是透過 REST API，此 API 可供任何能發出 HTTP/HTTPS 要求的語言呼叫。 另外，SQL API 還提供了幾種熱門語言的程式設計程式庫。 用戶端程式庫可透過處理諸如位址快取、例外狀況管理、自動重試等詳細資料，來簡化使用 API 的各個層面。 程式庫目前適用於下列語言和平台：  

| 下載 | 文件 |
| --- | --- |
| [.NET SDK](http://go.microsoft.com/fwlink/?LinkID=402989) |[.NET 參考文件](/dotnet/api/overview/azure/cosmosdb?view=azure-dotnet) |
| [Java SDK](http://go.microsoft.com/fwlink/?LinkID=402380) |[.NET 參考文件](/java/api/com.microsoft.azure.documentdb) |
| [JavaScript SDK](https://www.npmjs.com/package/@azure/cosmos) |[JavaScript 參考文件](https://docs.microsoft.com/javascript/api/@azure/cosmos/?view=azure-node-latest) |
| [Python SDK](https://pypi.python.org/pypi/pydocumentdb) |[Python 參考文件](https://github.com/Azure/azure-cosmos-python) |

您可以使用 [Azure Cosmos DB 模擬器](local-emulator.md)，透過 SQL API 在本機開發及測試應用程式，不需建立 Azure 訂用帳戶，也不會產生任何費用。 如果您滿意應用程式在模擬器中的運作方式，就可以切換成使用雲端的 Azure Cosmos DB 帳戶。

除了基本的建立、讀取、更新和刪除作業之外，SQL API 還提供可用來擷取 JSON 文件的豐富 SQL 查詢介面，並在伺服器端支援以交易方式執行 JavaScript 應用程式邏輯。 查詢和指令碼執行介面可以透過所有平台程式庫以及 REST API 來取得。 

### <a name="sql-query"></a>SQL 查詢
Azure Cosmos DB 支援使用根植於 JavaScript 類型系統的 SQL 語言及支援關聯式、階層式和空間查詢的運算式來查詢文件。 Azure Cosmos DB 查詢語言是可用來查詢 JSON 文件的簡單卻功能強大的介面。 此語言支援 ANSI SQL 文法的子集，並新增 JavaScript 物件、陣列、物件建構和函式叫用的深入整合。 SQL API 提供其查詢模型，而沒有來自開發人員的任何明確結構描述或編製索引提示。

使用者定義函式 (UDF) 可以向 SQL API 進行註冊，然後在 SQL 查詢中供參考，從而延伸文法來支援自訂應用程式邏輯。 這些 UDF 是以 JavaScript 程式的形式撰寫，並在資料庫內執行。 

對於 .NET 開發人員，SQL API [.NET SDK](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.linq.aspx) 也提供 LINQ 查詢提供者。 

### <a name="transactions-and-javascript-execution"></a>交易和 JavaScript 執行
SQL API 可讓您將應用程式邏輯撰寫成完全以 JavaScript 撰寫的具名程式。 這些程式會針對集合進行註冊，而可對指定之集合內的文件進行資料庫操作。 JavaScript 可以註冊成觸發程序、預存程序或使用者定義函式 (UDF) 來供執行。 觸發程序和預存程序可以建立、讀取、更新和刪除文件，而使用者定義函式則可在查詢執行邏輯中執行，而不需要對集合進行寫入存取。

在 Cosmos DB 內執行 JavaScript 的作法是仿造自關聯式資料庫系統所支援的概念，以 JavaScript 做為 Transact-SQL 的新式取代項目。 所有 JavaScript 邏輯都是以隔離的快照在環境 ACID 交易內執行。 在執行期間，如果 JavaScript 擲回例外狀況，則會中止整個交易。

## <a name="next-steps"></a>後續步驟
已經有 Azure 帳戶嗎？ 接著，您可以依照我們的[快速入門](../cosmos-db/create-sql-api-dotnet.md)開始使用 Azure Cosmos DB，這會逐步引導您建立帳戶及開始使用 Cosmos DB。

[1]: ./media/sql-api-introduction/json-database-resources1.png

