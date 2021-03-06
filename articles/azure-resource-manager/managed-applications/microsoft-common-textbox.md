---
title: TextBox UI 元素
description: 描述 Azure 入口網站的 Microsoft.Common.TextBox UI 元素。 用於新增未格式化的文字。
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: c89bc434d9d67144a95b5c2f23e7664078fe7825
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "87474303"
---
# <a name="microsoftcommontextbox-ui-element"></a>Microsoft.Common.TextBox UI 元素

可以用來編輯未格式化文字的控制項。

## <a name="ui-sample"></a>UI 範例

![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft-common-textbox.png)

## <a name="schema"></a>結構描述

```json
{
    "name": "nameInstance",
    "type": "Microsoft.Common.TextBox",
    "label": "Name",
    "defaultValue": "contoso123",
    "toolTip": "Use only allowed characters",
    "placeholder": "",
    "constraints": {
        "required": true,
        "validations": [
            {
                "regex": "^[a-z0-9A-Z]{1,30}$",
                "message": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
            },
            {
                "isValid": "[startsWith(steps('resourceConfig').nameInstance, 'contoso')]",
                "message": "Must start with 'contoso'."
            }
        ]
    },
    "visible": true
}
```

## <a name="sample-output"></a>範例輸出

```json
"contoso123"
```

## <a name="remarks"></a>備註

- 如果將 `constraints.required` 設為 **true**，則文字方塊必須有值，才能順利通過驗證。 預設值為 **false**。
- `validations`屬性是一個陣列，您可以在其中加入條件來檢查文字方塊中提供的值。
- `regex`屬性是 JavaScript 正則運算式模式。 如果指定，文字方塊的值就必須符合模式，才能順利通過驗證。 預設值為 **null**。
- `isValid`屬性包含評估結果為 true 或 false 的運算式。 在運算式中，您可以定義判斷文字方塊是否有效的條件。
- `message`當文字方塊的值無法通過驗證時，屬性是要顯示的字串。
- 當 `required` 設為 **false** 時，您可指定 `regex` 的值。 在此案例中，文字方塊不需要值，即可順利通過驗證。 如果指定的話，它必須符合規則運算式模式。
- `placeholder`當使用者開始編輯時，屬性是解說文字會消失。 如果 `placeholder` 和 `defaultValue` 都已定義， `defaultValue` 就會優先使用並顯示。

## <a name="example"></a>範例

下列範例會使用具有 [ArmApiControl](microsoft-solutions-armapicontrol.md) 控制項的文字方塊來檢查資源名稱的可用性。

```json
"basics": [
    {
        "name": "nameApi",
        "type": "Microsoft.Solutions.ArmApiControl",
        "request": {
            "method": "POST",
            "path": "[concat(subscription().id, '/providers/Microsoft.Storage/checkNameAvailability?api-version=2019-06-01')]",
            "body": "[parse(concat('{\"name\": \"', basics('txtStorageName'), '\", \"type\": \"Microsoft.Storage/storageAccounts\"}'))]"
        }
    },
    {
        "name": "txtStorageName",
        "type": "Microsoft.Common.TextBox",
        "label": "Storage account name",
        "constraints": {
            "validations": [
                {
                    "isValid": "[not(equals(basics('nameApi').nameAvailable, false))]",
                    "message": "[concat('Name unavailable: ', basics('txtStorageName'))]"
                }
            ]
        }
    }
]
```

## <a name="next-steps"></a>接下來的步驟

* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
