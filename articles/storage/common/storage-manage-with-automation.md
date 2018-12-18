---
title: 使用 Azure 自動化管理 Azure 儲存體
description: 了解如何使用 Azure 自動化服務大規模地管理 Azure 儲存體。
services: storage, automation
author: jodoglevy
ms.service: storage
ms.topic: article
ms.date: 05/23/2016
ms.author: eamono
ms.component: common
ms.openlocfilehash: 82ec929c8d3055187a83179432fc601baa191cc4
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526363"
---
# <a name="managing-azure-storage-using-azure-automation"></a>使用 Azure 自動化管理 Azure 儲存體
本指南將為您介紹 Azure 自動化服務，以及如何使用它來簡化 Azure 儲存體 Blob、資料表及佇列的管理。

## <a name="what-is-azure-automation"></a>什麼是 Azure 自動化？
[Azure 自動化](https://azure.microsoft.com/services/automation/) 是一項 Azure 服務，可經由程序自動化簡化雲端管理。 透過 Azure 自動化，長時間執行、手動、容易發生錯誤和經常重複的工作都可以自動化，以提高可靠性、效率，並為您的組織縮短創造價值時程。

Azure 自動化提供非常可靠且高度可用的工作流程執行引擎，可隨著組織的成長根據您的需求進行調整。 在 Azure 自動化中，程序可透過手動方式、經由協力廠商系統，或依照排程的間隔啟動，讓工作精準地在需要時執行。

將您的雲端管理工作交由「Azure 自動化」自動執行，以降低營運負擔並釋出 IT/開發維運人力，使其專注於能夠為企業創造價值的工作上。

## <a name="how-can-azure-automation-help-manage-azure-storage"></a>Azure 自動化為何有助於管理 Azure 儲存體？
您可以透過使用 [Azure PowerShell](https://msdn.microsoft.com/library/azure/jj156055.aspx)中提供的 PowerShell Cmdlet，於 Azure 自動化中管理 Azure 儲存體。 Azure 自動化的這些 儲存體 PowerShell Cmdlet 都是內建的，以便您在服務內執行所有 Blob、資料表及佇列管理工作。 您也可以在 Azure 自動化中將這些 Cmdlet 與其他 Azure 服務的 Cmdlet 配對，將跨 Azure 服務和協力廠商系統的複雜工作自動化。

[Azure 自動化 Runbook 資源庫](https://azure.microsoft.com/blog/2014/10/07/introducing-the-azure-automation-runbook-gallery/) 含有多種產品團隊和社群 Runbook，可供您開始針對 Azure 儲存體、其他 Azure 服務及協力廠商系統進行自動化管理。 資源庫 Runbook 包括：

* [使用自動化服務移除特定天數前的 Azure 儲存體 Blob](https://gallery.technet.microsoft.com/scriptcenter/Remove-Storage-Blobs-that-aae4b761)
* [從 Azure 儲存體下載 Blob](https://gallery.technet.microsoft.com/scriptcenter/a-Blob-from-Azure-Storage-6bc13745)
* [備份單一 Azure VM 中的所有磁碟，或雲端服務中所有 VM 的所有磁碟](https://gallery.technet.microsoft.com/scriptcenter/Backup-all-disks-for-a-ede940d5)

## <a name="next-steps"></a>後續步驟
了解 Azure 自動化的基本概念以及如何用它來管理 Azure 儲存體 Blob、資料表及佇列之後，請參考下列連結，以深入了解 Azure 自動化。

請參閱 Azure 自動化教學課程： [在 Azure 自動化中建立或匯入 Runbook](../../automation/automation-creating-importing-runbook.md)。

