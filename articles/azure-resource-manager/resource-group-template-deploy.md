---
title: 使用 PowerShell 與範本部署資源 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Azure PowerShell，將資源部署至 Azure。 資源會定義在 Resource Manager 範本中。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 55903f35-6c16-4c6d-bf52-dbf365605c3f
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/06/2017
ms.author: tomfitz
ms.openlocfilehash: e6dd119bcd72fa6a5c7515150bfa85f015ee5c0e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223086"
---
# <a name="deploy-resources-with-resource-manager-templates-and-azure-powershell"></a>使用 Resource Manager 範本與 Azure PowerShell 來部署資源

此文章說明如何使用 Azure PowerShell 與 Resource Manager 範本，將您的資源部署至 Azure。 如果您不熟悉部署和管理 Azure 解決方案的相關概念，請參閱 [Azure Resource Manager 概觀](resource-group-overview.md)。

您所部署的 Resource Manager 範本可以是電腦上的本機檔案，或是位於存放庫 (例如 GitHub) 的外部檔案。 在此文章的[範例範本](#sample-template)區段中會提供您要部署的範本，也可使用 [GitHub 中的儲存體帳戶範本](https://github.com/Azure/azure-quickstart-templates/blob/master/101-storage-account-create/azuredeploy.json)。

您可以視需要使用 [Azure PowerShell 指南 (英文)](/powershell/azure/overview) 中的指示來安裝 Azure PowerShell 模組，然後執行 `Connect-AzureRmAccount` 來建立與 Azure 的連線。

<a id="deploy-local-template" />

## <a name="deploy-a-template-from-your-local-machine"></a>從本機電腦部署範本

將資源部署至 Azure 時，您應該：

1. 登入您的 Azure 帳戶
2. 建立資源群組，作為已部署資源的容器。 資源群組的名稱只能包含英數字元、句點 (.)、底線、連字號及括弧。 最多可有 90 個字元。 不能以句點結束。
3. 將範本部署至資源群組，範本中定義要建立的資源

範本可以包含讓您自訂部署的參數。 例如，您可以提供針對特定環境 (例如開發、測試和生產) 量身訂做的值。 範例範本會定義儲存體帳戶 SKU 的參數。

下列範例會建立資源群組，並從您的本機電腦部署範本：

```powershell
Connect-AzureRmAccount

Select-AzureRmSubscription -SubscriptionName <yourSubscriptionName>
 
New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "South Central US"
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json -storageAccountType Standard_GRS
```

部署需要幾分鐘的時間才能完成。 完成後，您會看到包含結果的訊息：

```powershell
ProvisioningState       : Succeeded
```

## <a name="deploy-a-template-from-an-external-source"></a>從外部來源部署範本

您可能希望將 Resource Manager 範本儲存在外部位置，而不是儲存在您的本機電腦。 您可以將範本儲存在原始檔控制存放庫 (例如 GitHub) 中。 或者，您可以將它們儲存在 Azure 儲存體帳戶中，讓組織共用存取。

若要部署外部範本，請使用 **TemplateUri** 參數。 使用範本中的 URI 部署 GitHub 上的範例範本。

```powershell
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -storageAccountType Standard_GRS
```

上述範例針對範本需要可公開存取 URI，這適用於大部分的案例，因為您的範本不應該包含機密資料。 如果您需要指定機密資料 (例如系統管理員密碼)，請將該值以安全參數傳遞。 不過，如果不希望將範本公開存取，您可以將它儲存在私人儲存體容器中以保護它。 如需部署需要共用存取簽章 (SAS) 權杖之範本的相關資訊，請參閱[使用 SAS 權杖部署私人範本](resource-manager-powershell-sas-token.md)。

[!INCLUDE [resource-manager-cloud-shell-deploy.md](../../includes/resource-manager-cloud-shell-deploy.md)]

在 Cloud Shell 中使用下列命令︰

```powershell
New-AzureRmResourceGroup -Name ExampleResourceGroup -Location "South Central US"
New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri <copied URL> `
  -storageAccountType Standard_GRS
```

## <a name="deploy-to-more-than-one-resource-group-or-subscription"></a>部署至多個資源群組或訂用帳戶

一般而言，您要將範本中的所有資源部署至單一資源群組。 不過，在某些情況下，您要將一組資源部署在一起，但將它們放在不同的資源群組或訂用帳戶中。 您在單一部署中只能部署至五個資源群組。 如需詳細資訊，請參閱[將 Azure 資源部署至多個訂用帳戶或資源群組](resource-manager-cross-resource-group-deployment.md)。

<a id="parameter-file" />

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

複製上述的範例，然後另存檔案，名稱為 `storage.parameters.json`。

若要傳遞本機參數檔案，請使用 **TemplateParameterFile** 參數︰

```powershell
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

若要傳遞外部參數檔案，請使用 **TemplateParameterUri** 參數︰

```powershell
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -TemplateParameterUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.parameters.json
```

您可以在相同的部署作業中使用內嵌參數和本機參數檔案。 例如，您可以在部署期間指定本機參數檔案中的某些值，並新增其他內嵌值。 如果您同時為本機檔案中和內嵌的參數提供值，內嵌值的優先順序較高。

不過，當使用外部參數檔案時，您無法傳遞內嵌或本機檔案的其他值。 當您指定 **TemplateParameterUri** 參數中的參數檔案時，所有內嵌參數都會被忽略。 請提供外部檔案中的所有參數值。 如果您的範本包含不能包含在參數檔案中的機密值，請將該值新增至金鑰保存庫，或以動態方式提供所有內嵌參數值。

如果您範本所含的參數名稱與 PowerShell 命令中的其中一個參數一樣，PowerShell 會以加上後置 **FromTemplate** 的方式呈現您範本中的參數。 例如，範本中名為 **ResourceGroupName** 的參數會與 [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) Cmdlet 中的 **ResourceGroupName** 參數發生衝突。 系統會提示您為 **ResourceGroupNameFromTemplate** 提供值。 一般而言，在為參數命名時，請勿使用與部署作業所用參數相同的名稱，以避免發生這種混淆的情形。

## <a name="test-a-template-deployment"></a>測試範本部署

若要測試您的範本和參數值而不實際部署任何資源，請使用 [Test-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/test-azurermresourcegroupdeployment)。 

```powershell
Test-AzureRmResourceGroupDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\storage.json -storageAccountType Standard_GRS
```

如果未偵測到錯誤，命令即完成，不會有回應。 如果偵測到錯誤，命令會傳回錯誤訊息。 例如，嘗試針對儲存體帳戶 SKU 傳遞不正確的值，則會傳回下列錯誤：

```powershell
Test-AzureRmResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\storage.json -storageAccountType badSku

Code    : InvalidTemplate
Message : Deployment template validation failed: 'The provided value 'badSku' for the template parameter 'storageAccountType'
          at line '15' and column '24' is not valid. The parameter value is not part of the allowed value(s):
          'Standard_LRS,Standard_ZRS,Standard_GRS,Standard_RAGRS,Premium_LRS'.'.
Details :
```

如果您的範本有語法錯誤，命令會傳回錯誤，指出無法剖析範本。 訊息會指出剖析錯誤的行號和位置。

```powershell
Test-AzureRmResourceGroupDeployment : After parsing a value an unexpected character was encountered: 
  ". Path 'variables', line 31, position 3.
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
* 此主題中的範例會將資源部署到您預設訂用帳戶中的資源群組。 若要使用不同的訂用帳戶，請參閱[管理多個 Azure 訂用帳戶](/powershell/azure/manage-subscriptions-azureps)。
* 若要指定如何處理存在於資源群組中但尚未定義於範本中的資源，請參閱 [Azure Resource Manager 部署模式](deployment-modes.md)。
* 若要了解如何在您的範本中定義參數，請參閱[了解 Azure Resource Manager 範本的結構和語法](resource-group-authoring-templates.md)。
* 如需解決常見部署錯誤的秘訣，請參閱[使用 Azure Resource Manager 針對常見的 Azure 部署錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
* 如需部署需要 SAS 權杖之範本的詳細資訊，請參閱[使用 SAS 權杖部署私人範本](resource-manager-powershell-sas-token.md)。
* 若要安全地在多個區域推出您的服務，請參閱 [Azure 部署管理員](deployment-manager-overview.md)。
