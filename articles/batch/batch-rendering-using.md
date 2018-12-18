---
title: Azure Batch 轉譯
description: 如何使用 Azure Batch 轉譯功能
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: ef2abc147049d264899d227235cffe72a1a1568c
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036734"
---
# <a name="using-azure-batch-rendering"></a>使用 Azure Batch 轉譯

您可透過數種方式來使用 Azure Batch 轉譯：

* API：
  * 使用任何 Batch API 撰寫程式碼。  開發人員可將 Azure Batch 功能整合到其現有的應用程式或工作流程中，無論在雲端還是內部部署皆可。
* 命令列工具：
  * [Azure 命令列](https://docs.microsoft.com/cli/azure/)或 [PowerShell](https://docs.microsoft.com/powershell/azure/overview) 可用來編寫 Batch 適用的指令碼。
  * 特別是，[Batch CLI 範本](https://docs.microsoft.com/azure/batch/batch-cli-templates)支援大幅簡化了建立集區和提交作業的程序。
* Batch Explorer UI：
  * [Batch Explorer](https://github.com/Azure/BatchLabs) 是一種也可讓 Batch 帳戶受到管理及監視的跨平台用戶端工具。
  * 針對各種轉譯應用程式，我們提供了許多可用來輕鬆建立集區及提交作業的集區和作業範本。  應用程式 UI 中列出了範本集，以及可從 GitHub 存取的範本檔案。
  * 您可以從頭撰寫自訂範本，或是從 GitHub 複製和修改提供的範本。
* 用戶端應用程式外掛程式：
  * 外掛程式可讓使用者直接從用戶端設計和模型化應用程式內使用 Batch 轉譯。  外掛程式主要會以目前 3D 模型的相關內容資訊叫用 Batch Explorer 應用程式，並且包含有助於管理資產的功能。

試用 Azure Batch 轉譯的最佳方式，是直接使用 Batch Explorer 應用程式，或從用戶端應用程式外掛程式叫用該應用程式；這對於不是開發人員和 Azure 專家的使用者而言，也是最簡單的方式。

## <a name="using-batch-explorer"></a>使用 Batch Explorer

如需使用 Batch Explorer 執行轉譯的逐步教學課程，請參閱 [Blender 教學課程](https://docs.microsoft.com/azure/batch/tutorial-rendering-batchexplorer-blender)。

### <a name="download-and-install"></a>下載並安裝

Batch Explorer 有 Windows、OSX 和 Linux 所[適用的下載項目](https://azure.github.io/BatchExplorer/)。

### <a name="using-templates-to-create-pools-and-run-jobs"></a>使用範本建立集區並執行作業

Batch Explorer 可搭配使用完整的範本集，讓您能夠針對各種不同的轉譯應用程式輕鬆地建立集區並提交作業，而無須指定直接使用 Batch 建立集區、作業和工作所需的所有屬性。  Batch Explorer 中的可用範本會儲存並顯示在 [GitHub 存放庫](https://github.com/Azure/BatchExplorer-data/tree/master/ncj)中。

![Batch Explorer資源庫](./media/batch-rendering-using/batch-explorer-gallery.png)

所有存在於 Marketplace 轉譯 VM 映像上的應用程式都有適當的範本可供使用。  每個應用程式都有多個範本，包括適用於 CPU 和 GPU 集區、Windows 和 Linux 集區的集區範本，而作業範本則包含全畫幅或並排顯示 Blender 轉譯和 V-Ray 分散式轉譯。 提供的範本集將隨著時間而擴充，以因應其他 Batch 功能的需求，例如集區自動調整。

您也可以從頭開始或藉由修改提供的範本來產生自訂範本。 在 Batch Explorer 的 [資源庫] 區段中選取 [本機範本] 項目，即可使用自訂範本。

### <a name="file-system-and-data-movement"></a>檔案系統和資料移動

Batch Explorer 中的 [資料] 區段可用來在本機檔案系統與 Azure 儲存體帳戶之間複製檔案。

## <a name="client-application-plug-ins"></a>用戶端應用程式外掛程式

某些用戶端應用程式有適用的外掛程式。  這些外掛程式可直接從應用程式建立集區和作業，或叫用 Batch Explorer。

* [Blender](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/blender)
* [Autodesk 3ds Max](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max)
* [Autodesk Maya](https://github.com/Azure/azure-batch-maya)

## <a name="next-steps"></a>後續步驟

如需 Batch 轉譯的範例，請試著進行以下兩個教學課程：

* [使用 Azure CLI 進行轉譯](https://docs.microsoft.com/azure/batch/tutorial-rendering-cli)
* [使用 Batch Explorer 進行轉譯](https://docs.microsoft.com/azure/batch/tutorial-rendering-batchexplorer-blender)