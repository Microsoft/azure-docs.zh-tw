---
title: 在 Azure 的更新管理部署上設定前置和後置指令碼 (預覽)
description: 本文將說明如何設定及管理更新部署的前置和後置指令碼
services: automation
ms.service: automation
ms.component: update-management
author: georgewallace
ms.author: gwallace
ms.date: 09/18/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: c906a771a63b3d8320eab1d2d57e8c34916e1d39
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433187"
---
# <a name="manage-pre-and-post-scripts-preview"></a>管理前置和後置指令碼 (預覽)

透過前置和後置指令碼，您可以在更新部署的之前 (前置工作) 和之後 (後置工作)，於自動化帳戶中執行 PowerShell Runbook。 前置和後置指令碼會在 Azure 環境中執行，而不是在本機執行。

## <a name="runbook-requirements"></a>Runbook 需求

針對要作為前置或後置指令碼使用的 Runbook，此 Runbook 必須匯入到您的自動化帳戶中並發佈。 若要深入了解此程序，請參閱[發佈 Runbook](automation-creating-importing-runbook.md#publishing-a-runbook)。

## <a name="using-a-prepost-script"></a>使用前置/後置指令碼

若要在更新部署中使用前置和或後置指令碼，可藉由直接建立更新部署來開始。 選取 [前置指令碼 + 後置指令碼 (預覽)]。 這會開啟 [選取前置指令碼 + 後置指令碼] 頁面。  

![選取指令碼](./media/pre-post-scripts/select-scripts.png)

選取您想要使用的指令碼，在此範例中，您已使用 **UpdateManagement-TurnOnVms** Runbook。 當您選取了 Runbook 後，[設定指令碼] 頁面會隨即開啟，請提供參數的值，然後選擇 [前置指令碼]。 完成時按一下 [確定]。

![設定指令碼](./media/pre-post-scripts/configure-script.png)

對 **UpdateManagement-TurnOffVms** 指令碼重複此程序。 但是當您選擇 [指令碼類型] 時，請選擇 [後置指令碼]。

[選取的項目] 區段現在會顯示您選取的兩個指令碼，一個是前置指令碼，另一個是後置指令碼。

![選取的項目](./media/pre-post-scripts/selected-items.png)

完成更新部署的設定。

更新部署完成後，您可以移至 [更新部署] 來檢視結果。 您可以看到前置指令碼和後置指令碼的狀態都已提供。

![更新結果](./media/pre-post-scripts/update-results.png)

按一下並進入更新部署執行，您會看到前置和後置指令碼的其他詳細資料。 其中會顯示該執行進行時的指令碼來源連結。

![部署執行的結果](./media/pre-post-scripts/deployment-run.png)

## <a name="passing-parameters"></a>傳遞參數

當您設定了前置和後置指令碼後，您可以傳遞參數，就如同排程 Runbook 一樣。 參數會在建立更新部署時定義。 除了標準的 Runbook 參數外，還會有一個額外的參數。 這個參數是 **SoftwareUpdateConfigurationRunContext**。 此參數是 JSON 字串，如果您在前置或後置指令碼中定義該參數，更新部署就會自動傳遞該參數。 該參數包含更新部署的相關資訊，也就是 [SoftwareUpdateconfigurations API](/rest/api/automation/softwareupdateconfigurations/getbyname#updateconfiguration) 傳回的資訊子集 下表顯示在變數中提供的屬性：

### <a name="softwareupdateconfigurationruncontext-properties"></a>SoftwareUpdateConfigurationRunContext 屬性

|屬性  |說明  |
|---------|---------|
|SoftwareUpdateConfigurationName     | 軟體更新設定的名稱        |
|SoftwareUpdateConfigurationRunId     | 執行的唯一識別碼。        |
|SoftwareUpdateConfigurationSettings     | 與軟體更新設定相關的屬性集合         |
|SoftwareUpdateConfigurationSettings.operatingSystem     | 更新部署的目標作業系統         |
|SoftwareUpdateConfigurationSettings.duration     | 更新部署執行的持續時間上限 (也稱為「維護期間」)，根據 ISO8601，其格式為 `PT[n]H[n]M[n]S`          |
|SoftwareUpdateConfigurationSettings.Windows     | 與 Windows 電腦相關的屬性集合         |
|SoftwareUpdateConfigurationSettings.Windows.excludedKbNumbers     | 排除在更新部署外的 KB 清單        |
|SoftwareUpdateConfigurationSettings.Windows.includedUpdateClassifications     | 為更新部署選取的更新分類        |
|SoftwareUpdateConfigurationSettings.Windows.rebootSetting     | 更新部署的重新啟動設定        |
|azureVirtualMachines     | 更新部署中 Azure VM 的資源識別碼清單        |
|nonAzureComputerNames|更新部署中的非 Azure 電腦 FQDN 清單|

以下是傳遞至 **SoftwareUpdateConfigurationRunContext** 參數的 JSON 字串範例：

```json
"SoftwareUpdateConfigurationRunContext":{
      "SoftwareUpdateConfigurationName":"sampleConfiguration",
      "SoftwareUpdateConfigurationRunId":"00000000-0000-0000-0000-000000000000",
      "SoftwareUpdateConfigurationSettings":{
         "operatingSystem":"Windows",
         "duration":"PT2H0M",
         "windows":{
            "excludedKbNumbers":[
               "168934",
               "168973"
            ],
            "includedUpdateClassifications":"Critical",
            "rebootSetting":"IfRequired"
         },
         "azureVirtualMachines":[
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-01",
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-02",
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-03"
         ], 
         "nonAzureComputerNames":[
            "box1.contoso.com",
            "box2.contoso.com"
         ]
      }
   }
```

您可以在以下位置找到所有屬性的完整範例：[軟體更新設定 - 依名稱取得](/rest/api/automation/softwareupdateconfigurations/getbyname#examples)

> [!NOTE]
> 使用[動態群組 (預覽)](automation-update-management.md#using-dynamic-groups) 新增到部署的電腦目前不屬於 **SoftwareUpdateConfigurationRunContext**參數。

## <a name="samples"></a>範例

您可以在[指令碼中心資源庫](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&f%5B0%5D.Value=WindowsAzure&f%5B0%5D.Text=Windows%20Azure&f%5B1%5D.Type=SubCategory&f%5B1%5D.Value=WindowsAzure_automation&f%5B1%5D.Text=Automation&f%5B2%5D.Type=SearchText&f%5B2%5D.Value=update%20management&f%5B3%5D.Type=Tag&f%5B3%5D.Value=Patching&f%5B3%5D.Text=Patching&f%5B4%5D.Type=ProgrammingLanguage&f%5B4%5D.Value=PowerShell&f%5B4%5D.Text=PowerShell)中找到前置和後置指令碼範例，或透過 Azure 入口網站來匯入這些範例。 若要透過入口網站匯入這些範例，請在您自動化帳戶的 [程序自動化] 下方，選取 [Runbook 資源庫]。 使用 [更新管理] 作為篩選條件。

![資源庫清單](./media/pre-post-scripts/runbook-gallery.png)

或者，您可以透過指令碼名稱來搜尋這些範例，如下列清單所示：

* 更新管理 - 開啟 VM
* 更新管理 - 關閉 VM
* 更新管理 - 在本機執行指令碼
* 更新管理 - 前置/後置指令碼的範本
* 更新管理 - 以執行命令來執行指令碼

> [!IMPORTANT]
> 匯入 Runbook 之後，您必須先將其**發佈**，方能使用。 若要這麼做，請在自動化帳戶中尋找 Runbook，選取 [編輯]，然後按一下 [發佈]。

這些範例都會以下列範例中定義的基底範本為基礎。 此範本可用來建立您自己的 Runbook，以便搭配前置和後置指令碼使用。 其中包含向 Azure 進行驗證及處理 `SoftwareUpdateConfigurationRunContext` 參數的必要邏輯。

```powershell
<# 
.SYNOPSIS 
 Barebones script for Update Management Pre/Post 
 
.DESCRIPTION 
  This script is intended to be run as a part of Update Management Pre/Post scripts.  
  It requires a RunAs account. 
 
.PARAMETER SoftwareUpdateConfigurationRunContext 
  This is a system variable which is automatically passed in by Update Management during a deployment. 
#> 
 
param( 
    [string]$SoftwareUpdateConfigurationRunContext 
) 
#region BoilerplateAuthentication 
#This requires a RunAs account 
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection' 
 
Add-AzureRmAccount ` 
    -ServicePrincipal ` 
    -TenantId $ServicePrincipalConnection.TenantId ` 
    -ApplicationId $ServicePrincipalConnection.ApplicationId ` 
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint 
 
$AzureContext = Select-AzureRmSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID 
#endregion BoilerplateAuthentication 
 
#If you wish to use the run context, it must be converted from JSON 
$context = ConvertFrom-Json  $SoftwareUpdateConfigurationRunContext 
#Access the properties of the SoftwareUpdateConfigurationRunContext 
$vmIds = $context.SoftwareUpdateConfigurationSettings.AzureVirtualMachines 
$runId = $context.SoftwareUpdateConfigurationRunId 
 
Write-Output $context 
 
#Example: How to create and write to a variable using the pre-script: 
<# 
#Create variable named after this run so it can be retrieved 
New-AzureRmAutomationVariable -ResourceGroupName $ResourceGroup –AutomationAccountName $AutomationAccount –Name $runId -Value "" –Encrypted $false 
#Set value of variable  
Set-AutomationVariable –Name $runId -Value $vmIds 
#> 
 
#Example: How to retrieve information from a variable set during the pre-script 
<# 
$variable = Get-AutomationVariable -Name $runId 
#>      
```

## <a name="interacting-with-non-azure-machines"></a>與非 Azure 機器互動

前置和後置工作會在 Azure 環境中執行，而且沒有非 Azure 機器的存取權。 若要與非 Azure 機器互動，您必須具備下列項目：

* 執行身分帳戶
* 已安裝在機器上的混合式 Runbook 背景工作角色
* 您想要在本機上執行的 Runbook
* 父代 Runbook

若要與非 Azure 機器互動，父代 Runbook 應在 Azure 環境中執行。 此 Runbook 會使用 [Start-AzureRmAutomationRunbook](/powershell/module/azurerm.automation/start-azurermautomationrunbook) Cmdlet 來呼叫子 Runbook。 您必須指定 `-RunOn` 參數，並提供混合式 Runbook 背景工作角色的名稱，好讓指令碼在其中執行。

```powershell
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Add-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Select-AzureRmSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$resourceGroup = "AzureAutomationResourceGroup"
$aaName = "AzureAutomationAccountName"

$output = Start-AzureRmAutomationRunbook -Name "StartService" -ResourceGroupName $resourceGroup  -AutomationAccountName $aaName -RunOn "hybridWorker"

$status = Get-AzureRmAutomationJob -Id $output.jobid -ResourceGroupName $resourceGroup  -AutomationAccountName $aaName
while ($status.status -ne "Completed")
{ 
    Start-Sleep -Seconds 5
    $status = Get-AzureRmAutomationJob -Id $output.jobid -ResourceGroupName $resourceGroup  -AutomationAccountName $aaName
}

$summary = Get-AzureRmAutomationJobOutput -Id $output.jobid -ResourceGroupName $resourceGroup  -AutomationAccountName $aaName

if ($summary.Type -eq "Error")
{
    Write-Error -Message $summary.Summary
}
```

## <a name="known-issues"></a>已知問題

* 使用前置和後置指令碼時，您無法將物件或陣列傳遞給參數。 Runbook 將會失敗。
* 選擇前置或後置指令碼時，未發佈的 Runbook 顯示為可選取。 應只選擇已發佈的 Runbook，因為未發佈的 Runbook 並不能叫用，而且會導致失敗。
* 目前，使用[動態群組 (預覽)](automation-update-management.md#using-dynamic-groups) 新增到部署的電腦，不屬於傳遞至前置和後置指令碼的 **SoftwareUpdateConfigurationRunContext** 參數。

## <a name="next-steps"></a>後續步驟

繼續進行本教學課程，以了解如何管理 Windows 虛擬機器的更新。

> [!div class="nextstepaction"]
> [管理 Azure Windows VM 的更新和修補程式](automation-tutorial-update-management.md)