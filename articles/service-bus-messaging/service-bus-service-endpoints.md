---
title: Azure 服務匯流排的虛擬網路服務端點和規則 | Microsoft Docs
description: 將 Microsoft.ServiceBus 服務端點新增至虛擬網路。
services: event-hubs
documentationcenter: ''
author: clemensv
manager: timlt
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.date: 09/05/2018
ms.author: clemensv
ms.openlocfilehash: 05930dfce64378d792213ccaefa3d15057bd5dfd
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47404992"
---
# <a name="use-virtual-network-service-endpoints-with-azure-service-bus"></a>搭配 Azure 服務匯流排使用虛擬網路服務端點

服務匯流排與[虛擬網路 (VNet) 服務端點][vnet-sep]的整合可以安全存取來自虛擬機器 (繫結至虛擬網路) 工作負載的傳訊功能，兩端的網路流量路徑都受到保護。 

一旦設定為繫結到至少一個虛擬網路子網路服務端點，個別的服務匯流排命名空間除了授權的虛擬網路以外，無法再接受任何位置的流量。 從虛擬網路的觀點而言，將服務匯流排命名空間繫結至服務端點，會設定從虛擬網路子網路到傳訊服務的隔離網路通道。

結果是繫結至子網路的工作負載與個別服務匯流排命名空間之間的私人和隔離關係，儘管傳訊服務端點的可觀察網路位址是在公用 IP 範圍中。

## <a name="enable-service-endpoints-with-service-bus"></a>以服務匯流排啟用服務端點

唯有[進階層](service-bus-premium-messaging.md)服務匯流排命名空間支援虛擬網路。 

請務必留意，使用 VNet 服務端點搭配服務匯流排時，應避免在混合了標準層和進階層服務匯流排命名空間的應用程式中啟用這些端點。 因為標準層不支援 VNet，所以端點僅限於進階層命名空間。 VNet 會封鎖前往標準命名空間的流量。 

## <a name="advanced-security-scenarios-enabled-by-vnet-integration"></a>VNet 整合所實現的進階安全性案例 

需要緊密和劃分安全性，而且虛擬網路子網路分割各劃分服務的解決方案，通常仍然需要位於這些區間之各項服務之間的通訊路徑。

區間之間的任何立即 IP 路由，包括那些運輸 HTTPS over TCP/IP，會帶來上層網路的弱點危害風險。 傳訊服務提供完全隔離的通訊路徑，在其中，當訊息在對象之間轉換時，甚至會寫入到磁碟。 兩個都繫結到相同服務匯流排執行個體的不同虛擬網路，其中的工作負載可以透過訊息有效且可靠地通訊，同時保留各自的網路隔離界限完整性。
 
這表示您的安全性敏感雲端解決方案，不只獲得了 Azure 領先業界可靠和可擴充非同步傳訊功能支援，且現在可以使用傳訊來建立安全解決方案區間之間的通訊路徑，本質上會比任何對等通訊模式 (包括 HTTPS 和其他受 TLS 保護的通訊端通訊協定) 所能達成的更為安全。

## <a name="binding-service-bus-to-virtual-networks"></a>將服務匯流排繫結至虛擬網路

「虛擬網路規則」是防火牆安全性功能，可控制 Azure 服務匯流排伺服器是否接受來自特定虛擬網路子網路的連線。

將服務匯流排命名空間繫結至虛擬網路是一個雙步驟的程序。 您必須先在虛擬網路子網路上建立**虛擬網路服務端點**，然後針對 "Microsoft.ServiceBus" 啟用它，如同[服務端點概觀][vnet-sep]中所述。 一旦您新增服務端點，您就可使用「虛擬網路規則」將服務匯流排命名空間與它繫結。

虛擬網路規則是服務匯流排命名空間與虛擬網路子網路的具名關聯。 當此規則存在時，繫結至子網路的所有工作負載都會獲得授與服務匯流排命名空間的存取權。 服務匯流排本身永遠不會建立輸出連線，不需要獲得存取權，因此永遠不會因為啟用這項規則而獲得授與子網路的存取權。

### <a name="creating-a-virtual-network-rule-with-azure-resource-manager-templates"></a>利用 Azure Resource Manager 範本來建立虛擬網路規則

下列 Resource Manager 範本可讓您將虛擬網路規則新增至現有的服務匯流排命名空間。

範本參數：

* **namespaceName**：服務匯流排命名空間。
* **vnetRuleName**：要建立的虛擬網路規則名稱。
* **virtualNetworkingSubnetId**：虛擬網路子網路的完整 Resource Manager 路徑，例如，`/subscriptions/{id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/default` 適用於虛擬網路的預設子網路。

範本：

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
            "type":"Microsoft.ServiceBus/namespaces/VirtualNetworkRules",           
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
- [Azure 服務匯流排 IP 篩選][ip-filtering]

[vnet-sep]: ../virtual-network/virtual-network-service-endpoints-overview.md
[lnk-deploy]: ../azure-resource-manager/resource-group-template-deploy.md
[ip-filtering]: service-bus-ip-filtering.md