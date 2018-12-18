---
title: Azure 事件中樞適用的虛擬網路服務端點和規則 | Microsoft Docs
description: 新增 Microsoft.EventHub 服務端點至虛擬網路。
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.date: 08/16/2018
ms.author: shvija
ms.openlocfilehash: b8c3270149c254898ad3180b92a4ff398f3efb6c
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42745904"
---
# <a name="use-virtual-network-service-endpoints-with-azure-event-hubs"></a>將虛擬網路服務端點搭配 Azure 事件中樞使用

將事件中樞與[虛擬網路 (VNet) 服務端點][vnet-sep]整合，可以安全存取來自虛擬機器 (繫結至虛擬網路) 等工作負載的傳訊功能，而且兩端的網路流量路徑都能受到保護。 

一旦設定為繫結到至少一個虛擬網路子網路服務端點，這些端點各自的事件中樞命名空間除了授權的虛擬網路以外，無法再接受來自任何位置的流量。 從虛擬網路的角度來看，將事件中樞命名空間繫結至服務端點，會設定從虛擬網路子網路到傳訊服務的隔離式網路通道。

最終這會在繫結至子網路的工作負載與各自的事件中樞命名空間之間，建立私人且隔離的關係，儘管傳訊服務端點顯示的網路位址位於公用 IP 範圍中。

## <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>VNet 整合所實現的進階安全性案例 

需要緊密和劃分安全性，而且虛擬網路子網路分割各劃分服務的解決方案，通常仍然需要位於這些區間之各項服務之間的通訊路徑。

區間之間的任何立即 IP 路由，包括那些運輸 HTTPS over TCP/IP，會帶來上層網路的弱點危害風險。 傳訊服務提供完全隔離的通訊路徑，在該路徑中，若通訊雙方之間轉換訊息，甚至會將訊息寫入到磁碟。 兩個都繫結到相同事件中樞執行個體的不同虛擬網路，當中的工作負載可以透過訊息有效且可靠地通訊，並且保留各自的網路隔離界限完整性。
 
這表示您的安全性敏感雲端解決方案，不只獲得 Azure 可靠、可擴充的領先業界非同步傳訊功能的存取權，而且它們現在可以使用傳訊來建立安全解決方案區間之間的通訊路徑，這些區間本質上比任何對等通訊模式 (包括 HTTPS 和其他 TLS 保護的通訊端通訊協定) 更為安全。

## <a name="bind-event-hubs-to-virtual-networks"></a>將事件中樞繫結至虛擬網路

*虛擬網路規則*是防火牆安全性功能，可控制 Azure 事件中樞伺服器是否接受來自特定虛擬網路子網路的連線。

將事件中樞命名空間繫結至虛擬網路是一個雙步驟程序。 您必須先在虛擬網路子網路上建立**虛擬網路服務端點**，然後為 "Microsoft.EventHub" 啟用該端點，如[服務端點概觀][vnet-sep]所述。 一旦您新增服務端點，便會使用*虛擬網路規則*將事件中樞命名空間與其繫結。

虛擬網路規則是事件中樞命名空間與虛擬網路子網路的具名關聯。 此規則存在時，繫結至該子網路的所有工作負載都會獲得事件中樞命名空間的存取權。 事件中樞本身永遠不會建立輸出連線，不需要獲得存取權，因此永遠不會因為啟用這項規則而獲得您子網路的存取權。

### <a name="create-a-virtual-network-rule-with-azure-resource-manager-templates"></a>利用 Azure Resource Manager 範本來建立虛擬網路規則

下列 Resource Manager 範本可讓您將虛擬網路規則新增至現有的事件中樞命名空間。

範本參數：

* **namespaceName**：事件中樞命名空間。
* **vnetRuleName**：要建立之虛擬網路規則的名稱。
* **virtualNetworkingSubnetId**：虛擬網路子網路的完整 Resource Manager 路徑，例如，`subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/default` 適用於虛擬網路的預設子網路。

```json
{  
   "$schema":"http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
   "contentVersion":"1.0.0.0",
   "parameters":{     
          "namespaceName":{  
             "type":"string",
             "metadata":{  
                "description":"Name of the namespace"
             }
          },
          "vnetRuleName":{  
             "type":"string",
             "metadata":{  
                "description":"Name of the Authorization rule"
             }
          },
          "virtualNetworkSubnetId":{  
             "type":"string",
             "metadata":{  
                "description":"subnet Azure Resource Manager ID"
             }
          }
      },
    "resources": [
        {
            "apiVersion": "2018-01-01-preview",
            "name": "[concat(parameters('namespaceName'), '/', parameters('vnetRuleName'))]",
            "type":"Microsoft.EventHub/namespaces/VirtualNetworkRules",         
            "properties": {             
                "virtualNetworkSubnetId": "[parameters('virtualNetworkSubnetId')]"  
            }
        } 
    ]
}
```

若要部署範本，請依照 [Azure Resource Manager][lnk-deploy] 適用的指示執行。

## <a name="next-steps"></a>後續步驟

如需虛擬網路的詳細資訊，請參閱下列連結：

- [Azure 虛擬網路服務端點][vnet-sep]
- [Azure 事件中樞 IP 篩選][ip-filtering] (英文)

[vnet-sep]: ../virtual-network/virtual-network-service-endpoints-overview.md
[lnk-deploy]: ../azure-resource-manager/resource-group-template-deploy.md
[ip-filtering]: event-hubs-ip-filtering.md