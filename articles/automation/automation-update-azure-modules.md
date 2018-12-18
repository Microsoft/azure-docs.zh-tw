---
title: 更新 Azure 自動化中的 Azure 模組
description: 本文說明現在要如何更新 Azure 自動化中預設提供的通用 Azure PowerShell 模組。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/19/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: fbb57753117f3c60010fe910616b8d0af5178360
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434818"
---
# <a name="how-to-update-azure-powershell-modules-in-azure-automation"></a>如何更新 Azure 自動化中的 Azure PowerShell 模組

每個自動化帳戶依預設都會提供最通用的 Azure PowerShell 模組。 Azure 小組會定期更新 Azure 模組，因此在自動化帳戶中，我們會提供方法讓您在入口網站有新的版本可供使用時，更新帳戶中的模組。

因為會根據產品群組定期更新模組，所以內含 Cmdlet 可能會變更，這樣可能會根據變更類型 (例如重新命名參數，或完全淘汰 Cmdlet) 對 Runbook 造成負面影響。 若要避免影響 Runbook 以及它們所自動化的處理序，建議您先進行測試和驗證，再繼續進行。 如果您沒有適用於此用途的專用自動化帳戶，請考慮建立自動化帳戶，讓您除了更新 PowerShell 模組這類反覆變更之外，還可以在開發 Runbook 期間測試許多不同的案例和排列。 驗證結果且套用任何所需變更之後，請繼續協調任何需要修改之 Runbook 的移轉，並在生產環境中依說明所述執行以下更新。

> [!NOTE]
> 新的自動化帳戶可能不會包含最新的模組。

## <a name="updating-azure-modules"></a>更新 Azure 模組

1. 自動化帳戶的 [模組] 頁面中有一個稱為 [更新 Azure 模組] 的選項。 它一律為啟用狀態。<br><br> ![[模組] 頁面中的 [更新 Azure 模組] 選項](media/automation-update-azure-modules/automation-update-azure-modules-option.png)

  > [!NOTE]
  > 更新 Azure 模組之前，建議您先在測試用的自動化帳戶中進行更新，確保現有指令碼會如預期般運作後，再更新 Azure 模組。
  >
  > [更新 Azure 模組] 按鈕功能僅適用於公用雲端。 因此[主權區域](https://azure.microsoft.com/global-infrastructure/)並未提供。 請參閱[更新模組的替代方式](#alternative-ways-to-update-your-modules)一節，以深入了解。


2. 按一下 [更新 Azure 模組]，則會顯示確認通知，詢問您是否要繼續。<br><br> ![更新 Azure 模組通知](media/automation-update-azure-modules/automation-update-azure-modules-popup.png)

3. 按一下 [是] 就會開始模組更新程序。 更新程序大約需要 15-20 分鐘來更新下列模組︰

  * Azure
  * Azure.Storage
  * AzureRm.Automation
  * AzureRm.Compute
  * AzureRm.Profile
  * AzureRm.Resources
  * AzureRm.Sql
  * AzureRm.Storage

    如果模組已是最新狀態，此程序在幾秒鐘內便會完成。 更新程序完成時，您會收到通知。<br><br> ![更新 Azure 模組更新狀態](media/automation-update-azure-modules/automation-update-azure-modules-updatestatus.png)

    Azure 自動化中不支援 .NET Core 的 AzureRm 模組 (AzureRm.*.Core)，因此無法匯入。

> [!NOTE]
> 執行新的排程工作時，Azure 自動化會使用自動化帳戶中的最新模組。  

如果您在 Runbook 中使用來自這些 Azure PowerShell 模組的 Cmdlet，則您會想要每隔大約一個月執行一次此更新程序，以確保您擁有最新模組。 Azure 自動化使用 AzureRunAsConnection 連線以在更新模組時驗證，如果服務主體過期或不再存在於訂用帳戶層級，則模組更新將會失敗。

## <a name="alternative-ways-to-update-your-modules"></a>更新模組的替代方式

如上所述，[更新 Azure 模組] 按鈕無法在主權雲端中使用，該功能僅適用於全域 Azure 雲端。 這是因為 PowerShell 資源庫中的最新版 Azure PowerShell 模組，可能不適用於目前部署在這些雲端中的 Resource Manager 服務。

但您仍可藉由將 [Update-AzureModule.ps1](https://github.com/azureautomation/runbooks/blob/master/Utility/ARM/Update-AzureModule.ps1) Runbook 匯入到您自動化帳戶並加以執行，來更新模組。

使用 `AzureRmEnvironment` 參數可將正確的環境傳遞至 Runbook。  允許的值為 **AzureCloud**、**AzureChinaCloud**、**AzureGermanCloud** 和 **AzureUSGovernmentCloud**。 如果您沒有將值傳遞給這個參數，Runbook 會預設為 Azure 公用雲端 **AzureCloud**。

如果您想要使用特定的 Azure PowerShell 模組版本，而不是 PowerShell 資源庫上提供的最新版本，可將這些版本傳遞給 **Update-AzureModule** Runbook 的選擇性 `ModuleVersionOverrides` 參數。 如需範例，請參閱 [Update-AzureModule.ps1](https://github.com/azureautomation/runbooks/blob/master/Utility/ARM/Update-AzureModule.ps1) Runbook。 `ModuleVersionOverrides` 參數中未提及的 Azure PowerShell 模組會以 PowerShell 資源庫中最新模組版本來更新。 如果未將任何傳遞版本至 `ModuleVersionOverrides` 參數，則所有模組都會以 PowerShell 資源庫中的最新模組版本來更新，這是 [更新 Azure 模組] 按鈕的行為。

## <a name="next-steps"></a>後續步驟

* 若要深入了解整合模組，以及如何建立自訂模組以進一步整合自動化與其他系統、服務或解決方案，請參閱[整合模組](automation-integration-modules.md)。

* 請考慮使用 [GitHub Enterprise](automation-scenario-source-control-integration-with-github-ent.md) 或 [Azure DevOps](automation-scenario-source-control-integration-with-vsts.md) 的原始檔控制整合，以集中管理和控制自動化 Runbook 和設定組合的發行。  
