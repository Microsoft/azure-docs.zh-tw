---
title: 使用 Azure Resource Manager 範本建立 Azure Machine Learning 測試 | Microsoft Docs
description: 本文會提供範例，說明如何使用 Azure Resource Manager 範本建立 Azure Machine Learning 測試帳戶。
services: machine-learning
author: hning86
ms.author: haining
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.topic: article
ms.date: 11/14/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 24ed164f4a1dfdb9a3913efa78fe58fab2b53696
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991958"
---
# <a name="configure-the-azure-machine-learning-experimentation-service"></a>設定 Azure Machine Learning 測試服務

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

## <a name="overview"></a>概觀
Azure Machine Learning 測試服務帳戶、工作區及專案都是 Azure 資源。 因此，您可以使用 Resource Manager 範本部署這些資源。 Resource Manager 範本是 JSON 檔案，該檔案定義您需要為您的解決方案部署的資源。 若要了解部署和管理 Azure 解決方案的相關概念，請參閱 [Azure Resource Manager 概觀](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview)。

## <a name="deploy-a-template"></a>部署範本
若要部署範本，您只需要在 Azure 命令列介面或在 Azure 入口網站中進行幾個步驟。

### <a name="deploy-a-template-from-the-command-line"></a>從命令列部署範本
若使用命令列介面，只須單一命令即可在現有資源群組中部署範本。
請參閱以下內容，以了解如何建立範本。

```sh
# Login and validate your are in the right subscription context
az login

# Create a new resource group (you can use an existing one)
az group create --name <resource group name> --location <supported Azure region>
az group deployment create -n testdeploy -g <resource group name> --template-file <template-file.json> --parameters <parameters.json>
```

### <a name="deploy-a-template-from-the-azure-portal"></a>從 Azure 入口網站部署範本
如果您想要的話，也可以使用 Azure 入口網站來建立及部署範本。 做法如下所示：

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [所有服務]，並搜尋「範本」。
3. 選取 [範本]。
4. 按一下 [+ 新增] 並複製範本資訊。 
5. 選取您在步驟 4 建立的範本，然後按一下 [部署]。


## <a name="create-a-template-from-an-existing-azure-resource-in-the-azure-portal"></a>在 Azure 入口網站中從現有 Azure 資源建立範本
如果您已有可用的 Azure Machine Learning 測試帳戶，您可以在 [Azure 入口網站](https://portal.azure.com)中，透過該資源產生範本。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 Azure Machine Learning 測試帳戶。
2. 在 [設定] 下，按一下 [自動化指令碼]。
3. 下載範本。 

或者，您可以手動編輯範本檔案。 如需範本和參數檔案的範例，請參閱以下內容。 

### <a name="template-file-example"></a>範本檔案範例
使用以下內容建立名為「template-file.json」的檔案。 

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "parameters": {
        "accountName": {
            "type": "string",
            "metadata": {
                "description": "Name of the machine learning experimentation team account"
            }
        },
        "location": {
            "type": "string",
            "metadata": {
                "description": "Location of the machine learning experimentation account and other dependent resources."
            }
        },
        "storageAccountSku": {
            "type": "string",
            "defaultValue": "Standard_LRS",
            "metadata": {
                "description": "Sku of the storage account"
            }
        }
    },
    "variables": {
        "mlexpVersion": "2017-05-01-preview",
        "stgResourceId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
    },
    "resources": [
        {
            "name": "[parameters('accountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[parameters('location')]",
            "apiVersion": "2016-12-01",
            "tags": {
                "mlteamAccount": "[parameters('accountName')]"
            },
            "sku": {
                "name": "[parameters('storageAccountSku')]"
            },
            "kind": "Storage",
            "properties": {
                "encryption": {
                    "services": {
                        "blob": {
                            "enabled": true
                        }
                    },
                    "keySource": "Microsoft.Storage"
                }
            }
        },
        {
            "apiVersion": "[variables('mlexpVersion')]",
            "type": "Microsoft.MachineLearningExperimentation/accounts",
            "name": "[parameters('accountName')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.Storage/storageAccounts', parameters('accountName'))]"
            ],
            "properties": {
                "storageAccount": {
                    "storageAccountId": "[variables('stgResourceId')]",
                    "accessKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('accountName')), '2016-12-01').keys[0].value]"
                }
            }
        }
    ]
}
```

### <a name="parameters"></a>參數 
使用以下內容建立檔案，並將它儲存為 <parameters.json>。 

有三個值可供變更。 
* AccountName：測試帳戶的名稱。
* Location：其中一個支援的 Azure 區域。
* StorageAccountSKU：Azure ML 只支援標準儲存體，不支援進階儲存體。 如需儲存體的詳細資訊，請參閱[儲存體簡介](https://docs.microsoft.com/azure/storage/common/storage-introduction)。 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
     "accountName": {
         "value": "MyExperimentationAccount"
     },
     "location": {
         "value": "eastus2"
     },
     "storageAccountSku": {
         "value": "Standard_LRS"
     }
  }
}
```

## <a name="next-steps"></a>後續步驟
* [建立與安裝 Azure Machine Learning](quickstart-installation.md)
