---
title: Azure Resource Manager 範本參數區段 | Microsoft Docs
description: 使用宣告式 JSON 語法描述 Azure Resource Manager 範本的參數區段。
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
ms.date: 05/18/2018
ms.author: tomfitz
ms.openlocfilehash: 6d09a057d9b8a02c7f8313161e64aa3a42eb6db2
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34604330"
---
# <a name="parameters-section-of-azure-resource-manager-templates"></a>Azure Resource Manager 範本中的參數區段
在範本的 parameters 區段中，您會指定可在部署資源時輸入的值。 提供針對特定環境 (例如開發、測試和生產環境) 量身訂做的參數值，可讓您自訂部署。 您不必在範本中提供參數，但若沒有參數，您的範本一律會部署具有相同名稱、位置和屬性的相同資源。

您在範本中限制為 255 個參數。 您可以使用包含多個屬性的物件，以減少參數數目，如本文所示。

## <a name="define-and-use-a-parameter"></a>定義及使用參數

下列範例顯示簡單的參數定義。 其定義參數的名稱，並指定其需要字串值。 參數僅接受對其預期用途有意義的值。 在部署期間未提供任何值時，會指定預設值。 最後，參數包含其用途描述。 

```json
"parameters": {
  "storageSKU": {
    "type": "string",
    "allowedValues": [
      "Standard_LRS",
      "Standard_ZRS",
      "Standard_GRS",
      "Standard_RAGRS",
      "Premium_LRS"
    ],
    "defaultValue": "Standard_LRS",
    "metadata": {
      "description": "The type of replication to use for the storage account."
    }
  }   
}
```

