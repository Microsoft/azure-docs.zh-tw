---
title: Azure Resource Manager 範本函式 - 資源 | Microsoft Docs
description: 描述 Azure Resource Manager 範本中用來擷取資源相關值的函式。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/06/2018
ms.author: tomfitz
ms.openlocfilehash: 4e454030f77a22236da18c8d8a5215667784f7c5
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43301435"
---
# <a name="resource-functions-for-azure-resource-manager-templates"></a>Azure Resource Manager 範本的資源函式

資源管理員提供下列函式以取得資源值：

* [listAccountSas](#list)
* [listKeys](#listkeys)
* [listSecrets](#list)
* [list*](#list)
* [提供者](#providers)
* [reference](#reference)
* [resourceGroup](#resourcegroup)
* [resourceId](#resourceid)
* [訂用帳戶](#subscription)

若要從參數、變數或目前的部署中取得值，請參閱 [部署值函式](resource-group-template-functions-deployment.md)。

<a id="listkeys" />
<a id="list" />

## <a name="listaccountsas-listkeys-listsecrets-and-list"></a>listAccountSas、listKeys、listSecrets 和 list*
`listAccountSas(resourceName or resourceIdentifier, apiVersion, functionValues)`

`listKeys(resourceName or resourceIdentifier, apiVersion)`

`listSecrets(resourceName or resourceIdentifier, apiVersion)`

`list{Value}(resourceName or resourceIdentifier, apiVersion)`

對支援 list 作業的任何資源類型傳回值。 最常用的是 `listKeys` 和 `listSecrets`。 

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |字串 |資源的唯一識別碼。 |
| apiVersion |是 |字串 |資源執行階段狀態的 API 版本。 一般而言，格式為 **yyyy-mm-dd**。 |
| functionValues |否 |物件 | 具有函式值的物件。 只針對以下函式提供此物件：可支援在儲存體帳戶上接收具有參數值的物件，例如 **listAccountSas**。 | 

### <a name="return-value"></a>傳回值

從 listKeys 傳回的物件具有下列格式︰

```json
{
  "keys": [
    {
      "keyName": "key1",
      "permissions": "Full",
      "value": "{value}"
    },
    {
      "keyName": "key2",
      "permissions": "Full",
      "value": "{value}"
    }
  ]
}
```

其他 list 函式具有不同的傳回格式。 若要查看函式的格式，請將函式放入 [輸出] 區段，如範本範例所示。 

### <a name="remarks"></a>備註

開頭為 **list** 的任何作業都可在您的範本中用為函式。 可用作業不只包含 listKeys，還包含像 `list`、`listAdminKeys` 和 `listStatus` 等作業。 [List Account SAS](/rest/api/storagerp/storageaccounts#StorageAccounts_ListAccountSAS) 作業需要要求本文參數，例如 *signedExpiry*。 若要在範本中使用此函式，請提供具有本文參數值的物件。

為判斷哪一個資源類型具有 list 作業，您可以使用下列選項：

* 檢視資源提供者的 [REST API 作業](/rest/api/)，並尋找 list 作業。 例如，儲存體帳戶具有 [listKeys 作業](/rest/api/storagerp/storageaccounts#StorageAccounts_ListKeys)。
* 使用 [Get-AzureRmProviderOperation](/powershell/module/azurerm.resources/get-azurermprovideroperation) PowerShell Cmdlet。 下列範例會取得儲存體帳戶的所有 list 作業︰

  ```powershell
  Get-AzureRmProviderOperation -OperationSearchString "Microsoft.Storage/*" | where {$_.Operation -like "*list*"} | FT Operation
  ```
* 使用下列 Azure CLI 命令可只篩選出 list 作業：

  ```azurecli
  az provider operation show --namespace Microsoft.Storage --query "resourceTypes[?name=='storageAccounts'].operations[].name | [?contains(@, 'list')]"
  ```

使用資源名稱或 [resourceId 函式](#resourceid)來指定資源。 在部署所參考資源的相同範本中使用這個函式時，請使用資源名稱。

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/listkeys.json)顯示如何在 outputs 區段中從儲存體帳戶傳回主要和次要金鑰。 它也會傳回儲存體帳戶的 SAS 權杖。 為了取得該權杖，它會將物件傳遞至 listAccountSas 函式。 此範例的用意是要示範如何使用清單函式。 一般而言，您會在資源值中使用 SAS 權杖，而非將它傳回作為輸出值。 輸出值會儲存於部署歷程記錄，並不安全。 您必須指定未來的到期時間，部署才能成功。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storagename": {
            "type": "string"
        },
        "location": {
            "type": "string",
            "defaultValue": "southcentralus"
        },
        "requestContent": {
            "type": "object",
            "defaultValue": {
                "signedServices": "b",
                "signedResourceType": "c",
                "signedPermission": "r",
                "signedExpiry": "2018-08-20T11:00:00Z",
                "signedResourceTypes": "s"
            }
        }
    },
    "resources": [
        {
            "apiVersion": "2018-02-01",
            "name": "[parameters('storagename')]",
            "location": "[parameters('location')]",
            "type": "Microsoft.Storage/storageAccounts",
            "sku": {
                "name": "Standard_LRS"
            },
            "kind": "StorageV2",
            "properties": {
                "supportsHttpsTrafficOnly": false,
                "accessTier": "Hot",
                "encryption": {
                    "services": {
                        "blob": {
                            "enabled": true
                        },
                        "file": {
                            "enabled": true
                        }
                    },
                    "keySource": "Microsoft.Storage"
                }
            },
            "dependsOn": []
        }
    ],
    "outputs": {
        "keys": {
            "type": "object",
            "value": "[listKeys(parameters('storagename'), '2018-02-01')]"
        },
        "accountSAS": {
            "type": "object",
            "value": "[listAccountSas(parameters('storagename'), '2018-02-01', parameters('requestContent'))]"
        }
    }
}
``` 

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/listkeys.json --parameters storagename=<your-storage-account>
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/listkeys.json -storagename <your-storage-account>
```

<a id="providers" />

## <a name="providers"></a>提供者
`providers(providerNamespace, [resourceType])`

傳回資源提供者和其所支援資源類型的相關資訊。 如果未提供資源類型，則函式會傳回資源提供者所有的支援類型。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| providerNamespace |是 |字串 |提供者的命名空間 |
| resourceType |否 |字串 |所指定命名空間內的資源類型。 |

### <a name="return-value"></a>傳回值

每個支援類型都會以下列格式傳回： 

```json
{
    "resourceType": "{name of resource type}",
    "locations": [ all supported locations ],
    "apiVersions": [ all supported API versions ]
}
```

不保證所傳回的值會有陣列順序。

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/providers.json)顯示如何使用 provider 函式：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "providerNamespace": {
            "type": "string"
        },
        "resourceType": {
            "type": "string"
        }
    },
    "resources": [],
    "outputs": {
        "providerOutput": {
            "value": "[providers(parameters('providerNamespace'), parameters('resourceType'))]",
            "type" : "object"
        }
    }
}
```

針對 **Microsoft.Web** 資源提供者和**網站**資源類型，上述範例會以下列格式傳回物件：

```json
{
  "resourceType": "sites",
  "locations": [
    "South Central US",
    "North Europe",
    "West Europe",
    "Southeast Asia",
    ...
  ],
  "apiVersions": [
    "2016-08-01",
    "2016-03-01",
    "2015-08-01-preview",
    "2015-08-01",
    ...
  ]
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/providers.json --parameters providerNamespace=Microsoft.Web resourceType=sites
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/providers.json -providerNamespace Microsoft.Web -resourceType sites
```

<a id="reference" />

## <a name="reference"></a>reference
`reference(resourceName or resourceIdentifier, [apiVersion], ['Full'])`

傳回代表資源執行階段狀態的物件。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |字串 |資源的名稱或唯一識別碼。 |
| apiVersion |否 |字串 |指定的資源的 API 版本。 如果在相同的範本內未供應資源，則請包含此參數。 一般而言，格式為 **yyyy-mm-dd**。 |
| 'Full' |否 |字串 |值，指定是否要傳回完整資源物件。 如果您未指定 `'Full'`，則只會傳回資源的屬性物件。 完整物件包括例如資源識別碼和位置的值。 |

### <a name="return-value"></a>傳回值

每種資源類型都會針對 reference 函式傳回不同屬性。 此函式不會傳回單一的預先定義格式。 此外，傳回值會根據您是否指定完整物件而不同。 若要查看資源類型的屬性，請在輸出區段中傳回物件，如範例所示。

### <a name="remarks"></a>備註

reference 函數會從執行階段狀態衍生其值，因此不能用在 variables 區段中。 它可以用於範本或[連結範本](resource-group-linked-templates.md#link-or-nest-a-template)的輸出區段。 無法用於[巢狀範本](resource-group-linked-templates.md#link-or-nest-a-template)的輸出區段。 若要傳回巢狀範本中已部署資源的值，請將巢狀範本轉換成連結範本。 

如果在相同的範本內佈建所參考的資源且您會依其名稱 (而非資源識別碼) 來參考該資源，則可使用 reference 函式，隱含地宣告某一個資源相依於另一個資源。 您不需要同時使用 dependsOn 屬性。 所參考的資源完成部署之前不會評估函式。

若要查看資源類型的屬性名稱和值，請建立一個會在 outputs 區段中傳回物件的範本。 如果您有一個該類型的現有資源，您的範本就會傳回物件，而不會部署任何新資源。 

一般而言，您可以使用 **reference** 函式從物件傳回特定值，例如 Blob 端點 URI 或完整網域名稱。

```json
"outputs": {
    "BlobUri": {
        "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01').primaryEndpoints.blob]",
        "type" : "string"
    },
    "FQDN": {
        "value": "[reference(concat('Microsoft.Network/publicIPAddresses/', parameters('ipAddressName')), '2016-03-30').dnsSettings.fqdn]",
        "type" : "string"
    }
}
```

當您需要並非屬性結構描述一部分的資源值時，使用 `'Full'`。 例如，若要設定金鑰保存庫存取原則，請取得虛擬機器的身分識別屬性。

```json
{
  "type": "Microsoft.KeyVault/vaults",
  "properties": {
    "tenantId": "[reference(concat('Microsoft.Compute/virtualMachines/', variables('vmName')), '2017-03-30', 'Full').identity.tenantId]",
    "accessPolicies": [
      {
        "tenantId": "[reference(concat('Microsoft.Compute/virtualMachines/', variables('vmName')), '2017-03-30', 'Full').identity.tenantId]",
        "objectId": "[reference(concat('Microsoft.Compute/virtualMachines/', variables('vmName')), '2017-03-30', 'Full').identity.principalId]",
        "permissions": {
          "keys": [
            "all"
          ],
          "secrets": [
            "all"
          ]
        }
      }
    ],
    ...
```

如需上述範本的完整範例，請參閱 [Windows 到 Key Vault](https://github.com/rjmax/AzureSaturday/blob/master/Demo02.ManagedServiceIdentity/demo08.msiWindowsToKeyvault.json)。 [Linux](https://github.com/rjmax/AzureSaturday/blob/master/Demo02.ManagedServiceIdentity/demo07.msiLinuxToArm.json) 也有類似範例。

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/referencewithstorage.json)部署資源，並會參考該資源。

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageAccountName": { 
          "type": "string"
      }
  },
  "resources": [
    {
      "name": "[parameters('storageAccountName')]",
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2016-12-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {
      }
    }
  ],
  "outputs": {
      "referenceOutput": {
          "type": "object",
          "value": "[reference(parameters('storageAccountName'))]"
      },
      "fullReferenceOutput": {
        "type": "object",
        "value": "[reference(parameters('storageAccountName'), '2016-12-01', 'Full')]"
      }
    }
}
``` 

上述範例會傳回兩個物件。 屬性物件會使用下列格式：

```json
{
   "creationTime": "2017-10-09T18:55:40.5863736Z",
   "primaryEndpoints": {
     "blob": "https://examplestorage.blob.core.windows.net/",
     "file": "https://examplestorage.file.core.windows.net/",
     "queue": "https://examplestorage.queue.core.windows.net/",
     "table": "https://examplestorage.table.core.windows.net/"
   },
   "primaryLocation": "southcentralus",
   "provisioningState": "Succeeded",
   "statusOfPrimary": "available",
   "supportsHttpsTrafficOnly": false
}
```

完整物件會使用下列格式：

```json
{
  "apiVersion":"2016-12-01",
  "location":"southcentralus",
  "sku": {
    "name":"Standard_LRS",
    "tier":"Standard"
  },
  "tags":{},
  "kind":"Storage",
  "properties": {
    "creationTime":"2017-10-09T18:55:40.5863736Z",
    "primaryEndpoints": {
      "blob":"https://examplestorage.blob.core.windows.net/",
      "file":"https://examplestorage.file.core.windows.net/",
      "queue":"https://examplestorage.queue.core.windows.net/",
      "table":"https://examplestorage.table.core.windows.net/"
    },
    "primaryLocation":"southcentralus",
    "provisioningState":"Succeeded",
    "statusOfPrimary":"available",
    "supportsHttpsTrafficOnly":false
  },
  "subscriptionId":"<subscription-id>",
  "resourceGroupName":"functionexamplegroup",
  "resourceId":"Microsoft.Storage/storageAccounts/examplestorage",
  "referenceApiVersion":"2016-12-01",
  "condition":true,
  "isConditionTrue":true,
  "isTemplateResource":false,
  "isAction":false,
  "provisioningOperation":"Read"
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/referencewithstorage.json --parameters storageAccountName=<your-storage-account>
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/referencewithstorage.json -storageAccountName <your-storage-account>
```

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/reference.json)參照了本範本中未部署之儲存體帳戶。 此儲存體帳戶已經存在在同樣的資源群組中。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "storageAccountName": {
            "type": "string"
        }
    },
    "resources": [],
    "outputs": {
        "ExistingStorage": {
            "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01')]",
            "type" : "object"
        }
    }
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/reference.json --parameters storageAccountName=<your-storage-account>
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/reference.json -storageAccountName <your-storage-account>
```

<a id="resourcegroup" />

## <a name="resourcegroup"></a>resourceGroup
`resourceGroup()`

傳回代表目前資源群組的物件。 

### <a name="return-value"></a>傳回值

傳回的物件會使用下列格式：

```json
{
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
  "name": "{resourceGroupName}",
  "location": "{resourceGroupLocation}",
  "tags": {
  },
  "properties": {
    "provisioningState": "{status}"
  }
}
```

### <a name="remarks"></a>備註

resourceGroup 函式的常見用法是在和資源群組相同的位置中建立資源。 下列範例使用資源群組位置來指派網站的位置。

```json
"resources": [
   {
      "apiVersion": "2016-08-01",
      "type": "Microsoft.Web/sites",
      "name": "[parameters('siteName')]",
      "location": "[resourceGroup().location]",
      ...
   }
]
```

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/resourcegroup.json)會傳回資源群組的屬性。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "resourceGroupOutput": {
            "value": "[resourceGroup()]",
            "type" : "object"
        }
    }
}
```

上述範例會以下列格式傳回物件︰

```json
{
  "id": "/subscriptions/{subscription-id}/resourceGroups/examplegroup",
  "name": "examplegroup",
  "location": "southcentralus",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/resourcegroup.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/resourcegroup.json 
```

<a id="resourceid" />

## <a name="resourceid"></a>ResourceId
`resourceId([subscriptionId], [resourceGroupName], resourceType, resourceName1, [resourceName2]...)`

傳回資源的唯一識別碼。 如果資源名稱不確定或未佈建在相同的範本內，請使用此函數。 

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| subscriptionId |否 |字串 (GUID 格式) |預設值為目前的訂用帳戶。 需要擷取另一個訂用帳戶中的資源群組時，請指定此值。 |
| resourceGroupName |否 |字串 |預設值為目前資源群組。 需要擷取另一個訂用帳戶中的資源群組時，請指定此值。 |
| resourceType |是 |字串 |資源的類型 (包括資源提供者命名空間)。 |
| resourceName1 |是 |字串 |資源的名稱。 |
| resourceName2 |否 |字串 |如果是巢狀資源，則為下一個資源名稱區段。 |

### <a name="return-value"></a>傳回值

識別碼會以下列格式傳回：

```json
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
```

### <a name="remarks"></a>備註

您指定的參數值取決於資源是否在相同的訂用帳戶和資源群組中作為目前的部署。

若要在相同的訂用帳戶和資源群組中取得儲存體帳戶的資源識別碼，請使用：

```json
"[resourceId('Microsoft.Storage/storageAccounts','examplestorage')]"
```

若要在相同的訂用帳戶但不同的資源群組中取得儲存體帳戶的資源識別碼，請使用：

```json
"[resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
```

若要在不同的訂用帳戶和資源群組中取得儲存體帳戶的資源識別碼，請使用：

```json
"[resourceId('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]"
```

若要在不同的資源群組中取得資料庫的資源識別碼，請使用：

```json
"[resourceId('otherResourceGroup', 'Microsoft.SQL/servers/databases', parameters('serverName'), parameters('databaseName'))]"
```

通常，在替代資源群組中使用儲存體帳戶或虛擬網路時，需要使用此函數。 下列範例顯示如何輕鬆地使用外部資源群組中的資源：

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "virtualNetworkName": {
          "type": "string"
      },
      "virtualNetworkResourceGroup": {
          "type": "string"
      },
      "subnet1Name": {
          "type": "string"
      },
      "nicName": {
          "type": "string"
      }
  },
  "variables": {
      "vnetID": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
      "subnet1Ref": "[concat(variables('vnetID'),'/subnets/', parameters('subnet1Name'))]"
  },
  "resources": [
  {
      "apiVersion": "2015-05-01-preview",
      "type": "Microsoft.Network/networkInterfaces",
      "name": "[parameters('nicName')]",
      "location": "[parameters('location')]",
      "properties": {
          "ipConfigurations": [{
              "name": "ipconfig1",
              "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "subnet": {
                      "id": "[variables('subnet1Ref')]"
                  }
              }
          }]
       }
  }]
}
```

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/resourceid.json)會傳回資源群組中儲存體帳戶的資源識別碼：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "sameRGOutput": {
            "value": "[resourceId('Microsoft.Storage/storageAccounts','examplestorage')]",
            "type" : "string"
        },
        "differentRGOutput": {
            "value": "[resourceId('otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]",
            "type" : "string"
        },
        "differentSubOutput": {
            "value": "[resourceId('11111111-1111-1111-1111-111111111111', 'otherResourceGroup', 'Microsoft.Storage/storageAccounts','examplestorage')]",
            "type" : "string"
        },
        "nestedResourceOutput": {
            "value": "[resourceId('Microsoft.SQL/servers/databases', 'serverName', 'databaseName')]",
            "type" : "string"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| 名稱 | 類型 | 值 |
| ---- | ---- | ----- |
| sameRGOutput | 字串 | /subscriptions/{current-sub-id}/resourceGroups/examplegroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentRGOutput | 字串 | /subscriptions/{current-sub-id}/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| differentSubOutput | 字串 | /subscriptions/11111111-1111-1111-1111-111111111111/resourceGroups/otherResourceGroup/providers/Microsoft.Storage/storageAccounts/examplestorage |
| nestedResourceOutput | 字串 | /subscriptions/{current-sub-id}/resourceGroups/examplegroup/providers/Microsoft.SQL/servers/serverName/databases/databaseName |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/resourceid.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/resourceid.json 
```

<a id="subscription" />

## <a name="subscription"></a>訂用帳戶
`subscription()`

傳回目前部署的訂用帳戶詳細資料。 

### <a name="return-value"></a>傳回值

此函式會傳回下列格式︰

```json
{
    "id": "/subscriptions/{subscription-id}",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}",
    "displayName": "{name-of-subscription}"
}
```

### <a name="example"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/subscription.json)顯示在 outputs 區段中所呼叫的 subscription 函式。 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "subscriptionOutput": {
            "value": "[subscription()]",
            "type" : "object"
        }
    }
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/subscription.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/subscription.json 
```

## <a name="next-steps"></a>後續步驟
* 如需有關 Azure Resource Manager 範本中各區段的說明，請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)。
* 若要合併多個範本，請參閱[透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。
* 若要依指定的次數重複建立資源類型，請參閱 [在 Azure 資源管理員中建立資源的多個執行個體](resource-group-create-multiple.md)。
* 若要了解如何部署已建立的範本，請參閱[使用 Azure Resource Manager 範本部署應用程式](resource-group-template-deploy.md)。

