---
title: 使用 Azure CLI 與範本部署資源 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Azure CLI，將資源部署至 Azure。 資源會定義在 Resource Manager 範本中。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
ms.assetid: 493b7932-8d1e-4499-912c-26098282ec95
ms.service: azure-resource-manager
ms.devlang: azurecli
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/06/2018
ms.author: tomfitz
ms.openlocfilehash: 8c3d208b12166a590c68753fb4f58c9bb6e55610
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225526"
---
# <a name="deploy-resources-with-resource-manager-templates-and-azure-cli"></a>使用 Resource Manager 範本與 Azure CLI 部署資源

此文章說明如何使用 Azure CLI 與 Resource Manager 範本，將您的資源部署至 Azure。 如果您不熟悉部署和管理 Azure 解決方案的相關概念，請參閱 [Azure Resource Manager 概觀](resource-group-overview.md)。  

您部署的 Resource Manager 範本可以是電腦上的本機檔案，或是位於類似 GitHub 等存放庫中的外部檔案。 您會在此文章中部署的範本，可在[範例範本](#sample-template)一節中取得，也會以 [GitHub 中儲存體帳戶範本 (英文)](https://github.com/Azure/azure-quickstart-templates/blob/master/101-storage-account-create/azuredeploy.json) 的形式提供。

[!INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

如果您沒有安裝 Azure CLI，可以使用 [Cloud Shell](#deploy-template-from-cloud-shell)。

## <a name="deploy-local-template"></a>部署本機範本

將資源部署至 Azure 時，您應該：

1. 登入您的 Azure 帳戶
2. 建立資源群組，作為已部署資源的容器。 資源群組的名稱只能包含英數字元、句點 (.)、底線、連字號及括弧。 最多可有 90 個字元。 不能以句點結束。
3. 將範本部署至資源群組，範本中定義要建立的資源

範本可以包含讓您自訂部署的參數。 例如，您可以提供針對特定環境 (例如開發、測試和生產) 量身訂做的值。 範例範本會定義儲存體帳戶 SKU 的參數。 

下列範例會建立資源群組，並從您的本機電腦部署範本：

```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
az group deployment create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

部署需要幾分鐘的時間才能完成。 完成後，您會看到包含結果的訊息：

```azurecli
"provisioningState": "Succeeded",
```

## <a name="deploy-external-template"></a>部署外部範本

您可能希望將 Resource Manager 範本儲存在外部位置，而不是儲存在您的本機電腦。 您可以將範本儲存在原始檔控制存放庫 (例如 GitHub) 中。 或者，您可以將它們儲存在 Azure 儲存體帳戶中，以在組織內共用存取。

若要部署外部範本，請使用 **template-uri** 參數。 在範例中使用 URI 以部署來自 GitHub 的範例範本。
   
```azurecli-interactive
az group create --name ExampleGroup --location "Central US"
az group deployment create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

上述範例針對範本需要可公開存取 URI，這適用於大部分的案例，因為您的範本不應該包含機密資料。 如果您需要指定機密資料 (例如系統管理員密碼)，請將該值以安全參數傳遞。 不過，如果不希望將範本公開存取，您可以將它儲存在私人儲存體容器中保護它。 如需部署需要共用存取簽章 (SAS) 權杖之範本的相關資訊，請參閱[使用 SAS 權杖部署私人範本](resource-manager-cli-sas-token.md)。

[!INCLUDE [resource-manager-cloud-shell-deploy.md](../../includes/resource-manager-cloud-shell-deploy.md)]

在 Cloud Shell 中使用下列命令︰

```azurecli-interactive
az group create --name examplegroup --location "South Central US"
az group deployment create --resource-group examplegroup \
  --template-uri <copied URL> \
  --parameters storageAccountType=Standard_GRS
```

## <a name="deploy-to-more-than-one-resource-group-or-subscription"></a>部署至多個資源群組或訂用帳戶

一般而言，您要將範本中的所有資源部署至單一資源群組。 不過，在某些情況下，您要將一組資源部署在一起，但將它們放在不同的資源群組或訂用帳戶中。 您在單一部署中只能部署至五個資源群組。 如需詳細資訊，請參閱[將 Azure 資源部署至多個訂用帳戶或資源群組](resource-manager-cross-resource-group-deployment.md)。

## <a name="redeploy-when-deployment-fails"></a>部署失敗時重新部署

針對失敗的部署，您可以將部署歷程記錄中先前的部署指定為自動進行重新部署。 若要使用這個選項，您的部署必須有唯一的名稱，以便在歷程記錄中進行識別。 如果您沒有唯一的名稱，則目前失敗的部署可能會覆寫歷程記錄中先前成功的部署。 您只可以使用此選項搭配根層級部署。 從巢狀範本部署不適用於重新部署。

若要重新部署最後一個成功的部署，請新增 `--rollback-on-error` 參數作為旗標。

```azurecli-interactive
az group deployment create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error
```

若要重新部署特定部署，請使用 `--rollback-on-error` 參數，並提供部署的名稱。

```azurecli-interactive
az group deployment create \
  --name ExampleDeployment02 \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS \
  --rollback-on-error ExampleDeployment01
```

指定的部署必須成功。

## <a name="parameter-files"></a>參數檔案

相對於在您的指令碼中將參數做為內嵌值傳遞，使用包含該參數值的 JSON 檔案可能較為容易。 參數檔必須是下列格式︰

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
     "storageAccountType": {
         "value": "Standard_GRS"
     }
  }
}
```

請注意，參數區段包含符合於範本中所定義之參數 (storageAccountType) 的參數名稱。 參數檔案包含參數的值。 這個值會在部署期間自動傳遞至範本。 您可以針對不同的部署案例建立多個參數檔案，然後傳遞適當的參數檔案。 

複製上述的範例，然後將它另存為名叫 `storage.parameters.json` 的檔案。

若要傳遞本機參數檔案，請使用 `@` 來指定名為 storage.parameters.json 的本機檔案。

```azurecli-interactive
az group deployment create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

## <a name="test-a-template-deployment"></a>測試範本部署

若要測試您的範本和參數值而不實際部署任何資源，請使用 [az group deployment validate](/cli/azure/group/deployment#az-group-deployment-validate)。 

```azurecli-interactive
az group deployment validate \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

如果沒有偵測到任何錯誤，命令會傳回有關測試部署的資訊。 請特別注意到 **error** 值為 null。

```azurecli
{
  "error": null,
  "properties": {
      ...
```

如果偵測到錯誤，命令會傳回錯誤訊息。 例如，嘗試針對儲存體帳戶 SKU 傳遞不正確的值，則會傳回下列錯誤：

```azurecli
{
  "error": {
    "code": "InvalidTemplate",
    "details": null,
    "message": "Deployment template validation failed: 'The provided value 'badSKU' for the template parameter 
      'storageAccountType' at line '13' and column '20' is not valid. The parameter value is not part of the allowed 
      value(s): 'Standard_LRS,Standard_ZRS,Standard_GRS,Standard_RAGRS,Premium_LRS'.'.",
    "target": null
  },
  "properties": null
}
```

如果您的範本有語法錯誤，命令會傳回錯誤，指出無法剖析範本。 訊息會指出剖析錯誤的行號和位置。

```azurecli
{
  "error": {
    "code": "InvalidTemplate",
    "details": null,
    "message": "Deployment template parse failed: 'After parsing a value an unexpected character was encountered:
      \". Path 'variables', line 31, position 3.'.",
    "target": null
  },
  "properties": null
}
```

## <a name="sample-template"></a>範例範本

下列範本適用於此文章中的範例。 請複製它並另存為名叫 storage.json 的檔案。 若要了解如何建立此範本，請參閱[建立第一個 Azure Resource Manager 範本](resource-manager-create-first-template.md)。  

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
    }
  },
  "variables": {
    "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "apiVersion": "2016-01-01",
      "location": "[resourceGroup().location]",
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

## <a name="next-steps"></a>後續步驟
* 此主題中的範例會將資源部署到您預設訂用帳戶中的資源群組。 若要使用不同的訂用帳戶，請參閱[管理多個 Azure 訂用帳戶](/cli/azure/manage-azure-subscriptions-azure-cli)。
* 若要指定如何處理存在於資源群組中但尚未定義於範本中的資源，請參閱 [Azure Resource Manager 部署模式](deployment-modes.md)。
* 若要了解如何在您的範本中定義參數，請參閱[了解 Azure Resource Manager 範本的結構和語法](resource-group-authoring-templates.md)。
* 如需解決常見部署錯誤的秘訣，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
* 如需部署需要 SAS 權杖之範本的詳細資訊，請參閱[使用 SAS 權杖部署私人範本](resource-manager-cli-sas-token.md)。
* 若要安全地在多個區域推出您的服務，請參閱 [Azure 部署管理員](deployment-manager-overview.md)。
