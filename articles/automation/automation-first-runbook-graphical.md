---
title: 我在 Azure 自動化中的第一個圖形化 Runbook
description: 教學課程將逐步引導您建立、測試和發佈簡單的圖形化 Runbook。
keywords: runbook, runbook 範本, runbook 自動化, azure runbook
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 04/13/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 6a9e6bbf3bc507a4d45af03c791104c9490cd36f
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34195891"
---
# <a name="my-first-graphical-runbook"></a>我的第一個圖形化 Runbook

> [!div class="op_single_selector"]
> * [圖形化](automation-first-runbook-graphical.md)
> * [PowerShell](automation-first-runbook-textual-powershell.md)
> * [PowerShell 工作流程](automation-first-runbook-textual.md)
> * [Python](automation-first-runbook-textual-python2.md)
> 

本教學課程將逐步引導您在 Azure 自動化中建立 [圖形化 Runbook](automation-runbook-types.md#graphical-runbooks) 。 您先從測試和發佈的簡單 Runbook 開始，同時了解如何追蹤 Runbook 作業的狀態。 接著您會修改 Runbook 以實際管理 Azure 資源，在此情況下會啟動 Azure 虛擬機器。 接著您會新增 Runbook 參數和條件式連結，讓 Runbook 更加穩固，藉此完成本教學課程。

## <a name="prerequisites"></a>先決條件

若要完成此教學課程，您需要下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂戶權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或註冊[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。
* [自動化帳戶](automation-offering-get-started.md) ，用來保存 Runbook 以及向 Azure 資源驗證。 此帳戶必須擁有啟動和停止虛擬機器的權限。
* Azure 虛擬機器。 您會停止並啟動這部電腦，因此它不該是生產 VM。

## <a name="create-runbook"></a>建立 Runbook

您會由建立一個可輸出文字 Hello World 的簡單 Runbook 開始。

1. 在 Azure 入口網站中，開啟您的自動化帳戶。

   [自動化帳戶] 頁面提供這個帳戶中資源的快速檢視。 您應該已經有一些資產。 其中大部分資產是自動包含在新自動化帳戶的模組。 您應該也擁有 [必要條件](#prerequisites)中所述的認證資產。

2. 選取 [程序管理] 之下的 [Runbook]，以開啟 Runbook 清單。
3. 選取 [+新增 Runbook]，然後按一下 [建立新的 Runbook]，來建立新的 Runbook。
4. 為 Runbook 提供名稱「MyFirstRunbook-Graphical」 。
5. 在此情況下，您要建立[圖形化 Runbook](automation-graphical-authoring-intro.md)，因此請選取 [圖形] 作為 [Runbook 類型]。<br> ![新的 Runbook](media/automation-first-runbook-graphical/create-new-runbook.png)<br>
6. 按一下 [建立]  來建立 Runbook 並開啟圖形化編輯器。

## <a name="add-activities"></a>新增活動

編輯器左邊的 [程式庫] 控制項可讓您選取要加入到 Runbook 的活動。 您將會新增 **Write-Output** Cmdlet，以從 Runbook 輸出文字。

1. 在 [程式庫] 控制項中，按一下搜尋文字方塊並輸入 **Write-Output**。 搜尋結果顯示於下面的影像中： <br> ![Microsoft.PowerShell.Utility](media/automation-first-runbook-graphical/search-powershell-cmdlet-writeoutput.png)
1. 向下捲動到清單底部。 您可以用滑鼠右鍵按一下 [Write-Output] 並選取 [新增至畫布]，或按一下 Cmdlet 旁邊的省略符號，然後選取 [新增至畫布]。
1. 按一下畫布上的 **Write-Output** 活動。 此動作會開啟可讓您設定活動的 [設定] 控制項頁面。
1. [標籤] 會預設為 Cmdlet 的名稱，但您可以將它變更為比較好記的名稱。 將它變更為「Write Hello World to output」 。
1. 按一下 [參數] 來提供 Cmdlet 的參數值。

   某些 Cmdlet 有多個參數集，並且您必須選取您要使用的參數。 在此情況下，**Write-Output** 有只有一個參數集，因此您不需要選取其中一個。

1. 選取 **InputObject** 參數。 這是您在其中指定要傳送至輸出資料流之文字的參數。
1. 在 [資料來源] 下拉式清單中，選取 [PowerShell 運算式]。 [資料來源]  下拉式清單提供您用來填入參數值的不同來源。

   您可以使用來自這類來源的輸出，例如另一個活動、自動化資產或 PowerShell 運算式。 在此情況下，輸出就是 Hello World。 您可以使用 PowerShell 運算式，並指定字串。<br>

1. 在 [運算式] 方塊中，輸入 "Hello World"，然後按一下 [確定] 兩次以返回畫布。
1. 按一下 [儲存] 來儲存 Runbook。

## <a name="test-the-runbook"></a>測試 Runbook

在您發佈 Runbook 之前，為了使其可用於生產環境，您想要測試以確定它可以正常運作。 測試 Runbook 時，您會執行其 **草稿** 版本，並以互動方式檢視其輸出。

1. 選取 [測試窗格] 來開啟 [測試] 頁面。
1. 按一下 [開始]  以開始測試。 這應該是唯一啟用的選項。
1. 隨即會建立 [Runbook 工作](automation-runbook-execution.md) ，並在窗格中顯示其狀態。

   作業狀態一開始為「已排入佇列」，表示正在等候雲端中的 Runbook 背景工作可供使用。 然後當背景工作宣告該作業時，狀態會變更為 [正在開始]，然後 Runbook 實際開始執行時再變更為 [執行中]。

1. Runbook 工作完成時，會顯示其輸出。 在此情況下，您會看到 Hello World。<br> ![Hello World](media/automation-first-runbook-graphical/runbook-test-results.png)
1. 關閉 [測試] 頁面以返回畫布。

## <a name="publish-and-start-the-runbook"></a>發佈和啟動 Runbook

您建立的 Runbook 仍處於 [草稿] 模式。 您需要將它發佈，才能在生產環境中執行它。 當您發佈 Runbook 時，您會使用草稿版本覆寫現有的已發佈版本。 在此情況下，因為您剛剛建立 Runbook，所以還沒有已發佈的版本。

1. 選取 [發佈] 來發佈 Runbook，然後在系統提示時選取 [是]。
1. 如果您向左捲動以檢視 [Runbook] 頁面中的 Runbook，則會顯示 [已發佈] 的 [撰寫狀態]。
1. 捲回右側，以檢視 **MyFirstRunbook-Graphical** 的頁面。

   在頂端的選項可讓我們啟動 Runbook、排程它在未來的某個時間點啟動，或建立 [webhook](automation-webhooks.md) ，使得可以透過 HTTP 呼叫啟動它。

1. 選取 [開始]，然後在系統提示時選取 [是]，以啟動 Runbook。
1. 會針對已建立的 Runbook 作業開啟作業頁面。 確認 [作業狀態] 顯示 [已完成]。
1. 一旦 Runbook 狀態顯示 [已完成]，請按一下 [輸出]。 隨即開啟 [輸出] 頁面，而您可以在窗格中看到 Hello World。
1. 關閉 [輸出] 頁面。
1. 按一下 [All Logs] \(所有記錄)，以開啟 Runbook 作業的 [資料流] 頁面。 您應該只會在輸出資料流中看到 Hello World ，但可能也會顯示 Runbook 作業的其他資料流，例如 Runbook 寫入時發生的詳細資訊和錯誤。
1. 關閉 [All Logs] \(所有記錄) 頁面和 [作業] 頁面，以返回 MyFirstRunbook-Graphical 頁面。
1. 若要檢視 Runbook 的所有作業，請關閉 [作業] 頁面，然後選取 [資源] 下方的 [作業]。 這樣會列出此 Runbook 所建立的所有作業。 由於您只執行一次作業，因此應該只會看到列出一項作業。
1. 您可以按一下此作業，以開啟您啟動 Runbook 時所檢視的相同 [作業] 窗格。 這可讓您回到過去的時間並檢視針對特定 Runbook 所建立的任何工作的詳細資料。

## <a name="create-variable-assets"></a>建立變數資產

您已測試並發行您的 Runbook，但是到目前為止，它似乎並不實用。 您想要讓它管理 Azure 資源。 在設定要驗證的 Runbook 之前，您會建立變數來保存訂用帳戶識別碼，並在設定要在下面的步驟 6 進行驗證的活動後參考此變數。 包括訂用帳戶內容的參考，可讓您輕鬆地處理多個訂用帳戶。 在繼續之前，請從 [導覽] 窗格上的 [訂用帳戶] 選項複製您的訂用帳戶識別碼。

1. 在 [自動化帳戶] 頁面中，選取 [共用資源] 下方的 [變數]。
1. 選取 [新增變數]。
1. 在 [新增變數] 頁面中，於 [名稱] 方塊中輸入 **AzureSubscriptionId**，並於 [值] 方塊中輸入您的訂用帳戶識別碼。 讓 [類型] 保持是 [字串]，並讓 [加密] 保持預設值。
1. 按一下 [建立]  來建立變數。

## <a name="add-authentication"></a>新增驗證

現在您已擁有用來保存訂用帳戶識別碼的變數，接下來您可以設定 Runbook，以使用 [必要條件](#prerequisites)中提到的「執行身分」認證來進行驗證。 您的做法是將 [Azure 執行身分] 連線 **Asset** 和 **Connect-AzureRmAccount** Cmdlet 新增至畫布。

1. 巡覽回您的 Runbook，並在 MyFirstRunbook-Graphical 頁面上選取 [編輯]。
1. 您不再需要 [Write Hello World to output] \(撰寫 Hello World 到輸出)，因此請按一下省略符號 (...)，並選取 [刪除]。
1. 在 [程式庫] 控制項中，展開 [資產]、[連線] ，並選取 [新增至畫布] 以將 [AzureRunAsConnection] 新增至畫布。
1. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Connect-AzureRmAccount**。

   > [!IMPORTANT]
   > **Add-AzureRmAccount** 現在是 **Connect-AzureRMAccount** 的別名。 搜尋您的程式庫項目時，如果沒有看到 **Connect-AzureRMAccount**，便可以使用 **Add-AzureRmAccount**，或是在自動化帳戶中更新模組。

1. 將 **Connect-AzureRmAccount** 加入至畫布。
1. 將滑鼠停留在 [取得執行身分連線]，直到圖形的底端出現圓形。 按一下圓形，並將箭頭拖曳到 **Add-AzureRmAccount**。 您剛才建立的箭號是「連結」。 Runbook 會開始使用 [取得執行身分連線]，然後執行 **Connect-AzureRmAccount**。<br> ![建立活動之間的連結](media/automation-first-runbook-graphical/runbook-link-auth-activities.png)
1. 在畫布上選取 **Connect-AzureRmAccount**，然後在 [組態] 控制項窗格的 [標籤] 文字方塊中輸入 [登入 Azure]。
1. 按一下 [參數]，隨即會顯示 [Activity Parameter Configuration] \(活動參數設定) 頁面。
1. **Connect-AzureRmAccount** 有多個參數集，因此您必須先選取一個參數，才可以提供參數值。 按一下 [參數集]，然後選取 [ServicePrincipalCertificate] 參數集。
1. 選取參數集之後，參數就會顯示在 [Activity Parameter Configuration] \(活動參數設定) 頁面中。 按一下 [APPLICATIONID]。<br> ![加入 Azure RM 帳戶參數](media/automation-first-runbook-graphical/Add-AzureRmAccount-params.png)
1. 在 [參數值] 頁面中，針對 [資料來源] 選取 [活動輸出]，並從清單中選取 [Get Run As Connection] \(取得執行身分連線)，再於 [欄位路徑] 文字方塊中鍵入 **ApplicationId**，然後按一下 [確定]。 您會指定欄位路徑之屬性的名稱，因為活動會輸出具有多個屬性的物件。
1. 按一下 [CERTIFICATETHUMBPRINT]，然後在 [參數值] 頁面中，針對 [資料來源] 選取 [活動輸出]。 從清單中選取 [取得執行身分連線]，在 [欄位路徑] 文字方塊中輸入 **CertificateThumbprint**，然後按一下 [確定]。
1. 按一下 [SERVICEPRINCIPAL]，並在 [參數值] 頁面中，針對 [資料來源] 選取 [ConstantValue]，再按一下選項 [True]，然後按一下 [確定]。
1. 按一下 [TENANTID]，然後在 [參數值] 頁面中，針對 [資料來源] 選取 [活動輸出]。 從清單中選取 [取得執行身分連線]，在 [欄位路徑] 文字方塊中輸入 **TenantId**，然後按兩次 [確定]。
1. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Set-AzureRmContext** 。
1. 將 [Set-AzureRmContext]  加入至畫布。
1. 在畫布上選取 [Set-AzureRmContext]，然後在 [組態] 控制項窗格的 [標籤] 文字方塊中輸入 [指定訂用帳戶識別碼]。
1. 按一下 [參數]，隨即會顯示 [Activity Parameter Configuration] \(活動參數設定) 頁面。
1. **Set-AzureRmContext** 有多個參數集，因此您必須先選取一個參數，才可以提供參數值。 按一下 [參數集]，然後選取 [SubscriptionId] 參數集。
1. 選取參數集之後，參數就會顯示在 [Activity Parameter Configuration] \(活動參數設定) 頁面中。 按一下 [SubscriptionID] 
1. 在 [參數值] 頁面中，針對 [資料來源] 選取 [變數資產]，接著在清單中選取 [AzureSubscriptionId]，然後按兩次 [確定]。
1. 將滑鼠停留在 [登入 Azure]  ，直到圖形的底端出現圓形。 按一下圓形，並將箭頭拖曳到 [指定訂用帳戶識別碼] 。

此時您的 Runbook 看起來應該像下面這樣︰ <br>![Runbook 驗證組態](media/automation-first-runbook-graphical/runbook-auth-config.png)

## <a name="add-activity-to-start-a-vm"></a>新增活動來啟動 VM

您現在要新增 **Start-AzureRmVM** 活動，以啟動虛擬機器。 您可以在您的 Azure 訂用帳戶中挑選任何虛擬機器，而現在您會將該名稱硬式編碼成 Cmdlet。

1. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Start-AzureRm** 。
2. 將 [Start-AzureRmVM] 新增至畫布，然後對它按一下並拖曳到 [指定訂用帳戶識別碼] 底下。
1. 將滑鼠停留在 [指定訂用帳戶識別碼]  ，直到圖形的底端出現圓形。 按一下圓形，並將箭頭拖曳到 [Start-AzureRmVM] 。
1. 選取 [Start-AzureRmVM]。 按一下 [參數]，然後按一下 [參數集] 以檢視 [Start-AzureRmVM] 的參數集。 選取 [ResourceGroupNameParameterSetName]  參數集。 [ResourceGroupName] 和 [名稱] 旁邊具有驚嘆號。 這表示它們是必要的參數。 也請注意這兩者應該是字串值。
1. 選取 [Start-AzureRmVM] 。 在 [資料來源] 中選取 [PowerShell 運算式]，並輸入用雙引號括住的虛擬機器名稱，此為您要用此 Runbook 啟動的虛擬機器。 按一下 [確定]。
1. 選取 [ResourceGroupName]。 針對 [資料來源] 使用 [PowerShell 運算式]，並輸入用雙引號括住的資源群組名稱。 按一下 [確定]。
1. 按一下 [測試] 窗格，您便可測試 Runbook。
1. 按一下 [開始]  以開始測試。 當它完成時，請檢查虛擬機器已啟動。

此時您的 Runbook 看起來應該像下面這樣︰ <br>![Runbook 驗證組態](media/automation-first-runbook-graphical/runbook-startvm.png)

## <a name="add-additional-input-parameters"></a>新增其他輸入參數

我們的 Runbook 目前會啟動在 **Start-AzureRmVM** Cmdlet 中指定之資源群組內的虛擬機器。 如果可以在啟動 Runbook 時指定這兩項，Runbook 會更有用。 您現在會將輸入參數新增至 Runbook，以提供該功能。

1. 按一下 [MyFirstRunbook-Graphical] 窗格上的 [編輯] 來開啟圖形化編輯器。
1. 選取 [輸入和輸出]，然後選取 [新增輸入]，來開啟 [Runbook Input Parameter] \(Runbook 輸入參數) 窗格。
1. 對 [ *名稱* ] 指定 **VMName**。 將 [類型] 保留為 *string*，但將 [強制] 變更為 [是]。 按一下 [確定]。
1. 建立稱為 ResourceGroupName 的第二個必要輸入參數，然後按一下 [確定] 來關閉 [輸入和輸出] 窗格。<br> ![Runbook 輸入參數](media/automation-first-runbook-graphical/start-azurermvm-params-outputs.png)
1. 選取 [Start-AzureRmVM] 活動，然後按一下 [參數]。
1. 將 [名稱] 的 [資料來源] 變更為 [Runbook 輸入]，然後選取 [VMName]。
1. 將 [ResourceGroupName] 的 [資料來源] 變更為 [Runbook 輸入]，然後選取 [ResourceGroupName]。<br> ![Start-AzureVM 參數](media/automation-first-runbook-graphical/start-azurermvm-params-runbookinput.png)
1. 儲存 Runbook 並開啟 [測試] 窗格。 您現在可以提供測試中使用的兩個輸入變數的值。
1. 關閉 [測試] 窗格。
1. 按一下 [發佈]  來發行新版本的 Runbook。
1. 停止您在上一個步驟中啟動的虛擬機器。
1. 按一下 [開始]  以啟動 Runbook。 輸入您要啟動之虛擬機器的 [VMName] 和 [ResourceGroupName]。
1. Runbook 完成時，請檢查虛擬機器已啟動。

## <a name="create-a-conditional-link"></a>建立條件式連結

您現在會修改 Runbook，讓它只會在未啟動時嘗試啟動虛擬機器。 您可將 **Get-AzureRmVM** Cmdlet 新增到可取得虛擬機器之執行個體層級狀態的 Runbook，來執行此動作。 然後您可新增名為 **取得狀態** 的 PowerShell 工作流程程式碼模組與 PowerShell 程式碼片段，來判斷虛擬機器的狀態是正在執行還是已停止。 只有當目前的執行狀態為已停止時，**取得狀態**模組中的條件式連結才會執行 **Start-AzureRmVM**。 最後會輸出訊息，通知您 VM 已成功啟動或未使用 PowerShell Write-Output Cmdlet。

1. 在圖形化編輯器中開啟 **MyFirstRunbook-Graphical**。
1. 移除 [指定訂用帳戶識別碼] 和 [Start-AzureRmVM] 之間的連結，方法是按一下該連結，然後按 [刪除] 鍵。
1. 在 [程式庫] 控制項中，於搜尋文字方塊中輸入 **Get-AzureRm** 。
1. 將 [Get-AzureRmVM]  加入至畫布。
1. 選取 [Get-AzureRmVM]，然後選取 [參數集] 以檢視 [Get-AzureRmVM] 的參數集。 選取 [GetVirtualMachineInResourceGroupNameParamSet]  參數集。 [ResourceGroupName] 和 [名稱] 旁邊具有驚嘆號。 這表示它們是必要的參數。 也請注意這兩者應該是字串值。
1. 在 [名稱] 的 [資料來源] 底下，選取 [Runbook 輸入]，然後選取 [VMName]。 按一下 [確定]。
1. 在 [ResourceGroupName] 的 [資料來源] 底下，選取 [Runbook 輸入]，然後選取 [ResourceGroupName]。 按一下 [確定]。
1. 在 [狀態] 的 [資料來源] 底下，選取 [常數值]，然後按一下 [True]。 按一下 [確定]。
1. 建立從 [指定訂用帳戶識別碼] 至 [Get-AzureRmVM] 的連結。
1. 在 [程式庫] 控制項中展開 [Runbook 控制項]，並將 [程式碼] 新增至畫布。  
1. 建立從 [Get-AzureRmVM] 至 [程式碼] 的連結。  
1. 按一下 [程式碼]，接著在 [組態] 窗格中將標籤變更為 [取得狀態]。
1. 選取 [程式碼] 參數，隨即出現 [程式碼編輯器] 頁面。  
1. 在程式碼編輯器中，貼上下列程式碼片段︰

    ```powershell-interactive
     $StatusesJson = $ActivityOutput['Get-AzureRmVM'].StatusesText
     $Statuses = ConvertFrom-Json $StatusesJson
     $StatusOut =""
     foreach ($Status in $Statuses){
     if($Status.Code -eq "Powerstate/running"){$StatusOut = "running"}
     elseif ($Status.Code -eq "Powerstate/deallocated") {$StatusOut = "stopped"}
     }
     $StatusOut
     ```

1. 建立從 [取得狀態] 至 [Start-AzureRmVM] 的連結。<br> ![程式碼模組的 Runbook](media/automation-first-runbook-graphical/runbook-startvm-get-status.png)  
1. 選取該連結，然後在 [組態] 窗格中將 [套用條件] 變更為 [是]。 請注意，連結轉變為虛線表示目標活動只有在條件解析為 true 時才會執行。  
1. 在 [條件運算式] 中輸入 $ActivityOutput['Get Status'] -eq "Stopped"。 **Start-AzureRmVM** 現在只會在虛擬機器停止時執行。
1. 在 [程式庫] 控制項中，展開 **Cmdlet**，然後展開 **Microsoft.PowerShell.Utility**。
1. 將 [Write-Output] 新增至畫布兩次。
1. 在第一個 [Write-Output] 控制項中按一下 [參數]，然後將 [標籤] 值變更為 [通知 VM 已啟動]。
1. 在 [InputObject] 中將 [資料來源] 變更為 [PowerShell 運算式]，然後輸入運算式 "$VMName successfully started"。
1. 在第二個 [Write-Output] 控制項中按一下 [參數]，然後將 [標籤] 值變更為 [通知 VM 啟動失敗]
1. 在 [InputObject] 中將 [資料來源] 變更為 [PowerShell 運算式]，然後輸入運算式 "$VMName could not start"。
1. 建立從 [Start-AzureRmVM] 至 [通知 VM 已啟動] 和 [通知 VM 啟動失敗] 的連結。
1. 選取連往 [通知 VM 已啟動] 的連結並將 [套用條件] 變更為 [True]。
1. 在 [條件運算式] 中輸入 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -eq $true。 此 Write-Output 控制項現在只會在虛擬機器已成功啟動時執行。
1. 選取連往 [通知 VM 啟動失敗] 的連結並將 [套用條件] 變更為 [True]。
1. 在 [條件運算式] 中輸入 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -ne $true。 此 Write-Output 控制項現在只會在虛擬機器未成功啟動時執行。 您的 Runbook 應該與下圖類似︰ <br> ![Write-Output 的 Runbook](media/automation-first-runbook-graphical/runbook-startazurermvm-complete.png)
1. 儲存 Runbook 並開啟 [測試] 窗格。
1. 在虛擬機器停止下啟動 Runbook，它應該會啟動。

## <a name="next-steps"></a>後續步驟

* 若要深入了解圖形化編寫，請參閱 [Azure 自動化中的圖形化編寫](automation-graphical-authoring-intro.md)
* 若要開始使用 PowerShell Runbook，請參閱 [我的第一個 PowerShell Runbook](automation-first-runbook-textual-powershell.md)
* 若要開始使用 PowerShell 工作流程 Runbook，請參閱 [我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)

