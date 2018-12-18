---
title: ExpressRoute 客戶路由器組態範例 | Microsoft Docs
description: 此頁面提供適用於 Cisco 和 Juniper 路由器的路由器組態範例。
documentationcenter: na
services: expressroute
author: cherylmc
manager: carmonm
editor: ''
ms.assetid: d6ea716f-d5ee-4a61-92b0-640d6e7d6974
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/10/2016
ms.author: cherylmc
ms.openlocfilehash: 83a7da2db537a3c900e90432455d59e8ac56d917
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23012973"
---
# <a name="router-configuration-samples-to-set-up-and-manage-nat"></a>設定和管理 NAT 的路由器組態範例
此頁面提供適用於 Cisco ASA 和 Juniper SRX 系列路由器的 NAT 組態範例。 這些範例僅可用作指引，不能依原樣使用。 您可以和廠商合作來擬定適合您網路的組態。 

> [!IMPORTANT]
> 此頁面中的範例純粹只用作指引。 您必須和廠商的業務人員 / 技術小組及您的網路團隊合作，來擬定適當的組態以符合您的需求。 Microsoft 將不支援此頁面中所列組態的相關問題。 您必須連絡您的裝置廠商來支援問題。
> 
> 

* 下列路由器組態範例適用於 Azure Public 與 Microsoft 對等互連。 您必須設定 Azure 私人對等互連的 NAT。 如需詳細資訊，請檢閱 [ExpressRoute 對等互連](expressroute-circuit-peerings.md)和 [ExpressRoute NAT 需求](expressroute-nat.md)。

* 您必須使用個別的 NAT IP 集區來連線至網際網路和 ExpressRoute。 在網際網路與 ExpressRoute 中使用相同的 NAT IP 集區，將會導致非對稱路由和連線中斷。


## <a name="cisco-asa-firewalls"></a>Cisco ASA 防火牆
### <a name="pat-configuration-for-traffic-from-customer-network-to-microsoft"></a>適用於從客戶網路至 Microsoft 之流量的 PAT 組態
    object network MSFT-PAT
      range <SNAT-START-IP> <SNAT-END-IP>


    object-group network MSFT-Range
      network-object <IP> <Subnet_Mask>

    object-group network on-prem-range-1
      network-object <IP> <Subnet-Mask>

    object-group network on-prem-range-2
      network-object <IP> <Subnet-Mask>

    object-group network on-prem
      network-object object on-prem-range-1
      network-object object on-prem-range-2

    nat (outside,inside) source dynamic on-prem pat-pool MSFT-PAT destination static MSFT-Range MSFT-Range

### <a name="pat-configuration-for-traffic-from-microsoft-to-customer-network"></a>適用於從 Microsoft 至客戶網路之流量的 PAT 組態

**介面和方向：**

    Source Interface (where the traffic enters the ASA): inside
    Destination Interface (where the traffic exits the ASA): outside

**組態：**

NAT 集區：

    object network outbound-PAT
        host <NAT-IP>

目標伺服器：

    object network Customer-Network
        network-object <IP> <Subnet-Mask>

客戶 IP 位址的物件群組

    object-group network MSFT-Network-1
        network-object <MSFT-IP> <Subnet-Mask>

    object-group network MSFT-PAT-Networks
        network-object object MSFT-Network-1

NAT 命令：

    nat (inside,outside) source dynamic MSFT-PAT-Networks pat-pool outbound-PAT destination static Customer-Network Customer-Network


