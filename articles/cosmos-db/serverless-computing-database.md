---
title: 無伺服器資料庫計算 - Azure Functions 和 Azure Cosmos DB | Microsoft Docs
description: 了解 Azure Cosmos DB 和 Azure Functions 如何一起使用以建立事件驅動無伺服器計算的應用程式。
services: cosmos-db
author: SnehaGunda
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: sngun
ms.openlocfilehash: b7908f9f9aef51857f0ffb07102e1df245b7a259
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064449"
---
# <a name="azure-cosmos-db-serverless-database-computing-using-azure-functions"></a>Azure Cosmos DB：使用 Azure Functions 的無伺服器資料庫計算

無伺服器計算是關於著重在可重複和無狀態之個別邏輯項目的能力。 這些項目不需要任何基礎結構管理，而且它們只會針對執行的項目耗用資源數秒或數毫秒。 無伺服器計算移動的核心是 [Azure Functions](https://azure.microsoft.com/services/functions) 在 Azure 生態系統中提供的函式。

使用 [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) 和 Azure Functions 之間的原生整合，您可以直接從您的 Azure Cosmos DB 帳戶建立資料庫觸發程序、輸入繫結，以及輸出繫結。 使用 Azure Functions 與 Azure Cosmos DB，您可以建立及部署事件驅動無伺服器的應用程式，利用低延遲存取廣大用戶群的豐富資料。

## <a name="overview"></a>概觀

Azure Cosmos DB 與 Azure Functions 可讓您以下列方式整合資料庫與無伺服器的應用程式：

* 在 Azure Function 中建立事件驅動的 **Azure Cosmos DB 觸發程序**。 此觸發程序依賴[變更摘要](change-feed.md)串流，以監控 Azure Cosmos DB 容器的變更。 對容器進行任何變更時，變更摘要串流會傳送至觸發程序，其叫用 Azure Function。
* 或者，使用**輸入繫結**將 Azure Function 繫結至 Azure Cosmos DB 容器。 函式執行時，輸入繫結會從容器讀取資料。
* 使用**輸出繫結**將函式繫結至 Azure Cosmos DB 容器。 函式完成時，輸出繫結會將資料寫入容器。

> [!NOTE]
> 此時，Azure Cosmos DB 觸發程序、輸入繫結，以及輸出繫結只會使用 SQL API 和 Gremlin API 帳戶。

下圖逐一說明這三個整合： 

![Azure Cosmos DB 和 Azure Functions 如何整合](./media/serverless-computing-database/cosmos-db-azure-functions-integration.png)

Azure Cosmos DB 觸發程序、輸入繫結及輸出繫結皆可用於下列組合：
* Azure Cosmos DB 觸發程序可與不同 Azure Cosmos DB 容器的輸出繫結搭配使用。 在函式對變更摘要中的項目執行操作後，您可以將其寫入另一個容器 (將其寫入與來源相同的容器將有效地建立遞迴迴圈)。 或者，您可以利用輸出繫結，使用 Azure Cosmos DB 觸發程序有效地將所有已變更的項目從一個容器移轉到不同的容器。
* Azure Cosmos DB 的輸入繫結與輸出繫結可用於相同的 Azure Function。 這也在適用於修改後，當您想要利用輸入繫結尋找特定資料、在 Azure Function 中加以修改，然後儲存到相同的容器或不同的容器的情況下。
* Azure Cosmos DB 容器的輸入繫結可在與 Azure Cosmos DB 觸發程序的相同函式中使用，並且不一定要與輸出繫結搭配使用。 您可以使用此組合，將最新的貨幣兌換資訊 (使用兌換容器的輸入繫結提取) 套用至購物車服務中新訂單的變更摘要。 已更新的購物車總計，以及目前套用的貨幣轉換，可以使用輸出繫結寫入第三個容器。

## <a name="use-cases"></a>使用案例

下列使用案例示範幾個方法，您可以藉由將您的資料連接至事件驅動的 Azure Functions 來充分利用 Azure Cosmos DB 資料。

### <a name="iot-use-case---azure-cosmos-db-trigger-and-output-binding"></a>IoT 使用案例 - Azure Cosmos DB 觸發程序和輸出繫結

在 IoT 實作中，您可以在檢查已連線汽車中顯示的引擎燈時，叫用函式。

**實作：** 使用 Azure Cosmos DB 觸發程序和輸出繫結

1. **Azure Cosmos DB 觸發程序**用來觸發與汽車警示相關的事件，例如檢查已連線汽車亮起的引擎燈。
2. 檢查引擎燈時，感應器資料會傳送到 Azure Cosmos DB。
3. Azure Cosmos DB 會建立或更新新的感應器資料文件，然後這些變更會串流處理至 Azure Cosmos DB 觸發程序。
4. 在感應器資料收集每次資料變更時叫用觸發程序，因為所有變更會透過變更摘要串流處理。
5. 閾值條件用於函式中以將感應器資料傳送至保修部門。
6. 如果溫度亦超過特定值，也會向擁有者發送警報。
7. 函式上的**輸出繫結**會更新另一個 Azure Cosmos DB 容器中的汽車記錄，以儲存檢查引擎事件的相關資訊。

下圖顯示此觸發程序在 Azure 入口網站中撰寫的程式碼。

![在 Azure 入口網站中建立 Azure Cosmos DB 觸發程序](./media/serverless-computing-database/cosmos-db-trigger-portal.png)

### <a name="financial-use-case---timer-trigger-and-input-binding"></a>財務使用案例 - 計時器觸發程序和輸入繫結

在財務實作中，當銀行帳戶餘額低於特定金額時，您可以叫用函式。

**實作：** 具有 Azure Cosmos DB 輸入繫結的計時器觸發程序

1. 使用[計時器觸發程序](../azure-functions/functions-bindings-timer.md)，您可以使用**輸入繫結**，以定時間隔擷取儲存在 Azure Cosmos DB 容器的銀行帳戶餘額資訊。
2. 如果餘額低於使用者設定的低餘額閾值，則追蹤 Azure Function 中的動作。
3. 輸出繫結可以是 [SendGrid 整合](../azure-functions/functions-bindings-sendgrid.md)，將來自服務帳戶的電子郵件傳送至針對每個低餘額帳戶識別的電子郵件地址。

下列映像顯示此案例之 Azure 入口網站中的程式碼。

![財務案例之計時器觸發程序的 Index.js 檔案](./media/serverless-computing-database/cosmos-db-functions-financial-trigger.png)

![財務案例之計時器觸發程序的 Run.csx 檔案](./media/serverless-computing-database/azure-function-cosmos-db-trigger-run.png)

### <a name="gaming-use-case---azure-cosmos-db-trigger-and-output-binding"></a>遊戲使用案例 - Azure Cosmos DB 觸發程序和輸出繫結

在遊戲中建立新的使用者時，您可以使用 [Azure Cosmos DB Gremlin API](graph-introduction.md) 搜尋可能知道的其他使用者。 然後，您可以將結果寫入 [Azure Cosmos DB SQL 資料庫] 以方便擷取。

**實作：** 使用 Azure Cosmos DB 觸發程序和輸出繫結

1. 使用 Azure Cosmos DB [圖表資料庫](graph-introduction.md)來儲存所有使用者，您可以使用 Azure Cosmos DB 觸發程序來建立新的函式。 
2. 每當插入新的使用者時，會叫用函式，然後使用**輸出繫結**來儲存結果。
3. 函式會查詢圖表資料庫，以搜尋與新使用者直接相關的所有使用者，並將該資料集傳回函式。
4. 接著，此資料會儲存在 Azure Cosmos DB 中，讓任何顯示新使用者其連線好友的前端應用程式輕鬆地擷取。

### <a name="retail-use-case---multiple-functions"></a>零售使用案例 - 多個函式

在零售實作中，當使用者將項目加入其購物籃中時，您可以靈活地為可選的業務管道元件建立和叫用函式。

**實作：** 接聽單一容器的多個 Azure Cosmos DB 觸發程序

1. 您可以藉由將 Azure Cosmos DB 觸發程序新增至每個函式，以建立多個 Azure Functions，所有函式皆會接聽購物車資料的相同變更摘要。 請注意，若有多個函式接聽相同的變更摘要，則每個函式皆需要新的租用集合。 如需有關租用集合的詳細資訊，請參閱[了解變更摘要處理器程式庫](change-feed.md#understand-cf)。
2. 每當新的項目新增至使用者的購物車時，會根據購物車容器的變更摘要單獨叫用每個函式。
    * 某個函式可以使用目前購物籃的內容來變更使用者可能感興趣之其他項目的顯示。
    * 另一個函式可更新存貨總計。
    * 另一個函式可將特定產品的客戶資訊傳送至行銷部門，向他們傳送促銷郵件。 

    任何部門可以藉由接聽變更摘要來建立 Azure Cosmos DB 觸發程序，並確保他們不會在此過程中延遲重要訂單處理事件。

在所有這些使用案例中，由於函式已分離應用程式本身，因此您不需要一直啟動新的應用程式執行個體。 相反地，Azure Functions 會啟動個別函式，視需要完成離散程序。

## <a name="tooling"></a>工具

Azure 入口網站和 Visual Studio 2017 提供 Azure Cosmos DB 與 Azure Functions 之間的原生整合。

* 在 Azure Functions 入口網站中，您可以建立 Azure Cosmos DB 觸發程序。 如需快速入門指示，請參閱[在 Azure 入口網站中建立 Azure Cosmos DB 觸發程序](https://aka.ms/cosmosdbtriggerportalfunc)。
* 在 Azure Cosmos DB 入口網站中，您可以將 Azure Cosmos DB 觸發程序新增至相同資源群組中現有的 Azure Function 應用程式。
* 在 Visual Studio 2017 中，您可以使用 [Azure Functions Tools for Visual Studio 2017](../azure-functions/functions-develop-vs.md) 建立 Azure Cosmos DB 觸發程序：

    >[!VIDEO https://www.youtube.com/embed/iprndNsUeeg]

## <a name="why-choose-azure-functions-integration-for-serverless-computing"></a>為何選擇 Azure Functions 整合以用於無伺服器的計算？

Azure Functions 提供了建立可擴展工作單位的功能，或可視需要執行的簡單邏輯，而無需佈建或管理基礎架構。 藉由使用 Azure Functions，您不必建立成熟的應用程式以回應 Azure Cosmos DB 資料庫中的變更，您可以建立適用於特定工作的小型可重複使用函式。 此外，您也可以使用 Azure Cosmos DB 資料作為 Azure Function 的輸入或輸出，以回應如 HTTP 要求或定時觸發之類的事件。

無伺服器計算建議使用 Azure Cosmos DB 的資料庫，原因如下：

* **即時存取所有資料**：您可以精確存取儲存的每個值，因為 Azure Cosmos DB 依預設會[自動編製索引](indexing-policies.md)所有資料，並讓這些索引立即可用。 這表示您可以經常向資料庫查詢、更新和新增項目，並透過 Azure Functions 即時存取。

* **無結構描述**。 Azure Cosmos DB 是無結構描述的 - 因此它有獨特的能力可處理來自 Azure Function 的任何資料輸出。 這種「處理任何事」的方法使其可直接建立所有輸出至 Azure Cosmos DB 的各種函式。

* **可擴充的輸送量**。 輸送量可以在 Azure Cosmos DB 中即時地相應增加或相應減少。 如果您有數百或數千個功能查詢，並且寫入相同容器，則可以相應增加輸送量的 [RU/s](request-units.md) 以處理負載。 所有函式可以使用您分配的 RU/秒來並行工作，且保證您的資料[一致](consistency-levels.md)。

* **全域複寫**。 您可以複寫[全域範圍內](distribute-data-globally.md)的 Azure Cosmos DB 資料以降低延遲，並異地尋找最靠近使用者所在位置的資料。 如同所有 Azure Cosmos DB 查詢，事件驅動觸發程序的資料是從最靠近使用者的 Azure Cosmos DB 讀取資料。

如果您想要整合 Azure Functions 以儲存資料，而且不需要深度編製索引，或如果您需要儲存附件與媒體檔案，則 [Azure Blob 儲存體觸發程序](../azure-functions/functions-bindings-storage-blob.md)可能是更佳的選擇。

Azure Functions 的優點： 

* **事件驅動**。 Azure Functions 是事件驅動的，而且可接聽來自 Azure Cosmos DB 的變更摘要。 這表示您不需要建立接聽邏輯，您只要留意正在接聽的變更即可。 

* **無限制**。 函式會並行執行，而且該服務會根據您的需要啟動。 設定參數。

* **適用於快速工作**。 當事件觸發時，服務會啟動函式的新執行個體，並在函式完成後立即關閉這些執行個體。 您只需要針對函式有執行的時間來付費。

如果您不確定 Flow、Logic Apps、Azure Functions 或 WebJobs 是否適合您的實作，請參閱[在 Flow、Logic Apps、Functions 和 WebJobs 之間做選擇](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md)。

## <a name="next-steps"></a>後續步驟

現在，讓我們實際連接 Azure Cosmos DB 與 Azure Functions： 

* [在 Azure 入口網站中建立 Azure Cosmos DB 觸發程序](https://aka.ms/cosmosdbtriggerportalfunc)
* [使用 Azure Cosmos DB 輸入繫結建立 Azure Functions HTTP 觸發程序 (Create an Azure Functions HTTP trigger with an Azure Cosmos DB input binding)](https://aka.ms/cosmosdbinputbind)
* [Azure DB Cosmos 繫結和觸發程序](../azure-functions/functions-bindings-cosmosdb.md)


 



