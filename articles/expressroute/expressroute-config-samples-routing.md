---
title: ExpressRoute 客戶路由器組態範例 | Microsoft Docs
description: 此頁面提供適用於 Cisco 和 Juniper 路由器的路由器組態範例。
documentationcenter: na
services: expressroute
author: cherylmc
manager: carmonm
editor: ''
ms.assetid: 564826bc-017a-4683-a385-37c9fa814948
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/10/2016
ms.author: cherylmc
ms.openlocfilehash: 032e584dc5abf59e9e3e8d80673b402f1fbf721b
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23013103"
---
# <a name="router-configuration-samples-to-set-up-and-manage-routing"></a>設定和管理路由的路由器組態範例
此頁面提供適用於 Cisco IOS XE 和 Juniper MX 系列路由器的介面和路由組態範例。 這些範例僅可用作指引，不能依原樣使用。 您可以和廠商合作來擬定適合您網路的組態。 

> [!IMPORTANT]
> 此頁面中的範例純粹只用作指引。 您必須和廠商的業務人員 / 技術小組及您的網路團隊合作，來擬定適當的組態以符合您的需求。 Microsoft 將不支援此頁面中所列組態的相關問題。 您必須連絡您的裝置廠商來支援問題。
> 
> 

## <a name="mtu-and-tcp-mss-settings-on-router-interfaces"></a>路由器介面上的 MTU 和 TCP MSS 設定
* ExpressRoute 介面的 MTU 是 1500，這是路由器上乙太網路介面的典型預設 MTU。 除非您的路由器預設有不同的 MTU，否則沒有必要指定路由器介面上的值。
* 不同於 Azure VPN 閘道，ExpressRoute 線路的 TCP MSS 不需要指定。

下列路由器組態範例適用於所有對等互連。 如需路由的詳細資訊，請檢閱 [ExpressRoute 對等互連](expressroute-circuit-peerings.md)和 [ExpressRoute 路由需求](expressroute-routing.md)。


## <a name="cisco-ios-xe-based-routers"></a>Cisco IOS-XE 架構的路由器
本節中的範例適用於任何執行 IOS-XE 作業系統系列的路由器。

### <a name="1-configuring-interfaces-and-sub-interfaces"></a>1.設定介面和子介面
在您連線到 Microsoft 的每個路由器中，針對每個對等互連都需要有一個子介面。 子介面可使用 VLAN ID 或一組堆疊的 VLAN ID 和 IP 位址來識別。

**Dot1Q 介面定義**

此範例會針對含有單一 VLAN ID 的子介面提供子介面定義。 在每個對等互連中 VLAN ID 都是唯一的。 IPv4 位址的最後一個八位元一定是奇數。

    interface GigabitEthernet<Interface_Number>.<Number>
     encapsulation dot1Q <VLAN_ID>
     ip address <IPv4_Address><Subnet_Mask>

**QinQ 介面定義**

此範例會針對含有兩個 VLAN ID 的子介面提供子介面定義。 外部的 VLAN ID (s-tag)，如果使用方式在所有對等互連上維持不變。 在每個對等互連中內部的 VLAN ID (c-tag) 都是唯一的。 IPv4 位址的最後一個八位元一定是奇數。

    interface GigabitEthernet<Interface_Number>.<Number>
     encapsulation dot1Q <s-tag> seconddot1Q <c-tag>
     ip address <IPv4_Address><Subnet_Mask>

### <a name="2-setting-up-ebgp-sessions"></a>2.設定 eBGP 工作階段
您必須針對每個對等互連，設定與 Microsoft 的 BGP 工作階段。 下列範例可讓您設定與 Microsoft 的 BGP 工作階段。 如果您針對子介面使用的 IPv4 位址是 a.b.c.d，則 BGP 芳鄰 (Microsoft) 的 IP 位址會是 a.b.c.d+1。 BGP 芳鄰的 IPv4 位址的最後一個八位元一定是偶數。

    router bgp <Customer_ASN>
     bgp log-neighbor-changes
     neighbor <IP#2_used_by_Azure> remote-as 12076
     !        
     address-family ipv4
     neighbor <IP#2_used_by_Azure> activate
     exit-address-family
    !

