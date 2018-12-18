---
title: Azure TextBox UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Common.TextBox UI 元素。
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: managed-applications
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: fa3e5fff8080acb9e824ffe27f6c149054804830
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37063641"
---
# <a name="microsoftcommontextbox-ui-element"></a>Microsoft.Common.TextBox UI 元素
可以用來編輯未格式化文字的控制項。

## <a name="ui-sample"></a>UI 範例
![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft.common.textbox.png)

## <a name="schema"></a>結構描述
```json
{
  "name": "element1",
  "type": "Microsoft.Common.TextBox",
  "label": "Example text box 1",
  "defaultValue": "my text value",
  "toolTip": "Use only allowed characters",
  "constraints": {
    "required": true,
    "regex": "^[a-z0-9A-Z]{1,30}$",
    "validationMessage": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
  },
  "visible": true
}
```

## <a name="remarks"></a>備註
- 如果將 `constraints.required` 設為 **true**，則文字方塊必須有值，才能順利通過驗證。 預設值為 **false**。
- `constraints.regex` 是 JavaScript 規則運算式模式。 如果指定，文字方塊的值就必須符合模式，才能順利通過驗證。 預設值為 **null**。
- 當文字方塊的值無法通過驗證時，就會顯示 `constraints.validationMessage` 字串。 如果未指定，則會使用文字方塊的內建的驗證訊息。 預設值為 **null**。
- 當 `constraints.required` 設為 **false** 時，您可指定 `constraints.regex` 的值。 在此案例中，文字方塊不需要值，即可順利通過驗證。 如果指定的話，它必須符合規則運算式模式。

## <a name="sample-output"></a>範例輸出

```json
"my text value"
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
