---
title: Azure 雲端服務 NetworkConfiguration 結構描述 | Microsoft Docs
ms.custom: ''
ms.date: 12/07/2016
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: c1b94a9e-46e8-4a18-ac99-343c94b1d4bd
caps.latest.revision: 28
author: jpconnock
ms.author: jeconnoc
manager: timlt
ms.openlocfilehash: ed071d1da30a598eef830b4485c246ffae09c950
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39004109"
---
# <a name="azure-cloud-services-config-networkconfiguration-schema"></a>Azure 雲端服務組態 NetworkConfiguration 結構描述

服務組態檔的 `NetworkConfiguration` 元素會指定虛擬網路和 DNS 值。 雲端服務的這些設定是選擇性的。

若要深入了解虛擬網路和相關聯的結構描述，您可以使用下列資源：

- [雲端服務 (傳統) 組態結構描述](schema-cscfg-file.md)
- [雲端服務 (傳統) 定義結構描述](schema-csdef-file.md)
- [建立虛擬網路 (傳統)](../virtual-network/virtual-networks-create-vnet-classic-pportal.md)

## <a name="networkconfiguration-element"></a>NetworkConfiguration 元素
下列範例說明 `NetworkConfiguration` 元素和其子項目。

```xml
<ServiceConfiguration>
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="aclName1">
        <Rule order="<rule-order>" action="<rule-action>" remoteSubnet="<subnet-address>" description="rule-description"/>
      </AccessControl>
    </AccessControls>
    <EndpointAcls>
      <EndpointAcl role="<role-name>" endpoint="<endpoint-name>" accessControl="<acl-name>"/>
    </EndpointAcls>
    <Dns>
      <DnsServers>
        <DnsServer name="<server-name>" IPAddress="<server-address>" />
      </DnsServers>
    </Dns>
    <VirtualNetworkSite name="<site-name>"/>
    <AddressAssignments>
      <InstanceAddress roleName="<role-name>">
        <Subnets>
          <Subnet name="<subnet-name>"/>
        </Subnets>
      </InstanceAddress>
      <ReservedIPs>
        <ReservedIP name="<reserved-ip-name>"/>
      </ReservedIPs>
    </AddressAssignments>
  </NetworkConfiguration>
</ServiceConfiguration>
```

下表說明 `NetworkConfiguration` 元素的子項目。

| 元素       | 說明 |
| ------------- | ----------- |
| AccessControl | 選用。 指定存取雲端服務端點的規則。 存取控制名稱是由 `name` 屬性的字串所定義。 `AccessControl` 元素包含一或多個 `Rule` 元素。 可以定義多個 `AccessControl` 元素。|
| 規則 | 選用。 指定 IP 位址的指定子網路範圍所應採取的動作。 規則的順序是由 `order` 屬性的字串值所定義。 規則編號愈低，優先順序愈高。 例如，可以 100、200 和 300 的順序編號來指定規則。 順序編號為 100 的規則優先於順序為 200 的規則。<br /><br /> 規則的動作是由 `action` 屬性的字串所定義。 可能的值包括：<br /><br /> -   `permit` – 指定只有指定子網路範圍的封包可與端點通訊。<br />-   `deny` – 指定拒絕存取指定子網路範圍內的端點。<br /><br /> 受規則影響的 IP 位址的子網路範圍是由 `remoteSubnet` 屬性的字串所定義。 規則的說明是由 `description` 屬性的字串所定義。|
| EndpointAcl | 選用。 指定對端點之存取控制規則的指派。 包含端點的角色名稱是由 `role` 屬性的字串所定義。 端點的名稱是由 `endpoint` 屬性的字串所定義。 應套用至端點的 `AccessControl` 規則集名稱是由 `accessControl` 屬性的字串所定義。 可以定義多個 `EndpointAcl` 元素。|
| DnsServer | 選用。 指定 DNS 伺服器的設定。 您無須使用虛擬網路就可以指定 DNS 伺服器的設定。 DNS 伺服器的名稱是由 `name` 屬性的字串所定義。 DNS 伺服器的 IP 位址是由 `IPAddress` 屬性的字串所定義。 IP 位址必須是有效的 IPv4 位址。|
| VirtualNetworkSite | 選用。 指定您要在其中部署雲端服務的虛擬網路站台名稱。 此設定不會建立虛擬網路站台。 它會參考先前已在虛擬網路網路檔案中定義的站台。 雲端服務只能是一個虛擬網路的成員。 如果您未指定此設定，雲端服務就不會部署到虛擬網路。 虛擬網路站台的名稱是由 `name` 屬性的字串所定義。|
| InstanceAddress | 選用。 在虛擬網路中，將角色的關聯指定為子網路或子網路集。 當您將角色名稱與執行個體位址建立關聯時，可以指定您需要與此角色相關聯的子網路。 `InstanceAddress` 包含一個 Subnets 元素。 與一個或多個子網路相關聯的角色名稱是由 `roleName` 屬性的字串所定義。|
| 子網路 | 選用。 指定對應至網路組態檔中之子網路名稱的子網路。 子網路的名稱是由 `name` 屬性的字串所定義。|
| ReservedIP | 選用。 指定應與部署相關聯的保留 IP 位址。 您必須使用「建立保留的 IP 位址」才能建立保留的 IP 位址。 雲端服務中的每個部署都可與一個保留的 IP 位址相關聯。 保留 IP 位址的名稱是由 `name` 屬性的字串所定義。|

## <a name="see-also"></a>另請參閱
[雲端服務 (傳統) 組態結構描述](schema-cscfg-file.md)