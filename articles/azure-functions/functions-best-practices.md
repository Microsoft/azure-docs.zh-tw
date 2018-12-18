---
title: Azure Functions 的最佳作法 | Microsoft Docs
description: 了解 Azure Functions 的最佳作法與模式。
services: functions
documentationcenter: na
author: wesmc7777
manager: jeconnoc
keywords: azure functions, 模式, 最佳作法, 函數, 事件處理, webhook, 動態計算, 無伺服器架構
ms.assetid: 9058fb2f-8a93-4036-a921-97a0772f503c
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 10/16/2017
ms.author: glenga
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: aecd68734999c30f3dc8e7a2ea8c7d5e9cdfacb0
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166333"
---
# <a name="optimize-the-performance-and-reliability-of-azure-functions"></a>將 Azure Functions 效能和可靠性最佳化

本文提供指引來改善[無伺服器](https://azure.microsoft.com/solutions/serverless/)函式應用程式的效能和可靠性。 

## <a name="general-best-practices"></a>一般最佳作法

以下是如何使用 Azure Functions 建置及打造無伺服器解決方案的最佳做法。

### <a name="avoid-long-running-functions"></a>避免長時間執行的函式

大型長時間執行的函式可能會造成非預期的逾時問題。 函式可能會因為許多 Node.js 相依性而變大。 匯入相依性也可能會造成載入時間增加，而導致未預期的逾時。 系統會以明確和隱含方式載入相依性。 您的程式碼載入的單一模組可能會載入其本身的其他模組。  

在可能時，將大型函式重構為較小的函式集，共用運作並快速傳回回應。 例如，Webhook 或 HTTP 觸發程序函式可能要求在特定時間限制內的通知回應；Webhook 通常需要立即的回應。 您可以將 HTTP 觸發程序承載傳遞到要由佇列觸發程序函式處理的佇列中。 此方法可讓您延後實際工作，並傳回立即回應。


### <a name="cross-function-communication"></a>跨函式通訊

[Durable Functions](durable-functions-overview.md) 和 [Azure Logic Apps](../logic-apps/logic-apps-overview.md) 均建置用來管理多個函式之間的狀態轉換和通訊。

如果未使用 Durable Functions 或 Logic Apps 來與多個函式整合，使用儲存體佇列進行跨函式通訊通常是最佳做法。  主要原因是儲存體佇列更便宜和容易佈建。 

儲存體佇列中個別訊息大小限制在 64 KB。 如果您需要在函式之間傳遞更大型的訊息，Azure 服務匯流排佇列可用來支援標準層中大小上限為 256 KB 的訊息，以及進階層中上限為 1 MB 的訊息。

如果您需要在處理之前篩選訊息，服務匯流排主題會很實用。

事件中樞對於支援大量通訊很有用。


### <a name="write-functions-to-be-stateless"></a>撰寫無狀態函式 

若可能，Functions 應該是無狀態和具有等冪性。 將任何必要的狀態資訊與您的資料產生關聯。 例如，正在處理訂單就可能具有相關聯的 `state` 成員。 函式本身保持無狀態時，函式可以依據該狀態處理訂單。 

特別建議計時器觸發程序使用等冪函式。 例如，如果您有一天必須執行一次的項目，請編寫它，使得它可在一天中的任何時間執行，並具有相同的結果。 特定日沒有工作時，就可以結束函式。 如果先前的執行無法完成，下一次執行應該會定停止的位置開始。


### <a name="write-defensive-functions"></a>編寫防禦性函式

假設您的函式可能隨時會遇到例外狀況。 設計您的函式，使得它能夠在下一次執行期間從先前的失敗點繼續執行。 假設需要執行下列動作的案例︰

1. 查詢資料庫中的 10,000 個資料列。
2. 對每個資料列建立佇列訊息，來進一步處理向下一行。
 
根據系統的複雜程度，您可能會有︰相關的下游服務行為不當、網路中斷或到達配額限制等等。這所有方面都隨時會影響您的函式。 您必須設計您的函式，以對其做好準備。

如果在插入這些項目中的 5,000 個至佇列以進行處理之後發生失敗，您的程式碼如何因應？ 追蹤集合中您已完成的項目。 否則，您可能下一次又將它們插入。 這對您的工作流程會有嚴重影響。 

如果佇列項目已經過處理，請讓您的函式成為無作業。

利用已針對您在 Azure Functions 平台中所使用元件提供的防禦性措施。 例如，請參閱文件中**處理有害的佇列訊息**，以了解 [Azure 儲存體佇列觸發程序和繫結](functions-bindings-storage-queue.md#trigger---poison-messages)。 

## <a name="scalability-best-practices"></a>延展性最佳做法

有數個因素會影響應用程式函式執行個體的調整方式。 [函式調整](functions-scale.md)文件中會提供詳細資料。  以下是一些最佳做法，可確保函式應用程式的最佳延展性。

### <a name="share-and-manage-connections"></a>共用及管理連線

儘可能重複使用與外部資源的連線。  請參閱[如何管理 Azure Functions 中的連線](./manage-connections.md)。

### <a name="dont-mix-test-and-production-code-in-the-same-function-app"></a>不要在相同函式應用程式中混用測試和實際執行程式碼

函式應用程式內的 Functions 會共用資源。 例如，共用記憶體。 如果您在生產環境中使用函式應用程式，請勿對它新增與測試相關的函式和資源。 在實際執行程式碼執行期間可能導致發生未預期的額外負荷。

對於在實際執行函式應用程式中載入的項目，請務必小心。 記憶體會在應用程式中的每個函式間平均分配。

如果您在多個 .Net 函式中參考某個共用組件，請將它置於通用的共用資料夾中。 如果使用 C# 指令碼 (.csx)，請使用類似下列的範例陳述式來參考組件︰ 

    #r "..\Shared\MyAssembly.dll". 

否則，很容易不小心在函式之間部署相同二進位檔但不同行為的多個測試版本。

請勿在實際執行程式碼中使用詳細資訊記錄。 它對效能會有負面影響。

### <a name="use-async-code-but-avoid-blocking-calls"></a>使用非同步程式碼但避免封鎖呼叫

非同步程式設計是建議的最佳作法。 不過，請務必避免在 `Task` 執行個體上參考 `Result` 屬性或呼叫 `Wait` 方法。 這個方法可能會導致執行緒耗盡。

[!INCLUDE [HTTP client best practices](../../includes/functions-http-client-best-practices.md)]

### <a name="receive-messages-in-batch-whenever-possible"></a>儘可能分批接收訊息

某些觸發程序 (如事件中樞) 能夠在單一引動過程中接收一批訊息。  分批處理訊息的效能比較好。  如 [host.json 參考文件](functions-host-json.md)所述，您可以在 `host.json` 檔案中設定批次大小上限。

若為 C# 函式，您可以將類型變更為強型別陣列。  例如，方法簽章可能是 `EventData[] sensorEvent`，而不是 `EventData sensorEvent`。  若為其他語言，您必須在 `function.json` 中將基數屬性明確設定為 `many`，才能啟用批次處理 ([如這裡所示](https://github.com/Azure/azure-webjobs-sdk-templates/blob/df94e19484fea88fc2c68d9f032c9d18d860d5b5/Functions.Templates/Templates/EventHubTrigger-JavaScript/function.json#L10))。

### <a name="configure-host-behaviors-to-better-handle-concurrency"></a>設定主機的行為，更妥善處理並行作業

函式應用程式中的 `host.json` 檔案能夠設定主機執行階段和觸發程序行為。  除了批次處理行為，您可以管理數個觸發程序的並行作業。  經常調整這些選項中的值，可協助每個執行個體針對所叫用函式的需求進行適當調整。

主機檔案中的設定會套用到應用程式內、函式的「單一執行個體」內的所有函式。 比方說，如果您的函式應用程式具有 2 個 HTTP 函式，且並行要求設定為 25，對任一 HTTP 觸發程序的要求會計入共用的 25 個並行要求中。  如果該函式應用程式調整為 10 個執行個體，這 2 個函式可有效地允許 250 個並行要求 (10 個執行個體 * 每個執行個體 25 個並行要求)。

**HTTP 並行主機選項**

[!INCLUDE [functions-host-json-http](../../includes/functions-host-json-http.md)]

在[主機設定文件](functions-host-json.md)中可以找到其他主機設定選項。

## <a name="next-steps"></a>後續步驟

如需詳細資訊，請參閱下列資源：

* [如何管理 Azure Functions 中的連線](manage-connections.md)
* [Azure App Service 最佳做法](../app-service/app-service-best-practices.md)
