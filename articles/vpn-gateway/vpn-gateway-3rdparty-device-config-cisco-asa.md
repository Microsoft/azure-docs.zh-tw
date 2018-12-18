---
title: 將 Cisco ASA 裝置連線到 Azure VPN 閘道的範例設定 | Microsoft Docs
description: 本文提供將 Cisco ASA 裝置連線到 Azure VPN 閘道的範例設定。
services: vpn-gateway
documentationcenter: na
author: yushwang
manager: rossort
editor: ''
tags: ''
ms.assetid: a8bfc955-de49-4172-95ac-5257e262d7ea
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/20/2017
ms.author: yushwang
ms.openlocfilehash: fbe22b70b4fe3463ffc7b0e9a7ebd683f681117d
ms.sourcegitcommit: 2a70752d0987585d480f374c3e2dba0cd5097880
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27910753"
---
# <a name="sample-configuration-cisco-asa-device-ikev2no-bgp"></a>範例組態：Cisco ASA 裝置 (IKEv2/無 BGP)
本文提供將 Cisco Adaptive Security Appliance (ASA) 裝置連線到 Azure VPN 閘道的範例設定。 此範例適用於執行 IKEv2 且不含邊界閘道協定 (BGP) 的 Cisco ASA 裝置。 

## <a name="device-at-a-glance"></a>裝置速覽

|                        |                                   |
| ---                    | ---                               |
| 裝置廠商          | Cisco                             |
| 裝置型號           | ASA                               |
| 目標版本         | 8.4 和更新版本                     |
| 測試的型號           | ASA 5505                          |
| 測試的版本         | 9.2                               |
| IKE 版本            | IKEv2                             |
| BGP                    | 否                                |
| Azure VPN 閘道類型 | 路由式 VPN 閘道           |
|                        |                                   |

> [!NOTE]
> 範例設定會將 Cisco ASA 裝置連線到 Azure **路由式** VPN 閘道。 連線會使用自訂 IPsec/IKE 原則與 **UsePolicyBasedTrafficSelectors** 選項，如[本文](vpn-gateway-connect-multiple-policybased-rm-ps.md)所述。
>
> 範例會要求 ASA 裝置使用 **IKEv2** 原則搭配以存取清單為基礎的設定，而非 VTI 式設定。 請確認 VPN 裝置廠商規格，確認內部部署 VPN 裝置支援 IKEv2 原則。


## <a name="vpn-device-requirements"></a>VPN 裝置需求
Azure VPN 閘道會使用標準的 IPsec/IKE 通訊協定組合來建立站對站 (S2S) VPN 通道。 如需詳細的 IPsec/IKE 通訊協定參數，以及 Azure VPN 閘道的預設密碼編譯演算法，請參閱[關於 VPN 裝置](vpn-gateway-about-vpn-devices.md)。

> [!NOTE]
> 您可以選擇性地針對特定連線指定密碼編譯演算法和金鑰長度的確切組合，如[關於密碼編譯需求](vpn-gateway-about-compliance-crypto.md)中所述。 如果您指定演算法和金鑰長度的確切組合，請確定您會在 VPN 裝置上使用對應的規格。

