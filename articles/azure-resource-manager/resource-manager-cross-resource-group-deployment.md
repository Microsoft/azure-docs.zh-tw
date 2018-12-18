---
title: 將 Azure 資源部署至多個訂用帳戶和資源群組 | Microsoft Docs
description: 示範如何在部署期間將目標放在多個 Azure 訂用帳戶和資源群組。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/02/2018
ms.author: tomfitz
ms.openlocfilehash: fec075a744b5f47a4be7f1b960cceedfea7b9a2c
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35632984"
---
# <a name="deploy-azure-resources-to-more-than-one-subscription-or-resource-group"></a>將 Azure 資源部署至多個訂用帳戶和資源群組

一般而言，您要將範本中的所有資源都部署至單一[資源群組](resource-group-overview.md)。 不過，在某些情況下，您要將一組資源部署在一起，但將它們放在不同的資源群組或訂用帳戶中。 例如，建議您將 Azure Site Recovery 的備份虛擬機器部署至不同的資源群組和位置。 Resource Manager 可讓您使用巢狀的範本，將目標放在與父範本所使用之訂用帳戶和資源群組不同的訂用帳戶和資源群組。

> [!NOTE]
> 您在單一部署中只能部署至五個資源群組。 一般而言，此限制表示您可以部署至一個指定用於父範本的資源群組，並且可在巢狀或連結的部署中部署至最多四個資源群組。 不過，如果父範本只包含巢狀或連結的範本，而本身未部署任何資源，則您可以在巢狀或連結的部署中包含最多五個資源群組。

## <a name="specify-a-subscription-and-resource-group"></a>指定訂用帳戶和資源群組

若要以不同的資源為目標，請使用巢狀或連結的範本。 `Microsoft.Resources/deployments` 資源類型提供 `subscriptionId` 和 `resourceGroup` 的參數。 這些屬性可讓您指定不同的訂用帳戶和資源群組來進行巢狀部署。 執行部署之前，所有資源群組都必須存在。 如果您未指定訂用帳戶識別碼或資源群組，則會使用父範本中的訂用帳戶及資源群組。

您用來部署範本的帳戶必須具有可部署到指定訂用帳戶 ID 的權限。 如果指定的訂用帳戶在不同的 Azure Active Directory 租用戶中，您必須[從另一個目錄中新增來賓使用者](../active-directory/active-directory-b2b-what-is-azure-ad-b2b.md)。

若要指定不同的資源群組和訂用帳戶，請使用：

```json
"resources": [
    {
        "apiVersion": "2017-05-10",
        "name": "nestedTemplate",
        "type": "Microsoft.Resources/deployments",
        "resourceGroup": "[parameters('secondResourceGroup')]",
        "subscriptionId": "[parameters('secondSubscriptionID')]",
        ...
    }
]
```

如果您的資源群組位於相同的訂用帳戶中，則可以移除 **subscriptionId** 值。

