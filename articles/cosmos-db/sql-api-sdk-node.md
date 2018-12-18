---
title: Azure Cosmos DB：SQL Node.js API、SDK 和資源 | Microsoft Docs
description: 了解所有 SQL Node.js API 和 SDK 相關資訊，包括發行日期、停用日期及 Azure Cosmos DB Node.js SDK 每個版本之間所做的變更。
services: cosmos-db
author: deborahc
editor: cgronlun
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: nodejs
ms.topic: reference
ms.date: 09/24/2018
ms.author: rnagpal
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b6983b49135b5a8adbef1d0cfc5a407cb7b0c7ac
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960544"
---
# <a name="azure-cosmos-db-nodejs-sdk-for-sql-api-release-notes-and-resources"></a>Azure Cosmos DB Node.js SDK for SQL API：版本資訊與資源
> [!div class="op_single_selector"]
> * [.NET](sql-api-sdk-dotnet.md)
> * [.NET 變更摘要](sql-api-sdk-dotnet-changefeed.md)
> * [.NET Core](sql-api-sdk-dotnet-core.md)
> * [Node.js](sql-api-sdk-node.md)
> * [非同步 Java](sql-api-sdk-async-java.md)
> * [Java](sql-api-sdk-java.md)
> * [Python](sql-api-sdk-python.md)
> * [REST](https://docs.microsoft.com/rest/api/cosmos-db/)
> * [REST 資源提供者](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/)
> * [SQL](https://msdn.microsoft.com/library/azure/dn782250.aspx)
> * [BulkExecutor - .NET](sql-api-sdk-bulk-executor-dot-net.md)
> * [BulkExecutor - Java](sql-api-sdk-bulk-executor-java.md)

|資源  |連結  |
|---------|---------|
|下載 SDK  |   [NPM](https://www.npmjs.com/package/@azure/cosmos) 
|API 文件  |  [JavaScript SDK 參考文件](https://docs.microsoft.com/javascript/api/%40azure/cosmos/?view=azure-node-latest)
|SDK 安裝指示  |  [安裝指示](https://github.com/Azure/azure-cosmos-js#installation)
|參與 SDK | [GitHub](https://github.com/Azure/azure-cosmos-js/tree/master)
| 範例 | [Node.js 程式碼範例](sql-api-nodejs-samples.md)
| 入門教學課程 | [開始使用 JavaScript SDK](sql-api-nodejs-get-started.md)
| Web 應用程式教學課程 | [使用 Azure Cosmos DB 來建置 Node.js Web 應用程式](sql-api-nodejs-application.md)
| 目前支援的平台 | [Node.js v6.x](https://nodejs.org/en/blog/release/v6.10.3/) - SDK 2.0.0 版和更新版本的必要項目。<br/>[Node.js v4.2.0](https://nodejs.org/en/blog/release/v4.2.0/)<br/> [Node.js v0.12](https://nodejs.org/en/blog/release/v0.12.0/)<br/> [Node.js v0.10](https://nodejs.org/en/blog/release/v0.10.0/) 

## <a name="release-notes"></a>版本資訊

### <a name="2.0.0"/>2.0.0</a>
* JavaScript SDK 2.0.0 版的公開推出
* 已新增對多重區域寫入的支援。

### <a name="2.0.0-3"/>2.0.0-3</a>
* JavaScript SDK 2.0.0 版的 RC1 (用於公開預覽)。
* 新的物件模型，具有最上層 CosmosClient，且方法分成相關的資料庫、容器和項目類別。 
* 支援 [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises)。 
* SDK 轉換成 TypeScript。

### <a name="1.14.4"/>1.14.4</a>
* 修正的 npm 文件。

### <a name="1.14.3"/>1.14.3</a>
* 已新增對連線問題預設重試次數的支援。
* 已新增對讀取集合變更摘要的支援。
* 已修正間歇造成「無法讀取工作階段」的工作階段一致性錯誤。
* 已新增對查詢度量資訊的支援。
* 已修正 http 代理程式的最大連接數目。

### <a name="1.14.2"/>1.14.2</a>
* 更新參考 Azure Cosmos DB 而非 Azure DocumentDB 的文件。
* 在 ConnectionPolicy 加入對 proxyUrl 設定的支援。

### <a name="1.14.1"/>1.14.1</a>
* 區分大小寫的檔案系統的次要修正。

### <a name="1.14.0"/>1.14.0</a>
* 新增工作階段一致性的支援。
* 此 SDK 版本需要使用從 https://aka.ms/cosmosdb-emulator 下載之最新版本的 Azure Cosmos DB 模擬器。

### <a name="1.13.0"/>1.13.0</a>
* 分割已經過證明的跨分割區查詢。
* 新增對資源連結開頭和尾端斜線 (和對應的測試) 的支援。

### <a name="1.12.2"/>1.12.2</a>
*   修正的 npm 文件。

### <a name="1.12.1"/>1.12.1</a>
* 已修正 executeStoredProcedure 中的錯誤，其中涉及的文件有特殊 Unicode 字元 (LS、PS)。
* 已修正在分割區索引鍵中使用 Unicode 字元處理文件的錯誤。
* 已修正使用名稱媒體建立集合的支援。 GitHub 問題 #114。
* 已修正權限授權權杖的支援。 GitHub 問題 #178。

### <a name="1.12.0"/>1.12.0</a>
* 新增對新[一致性層級](consistency-levels.md) ConsistentPrefix 的支援。
* 新增 UriFactory 的支援。
* 已修正 Unicode 支援錯誤。 GitHub 問題 #171。

### <a name="1.11.0"/>1.11.0</a>
* 新增彙總查詢的支援 (COUNT、MIN、MAX、SUM 和 AVG)。
* 新增控制跨分割區查詢平行處理程度的選項。
* 新增針對 Azure Cosmos DB 模擬器執行時停用 SSL 驗證的選項。
* 已將分割區集合的最小輸送量從 10,100 RU/s 降低為 2500 RU/s。
* 已修正單一資料分割集合的接續權杖錯誤。 GitHub 問題 #107。
* 已修正將 0 做為單一參數處理的 executeStoredProcedure 錯誤。 GitHub 問題 #155。

### <a name="1.10.2"/>1.10.2</a>
* 修正使用者-代理程式標頭以包含 SDK 版本。
* 次要程式碼清除。

### <a name="1.10.1"/>1.10.1</a>
* 當使用 SDK 以模擬器 (hostname=localhost) 為目標時，停用 SSL 驗證。
* 在預存程序執行期間，加入支援指令碼記錄功能。

### <a name="1.10.0"/>1.10.0</a>
* 新增對跨資料分割平行查詢的支援。
* 新增對已分割集合的 TOP/ORDER BY 查詢支援。

### <a name="1.9.0"/>1.9.0</a>
* 新加入已節流處理要求的重試原則支援。 (已節流處理的要求會收到要求率太大的例外狀況，即錯誤碼 429。)根據預設，發生錯誤碼 429 時，Azure Cosmos DB 會遵守回應標頭中的 retryAfter 時間，並針對每個要求重試九次。 如果您想要忽略伺服器在多次重試之間傳回的 retryAfter 時間，現在可以在 ConnectionPolicy 物件上的 RetryOptions 屬性中設定固定的重試間隔時間。 Azure Cosmos DB 現在會針對每個要進行節流處理的要求等候最多 30 秒 (不論重試計數為何)，並傳回包含錯誤碼 429 的回應。 您也可以在 ConnectionPolicy 物件上的 RetryOptions 屬性中覆寫該時間。
* Cosmos DB 現在會傳回 x-ms-throttle-retry-count 和 x-ms-throttle-retry-wait-time-ms 做為每個要求的回應標頭，其代表節流重試計數和要求歷經多次重試的累積時間。
* 新增 RetryOptions 類別，以及公開 ConnectionPolicy 類別上的 RetryOptions 屬性，使其能用來覆寫某些預設的重試選項。

### <a name="1.8.0"/>1.8.0</a>
* 新增對多重區域資料庫帳戶的支援。

### <a name="1.7.0"/>1.7.0</a>
* 新加入文件的存留時間 (TTL) 功能支援。

### <a name="1.6.0"/>1.6.0</a>
* 實作[已分割的集合](partition-data.md)和[使用者定義的效能等級](performance-levels.md)。

### <a name="1.5.6"/>1.5.6</a>
* 已修正 RangePartitionResolver.resolveForRead 錯誤，其因錯誤的結果 CONCAT 而無法傳回連結。

### <a name="1.5.5"/>1.5.5</a>
* 修正 hashParitionResolver resolveForRead()：所提供的資料分割索引鍵都未擲回例外狀況時，而不會傳回所有已註冊連結的清單。

### <a name="1.5.4"/>1.5.4</a>
* 修正問題 [#100](https://github.com/Azure/azure-documentdb-node/issues/100)- HTTPS 專用代理程式：避免基於 Azure Cosmos DB 用途而修改全域代理程式。 針對所有 lib 要求使用專用的代理程式。

### <a name="1.5.3"/>1.5.3</a>
* 修正問題 [#81](https://github.com/Azure/azure-documentdb-node/issues/81) - 正確處理媒體識別碼中的連字號。

### <a name="1.5.2"/>1.5.2</a>
* 修正問題 [#95](https://github.com/Azure/azure-documentdb-node/issues/95) - EventEmitter 接聽程式洩漏警告。

### <a name="1.5.1"/>1.5.1</a>
* 修正問題 [#92](https://github.com/Azure/azure-documentdb-node/issues/90) - 針對區分大小寫的系統，將資料夾 Hash 重新命名為 hash。

### <a name="1.5.0"/>1.5.0</a>
* 藉由新增雜湊和範圍分割解析程式來實作分區化支援。

### <a name="1.4.0"/>1.4.0</a>
* 實作 Upsert。 documentClient 上新的 upsertXXX 方法。

### <a name="1.3.0"/>1.3.0</a>
* 已略過以配合其他 SDK 的版本號碼。

### <a name="1.2.2"/>1.2.2</a>
* 將 Q Pomise 包裝函式分割至新的儲存機制。
* 更新至 npm 登錄的封裝檔案。

### <a name="1.2.1"/>1.2.1</a>
* 實作以識別碼為基礎的路由。
* 修正問題 [#49](https://github.com/Azure/azure-documentdb-node/issues/49) - 目前屬性與 current() 方法衝突。

### <a name="1.2.0"/>1.2.0</a>
* 新增對地理空間索引的支援。
* 驗證所有資源的識別碼屬性。 資源的識別碼不能包含 ?、/、#、&#47;&#47; 等字元，或在結尾處使用空格。
* 將新標頭「索引轉換進度」加至 ResourceResponse。

### <a name="1.1.0"/>1.1.0</a>
* 實作 V2 索引原則。

### <a name="1.0.3"/>1.0.3</a>
* 問題 [#40](https://github.com/Azure/azure-documentdb-node/issues/40) - 核心與 Promise SDK 中實作的 eslint 和 grunt 組態。

### <a name="1.0.2"/>1.0.2</a>
* 問題 [#45](https://github.com/Azure/azure-documentdb-node/issues/45) - Promise 包裝函式不包括標頭並發生錯誤。

### <a name="1.0.1"/>1.0.1</a>
* 已實作透過新增 readConflicts、readConflictAsync 及 queryConflicts 來查詢衝突的功能。
* 已更新 API 文件。
* 問題 [#41](https://github.com/Azure/azure-documentdb-node/issues/41) - client.createDocumentAsync 錯誤。

### <a name="1.0.0"/>1.0.0</a>
* GA SDK。

## <a name="release--retirement-dates"></a>發行和停用日期
Microsoft 至少會在停用 SDK 的 **12 個月** 之前提供通知，以供順利轉換至較新/支援的版本。

新的功能與最佳化項目只會新增至目前的 SDK，因此建議您一律盡早升級至最新的 SDK 版本。

服務會拒絕使用已停用 SDK 的任何 Cosmos DB 要求。

<br/>

| 版本 | 發行日期 | 停用日期 |
| --- | --- | --- |
| [2.0.0-3 (RC)](#2.0.0-3) |2018 年 8 月 2 日 |--- |
| [1.14.4](#1.14.4) |2018 年 5 月 3 日 |--- |
| [1.14.3](#1.14.3) |2018 年 5 月 3 日 |--- |
| [1.14.2](#1.14.2) |2017 年 12 月 21 日 |--- |
| [1.14.1](#1.14.1) |2017 年 11 月 10 日 |--- |
| [1.14.0](#1.14.0) |2017 年 11 月 9 日 |--- |
| [1.13.0](#1.13.0) |2017 年 10 月 11 日 |--- |
| [1.12.2](#1.12.2) |2017 年 8 月 10 日 |--- |
| [1.12.1](#1.12.1) |2017 年 8 月 10 日 |--- |
| [1.12.0](#1.12.0) |2017 年 5 月 10 日 |--- |
| [1.11.0](#1.11.0) |2017 年 3 月 16 日 |--- |
| [1.10.2](#1.10.2) |2017 年 1 月 27 日 |--- |
| [1.10.1](#1.10.1) |2016 年 12 月 22 日 |--- |
| [1.10.0](#1.10.0) |2016 年 10 月 3 日 |--- |
| [1.9.0](#1.9.0) |2016 年 7 月 7 日 |--- |
| [1.8.0](#1.8.0) |2016 年 6 月 14 日 |--- |
| [1.7.0](#1.7.0) |2016 年 4 月 26 日 |--- |
| [1.6.0](#1.6.0) |2016 年 3 月 29 日 |--- |
| [1.5.6](#1.5.6) |2016 年 3 月 8 日 |--- |
| [1.5.5](#1.5.5) |2016 年 2 月 2 日 |--- |
| [1.5.4](#1.5.4) |2016 年 2 月 1 日 |--- |
| [1.5.2](#1.5.2) |2016 年 1 月 26 日 |--- |
| [1.5.2](#1.5.2) |2016 年 1 月 22 日 |--- |
| [1.5.1](#1.5.1) |2016 年 1 月 4 日 |--- |
| [1.5.0](#1.5.0) |2015 年 12 月 31 日 |--- |
| [1.4.0](#1.4.0) |2015 年 10 月 6 日 |--- |
| [1.3.0](#1.3.0) |2015 年 10 月 6 日 |--- |
| [1.2.2](#1.2.2) |2015 年 9 月 10 日 |--- |
| [1.2.1](#1.2.1) |2015 年 8 月 15 日 |--- |
| [1.2.0](#1.2.0) |2015 年 8 月 5 日 |--- |
| [1.1.0](#1.1.0) |2015 年 7 月 9 日 |--- |
| [1.0.3](#1.0.3) |2015 年 6 月 4 日 |--- |
| [1.0.2](#1.0.2) |2015 年 5 月 23 日 |--- |
| [1.0.1](#1.0.1) |2015 年 5 月 15 日 |--- |
| [1.0.0](#1.0.0) |2015 年 4 月 8 日 |--- |

## <a name="faq"></a>常見問題集
[!INCLUDE [cosmos-db-sdk-faq](../../includes/cosmos-db-sdk-faq.md)]

## <a name="see-also"></a>另請參閱
若要深入了解 Cosmos DB，請參閱 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務頁面。