### <a name="3-setting-up-prefixes-to-be-advertised-over-the-bgp-session"></a>3.將前置詞設定為要透過 BGP 工作階段進行公告
您可以設定路由器，將選取前置詞公告給 Microsoft。 您可以使用下列範例來執行此動作。

    router bgp <Customer_ASN>
     bgp log-neighbor-changes
     neighbor <IP#2_used_by_Azure> remote-as 12076
     !        
     address-family ipv4
      network <Prefix_to_be_advertised> mask <Subnet_mask>
      neighbor <IP#2_used_by_Azure> activate
     exit-address-family
    !

### <a name="4-route-maps"></a>4.路由對應
您可以使用路由對應和前置詞清單，來篩選要傳播到您網路中的前置詞。 您可以使用下列範例來完成此工作。 確定您已設定適當的前置詞清單。

    router bgp <Customer_ASN>
     bgp log-neighbor-changes
     neighbor <IP#2_used_by_Azure> remote-as 12076
     !        
     address-family ipv4
      network <Prefix_to_be_advertised> mask <Subnet_mask>
      neighbor <IP#2_used_by_Azure> activate
      neighbor <IP#2_used_by_Azure> route-map <MS_Prefixes_Inbound> in
     exit-address-family
    !
    route-map <MS_Prefixes_Inbound> permit 10
     match ip address prefix-list <MS_Prefixes>
    !


## <a name="juniper-mx-series-routers"></a>Juniper MX 系列路由器
本節中的範例適用於所有的 Juniper MX 系列路由器。

### <a name="1-configuring-interfaces-and-sub-interfaces"></a>1.設定介面和子介面

**Dot1Q 介面定義**

此範例會針對含有單一 VLAN ID 的子介面提供子介面定義。 在每個對等互連中 VLAN ID 都是唯一的。 IPv4 位址的最後一個八位元一定是奇數。

    interfaces {
        vlan-tagging;
        <Interface_Number> {
            unit <Number> {
                vlan-id <VLAN_ID>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }
            }
        }
    }


**QinQ 介面定義**

此範例會針對含有兩個 VLAN ID 的子介面提供子介面定義。 外部的 VLAN ID (s-tag)，如果使用方式在所有對等互連上維持不變。 在每個對等互連中內部的 VLAN ID (c-tag) 都是唯一的。 IPv4 位址的最後一個八位元一定是奇數。

    interfaces {
        <Interface_Number> {
            flexible-vlan-tagging;
            unit <Number> {
                vlan-tags outer <S-tag> inner <C-tag>;
                family inet {
                    address <IPv4_Address/Subnet_Mask>;
                }                           
            }                               
        }                                   
    }                           

### <a name="2-setting-up-ebgp-sessions"></a>2.設定 eBGP 工作階段
您必須針對每個對等互連，設定與 Microsoft 的 BGP 工作階段。 下列範例可讓您設定與 Microsoft 的 BGP 工作階段。 如果您針對子介面使用的 IPv4 位址是 a.b.c.d，則 BGP 芳鄰 (Microsoft) 的 IP 位址會是 a.b.c.d+1。 BGP 芳鄰的 IPv4 位址的最後一個八位元一定是偶數。

    routing-options {
        autonomous-system <Customer_ASN>;
    }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }

### <a name="3-setting-up-prefixes-to-be-advertised-over-the-bgp-session"></a>3.將前置詞設定為要透過 BGP 工作階段進行公告
您可以設定路由器，將選取前置詞公告給 Microsoft。 您可以使用下列範例來執行此動作。

    policy-options {
        policy-statement <Policy_Name> {
            term 1 {
                from protocol OSPF;
        route-filter <Prefix_to_be_advertised/Subnet_Mask> exact;
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }


### <a name="4-route-maps"></a>4.路由對應
您可以使用路由對應和前置詞清單，來篩選要傳播到您網路中的前置詞。 您可以使用下列範例來完成此工作。 確定您已設定適當的前置詞清單。

    policy-options {
        prefix-list MS_Prefixes {
            <IP_Prefix_1/Subnet_Mask>;
            <IP_Prefix_2/Subnet_Mask>;
        }
        policy-statement <MS_Prefixes_Inbound> {
            term 1 {
                from {
        prefix-list MS_Prefixes;
                }
                then {
                    accept;
                }
            }
        }
    }
    protocols {
        bgp { 
            group <Group_Name> { 
                export <Policy_Name>
                import <MS_Prefixes_Inbound>
                peer-as 12076;              
                neighbor <IP#2_used_by_Azure>;
            }                               
        }                                   
    }

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱〈 [ExpressRoute 常見問題集](expressroute-faqs.md) 〉。