下列範例會部署兩個儲存體帳戶 - 一個是在部署期間指定的資源群組中，另一個是在 `secondResourceGroup` 參數內指定的資源群組中：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storagePrefix": {
            "type": "string",
            "maxLength": 11
        },
        "secondResourceGroup": {
            "type": "string"
        },
        "secondSubscriptionID": {
            "type": "string",
            "defaultValue": ""
        },
        "secondStorageLocation": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]"
        }
    },
    "variables": {
        "firstStorageName": "[concat(parameters('storagePrefix'), uniqueString(resourceGroup().id))]",
        "secondStorageName": "[concat(parameters('storagePrefix'), uniqueString(parameters('secondSubscriptionID'), parameters('secondResourceGroup')))]"
    },
    "resources": [
        {
            "apiVersion": "2017-05-10",
            "name": "nestedTemplate",
            "type": "Microsoft.Resources/deployments",
            "resourceGroup": "[parameters('secondResourceGroup')]",
            "subscriptionId": "[parameters('secondSubscriptionID')]",
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Storage/storageAccounts",
                            "name": "[variables('secondStorageName')]",
                            "apiVersion": "2017-06-01",
                            "location": "[parameters('secondStorageLocation')]",
                            "sku":{
                                "name": "Standard_LRS"
                            },
                            "kind": "Storage",
                            "properties": {
                            }
                        }
                    ]
                },
                "parameters": {}
            }
        },
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('firstStorageName')]",
            "apiVersion": "2017-06-01",
            "location": "[resourceGroup().location]",
            "sku":{
                "name": "Standard_LRS"
            },
            "kind": "Storage",
            "properties": {
            }
        }
    ]
}
```

如果您將 `resourceGroup` 設定為不存在的資源群組名稱，部署就會失敗。

## <a name="use-the-resourcegroup-and-subscription-functions"></a>使用 resourceGroup() 和 subscription() 函式

對於跨資源群組部署，[resourceGroup()](resource-group-template-functions-resource.md#resourcegroup) 和 [subscription()](resource-group-template-functions-resource.md#subscription) 函式會根據您指定巢狀範本的方式，以不同的方式進行解析。 

如果您將某個範本內嵌於另一個範本內，巢狀範本中的函式將會解析至父資源群組和訂用帳戶。 內嵌範本會使用下列格式：

```json
"apiVersion": "2017-05-10",
"name": "embeddedTemplate",
"type": "Microsoft.Resources/deployments",
"resourceGroup": "crossResourceGroupDeployment",
"properties": {
    "mode": "Incremental",
    "template": {
        ...
        resourceGroup() and subscription() refer to parent resource group/subscription
    }
}
```

如果您連結至個別範本，連結範本中的函式將會解析至巢狀資源群組和訂用帳戶。 連結範本會使用下列格式：

```json
"apiVersion": "2017-05-10",
"name": "linkedTemplate",
"type": "Microsoft.Resources/deployments",
"resourceGroup": "crossResourceGroupDeployment",
"properties": {
    "mode": "Incremental",
    "templateLink": {
        ...
        resourceGroup() and subscription() in linked template refer to linked resource group/subscription
    }
}
```

## <a name="example-templates"></a>範本的範例

下列範本示範多個資源群組部署。 表格後面會顯示要部署範本的指令碼。

|範本  |說明  |
|---------|---------|
|[跨訂用帳戶範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crosssubscription.json) |將一個儲存體帳戶部署至一個資源群組，並將一個儲存體帳戶部署至第二個資源群組。 第二個資源群組位於不同的訂用帳戶時，請包含訂用帳戶識別碼的值。 |
|[跨資源群組內容範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/crossresourcegroupproperties.json) |示範如何解析 `resourceGroup()` 函式。 它不會部署任何資源。 |

### <a name="powershell"></a>PowerShell

在 PowerShell 中，若要將兩個儲存體帳戶部署至**相同訂用帳戶**中的兩個資源群組，請使用：

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

New-AzureRmResourceGroup -Name $firstRG -Location southcentralus
New-AzureRmResourceGroup -Name $secondRG -Location eastus

New-AzureRmResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus
```

在 PowerShell 中，若要將兩個儲存體帳戶部署至**兩個訂用帳戶**，請使用：

```azurepowershell-interactive
$firstRG = "primarygroup"
$secondRG = "secondarygroup"

$firstSub = "<first-subscription-id>"
$secondSub = "<second-subscription-id>"

Select-AzureRmSubscription -Subscription $secondSub
New-AzureRmResourceGroup -Name $secondRG -Location eastus

Select-AzureRmSubscription -Subscription $firstSub
New-AzureRmResourceGroup -Name $firstRG -Location southcentralus

New-AzureRmResourceGroupDeployment `
  -ResourceGroupName $firstRG `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json `
  -storagePrefix storage `
  -secondResourceGroup $secondRG `
  -secondStorageLocation eastus `
  -secondSubscriptionID $secondSub
```

在 PowerShell 中，若要測試如何解析父代範本、內嵌範本和連結範本的**資源群組物件**，請使用：

```azurepowershell-interactive
New-AzureRmResourceGroup -Name parentGroup -Location southcentralus
New-AzureRmResourceGroup -Name inlineGroup -Location southcentralus
New-AzureRmResourceGroup -Name linkedGroup -Location southcentralus

New-AzureRmResourceGroupDeployment `
  -ResourceGroupName parentGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json
