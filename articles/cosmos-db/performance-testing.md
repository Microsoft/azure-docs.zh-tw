---
title: Azure Cosmos DB 規模和效能測試 | Microsoft Docs
description: 了解使用 Azure Cosmos DB 來執行規模和效能測試
keywords: 效能測試
services: cosmos-db
author: SnehaGunda
manager: kfile
editor: ''
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 08/29/2017
ms.author: sngun
ms.openlocfilehash: ce2c0ddcce3813990bf819477f7db425d70e3595
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34612294"
---
# <a name="performance-and-scale-testing-with-azure-cosmos-db"></a>Azure Cosmos DB 的效能和規模測試

效能和規模測試是應用程式開發過程中的關鍵步驟。 對許多應用程式而言，資料庫層在整體效能和延展性上有很大的影響。 因此，它是效能測試的重要元素。 [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 就是專門為彈性化規模和可預測效能而建置。 這些功能完全符合應用程式需要高效能資料庫層的要求。 

本文可讓開發人員在為他們的 Azure Cosmos DB 工作負載實作效能測試套件時作為參考。 也可以用來針對高效能應用程式案例評估 Azure Cosmos DB。 主要是著重在隔離的資料庫效能測試，但也包含適用於實際執行應用程式的最佳做法。

閱讀本文後，您將能夠回答下列問題： 

* 哪裡可以找到可供進行 Azure Cosmos DB 效能測試的範例 .NET 用戶端應用程式？ 
* 如何藉由 Azure Cosmos DB 從我的用戶端應用程式達到高輸送量層級？

若要開始使用程式碼，請從 [Azure Cosmos DB 效能測試範例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)下載專案。 

> [!NOTE]
> 此應用程式的目標，在於示範如何透過少數用戶端電腦發揮 Azure Cosmos DB 最佳效能。 此範例的目標不是為了達到 Azure Cosmos DB 的尖峰輸送量容量 (可以無限制調整)。
> 
> 

如果您要尋找用戶端設定選項，以改善 Azure Cosmos DB 效能，請參閱 [Azure Cosmos DB 效能祕訣](performance-tips.md)。

## <a name="run-the-performance-testing-application"></a>執行效能測試應用程式
若要開始使用，最快的方法就是依以下步驟所述，編譯並執行 .NET 範例。 您也可以檢閱原始程式碼，然後對自己的用戶端應用程式實作類似的組態。

**步驟 1：** 從 [Azure Cosmos DB 效能測試範例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark) 或 GitHub 存放庫的分支下載專案。

**步驟 2：** 修改 App.config 中 EndpointUrl、AuthorizationKey、CollectionThroughput 及 DocumentTemplate (選擇性) 的設定。

> [!NOTE]
> 以高輸送量佈建集合之前，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/cosmos-db/)以估算每個集合的成本。 Azure Cosmos DB 會以小時為單位，對儲存體和輸送量分別計價。 測試完成後，您可以藉由刪除或降低 Azure Cosmos DB 集合的輸送量來節省成本。
> 
> 

**步驟 3：** 從命令列編譯並執行主控台應用程式。 您應該會看到如以下的輸出：

    C:\Users\cosmosdb\Desktop\Benchmark>DocumentDBBenchmark.exe
    Summary:
    ---------------------------------------------------------------------
    Endpoint: https://arramacquerymetrics.documents.azure.com:443/
    Collection : db.data at 100000 request units per second
    Document Template*: Player.json
    Degree of parallelism*: -1
    ---------------------------------------------------------------------

    DocumentDBBenchmark starting...
    Found collection data with 100000 RU/s
    Starting Inserts with 100 tasks
    Inserted 4503 docs @ 4491 writes/s, 47070 RU/s (122B max monthly 1KB reads)
    Inserted 17910 docs @ 8862 writes/s, 92878 RU/s (241B max monthly 1KB reads)
    Inserted 32339 docs @ 10531 writes/s, 110366 RU/s (286B max monthly 1KB reads)
    Inserted 47848 docs @ 11675 writes/s, 122357 RU/s (317B max monthly 1KB reads)
    Inserted 58857 docs @ 11545 writes/s, 120992 RU/s (314B max monthly 1KB reads)
    Inserted 69547 docs @ 11378 writes/s, 119237 RU/s (309B max monthly 1KB reads)
    Inserted 80687 docs @ 11345 writes/s, 118896 RU/s (308B max monthly 1KB reads)
    Inserted 91455 docs @ 11272 writes/s, 118131 RU/s (306B max monthly 1KB reads)
    Inserted 102129 docs @ 11208 writes/s, 117461 RU/s (304B max monthly 1KB reads)
    Inserted 112444 docs @ 11120 writes/s, 116538 RU/s (302B max monthly 1KB reads)
    Inserted 122927 docs @ 11063 writes/s, 115936 RU/s (301B max monthly 1KB reads)
    Inserted 133157 docs @ 10993 writes/s, 115208 RU/s (299B max monthly 1KB reads)
    Inserted 144078 docs @ 10988 writes/s, 115159 RU/s (298B max monthly 1KB reads)
    Inserted 155415 docs @ 11013 writes/s, 115415 RU/s (299B max monthly 1KB reads)
    Inserted 166126 docs @ 10992 writes/s, 115198 RU/s (299B max monthly 1KB reads)
    Inserted 173051 docs @ 10739 writes/s, 112544 RU/s (292B max monthly 1KB reads)
    Inserted 180169 docs @ 10527 writes/s, 110324 RU/s (286B max monthly 1KB reads)
    Inserted 192469 docs @ 10616 writes/s, 111256 RU/s (288B max monthly 1KB reads)
    Inserted 199107 docs @ 10406 writes/s, 109054 RU/s (283B max monthly 1KB reads)
    Inserted 200000 docs @ 9930 writes/s, 104065 RU/s (270B max monthly 1KB reads)

    Summary:
    ---------------------------------------------------------------------
    Inserted 200000 docs @ 9928 writes/s, 104063 RU/s (270B max monthly 1KB reads)
    ---------------------------------------------------------------------
    DocumentDBBenchmark completed successfully.
    Press any key to exit...


**步驟 4 (如有必要)：** 從工具回報的輸送量 (RU/s) 應該等於或大於佈建的集合或一組集合的輸送量。 如果情況並非如此，向上微調 DegreeOfParallelism 可協助您達到該限制。 如果來自用戶端應用程式的輸送量達持平狀態，請在其他用戶端電腦上啟動多個應用程式執行個體。 如果您需要協助進行這個步驟，請傳送電子郵件至 askcosmosdb@microsoft.com 或在 [Azure 入口網站](https://portal.azure.com)提出支援票證。

讓應用程式處於執行狀態之後，您便可以嘗試不同的[索引編製原則](indexing-policies.md)和[一致性層級](consistency-levels.md)，以了解它們對輸送量和延遲的影響。 您也可以檢閱原始程式碼，然後對自己的測試套件或實際執行應用程式實作類似的組態。

## <a name="next-steps"></a>後續步驟
在這篇文章中，我們探討了如何使用 .NET 主控台應用程式來執行 Azure Cosmos DB 的相關效能和規模測試。 如需詳細資訊，請參閱下列文章：

* [Azure Cosmos DB 效能測試範例](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/documentdb-benchmark)
* [改善 Azure Cosmos DB 效能的用戶端設定選項](performance-tips.md)
* [Azure Cosmos DB 中的伺服器端資料分割](partition-data.md)


