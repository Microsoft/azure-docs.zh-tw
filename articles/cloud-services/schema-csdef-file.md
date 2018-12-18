---
title: Azure 雲端服務定義結構描述 (.csdef 檔) | Microsoft Docs
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: b7735dbf-8e91-4d1b-89f7-2f17e9302469
caps.latest.revision: 42
author: jpconnock
ms.author: jeconnoc
manager: timlt
ms.openlocfilehash: df2f7c1bf99c13779e5720e15d8d669aa4f945c0
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39002744"
---
# <a name="azure-cloud-services-definition-schema-csdef-file"></a>Azure 雲端服務定義結構描述 (.csdef 檔)
服務定義檔會定義應用程式的服務模型。 檔案會包含可供雲端服務使用的角色定義、指定服務端點，並建立服務的組態設定。 組態設定值是在服務組態檔中設定，如[雲端服務 (傳統) 組態結構描述](http://msdn.microsoft.com/library/b1ae68cd-cc95-48cb-a4a4-da91dc708a35)所述。

根據預設，Azure 診斷組態結構描述檔案是安裝於 `C:\Program Files\Microsoft SDKs\Windows Azure\.NET SDK\<version>\schemas` 目錄中。 請使用安裝的 [Azure SDK](http://www.windowsazure.com/develop/downloads/) 版本來取代 `<version>`。

服務定義檔的預設副檔名為 .csdef。

## <a name="basic-service-definition-schema"></a>基本服務定義結構描述
服務定義檔必須包含一個 `ServiceDefinition` 元素。 服務定義必須至少包含一個角色 (`WebRole` 或 `WorkerRole`) 元素。 它可以包含單一定義中所定義的最多 25 個角色，還可以混合角色類型。 服務定義也包含選擇性 `NetworkTrafficRules` 元素，可限制哪些角色可與指定的內部端點通訊。 服務定義也包含選擇性 `LoadBalancerProbes` 元素，其內含客戶定義之端點的健康情況探查。

服務定義檔的基本格式如下所示。

```xml
<ServiceDefinition name="<service-name>" topologyChangeDiscovery="<change-type>" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" upgradeDomainCount="<number-of-upgrade-domains>" schemaVersion="<version>">
  
  <LoadBalancerProbes>
         …
  </LoadBalancerProbes>
  
  <WebRole …>
         …
  </WebRole>
  
  <WorkerRole …>
         …
  </WorkerRole>
  
  <NetworkTrafficRules>
         …
  </NetworkTrafficRules>

</ServiceDefinition>
```

## <a name="schema-definitions"></a>結構描述定義
下列主題說明結構描述：

- [LoadBalancerProbe 結構描述](schema-csdef-loadbalancerprobe.md)
- [WebRole 結構描述](schema-csdef-webrole.md)
- [WorkerRole 結構描述](schema-csdef-workerrole.md)
- [NetworkTrafficRules 結構描述](schema-csdef-networktrafficrules.md)

##  <a name="ServiceDefinition"></a> ServiceDefinition 元素
`ServiceDefinition` 元素是服務定義檔的最上層元素。

下表說明 `ServiceDefinition` 元素的屬性。

| 屬性               | 說明 |
| ----------------------- | ----------- |
| name                    |必要。 服務的名稱。 名稱在服務帳戶內必須是唯一的。|
| topologyChangeDiscovery | 選用。 指定拓撲變更通知的類型。 可能的值包括：<br /><br /> -   `Blast` - 儘速將更新傳送到所有角色執行個體。 如果您選擇選項，角色就應該能夠處理拓撲更新而不必重新啟動。<br />-   `UpgradeDomainWalk` - 在前一個執行個體成功接受更新之後，以循序方式將更新傳送到每個角色執行個體。|
| schemaVersion           | 選用。 指定服務定義結構描述的版本。 如果已並行安裝一個以上的 SDK 版本，結構描述版本就可讓 Visual Studio 選取要用於結構描述驗證的正確 SDK 工具。|
| upgradeDomainCount      | 選用。 指定要配置此服務中角色的升級網域數目。 部署服務時，會將角色執行個體配置給升級網域。 如需詳細資訊，請參閱[更新雲端服務角色或部署](cloud-services-how-to-manage-portal.md#update-a-cloud-service-role-or-deployment)。<br /><br /> 您可以指定最多 20 個升級網域。 如果未指定，預設的升級網域數目為 5。|