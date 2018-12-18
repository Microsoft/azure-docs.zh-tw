---
title: Azure Batch 轉譯概觀
description: 使用 Azure 進行轉譯的簡介及 Azure Batch 轉譯功能的概觀
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: fc26e1d32332bb0ed9624b7442e38ea79b7bfb1d
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47393548"
---
# <a name="rendering-using-azure"></a>使用 Azure 進行轉譯

轉譯是取用 3D 模型並將其轉換成 2D 影像的程序。 3D 場景檔案可在 Autodesk 3ds Max、Autodesk Maya 和 Blender 等應用程式中撰寫。  Autodesk Maya、Autodesk Arnold、Chaos Group V-Ray 和 Blender Cycles 等轉譯應用程式可產生 2D 影像。  有時候，會從場景檔案建立單一映像。 不過，一般通常會建立多個影像的模型並進行轉譯，然後將其結合在動畫中。

媒體和娛樂業的特效 (VFX) 會大量使用轉譯工作負載。 其他如廣告、零售、石油和天然氣及製造等許多產業，也會使用轉譯。

轉譯的程序會耗用大量運算資源；其間可能會產生許多畫面格/影像，且每個影像的轉譯可能都會耗時數小時。  因此，轉譯非常適合使用批次處理工作負載，利用 Azure 和 Azure Batch 以平行方式執行許多轉譯。

## <a name="why-use-azure-for-rendering"></a>為何要使用 Azure 進行轉譯？

基於眾多原因，轉譯是非常適合使用 Azure 和 Azure Batch 的工作負載：

* 轉譯作業可以分割成許多可使用多個 VM 以平行方式執行的部分：
  * 動畫由許多畫面格組成，且每個畫面格可以平行方式轉譯。  可用來處理每個畫面格的 VM 愈多，所有畫面格和動畫的產生速度就愈快。
  * 某些轉譯軟體可讓單一畫面格分成多個部分，例如圖格或配量。  每個部分可個別進行轉譯，然後在每個部分都完成後再結合為最終影像。  可用的 VM 愈多，轉譯畫面格的速度就愈快。
* 轉譯專案可能需要大幅的調整：
  * 即使使用高階硬體，個別畫面格仍可能十分複雜而需要數小時進行轉譯，而動畫可能包含數十萬個畫面格。  若要在合理的時間內呈現高品質的動畫，必須經過大量計算。  在某些情況下，用來以平行方式轉譯數千個畫面格的核心，會超過 100,000 個。
* 轉譯專案以專案為基礎，且需要不同的計算量：
  * 在必要時配置計算和儲存體容量、根據專案期間的負載將其相應增加或相應減少，並在專案完成後將它移除。
  * 在容量配置時支付其費用，但在沒有負載時無須付費，例如在專案間的空窗期。
  * 因應非預期的變更所產生的高載；如果在專案晚期才出現非預期的變更，且這些變更需要以緊迫的時程處理，請進行擴充調整。
* 根據應用程式、工作負載和時間範圍選擇各式各樣的硬體：
  * 在 Azure 中可選擇使用多種可透過 Batch 來配置和管理的硬體。
  * 視專案的不同，最佳價格/效能或最佳整體效能會有不同的需求。  不同場景和/或轉譯應用程式會有不同的記憶體需求。  某些轉譯應用程式可利用 GPU 來達到最佳效能或執行特定功能。 
* 低優先順序 VM 可降低成本：
  * 優先順序 VM 可用遠低於一般隨選 VM 的價格取得，適用於某些作業類型。
  * 低優先順序 VM 可由 Azure Batch 配置，因為 Batch 讓 VM 可透過彈性使用因應各種不同的需求。  Batch 集區可同時包含專用和低優先順序 VM，並且可隨時變更 VM 類型的混用方式。

## <a name="options-for-rendering-on-azure"></a>Azure 上的轉譯選項

有許多 Azure 功能可用於轉譯工作負載。  應使用哪些功能，取決於現有的環境和需求。

### <a name="existing-on-premises-rendering-environment-using-a-render-management-application"></a>使用轉譯管理應用程式的現有內部部署轉譯環境

最常見的案例，是要以 PipelineFX Qube、Royal Render 或 Thinkbox Deadline 等轉譯管理應用程式來管理現有內部部署轉譯伺服器陣列的環境。  其需求是必須使用 Azure VM 擴充內部部署轉譯伺服器陣列容量。

