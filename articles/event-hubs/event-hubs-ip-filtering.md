---
title: Azure 事件中樞 IP 連線篩選器 | Microsoft Docs
description: 使用 IP 篩選來封鎖從特定 IP 位址連至 Azure 事件中樞的連線。
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.date: 08/26/2018
ms.author: shvija
ms.openlocfilehash: 6d96eac3ecd249de3ba0da82eff95c45e45fa02d
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746190"
---
# <a name="use-ip-filters"></a>使用 IP 篩選器

如果您遇到只能從特定知名網站存取 Azure 事件中樞的情況，可以使用 *IP 篩選器*功能來設定各種規則以拒絕或接受源自特定 IPv4 位址的流量。 例如，這些位址可能是企業 NAT 閘道的位址。

## <a name="when-to-use"></a>使用時機

以下是兩個重要的使用案例，非常適合封鎖特定 IP 位址的事件中樞端點：

- 您的事件中樞只應接收來自指定 IP 位址範圍的流量，並拒絕其他所有流量。 例如，您搭配 [Azure Express Route][express-route] 使用事件中樞，以建立連到內部部署基礎結構的私人連線。 
- 您需要拒絕事件中樞系統管理員認為可疑的 IP 位址所傳來的流量。

## <a name="how-filter-rules-are-applied"></a>篩選器規則的套用方式

IP 篩選器規則會套用在事件中樞命名空間層級。 因此，規則會套用至來自用戶端的所有連接 (使用任何受支援的通訊協定)。

任何來自某一 IP 位址的連接嘗試，只要符合事件中樞命名空間上的拒絕 IP 規則，便視為未經授權而予以拒絕。 回應則不涉及 IP 規則。

## <a name="default-setting"></a>預設設定

根據預設，事件中樞入口網站中的 **IP 篩選器**方格是空的。 這個預設設定表示您的事件中樞會接受來自任何 IP 位址的連線。 這項預設設定等同於可接受 0.0.0.0/0 IP 位址範圍的規則。

## <a name="ip-filter-rule-evaluation"></a>IP 篩選器規則評估

IP 篩選器規則會依序套用，而且第一個符合 IP 位址的規則會決定接受或拒絕動作。

例如，如果您想要接受 70.37.104.0/24 範圍中的位址，並拒絕其他所有位址，則方格中的第一個規則應接受位址範圍 70.37.104.0/24。 下一個規則應使用 0.0.0.0/0 範圍拒絕所有位址。

> [!NOTE]
> 拒絕 IP 位址可防止其他 Azure 服務 (例如 Azure 串流分析、Azure 虛擬機器，或入口網站中的裝置總管) 與事件中樞互動。

### <a name="creating-a-virtual-network-rule-with-azure-resource-manager-templates"></a>利用 Azure Resource Manager 範本來建立虛擬網路規則

下列 Resource Manager 範本可讓您將虛擬網路規則加入至現有的事件中樞命名空間。

範本參數：

- **ipFilterRuleName** 篩選規則名稱必須是唯一的、不區分大小寫的英數字元字串，長度上限為 128 個字元。
- **ipFilterAction** 是 **拒絕**或**接受**為 IP 篩選器規則套用的動作。
- **ipMask** 是單一 IPv4 位址或以 CIDR 標記法表示的 IP 位址區塊。 例如，在 CIDR 標記法中，70.37.104.0/24 表示從 70.37.104.0 開始，到 70.37.104.255 為止，總共 256 個 IPv4 位址，而 24 則表示該範圍內的顯著前置詞位元。

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
          "ipFilterRuleName":{  
             "type":"string",
             "metadata":{  
                "description":"Name of the Authorization rule"
             }
          },
          "ipFilterAction":{  
             "type":"string",
             "allowedValues": ["Reject", "Accept"],
             "metadata":{  
                "description":"IP Filter Action"
             }
          },
          "IpMask":{  
             "type":"string",
             "metadata":{  
                "description":"IP Mask"
             }
          }
      },
    "resources": [
        {
            "apiVersion": "2018-01-01-preview",
            "name": "[concat(parameters('namespaceName'), '/', parameters('ipFilterRuleName'))]",
            "type": "Microsoft.EventHub/Namespaces/IPFilterRules",
            "properties": {
                "FilterName":"[parameters('ipFilterRuleName')]",
                "Action":"[parameters('ipFilterAction')]",              
                "IpMask": "[parameters('IpMask')]"
            }
        } 
    ]
}
```

若要部署範本，請依照 [Azure Resource Manager][lnk-deploy] 適用的指示執行。

## <a name="next-steps"></a>後續步驟

若要將對事件中樞的存取權限制為 Azure 虛擬網路，請參閱下列連結：

- [事件中樞的虛擬網路服務端點][lnk-vnet]

<!-- Links -->

[express-route]:  /azure/expressroute/expressroute-faqs#supported-services
[lnk-deploy]: ../azure-resource-manager/resource-group-template-deploy.md
[lnk-vnet]: event-hubs-service-endpoints.md