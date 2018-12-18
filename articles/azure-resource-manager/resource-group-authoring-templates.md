---
title: Azure Resource Manager 範本結構和語法 | Microsoft Docs
description: 使用宣告式 JSON 語法描述 Azure Resource Manager 範本的結構和屬性。
services: azure-resource-manager
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: 19694cb4-d9ed-499a-a2cc-bcfc4922d7f5
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/30/2018
ms.author: tomfitz
ms.openlocfilehash: 129d02bea6fe3668a308da0ab2a46ca8b59928e7
ms.sourcegitcommit: f983187566d165bc8540fdec5650edcc51a6350a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/13/2018
ms.locfileid: "45542239"
---
# <a name="understand-the-structure-and-syntax-of-azure-resource-manager-templates"></a>了解 Azure Resource Manager 範本的結構和語法
本文說明 Azure Resource Manager 範本的結構。 它會呈現範本的不同區段，以及這些區段中可用的屬性。 範本由 JSON 與運算式所組成，可讓您用來為部署建構值。 如需建立範本的逐步教學課程，請參閱[建立第一個 Azure Resource Manager 範本](resource-manager-create-first-template.md)。

## <a name="template-format"></a>範本格式
在最簡單的結構中，範本具有下列元素：

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "",
    "parameters": {  },
    "variables": {  },
    "functions": [  ],
    "resources": [  ],
    "outputs": {  }
}
```

| 元素名稱 | 必要 | 說明 |
|:--- |:--- |:--- |
| $schema |是 |JSON 結構描述檔案的位置，說明範本語言的版本。 使用上述範例所示的 URL。 |
| contentVersion |是 |範本版本 (例如 1.0.0.0)。 您可以為此元素提供任何值。 使用此值在範本中記載重大變更。 使用範本部署資源時，這個值可用來確定使用的是正確的範本。 |
| parameters |否 |執行部署以自訂資源部署時所提供的值。 |
| variables |否 |範本中做為 JSON 片段以簡化範本語言運算式的值。 |
| functions |否 |範本中可用的使用者定義函式。 |
| resources |是 |在資源群組中部署或更新的資源類型。 |
| outputs |否 |部署後傳回的值。 |

每個元素都有可以設定的屬性。 下列範例顯示範本的完整語法：

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "",
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
    },
    "variables": {
        "<variable-name>": "<variable-value>",
        "<variable-object-name>": {
            <variable-complex-type-value>
        },
        "<variable-object-name>": {
            "copy": [
                {
                    "name": "<name-of-array-property>",
                    "count": <number-of-iterations>,
                    "input": {
                        <properties-to-repeat>
                    }
                }
            ]
        },
        "copy": [
            {
                "name": "<variable-array-name>",
                "count": <number-of-iterations>,
                "input": {
                    <properties-to-repeat>
                }
            }
        ]
    },
    "functions": [
      {
        "namespace": "<namespace-for-your-function>",
        "members": {
          "<function-name>": {
            "parameters": [
              {
                "name": "<parameter-name>",
                "type": "<type-of-parameter-value>"
              }
            ],
            "output": {
              "type": "<type-of-output-value>",
              "value": "<function-expression>"
            }
          }
        }
      }
    ],
    "resources": [
      {
          "condition": "<boolean-value-whether-to-deploy>",
          "apiVersion": "<api-version-of-resource>",
          "type": "<resource-provider-namespace/resource-type-name>",
          "name": "<name-of-the-resource>",
          "location": "<location-of-resource>",
          "tags": {
              "<tag-name1>": "<tag-value1>",
              "<tag-name2>": "<tag-value2>"
          },
          "comments": "<your-reference-notes>",
          "copy": {
              "name": "<name-of-copy-loop>",
              "count": "<number-of-iterations>",
              "mode": "<serial-or-parallel>",
              "batchSize": "<number-to-deploy-serially>"
          },
          "dependsOn": [
              "<array-of-related-resource-names>"
          ],
          "properties": {
              "<settings-for-the-resource>",
              "copy": [
                  {
                      "name": ,
                      "count": ,
                      "input": {}
                  }
              ]
          },
          "resources": [
              "<array-of-child-resources>"
          ]
      }
    ],
    "outputs": {
        "<outputName>" : {
            "type" : "<type-of-output-value>",
            "value": "<output-value-expression>"
        }
    }
}
```

本文將詳細說明範本的各個區段。

## <a name="syntax"></a>語法
範本的基本語法是 JSON。 不過，運算式和函式可延伸範本中提供的 JSON 值。  運算式是在 JSON 字串常值內撰寫，其第一個字元和最後一個字元分別為下列兩種方括號：`[` 和 `]`。 部署範本時，會評估運算式的值。 雖然是以字串常值撰寫，評估運算式的結果，根據實際的運算式可能會是不同的 JSON 類型 (例如陣列或整數)。  針對開頭為方括號 `[` 的常值字串，若不要將它解譯為運算式，請加入額外的方括號，使字串開頭成為 `[[`。

一般而言，您可以將運算式搭配函數使用，以執行可設定部署的作業。 和在 JavaScript 中相同，函式呼叫的格式為 `functionName(arg1,arg2,arg3)`。 您可以使用點與 [index] 運算子來參考屬性。

