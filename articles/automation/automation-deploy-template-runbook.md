---
title: 在 Azure 自動化 Runbook 中部署 Azure Resource Manager 範本
description: 如何從 Runbook 部署儲存在 Azure 儲存體中的 Azure Resource Manager 範本
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
keywords: powershell, runbook, json, azure 自動化
ms.openlocfilehash: fe7a3632936e13a0762ebc0afcc357965e019146
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918601"
---
# <a name="deploy-an-azure-resource-manager-template-in-an-azure-automation-powershell-runbook"></a>在 Azure 自動化 PowerShell Runbook 中部署 Azure Resource Manager 範本

您可以使用 [Azure 資源管理範本](../azure-resource-manager/resource-manager-create-first-template.md)撰寫可部署 Azure 資源的 [Azure 自動化 PowerShell Runbook](automation-first-runbook-textual-powershell.md)。

如此一來，您就可以自動化部署 Azure 資源。 您可以在中央、安全的位置 (例如 Azure 儲存體) 中維護 Resource Manager 範本。

在本主題中，我們會建立使用儲存在 [Azure 儲存體](../storage/common/storage-introduction.md)中 Resource Manager 範本的 PowerShell Runbook，來部署新的 Azure 儲存體帳戶。

## <a name="prerequisites"></a>必要條件

若要完成此教學課程，您需要下列項目：

* Azure 訂用帳戶。 如果您沒有這類帳戶，可以[啟用自己的 MSDN 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或[註冊免費帳戶](https://azure.microsoft.com/free/)。
* [自動化帳戶](automation-sec-configure-azure-runas-account.md) ，用來保存 Runbook 以及向 Azure 資源驗證。  此帳戶必須擁有啟動和停止虛擬機器的權限。
* 用來儲存 Resource Manager 範本的 [Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)
* 本機電腦上安裝的 Azure Powershell。 如需如何取得 Azure PowerShell 的詳細資訊，請參閱[安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.1.0)。

## <a name="create-the-resource-manager-template"></a>建立 Resource Manager 範本

此範例中，我們使用 Resource Manager 範本來部署新的 Azure 儲存體帳戶。

在文字編輯器中，複製下列文字：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountType": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS",
        "Premium_LRS"
      ],
      "metadata": {
        "description": "Storage Account type"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "apiVersion": "2018-02-01",
      "location": "[parameters('location')]",
      "sku": {
          "name": "[parameters('storageAccountType')]"
      },
      "kind": "Storage", 
      "properties": {
      }
    }
  ],
  "outputs": {
      "storageAccountName": {
          "type": "string",
          "value": "[variables('storageAccountName')]"
      }
  }
}
```

將檔案本機儲存為 `TemplateTest.json`。

## <a name="save-the-resource-manager-template-in-azure-storage"></a>在 Azure 儲存體中儲存 Resource Manager 範本

現在我們可以使用 PowerShell 來建立 Azure 儲存體檔案共用，並上傳 `TemplateTest.json` 檔案。
如需如何在 Azure 入口網站建立檔案共用並上傳檔案的指示，請參閱[在 Windows 上開始使用 Azure 檔案儲存體](../storage/files/storage-dotnet-how-to-use-files.md)。

在您的本機電腦上啟動 PowerShell 並執行下列命令，可建立檔案共用，並將 Resource Manager 範本上傳至該檔案共用。

```powershell
# Login to Azure
Connect-AzureRmAccount

# Get the access key for your storage account
$key = Get-AzureRmStorageAccountKey -ResourceGroupName 'MyAzureAccount' -Name 'MyStorageAccount'

# Create an Azure Storage context using the first access key
$context = New-AzureStorageContext -StorageAccountName 'MyStorageAccount' -StorageAccountKey $key[0].value

# Create a file share named 'resource-templates' in your Azure Storage account
$fileShare = New-AzureStorageShare -Name 'resource-templates' -Context $context

# Add the TemplateTest.json file to the new file share
# "TemplatePath" is the path where you saved the TemplateTest.json file
$templateFile = 'C:\TemplatePath'
Set-AzureStorageFileContent -ShareName $fileShare.Name -Context $context -Source $templateFile
```

## <a name="create-the-powershell-runbook-script"></a>建立 PowerShell Runbook 指令碼

現在我們建立的 PowerShell 指令碼，會從 Azure 儲存體取得 `TemplateTest.json` 檔案，並部署範本來建立新的 Azure 儲存體帳戶。

在文字編輯器中，貼上下列文字：

```powershell
param (
    [Parameter(Mandatory=$true)]
    [string]
    $ResourceGroupName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountName,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageAccountKey,

    [Parameter(Mandatory=$true)]
    [string]
    $StorageFileName
)