轉譯管理軟體可能內建有 Azure 支援，或者，我們可以啟用新增 Azure 支援的外掛程式。 如需與支援的轉譯管理員和啟用的功能有關的詳細資訊，請參閱[使用轉譯管理員](https://docs.microsoft.com/azure/batch/batch-rendering-render-managers)的相關文章。

### <a name="custom-rendering-workflow"></a>自訂轉譯工作流程

需求是 VM 必須擴充現有的轉譯伺服器陣列。  Azure Batch 集區可以配置大量 VM，而得以使用低優先順序 VM 以及動態自動調整原定價格的 VM，並為常用的轉譯應用程式提供按使用付費的授權。

### <a name="no-existing-render-farm"></a>沒有現有的轉譯伺服器陣列

用戶端工作站可以執行轉譯，但轉譯工作負載會增加，且單獨使用工作站容量會耗時過久。  Azure Batch 可用來配置隨需的轉譯伺服器陣列計算，以及排程 Azure 轉譯伺服器陣列的轉譯作業。

## <a name="azure-batch-rendering-capabilities"></a>Azure Batch 轉譯功能

Azure Batch 可讓您在 Azure 中執行平行工作負載。  它可用來建立和管理要安裝並執行應用程式的大量 VM。  它也提供完整的作業排程功能，用以執行這些應用程式的執行個體，進而提供將工作指派給 VM、佇列、應用程式監視等功能。

Azure Batch 可用於許多工作負載，但下列功能是專為簡化和加速轉譯工作負載的執行而設計的。

* 已預先安裝圖形和轉譯應用程式的 VM 映像：
  * 您可以使用包含常用圖形和轉譯應用程式的 Azure Marketplace VM 映像，如此即無須自行安裝的應用程式，或使用已安裝的應用程式建立您自己的自訂映像。 
* 轉譯應用程式按使用次數付費的授權：
  * 除了支付計算 VM 的費用以外，您也可以選擇依分鐘數支付應用程式的費用，如此即無須購買授權或設定應用程式的授權伺服器。  按使用量付費意味著能夠因應變動和非預期的負載，因為並沒有固定的授權數。
  * 您也可以憑藉自己的授權使用預先安裝的應用程式，而不使用按使用次數付費的授權。 若要這樣做，通常您會安裝一個內部部署或 Azure 型授權伺服器，然後使用 Azure 虛擬網路將轉譯集區連線至授權伺服器。
* 適用於用戶端設計和模型化應用程式的外掛程式：
  * 外掛程式可讓使用者直接從用戶端應用程式 (例如 Autodesk Maya) 使用 Azure Batch，進而能夠建立集區、提交作業，以及使用更多計算容量來執行更快速的轉譯。
* 轉譯管理員整合：
  * Azure Batch 可以整合至轉譯管理應用程式中，或者，您可以使用外掛程式提供 Azure Batch 整合功能。

您可透過數種方式來使用 Azure Batch，而這些方法也都適用於 Azure Batch 轉譯。

* API：
  * 使用 [REST](https://docs.microsoft.com/rest/api/batchservice)、[.NET](https://docs.microsoft.com/dotnet/api/overview/azure/batch)、[Python](https://docs.microsoft.com/python/api/overview/azure/batch)、[Java](https://docs.microsoft.com/java/api/overview/azure/batch) 或其他支援的 API 撰寫程式碼。  開發人員可將 Azure Batch 功能整合到其現有的應用程式或工作流程中，無論在雲端還是內部部署皆可。  例如，[Autodesk Maya 外掛程式](https://github.com/Azure/azure-batch-maya)可利用 Batch Python API 來叫用 Batch、建立和管理集區、提交作業和工作，以及監視狀態。
* 命令列工具：
  * [Azure 命令列](https://docs.microsoft.com/cli/azure/)或 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview) 可用來編寫 Batch 適用的指令碼。
  * 特別是，Batch CLI 範本支援大幅簡化了建立集區和提交作業的程序。
* UI：
  * [Batch Explorer](https://github.com/Azure/BatchExplorer) 是一種跨平台的用戶端工具，不僅也可讓 Batch 帳戶受到管理及監視，還提供比 Azure 入口網站 UI 更豐富的功能。  我們提供了針對各種支援的應用程式而設計的集區和作業範本集，可用來輕鬆建立集區及提交作業。
  * Azure 入口網站可用來管理和監視 Azure Batch。
* 用戶端應用程式外掛程式：
  * 外掛程式可讓使用者直接從用戶端設計和模型化應用程式內使用 Batch 轉譯。 外掛程式主要會以目前 3D 模型的相關內容資訊叫用 Batch Explorer 應用程式。
  * 可用的外掛程式如下：
    * [Azure Batch for Maya](https://github.com/Azure/azure-batch-maya)
    * [3ds Max](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max)
    * [Blender](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/blender)

## <a name="getting-started-with-azure-batch-rendering"></a>開始使用 Azure Batch 轉譯

請參閱下列簡介教學課程，開始試用 Azure Batch 轉譯：

* [使用 Batch Explorer 轉譯 Blender 場景](https://docs.microsoft.com/azure/batch/tutorial-rendering-batchexplorer-blender)
* [使用 Batch CLI 轉譯 Autodesk 3ds Max 場景](https://docs.microsoft.com/azure/batch/tutorial-rendering-cli)

## <a name="next-steps"></a>後續步驟

在[這篇文章](https://docs.microsoft.com/azure/batch/batch-rendering-applications)中確認 Azure Marketplace VM 映像包含的轉譯應用程式和版本清單。