## <a name="single-vpn-tunnel"></a>單一 VPN 通道
此設定是由 Azure VPN 閘道與您內部部署 VPN 裝置之間的單一 S2S VPN 通道所組成。 您可以選擇[透過 VPN 通道設定 BGP](#bgp)。

![單一 S2S VPN 通道](./media/vpn-gateway-3rdparty-device-config-cisco-asa/singletunnel.png)

如需建置 Azure 設定的詳細逐步指示，請參閱[單一 VPN 通道設定](vpn-gateway-3rdparty-device-config-overview.md#singletunnel)。

### <a name="virtual-network-and-vpn-gateway-information"></a>虛擬網路和 VPN 閘道資訊
本節會列出適用於此範例的參數。

| **參數**                | **值**                    |
| ---                          | ---                          |
| 虛擬網路位址首碼        | 10.11.0.0/16<br>10.12.0.0/16 |
| Azure VPN 閘道 IP         | Azure_Gateway_Public_IP      |
| 內部部署位址首碼 | 10.51.0.0/16<br>10.52.0.0/16 |
| 內部部署 VPN 裝置 IP    | OnPrem_Device_Public_IP     |
| * 虛擬網路 BGP ASN                | 65010                        |
| * Azure BGP 對等 IP           | 10.12.255.30                 |
| * 內部部署 BGP ASN         | 65050                        |
| * 內部部署 BGP 對等 IP     | 10.52.255.254                |
|                              |                              |

\* 僅適用於 BGP 的選擇性參數。

### <a name="ipsecike-policy-and-parameters"></a>IPsec/IKE 原則與參數
下表列出範例中所使用的 IPsec/IKE 演算法與參數。 請參閱您的 VPN 裝置規格，以確認您的 VPN 裝置型號和軔體版本支援的演算法。

| **IPsec/IKEv2**  | **值**                            |
| ---              | ---                                  |
| IKEv2 加密 | AES256                               |
| IKEv2 完整性  | SHA384                               |
| DH 群組         | DHGroup24                            |
| * IPsec 加密 | AES256                               |
| * IPsec 完整性  | SHA1                                 |
| PFS 群組        | PFS24                                |
| QM SA 存留期   | 7,200 秒                         |
| 流量選取器 | UsePolicyBasedTrafficSelectors $True |
| 預先共用金鑰   | PreSharedKey                         |
|                  |                                      |

\* 在某些裝置上，當 IPsec 加密演算法為 AES-GCM 時，IPsec 完整性必須是 null 值。

### <a name="asa-device-support"></a>ASA 裝置支援

* 對於 IKEv2 的支援需要 ASA 8.4 版和更新版本。

* 對於 DH 群組和 PFS 群組支援 (除了群組 5) 的支援需要 ASA 版本 9.x。

* 對於具有 AES-GCM 的 IPsec 加密和具有 SHA-256、SHA-384 或 SHA-512 的 IPsec 完整性的支援，需要 ASA 版本 9.x。 此支援需求適用於較新的 ASA 裝置。 發行時，ASA 模型 5505、5510、5520、5540、5550 和 5580 不支援這些演算法。 請參閱您的 VPN 裝置規格，以確認您的 VPN 裝置型號和軔體版本支援的演算法。


### <a name="sample-device-configuration"></a>範例裝置設定
指令碼提供的範例是根據上一節所述的設定和參數。 S2S VPN 通道組態包含下列組件：

1. 介面和路由
2. 存取清單存取清單
3. IKE 原則和參數 (第 1 階段或主要模式)
4. IPsec 原則和參數 (第 2 階段或快速模式)
5. 其他參數，例如 TCP MSS 固定

> [!IMPORTANT]
> 使用範例指令碼之前，您必須先完成下列步驟。 將指令碼中的預留位置值取代為設定的裝置設定。

* 同時為內部和外部介面指定介面設定。
* 識別適用於您內部/私用和外部/公用網路的路由。
* 確定所有名稱和原則編號在裝置上都是唯一的。
* 確定您的裝置上支援密碼編譯演算法。
* 將下列**預留位置值**取代為設定的實際值：
  - 外部介面名稱：**outside**
  - **Azure_Gateway_Public_IP**
  - **OnPrem_Device_Public_IP**
  - IKE：**Pre_Shared_Key**
  - 虛擬網路與區域網路閘道名稱：**VNetName** 和 **LNGName**
  - 虛擬網路和內部部署網路位址**首碼**
  - 適當的**網路遮罩**

#### <a name="sample-script"></a>範例指令碼

```
! Sample ASA configuration for connecting to Azure VPN gateway
!
! Tested hardware: ASA 5505
! Tested version:  ASA version 9.2(4)
!
! Replace the following place holders with your actual values:
!   - Interface names - default are "outside" and "inside"
!   - <Azure_Gateway_Public_IP>
!   - <OnPrem_Device_Public_IP>
!   - <Pre_Shared_Key>
!   - <VNetName>*
!   - <LNGName>* ==> LocalNetworkGateway - the Azure resource that represents the
!     on-premises network, specifies network prefixes, device public IP, BGP info, etc.
!   - <PrivateIPAddress> ==> Replace it with a private IP address if applicable
!   - <Netmask> ==> Replace it with appropriate netmasks
!   - <Nexthop> ==> Replace it with the actual nexthop IP address
!
! (*) Must be unique names in the device configuration
!
! ==> Interface & route configurations
!
!     > <OnPrem_Device_Public_IP> address on the outside interface or vlan
!     > <PrivateIPAddress> on the inside interface or vlan; e.g., 10.51.0.1/24
!     > Route to connect to <Azure_Gateway_Public_IP> address
!
!     > Example:
!
!       interface Ethernet0/0
!        switchport access vlan 2
!       exit
!
!       interface vlan 1
!        nameif inside
!        security-level 100
!        ip address <PrivateIPAddress> <Netmask>
!       exit
!
!       interface vlan 2
!        nameif outside
!        security-level 0
!        ip address <OnPrem_Device_Public_IP> <Netmask>
!       exit
!
!       route outside 0.0.0.0 0.0.0.0 <NextHop IP> 1
!
! ==> Access lists
!
!     > Most firewall devices deny all traffic by default. Create access lists to
!       (1) Allow S2S VPN tunnels between the ASA and the Azure gateway public IP address
!       (2) Construct traffic selectors as part of IPsec policy or proposal
!
access-list outside_access_in extended permit ip host <Azure_Gateway_Public_IP> host <OnPrem_Device_Public_IP>
!
!     > Object group that consists of all VNet prefixes (e.g., 10.11.0.0/16 &
!       10.12.0.0/16)
!
object-group network Azure-<VNetName>
 description Azure virtual network <VNetName> prefixes
 network-object 10.11.0.0 255.255.0.0
 network-object 10.12.0.0 255.255.0.0
exit
!
!     > Object group that corresponding to the <LNGName> prefixes.
!       E.g., 10.51.0.0/16 and 10.52.0.0/16. Note that LNG = "local network gateway".
!       In Azure network resource, a local network gateway defines the on-premises
!       network properties (address prefixes, VPN device IP, BGP ASN, etc.)
!
object-group network <LNGName>
 description On-Premises network <LNGName> prefixes
 network-object 10.51.0.0 255.255.0.0
 network-object 10.52.0.0 255.255.0.0
exit
!
!     > Specify the access-list between the Azure VNet and your on-premises network.
!       This access list defines the IPsec SA traffic selectors.
!
access-list Azure-<VNetName>-acl extended permit ip object-group <LNGName> object-group Azure-<VNetName>
!
!     > No NAT required between the on-premises network and Azure VNet
!
nat (inside,outside) source static <LNGName> <LNGName> destination static Azure-<VNetName> Azure-<VNetName>
!
! ==> IKEv2 configuration
!
!     > General IKEv2 configuration - enable IKEv2 for VPN
!
group-policy DfltGrpPolicy attributes
 vpn-tunnel-protocol ikev1 ikev2
exit
!
crypto isakmp identity address
crypto ikev2 enable outside
!
!     > Define IKEv2 Phase 1/Main Mode policy
!       - Make sure the policy number is not used
!       - integrity and prf must be the same
!       - DH group 14 and above require ASA version 9.x.
!
crypto ikev2 policy 1
 encryption       aes-256
 integrity        sha384
 prf              sha384
 group            24
 lifetime seconds 86400
exit
!
!     > Set connection type and pre-shared key
!
tunnel-group <Azure_Gateway_Public_IP> type ipsec-l2l
tunnel-group <Azure_Gateway_Public_IP> ipsec-attributes
 ikev2 remote-authentication pre-shared-key <Pre_Shared_Key> 
 ikev2 local-authentication  pre-shared-key <Pre_Shared_Key> 
exit
!
! ==> IPsec configuration
!
!     > IKEv2 Phase 2/Quick Mode proposal
!       - AES-GCM and SHA-2 requires ASA version 9.x on newer ASA models. ASA
!         5505, 5510, 5520, 5540, 5550, 5580 are not supported.
!       - ESP integrity must be null if AES-GCM is configured as ESP encryption
!
crypto ipsec ikev2 ipsec-proposal AES-256
 protocol esp encryption aes-256
 protocol esp integrity  sha-1
exit
!
!     > Set access list & traffic selectors, PFS, IPsec protposal, SA lifetime
!       - This sample uses "Azure-<VNetName>-map" as the crypto map name
!       - ASA supports only one crypto map per interface, if you already have
!         an existing crypto map assigned to your outside interface, you must use
!         the same crypto map name, but with a different sequence number for
!         this policy
!       - "match address" policy uses the access-list "Azure-<VNetName>-acl" defined 
!         previously
!       - "ipsec-proposal" uses the proposal "AES-256" defined previously 
!       - PFS groups 14 and beyond requires ASA version 9.x.
!
crypto map Azure-<VNetName>-map 1 match address Azure-<VNetName>-acl
crypto map Azure-<VNetName>-map 1 set pfs group24
crypto map Azure-<VNetName>-map 1 set peer <Azure_Gateway_Public_IP>
crypto map Azure-<VNetName>-map 1 set ikev2 ipsec-proposal AES-256
crypto map Azure-<VNetName>-map 1 set security-association lifetime seconds 7200
crypto map Azure-<VNetName>-map interface outside
!
! ==> Set TCP MSS to 1350
!
sysopt connection tcpmss 1350
!
```

## <a name="simple-debugging-commands"></a>簡單偵錯命令

針對偵錯用途使用下列 ASA 命令：

* 顯示 IPsec 或 IKE 安全性關聯 (SA)：
    ```
    show crypto ipsec sa
    show crypto ikev2 sa
    ```

* 進入偵錯模式：
    ```
    debug crypto ikev2 platform <level>
    debug crypto ikev2 protocol <level>
    ```
    `debug` 命令可以在主控台上產生顯著的輸出。

* 在裝置上顯示目前的設定：
    ```
    show run
    ```
    使用 `show` 子命令以列出特定部分的裝置設定，例如：
    ```
    show run crypto
    show run access-list
    show run tunnel-group
    ```

## <a name="next-steps"></a>後續步驟
若要設定主動-主動跨單位和 VNet 對 VNet 連線，請參閱[設定主動-主動 VPN 閘道](vpn-gateway-activeactive-rm-powershell.md)。
