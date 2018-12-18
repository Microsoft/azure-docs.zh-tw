---
title: Azure OptionsGroup UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Common.OptionsGroup UI 元素。
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
ms.openlocfilehash: e51061dc781e4ec6e822cde9cc450887ff3b1368
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061180"
---
# <a name="microsoftcommonoptionsgroup-ui-element"></a>Microsoft.Common.OptionsGroup UI 元素
包含可用選項資料列的選取控制項。

## <a name="ui-sample"></a>UI 範例
![Microsoft.Common.OptionsGroup](./media/managed-application-elements/microsoft.common.optionsgroup.png)

## <a name="schema"></a>結構描述
```json
{
  "name": "element1",
  "type": "Microsoft.Common.OptionsGroup",
  "label": "Some options group",
  "defaultValue": "Value two",
  "toolTip": "",
  "constraints": {
    "allowedValues": [
      {
        "label": "Value one",
        "value": "one"
      },
      {
        "label": "Value two",
        "value": "two"
      }
    ],
    "required": true
  },
  "visible": true
}
```

## <a name="remarks"></a>備註
- `constraints.allowedValues` 的標籤是項目的顯示文字，其值為選取時的元素輸出值。
- 如果指定，預設值必須是 `constraints.allowedValues` 中存在的標籤。 如果未指定，依預設會選取 `constraints.allowedValues` 中的第一個項目。 預設值為 **null**。
- `constraints.allowedValues` 必須有至少一個項目。

## <a name="sample-output"></a>範例輸出
```json
"two"
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
