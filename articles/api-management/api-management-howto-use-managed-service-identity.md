---
title: 在 Azure API 管理中使用 Azure 受控服務識別 | Microsoft Docs
description: 了解如何在 API 管理中使用 Azure 受控服務識別
services: api-management
documentationcenter: ''
author: miaojiang
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 10/18/2017
ms.author: apimpm
ms.openlocfilehash: 98aa70935a3efbbe2edb07aade85fa3ea17ce786
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32150425"
---
# <a name="use-azure-managed-service-identity-in-azure-api-management"></a>在 Azure API 管理中使用 Azure 受控服務識別

本文說明如何為「API 管理」服務執行個體建立受控服務識別，以及如何存取其他資源。 Azure Active Directory (Azure AD) 所產生的受控服務識別可讓您的「API 管理」執行個體輕鬆且安全地存取其他受 Azure AD 保護的資源 (例如 Azure Key Vault)。 這個受控服務識別是由 Azure 所管理的，因此您不需要佈建或輪替任何祕密。 如需有關「Azure 受控服務識別」的詳細資訊，請參閱[受控服務識別 (適用於 Azure 資源)](../active-directory/msi-overview.md)。

## <a name="create-a-managed-service-identity-for-an-api-management-instance"></a>為 API 管理執行個體建立受控服務識別

### <a name="using-the-azure-portal"></a>使用 Azure 入口網站

若要在入口網站中設定受控服務識別，您需要先像平常一樣建立 API 管理執行個體，然後再啟用功能。

1. 像平常一樣在入口網站中建立 API 管理執行個體。 在入口網站中瀏覽至該應用程式。
2. 選取 [受控服務識別]。
3. 將 [向 Azure Active Directory 註冊應用程式] 切換為 [開啟]。 按一下 [儲存]。

![啟用 MSI](./media/api-management-msi/enable-msi.png)

### <a name="using-the-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本

您可以在資源定義中包含以下屬性，來建立具有身分識別的「API 管理」執行個體： 

```json
"identity" : {
    "type" : "SystemAssigned"
}
```

這會告訴 Azure 為您的「API 管理」執行個體建立及管理身分識別。 

