---
title: 我在 Azure 自動化中的第一個 PowerShell 工作流程 Runbook
description: 本教學課程逐步引導您使用 PowerShell 工作流程建立、測試和發佈簡單的文字 Runbook。
keywords: powershell 工作流程, powershell 工作流程範例, 工作流程 powershell
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 09/24/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 90d2dcb81a6b54d1a8437f0d3ae01aa96cdac3db
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165394"
---
# <a name="my-first-powershell-workflow-runbook"></a>我的第一個 PowerShell 工作流程 Runbook

> [!div class="op_single_selector"]
> * [圖形化](automation-first-runbook-graphical.md)
> * [PowerShell](automation-first-runbook-textual-powershell.md)
> * [PowerShell 工作流程](automation-first-runbook-textual.md)
> * [Python](automation-first-runbook-textual-python2.md)

本教學課程將逐步引導您在 Azure 自動化中建立 [PowerShell 工作流程 Runbook](automation-runbook-types.md#powershell-workflow-runbooks)。 您先從測試和發行的簡單 Runbook 開始，同時說明如何追蹤 Runbook 作業的狀態。 接著您會修改 Runbook 以實際管理 Azure 資源，在此情況下會啟動 Azure 虛擬機器。 最後您藉由新增 Runbook 參數，讓 Runbook 更穩固。

## <a name="prerequisites"></a>必要條件

若要完成此教學課程，您需要下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md) ，用來保存 Runbook 以及向 Azure 資源驗證。  此帳戶必須擁有啟動和停止虛擬機器的權限。
* Azure 虛擬機器。 您會停止然後啟動這部機器，因此它不該是生產 VM。

## <a name="step-1---create-new-runbook"></a>步驟 1 - 建立新的 Runbook

您會由建立一個可輸出 Hello World 文字的簡單 Runbook 開始。

