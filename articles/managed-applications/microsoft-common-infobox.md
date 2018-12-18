---
title: Azure InfoBox UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Common.TextBlock UI 元素。
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
ms.date: 06/15/2018
ms.author: tomfitz
ms.openlocfilehash: abd1329f2ebac90bf846dfd5fc5b307ddb5e52bd
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095475"
---
# <a name="microsoftcommoninfobox-ui-element"></a>Microsoft.Common.InfoBox UI 元素
可新增資訊方塊的控制項。 此方塊包含重要的文字或警告，可協助使用者了解他們所要提供的值。 它也可以連結至 URI 來提供更多資訊。

## <a name="ui-sample"></a>UI 範例
![Microsoft.Common.InfoBox](./media/managed-application-elements/microsoft.common.infobox.png)


## <a name="schema"></a>結構描述
```json
{
  "name": "text1",
  "type": "Microsoft.Common.InfoBox",
  "visible": true,
  "options": {
    "icon": "None",
    "text": "Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo.",
    "uri": "https://www.microsoft.com"
  }
}
```

## <a name="remarks"></a>備註

* 對於 `icon`，請使用 **無**、**資訊**、**警告**或**錯誤**。
* `uri` 是選用屬性。

## <a name="sample-output"></a>範例輸出

```json
"Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo."
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
