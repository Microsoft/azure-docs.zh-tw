---
title: Azure 串流分析輸出至 Cosmos DB
description: 本文說明如何使用 Azure 串流分析將輸出儲存至 JSON 輸出的 Azure Cosmos DB，以針對非結構化 JSON 資料進行資料封存和低延遲查詢。
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/28/2017
ms.openlocfilehash: 95cfc7e6d9515274aa7a3c5fde382244f3b33fab
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42142127"
---
# <a name="azure-stream-analytics-output-to-azure-cosmos-db"></a>Azure 串流分析輸出至 Azure Cosmos DB  
「串流分析」可以將 [Azure Cosmos DB](https://azure.microsoft.com/services/documentdb/) 設定為 JSON 輸出的目標，讓您能夠針對非結構化的 JSON 資料進行資料封存和低延遲查詢。 本文件涵蓋實作這種組態的一些最佳作法。

如果您不熟悉 Cosmos DB，請參閱 [Azure Cosmos DB 的學習路徑](https://azure.microsoft.com/documentation/learning-paths/documentdb/)來開始著手。 

> [!Note]
> 此時 Azure 串流分析僅支援使用 **SQL API** 連線至 Azure CosmosDB。
> 尚不支援其他 Azure Cosmos DB API。 如果您將 Azure Stream Analytics 指向使用其他 API 建立的 Azure Cosmos DB 帳戶，可能無法正確儲存資料。 

## <a name="basics-of-cosmos-db-as-an-output-target"></a>將 Cosmos DB 設定為輸出目標的基本概念
「串流分析」中的 Azure Cosmos DB 輸出可將串流處理結果以 JSON 輸出的形式寫入到您的 Cosmos DB 集合中。 串流分析不會在資料庫中建立集合，而是需要您預先建立集合。 如此一來，Cosmos DB 集合的計費成本就在您的掌握之中，而您也能直接使用 [Cosmos DB API](https://msdn.microsoft.com/library/azure/dn781481.aspx) 來微調集合的效能、一致性及容量。 

以下詳細說明一些 Cosmos DB 集合選項。

## <a name="tune-consistency-availability-and-latency"></a>微調一致性、 可用性及延遲
為了符合應用程式需求，Azure Cosmos DB 允許您微調資料庫與集合，並在一致性、可用性及延遲之間進行取捨。 您可以視案例針對讀取與寫入延遲所需的讀取一致性層級，來選擇資料庫帳戶上的一致性層級。 此外，Azure Cosmos DB 預設會在對您集合進行的每個 CRUD 作業進行同步索引編製。 這是另一個可在 Azure Cosmos DB 中控制寫入/讀取效能的實用選項。 如需詳細資訊，請檢閱[變更資料庫及查詢的一致性層級](../cosmos-db/consistency-levels.md)一文。

## <a name="upserts-from-stream-analytics"></a>來自串流分析的 Upsert
「串流分析」與 Azure Cosmos DB 的整合可讓您根據指定的「文件識別碼」資料行，在集合中插入或更新記錄。 這也稱為「Upsert」 。

串流分析可使用最佳化的 upsert 方法，而此方法只會在因為文件識別碼發生衝突而插入失敗時，才進行更新。 此更新會以修補程式的方式執行，對文件進行部分更新，也就是以漸進方式新增屬性或取代現有屬性。 但是，變更 JSON 文件中陣列屬性的值，會造成整個陣列遭到覆寫，也就是不會合併陣列。

## <a name="data-partitioning-in-cosmos-db"></a>Cosmos DB 中的資料分割
Azure Cosmos DB [無限制](../cosmos-db/partition-data.md)是建議的資料分割方法，因為 Azure Cosmos DB 會根據您的工作負載自動調整資料分割。 當寫入無限制的容器時，串流分析會使用與先前查詢步驟或輸入資料分割配置同樣數目的平行寫入器。
> [!Note]
> 目前，「Azure 串流分析」只支援分割區索引鍵位於頂層的無限制集合。 例如，支援 `/region`。 不支援巢狀分割區索引鍵 (例如 `/region/name`)。 

針對固定 Azure Cosmos DB 集合，在空間已滿時，串流分析不允許以任何方式相應增加或相應放大。 集合有 10GB 和 10000 RU/秒的輸送量上限。  若要將資料從固定容器移轉到無限制的容器 (例如，輸送量至少為 1000 RU/s，且具有分割區索引鍵的容器)，您必須使用[資料移轉工具](../cosmos-db/import-data.md)或[變更摘要庫](../cosmos-db/change-feed.md)。

我們正在取代寫入至多個固定容器，因此這不是將串流分析作業相應放大的建議方法。 [Cosmos DB 中的資料分割與調整](../cosmos-db/sql-api-partition-data.md)一文提供進一步的詳細資料。

## <a name="cosmos-db-settings-for-json-output"></a>適用於 JSON 輸出的 Cosmos DB 設定
如果在「串流分析」中建立 Cosmos DB 作為輸出，將會產生如以下所示的資訊提示。 本節說明屬性定義。


![documentdb 串流分析輸出畫面](media/stream-analytics-documentdb-output/stream-analytics-documentdb-output-1.png)

欄位           | 說明 
-------------   | -------------
輸出別名    | 在您的 ASA 查詢中參照此輸出時所用的別名   
帳戶名稱    | Azure Cosmos DB 帳戶的名稱或端點 URI 
帳戶金鑰     | Azure Cosmos DB 帳戶的共用存取金鑰
資料庫        | Azure Cosmos DB 資料庫名稱
集合名稱 | 要使用的集合之集合名稱。 `MyCollection` 是範例有效輸入 - 必須有名稱為 `MyCollection` 的集合存在。  
文件識別碼     | 選用。 輸出事件中的資料行名稱，輸出事件是作為唯一索引鍵，插入或更新作業必須依據該索引鍵。 如果保留空白，所有事件都會插入，沒有更新選項。