1. 在 Azure 入口網站中，開啟您的自動化帳戶。

   [自動化帳戶] 頁面提供這個帳戶中資源的快速檢視。 您應該已經有一些資產。 其中大部分是會自動包含在新自動化帳戶的模組。 您應該也擁有 [必要條件](#prerequisites)中所述的認證資產。

1. 按一下 [程序自動化] 底下的 [Runbook]，以開啟 Runbook 清單。
1. 按一下 [+ 新增 Runbook] 按鈕，然後按一下 [建立新的 Runbook] 來建立新的 Runbook。
1. 將 Runbook 命名為「MyFirstRunbook-Workflow」 。
1. 在此情況下，您要建立 [PowerShell 工作流程 Runbook](automation-runbook-types.md#powershell-workflow-runbooks)，因此請選取 [PowerShell 工作流程] 作為 [Runbook 類型]。
1. 按一下 [建立]  來建立 Runbook 並開啟文字式編輯器。

## <a name="step-2---add-code-to-the-runbook"></a>步驟 2 - 將程式碼加入至 runbook

您可以直接將程式碼輸入到 runbook 中，或從程式庫控制項選取 cmdlet、runbook 和資產，並利用任何相關的參數將它們加入至 runbook。 在此逐步解說中，您直接輸入至 Runbook。

1. 您的 Runbook 目前是空白的，只有必要的「工作流程」關鍵字、Runbook 名稱以及括住整個工作流程的大括弧。

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   }
   ```

1. 在括號之間輸入「」  。

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   Write-Output "Hello World"
   }
   ```

1. 按一下 [儲存] 來儲存 Runbook。

## <a name="step-3---test-the-runbook"></a>步驟 3 - 測試 Runbook

在您發佈 Runbook 之前，為了使其可用於生產環境，您想要測試以確定它可以正常運作。 測試 Runbook 時，您會執行其 **草稿** 版本，並以互動方式檢視其輸出。

1. 按一下 [測試窗格]  來開啟 [測試] 窗格。
1. 按一下 [開始]  以開始測試。 這應該是唯一啟用的選項。
1. 隨即會建立 [Runbook 工作](automation-runbook-execution.md) ，並顯示其狀態。

   工作狀態會從「已排入佇列」  開始，表示等候雲端中的 Runbook 背景工作可供使用。 當背景工作角色宣告該作業時，狀態會變更為 [正在開始]，然後 Runbook 真正開始執行時再變更為 [執行中]。  

1. Runbook 工作完成時，會顯示其輸出。 在您的案例中，您應該會看到「Hello World」。

   ![Hello World](media/automation-first-runbook-textual/test-output-hello-world.png)

1. 關閉 [測試] 窗格以返回畫布。

## <a name="step-4---publish-and-start-the-runbook"></a>步驟 4 - 發佈和啟動 Runbook

您建立的 Runbook 仍處於 [草稿] 模式。 您需要將它發佈，才能在生產環境中執行它。 當您發佈 Runbook 時，您會使用草稿版本覆寫現有的已發佈版本。 在您的案例中，因為您剛剛建立 Runbook，所以還沒有已發佈的版本。

1. 按一下 [發佈] 來發佈 Runbook，然後出現提示時按一下 [是]。
1. 如果您現在向左捲動以檢視 [Runbook] 窗格中的 Runbook，則畫面會顯示 [已發佈] 的 [撰寫狀態]。
1. 捲動回到右側，檢視 **MyFirstRunbook-Workflow**的窗格。  
   在頂端的選項可讓我們啟動 Runbook、排程它在未來的某個時間點啟動，或建立 [webhook](automation-webhooks.md) ，使得可以透過 HTTP 呼叫啟動它。
1. 您只想要啟動 Runbook，因此按一下 [開始]，然後在提示出現時按一下 [是]。

   ![啟動 Runbook](media/automation-first-runbook-textual/automation-runbook-controls-start.png)

1. 作業窗格會針對您所建立的 Runbook 作業開啟。 您可以關閉此窗格，但在此情況下，您會讓它開啟，以便觀看作業的進度。
1. [作業摘要] 中會顯示作業狀態，且符合當您測試 Runbook 時看到的狀態。

   ![工作摘要](media/automation-first-runbook-textual/job-pane-status-blade-jobsummary.png)

1. 一旦 Runbook 狀態顯示 [已完成]，請按一下 [輸出]。 [輸出] 窗格會開啟，而且可以看到您的「Hello World」。

   ![工作摘要](media/automation-first-runbook-textual/job-pane-status-blade-outputtile.png)  

1. 關閉 [輸出] 窗格。
1. 按一下 [所有記錄檔]  以開啟 Runbook 作業的 [資料流] 窗格。 您應該只會在輸出資料流中看到「Hello World」，但可能也會顯示 Runbook 作業的其他串流，例如 Runbook 寫入時發生的詳細資訊和錯誤。

   ![工作摘要](media/automation-first-runbook-textual/job-pane-status-blade-alllogstile.png)

1. 關閉 [資料流] 窗格和 [工作] 窗格，以返回 MyFirstRunbook 窗格。
1. 按一下 [作業]  以開啟此 Runbook 的 [工作] 窗格。 這樣會列出此 Runbook 所建立的所有工作。 由於您只執行一次作業，因此應該只會看到列出一項作業。

   ![工作](media/automation-first-runbook-textual/runbook-control-job-tile.png)

1. 您可以按一下此作業，以開啟您啟動 Runbook 時所檢視的相同 [作業] 窗格。 這可讓您回到過去的時間並檢視針對特定 Runbook 所建立的任何工作的詳細資料。

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>步驟 5 - 加入驗證來管理 Azure 資源

您已測試並發行您的 Runbook，但是到目前為止，它似乎並不實用。 您想要讓它管理 Azure 資源。 不過它無法辦到這點，除非您使用在[必要條件](#prerequisites)中提及的認證對其進行驗證。 您必須利用 **Connect-AzureRmAccount** Cmdlet 來執行。

1. 按一下 MyFirstRunbook-Workflow 窗格上的 [編輯] 以開啟文字編輯器。
2. 您不再需要 **Write-Output** 行，因此可以放心刪除。
3. 將游標放在大括弧之間的空白行。
4. 輸入或是複製並貼上下列程式碼，此程式碼會處理您的自動化執行身分帳戶的驗證︰

   ```powershell-interactive
   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID `
   -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Select-AzureRmSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID
   ```

   > [!IMPORTANT]
   > **Add-AzureRmAccount** 和 **Login-AzureRmAccount** 現在是 **Connect-AzureRMAccount** 的別名。 如果 **Connect-AzureRMAccount** Cmdlet 不存在，您可以使用 **Add-AzureRmAccount** 或 **Login-AzureRmAccount**，或者在自動化帳戶中[更新模組](automation-update-azure-modules.md)至最新版本。

> [!NOTE]
> 即使您才剛建立新的自動化帳戶，可能還是需要[更新模組](automation-update-azure-modules.md)。

1. 按一下 [測試] 窗格，您便可測試 Runbook。
1. 按一下 [開始]  以開始測試。 測試完成時，您應該會從帳戶收到如同以下顯示基本資訊的輸出。 這可確認認證有效。

   ![驗證](media/automation-first-runbook-textual/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>步驟 6 - 加入程式碼以啟動虛擬機器

由於您的 Runbook 正在驗證您的 Azure 訂用帳戶，所以您可以管理資源。 您會新增一個命令以啟動虛擬機器。 您可以在您的 Azure 訂用帳戶中挑選任何虛擬機器，而現在您會在 Runbook 中將該名稱硬式編碼。 如果您要管理橫跨多個訂用帳戶的資源，您需要使用 **-AzureRmContext** 參數搭配 [Get-azurermcontext](/powershell/module/azurerm.profile/get-azurermcontext)。

1. 在 *Connect-AzureRmAccount* 後面輸入 Start-AzureRmVM -Name 'VMName' -ResourceGroupName 'NameofResourceGroup'，提供所要啟動虛擬機器的名稱和資源群組名稱。  

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Select-AzureRmSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

   Start-AzureRmVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName' -AzureRmContext $AzureContext
   }
   ```

1. 儲存 Runbook，然後按一下 [測試] 窗格，您便能加以測試。
1. 按一下 [開始]  以開始測試。 當它完成時，請檢查虛擬機器已啟動。

## <a name="step-7---add-an-input-parameter-to-the-runbook"></a>步驟 7 - 將輸入參數加入至 Runbook

您的 Runbook 目前會啟動您在 Runbook 中硬式編碼的虛擬機器，但如果可以在啟動 Runbook 時指定虛擬機器，它會更有用。 您會將輸入參數新增至 Runbook，以提供該功能。

1. 如下列範例所示，將 [VMName] 和 [ResourceGroupName] 的參數新增至 Runbook，並搭配使用這些變數與 **Start-AzureRmVM** Cmdlet。

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
    Param(
     [string]$VMName,
     [string]$ResourceGroupName
    )  
   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
   Start-AzureRmVM -Name $VMName -ResourceGroupName $ResourceGroupName
   }
   ```

2. 儲存 Runbook 並開啟 [測試] 窗格。 您現在可以提供測試中兩個輸入變數的值。
3. 關閉 [測試] 窗格。
4. 按一下 [發佈]  來發行新版本的 Runbook。
5. 停止您在上一個步驟中啟動的虛擬機器。
6. 按一下 [開始]  以啟動 Runbook。 輸入您要啟動之虛擬機器的 [VMName] 和 [ResourceGroupName]。

   ![啟動 Runbook](media/automation-first-runbook-textual/automation-pass-params.png)

7. Runbook 完成時，請檢查虛擬機器已啟動。  

## <a name="next-steps"></a>後續步驟

* 若要開始使用圖形化 Runbook，請參閱 [我的第一個圖形化 Runbook](automation-first-runbook-graphical.md)
* 若要開始使用 PowerShell Runbook，請參閱 [我的第一個 PowerShell Runbook](automation-first-runbook-textual-powershell.md)
* 若要深入了解 Runbook 類型、其優點和限制，請參閱 [Azure 自動化 Runbook 類型](automation-runbook-types.md)
* 如需 PowerShell 指令碼支援功能的詳細資訊，請參閱 [Azure 自動化中的原生 PowerShell 指令碼支援](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/)
