---
title: Batch 轉譯功能
description: Azure Batch 中的特定轉譯功能
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: 6d79cdf279022320f654fbbeadb870f82db88cab
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392772"
---
# <a name="azure-batch-rendering-capabilities"></a>Azure Batch 轉譯功能

標準的 Azure Batch 功能可用來執行轉譯工作負載和應用程式。 Batch 也包含特定的功能來支援轉譯工作負載。

如需 Batch 概念 (包括集區、作業和工作) 的概觀，請參閱[這篇文章](https://docs.microsoft.com/azure/batch/batch-api-basics)。

## <a name="batch-pools"></a>Batch 集區

### <a name="rendering-application-installation"></a>轉譯應用程式安裝

只有在需要使用預先安裝的應用程式時，才能在集區設定中指定 Azure Marketplace 轉譯 VM 映像。

目前提供 Windows 2016 映像和 CentOS 映像。  您可以在 [Azure Marketplace](https://azuremarketplace.microsoft.com) 中，透過搜尋「batch 轉譯」來找到 VM 映像。

如需範例集區設定，請參閱 [Azure CLI 轉譯教學課程](https://docs.microsoft.com/azure/batch/tutorial-rendering-cli)。  Azure 入口網站和 Batch Explorer 提供 GUI 工具，讓您在建立集區時選取轉譯 VM 映像。  如果使用 Batch API，則在建立集區時指定 [ImageReference](https://docs.microsoft.com/rest/api/batchservice/pool/add#imagereference) 的下列屬性值：

| 發行者 | 供應項目 | SKU | 版本 |
|---------|---------|---------|--------|
| 批次 | rendering-centos73 | 轉譯 | 最新 |
| 批次 | rendering-windows2016 | 轉譯 | 最新 |

如果集區 VM 上需要額外的應用程式，則適用其他選項：

* 以標準 Marketplace 映像為基礎的自訂映像：
  * 使用此選項，可以為您的虛擬機器設定所需要的確切應用程式與特定版本。 如需詳細資訊，請參閱[使用自訂映像來建立虛擬機器的集區](https://docs.microsoft.com/azure/batch/batch-custom-images)。 Autodesk 和 Chaos Group 已分別修改 Arnold 和 V-Ray，以驗證 Azure Batch 授權服務。 請確定您的應用程式版本具有此支援，否則按使用次數付費授權將無法運作。 目前版本的 Maya 或 3ds Max 在執行無周邊 (以批次/命令列模式) 時不需要授權伺服器。 如果您不確定如何繼續使用此選項，請連絡 Azure 支援。
* [應用程式套件](https://docs.microsoft.com/azure/batch/batch-application-packages)：
  * 將使用一或多個 ZIP 檔案的應用程式檔案套件、透過 Azure 入口網站上傳，然後在集區設定中指定套件。 建立集區 VM 時，會下載 ZIP 檔案並將檔案解壓縮。
* 資源檔：
  * 應用程式檔案會上傳至 Azure Blob 儲存體，而且您要在[集區啟動工作](https://docs.microsoft.com/rest/api/batchservice/pool/add#starttask) \(英文\) 中指定檔案參考。 建立集區 VM 時，會將資源檔下載到每個 VM。

### <a name="pay-for-use-licensing-for-pre-installed-applications"></a>適用於預先安裝之應用程式的按使用次數付費授權

您需要在集區設定中指定將要使用且有授權費的應用程式。

* 在[建立集區](https://docs.microsoft.com/rest/api/batchservice/pool/add#request-body) \(英文\) 時指定 `applicationLicenses` 屬性。  您可以在字串陣列中指定下列值："vray"、"arnold"、"3dsmax"、"maya"。
* 當您指定一或多個應用程式時，會將這些應用程式的成本加入至 VM 的成本。  應用程式的價格會列在 [Azure Batch 定價頁面](https://azure.microsoft.com/pricing/details/batch/#graphic-rendering)上。

> [!NOTE]
> 如果您改為連線至授權伺服器來使用轉譯應用程式，則請勿指定 `applicationLicenses` 屬性。

您可以使用 Azure 入口網站或 Batch Explorer 來選取應用程式，並顯示應用程式價格。

如果嘗試使用某個應用程式，但該應用程式在集區設定的 `applicationLicenses` 屬性中尚未指定，或未連線到授權伺服器，則該應用程式執行就會失敗，其中會產生授權錯誤和非零的結束代碼。

### <a name="environment-variables-for-pre-installed-applications"></a>適用於預先安裝之應用程式的環境變數

若要能夠建立適用於轉譯工作的命令列，就必須指定轉譯應用程式可執行檔的安裝位置。  系統環境變數已建立於 Azure Marketplace VM 映像上，您可以改用這類環境變數，而不需指定實際路徑。  除了針對每個工作建立的[標準 Batch 環境變數](https://docs.microsoft.com/azure/batch/batch-compute-node-environment-variables)，還有下列環境變數。

|Application|應用程式可執行檔|環境變數|
|---------|---------|---------|
|Autodesk 3ds Max 2018|3dsmaxcmdio.exe|3DSMAX_2018_EXEC|
|Autodesk 3ds Max 2019|3dsmaxcmdio.exe|3DSMAX_2019_EXEC|
|Autodesk Maya 2017|render.exe|MAYA_2017_EXEC|
|Autodesk Maya 2018|render.exe|MAYA_2018_EXEC|
|Chaos Group V-Ray Standalone|vray.exe|VRAY_3.60.4_EXEC|
Arnold 2017 命令列|kick.exe|ARNOLD_2017_EXEC|
|Arnold 2018 命令列|kick.exe|ARNOLD_2018_EXEC|
|Blender|blender.exe|BLENDER_2018_EXEC|

### <a name="azure-vm-families"></a>Azure VM 系列

如同其他工作負載，轉譯應用程式系統需求不盡相同，而且對於作業和專案的效能需求也有所不同。  Azure 中根據您的需求提供各種不同的 VM 系列：最低成本、最優惠的價格/效能、最佳效能等。
某些轉譯應用程式 (例如 Arnold) 是以 CPU 為基礎；其他像是 V-ray 和 Blender Cycles 的轉譯應用程式則可使用 CPU 和/或 GPU。
如需可用 VM 系列和 VM 大小的說明，請參閱 [VM 類型和大小](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)。

### <a name="low-priority-vms"></a>低優先順序的 VM

如同其他工作負載，可以在 Batch 集區中使用低優先順序的 VM 進行轉譯。  低優先順序的 VM 執行方式與一般專用的 VM 相同，但會利用剩餘的 Azure 容量，並獲得較大的折扣。  使用低優先順序 VM 的代價是，這些 VM 可能無法用於配置，或可能隨時會有高優先順序的 VM 先佔，視可用容量而定。 基於這個理由，低優先順序的 VM 將不適用於所有轉譯作業。 例如，如果映像需要花數個小時來轉譯，則很可能因為無法接受已遭先占的 VM 而中斷那些映像的轉譯並重新啟動。

如需有關低優先順序 VM 的特性及使用 Batch 來設定它們之各種方式的詳細資訊，請參閱[使用低優先順序的 VM 搭配 Batch](https://docs.microsoft.com/azure/batch/batch-low-pri-vms)。

## <a name="jobs-and-tasks"></a>工作 (Job) 和工作 (Task)

不需要針對作業和工作提供轉譯特定的支援。  主要設定項目是工作命令列，其必須參考必要的應用程式。
使用 Azure Marketplace VM 映像時，則最佳做法是使用環境變數來指定路徑和應用程式可執行檔。

## <a name="next-steps"></a>後續步驟

如需 Batch 轉譯的範例，請試著進行以下兩個教學課程：

* [使用 Azure CLI 進行轉譯](https://docs.microsoft.com/azure/batch/tutorial-rendering-cli)
* [使用 Batch Explorer 進行轉譯](https://docs.microsoft.com/azure/batch/tutorial-rendering-batchexplorer-blender)
