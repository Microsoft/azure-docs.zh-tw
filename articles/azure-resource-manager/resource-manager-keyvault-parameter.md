---
title: 金鑰保存庫密碼與 Azure Resource Manager 範本 | Microsoft Docs
description: 示範如何在部署期間從金鑰保存庫中傳遞密碼做為參數。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
editor: tysonn
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/09/2018
ms.author: tomfitz
ms.openlocfilehash: 3a29319a0d478537dfc4905ee77865b8fea64587
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38598402"
---
# <a name="use-azure-key-vault-to-pass-secure-parameter-value-during-deployment"></a>在部署期間使用 Azure Key Vault 以傳遞安全的參數值

當您需要在部署期間，傳送安全值 (例如密碼) 做為參數時，您可以從 [Azure Key Vault](../key-vault/key-vault-whatis.md) 擷取值。 您可以藉由參考金鑰保存庫和參數檔案中的密碼來擷取值。 您只參考其金鑰保存庫識別碼，因此該值絕不會公開。 金鑰保存庫可以存在於與您部署的資源群組的不同訂用帳戶中。

## <a name="enable-access-to-the-secret"></a>啟用密碼的存取權

在範本部署期間，要存取金鑰保存庫有兩個重要條件存在：

1. 金鑰保存庫屬性 `enabledForTemplateDeployment` 必須為 `true`。
2. 部署範本的使用者必須有密碼的存取權。 使用者必須具有金鑰保存庫的 `Microsoft.KeyVault/vaults/deploy/action` 權限。 [擁有者](../role-based-access-control/built-in-roles.md#owner)和[參與者](../role-based-access-control/built-in-roles.md#contributor)角色皆可授與此權限。

將金鑰保存庫與[受控應用程式](../managed-applications/overview.md)的範本搭配使用時，您必須授與**設備資源提供者**服務主體的存取權。 如需詳細資訊，請參閱[在部署 Azure 受控應用程式時存取金鑰保存庫密碼](../managed-applications/key-vault-access.md) (英文)。


## <a name="deploy-a-key-vault-and-secret"></a>部署金鑰保存庫和密碼

若要建立金鑰保存庫與密碼，請使用 Azure CLI 或 PowerShell。 請注意，金鑰保存庫會針對範本部署啟用。 

對於 Azure CLI，請使用：

```azurecli-interactive
vaultname={your-unique-vault-name}
password={password-value}

az group create --name examplegroup --location 'South Central US'
az keyvault create \
  --name $vaultname \
  --resource-group examplegroup \
  --location 'South Central US' \
  --enabled-for-template-deployment true
az keyvault secret set --vault-name $vaultname --name examplesecret --value $password
```

對於 PowerShell，請使用：

```powershell
$vaultname = "{your-unique-vault-name}"
$password = "{password-value}"

New-AzureRmResourceGroup -Name examplegroup -Location "South Central US"
New-AzureRmKeyVault `
  -VaultName $vaultname `
  -ResourceGroupName examplegroup `
  -Location "South Central US" `
  -EnabledForTemplateDeployment
$secretvalue = ConvertTo-SecureString $password -AsPlainText -Force
Set-AzureKeyVaultSecret -VaultName $vaultname -Name "examplesecret" -SecretValue $secretvalue
```

## <a name="reference-a-secret-with-static-id"></a>使用靜態識別碼參考密碼

接收金鑰保存庫密碼的範本與其他任何範本都很像。 這是因為**您參考了參數檔案中的金鑰保存庫，而非範本。** 下圖顯示參數檔案如何參考祕密，並將該值傳遞至範本。

![靜態識別碼](./media/resource-manager-keyvault-parameter/statickeyvault.png)

例如，[下列範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/keyvaultparameter/sqlserver.json)部署了包含系統管理員密碼的 SQL 資料庫。 密碼參數會設定為安全字串。 但是，範本並未指定該值來自何處。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "adminLogin": {
      "type": "string"
    },
    "adminPassword": {
      "type": "securestring"
    },
    "sqlServerName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "name": "[parameters('sqlServerName')]",
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2015-05-01-preview",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {
        "administratorLogin": "[parameters('adminLogin')]",
        "administratorLoginPassword": "[parameters('adminPassword')]",
        "version": "12.0"
      }
    }
  ],
  "outputs": {
  }
}
```

現在，請為前述範本建立參數檔案。 在參數檔案中，指定符合範本中參數名稱的參數。 針對參數值，參考來自金鑰保存庫的密碼。 您可以藉由傳遞金鑰保存庫的資源識別碼和密碼的名稱來參考密碼。 在[下列參數檔案](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/keyvaultparameter/sqlserver.parameters.json)中，金鑰保存庫祕密必須已經存在，而且您要針對其資源識別碼提供靜態值。 在本機複製這個檔案，並設定訂用帳戶識別碼、保存庫名稱及 SQL 伺服器名稱。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "adminLogin": {
            "value": "exampleadmin"
        },
        "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "/subscriptions/<subscription-id>/resourceGroups/examplegroup/providers/Microsoft.KeyVault/vaults/<vault-name>"
              },
              "secretName": "examplesecret"
            }
        },
        "sqlServerName": {
            "value": "<your-server-name>"
        }
    }
}
```

