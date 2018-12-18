---
title: Azure Cosmos DB 大量執行程式程式庫概觀 | Microsoft Docs
description: 了解 Azure Cosmos DB 大量執行程式程式庫、使用程式庫的優點及其架構。
keywords: Java 大量執行程式
services: cosmos-db
author: tknandu
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: ramkris
ms.openlocfilehash: 7c490aa958cf9e78c260dd0fbcf7952b55d8d88c
ms.sourcegitcommit: d7725f1f20c534c102021aa4feaea7fc0d257609
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37096170"
---
# <a name="azure-cosmos-db-bulk-executor-library-overview"></a>Azure Cosmos DB 大量執行程式程式庫概觀
 
Azure Cosmos DB 是一種快速且具有彈性的全域分散式資料庫服務，其彈性相應放大的設計可支援： 

* 大型讀取和寫入輸送量 (每秒百萬個作業)。  
* 儲存大量 (數百 TB 或更多) 且可預測延遲毫秒數的交易和作業資料。  

大量執行程式程式庫可協助您運用此大規模的輸送量和儲存體，大量執行程式程式庫可讓您透過大量匯入和大量更新 API，在 Azure Cosmos DB 中執行大量作業。 您可以在下列各節中深入了解大量執行程式程式庫的功能。 

> [!NOTE] 
> 目前，大量執行程式程式庫支援匯入和更新作業，而且僅有 Azure Cosmos DB SQL API 帳戶可支援此程式庫。 如需有關程式庫的任何更新，請參閱 [.NET](sql-api-sdk-bulk-executor-dot-net.md) 和 [Java](sql-api-sdk-bulk-executor-java.md) 的版本資訊。
 
## <a name="key-features-of-the-bulk-executor-library"></a>大量執行程式程式庫的主要功能  
 
* 它會大幅減少佔滿輸送量 (配置到容器) 的必要用戶端運算資源。 比起平行寫入資料而佔滿用戶端機器 CPU 的多執行緒應用程式，使用大量匯入 API 寫入資料的單一執行緒應用程式，可達到 10 倍多的寫入輸送量。  

* 它以概念來簡單化撰寫應用程式邏輯的沉悶工作，有效率地在程式庫中處理要求速率限制、要求逾時和其他暫時性例外狀況。  

* 它為執行大量作業的應用程式提供可相應放大的簡化機制。在 Azure VM 上執行的單一大量執行程式執行個體取用的輸送量可大於 50 萬 RU/秒，而您可以藉由在個別用戶端 VM 上新增額外的執行個體來提高輸送量。  
 
* 它可以使用相應放大架構，在一小時內大量匯入 1 TB 以上的資料。  

* 它能夠以修補程式的形式，大量更新 Azure Cosmos DB 容器中的現有資料。 
 
## <a name="how-does-the-bulk-executor-operate"></a>Bulk Executor 如何運作？ 

當以實體批次觸發匯入或更新文件的大量作業時，這些資料一開始會隨機打散到與 Azure Cosmos DB 分割區索引鍵範圍對應的貯體。 在與分割區索引鍵範圍對應的每個貯體中，資料會細分為小型批次，而每個小型批次都會作為可在伺服器端上認可的承載。 大量執行程式程式庫針對不論是在分割區索引鍵範圍內還是跨分割區索引鍵範圍，並行執行這些迷你批次，都提供內建的最佳化。 下圖說明大量執行程式如何將資料批次處理到不同的分割區索引鍵：  

![大量執行程式架構](./media/bulk-executor-overview/bulk-executor-architecture.png)

Bulk Executor 程式庫可確保充分利用配置到集合的輸送量。 它會為每個 Azure Cosmos DB 分割區索引鍵範圍使用  [AIMD 樣式的壅塞控制機制](https://tools.ietf.org/html/rfc5681)，以有效率地處理速率限制和逾時。 

## <a name="next-steps"></a>後續步驟 
  
* 您可以嘗試透過 [.NET](bulk-executor-dot-net.md) 和 [Java](bulk-executor-java.md) 使用應用程式範例來取用 Bulk Executor 程式庫，以對其有進一步了解。  
* 查看 [.NET](sql-api-sdk-bulk-executor-dot-net.md) 和 [Java](sql-api-sdk-bulk-executor-java.md) 中的大量執行程式 SDK 資訊和版本資訊。
* Bulk Executor 程式庫已整合至 Cosmos DB Spark 連接器，若要進一步了解，請參閱 [Azure Cosmos DB Spark 連接器](spark-connector.md)一文。  
* 大量執行程式程式庫也已整合至新版 [Azure Cosmos DB 連接器](https://aka.ms/bulkexecutor-adf-v2)，可供 Azure Data Factory 複製資料。
