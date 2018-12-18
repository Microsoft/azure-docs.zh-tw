---
title: Azure Cosmos DB Cassandra API 簡介 | Microsoft Docs
description: 了解如何透過您已熟悉的 Cassandra 驅動程式和 CQL 使用 Cassandra API，來利用 Azure Cosmos DB「隨即轉移」現有應用程式和建立新的應用程式。
services: cosmos-db
author: kanshiG
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-cassandra
ms.devlang: na
ms.topic: overview
ms.date: 09/24/2018
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: fe0ed06c5ecd0a84e9999c09cd312ddea2557f80
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47221638"
---
# <a name="introduction-to-the-azure-cosmos-db-cassandra-api"></a>Azure Cosmos DB Cassandra API 簡介

Azure Cosmos DB Cassandra API 可供針對 [Apache Cassandra](https://cassandra.apache.org/) 撰寫的應用程式作為資料存放區。 這表示，藉由使用符合 CQLv4 的現有 [Apache 驅動程式](https://cassandra.apache.org/doc/latest/getting_started/drivers.html?highlight=driver)，您現有的 Cassandra 應用程式現在可以與 Azure Cosmos DB Cassandra API 通訊。 在許多情況下，只需變更連接字串，您就可以從使用 Apache Cassandra 切換為使用 Azure Cosmos DB 的 Cassandra API。 

使用 Cassandra 查詢語言 (CQL)、Cassandra 型工具 (例如 cqlsh)，以及您熟悉的 Cassandra 用戶端驅動程式，Cassandra API 可讓您與儲存在 Azure Cosmos DB 中的資料互動。

## <a name="what-is-the-benefit-of-using-apache-cassandra-api-for-azure-cosmos-db"></a>使用適用於 Azure Cosmos DB 的 Apache Cassandra API 有什麼優點？

**沒有作業管理**：Azure Cosmos DB Cassandra API 是完全受控的雲端服務，可卸除因管理及監視各個作業系統、JVM 和 yaml 檔案及其互動的各種設定而產生的工作負荷。 Azure Cosmos DB 可監視輸送量、延遲、儲存體、可用性和可設定的警示。

**效能管理**：Azure Cosmos DB 提供受 SLA 支援的保證低度延遲讀取和寫入 (第 99 個百分位數)。 使用者無須擔心作業的負荷，同時可確保高效能和低延遲的讀取和寫入。 這表示，使用者不需要處理排程壓縮、管理標記，以及手動設定 Bloom 篩選和複本。 Azure Cosmos DB 可卸除管理這些問題的工作負荷，讓您專注於應用程式的邏輯。

**可使用現有的程式碼和工具**：Azure Cosmos DB 提供與現有 Cassandra SDK 和工具相容的有線通訊協定層級。 此相容性可確保您可以透過 Azure Cosmos DB Cassandra API 使用現有程式碼基底執行瑣碎的變更。

**輸送量與儲存體彈性**：Azure Cosmos DB 可透過 Azure 入口網站、PowerShell 或 CLI 作業提供跨所有區域的保證輸送量，以及調整佈建的輸送量。 您可以依據可預測的效能，視需要為您的資料表彈性地調整儲存體和輸送量。

**全域散發和可用性**：Azure Cosmos DB 可讓您跨所有 Azure 區域全域散發資料以及在本機提供資料，同時確保低延遲的的資料存取和高可用性。 Azure Cosmos DB 在單一區域內可提供 99.99% 的高可用性，且在多個區域間可提供 99.999% 的讀取和寫入可用性，且不會產生額外的作業負荷。 請參閱[全域散發資料](distribute-data-globally.md)一文以深入了解。 

**一致性選擇**：Azure Cosmos DB 提供五個定義完善的一致性層級可選擇，讓您能在一致性與效能之間做出最好的取捨。 這些一致性層級包括強式、限定過期、工作階段、一致前置詞和最終。 這些定義完善、實用且直觀的一致性層級可讓開發人員在一致性、可用性與延遲三者間做出正確的取捨。 請參閱[一致性層級](consistency-levels.md)一文以深入了解。 

**企業級**：Azure cosmos DB 會提供[相容性認證](https://www.microsoft.com/trustcenter)，確保使用者可以安全地使用平台。 Azure Cosmos DB 也會為控制層活動提供待用加密和移動中加密、IP 防火牆和稽核記錄。

## <a name="next-steps"></a>後續步驟

* 您可以快速地開始建置下列特定語言的應用程式，以建立和管理 Cassandra API 資料：
  - [Node.js 應用程式](create-cassandra-nodejs.md)
  - [.NET 應用程式](create-cassandra-dotnet.md)
  - [Python 應用程式](create-cassandra-python.md)

* 開始使用 Java 應用程式來[建立 Cassandra API 帳戶、資料庫及資料表](create-cassandra-api-account-java.md)。

* 使用 Java 應用程式[將範例資料載入至 Azure Cosmos DB Cassandra API 資料表](cassandra-api-load-data.md)。

* 使用 Java 應用程式[從 Cassandra API 帳戶查詢資料](cassandra-api-query-data.md)。

* 若要深入了解 Azure Cosmos DB Cassandra API 所支援的 Apache Cassandra 功能，請參閱 [Cassandra 支援](cassandra-support.md)一文。

* 請參閱[常見問題集](faq.md#cassandra)。