# Authenticate to Azure if running from Azure Automation
$ServicePrincipalConnection = Get-AutomationConnection -Name "AzureRunAsConnection"
Connect-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint | Write-Verbose

#Set the parameter values for the Resource Manager template
$Parameters = @{
    "storageAccountType"="Standard_LRS"
    }

# Create a new context
$Context = New-AzureStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

Get-AzureStorageFileContent -ShareName 'resource-templates' -Context $Context -path 'TemplateTest.json' -Destination 'C:\Temp'

$TemplateFile = Join-Path -Path 'C:\Temp' -ChildPath $StorageFileName

# Deploy the storage account
New-AzureRmResourceGroupDeployment -ResourceGroupName $ResourceGroupName -TemplateFile $TemplateFile -TemplateParameterObject $Parameters 
``` 

將檔案本機儲存為 `DeployTemplate.ps1`。

## <a name="import-and-publish-the-runbook-into-your-azure-automation-account"></a>將 Runbook 匯入您的 Azure 自動化帳戶並加以發佈

現在，我們會使用 PowerShell 將 Runbook 匯入您的 Azure 自動化帳戶，然後發佈 Runbook。
如需關於如何在 Azure 入口網站中匯入並發佈 Runbook 的資訊，請參閱[在 Azure 自動化中建立或匯入 Runbook](automation-creating-importing-runbook.md)。

若要將 `DeployTemplate.ps1` 匯入您的自動化帳戶中作為 PowerShell Runbook，請執行下列 PowerShell 命令：

```powershell
# MyPath is the path where you saved DeployTemplate.ps1
# MyResourceGroup is the name of the Azure ResourceGroup that contains your Azure Automation account
# MyAutomationAccount is the name of your Automation account
$importParams = @{
    Path = 'C:\MyPath\DeployTemplate.ps1'
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Type = 'PowerShell'
}
Import-AzureRmAutomationRunbook @importParams

# Publish the runbook
$publishParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
}
Publish-AzureRmAutomationRunbook @publishParams
```

## <a name="start-the-runbook"></a>啟動 Runbook

現在我們會呼叫 [Start-AzureRmAutomationRunbook](https://docs.microsoft.com/powershell/module/azurerm.automation/start-azurermautomationrunbook?view=azurermps-4.1.0) Cmdlet 來啟動 Runbook。

如需關於如何在 Azure 入口網站中啟動 Runbook 的資訊，請參閱[在 Azure 自動化中啟動 Runbook](automation-starting-a-runbook.md)。

在 PowerShell 主控台中執行下列命令：

```powershell
# Set up the parameters for the runbook
$runbookParams = @{
    ResourceGroupName = 'MyResourceGroup'
    StorageAccountName = 'MyStorageAccount'
    StorageAccountKey = $key[0].Value # We got this key earlier
    StorageFileName = 'TemplateTest.json' 
}

# Set up parameters for the Start-AzureRmAutomationRunbook cmdlet
$startParams = @{
    ResourceGroupName = 'MyResourceGroup'
    AutomationAccountName = 'MyAutomationAccount'
    Name = 'DeployTemplate'
    Parameters = $runbookParams
}

# Start the runbook
$job = Start-AzureRmAutomationRunbook @startParams
```

Runbook 執行時，您可以執行 `$job.Status` 來檢查其狀態。

Runbook 會取得 Resource Manager 範本，並使用它來部署新的 Azure 儲存體帳戶。
您可以執行下列命令來看到建立的新儲存體帳戶：
```powershell
Get-AzureRmStorageAccount
```

## <a name="summary"></a>總結

就這麼簡單！ 現在您可以使用 Azure 自動化和 Azure 儲存體，以及 Resource Manager 範本來部署您所有的 Azure 資源。

## <a name="next-steps"></a>後續步驟

* 若要深入了解 Resource Manager 範本，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)
* 若要開始使用 Azure 儲存體，請參閱 [Azure 儲存體簡介](../storage/common/storage-introduction.md)。
* 若要尋找其他實用的 Azure 自動化 Runbook，請參閱 [Azure 自動化的 Runbook 和模組資源庫](automation-runbook-gallery.md)。
* 若要尋找其他實用的 Resource Manager 範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/resources/templates/)

