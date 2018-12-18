---
title: Azure 雲端服務角色結構描述 | Microsoft Docs
ms.custom: ''
ms.date: 12/07/2016
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: e4fbffc1-98eb-449c-971c-de415e45ab34
caps.latest.revision: 12
author: jpconnock
ms.author: jeconnoc
manager: timlt
ms.openlocfilehash: 20f4186426152d2dc9b445981a69881c35587eb6
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39005776"
---
# <a name="azure-cloud-services-config-role-schema"></a>Azure 雲端服務設定角色結構描述

組態檔的 `Role` 元素會指定要為服務中每個角色部署的角色執行個體數目、任何組態設定的值，以及與角色相關聯之任何憑證的指紋。

如需關於 Azure 組態結構描述的詳細資訊，請參閱[雲端服務 (傳統) 組態結構描述](schema-cscfg-file.md)。 如需關於 Azure 服務定義結構描述的詳細資訊，請參閱[雲端服務 (傳統) 定義結構描述](schema-csdef-file.md)。

##  <a name="Role"></a> Role 元素
下列範例說明 `Role` 元素和其子元素。

```xml 
<ServiceConfiguration>
  <Role name="<role-name>" vmName="<vm-name>">
    <Instances count="<number-of-instances>"/>
    <ConfigurationSettings>
      <Setting name="<setting-name>" value="<setting-value>" />
    </ConfigurationSettings>
    <Certificates>
      <Certificate name="<certificate-name>" thumbprint="<certificate-thumbprint>" thumbprintAlgorithm="<algorithm>"/>
    </Certificates>
  </Role>
</ServiceConfiguration>
```

下表說明 `Role` 元素的屬性。

| 屬性 | 說明 |
| --------- | ----------- |
| name   | 必要。 指定角色的名稱。 名稱必須符合為服務定義檔中之角色提供的名稱。|
| vmName | 選用。 指定虛擬機器的 DNS 名稱。 名稱長度必須等於或小於 10 個字元。|

下表說明 `Role` 元素的子項目。

| 元素 | 說明 |
| ------- | ----------- |
| 執行個體 | 必要。 指定要為角色部署的執行個體數目。 執行個體數目是由 `count` 屬性的整數所定義。|
| 設定   | 選用。 指定角色設定集合中的設定名稱和值。 設定名稱是由 `name` 屬性的字串所定義，而設定值是由 `value` 屬性的字串所定義。|
| 憑證 | 選用。 指定名稱、指紋和要與角色相關聯之服務憑證的演算法。 憑證名稱是由 `name` 屬性的字串所定義。 憑證指紋是由十六進位數字的字串所定義，該字串針對 `thumbprint` 屬性不包含空格。 十六進位數字必須使用數字和大寫英文字元來表示。 憑證演算法是由 `thumbprintAlgorithm` 屬性的字串所定義。|

## <a name="see-also"></a>另請參閱
[雲端服務 (傳統) 組態結構描述](schema-cscfg-file.md)