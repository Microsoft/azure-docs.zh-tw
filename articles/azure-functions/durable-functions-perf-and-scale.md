---
title: Durable Functions 中的效能和級別 - Azure
description: Azure Functions 的 Durable Functions 擴充簡介。
services: functions
author: cgillum
manager: jeconnoc
keywords: ''
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 04/25/2018
ms.author: azfuncdf
ms.openlocfilehash: 669a436293ddf6f13760db5e6802aaae82ddd74b
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45577508"
---
# <a name="performance-and-scale-in-durable-functions-azure-functions"></a>Durable Functions (Azure Functions) 中的效能和級別

若要最佳化效能和延展性，務必了解 [Durable Functions](durable-functions-overview.md) 獨特的調整規模特性。

若要了解調整規模行為，您必須了解基礎 Azure 儲存體提供者的一些詳細資料。

## <a name="history-table"></a>記錄資料表

[歷程記錄] 資料表是 Azure 儲存體資料表，含有工作中樞內所有協調流程執行個體的歷程記錄事件。 此資料表的名稱格式為 *TaskHubName*History。 隨著執行個體執行，此資料表中會新增資料列。 此資料表的資料分割索引鍵衍生自協調流程的執行個體識別碼。 在大部分情況下，執行個體識別碼都是隨機的，可確保 Azure 儲存體中的內部資料分割有最佳的分佈。

