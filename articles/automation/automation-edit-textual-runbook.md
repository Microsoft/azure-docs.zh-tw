---
title: 在 Azure 自動化中編輯文字式 Runbook
description: 本文提供使用文字式編輯器在 Azure 自動化中使用 PowerShell 和 PowerShell 工作流程 Runbook 的不同程序。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 08/01/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: b17fc82d6e9cbda6ffa94ac2ee5c97835b089a7e
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39399714"
---
# <a name="editing-textual-runbooks-in-azure-automation"></a>在 Azure 自動化中編輯文字式 Runbook

Azure 自動化中的文字式編輯器可以用來編輯 [PowerShell Runbook](automation-runbook-types.md#powershell-runbooks) 和 [PowerShell 工作流程 Runbook](automation-runbook-types.md#powershell-workflow-runbooks)。 它具備與其他程式碼編輯器一樣的典型功能 (例如 Intellisense 和色彩編碼)，以及額外的特殊功能以協助您存取 Runbook 通用的資源。 本文提供執行與此編輯器不同的功能的詳細步驟。

此文字式編輯器具有將 Cmdlet、資產、子 Runbook 程式碼插入 Runbook 的功能。 並非是自行輸入程式碼，您可以從可用資源的清單中選取，並且讓適當的程式碼插入到 Runbook。

Azure 自動化中的每個 Runbook 有兩個版本，「草稿」和「已發佈」。 您編輯 Runbook 的「草稿」版本然後發佈，讓它可以執行。 無法編輯「已發佈」版本。 如需詳細資訊，請參閱 [發佈 Runbook](automation-creating-importing-runbook.md#publishing-a-runbook) 。

若要使用[圖形化 Runbook](automation-runbook-types.md#graphical-runbooks)，請參閱 [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md)。

## <a name="to-edit-a-runbook-with-the-azure-portal"></a>使用 Azure 入口網站編輯 Runbook

使用下列程序以開啟 Runbook，在文字式編輯器中進行編輯。

1. 在 Azure 入口網站中，選取您的自動化帳戶。
2. 在 [程序自動化] 底下選取 [Runbook] 開啟 Runbook 清單。
3. 選取您要編輯的 Runbook，然後按一下 [編輯] 按鈕。
4. 執行必要的編輯。
5. 當您完成編輯時，按一下 [儲存]  。
6. 如果您要發佈 Runbook 的最新草稿版本，請按一下 [發佈]  。

### <a name="to-insert-a-cmdlet-into-a-runbook"></a>將 Cmdlet 插入 Runbook

1. 在文字式編輯器的 [畫布] 中，將游標移至您要放置 Cmdlet 的位置。
2. 在程式庫控制項中，展開 **Cmdlet** 節點。
3. 展開包含您想要使用的 Cmdlet 的模組。
4. 以滑鼠右鍵按一下要插入的 Cmdlet，然後選取 [新增至畫布] 。 如果 Cmdlet 有一個以上的參數集合，會新增預設集合。 您也以展開 Cmdlet 以選取不同的參數集合。
5. 會插入 Cmdlet 的程式碼且具有參數的完整清單。
6. 對於任何必要的參數，提供適當的值來取代以大括號 <> 括住的資料類型。 移除您不需要的任何參數。

### <a name="to-insert-code-for-a-child-runbook-into-a-runbook"></a>將子 Runbook 的程式碼插入 Runbook

1. 在文字式編輯器的 [畫布] 中，將游標移至您要放置 [子 Runbook](automation-child-runbooks.md)程式碼的位置。
2. 在程式庫控制項中，展開 **Runbook** 節點。
3. 以滑鼠右鍵按一下要插入的 Runbook，然後選取 [新增至畫布] 。
4. 會插入子 Runbook 的程式碼且具有任何 Runbook 參數的預留位置。
5. 針對每個參數以適當值取代預留位置。

### <a name="to-insert-an-asset-into-a-runbook"></a>將資產插入 Runbook

1. 在文字式編輯器的 [畫布] 中，將游標移至您要放置子 Runbook 程式碼的位置。
2. 在程式庫控制項中，展開 [資產]  節點。
3. 展開您想要的資產類型的節點。
4. 以滑鼠右鍵按一下要插入的資產，然後選取 [新增至畫布] 。 針對[變數資產](automation-variables.md)，根據您要取得或設定變數而定，選取 [將「取得變數」加入畫布] 或 [將「設定變數」加入畫布]。
5. 資產的程式碼會插入至 Runbook。

## <a name="to-edit-an-azure-automation-runbook-using-windows-powershell"></a>使用 Windows PowerShell 編輯 Azure 自動化 Runbook

若要使用 Windows PowerShell 編輯 Runbook，請使用您選擇的編輯器，然後將它儲存為 .ps1 檔案。 您可以使用 [Export-AzureRmAutomationRunbook](/powershell/module/AzureRM.Automation/Export-AzureRmAutomationRunbook) Cmdlet 來擷取 Runbook 的內容，然後使用 [Import-AzureRmAutomationRunbook](/powershell/module/AzureRM.Automation/import-azurermautomationrunbook) Cmdlet，以修改過的 Runbook 取代現有的草稿 Runbook。

### <a name="to-retrieve-the-contents-of-a-runbook-using-windows-powershell"></a>使用 Windows PowerShell 擷取 Runbook 的內容

以下範例命令顯示如何擷取 Runbook 的指令碼，然後將其儲存至指令碼檔案。 在此範例中，會擷取「草稿」版本。 也可以擷取「已發佈」版本的 Runbook，雖然這個版本不能變更。

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Export-AzureRmAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -OutputFolder $scriptFolder -Slot Draft
```

### <a name="to-change-the-contents-of-a-runbook-using-windows-powershell"></a>使用 Windows PowerShell 變更 Runbook 的內容

下列範例命令顯示如何以指令碼檔案內容取代現有 Runbook 的內容。 請注意，這個範例程序與[使用 Windows PowerShell 從指令碼檔案匯入 Runbook](automation-creating-importing-runbook.md) 的相同。

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Import-AzureRmAutomationRunbook -Path "$scriptfolder\Hello-World.ps1" -Name $runbookName -Type PowerShell -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -Force
Publish-AzureRmAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName
```

## <a name="related-articles"></a>相關文章

* [在 Azure 自動化中建立或匯入 Runbook](automation-creating-importing-runbook.md)
* [了解 PowerShell 工作流程](automation-powershell-workflow.md)
* [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md)
* [Certificates](automation-certificates.md)
* [連線](automation-connections.md)
* [認證](automation-credentials.md)
* [排程](automation-schedules.md)
* [變數](automation-variables.md)