```

在上述範例中，**parentRG** 和 **inlineRG** 會同時解析為 **parentGroup**。 **linkedRG** 會解析為 **linkedGroup**。 前述範例的輸出為：

```powershell
 Name             Type                       Value
 ===============  =========================  ==========
 parentRG         Object                     {
                                               "id": "/subscriptions/<subscription-id>/resourceGroups/parentGroup",
                                               "name": "parentGroup",
                                               "location": "southcentralus",
                                               "properties": {
                                                 "provisioningState": "Succeeded"
                                               }
                                             }
 inlineRG         Object                     {
                                               "id": "/subscriptions/<subscription-id>/resourceGroups/parentGroup",
                                               "name": "parentGroup",
                                               "location": "southcentralus",
                                               "properties": {
                                                 "provisioningState": "Succeeded"
                                               }
                                             }
 linkedRG         Object                     {
                                               "id": "/subscriptions/<subscription-id>/resourceGroups/linkedGroup",
                                               "name": "linkedGroup",
                                               "location": "southcentralus",
                                               "properties": {
                                                 "provisioningState": "Succeeded"
                                               }
                                             }
```

### <a name="azure-cli"></a>Azure CLI

在 Azure CLI 中，若要將兩個儲存體帳戶部署至**相同訂用帳戶**中的兩個資源群組，請使用：

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

az group create --name $firstRG --location southcentralus
az group create --name $secondRG --location eastus
az group deployment create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=tfstorage secondResourceGroup=$secondRG secondStorageLocation=eastus
```

在 Azure CLI 中，若要將兩個儲存體帳戶部署至**兩個訂用帳戶**，請使用：

```azurecli-interactive
firstRG="primarygroup"
secondRG="secondarygroup"

firstSub="<first-subscription-id>"
secondSub="<second-subscription-id>"

az account set --subscription $secondSub
az group create --name $secondRG --location eastus

az account set --subscription $firstSub
az group create --name $firstRG --location southcentralus

az group deployment create \
  --name ExampleDeployment \
  --resource-group $firstRG \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crosssubscription.json \
  --parameters storagePrefix=storage secondResourceGroup=$secondRG secondStorageLocation=eastus secondSubscriptionID=$secondSub
```

在 Azure CLI 中，若要測試如何解析父代範本、內嵌範本和連結範本的**資源群組物件**，請使用：

```azurecli-interactive
az group create --name parentGroup --location southcentralus
az group create --name inlineGroup --location southcentralus
az group create --name linkedGroup --location southcentralus

az group deployment create \
  --name ExampleDeployment \
  --resource-group parentGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/crossresourcegroupproperties.json 
```

在上述範例中，**parentRG** 和 **inlineRG** 會同時解析為 **parentGroup**。 **linkedRG** 會解析為 **linkedGroup**。 前述範例的輸出為：

```azurecli
...
"outputs": {
  "inlineRG": {
    "type": "Object",
    "value": {
      "id": "/subscriptions/<subscription-id>/resourceGroups/parentGroup",
      "location": "southcentralus",
      "name": "parentGroup",
      "properties": {
        "provisioningState": "Succeeded"
      }
    }
  },
  "linkedRG": {
    "type": "Object",
    "value": {
      "id": "/subscriptions/<subscription-id>/resourceGroups/linkedGroup",
      "location": "southcentralus",
      "name": "linkedGroup",
      "properties": {
        "provisioningState": "Succeeded"
      }
    }
  },
  "parentRG": {
    "type": "Object",
    "value": {
      "id": "/subscriptions/<subscription-id>/resourceGroups/parentGroup",
      "location": "southcentralus",
      "name": "parentGroup",
      "properties": {
        "provisioningState": "Succeeded"
      }
    }
  }
},
...
```

## <a name="next-steps"></a>後續步驟

* 若要了解如何在您的範本中定義參數，請參閱[了解 Azure Resource Manager 範本的結構和語法](resource-group-authoring-templates.md)。
* 如需解決常見部署錯誤的秘訣，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
* 如需部署需要 SAS 權杖之範本的詳細資訊，請參閱[使用 SAS 權杖部署私人範本](resource-manager-powershell-sas-token.md)。
