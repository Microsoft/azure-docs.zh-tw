---
title: Azure VirtualNetworkCombo UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Network.VirtualNetworkCombo UI 元素。
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
ms.openlocfilehash: 2c2553d9ffb1dfbe032385fb77e234a8b96cb239
ms.sourcegitcommit: 5a7f13ac706264a45538f6baeb8cf8f30c662f8f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37110060"
---
# <a name="microsoftnetworkvirtualnetworkcombo-ui-element"></a>Microsoft.Network.VirtualNetworkCombo UI 元素
選取新的或現有虛擬網路的控制項群組。

## <a name="ui-sample"></a>UI 範例
當使用者選取新的虛擬網路時，可以自訂每一個子網路的名稱和位址前置詞。 設定子網路為選擇性的。

![Microsoft.Network.VirtualNetworkCombo new](./media/managed-application-elements/microsoft.network.virtualnetworkcombo-new.png)

當使用者選取現有的虛擬網路時，必須將部署範本所需的每個子網路對應到現有的子網路。 在此情況下，設定子網路為必要的。

![Microsoft.Network.VirtualNetworkCombo existing](./media/managed-application-elements/microsoft.network.virtualnetworkcombo-existing.png)

## <a name="schema"></a>結構描述
```json
{
  "name": "element1",
  "type": "Microsoft.Network.VirtualNetworkCombo",
  "label": {
    "virtualNetwork": "Virtual network",
    "subnets": "Subnets"
  },
  "toolTip": {
    "virtualNetwork": "",
    "subnets": ""
  },
  "defaultValue": {
    "name": "vnet01",
    "addressPrefixSize": "/16"
  },
  "constraints": {
    "minAddressPrefixSize": "/16"
  },
  "options": {
    "hideExisting": false
  },
  "subnets": {
    "subnet1": {
      "label": "First subnet",
      "defaultValue": {
        "name": "subnet-1",
        "addressPrefixSize": "/24"
      },
      "constraints": {
        "minAddressPrefixSize": "/24",
        "minAddressCount": 12,
        "requireContiguousAddresses": true
      }
    },
    "subnet2": {
      "label": "Second subnet",
      "defaultValue": {
        "name": "subnet-2",
        "addressPrefixSize": "/26"
      },
      "constraints": {
        "minAddressPrefixSize": "/26",
        "minAddressCount": 8,
        "requireContiguousAddresses": true
      }
    }
  },
  "visible": true
}
```

## <a name="remarks"></a>備註
- 如果指定，就會根據使用者訂用帳戶中的現有虛擬網路，自動決定 `defaultValue.addressPrefixSize` 大小的第一個非重疊位址前置詞。
- `defaultValue.name` 和 `defaultValue.addressPrefixSize` 的預設值為 **null**。
- 必須指定 `constraints.minAddressPrefixSize`。 任何現有虛擬網路的位址空間若低於指定的值，就無法加以選取。
- 必須指定 `subnets`，且必須指定每個子網路的 `constraints.minAddressPrefixSize`。
- 在建立新的虛擬網路時，會根據虛擬網路的位址前置詞和個別的 `addressPrefixSize`自動計算每個子網路的位址首碼。
- 在使用現有的虛擬網路時，無法選取任何小於個別 `constraints.minAddressPrefixSize` 的子網路。 此外，如果指定，就無法選取未至少包含 `minAddressCount` 個可用位址的子網路。 預設值為 **0**。 若要確保可用位址是連續的，請將 `requireContiguousAddresses` 指定為 **true**。 預設值為 **true**。
- 不支援在現有的虛擬網路中建立子網路。
- 如果 `options.hideExisting` 為 **true**，使用者就無法選擇現有的虛擬網路。 預設值為 **false**。

## <a name="sample-output"></a>範例輸出

```json
{
  "name": "vnet01",
  "resourceGroup": "rg01",
  "addressPrefixes": ["10.0.0.0/16"],
  "newOrExisting": "new",
  "subnets": {
    "subnet1": {
      "name": "subnet-1",
      "addressPrefix": "10.0.0.0/24",
      "startAddress": "10.0.0.1"
    },
    "subnet2": {
      "name": "subnet-2",
      "addressPrefix": "10.0.1.0/26",
      "startAddress": "10.0.1.1"
    }
  }
}
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