例如，完整的 Azure Resource Manager 範本可能看起來如下：

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0"
    },
    "resources": [
        {
            "apiVersion": "2017-03-01",
            "name": "contoso",
            "type": "Microsoft.ApiManagement/service",
            "location": "[resourceGroup().location]",
            "tags": {},
            "sku": {
                "name": "Developer",
                "capacity": "1"
            },
            "properties": {
                "publisherEmail": "admin@contoso.com",
                "publisherName": "Contoso"
            },
            "identity": { 
                "type": "systemAssigned" 
            }
        }
    ]
}
```
## <a name="use-the-managed-service-identity-to-access-other-resources"></a>使用受控服務識別來存取其他資源

> [!NOTE]
> 目前，受控服務識別可用來從 API 管理自訂網域名稱的 Azure Key Vault 中取得憑證。 很快就會支援更多的案例。
> 
>


### <a name="obtain-a-certificate-from-azure-key-vault"></a>從 Azure Key Vault 取得憑證

#### <a name="prerequisites"></a>先決條件
1. 包含 pfx 憑證的 Key Vault 必須位於相同的 Azure 訂用帳戶中，以及與 API 管理服務相同的資源群組中。 這是 Azure Resource Manager 範本的需求。 
2. 秘密的內容類型必須是 application/x-pkcs12。 您可以使用下列指令碼來上傳憑證：

```powershell
$pfxFilePath = "PFX_CERTIFICATE_FILE_PATH" # Change this path 
$pwd = "PFX_CERTIFICATE_PASSWORD" # Change this password 
$flag = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable 
$collection = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2Collection 
$collection.Import($pfxFilePath, $pwd, $flag) 
$pkcs12ContentType = [System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12 
$clearBytes = $collection.Export($pkcs12ContentType) 
$fileContentEncoded = [System.Convert]::ToBase64String($clearBytes) 
$secret = ConvertTo-SecureString -String $fileContentEncoded -AsPlainText –Force 
$secretContentType = 'application/x-pkcs12' 
Set-AzureKeyVaultSecret -VaultName KEY_VAULT_NAME -Name KEY_VAULT_SECRET_NAME -SecretValue $Secret -ContentType $secretContentType
```

> [!Important]
> 如果未提供憑證的物件版本，將憑證上傳到 Key Vault 之後，「API 管理」將會自動取得較新版的憑證。 

下列範例顯示的 Azure Resource Manager 範本包含下列步驟：

1. 使用受控服務識別建立 API 管理執行個體。
2. 更新 Azure Key Vault 執行個體的存取原則，以允許「API 管理」執行個體從它取得祕密。
3. 透過來自 Key Vault 執行個體的憑證來設定自訂網域名稱，以更新「API 管理」執行個體。

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publisherEmail": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "The email address of the owner of the service"
            }
        },
        "publisherName": {
            "type": "string",
            "defaultValue": "Contoso",
            "minLength": 1,
            "metadata": {
                "description": "The name of the owner of the service"
            }
        },
        "sku": {
            "type": "string",
            "allowedValues": ["Developer",
            "Standard",
            "Premium"],
            "defaultValue": "Developer",
            "metadata": {
                "description": "The pricing tier of this API Management service"
            }
        },
        "skuCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
                "description": "The instance size of this API Management service."
            }
        },
        "keyVaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the vault"
            }
        },
        "proxyCustomHostname1": {
            "type": "string",
            "metadata": {
                "description": "Proxy Custom hostname."
            }
        },
        "keyVaultIdToCertificate": {
            "type": "string",
            "metadata": {
                "description": "Reference to the KeyVault certificate."
            }
        }
    },
    "variables": {
        "apiManagementServiceName": "[concat('apiservice', uniqueString(resourceGroup().id))]",
        "apimServiceIdentityResourceId": "[concat(resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName')),'/providers/Microsoft.ManagedIdentity/Identities/default')]"
    },
    "resources": [{
        "apiVersion": "2017-03-01",
        "name": "[variables('apiManagementServiceName')]",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {
            
        },
        "sku": {
            "name": "[parameters('sku')]",
            "capacity": "[parameters('skuCount')]"
        },
        "properties": {
            "publisherEmail": "[parameters('publisherEmail')]",
            "publisherName": "[parameters('publisherName')]"
        },
        "identity": {
            "type": "systemAssigned"
        }
    },
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2015-06-01",        
      "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
      ],
        "properties": {
            "accessPolicies": [{
                "tenantId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').tenantId]",
                "objectId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').principalId]",
                "permissions": {
                    "secrets": ["get"]
                }
            }]
        }
    },
    { 
      "apiVersion": "2017-05-10", 
      "name": "apimWithKeyVault", 
      "type": "Microsoft.Resources/deployments",
      "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
      ],
      "properties": { 
        "mode": "incremental", 
        "templateLink": {
          "uri": "https://raw.githubusercontent.com/solankisamir/arm-templates/master/basicapim.keyvault.json",
          "contentVersion": "1.0.0.0"
        }, 
        "parameters": {
            "publisherEmail": { "value": "[parameters('publisherEmail')]"},
            "publisherName": { "value": "[parameters('publisherName')]"},
            "sku": { "value": "[parameters('sku')]"},
            "skuCount": { "value": "[parameters('skuCount')]"},
            "proxyCustomHostname1": {"value" : "[parameters('proxyCustomHostname1')]"},
            "keyVaultIdToCertificate": {"value" : "[parameters('keyVaultIdToCertificate')]"}
        }
      } 
    }]
}
```

## <a name="next-steps"></a>後續步驟

深入了解「Azure 受控服務識別」：

* [受控服務識別 (適用於 Azure 資源)](../active-directory/msi-overview.md)
* [Azure 資源管理員範本](https://github.com/Azure/azure-quickstart-templates)

