---
title: Azure Resource Manager 範本函式 - 字串 | Microsoft Docs
description: 描述 Azure Resource Manager 範本中用來使用字串的函式。
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
ms.date: 09/05/2017
ms.author: tomfitz
ms.openlocfilehash: 33a49a9fb66240382b0bb4e0bedbb07b8d78a763
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34360354"
---
# <a name="string-functions-for-azure-resource-manager-templates"></a>Azure Resource Manager 範本的字串函式

資源管理員提供下列函式以使用字串：

* [base64](#base64)
* [base64ToJson](#base64tojson)
* [base64ToString](#base64tostring)
* [concat](#concat)
* [contains](#contains)
* [dataUri](#datauri)
* [dataUriToString](#datauritostring)
* [empty](#empty)
* [endsWith](#endswith)
* [first](#first)
* [guid](#guid)
* [indexOf](#indexof)
* [last](#last)
* [lastIndexOf](#lastindexof)
* [length](#length)
* [padLeft](#padleft)
* [replace](#replace)
* [skip](#skip)
* [分割](#split)
* [startsWith](resource-group-template-functions-string.md#startswith)
* [字串](#string)
* [substring](#substring)
* [take](#take)
* [toLower](#tolower)
* [toUpper](#toupper)
* [修剪](#trim)
* [uniqueString](#uniquestring)
* [uri](#uri)
* [uriComponent](resource-group-template-functions-string.md#uricomponent)
* [uriComponentToString](resource-group-template-functions-string.md#uricomponenttostring)

<a id="base64" />

## <a name="base64"></a>base64
`base64(inputString)`

傳回輸入字串的 base64 表示法。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| inputString |yes |字串 |要以 base64 表示法傳回的值。 |

### <a name="return-value"></a>傳回值

字串，包含 base64 表示法。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/base64.json)顯示如何使用 base64 函式。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringData": {
            "type": "string",
            "defaultValue": "one, two, three"
        },
        "jsonFormattedData": {
            "type": "string",
            "defaultValue": "{'one': 'a', 'two': 'b'}"
        }
    },
    "variables": {
        "base64String": "[base64(parameters('stringData'))]",
        "base64Object": "[base64(parameters('jsonFormattedData'))]"
    },
    "resources": [
    ],
    "outputs": {
        "base64Output": {
            "type": "string",
            "value": "[variables('base64String')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[base64ToString(variables('base64String'))]"
        },
        "toJsonOutput": {
            "type": "object",
            "value": "[base64ToJson(variables('base64Object'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| base64Output | 字串 | b25lLCB0d28sIHRocmVl |
| toStringOutput | 字串 | one, two, three |
| toJsonOutput | Object | {"one": "a", "two": "b"} |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

<a id="base64tojson" />

## <a name="base64tojson"></a>base64ToJson
`base64tojson`

將 base64 表示法轉換為 JSON 物件。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| base64Value |yes |字串 |要轉換為 JSON 物件的 base64 表示法。 |

### <a name="return-value"></a>傳回值

JSON 物件。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/base64.json)會使用 base64ToJson 函式來轉換 base64 值︰

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringData": {
            "type": "string",
            "defaultValue": "one, two, three"
        },
        "jsonFormattedData": {
            "type": "string",
            "defaultValue": "{'one': 'a', 'two': 'b'}"
        }
    },
    "variables": {
        "base64String": "[base64(parameters('stringData'))]",
        "base64Object": "[base64(parameters('jsonFormattedData'))]"
    },
    "resources": [
    ],
    "outputs": {
        "base64Output": {
            "type": "string",
            "value": "[variables('base64String')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[base64ToString(variables('base64String'))]"
        },
        "toJsonOutput": {
            "type": "object",
            "value": "[base64ToJson(variables('base64Object'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| base64Output | 字串 | b25lLCB0d28sIHRocmVl |
| toStringOutput | 字串 | one, two, three |
| toJsonOutput | Object | {"one": "a", "two": "b"} |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

<a id="base64tostring" />

## <a name="base64tostring"></a>base64ToString
`base64ToString(base64Value)`

將 base64 表示法轉換為字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| base64Value |yes |字串 |要轉換為字串的 base64 表示法。 |

### <a name="return-value"></a>傳回值

轉換後之 base64 值的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/base64.json)會使用 base64ToString 函式來轉換 base64 值︰

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringData": {
            "type": "string",
            "defaultValue": "one, two, three"
        },
        "jsonFormattedData": {
            "type": "string",
            "defaultValue": "{'one': 'a', 'two': 'b'}"
        }
    },
    "variables": {
        "base64String": "[base64(parameters('stringData'))]",
        "base64Object": "[base64(parameters('jsonFormattedData'))]"
    },
    "resources": [
    ],
    "outputs": {
        "base64Output": {
            "type": "string",
            "value": "[variables('base64String')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[base64ToString(variables('base64String'))]"
        },
        "toJsonOutput": {
            "type": "object",
            "value": "[base64ToJson(variables('base64Object'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| base64Output | 字串 | b25lLCB0d28sIHRocmVl |
| toStringOutput | 字串 | one, two, three |
| toJsonOutput | Object | {"one": "a", "two": "b"} |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/base64.json
```

<a id="concat" />

## <a name="concat"></a>concat
`concat (arg1, arg2, arg3, ...)`

結合多個字串值並傳回串連字串，或結合多個陣列並傳回串連陣列。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| arg1 |yes |字串或陣列 |串連的第一個值。 |
| 其他引數 |否 |字串 |串連的其他值 (循序順序)。 |

### <a name="return-value"></a>傳回值
串連值的字串或陣列。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/concat-string.json)顯示如何結合兩個字串值並傳回串連字串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "prefix": {
            "type": "string",
            "defaultValue": "prefix"
        }
    },
    "resources": [],
    "outputs": {
        "concatOutput": {
            "value": "[concat(parameters('prefix'), '-', uniqueString(resourceGroup().id))]",
            "type" : "string"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| concatOutput | 字串 | prefix-5yj4yjf5mbg72 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-string.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-string.json
```

下一個[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/concat-array.json)顯示如何結合兩個陣列。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { 
        "firstArray": { 
            "type": "array", 
            "defaultValue": [ 
                "1-1", 
                "1-2", 
                "1-3" 
            ] 
        },
        "secondArray": {
            "type": "array", 
            "defaultValue": [ 
                "2-1", 
                "2-2",
                "2-3" 
            ] 
        }
    },
    "resources": [
    ],
    "outputs": {
        "return": {
            "type": "array",
            "value": "[concat(parameters('firstArray'), parameters('secondArray'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| return | 陣列 | ["1-1", "1-2", "1-3", "2-1", "2-2", "2-3"] |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-array.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/concat-array.json
```

<a id="contains" />

## <a name="contains"></a>contains
`contains (container, itemToFind)`

檢查陣列中是否包含值、物件中是否包含索引鍵，或字串中是否包含子字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| container |yes |陣列、物件或字串 |其中包含要尋找之值的值。 |
| itemToFind |yes |字串或整數 |要尋找的值。 |

### <a name="return-value"></a>傳回值

找到項目則傳回 **True**，否則會傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/contains.json)顯示如何使用不同類型的 contains：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringToTest": {
            "type": "string",
            "defaultValue": "OneTwoThree"
        },
        "objectToTest": {
            "type": "object",
            "defaultValue": {"one": "a", "two": "b", "three": "c"}
        },
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "stringTrue": {
            "type": "bool",
            "value": "[contains(parameters('stringToTest'), 'e')]"
        },
        "stringFalse": {
            "type": "bool",
            "value": "[contains(parameters('stringToTest'), 'z')]"
        },
        "objectTrue": {
            "type": "bool",
            "value": "[contains(parameters('objectToTest'), 'one')]"
        },
        "objectFalse": {
            "type": "bool",
            "value": "[contains(parameters('objectToTest'), 'a')]"
        },
        "arrayTrue": {
            "type": "bool",
            "value": "[contains(parameters('arrayToTest'), 'three')]"
        },
        "arrayFalse": {
            "type": "bool",
            "value": "[contains(parameters('arrayToTest'), 'four')]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| stringTrue | Bool | True |
| stringFalse | Bool | False |
| objectTrue | Bool | True |
| objectFalse | Bool | False |
| arrayTrue | Bool | True |
| arrayFalse | Bool | False |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/contains.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/contains.json
```

<a id="datauri" />

## <a name="datauri"></a>dataUri
`dataUri(stringToConvert)`

將值轉換為資料 URI。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToConvert |yes |字串 |要轉換為資料 URI 的值。 |

### <a name="return-value"></a>傳回值

格式化為資料 URI 的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/datauri.json)會將值轉換為資料 URI，並將資料 URI 轉換為字串︰

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringToTest": {
            "type": "string",
            "defaultValue": "Hello"
        },
        "dataFormattedString": {
            "type": "string",
            "defaultValue": "data:;base64,SGVsbG8sIFdvcmxkIQ=="
        }
    },
    "resources": [],
    "outputs": {
        "dataUriOutput": {
            "value": "[dataUri(parameters('stringToTest'))]",
            "type" : "string"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[dataUriToString(parameters('dataFormattedString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| dataUriOutput | 字串 | data:text/plain;charset=utf8;base64,SGVsbG8= |
| toStringOutput | 字串 | Hello, World! |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/datauri.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/datauri.json
```

<a id="datauritostring" />

## <a name="datauritostring"></a>dataUriToString
`dataUriToString(dataUriToConvert)`

將資料 URI 格式化值轉換為字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| dataUriToConvert |yes |字串 |要轉換的資料 URI 值。 |

### <a name="return-value"></a>傳回值

字串，包含已轉換的值。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/datauri.json)會將值轉換為資料 URI，並將資料 URI 轉換為字串︰

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "stringToTest": {
            "type": "string",
            "defaultValue": "Hello"
        },
        "dataFormattedString": {
            "type": "string",
            "defaultValue": "data:;base64,SGVsbG8sIFdvcmxkIQ=="
        }
    },
    "resources": [],
    "outputs": {
        "dataUriOutput": {
            "value": "[dataUri(parameters('stringToTest'))]",
            "type" : "string"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[dataUriToString(parameters('dataFormattedString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| dataUriOutput | 字串 | data:text/plain;charset=utf8;base64,SGVsbG8= |
| toStringOutput | 字串 | Hello, World! |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/datauri.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/datauri.json
```

<a id="empty" /> 

## <a name="empty"></a>empty
`empty(itemToTest)`

判斷陣列、物件或字串是否空白。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| itemToTest |yes |陣列、物件或字串 |要檢查其是否為空白的值。 |

### <a name="return-value"></a>傳回值

如果值空白則傳回 **True**，否則會傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/empty.json)會檢查陣列、物件和字串是否空白。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": []
        },
        "testObject": {
            "type": "object",
            "defaultValue": {}
        },
        "testString": {
            "type": "string",
            "defaultValue": ""
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testArray'))]"
        },
        "objectEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testObject'))]"
        },
        "stringEmpty": {
            "type": "bool",
            "value": "[empty(parameters('testString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayEmpty | Bool | True |
| objectEmpty | Bool | True |
| stringEmpty | Bool | True |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/empty.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/empty.json
```

<a id="endswith" />

## <a name="endswith"></a>endsWith
`endsWith(stringToSearch, stringToFind)`

判斷字串結尾是否為值。 此比較不區分大小寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToSearch |yes |字串 |其中包含要尋找之項目的值。 |
| stringToFind |yes |字串 |要尋找的值。 |

### <a name="return-value"></a>傳回值

如果最後一個字元或字串字元與該值相符，則傳回 **True**；否則會傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/startsendswith.json)顯示如何使用 startsWith 和 endsWith 函式：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "startsTrue": {
            "value": "[startsWith('abcdef', 'ab')]",
            "type" : "bool"
        },
        "startsCapTrue": {
            "value": "[startsWith('abcdef', 'A')]",
            "type" : "bool"
        },
        "startsFalse": {
            "value": "[startsWith('abcdef', 'e')]",
            "type" : "bool"
        },
        "endsTrue": {
            "value": "[endsWith('abcdef', 'ef')]",
            "type" : "bool"
        },
        "endsCapTrue": {
            "value": "[endsWith('abcdef', 'F')]",
            "type" : "bool"
        },
        "endsFalse": {
            "value": "[endsWith('abcdef', 'e')]",
            "type" : "bool"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| startsTrue | Bool | True |
| startsCapTrue | Bool | True |
| startsFalse | Bool | False |
| endsTrue | Bool | True |
| endsCapTrue | Bool | True |
| endsFalse | Bool | False |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/startsendswith.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/startsendswith.json
```

<a id="first" />

## <a name="first"></a>first
`first(arg1)`

傳回字串的第一個字元或陣列的第一個元素。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| arg1 |yes |陣列或字串 |要擷取其第一個元素或字元的值。 |

### <a name="return-value"></a>傳回值

陣列中第一個字元的字串或第一個元素的類型 (字串、整數、陣列或物件)。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/first.json)顯示如何搭配使用 first 函式與陣列和字串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayOutput": {
            "type": "string",
            "value": "[first(parameters('arrayToTest'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[first('One Two Three')]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayOutput | 字串 | one |
| stringOutput | 字串 | O |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/first.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/first.json
```

## <a name="guid"></a>GUID

`guid (baseString, ...)`

建立一個值，格式為根據提供作為參數之值的全域唯一識別碼。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| baseString |yes |字串 |雜湊函式中用來建立 GUID 的值。 |
| 視需要，也會使用其他參數 |否 |字串 |您可以視需要新增多個字串，來建立指定唯一性層級的值。 |

### <a name="remarks"></a>備註

當您需要建立格式為全域唯一識別碼的值時，此函式很有幫助。 您提供限制結果唯一性範圍的參數值。 您可以指定名稱對於訂用帳戶、資源群組或部署是否唯一。

傳回的值不是隨機字串，而是雜湊函式的結果。 傳回的值為 36 個字元長。 它不是全域唯一的。

下列範例顯示如何使用 guid 來建立常用層級的唯一值。

在訂用帳戶範圍內是唯一

```json
"[guid(subscription().subscriptionId)]"
```

在資源群組範圍內是唯一

```json
"[guid(resourceGroup().id)]"
```

在資源群組的部署範圍內是唯一

```json
"[guid(resourceGroup().id, deployment().name)]"
```

### <a name="return-value"></a>傳回值

字串，包含 36 個字元，格式為全域唯一識別碼。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/guid.json)會從 guid 傳回結果：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {
        "guidPerSubscription": {
            "value": "[guid(subscription().subscriptionId)]",
            "type": "string"
        },
        "guidPerResourceGroup": {
            "value": "[guid(resourceGroup().id)]",
            "type": "string"
        },
        "guidPerDeployment": {
            "value": "[guid(resourceGroup().id, deployment().name)]",
            "type": "string"
        }
    }
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/guid.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/guid.json
```

<a id="indexof" />

## <a name="indexof"></a>indexOf
`indexOf(stringToSearch, stringToFind)`

傳回值在字串內的第一個位置。 此比較不區分大小寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToSearch |yes |字串 |其中包含要尋找之項目的值。 |
| stringToFind |yes |字串 |要尋找的值。 |

### <a name="return-value"></a>傳回值

整數，代表要尋找之項目的位置。 此值是以零為起始。 如果找不到項目，則傳回 -1。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/indexof.json)顯示如何使用 indexOf 和 lastIndexOf 函式：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "firstT": {
            "value": "[indexOf('test', 't')]",
            "type" : "int"
        },
        "lastT": {
            "value": "[lastIndexOf('test', 't')]",
            "type" : "int"
        },
        "firstString": {
            "value": "[indexOf('abcdef', 'CD')]",
            "type" : "int"
        },
        "lastString": {
            "value": "[lastIndexOf('abcdef', 'AB')]",
            "type" : "int"
        },
        "notFound": {
            "value": "[indexOf('abcdef', 'z')]",
            "type" : "int"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| firstT | int | 0 |
| lastT | int | 3 |
| firstString | int | 2 |
| lastString | int | 0 |
| notFound | int | -1 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/indexof.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/indexof.json
```

<a id="last" />

## <a name="last"></a>last
`last (arg1)`

傳回字串的最後一個字元或陣列的最後一個元素。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| arg1 |yes |陣列或字串 |要擷取其最後一個元素或字元的值。 |

### <a name="return-value"></a>傳回值

陣列中最後一個字元的字串，或最後一個元素的類型 (字串、整數、陣列或物件)。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/last.json)顯示如何搭配使用 last 函式與陣列和字串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": ["one", "two", "three"]
        }
    },
    "resources": [
    ],
    "outputs": {
        "arrayOutput": {
            "type": "string",
            "value": "[last(parameters('arrayToTest'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[last('One Two Three')]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayOutput | 字串 | three |
| stringOutput | 字串 | e |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/last.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/last.json
```

<a id="lastindexof" />

## <a name="lastindexof"></a>lastIndexOf
`lastIndexOf(stringToSearch, stringToFind)`

傳回值在字串內的最後一個位置。 此比較不區分大小寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToSearch |yes |字串 |其中包含要尋找之項目的值。 |
| stringToFind |yes |字串 |要尋找的值。 |

### <a name="return-value"></a>傳回值

整數，代表要尋找之項目的最後一個位置。 此值是以零為起始。 如果找不到項目，則傳回 -1。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/indexof.json)顯示如何使用 indexOf 和 lastIndexOf 函式：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "firstT": {
            "value": "[indexOf('test', 't')]",
            "type" : "int"
        },
        "lastT": {
            "value": "[lastIndexOf('test', 't')]",
            "type" : "int"
        },
        "firstString": {
            "value": "[indexOf('abcdef', 'CD')]",
            "type" : "int"
        },
        "lastString": {
            "value": "[lastIndexOf('abcdef', 'AB')]",
            "type" : "int"
        },
        "notFound": {
            "value": "[indexOf('abcdef', 'z')]",
            "type" : "int"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| firstT | int | 0 |
| lastT | int | 3 |
| firstString | int | 2 |
| lastString | int | 0 |
| notFound | int | -1 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/indexof.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/indexof.json
```

<a id="length" />

## <a name="length"></a>length
`length(string)`

傳回字串中的字元數目或陣列中的元素數目。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| arg1 |yes |陣列或字串 |要用來取得元素數目的陣列，或用來取得字元數目的字串。 |

### <a name="return-value"></a>傳回值

整數。 

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/length.json)顯示如何搭配使用 length 與陣列和字串：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "arrayToTest": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "stringToTest": {
            "type": "string",
            "defaultValue": "One Two Three"
        }
    },
    "resources": [],
    "outputs": {
        "arrayLength": {
            "type": "int",
            "value": "[length(parameters('arrayToTest'))]"
        },
        "stringLength": {
            "type": "int",
            "value": "[length(parameters('stringToTest'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayLength | int | 3 |
| stringLength | int | 13 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/length.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/length.json
```

<a id="padleft" />

## <a name="padleft"></a>padLeft
`padLeft(valueToPad, totalLength, paddingCharacter)`

藉由將字元新增至左邊，直到到達指定的總長度，以傳回靠右對齊的字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| valueToPad |yes |字串或整數 |要靠右對齊的值。 |
| totalLength |yes |int |傳回字串中的字元總數。 |
| paddingCharacter |否 |單一字元 |要用於左側填補直到達到總長度的字元。 預設值是空格。 |

如果原始字串長度超過要填補的字元數，則不會新增任何字元。

### <a name="return-value"></a>傳回值

至少含有指定字元數的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/padleft.json)顯示如何藉由新增「零」字元直到符合字元總數，以填補使用者提供的參數值。 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "123"
        }
    },
    "resources": [],
    "outputs": {
        "stringOutput": {
            "type": "string",
            "value": "[padLeft(parameters('testString'),10,'0')]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| stringOutput | 字串 | 0000000123 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/padleft.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/padleft.json
```

<a id="replace" />

## <a name="replace"></a>取代
`replace(originalString, oldString, newString)`

傳回具備由另一個字串取代的一個字串之所有執行個體的新字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| originalString |yes |字串 |具備由另一個字串取代的一個字串之所有執行個體的值。 |
| oldString |yes |字串 |要從原始字串中移除的字串。 |
| newString |yes |字串 |要新增來取代移除之字串的字串。 |

### <a name="return-value"></a>傳回值

具有已取代字元的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/replace.json)會示範如何從使用者提供的字串中將所有的連字號移除，以及如何使用另一個字串來取代一部分的字串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "123-123-1234"
        }
    },
    "resources": [],
    "outputs": {
        "firstOutput": {
            "type": "string",
            "value": "[replace(parameters('testString'),'-', '')]"
        },
        "secodeOutput": {
            "type": "string",
            "value": "[replace(parameters('testString'),'1234', 'xxxx')]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| firstOutput | 字串 | 1231231234 |
| secodeOutput | 字串 | 123-123-xxxx |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/replace.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/replace.json
```

<a id="skip" />

## <a name="skip"></a>skip
`skip(originalValue, numberToSkip)`

傳回位於指定字元數目之後的所有字元所組成的字串，或傳回位於指定元素數目之後的所有元素所形成的陣列。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| originalValue |yes |陣列或字串 |要用於略過的陣列或字串。 |
| numberToSkip |yes |int |要略過的元素或字元數。 如果此值為 0 或更小的值，則會傳回值內的所有元素或字元。 如果此值大於陣列或字串的長度，則會傳回空白陣列或字串。 |

### <a name="return-value"></a>傳回值

陣列或字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/skip.json)會略過陣列中指定的元素數目，以及字串中指定的字元數。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "elementsToSkip": {
            "type": "int",
            "defaultValue": 2
        },
        "testString": {
            "type": "string",
            "defaultValue": "one two three"
        },
        "charactersToSkip": {
            "type": "int",
            "defaultValue": 4
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "array",
            "value": "[skip(parameters('testArray'),parameters('elementsToSkip'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[skip(parameters('testString'),parameters('charactersToSkip'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayOutput | 陣列 | ["three"] |
| stringOutput | 字串 | two three |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/skip.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/skip.json
```

<a id="split" />

## <a name="split"></a>split
`split(inputString, delimiter)`

傳回包含輸入字串之子字串的字串陣列，其中的子字串已使用指定的分隔符號分隔。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| inputString |yes |字串 |要分割的字串。 |
| 分隔符號 |yes |字串或字串陣列 |用於分割字串的分隔符號。 |

### <a name="return-value"></a>傳回值

字串的陣列。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/split.json)會分割輸入字串，所使用的分割字元為逗號或分號。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "firstString": {
            "type": "string",
            "defaultValue": "one,two,three"
        },
        "secondString": {
            "type": "string",
            "defaultValue": "one;two,three"
        }
    },
    "variables": {
        "delimiters": [ ",", ";" ]
    },
    "resources": [],
    "outputs": {
        "firstOutput": {
            "type": "array",
            "value": "[split(parameters('firstString'),',')]"
        },
        "secondOutput": {
            "type": "array",
            "value": "[split(parameters('secondString'),variables('delimiters'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| firstOutput | 陣列 | ["one", "two", "three"] |
| secondOutput | 陣列 | ["one", "two", "three"] |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/split.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/split.json
```

<a id="startswith" />

## <a name="startswith"></a>startsWith
`startsWith(stringToSearch, stringToFind)`

判斷字串開頭是否為值。 此比較不區分大小寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToSearch |yes |字串 |其中包含要尋找之項目的值。 |
| stringToFind |yes |字串 |要尋找的值。 |

### <a name="return-value"></a>傳回值

如果第一個字元或字串字元與該值相符，則傳回 **True**；否則傳回 **False**。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/startsendswith.json)顯示如何使用 startsWith 和 endsWith 函式：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "startsTrue": {
            "value": "[startsWith('abcdef', 'ab')]",
            "type" : "bool"
        },
        "startsCapTrue": {
            "value": "[startsWith('abcdef', 'A')]",
            "type" : "bool"
        },
        "startsFalse": {
            "value": "[startsWith('abcdef', 'e')]",
            "type" : "bool"
        },
        "endsTrue": {
            "value": "[endsWith('abcdef', 'ef')]",
            "type" : "bool"
        },
        "endsCapTrue": {
            "value": "[endsWith('abcdef', 'F')]",
            "type" : "bool"
        },
        "endsFalse": {
            "value": "[endsWith('abcdef', 'e')]",
            "type" : "bool"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| startsTrue | Bool | True |
| startsCapTrue | Bool | True |
| startsFalse | Bool | False |
| endsTrue | Bool | True |
| endsCapTrue | Bool | True |
| endsFalse | Bool | False |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/startsendswith.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/startsendswith.json
```

<a id="string" />

## <a name="string"></a>字串
`string(valueToConvert)`

將指定的值轉換成字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| valueToConvert |yes | 任意 |要轉換成字串的值。 任何類型的值均可轉換，包括物件和陣列。 |

### <a name="return-value"></a>傳回值

轉換值的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/string.json)顯示如何將不同類型的值轉換為字串︰

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testObject": {
            "type": "object",
            "defaultValue": {
                "valueA": 10,
                "valueB": "Example Text"
            }
        },
        "testArray": {
            "type": "array",
            "defaultValue": [
                "a",
                "b",
                "c"
            ]
        },
        "testInt": {
            "type": "int",
            "defaultValue": 5
        }
    },
    "resources": [],
    "outputs": {
        "objectOutput": {
            "type": "string",
            "value": "[string(parameters('testObject'))]"
        },
        "arrayOutput": {
            "type": "string",
            "value": "[string(parameters('testArray'))]"
        },
        "intOutput": {
            "type": "string",
            "value": "[string(parameters('testInt'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| objectOutput | 字串 | {"valueA":10,"valueB":"Example Text"} |
| arrayOutput | 字串 | ["a","b","c"] |
| intOutput | 字串 | 5 |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/string.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/string.json
```

<a id="substring" />

## <a name="substring"></a>substring
`substring(stringToParse, startIndex, length)`

傳回起始於指定字元位置的子字串，其中包含指定的字元數。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToParse |yes |字串 |要用來擷取子字串的原始字串。 |
| startIndex |否 |int |起始字元位置為零的子字串。 |
| length |否 |int |子字串的字元數。 必須參考字串內的位置。 |

### <a name="return-value"></a>傳回值

子字串。

### <a name="remarks"></a>備註

子字串延伸超過字串的結尾時，此函式會失敗。 下列範例會失敗，並出現錯誤「索引與長度參數必須參考字串內的位置。 索引參數: '0'，長度參數: '11'，字串參數的長度: '10'。」。

```json
"parameters": {
    "inputString": { "type": "string", "value": "1234567890" }
},
"variables": { 
    "prefix": "[substring(parameters('inputString'), 0, 11)]"
}
```

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/substring.json)會從參數中擷取子字串。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "one two three"
        }
    },
    "resources": [],
    "outputs": {
        "substringOutput": {
            "value": "[substring(parameters('testString'), 4, 3)]",
            "type": "string"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| substringOutput | 字串 | two |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/substring.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/substring.json
```

<a id="take" />

## <a name="take"></a>take
`take(originalValue, numberToTake)`

傳回由字串開頭的指定字元數目所形成的字串，或傳回由陣列開頭的指定元素數目所組成的陣列。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| originalValue |yes |陣列或字串 |要從其中擷取元素的陣列或字串。 |
| numberToTake |yes |int |要擷取的元素或字元數。 如果此值為 0 或更小的值，則會傳回空白陣列或字串。 如果此值大於給定陣列或字串的長度，則會傳回陣列或字串中的所有元素。 |

### <a name="return-value"></a>傳回值

陣列或字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/take.json)會從陣列中取得指定的元素數目，以及從字串中取得指定的字元數目。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testArray": {
            "type": "array",
            "defaultValue": [
                "one",
                "two",
                "three"
            ]
        },
        "elementsToTake": {
            "type": "int",
            "defaultValue": 2
        },
        "testString": {
            "type": "string",
            "defaultValue": "one two three"
        },
        "charactersToTake": {
            "type": "int",
            "defaultValue": 2
        }
    },
    "resources": [],
    "outputs": {
        "arrayOutput": {
            "type": "array",
            "value": "[take(parameters('testArray'),parameters('elementsToTake'))]"
        },
        "stringOutput": {
            "type": "string",
            "value": "[take(parameters('testString'),parameters('charactersToTake'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| arrayOutput | 陣列 | ["one", "two"] |
| stringOutput | 字串 | on |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/take.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/take.json
```

<a id="tolower" />

## <a name="tolower"></a>toLower
`toLower(stringToChange)`

將指定的字串轉換為小寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToChange |yes |字串 |要轉換成小寫字母的值。 |

### <a name="return-value"></a>傳回值

字串已轉換成小寫。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/tolower.json)會將參數值轉換為小寫和大寫。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "One Two Three"
        }
    },
    "resources": [],
    "outputs": {
        "toLowerOutput": {
            "value": "[toLower(parameters('testString'))]",
            "type": "string"
        },
        "toUpperOutput": {
            "type": "string",
            "value": "[toUpper(parameters('testString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| toLowerOutput | 字串 | one two three |
| toUpperOutput | 字串 | ONE TWO THREE |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/tolower.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/tolower.json
```

<a id="toupper" />

## <a name="toupper"></a>toUpper
`toUpper(stringToChange)`

將指定的字串轉換為大寫。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToChange |yes |字串 |要轉換成大寫字母的值。 |

### <a name="return-value"></a>傳回值

字串已轉換成大寫。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/tolower.json)會將參數值轉換為小寫和大寫。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "One Two Three"
        }
    },
    "resources": [],
    "outputs": {
        "toLowerOutput": {
            "value": "[toLower(parameters('testString'))]",
            "type": "string"
        },
        "toUpperOutput": {
            "type": "string",
            "value": "[toUpper(parameters('testString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| toLowerOutput | 字串 | one two three |
| toUpperOutput | 字串 | ONE TWO THREE |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/tolower.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/tolower.json
```

<a id="trim" />

## <a name="trim"></a>修剪
`trim (stringToTrim)`

從指定的字串中移除所有開頭和尾端空白字元。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToTrim |yes |字串 |要修剪的值。 |

### <a name="return-value"></a>傳回值

沒有開頭和尾端空白字元的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/trim.json)會修剪參數的空白字元。

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "testString": {
            "type": "string",
            "defaultValue": "    one two three   "
        }
    },
    "resources": [],
    "outputs": {
        "return": {
            "type": "string",
            "value": "[trim(parameters('testString'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| 傳回 | 字串 | one two three |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/trim.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/trim.json
```

<a id="uniquestring" />

## <a name="uniquestring"></a>uniqueString
`uniqueString (baseString, ...)`

根據當作參數提供的值，建立具決定性的雜湊字串。 

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| baseString |yes |字串 |雜湊函式中用來建立唯一字串的值。 |
| 視需要，也會使用其他參數 |否 |字串 |您可以視需要新增多個字串，來建立指定唯一性層級的值。 |

### <a name="remarks"></a>備註

當您需要建立資源的唯一名稱時，這個函式很有幫助。 您提供限制結果唯一性範圍的參數值。 您可以指定名稱對於訂用帳戶、資源群組或部署是否唯一。 

傳回的值不是隨機字串，而是雜湊函式的結果。 傳回的值為 13 個字元長。 它不是全域唯一的。 建議您將值與來自命名慣例的前置詞結合，建立有意義的名稱。 下列範例顯示傳回值的格式。 依提供的參數而改變的實際值。

    tcvhiyu5h2o5o

下列範例顯示如何使用 uniqueString 來建立常用層級的唯一值。

在訂用帳戶範圍內是唯一

```json
"[uniqueString(subscription().subscriptionId)]"
```

在資源群組範圍內是唯一

```json
"[uniqueString(resourceGroup().id)]"
```

在資源群組的部署範圍內是唯一

```json
"[uniqueString(resourceGroup().id, deployment().name)]"
```

下列範例顯示如何根據您的資源群組建立儲存體帳戶的唯一名稱。 在資源群組內，如果名稱是以相同方式建構，就不是唯一的名稱。

```json
"resources": [{ 
    "name": "[concat('storage', uniqueString(resourceGroup().id))]", 
    "type": "Microsoft.Storage/storageAccounts", 
    ...
```

### <a name="return-value"></a>傳回值

包含 13 個字元的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/uniquestring.json)會從 uniquestring 傳回結果：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [],
    "outputs": {
        "uniqueRG": {
            "value": "[uniqueString(resourceGroup().id)]",
            "type" : "string"
        },
        "uniqueDeploy": {
            "value": "[uniqueString(resourceGroup().id, deployment().name)]",
            "type" : "string"
        }
    }
}
```

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uniquestring.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uniquestring.json
```

<a id="uri" />

## <a name="uri"></a>uri
`uri (baseUri, relativeUri)`

藉由結合 baseUri 和 relativeUri 字串建立絕對 URI。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| baseUri |yes |字串 |基底 uri 的字串。 |
| relativeUri |yes |字串 |要加入至基底 uri 字串的相對 uri 字串。 |

**baseUri** 參數的值可包含特定檔案，但在建構 URI 時只會使用基底路徑。 例如，將 `http://contoso.com/resources/azuredeploy.json` 作為 baseUri 參數傳遞時，會產生 `http://contoso.com/resources/` 的基底 URI。

### <a name="return-value"></a>傳回值

字串，代表基底和相對值的絕對 URI。

### <a name="examples"></a>範例

下列範例顯示如何根據上層範本的值建構巢狀範本的連結。

```json
"templateLink": "[uri(deployment().properties.templateLink.uri, 'nested/azuredeploy.json')]"
```

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/uri.json)顯示如何使用 uri、uriComponent 和 uriComponentToString：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "uriFormat": "[uri('http://contoso.com/resources/', 'nested/azuredeploy.json')]",
        "uriEncoded": "[uriComponent(variables('uriFormat'))]" 
    },
    "resources": [
    ],
    "outputs": {
        "uriOutput": {
            "type": "string",
            "value": "[variables('uriFormat')]"
        },
        "componentOutput": {
            "type": "string",
            "value": "[variables('uriEncoded')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[uriComponentToString(variables('uriEncoded'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| uriOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |
| componentOutput | 字串 | http%3A%2F%2Fcontoso.com%2Fresources%2Fnested%2Fazuredeploy.json |
| toStringOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

<a id="uricomponent" />

## <a name="uricomponent"></a>uriComponent
`uricomponent(stringToEncode)`

將 URI 編碼。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| stringToEncode |yes |字串 |要編碼的值。 |

### <a name="return-value"></a>傳回值

URI 編碼值的字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/uri.json)顯示如何使用 uri、uriComponent 和 uriComponentToString：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "uriFormat": "[uri('http://contoso.com/resources/', 'nested/azuredeploy.json')]",
        "uriEncoded": "[uriComponent(variables('uriFormat'))]" 
    },
    "resources": [
    ],
    "outputs": {
        "uriOutput": {
            "type": "string",
            "value": "[variables('uriFormat')]"
        },
        "componentOutput": {
            "type": "string",
            "value": "[variables('uriEncoded')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[uriComponentToString(variables('uriEncoded'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| uriOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |
| componentOutput | 字串 | http%3A%2F%2Fcontoso.com%2Fresources%2Fnested%2Fazuredeploy.json |
| toStringOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

<a id="uricomponenttostring" />

## <a name="uricomponenttostring"></a>uriComponentToString
`uriComponentToString(uriEncodedString)`

傳回 URI 編碼值的字串。

### <a name="parameters"></a>參數

| 參數 | 必要 | 類型 | 說明 |
|:--- |:--- |:--- |:--- |
| uriEncodedString |yes |字串 |要轉換為字串的 URI 編碼值。 |

### <a name="return-value"></a>傳回值

URI 編碼值的解碼字串。

### <a name="examples"></a>範例

下列[範例範本](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/functions/uri.json)顯示如何使用 uri、uriComponent 和 uriComponentToString：

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "variables": {
        "uriFormat": "[uri('http://contoso.com/resources/', 'nested/azuredeploy.json')]",
        "uriEncoded": "[uriComponent(variables('uriFormat'))]" 
    },
    "resources": [
    ],
    "outputs": {
        "uriOutput": {
            "type": "string",
            "value": "[variables('uriFormat')]"
        },
        "componentOutput": {
            "type": "string",
            "value": "[variables('uriEncoded')]"
        },
        "toStringOutput": {
            "type": "string",
            "value": "[uriComponentToString(variables('uriEncoded'))]"
        }
    }
}
```

上述範例中具有預設值的輸出如下：

| Name | 類型 | 值 |
| ---- | ---- | ----- |
| uriOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |
| componentOutput | 字串 | http%3A%2F%2Fcontoso.com%2Fresources%2Fnested%2Fazuredeploy.json |
| toStringOutput | 字串 | http://contoso.com/resources/nested/azuredeploy.json |

若要使用 Azure CLI 部署此範例範本，請使用：

```azurecli-interactive
az group deployment create -g functionexamplegroup --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

若要使用 PowerShell 部署此範例範本，請使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName functionexamplegroup -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/azure-resource-manager/functions/uri.json
```

## <a name="next-steps"></a>後續步驟
* 如需有關 Azure Resource Manager 範本中各區段的說明，請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)。
* 若要合併多個範本，請參閱[透過 Azure Resource Manager 使用連結的範本](resource-group-linked-templates.md)。
* 若要依指定的次數重複建立資源類型，請參閱 [在 Azure 資源管理員中建立資源的多個執行個體](resource-group-create-multiple.md)。
* 若要了解如何部署已建立的範本，請參閱[使用 Azure Resource Manager 範本部署應用程式](resource-group-template-deploy.md)。

