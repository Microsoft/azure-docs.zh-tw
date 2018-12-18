---
title: 將 JSON 物件傳送至 Azure 自動化 Runbook
description: 如何將參數傳遞給 Runbook 作為 JSON 物件
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
keywords: powershell, runbook, json, azure 自動化
ms.openlocfilehash: 5e1ab8d6bd2de24251851cfc60d270a2fef4090d
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918813"
---
# <a name="pass-a-json-object-to-an-azure-automation-runbook"></a>將 JSON 物件傳送至 Azure 自動化 Runbook

這適用於在 JSON 檔案中儲存您要傳送至 Runbook 的資料。
例如，您可以建立 JSON 檔案，其中包含您要傳送至 Runbook 的所有參數。
若要這樣做，您必須將 JSON 轉換為字串，然後先將字串轉換為 PowerShell 物件，再將其內容傳送至 Runbook。

在此範例中，我們將建立 PowerShell 指令碼，該指令碼會呼叫 [Start-AzureRmAutomationRunbook](https://docs.microsoft.com/powershell/module/azurerm.automation/start-azurermautomationrunbook) 以啟動 PowerShell Runbook，並將 JSON 內容傳送至 Runbook。
PowerShell Runbook 會啟動 Azure VM，並從傳入的 JSON 取得 VM 的參數。

## <a name="prerequisites"></a>必要條件
若要完成此教學課程，您需要下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或[註冊免費帳戶](https://azure.microsoft.com/free/)。
* [自動化帳戶](automation-sec-configure-azure-runas-account.md) ，用來保存 Runbook 以及向 Azure 資源驗證。  此帳戶必須擁有啟動和停止虛擬機器的權限。
* Azure 虛擬機器。 我們會停止並啟動這部電腦，因此它不該是生產 VM。
* 本機電腦上安裝的 Azure Powershell。 如需如何取得 Azure PowerShell 的詳細資訊，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.1.0)。

## <a name="create-the-json-file"></a>建立 JSON 檔案

在文字檔案中輸入下列測試，然後在本機電腦上的某處將它儲存為 `test.json`。

```json
{
   "VmName" : "TestVM",
   "ResourceGroup" : "AzureAutomationTest"
}
```

## <a name="create-the-runbook"></a>建立 runbook

在 Azure 自動化中建立名為 "Test-Json" 的新 PowerShell Runbook。
若要了解如何建立新的 PowerShell Runbook，請參閱[我的第一個 PowerShell Runbook](automation-first-runbook-textual-powershell.md)。

若要接受 JSON 資料，Runbook 必須採用物件作為輸入參數。

然後 Runbook 可以使用 JSON 中定義的屬性。

```powershell
Param(
     [parameter(Mandatory=$true)]
     [object]$json
)

# Connect to Azure account   
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID `
    -ApplicationID $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Convert object to actual JSON
$json = $json | ConvertFrom-Json

# Use the values from the JSON object as the parameters for your command
Start-AzureRmVM -Name $json.VMName -ResourceGroupName $json.ResourceGroup
 ```

 在您的自動化帳戶中儲存並發佈此 Runbook。

## <a name="call-the-runbook-from-powershell"></a>從 PowerShell 呼叫 Runbook

您現在可以使用 Azure PowerShell，從本機電腦呼叫 Runbook。
執行下列 PowerShell 命令：

1. 登入 Azure：
   ```powershell
   Connect-AzureRmAccount
   ```
    系統會提示您輸入 Azure 認證。

   > [!IMPORTANT]
   > **Add-AzureRmAccount** 現在是 **Connect-AzureRMAccount** 的別名。 搜尋您的程式庫項目時，如果沒有看到 **Connect-AzureRMAccount**，便可以使用 **Add-AzureRmAccount**，或是在自動化帳戶中更新模組。

1. 取得 JSON 檔案的內容，並將它轉換成字串：
    ```powershell
    $json =  (Get-content -path 'JsonPath\test.json' -Raw) | Out-string
    ```
    `JsonPath` 是您儲存 JSON 檔案的路徑。
1. 將 `$json` 的字串內容轉換為 PowerShell 物件：
   ```powershell
   $JsonParams = @{"json"=$json}
   ```
1. 建立 `Start-AzureRmAutomationRunbook` 參數的雜湊表：
   ```powershell
   $RBParams = @{
        AutomationAccountName = 'AATest'
        ResourceGroupName = 'RGTest'
        Name = 'Test-Json'
        Parameters = $JsonParams
   }
   ```
   請注意，您要將 `Parameters` 的值設定為 PowerShell 物件，其中包含 JSON 檔案中的值。 
1. 啟動 Runbook
   ```powershell
   $job = Start-AzureRmAutomationRunbook @RBParams
   ```

Runbook 會使用 JSON 檔案中的值來啟動 VM。

## <a name="next-steps"></a>後續步驟

* 若要深入了解使用文字編輯器編輯 PowerShell 和 PowerShell 工作流程 Runbook，請參閱 [在 Azure 自動化中編輯文字式 Runbook](automation-edit-textual-runbook.md) 
* 若要了解如何建立和匯入 Runbook，請參閱[在 Azure 自動化中建立或匯入 Runbook](automation-creating-importing-runbook.md)


