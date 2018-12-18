---
title: 使用 Azure Resource Manager 部署 Machine Learning 工作區 | Microsoft Docs
description: 如何使用 Azure Resource Manager 範本部署 Azure Machine Learning 的工作區
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: 4955ac4d-ff99-4908-aa27-69b6bfcc8e85
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/05/2018
ms.openlocfilehash: 82d2316b3f72fbb0c5c3ee1ea9424afcc7661361
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34833966"
---
# <a name="deploy-machine-learning-workspace-using-azure-resource-manager"></a>使用 Azure Resource Manager 部署 Machine Learning 工作區
## <a name="introduction"></a>簡介
使用 Azure Resource Manager 部署範本提供了可擴充的方式來部署具有驗證和重試機制的互連元件，為您節省時間。 若要設定 Azure Machine Learning 工作區，例如，您需要先設定 Azure 儲存體帳戶，然後再部署您的工作區。 假想您要對數百個工作區手動進行此動作。 簡單的替代方法是使用 Azure Resource Manager 範本來部署 Azure Machine Learning 工作區和所有相依性。 這篇文章會帶領您逐步完成此程序。 如需 Azure Resource Manager 的詳細概觀，請參閱 [Azure Resource Manager 概觀](../../azure-resource-manager/resource-group-overview.md)。

## <a name="step-by-step-create-a-machine-learning-workspace"></a>逐步說明：建立 Machine Learning 工作區
我們將建立 Azure 資源群組，然後使用 Resource Manager 範本部署新的 Azure 儲存體帳戶和新的 Azure Machine Learning 工作區。 部署完成之後，我們會印出所建立的工作區的重要資訊 (主索引鍵、工作區識別碼和工作區的 URL)。

### <a name="create-an-azure-resource-manager-template"></a>建立 Azure Resource Manager 範本
Machine Learning 工作區需有 Azure 儲存體帳戶來儲存連結到它的資料集。
以下範本會使用資源群組的名稱來產生儲存體帳戶名稱和工作區名稱。  建立工作區時，它也會使用儲存體帳戶名稱做為屬性。

```
{
    "contentVersion": "1.0.0.0",
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "variables": {
        "namePrefix": "[resourceGroup().name]",
        "location": "[resourceGroup().location]",
        "mlVersion": "2016-04-01",
        "stgVersion": "2015-06-15",
        "storageAccountName": "[concat(variables('namePrefix'),'stg')]",
        "mlWorkspaceName": "[concat(variables('namePrefix'),'mlwk')]",
        "mlResourceId": "[resourceId('Microsoft.MachineLearning/workspaces', variables('mlWorkspaceName'))]",
        "stgResourceId": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "storageAccountType": "Standard_LRS"
    },
    "resources": [
        {
            "apiVersion": "[variables('stgVersion')]",
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[variables('location')]",
            "properties": {
                "accountType": "[variables('storageAccountType')]"
            }
        },
        {
            "apiVersion": "[variables('mlVersion')]",
            "type": "Microsoft.MachineLearning/workspaces",
            "name": "[variables('mlWorkspaceName')]",
            "location": "[variables('location')]",
            "dependsOn": ["[variables('stgResourceId')]"],
            "properties": {
                "UserStorageAccountId": "[variables('stgResourceId')]"
            }
        }
    ],
    "outputs": {
        "mlWorkspaceObject": {"type": "object", "value": "[reference(variables('mlResourceId'), variables('mlVersion'))]"},
        "mlWorkspaceToken": {"type": "string", "value": "[listWorkspaceKeys(variables('mlResourceId'), variables('mlVersion')).primaryToken]"},
        "mlWorkspaceWorkspaceID": {"type": "string", "value": "[reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId]"},
        "mlWorkspaceWorkspaceLink": {"type": "string", "value": "[concat('https://studio.azureml.net/Home/ViewWorkspace/', reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId)]"}
    }
}

```
將此範本在 c:\temp\ 下儲存為 mlworkspace.json 檔案。

### <a name="deploy-the-resource-group-based-on-the-template"></a>依據範本部署資源群組
* 開啟 PowerShell
* 安裝 Azure Resource Manager 和 Azure 服務管理的模組  

```
# Install the Azure Resource Manager modules from the PowerShell Gallery (press “A”)
Install-Module AzureRM -Scope CurrentUser

# Install the Azure Service Management modules from the PowerShell Gallery (press “A”)
Install-Module Azure -Scope CurrentUser
```

   下列步驟會下載並安裝完成剩餘步驟所需的模組。 這只需要在您執行 PowerShell 命令的環境中執行一次。   

* 向 Azure 驗證  

```
# Authenticate (enter your credentials in the pop-up window)
Connect-AzureRmAccount
```
需要為每個工作階段重複執行這個步驟。 驗證之後，應該會顯示您的訂用帳戶資訊。

![Azure 帳戶][1]

現在，我們已經可以存取 Azure，便可以建立資源群組。

* 建立資源群組

```
$rg = New-AzureRmResourceGroup -Name "uniquenamerequired523" -Location "South Central US"
$rg
```

請確認資源群組已正確佈建。 **ProvisioningState** 應該是 “Succeeded”。
範本會使用資源群組名稱來產生儲存體帳戶名稱。 儲存體帳戶名稱必須介於 3 到 24 個字元的長度，而且只能使用數字和小寫字母。

![資源群組][2]

* 使用資源群組部署，部署新的 Machine Learning 工作區。

```
# Create a Resource Group, TemplateFile is the location of the JSON template.
$rgd = New-AzureRmResourceGroupDeployment -Name "demo" -TemplateFile "C:\temp\mlworkspace.json" -ResourceGroupName $rg.ResourceGroupName
```

一旦完成部署之後，就可以直接存取您所部署的工作區的屬性。 例如，您可以存取主要金鑰權杖。

```
# Access Azure ML Workspace Token after its deployment.
$rgd.Outputs.mlWorkspaceToken.Value
```

另一種擷取現有工作區權杖的方式是使用 Invoke-AzureRmResourceAction 命令。 例如，您可以列出所有工作區的主要和次要權杖。

```  
# List the primary and secondary tokens of all workspaces
Get-AzureRmResource |? { $_.ResourceType -Like "*MachineLearning/workspaces*"} |% { Invoke-AzureRmResourceAction -ResourceId $_.ResourceId -Action listworkspacekeys -Force}  
```
佈建工作區之後，您也可以使用 [適用於 Azure Machine Learning 的 PowerShell 模組](http://aka.ms/amlps)將許多 Azure Machine Learning Studio 工作自動化。

## <a name="next-steps"></a>後續步驟
* 深入了解 [編寫 Azure Resource Manager 範本](../../azure-resource-manager/resource-group-authoring-templates.md)。 
* 看看 [Azure 快速入門範本儲存機制](https://github.com/Azure/azure-quickstart-templates)。 
* 觀看這段 [Azure Resource Manager](https://channel9.msdn.com/Events/Ignite/2015/C9-39)影片。 

<!--Image references-->
[1]: ./media/deploy-with-resource-manager-template/azuresubscription.png
[2]: ./media/deploy-with-resource-manager-template/resourcegroupprovisioning.png


<!--Link references-->