## <a name="juniper-srx-series-routers"></a>Juniper SRX 系列路由器
### <a name="1-create-redundant-ethernet-interfaces-for-the-cluster"></a>1.建立叢集的備援乙太網路介面
    interfaces {
        reth0 {
            description "To Internal Network";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 1;
            }
            unit 100 {
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
        reth1 {
            description "To Microsoft via Edge Router";
            vlan-tagging;
            redundant-ether-options {
                redundancy-group 2;
            }
            unit 100 {
                description "To Microsoft via Edge Router";
                vlan-id 100;
                family inet {
                    address <IP-Address/Subnet-mask>;
                }
            }
        }
    }


### <a name="2-create-two-security-zones"></a>2.建立兩個安全性區域
* 內部網路的信任區域和外部網路面向邊緣路由器的未受信任區域
* 將適當的介面指派給區域
* 在介面上允許一些服務

    security {       zones {           security-zone Trust {               host-inbound-traffic {                   system-services {                       ping;                   }                   protocols {                       bgp;                   }               }               interfaces {                   reth0.100;               }           }           security-zone Untrust {               host-inbound-traffic {                   system-services {                       ping;                   }                   protocols {                       bgp;                   }               }               interfaces {                   reth1.100;               }           }       }   }


### <a name="3-create-security-policies-between-zones"></a>3.建立區域之間的安全性原則
    security {
        policies {
            from-zone Trust to-zone Untrust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
            from-zone Untrust to-zone Trust {
                policy allow-any {
                    match {
                        source-address any;
                        destination-address any;
                        application any;
                    }
                    then {
                        permit;
                    }
                }
            }
        }
    }


### <a name="4-configure-nat-policies"></a>4.設定 NAT 原則
* 建立兩個 NAT 集區。 一個集區將用於輸出到 Microsoft 的 NAT 流量，另一個集區則用於從 Microsoft 至客戶的 NAT 流量。
* 建立各自流量的 NAT 規則
  
       security {
           nat {
               source {
                   pool SNAT-To-ExpressRoute {
                       routing-instance {
                           External-ExpressRoute;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   pool SNAT-From-ExpressRoute {
                       routing-instance {
                           Internal;
                       }
                       address {
                           <NAT-IP-address/Subnet-mask>;
                       }
                   }
                   rule-set Outbound_NAT {
                       from routing-instance Internal;
                       to routing-instance External-ExpressRoute;
                       rule SNAT-Out {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-To-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
                   rule-set Inbound-NAT {
                       from routing-instance External-ExpressRoute;
                       to routing-instance Internal;
                       rule SNAT-In {
                           match {
                               source-address 0.0.0.0/0;
                           }
                           then {
                               source-nat {
                                   pool {
                                       SNAT-From-ExpressRoute;
                                   }
                               }
                           }
                       }
                   }
               }
           }
       }

### <a name="5-configure-bgp-to-advertise-selective-prefixes-in-each-direction"></a>5.設定 BGP 以通告每個方向的選擇性前置詞
請參考 [路由組態範例 ](expressroute-config-samples-routing.md) 頁面中的範例。

### <a name="6-create-policies"></a>6.建立原則
    routing-options {
                  autonomous-system <Customer-ASN>;
    }
    policy-options {
        prefix-list Microsoft-Prefixes {
            <IP-Address/Subnet-Mask;
            <IP-Address/Subnet-Mask;
        }
        prefix-list private-ranges {
            10.0.0.0/8;
            172.16.0.0/12;
            192.168.0.0/16;
            100.64.0.0/10;
        }
        policy-statement Advertise-NAT-Pools {
            from {
                protocol static;
                route-filter <NAT-Pool-Address/Subnet-mask> prefix-length-range /32-/32;
            }
            then accept;
        }
        policy-statement Accept-from-Microsoft {
            term 1 {
                from {
                    instance External-ExpressRoute;
                    prefix-list-filter Microsoft-Prefixes orlonger;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
        policy-statement Accept-from-Internal {
            term no-private {
                from {
                    instance Internal;
                    prefix-list-filter private-ranges orlonger;
                }
                then reject;
            }
            term bgp {
                from {
                    instance Internal;
                    protocol bgp;
                }
                then accept;
            }
            term deny {
                then reject;
            }
        }
    }
    routing-instances {
        Internal {
            instance-type virtual-router;
            interface reth0.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Microsoft;
            }
            protocols {
                bgp {
                    group customer {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-ASN-1>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
        External-ExpressRoute {
            instance-type virtual-router;
            interface reth1.100;
            routing-options {
                static {
                    route <NAT-Pool-IP-Address/Subnet-mask> discard;
                }
                instance-import Accept-from-Internal;
            }
            protocols {
                bgp {
                    group edge-router {
                        export <Advertise-NAT-Pools>;
                        peer-as <Customer-Public-ASN>;
                        neighbor <BGP-Neighbor-IP-Address>;
                    }
                }
            }
        }
    }

## <a name="next-steps"></a>後續步驟
如需詳細資訊，請參閱〈 [ExpressRoute 常見問題集](expressroute-faqs.md) 〉。