下列範例示範如何在建構值時使用數個函數：

```json
"variables": {
    "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
}
```

如需範本函數的完整清單，請參閱 [Azure 資源管理員範本函數](resource-group-template-functions.md)。 

## <a name="parameters"></a>參數
在範本的 parameters 區段中，您會指定可在部署資源時輸入的值。 提供針對特定環境 (例如開發、測試和生產環境) 量身訂做的參數值，可讓您自訂部署。 您不必在範本中提供參數，但若沒有參數，您的範本一律會部署具有相同名稱、位置和屬性的相同資源。

下列範例顯示簡單的參數定義：

```json
"parameters": {
  "siteNamePrefix": {
    "type": "string",
    "metadata": {
      "description": "The name prefix of the web app that you wish to create."
    }
  },
},
```

如需定義參數的詳細資訊，請參閱 [Azure Resource Manager 範本的 Parameters 區段](resource-manager-templates-parameters.md)。

## <a name="variables"></a>變數
在 variables 區段中，您會建構可用於整個範本中的值。 您不需要定義變數，但它們通常會經由減少複雜運算式來簡化您的範本。

下列範例顯示簡單的變數定義：

```json
"variables": {
  "webSiteName": "[concat(parameters('siteNamePrefix'), uniqueString(resourceGroup().id))]",
},
```

如需定義變數的詳細資訊，請參閱 [Azure Resource Manager 範本的 Variables 區段](resource-manager-templates-variables.md)。

## <a name="functions"></a>Functions

在您的範本內，您可以建立自己的函式。 這些函式可供您在範本中使用。 您通常會定義不想在整個範本中重複使用的複雜運算式。 您會從範本中支援的運算式和[函式](resource-group-template-functions.md)建立使用者定義的函式。

在定義使用者函式時，有一些限制：

* 此函式無法存取變數。
* 此函式無法存取範本參數。 也就是，會將[參數函式](resource-group-template-functions-deployment.md#parameters)限制為函式參數。
* 此函式無法呼叫其他的使用者定義函式。
* 此函式不能使用[參考函式](resource-group-template-functions-resource.md#reference)。
* 函式的參數不能有預設值。

您的函式需要命名空間值，以避免範本函式發生命名衝突。 下列範例說明可傳回儲存體帳戶名稱的函式：

```json
"functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
],
```

您呼叫函式的方式：

```json
"resources": [
  {
    "name": "[contoso.uniqueName(parameters('storageNamePrefix'))]",
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2016-01-01",
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "Storage",
    "location": "South Central US",
    "tags": {},
    "properties": {}
  }
]
```

## <a name="resources"></a>資源
在資源區段中，您會定義要部署或更新的資源。 此區段會變得複雜，因為您必須了解您要部署的類型才能提供正確的值。

```json
"resources": [
  {
    "apiVersion": "2016-08-01",
    "name": "[variables('webSiteName')]",
    "type": "Microsoft.Web/sites",
    "location": "[resourceGroup().location]",
    "properties": {
      "serverFarmId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.Web/serverFarms/<plan-name>"
    }
  }
],
```

如需詳細資訊，請參閱 [Azure Resource Manager 範本的 Resources 區段](resource-manager-templates-resources.md)。

## <a name="outputs"></a>輸出
在輸出區段中，您可以指定從部署傳回的值。 例如，您可以傳回 URI 以存取所部署的資源。

```json
"outputs": {
  "newHostName": {
    "type": "string",
    "value": "[reference(variables('webSiteName')).defaultHostName]"
  }
}
```

如需詳細資訊，請參閱 [Azure Resource Manager 範本的 Outputs 區段](resource-manager-templates-outputs.md)。

## <a name="template-limits"></a>範本限制

將範本大小限制為 1 MB，並將每個參數檔案限制為 64 KB。 1 MB 的限制適用於已增加反覆資源定義和變數和參數值之範本的最終狀態。 

您也受限於：

* 256 個參數
* 256 個變數
* 800 個資源 (包括複本計數)
* 64 個輸出值
* 範本運算式中的 24,576 個字元

使用巢狀範本，即可超出一些範本限制。 如需詳細資訊，請參閱[在部署 Azure 資源時使用連結的範本](resource-group-linked-templates.md)。 若要減少參數、變數或輸出數目，您可以將數個值合併成一個物件。 如需詳細資訊，請參閱[物件作為參數](resource-manager-objects-as-parameters.md)。

## <a name="next-steps"></a>後續步驟
* 若要檢視許多不同類型解決方案的完整範本，請參閱 [Azure 快速入門範本](https://azure.microsoft.com/documentation/templates/)。
* 如需您可以在範本內使用哪些函式的詳細資料，請參閱 [Azure Resource Manager 範本函式](resource-group-template-functions.md)。
* 若要在部署期間合併多個範本，請參閱 [透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。
* 如需對於建立可跨全域 Azure、Azure 主權雲端與 Azure Stack 使用的 Resource Manager 範本相關建議，請參閱[開發針對雲端一致性的 Azure Resource Manager 範本](templates-cloud-consistency.md)。