當協調流程執行個體需要執行時，系統會將 [歷程記錄] 資料表的適當資料列載入記憶體中。 這些「歷程記錄事件」會接著重新顯示為協調器函式程式碼，使其回到先前的檢查點狀態。 以這種方式使用執行歷程記錄重建狀態，會受[事件來源模式](https://docs.microsoft.com/azure/architecture/patterns/event-sourcing)所影響。

## <a name="instances-table"></a>執行個體資料表

[執行個體] 資料表是另一個 Azure 儲存體資料表，含有工作中樞內所有協調流程執行個體的狀態。 隨著執行個體的建立，此資料表中會新增資料列。 此資料表的資料分割索引鍵是協調流程執行個體識別碼，而資料列索引鍵是固定的常數。 每個協調流程執行個體都有一個資料列。

此資料表用來滿足來自 [GetStatusAsync](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationClient.html#Microsoft_Azure_WebJobs_DurableOrchestrationClient_GetStatusAsync_System_String_) API 以及[狀態查詢 HTTP API](https://docs.microsoft.com/azure/azure-functions/durable-functions-http-api#get-instance-status) 的執行個體查詢要求。 它終於與先前所述的 [歷程記錄] 資料表內容保持一致。 以這種方式使用不同的 Azure 儲存體資料表有效地滿足執行個體查詢作業，會受到[命令和查詢責任隔離 (CQRS) 模式](https://docs.microsoft.com/azure/architecture/patterns/cqrs)所影響。

## <a name="internal-queue-triggers"></a>內部佇列觸發程序

協調器函式和活動函式都是由函式應用程式的工作中樞內的內部佇列所觸發。 以這種方式使用佇列會提供可靠的「至少一次」訊息傳遞保證。 Durable Functions 中有兩種佇列：**控制佇列**和**工作項目佇列**。

### <a name="the-work-item-queue"></a>工作項目佇列

在 Durable Functions 中，每個工作中樞各有一個工作項目佇列。 這是基本佇列，運作方式類似於 Azure Functions 中的其他任何 `queueTrigger` 佇列。 此佇列用來觸發無狀態「活動函式」，做法是一次從佇列中清除一則訊息。 上述每則訊息都包含活動函式輸入和其他中繼資料，例如要執行哪個函式。 當 Durable Functions 應用程式向外延展至多個虛擬機器時，這些虛擬機器全部會從工作項目佇列中爭奪工作。

### <a name="control-queues"></a>控制佇列

在 Durable Functions 中，每個工作中樞都有多個「控制佇列」。 「控制佇列」比簡單的工作項目佇列更複雜。 控制佇列可用來觸發具狀態協調器函式。 因為協調器函式執行個體是具狀態單次個體，所以無法使用競爭取用者模型在虛擬機器之間分散負載。 相反地，協調器訊息會分散至多個控制佇列來平衡負載。 後續各節對此行為有更詳細的說明。

控制佇列包含各種不同的協調流程生命週期訊息類型。 例如，[協調器控制訊息](durable-functions-instance-management.md)、活動函式「回應」訊息，以及計時器訊息。 將有 32 則訊息會在單一輪詢中從控制佇列中清除。 這些訊息包含承載資料以及中繼資料，包括適用於哪個協調流程執行個體。 如果有多則已從佇列中清除的訊息適用於相同的協調流程執行個體，系統會將這些訊息當作一個批次處理。

## <a name="storage-account-selection"></a>儲存體帳戶選取

Durable Functions 所用的佇列、資料表和 blob 是由已設定的 Azure 儲存體帳戶所建立。 使用 **host.json** 檔案中的 `durableTask/azureStorageConnectionStringName` 設定，可以指定要使用的帳戶。

```json
{
  "durableTask": {
    "azureStorageConnectionStringName": "MyStorageAccountAppSetting"
  }
}
```

如果未指定，則會使用預設 `AzureWebJobsStorage` 儲存體帳戶。 不過，對於重視效能的工作負載，建置您設定非預設的儲存體帳戶。 Durable Functions 會密集使用 Azure 儲存體，而使用專用儲存體帳戶會隔離 Durable Functions 儲存體使用量與 Azure Functions 主機的內部使用量。

## <a name="orchestrator-scale-out"></a>協調器向外延展

活動函式是無狀態，且經由新增虛擬機器而自動相應放大。 相反地，協調器函式會「分割」至一或多個控制佇列。 控制佇列數目會定義於 **host.json** 檔案中。 下列範例 host.json 程式碼片段會將 `durableTask/partitionCount` 屬性設為 `3`。

```json
{
  "durableTask": {
    "partitionCount": 3
  }
}
```
一個工作中樞可設定為有 1 到 16 個資料分割。 若未指定，則預設資料分割計數為 **4**。

向外延展至多個函式主機執行個體時 (通常在不同的虛擬機器上)，每個執行個體取得其中一個控制佇列的鎖定。 這些鎖定會在內部實作為 blob 儲存體租用，並確保協調流程執行個體一次只會在單一主機執行個體上執行。 如果工作中樞已設定三個控制佇列，則協調流程執行個體最多可以跨三個虛擬機器來平衡負載。 您可以新增更多虛擬機器，以增加容量來執行活動函式。

下圖說明在向外延展環境中，Azure Functions 主機與儲存體實體之間的互動方式。

![級別圖表](media/durable-functions-perf-and-scale/scale-diagram.png)

如上圖所示，所有虛擬機器會爭奪工作項目佇列上的訊息。 不過，只有三個虛擬機器可以從控制佇列中取得訊息，每個虛擬機器會鎖定單一控制佇列。

協調流程執行個體會分散到所有控制佇列執行個體。 雜湊協調流程的執行個體識別碼，即可進行散發。 執行個體識別碼預設是隨機的 Guid，可確保執行個體平均分散到所有控制佇列。

一般而言，協調器函式是輕巧的設計，應該不需要大量運算能力。 因此，不需要建立大量的控制佇列資料分割來獲得很高的輸送量。 大部分繁重的工作應在無狀態活動函式中進行，這還可以無限制地相應放大。

## <a name="auto-scale"></a>自動調整規模

由於所有 Azure Functions 都在使用情況方案中執行，Durable Functions 支援透過 [Azure Functions 縮放控制器](functions-scale.md#runtime-scaling)來自動調整規模。 縮放控制器藉由定期發出 _peek_ 命令來監視所有佇列的延遲。 縮放控制器會以所查看訊息的延遲為基礎，決定是要新增或移除 VM。

如果縮放控制器判斷控制佇列訊息延遲太高，它會新增 VM 執行個體，直到訊息延遲減少到可接受的程度或達到控制佇列資料分割計數。 同樣地，如果工作項目佇列延遲偏高，不論資料分割計數為何，縮放控制器都會持續新增 VM 執行個體。

## <a name="thread-usage"></a>執行緒使用方式

協調器函式會在單一執行緒上執行，確保執行作業可透過多次重新執行來下決定。 由於此單一執行緒執行作業的緣故，所以協調器函式執行緒切勿執行需要大量 CPU 的工作、進行 I/O 或因故封鎖。 任何可能需要 I/O、封鎖或多個執行緒的工作，都應該移入活動函式中。

活動函式和一般佇列觸發的函式有完全相同的行為。 它們可以安全地執行 I/O、執行需要大量 CPU 的作業，以及使用多個執行緒。 因為活動觸發程序是無狀態，所以能夠自由地向外延展至不限數量的虛擬機器。

## <a name="concurrency-throttles"></a>並行節流

Azure Functions 支援在單一應用程式執行個體中同時執行多個函式。 此並行執行作業有助於提升平行處理原則，且盡可能減少典型應用程式在一段時間內會遇到的「冷啟動」數目。 不過，高並行處理可能會導致每部 VM 的記憶體使用量很高。 視函式應用程式的需求而定，每個執行個體並行處理可能必須進行節流，以避免在高負載情況下記憶體不足的可能性。

您可以在 **host.json** 檔案中設定活動函式與協調器函式並行限制。 相關的設定分別是 `durableTask/maxConcurrentActivityFunctions` 和 `durableTask/maxConcurrentOrchestratorFunctions`。

```json
{
  "durableTask": {
    "maxConcurrentActivityFunctions": 10,
    "maxConcurrentOrchestratorFunctions": 10,
  }
}
```

在上述範例中，在單一 VM 上最多可以同時執行 10 個協調器函式和 10 個活動函式。 如果未指定，並行活動與協調器函式執行次數的上限為 VM 上核心數目的 10 倍。

> [!NOTE]
> 這些設定很有用，有助於管理單一 VM 上的記憶體和 CPU 使用量。 不過，若跨越多部 VM 相應放大，則每部 VM 都會有自己的限制集合。 這些設定不能用來控制全域層級的並行存取。

## <a name="orchestrator-function-replay"></a>協調器函式重新執行
如先前所述，使用 [歷程記錄] 資料表的內容可重新執行協調器函式。 根據預設，每次從控制佇列中清除一批訊息時，就會重新執行協調器函式程式碼。

啟用**擴充工作階段**，即可停用此積極重新執行行為。 啟用擴充工作階段後，協調器函式執行個體保留在記憶體中的時間更長，而不需完全重新執行即可處理新訊息。 在 **host.json** 檔案中將 `durableTask/extendedSessionsEnabled` 設定為 `true`，就會啟用擴充工作階段。 `durableTask/extendedSessionIdleTimeoutInSeconds` 設定用來控制閒置工作階段會保留在記憶體中多久：

```json
{
  "durableTask": {
    "extendedSessionsEnabled": true,
    "extendedSessionIdleTimeoutInSeconds": 30
  }
}
```

啟用擴充工作階段的典型效果，就是減少對 Azure 儲存體帳戶的 I/O 及提升整體輸送量。

不過，這項功能有一個潛在缺點，就是該閒置協調器函式執行個體保留在記憶體中的時間更長。 有兩個值得注意的影響：

1. 整體增加函式應用程式記憶體使用量。
2. 如果有許多並行的短期協調器函式執行作業，整體輸送量就會降低。

例如，如果`durableTask/extendedSessionIdleTimeoutInSeconds` 設為 30 秒，則執行時間少於 1 秒的短期協調器函式片段仍會佔用記憶體 30 秒。 它也會計入先前所述的 `durableTask/maxConcurrentOrchestratorFunctions` 配額中，而可能讓其他協調器函式無法執行。

> [!NOTE]
> 只有在協調器函式經過完整開發及測試之後，才能使用這些設定。 預設的積極重新執行行為很適合在開發階段用於偵測協調器函式中的冪等性錯誤。

## <a name="performance-targets"></a>效能目標

在規劃將 Durable Functions 用於生產應用程式時，請務必在規劃初期考慮效能需求。 本節涵蓋一些基本使用案例，以及預期的最大輸送量數字。

* **循序活動執行**：這個案例說明的協調器函式會一個接一個執行一系列活動函式。 它最類似於[函式鏈結](durable-functions-sequence.md)範例。
* **平行活動執行**：這個案例說明的協調器函式會使用[展開傳送、收合傳送](durable-functions-cloud-backup.md)模式平行執行許多活動函式。
* **平行處理回應**：這個案例是[展開傳送、收合傳送](durable-functions-cloud-backup.md)模式的第二個部份。 它著重於收合傳送的效能。 請務必請注意，不同於展開傳送，收合傳送是由單一協調器函式執行個體進行，因此只能在單一 VM 上執行。
* **外部事件處理**：這個案例代表為[外部事件](durable-functions-external-events.md)服務的單一協調器函式執行個體 (一次一個)。

> [!TIP]
> 不同於展開傳送，收合傳送作業受限制於單一 VM。 如果您的應用程式使用展開傳送、收合傳送模式，而且您很在意收合傳送效能，請考慮將活動函式展開傳送細分到多個[子協調流程](durable-functions-sub-orchestrations.md)。

下表顯示先前所述案例的預期「最大」輸送量數字。 「執行個體」是指在 Azure App Service 中單一小型 ([A1](../virtual-machines/windows/sizes-previous-gen.md#a-series)) VM 上執行之協調器函式的單一執行個體。 在所有情況下，假設已啟用[擴充工作階段](#orchestrator-function-replay)。 實際結果可能會因函式程式碼所執行的 CPU 或 I/O 工作而有所不同。

| 案例 | 最大輸送量 |
|-|-|
| 循序活動執行 | 每個執行個體每秒 5 個活動 |
| 平行活動執行 (展開傳送) | 每個執行個體每秒 100 個活動 |
| 平行回應處理 (收合傳送) | 每個執行個體每秒 150 個回應 |
| 外部事件處理 | 每個執行個體每秒 50 個事件 |

> [!NOTE]
> 這些數字是 Durable Functions 擴充功能 v1.4.0 (GA) 版的現行資料。 隨著功能更臻成熟及最佳化的進行，這些數字可能會隨時間改變。

如果您未看見您預期的輸送量數字，而且您的 CPU 和記憶體使用狀況似乎良好，請查看原因是否與[儲存體帳戶的健康情況](../storage/common/storage-monitoring-diagnosing-troubleshooting.md#troubleshooting-guidance)相關。 Durable Functions 擴充功能可能為 Azure 儲存體帳戶帶來大量負載，而極高的負載可能導致儲存體帳戶節流。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [安裝 Durable Functions 擴充功能和範例](durable-functions-install.md)