在範本中，您可以參考使用下列語法的參數值：

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "sku": {
      "name": "[parameters('storageSKU')]"
    },
    ...
  }
]
```

## <a name="available-properties"></a>可用屬性

上述範例僅示範一部分您可以在參數區段中使用的屬性。 可用屬性為︰

```json
"parameters": {
    "<parameter-name>" : {
        "type" : "<type-of-parameter-value>",
        "defaultValue": "<default-value-of-parameter>",
        "allowedValues": [ "<array-of-allowed-values>" ],
        "minValue": <minimum-value-for-int>,
        "maxValue": <maximum-value-for-int>,
        "minLength": <minimum-length-for-string-or-array>,
        "maxLength": <maximum-length-for-string-or-array-parameters>,
        "metadata": {
            "description": "<description-of-the parameter>" 
        }
    }
}
```

| 元素名稱 | 必要 | 說明 |
|:--- |:--- |:--- |
| parameterName |yes |參數名稱。 必須是有效的 JavaScript 識別碼。 |
| type |yes |參數值類型。 允許的類型和值為 **string**、**securestring**、**int**、**bool**、**object**、**secureObject**，以及 **array**。 |
| defaultValue |否 |如果未提供參數值，則會使用參數的預設值。 |
| allowedValues |否 |參數的允許值陣列，確保提供正確的值。 |
| minValue |否 |int 類型參數的最小值，含此值。 |
| maxValue |否 |int 類型參數的最大值，含此值。 |
| minLength |否 |字串、securestring 及陣列類型參數長度的最小值，含此值。 |
| maxLength |否 |字串、securestring 及陣列類型參數長度的最大值，含此值。 |
| 說明 |否 |透過入口網站向使用者顯示的參數說明。 |

## <a name="template-functions-with-parameters"></a>使用參數的範本函式

為參數提供預設值時，您可以使用大部分的範本函式。 您可以使用另一個參數以建立預設值。 下列範本示範如何以預設值使用函式：

```json
"parameters": {
  "siteName": {
    "type": "string",
    "defaultValue": "[concat('site', uniqueString(resourceGroup().id))]",
    "metadata": {
      "description": "The site name. To use the default value, do not specify a new value."
    }
  },
  "hostingPlanName": {
    "type": "string",
    "defaultValue": "[concat(parameters('siteName'),'-plan')]",
    "metadata": {
      "description": "The host name. To use the default value, do not specify a new value."
    }
  }
}
```

您無法使用參數區段中的 `reference` 函式。 會在部署之前評估參數，因此，`reference` 函式無法取得資源的執行階段狀態。 

## <a name="objects-as-parameters"></a>作為參數的物件

藉由將相關值以物件的方式傳遞，可以更輕鬆地進行組織。 此方法也會減少範本中的參數數目。

部署期間，請在範本中定義參數並指定 JSON 物件，而非單一值。 

```json
"parameters": {
  "VNetSettings": {
    "type": "object",
    "defaultValue": {
      "name": "VNet1",
      "location": "eastus",
      "addressPrefixes": [
        {
          "name": "firstPrefix",
          "addressPrefix": "10.0.0.0/22"
        }
      ],
      "subnets": [
        {
          "name": "firstSubnet",
          "addressPrefix": "10.0.0.0/24"
        },
        {
          "name": "secondSubnet",
          "addressPrefix": "10.0.1.0/24"
        }
      ]
    }
  }
},
```

然後，使用點運算子來參考參數的子屬性。

```json
"resources": [
  {
    "apiVersion": "2015-06-15",
    "type": "Microsoft.Network/virtualNetworks",
    "name": "[parameters('VNetSettings').name]",
    "location": "[parameters('VNetSettings').location]",
    "properties": {
      "addressSpace":{
        "addressPrefixes": [
          "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
        ]
      },
      "subnets":[
        {
          "name":"[parameters('VNetSettings').subnets[0].name]",
          "properties": {
            "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
          }
        },
        {
          "name":"[parameters('VNetSettings').subnets[1].name]",
          "properties": {
            "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
          }
        }
      ]
    }
  }
]
```

## <a name="recommendations"></a>建議
當您使用參數時，下列資訊可能會很有幫助︰

* 將參數的使用最小化。 可能的話，使用變數或常值。 只針對這些案例使用參數︰
   
   * 您想要根據環境 (SKU、大小、容量) 使用變化的設定。
   * 您希望能方便識別而指定的資源名稱。
   * 您經常用來完成其他工作 (例如，管理使用者名稱) 的值。
   * 機密資料 (例如密碼)。
   * 建立多個資源類型執行個體時要使用的數字或值陣列。
* 參數名稱使用駝峰式大小寫。
* 在中繼資料中提供每個參數的描述：

   ```json
   "parameters": {
       "storageAccountType": {
           "type": "string",
           "metadata": {
               "description": "The type of the new storage account created to store the VM disks."
           }
       }
   }
   ```

* 定義參數的預設值 (密碼和 SSH 金鑰除外)。 藉由提供預設值，該參數在部署期間變成選用。 預設值可以是空字串。 
   
   ```json
   "parameters": {
        "storageAccountType": {
            "type": "string",
            "defaultValue": "Standard_GRS",
            "metadata": {
                "description": "The type of the new storage account created to store the VM disks."
            }
        }
   }
   ```

* 所有密碼都要使用 **securestring** 。 如果您在 JSON 物件中傳遞敏感資料，請使用 **secureObject** 類型。 部署資源後，無法讀取類型為 securestring 或 secureObject 的範本參數。 
   
   ```json
   "parameters": {
       "secretValue": {
           "type": "securestring",
           "metadata": {
               "description": "The value of the secret to store in the vault."
           }
       }
   }
   ```

* 請使用參數以指定位置，並盡可能和可能位於相同位置的資源一起共用該參數值。 此方法可以降低系統要求使用者提供位置資訊的次數。 如果只在有限的位置支援某資源類型，您可能想要直接在範本中指定有效的位置，或新增其他位置參數。 當組織限制其使用者的允許區域時，**resourceGroup().location** 運算式可能會導致使用者無法部署範本。 例如，某位使用者在區域中建立資源群組。 第二個使用者必須部署至該資源群組，但沒有該區域的存取權。 
   
   ```json
   "resources": [
     {
         "name": "[variables('storageAccountName')]",
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2016-01-01",
         "location": "[parameters('location')]",
         ...
     }
   ]
   ```
    
* 資源類型的 API 版本要避免使用參數或變數。 資源屬性和值可能會隨版本號碼而不同。 將 API 版本設定為參數或變數時，程式碼編輯器中的 Intellisense 會無法判斷正確的結構描述。 請改為將 API 版本硬式編碼在範本中。
* 請避免在範本中指定與部署命令中的參數相符合的參數名稱。 資源管理員會在範本參數加上後置詞 **FromTemplate** 以避免命名衝突。 例如，如果您在範本中包含名為 **ResourceGroupName** 的參數，這會與 [New-AzureRmResourceGroupDeployment](/powershell/module/azurerm.resources/new-azurermresourcegroupdeployment) Cmdlet 中的 **ResourceGroupName** 參數發生衝突。 部署期間，系統會提示您為 **ResourceGroupNameFromTemplate** 提供值。

## <a name="example-templates"></a>範本的範例

這些範例範本示範使用參數的一些情況。 部署這些參數以測試如何在不同的情況下處理參數。

|範本  |說明  |
|---------|---------|
|[具有預設值之帶有函式的參數](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/parameterswithfunctions.json) | 示範定義參數的預設值時，如何使用範本函式。 範本不會部署任何資源。 它會建構參數值，並傳回這些值。 |
|[參數物件](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/parameterobject.json) | 示範如何針對參數使用物件。 範本不會部署任何資源。 它會建構參數值，並傳回這些值。 |

## <a name="next-steps"></a>後續步驟

* 若要檢視許多不同類型解決方案的完整範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)。
* 如需如何在部署期間輸入參數值的資訊，請參閱 [使用 Azure Resource Manager 範本部署資源](resource-group-template-deploy.md)。 
* 如需您可以在範本內使用哪些函式的詳細資料，請參閱 [Azure Resource Manager 範本函式](resource-group-template-functions.md)。
* 如需使用參數物件的資訊，請參閱[在 Azure Resource Manager 範本中使用物件作為參數](/azure/architecture/building-blocks/extending-templates/objects-as-parameters)。
