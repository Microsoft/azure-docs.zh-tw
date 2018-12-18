---
title: Azure StorageAccountSelector UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Storage.StorageAccountSelector UI 元素。
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
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: 5de536a562d234a4c463c862aedffc7c7ca5228d
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37112281"
---
# <a name="microsoftstoragestorageaccountselector-ui-element"></a>Microsoft.Storage.StorageAccountSelector UI 元素
選取新的或現有儲存體帳戶的控制項。

## <a name="ui-sample"></a>UI 範例

控制項會顯示預設值。

![Microsoft.Storage.StorageAccountSelector](./media/managed-application-elements/microsoft.storage.storageaccountselector.png)

控制項可讓使用者建立新儲存體帳戶，或者選取現有的儲存體帳戶。

![Microsoft.Storage.StorageAccountSelector new](./media/managed-application-elements/microsoft.storage.storageaccountselector-new.png)

## <a name="schema"></a>結構描述

```json
{
  "name": "element1",
  "type": "Microsoft.Storage.StorageAccountSelector",
  "label": "Storage account",
  "toolTip": "",
  "defaultValue": {
    "name": "storageaccount01",
    "type": "Premium_LRS"
  },
  "constraints": {
    "allowedTypes": [],
    "excludedTypes": []
  },
  "options": {
    "hideExisting": false
  },
  "visible": true
}
```

## <a name="remarks"></a>備註
- 如果指定，就會自動驗證 `defaultValue.name` 的唯一性。 如果儲存體帳戶名稱不是唯一的，使用者就必須指定不同的名稱，或選擇現有的儲存體帳戶。
- `defaultValue.type` 的預設值為 **Premium_LRS**。
- `constraints.allowedTypes` 中未指定的任何類型都會加以隱藏，`constraints.excludedTypes` 中未指定的任何類型都會加以顯示。 `constraints.allowedTypes` 和 `constraints.excludedTypes` 都是選擇性的，但不能同時使用。
- 如果 `options.hideExisting` 為 **true**，使用者就無法選擇現有的儲存體帳戶。 預設值為 **false**。

## <a name="sample-output"></a>範例輸出

```json
{
  "name": "storageaccount01",
  "resourceGroup": "rg01",
  "type": "Premium_LRS",
  "newOrExisting": "new"
}
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
