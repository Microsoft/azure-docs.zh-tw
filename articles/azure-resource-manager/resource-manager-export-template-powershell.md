---
title: 使用 Azure PowerShell 來匯出 Resource Manager 範本 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Azure PowerShell 從資源群組匯出範本。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/23/2018
ms.author: tomfitz
ms.openlocfilehash: c69bab9d2956568473dd6def86ecbd9bbb6577cf
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34359215"
---
# <a name="export-azure-resource-manager-templates-with-powershell"></a>使用 PowerShell 來匯出 Azure Resource Manager 範本

Resource Manager 可讓您從您的訂用帳戶中現有的資源匯出 Resource Manager 範本。 您可以使用產生的範本了解範本語法，或視需要自動重新部署解決方案。

請務必注意，有兩種不同的方式可匯出範本︰

* 您可以匯出**用於部署的實際範本**。 匯出的範本包含與原始範本完全相同的所有參數和變數。 當您需要擷取範本時，這個方法很有用。
* 您可以匯出**代表資源群組目前狀態的已產生範本**。 匯出的範本不是以任何用於部署的範本為基礎。 反而，它所建立的範本是資源群組的「快照集」或「備份」。 匯出的範本會有許多硬式編碼值，但數量可能不如您通常會定義的參數數量。 使用此選項來將資源重新部署至相同的資源群組。 若要對其他資源群組使用此範本，您可能必須大幅修改它。

本文章說明這兩種方法。

## <a name="deploy-a-solution"></a>部署解決方案

為了說明這兩種用來匯出範本的方法，一開始先讓我們將解決方案部署到您的訂用帳戶。 如果您的訂用帳戶中已有想要匯出的資源群組，就不需要部署此解決方案。 不過，本文其餘部分會參考此解決方案的範本。 指令碼範例會部署儲存體帳戶。

```powershell
New-AzureRmResourceGroup -Name ExampleGroup -Location "South Central US"
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup `
  -DeploymentName NewStorage
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```  

## <a name="save-template-from-deployment-history"></a>從部署歷程記錄儲存範本

您可以使用 [Save-AzureRmResourceGroupDeploymentTemplate](/powershell/module/azurerm.resources/save-azurermresourcegroupdeploymenttemplate) 命令來從部署歷程記錄中擷取範本。 下列範例會儲存您先前部署的範本︰

```powershell
Save-AzureRmResourceGroupDeploymentTemplate -ResourceGroupName ExampleGroup -DeploymentName NewStorage
```

它會傳回範本的位置。

```powershell
Path
----
C:\Users\exampleuser\NewStorage.json
```

開啟檔案，並請注意它正是您用於部署的範本。 其參數和變數會與 GitHub 中的範本相符。 您可以重新部署此範本。

## <a name="export-resource-group-as-template"></a>匯出資源群組以作為範本

您不必從部署歷程記錄擷取範本，而是可以使用 [Export-AzureRmResourceGroup](/powershell/module/azurerm.resources/export-azurermresourcegroup) 命令來擷取範本，以代表資源群組的目前狀態。 當您對資源群組做了許多變更，且現有的範本均無法完全呈現這些變更時，就可以使用這個命令。 其目的在作為資源群組的快照集，以供您用來重新部署至相同的資源群組。 若要對其他解決方案使用所匯出的範本，您必須大幅修改它。

```powershell
Export-AzureRmResourceGroup -ResourceGroupName ExampleGroup
```

它會傳回範本的位置。

```powershell
Path
----
C:\Users\exampleuser\ExampleGroup.json
```

開啟檔案，並請注意它與 GitHub 中的範本不同。 它有不同的參數，而且沒有任何變數。 儲存體 SKU 和位置皆已硬式編碼為值。 下列範例會顯示所匯出的範本，但您的範本會有稍微不同的參數名稱︰

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccounts_nf3mvst4nqb36standardsa_name": {
      "defaultValue": null,
      "type": "String"
    }
  },
  "variables": {},
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "sku": {
        "name": "Standard_LRS",
        "tier": "Standard"
      },
      "kind": "Storage",
      "name": "[parameters('storageAccounts_nf3mvst4nqb36standardsa_name')]",
      "apiVersion": "2016-01-01",
      "location": "southcentralus",
      "tags": {},
      "properties": {},
      "dependsOn": []
    }
  ]
}
```

您可以重新部署此範本，但必須猜測儲存體帳戶的唯一名稱。 您的參數名稱會稍微不同。

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup `
  -TemplateFile C:\Users\exampleuser\ExampleGroup.json `
  -storageAccounts_nf3mvst4nqb36standardsa_name tfnewstorage0501
```

## <a name="customize-exported-template"></a>自訂匯出的範本

您可以修改此範本，讓它變得更方便使用且更有彈性。 若要允許更多位置，請將 location 屬性變更為使用與資源群組相同的位置︰

```json
"location": "[resourceGroup().location]",
```

為了避免必須猜測儲存體帳戶的唯一名稱，請移除儲存體帳戶名稱的參數。 新增儲存體名稱尾碼的參數，以及儲存體 SKU︰

```json
"parameters": {
    "storageSuffix": {
      "type": "string",
      "defaultValue": "standardsa"
    },
    "storageAccountType": {
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS"
      ],
      "type": "string",
      "metadata": {
        "description": "Storage Account type"
      }
    }
},
```

新增變數，以便使用 uniqueString 函式來建構儲存體帳戶名稱︰

```json
"variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
```

將儲存體帳戶名稱設定為變數︰

```json
"name": "[variables('storageAccountName')]",
```

將 SKU 設定為參數︰

```json
"sku": {
    "name": "[parameters('storageAccountType')]",
    "tier": "Standard"
},
```

您的範本現在如下所示：

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageSuffix": {
      "type": "string",
      "defaultValue": "standardsa"
    },
    "storageAccountType": {
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS"
      ],
      "type": "string",
      "metadata": {
        "description": "Storage Account type"
      }
    }
  },
  "variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), parameters('storageSuffix'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "sku": {
        "name": "[parameters('storageAccountType')]",
        "tier": "Standard"
      },
      "kind": "Storage",
      "name": "[variables('storageAccountName')]",
      "apiVersion": "2016-01-01",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {},
      "dependsOn": []
    }
  ]
}
```

重新部署修改過的範本。

## <a name="next-steps"></a>後續步驟
* 如需使用入口網站來匯出範本的相關資訊，請參閱[從現有資源匯出 Azure Resource Manager 範本](resource-manager-export-template.md)。
* 若要在範本中定義參數，請參閱 [編寫範本](resource-group-authoring-templates.md#parameters)。
* 如需解決常見部署錯誤的秘訣，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