如果需要使用目前版本以外的祕密版本，請使用 `secretVersion` 屬性。

```json
"secretName": "examplesecret",
"secretVersion": "cd91b2b7e10e492ebb870a6ee0591b68"
```

現在可部署範本，並在參數檔案中傳遞。 您可以使用 GitHub 中的範例範本，但您必須使用其值設定為您的環境的本機參數檔案。

對於 Azure CLI，請使用：

```azurecli-interactive
az group create --name datagroup --location "South Central US"
az group deployment create \
    --name exampledeployment \
    --resource-group datagroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver.json \
    --parameters @sqlserver.parameters.json
```

對於 PowerShell，請使用：

```powershell
New-AzureRmResourceGroup -Name datagroup -Location "South Central US"
New-AzureRmResourceGroupDeployment `
  -Name exampledeployment `
  -ResourceGroupName datagroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver.json `
  -TemplateParameterFile sqlserver.parameters.json
```

## <a name="reference-a-secret-with-dynamic-id"></a>使用動態識別碼參考密碼

上一節已說明如何傳遞金鑰保存庫密碼的靜態資源識別碼。 不過，在某些情況下，您需要參考會隨目前的部署而變化的金鑰保存庫密碼。 在該情況下，您將無法在參數檔中硬式編碼資源識別碼。 不幸的是，因為參數檔中不允許使用範本運算式，因此您無法在參數檔中以動態方式產生資源識別碼。

若要以動態方式產生金鑰保存庫祕密的資源識別碼，您必須將需要密碼的資源移動到連結的範本。 在父代範本中，您必須新增連結的範本，並傳入包含動態產生的資源識別碼的參數。 下圖顯示連結的範本中的參數如何參考祕密。

![動態識別碼](./media/resource-manager-keyvault-parameter/dynamickeyvault.png)

連結的範本必須可透過外部 URI 使用。 一般而言，您可將範本新增至儲存體帳戶，並透過類似 `https://<storage-name>.blob.core.windows.net/templatecontainer/sqlserver.json` 的 URI 進行存取。

[下列範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/keyvaultparameter/sqlserver-dynamic-id.json)會動態建立金鑰保存庫識別碼，並將它當作參數傳遞。 它會連結到 GitHub 中的[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/keyvaultparameter/sqlserver.json)。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "vaultName": {
        "type": "string"
      },
      "vaultResourceGroup": {
        "type": "string"
      },
      "secretName": {
        "type": "string"
      },
      "adminLogin": {
        "type": "string"
      },
      "sqlServerName": {
        "type": "string"
      }
    },
    "resources": [
    {
      "apiVersion": "2015-01-01",
      "name": "nestedTemplate",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "incremental",
        "templateLink": {
          "uri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver.json",
          "contentVersion": "1.0.0.0"
        },
        "parameters": {
          "adminPassword": {
            "reference": {
              "keyVault": {
                "id": "[resourceId(subscription().subscriptionId,  parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
              },
              "secretName": "[parameters('secretName')]"
            }
          },
          "adminLogin": { "value": "[parameters('adminLogin')]" },
          "sqlServerName": {"value": "[parameters('sqlServerName')]"}
        }
      }
    }],
    "outputs": {}
}
```

請部署上述範本，並提供參數值。 您可以使用 GitHub 中的範例範本，但您必須為您的環境提供參數值。

對於 Azure CLI，請使用：

```azurecli-interactive
az group create --name datagroup --location "South Central US"
az group deployment create \
    --name exampledeployment \
    --resource-group datagroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver-dynamic-id.json \
    --parameters vaultName=<your-vault> vaultResourceGroup=examplegroup secretName=examplesecret adminLogin=exampleadmin sqlServerName=<server-name>
```

對於 PowerShell，請使用：

```powershell
New-AzureRmResourceGroup -Name datagroup -Location "South Central US"
New-AzureRmResourceGroupDeployment `
  -Name exampledeployment `
  -ResourceGroupName datagroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/keyvaultparameter/sqlserver-dynamic-id.json `
  -vaultName <your-vault> -vaultResourceGroup examplegroup -secretName examplesecret -adminLogin exampleadmin -sqlServerName <server-name>
```

## <a name="next-steps"></a>後續步驟
* 如需金鑰保存庫的一般資訊，請參閱[開始使用 Azure 金鑰保存庫](../key-vault/key-vault-get-started.md)。
* 如需參考金鑰密碼的完整範例，請參閱 [金鑰保存庫範例](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples)。
