---
title: 檢查是否出現 Batch 集區和節點錯誤
description: 建立集區和節點時所要檢查的錯誤及如何避免
services: batch
author: mscurrell
ms.author: markscu
ms.date: 9/25/2018
ms.topic: conceptual
ms.openlocfilehash: bc09089fa9984e9042166938da19499afca21509
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47435822"
---
# <a name="checking-for-pool-and-node-errors"></a>檢查是否出現集區和節點錯誤

建立及管理 Batch 集區時，有立即發生的作業，也有非立即發生、在背景中執行及可能需要數分鐘才能完成的非同步作業。

偵測立即發生之作業的失敗相當簡單，因為 API、CLI 或 UI 會立即傳回所有失敗。

此文章涵蓋可能針對集區和集區節點發生的背景作業 - 其中會指明如何偵測失敗，以及如何避免失敗。

## <a name="pool-errors"></a>集區錯誤

### <a name="resize-timeout-or-failure"></a>調整逾時或失敗

建立新集區或調整現有集區的大小時，會指定目標節點數目。  此作業會立即完成，但新節點的實際配置或現有節點的移除會在背景中發生，時間可能長達數分鐘。  調整逾時是在[建立](https://docs.microsoft.com/rest/api/batchservice/pool/add)或[調整大小](https://docs.microsoft.com/rest/api/batchservice/pool/resize) API 中指定的 - 如果在調整逾時期間無法取得目標節點數目，作業就會停止，其中集區會進入穩定狀態而出現調整大小錯誤。

調整逾時會由上次評估的 [ResizeError](https://docs.microsoft.com/rest/api/batchservice/pool/get#resizeerror) 屬性回報，這會列出一或多個已發生的錯誤。

調整逾時的常見原因包括：
- 調整逾時太短
  - 預設逾時為 15 分鐘，通常已足以進行集區節點的配置或移除。
  - 當配置大量節點 (超過 1000 個來自 Marketplace 映像的節點，或超過 300 個來自自訂映像的節點) 時，則在進行集區建立或調整大小的期間，建議將逾時設定為 30 分鐘。
- 核心配額不足
  - Batch 帳戶針對可配置給所有集區的核心數目有配額。  當達到該配額之後，Batch 就不會配置節點。  您[可以提升](https://docs.microsoft.com/azure/batch/batch-quota-limit)核心配額來配置更多節點。
- 當[集區位於虛擬網路中](https://docs.microsoft.com/azure/batch/batch-virtual-network)時，子網路 IP 不足
  - 虛擬網路子網路必須有足夠的未指派 IP 位址可供配置給所要求的所有集區節點，否則會無法建立節點。
- 當[集區位於虛擬網路中](https://docs.microsoft.com/azure/batch/batch-virtual-network)時，資源不足
  - 資源 (負載平衡器、公用 IP 及 NSG) 會建立在用來建立 Batch 帳戶的訂用帳戶中。  這些資源的訂用帳戶配額必須足夠。
- 針對大型集區使用自訂 VM 映像
  - 使用自訂映像的大型集區可能需要較長的時間進行配置，因此可能發生調整逾時。  限制及設定建議皆於[特定文章](https://docs.microsoft.com/azure/batch/batch-custom-images)中有相關說明。 

### <a name="auto-scale-failures"></a>自動調整失敗

您可以不要明確設定集區建立或調整大小作業中集區的目標節點數目，而是以自動方式調整集區中的節點數目。  您可以[為集區建立自動調整公式](https://docs.microsoft.com/azure/batch/batch-automatic-scaling)，系統會依據一個可設定的定期間隔來評估此公式，藉此為集區設定目標節點數目。  以下是可能發生及偵測到的問題類型：

- 自動調整評估可能失敗。
- 產生的調整大小作業可能失敗並逾時。
- 自動調整公式可能有問題，導致節點目標值不正確，而調整大小發揮作用或逾時。

若要取得上次自動調整評估的相關資訊，請使用 [autoScaleRun](https://docs.microsoft.com/rest/api/batchservice/pool/get#autoscalerun) 屬性，此屬性會回報評估的時間、評估的值與結果，以及執行評估時發生的任何錯誤。

至於所有評估的相關資訊，則會由[集區調整大小完成事件](https://docs.microsoft.com/azure/batch/batch-pool-resize-complete-event)自動擷取。

### <a name="delete"></a>刪除

假設集區中有節點，然後某個集區刪除作業會導致先刪除節點，接著再刪除集區物件本身。  刪除這些節點可能需要幾分鐘的時間。

[集區狀態](https://docs.microsoft.com/rest/api/batchservice/pool/get#poolstate)在刪除過程中將設定為 'deleting' \(刪除中\)。  呼叫端應用程式可以使用 'state' 和 'stateTransitionTime' 屬性來偵測集區刪除作業是否耗費太多時間。

## <a name="pool-compute-node-errors"></a>集區計算節點錯誤

節點可能順利配置在集區中，但可能有各種問題導致節點狀況不良而無法供使用。  在將節點配置在集區中之後，它們會產生費用，因此偵測問題以避免支付無用節點費用相當重要。

### <a name="start-task-failure"></a>開始工作失敗

您可以視需要為集區指定[開始工作](https://docs.microsoft.com/rest/api/batchservice/pool/add#starttask)。  與任何工作一樣，可以指定命令列和要從儲存體下載的資源檔案。  開始工作是為集區指定的，但會在每個節點上執行 - 啟動每個節點之後，就會執行開始工作。  [開始工作](https://docs.microsoft.com/rest/api/batchservice/pool/add#starttask)有一個進一步的屬性 'waitForSuccess'，此屬性會指定 Batch 是否應該先等候開始工作順利完成再為節點排定任何工作。

如果開始工作失敗，且開始工作設定已指定為等候順利完成，則節點將無法供使用且仍將產生費用。

若要偵測開始工作失敗，可以使用最上層 [startTaskInfo](https://docs.microsoft.com/rest/api/batchservice/computenode/get#starttaskinformation) 節點屬性的 [result](https://docs.microsoft.com/rest/api/batchservice/computenode/get#taskexecutionresult) 和 [failureInfo](https://docs.microsoft.com/rest/api/batchservice/computenode/get#taskfailureinformation) 屬性。

失敗的開始工作也會導致節點[狀態](https://docs.microsoft.com/rest/api/batchservice/computenode/get#computenodestate)被設定為 'starttaskfailed'，但前提是 'waitForSuccess' 已設定為 'true'。

與任何工作一樣，開始工作失敗的原因可能有許多個。  若要進行疑難排解，應檢查 stdout、stderr 及任何進一步的工作特定記錄檔。

### <a name="application-package-download-failure"></a>應用程式套件下載失敗

您可以視需要為集區指定一或多個應用程式套件，其中指定的套件檔案會下載至每個節點，並在節點啟動後但在排定工作之前解壓縮。  將開始工作命令列與應用程式套件搭配使用是常見的做法，例如，用來將檔案複製到不同的位置或執行安裝程式。

節點的 [errors](https://docs.microsoft.com/rest/api/batchservice/computenode/get#computenodeerror)屬性會回報應用程式套件的下載和解壓縮失敗。  節點狀態將設定為 'unusable' \(無法使用\)。

### <a name="node-in-unusable-state"></a>處於無法使用狀態的節點

[節點狀態](https://docs.microsoft.com/rest/api/batchservice/computenode/get#computenodestate)可能因許多原因而設定為 'unusable' \(無法使用\)。  當節點處於 'unusable' \(無法使用\) 狀態時，無法為節點排定工作，但節點仍將產生費用。

Batch 一律會嘗試復原無法使用的節點，但視原因而定，可能可以進行復原，也可能無法進行復原。

在可判斷原因的情況下，節點 [errors](https://docs.microsoft.com/rest/api/batchservice/computenode/get#computenodeerror) 屬性會回報原因。

'unusable' \(無法使用\) 節點的一些其他範例：

- 自訂映像無效；例如，未以正確方式準備。
- 基礎結構失敗或低階升級導致基礎 VM 被移動；Batch 將會復原節點。

### <a name="node-agent-log-files"></a>節點代理程式記錄檔

如果有必要連絡支援人員來解決集區節點問題，則可以取得來自在每個集區節點上執行之 Batch 代理程式程序的記錄檔。  您可以透過 Azure 入口網站、Batch Explorer 或 [API](https://docs.microsoft.com/rest/api/batchservice/computenode/uploadbatchservicelogs)來上傳節點的記錄檔。  上傳及儲存記錄檔極為有用，因為如此一來便可將節點或集區刪除，以節省執行中節點的成本。

## <a name="next-steps"></a>後續步驟

請確定您的應用程式已實作完整的錯誤檢查 (特別是針對非同步作業)，以便迅速偵測出問題並進行診斷。
