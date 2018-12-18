---
title: 使用 ExpressRoute 將 Azure Stack 連線至 Azure
description: 了解如何使用 ExpressRoute 將 Azure Stack 中的虛擬網路連線至 Azure 中的虛擬網路。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: c30e70802d125744432f428f903f6ac6789f631e
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389220"
---
# <a name="connect-azure-stack-to-azure-using-azure-expressroute"></a>使用 Azure ExpressRoute 將 Azure Stack 連線至 Azure

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

本文說明如何使用 Microsoft Azure ExpressRoute 直接連接，將 Azure Stack 虛擬網路連接到 Azure 虛擬網路。

您可以使用本文作為教學課程，並使用範例來設定相同的測試環境。 或者，您也可以使用本文作為逐步指示，來引導您完成 ExpressRoute 環境的設定。

## <a name="overview-assumptions-and-prerequisites"></a>概觀、假設和先決條件

Azure ExpressRoute 可讓您透過連線提供者所提供的私人連線，將內部部署網路延伸至 Microsoft 雲端。 ExpressRoute 不是透過公用網際網路的 VPN 連線。

如需 Azure ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 概觀](../expressroute/expressroute-introduction.md)。

### <a name="assumptions"></a>假設

本文假設：

* 您具備 Azure 的使用知識。
* 您對 Azure stack 有基本的了解。
* 您對網路有基本的了解。

### <a name="prerequisites"></a>必要條件

若要使用 ExpressRoute 連接 Azure Stack 和 Azure，您必須符合下列需求：

* 透過[連線提供者](../expressroute/expressroute-locations.md)佈建的 [ExpressRoute 線路](../expressroute/expressroute-circuit-peerings.md)。
* Azure 訂用帳戶，用於 Azure 中建立 ExpressRoute 線路和 VNet。
* 擁有路由器，且其必須：
  * 在其 LAN 介面和 Azure Stack 多租用戶閘道之間，支援站對站 VPN 連線。
  * 支援建立多個 VRF (虛擬路由和轉送) (如果 Azure Stack 部署中有多個租用戶)。
* 擁有路由器，且其具有：
  * 連線至 ExpressRoute 線路的 WAN 連接埠。
  * 連線至 Azure Stack 多租用戶閘道的 LAN 連接埠。

### <a name="expressroute-network-architecture"></a>ExpressRoute 網路架構

下圖說明使用本文中的範例完成 ExpressRoute 設定之後的 Azure Stack 和 Azure 環境。

*圖 1.ExpressRoute 網路*

![ExpressRoute 網路](media/azure-stack-connect-expressroute/Conceptual.png)

下一個架構圖說明多個租用戶如何從 Azure Stack 基礎結構，透過 ExpressRoute 路由器，連線至 Microsoft Edge 的 Azure。

*圖 2.多租用戶連線*

![使用 ExpressRoute 的多租用戶連線](media/azure-stack-connect-expressroute/Architecture.png)

本文範例會使用圖 2 所示的相同多租用戶架構，利用 ExpressRoute 私人對等互連將 Azure Stack 連線至 Azure。 在作法上是使用站對站 VPN 連線，從 Azure Stack 中的虛擬網路閘道連線至 ExpressRoute 路由器。

本文中的步驟說明如何在兩個 VNet 之間建立端對端連線：從 Azure Stack 中的不同租用戶連到 Azure 中的對應 VNet。 設定兩個租用戶是選擇性作業，您也可以使用這些步驟來設定單一租用戶。

## <a name="configure-azure-stack"></a>設定 Azure Stack

若要為第一個租用戶設定 Azure Stack 環境，請使用下圖中的步驟作為指南。 如果您要設定多個租用戶，請重複這些步驟。

>[!NOTE]
>以下步驟說明如何使用 Azure Stack 入口網站來建立資源，但您也可以使用 PowerShell。

![Azure Stack 網路設定](media/azure-stack-connect-expressroute/image2.png)

### <a name="before-you-begin"></a>開始之前

開始設定 Azure Stack 之前，您需要：

* Azure Stack 整合系統部署，或 Azure Stack 開發套件 (ASDK) 部署。 如需有關部署 ASDK 的資訊，請參閱 [Azure Stack 開發套件部署快速入門](azure-stack-deploy-overview.md)。
* Azure Stack 中可供使用者訂閱的供應項目。 如需詳細資訊，請參閱[方案、供應項目和訂用帳戶](azure-stack-plan-offer-quota-overview.md)。

### <a name="create-network-resources-in-azure-stack"></a>在 Azure Stack 中建立網路資源

請使用下列程序在 Azure Stack 中建立租用戶所需的網路資源。

#### <a name="create-the-virtual-network-and-vm-subnet"></a>建立虛擬網路和 VM 子網路

1. 以使用者 (租用戶) 帳戶來登入使用者入口網站。
1. 在入口網站中，選取 [+ 建立資源]。

1. 在 [Azure Marketplace] 底下，選取 [網路]。

1. 在 [精選] 底下，選取 [虛擬網路]。

1. 在 [建立虛擬網路] 底下，於適當欄位中輸入下表所顯示的值。

   |欄位  |值  |
   |---------|---------|
   |名稱     |Tenant1VNet1         |
   |位址空間     |10.1.0.0/16|
   |子網路名稱     |Tenant1-Sub1|
   |子網路位址範圍     |10.1.1.0/24|

1. 您應會看到稍早建立的訂用帳戶填入 [訂用帳戶] 欄位中。 至於其餘欄位：

    * 在 [資源群組] 底下，選取 [新建] 以建立新的資源群組，如果您已有資源群組，則選取 [使用現有的]。
    * 確認預設 [位置]。
    * 選取 [建立] 。
    * (選擇性) 選取 [釘選到儀表板]。

#### <a name="create-the-gateway-subnet"></a>建立閘道子網路

1. 在 [虛擬網路] 底下，選取 [Tenant1VNet1]。
1. 在 [設定] 底下，選取 [子網路]。
1. 選取 [+ 閘道子網路] 以將閘道子網路新增到虛擬網路。
1. 子網路名稱預設為 **GatewaySubnet**。 閘道子網路屬特殊案例，其必須使用此名稱才能正常運作。
1. 確認 [位址範圍] 是 **10.1.0.0/24**。
1. 選取 [確定] 以建立閘道子網路。

#### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道

1. 在 Azure Stack 使用者入口網站中，選取 [+ 建立資源]。
1. 在 [Azure Marketplace] 底下，選取 [網路]。
1. 從網路資源清單中選取 [虛擬網路閘道]。
1. 在 [名稱] 欄位中，輸入 **GW1**。
1. 選取 [虛擬網路]。
1. 從下拉式清單中選取 [Tenant1VNet1]。
1. 選取 [公用 IP 位址] > [選擇公用 IP 位址]，然後選取 [新建]。
1. 在 [名稱] 欄位中輸入 **GW1-PiP**，然後選取 [確定]。
1. [VPN 類型] 預設應選取 [路由式]。 保留此設定。
1. 確認 [訂用帳戶] 和 [位置] 均正確無誤。 選取 [建立] 。

#### <a name="create-the-local-network-gateway"></a>建立區域網路閘道

區域網路閘道資源可識別 VPN 連線另一端的遠端閘道。 在此範例中，連線的遠端是 ExpressRoute 路由器的 LAN 子介面。 圖 2 所示的租用戶 1 ，其遠端位址是 10.60.3.255。

1. 以使用者帳戶登入 Azure Stack 使用者入口網站，然後選取 [+ 建立資源]。
1. 在 [Azure Marketplace] 底下，選取 [網路]。
1. 從資源清單中選取 [區域網路閘道]。
1. 在 [名稱] 欄位中輸入 **ER-Router-GW**。
1. 關於 [IP 位址] 欄位，請參閱圖 2。 對於租用戶 1，ExpressRoute 路由器的 LAN 子介面 IP 位址是 10.60.3.255。 根據您自己的環境，輸入路由器相對應介面的 IP 位址。
1. 在 [位址空間] 欄位中，輸入您想要連線至 Azure 中之 VNet 的位址空間。 圖 2 中租用戶 1 的子網路是：

   * 192.168.2.0/24 是 Azure 中的中樞 VNet。
   * 10.100.0.0/16 是 Azure 中的輪輻 VNet。

   > [!IMPORTANT]
   > 對於 Azure Stack 閘道和 ExpressRoute 路由器之間的站對站 VPN 連線，這個範例假設您使用靜態路由。

1. 確認 [訂用帳戶]、[資源群組] 和 [位置] 正確無誤。 選取 [建立] 。

#### <a name="create-the-connection"></a>建立連線

1. 在 Azure Stack 使用者入口網站中，選取 [+ 建立資源]。
1. 在 [Azure Marketplace] 底下，選取 [網路]。
1. 從資源清單中選取 [連線]。
1. 在 [基本] 底下，選擇 [站對站 (IPSec)] 作為 [連線類型]。
1. 選取 [訂用帳戶]、[資源群組] 和 [位置]。 選取 [確定] 。
1. 在 [設定] 底下，選取 [虛擬網路閘道]，然後選取 [GW1]。
1. 選取 [區域網路閘道]，然後選取 [ER Router GW]。
1. 在 [連線名稱] 欄位中，輸入 **ConnectToAzure**。
1. 在 [共用金鑰 (PSK)] 欄位中輸入 **abc123**，然後選取 [確定]。
1. 在 [摘要] 底下，選取 [確定]。

**取得虛擬網路閘道公用 IP 位址**

建立虛擬網路閘道之後，您可以取得閘道的公用 IP 位址。 請記下此位址，以免稍後需要在部署中用到。 根據您的部署，此位址可作為 [內部 IP 位址]。

1. 在 Azure Stack 使用者入口網站中，選取 [所有資源]。
1. 在 [所有資源] 底下選取虛擬網路閘道，在此範例中為 **GW1**。
1. 在 [虛擬網路閘道] 下方，從資源清單中選取 [概觀]。 或者，您也可以選取 [屬性]。
1. 您要記下的 IP 位址列在 [公用 IP 位址] 底下。 在設定範例中，此位址是 192.68.102.1。

#### <a name="create-a-virtual-machine"></a>建立虛擬機器

若要測試透過 VPN 連線傳輸的資料流量，您需要有虛擬機器以在 Azure Stack Vnet 中傳送和接收資料。 建立虛擬機器，並將它部署到虛擬網路的 VM 子網路。

1. 在 Azure Stack 使用者入口網站中，選取 [+ 建立資源]。
1. 在 [Azure Marketplace] 底下，選取 [計算]。
1. 在虛擬機器映像清單中，選取 [Windows Server 2016 Datacenter 評估版] 映像。

   >[!NOTE]
   >如果您無法取得本文所使用的映像，請要求 Azure Stack 操作員提供不同的 Windows Server 映像。

1. 在 [建立虛擬機器] > [基本資料] 中，輸入 **VM01** 作為 [名稱]。
1. 輸入有效的使用者名稱和密碼。 建立 VM 之後，您將使用此帳戶來登入 VM。
1. 提供 [訂用帳戶]、[資源群組] 和 [位置]。 選取 [確定] 。
1. 在 [選擇大小] 底下，選取此執行個體的虛擬機器大小，然後選取 [選取]。
1. 在 [設定] 底下，確認：

   * 虛擬網路是 **Tenant1VNet1**。
   * 子網路已設定為 **10.1.1.0/24**。

   使用預設設定，然後選取 [確定]。

1. 在 [摘要] 底下檢閱 VM 設定，然後選取 [確定]。

>[!NOTE]
>
>若要新增更多租用戶，請重複您在下列章節中所遵循的步驟：
>
>* 建立虛擬網路和 VM 子網路
>* 建立閘道子網路
>* 建立虛擬網路閘道
>* 建立區域網路閘道
>* 建立連線
>* 建立虛擬機器
>
>如果您要使用租用戶 2 作為範例，請記得變更 IP 位址以避免位址重疊。

### <a name="configure-the-nat-virtual-machine-for-gateway-traversal"></a>設定用於閘道周遊的 NAT 虛擬機器

> [!IMPORTANT]
> 本節僅適用於 Azure Stack 開發套件部署。 多重節點部署不需要 NAT。

Azure Stack 開發套件是獨立的，而且與部署實體主機的網路隔離。 閘道所連線的 VIP 網路不是在外部，而是隱藏在執行網路位址轉譯 (NAT) 的路由器背後。

路由器是 Windows Server 虛擬機器 (AzS-BGPNAT01)，扮演「路由及遠端存取服務」(RRAS) 角色。 您必須在 AzS-BGPNAT01 虛擬機器上設定 NAT，以便啟用站對站 VPN 連線來連接兩端。

#### <a name="configure-the-nat"></a>設定 NAT

1. 以系統管理員帳戶登入 Azure Stack 主機電腦。
1. 複製並編輯下列 PowerShell 指令碼。  將 `"<your administrator password>"` 更換為系統管理員密碼，然後在提升權限的 PowerShell ISE 中執行指令碼。 此指令碼會傳回「外部 BGPNAT 位址」。

   ```PowerShell
   cd \AzureStack-Tools-master\connect
   Import-Module .\AzureStack.Connect.psm1
   $Password = ConvertTo-SecureString "<your administrator password>" `
    -AsPlainText `
    -Force
   Get-AzureStackNatServerAddress `
    -HostComputer "azs-bgpnat01" `
    -Password $Password

   ```

1. 若要設定 NAT，請複製並編輯下列 PowerShell 指令碼。 編輯指令碼以將 `'<External BGPNAT address>'` 和 `'<Internal IP address>'` 更換為下列值範例：

   * 針對 [外部 BGPNAT 位址]，請使用 10.10.0.62
   * 針對 [內部 IP 位址]，請使用 192.168.102.1

   從提高權限的 PowerShell ISE 執行下列指令碼：

   ```PowerShell
   $ExtBgpNat = '<External BGPNAT address>'
   $IntBgpNat = '<Internal IP address>'

   # Designate the external NAT address for the ports that use the IKE authentication.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress $Using:ExtBgpNat `
      -PortStart 499 `
      -PortEnd 501}
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatExternalAddress `
      -NatName BGPNAT `
      -IPAddress $Using:ExtBgpNat `
      -PortStart 4499 `
      -PortEnd 4501}
   # Create a static NAT mapping to map the external address to the Gateway public IP address to map the ISAKMP port 500 for PHASE 1 of the IPSEC tunnel.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress $Using:ExtBgpNat `
      -InternalIPAddress $Using:IntBgpNat `
      -ExternalPort 500 `
      -InternalPort 500}
   # Configure NAT traversal which uses port 4500 to  establish the complete IPSEC tunnel over NAT devices.
   Invoke-Command `
    -ComputerName azs-bgpnat01 `
     {Add-NetNatStaticMapping `
      -NatName BGPNAT `
      -Protocol UDP `
      -ExternalIPAddress $Using:ExtBgpNat `
      -InternalIPAddress $Using:IntBgpNat `
      -ExternalPort 4500 `
      -InternalPort 4500}

   ```

## <a name="configure-azure"></a>設定 Azure

完成 Azure Stack 的設定之後，您就可以部署 Azure 資源。 下圖說明 Azure 中的租用戶虛擬網路範例。 針對您在 Azure 中的 VNet，您可以使用任何名稱和定址配置。 不過，在 Azure 和 Azure Stack 中，VNet 的位址範圍必須是唯一的，而且不重疊。

*圖 3.Azure VNet*

![Azure VNet](media/azure-stack-connect-expressroute/AzureArchitecture.png)

您在 Azure 中部署的資源，類似於您在 Azure Stack 中部署的資源。 您會部署下列元件：

* 虛擬網路和子網路
* 閘道子網路
* 虛擬網路閘道
* 連線
* ExpressRoute 線路

Azure 網路基礎結構範例的設定方式如下：

* 標準中樞 (192.168.2.0/24) 和輪輻 (10.100.0.0./16) VNet 模型。 如需中樞輪輻網路拓撲的詳細資訊，請參閱[在 Azure 中實作中樞輪輻網路拓撲](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)。
* 工作負載部署在輪輻 VNet 中，而 ExpressRoute 線路連線至中樞 VNet。
* 使用 VNet 對等互連將兩個 VNet 連線。

### <a name="configure-the-azure-vnets"></a>設定 Azure VNet

1. 使用您的 Azure 認證登入 Azure 入口網站。
1. 使用 192.168.2.0/24 位址範圍建立中樞 VNet。
1. 使用 192.168.2.0/25 位址範圍建立子網路，並使用 192.168.2.128/27 位址範圍新增閘道子網路。
1. 使用 10.100.0.0/16 位址範圍建立輪輻 VNet 和子網路。

如需有關在 Azure 中建立虛擬網路的詳細資訊，請參閱[建立虛擬網路](../virtual-network/manage-virtual-network.md#create-a-virtual-network)。

### <a name="configure-an-expressroute-circuit"></a>設定 ExpressRoute 線路

1. 檢閱 [ExpressRoute 必要條件和檢查清單](../expressroute/expressroute-prerequisites.md)中的 ExpressRoute 必要條件。

1. 請依照[建立和修改 ExpressRoute 線路](../expressroute/expressroute-howto-circuit-portal-resource-manager.md)中的步驟，使用您的 Azure 訂用帳戶建立 ExpressRoute 線路。

   >[!NOTE]
   >將線路的服務金鑰提供給服務，以便服務可以在它們那一端設定 ExpressRoute 線路。

1. 請依照[建立和修改 ExpressRoute 線路的對等互連](../expressroute/expressroute-howto-routing-portal-resource-manager.md)中的步驟，在 ExpressRoute 線路上設定私用對等互連。

### <a name="create-the-virtual-network-gateway"></a>建立虛擬網路閘道

請依照[使用 PowerShell 為 ExpressRoute 設定虛擬網路閘道](../expressroute/expressroute-howto-add-gateway-resource-manager.md)中的步驟，在中樞 VNet 中為 ExpressRoute 建立虛擬網路閘道。

### <a name="create-the-connection"></a>建立連線

若要將 ExpressRoute 線路連結至中樞 VNet，請依照[將虛擬網路連線至 ExpressRoute 線路](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)中的步驟。

### <a name="peer-the-vnets"></a>將 VNet 對等互連

依照[使用 Azure 入口網站建立虛擬網路對等互連](../virtual-network/virtual-networks-create-vnetpeering-arm-portal.md)中的步驟，將中樞和輪輻 VNet 對等互連。 設定 VNet 對等互連時，務必使用下列選項：

* 從中樞至輪輻：**允許閘道傳輸**。
* 從輪輻至中樞：**使用遠端閘道**。

### <a name="create-a-virtual-machine"></a>建立虛擬機器

將工作負載虛擬機器部署至輪輻 VNet。

對於您在 Azure 中想要透過各自 ExpressRoute 線路來連線的其他任何租用戶 VNet，重複這些步驟。

## <a name="configure-the-router"></a>設定路由器

您可以使用下列「ExpressRoute 路由器設定」圖表作為 ExpressRoute 路由器的設定指南。 這個圖表顯示兩個租用戶 (租用戶 1 和租用戶 2) 有各自的 ExpressRoute 線路。 每個租用戶都連結至各自在 ExpressRoute 路由器 LAN 和 WAN 端的 VRF (虛擬路由和轉送)。 此設定可確保兩個租用戶之間的端對端隔離。 隨著您進行設定範例，請記下路由器介面中使用的 IP 位址。

*圖 4：ExpressRoute 路由器設定*

![ExpressRoute 路由器設定](media/azure-stack-connect-expressroute/EndToEnd.png)

您可以使用任何支援 IKEv2 VPN 和 BGP 的路由器，以終止 Azure Stack 的站對站 VPN 連線。 相同的路由器用於透過 ExpressRoute 線路來連線至 Azure。

下列 Cisco ASR 1000 Series Aggregation Services Router 設定範例可支援「ExpressRoute 路由器設定」圖表顯示的網路基礎結構。

**Cisco ASR 1000 設定範例**

```
ip vrf Tenant 1
 description Routing Domain for PRIVATE peering to Azure for Tenant 1
 rd 1:1
!
ip vrf Tenant 2
 description Routing Domain for PRIVATE peering to Azure for Tenant 2
 rd 1:5
!
crypto ikev2 proposal V2-PROPOSAL2
description IKEv2 proposal for Tenant 1
encryption aes-cbc-256
 integrity sha256
 group 2
crypto ikev2 proposal V4-PROPOSAL2
description IKEv2 proposal for Tenant 2
encryption aes-cbc-256
 integrity sha256
 group 2
!
crypto ikev2 policy V2-POLICY2
description IKEv2 Policy for Tenant 1
match fvrf Tenant 1
 match address local 10.60.3.255
 proposal V2-PROPOSAL2
description IKEv2 Policy for Tenant 2
crypto ikev2 policy V4-POLICY2
 match fvrf Tenant 2
 match address local 10.60.3.251
 proposal V4-PROPOSAL2
!
crypto ikev2 profile V2-PROFILE
description IKEv2 profile for Tenant 1
match fvrf Tenant 1
 match address local 10.60.3.255
 match identity remote any
 authentication remote pre-share key abc123
 authentication local pre-share key abc123
 ivrf Tenant 1
!
crypto ikev2 profile V4-PROFILE
description IKEv2 profile for Tenant 2
 match fvrf Tenant 2
 match address local 10.60.3.251
 match identity remote any
 authentication remote pre-share key abc123
 authentication local pre-share key abc123
 ivrf Tenant 2
!
crypto ipsec transform-set V2-TRANSFORM2 esp-gcm 256
 mode tunnel
crypto ipsec transform-set V4-TRANSFORM2 esp-gcm 256
 mode tunnel
!
crypto ipsec profile V2-PROFILE
 set transform-set V2-TRANSFORM2
 set ikev2-profile V2-PROFILE
!
crypto ipsec profile V4-PROFILE
 set transform-set V4-TRANSFORM2
 set ikev2-profile V4-PROFILE
!
interface Tunnel10
description S2S VPN Tunnel for Tenant 1
 ip vrf forwarding Tenant 1
 ip address 11.0.0.2 255.255.255.252
 ip tcp adjust-mss 1350
 tunnel source TenGigabitEthernet0/1/0.211
 tunnel mode ipsec ipv4
 tunnel destination 10.10.0.62
 tunnel vrf Tenant 1
 tunnel protection ipsec profile V2-PROFILE
!
interface Tunnel20
description S2S VPN Tunnel for Tenant 2
 ip vrf forwarding Tenant 2
 ip address 11.0.0.2 255.255.255.252
 ip tcp adjust-mss 1350
 tunnel source TenGigabitEthernet0/1/0.213
 tunnel mode ipsec ipv4
 tunnel destination 10.10.0.62
 tunnel vrf VNET3
 tunnel protection ipsec profile V4-PROFILE
!
interface GigabitEthernet0/0/1
 description PRIMARY ExpressRoute Link to AZURE over Equinix
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/1.100
description Primary WAN interface of Tenant 1
 description PRIMARY ER link supporting Tenant 1 to Azure
 encapsulation dot1Q 101
 ip vrf forwarding Tenant 1
 ip address 192.168.1.1 255.255.255.252
!
interface GigabitEthernet0/0/1.102
description Primary WAN interface of Tenant 2
 description PRIMARY ER link supporting Tenant 2 to Azure
 encapsulation dot1Q 102
 ip vrf forwarding Tenant 2
 ip address 192.168.1.17 255.255.255.252
!
interface GigabitEthernet0/0/2
 description BACKUP ExpressRoute Link to AZURE over Equinix
 no ip address
 negotiation auto
!
interface GigabitEthernet0/0/2.100
description Secondary WAN interface of Tenant 1
 description BACKUP ER link supporting Tenant 1 to Azure
 encapsulation dot1Q 101
 ip vrf forwarding Tenant 1
 ip address 192.168.1.5 255.255.255.252
!
interface GigabitEthernet0/0/2.102
description Secondary WAN interface of Tenant 2
description BACKUP ER link supporting Tenant 2 to Azure
 encapsulation dot1Q 102
 ip vrf forwarding Tenant 2
 ip address 192.168.1.21 255.255.255.252
!
interface TenGigabitEthernet0/1/0
 description Downlink to ---Port 1/47
 no ip address
!
interface TenGigabitEthernet0/1/0.211
 description LAN interface of Tenant 1
description Downlink to --- Port 1/47.211
 encapsulation dot1Q 211
 ip vrf forwarding Tenant 1
 ip address 10.60.3.255 255.255.255.254
!
interface TenGigabitEthernet0/1/0.213
description LAN interface of Tenant 2
 description Downlink to --- Port 1/47.213
 encapsulation dot1Q 213
 ip vrf forwarding Tenant 2
 ip address 10.60.3.251 255.255.255.254
!
router bgp 65530
 bgp router-id <removed>
 bgp log-neighbor-changes
 description BGP neighbor config and route advertisement for Tenant 1 VRF
 address-family ipv4 vrf Tenant 1
  network 10.1.0.0 mask 255.255.0.0
  network 10.60.3.254 mask 255.255.255.254
  network 192.168.1.0 mask 255.255.255.252
  network 192.168.1.4 mask 255.255.255.252
  neighbor 10.10.0.62 remote-as 65100
  neighbor 10.10.0.62 description VPN-BGP-PEER-for-Tenant 1
  neighbor 10.10.0.62 ebgp-multihop 5
  neighbor 10.10.0.62 activate
  neighbor 10.60.3.254 remote-as 4232570301
  neighbor 10.60.3.254 description LAN peer for CPEC:INET:2112 VRF
  neighbor 10.60.3.254 activate
  neighbor 10.60.3.254 route-map BLOCK-ALL out
  neighbor 192.168.1.2 remote-as 12076
  neighbor 192.168.1.2 description PRIMARY ER peer for Tenant 1 to Azure
  neighbor 192.168.1.2 ebgp-multihop 5
  neighbor 192.168.1.2 activate
  neighbor 192.168.1.2 soft-reconfiguration inbound
  neighbor 192.168.1.2 route-map Tenant 1-ONLY out
  neighbor 192.168.1.6 remote-as 12076
  neighbor 192.168.1.6 description BACKUP ER peer for Tenant 1 to Azure
  neighbor 192.168.1.6 ebgp-multihop 5
  neighbor 192.168.1.6 activate
  neighbor 192.168.1.6 soft-reconfiguration inbound
  neighbor 192.168.1.6 route-map Tenant 1-ONLY out
  maximum-paths 8
 exit-address-family
 !
description BGP neighbor config and route advertisement for Tenant 2 VRF
address-family ipv4 vrf Tenant 2
  network 10.1.0.0 mask 255.255.0.0
  network 10.60.3.250 mask 255.255.255.254
  network 192.168.1.16 mask 255.255.255.252
  network 192.168.1.20 mask 255.255.255.252
  neighbor 10.10.0.62 remote-as 65300
  neighbor 10.10.0.62 description VPN-BGP-PEER-for-Tenant 2
  neighbor 10.10.0.62 ebgp-multihop 5
  neighbor 10.10.0.62 activate
  neighbor 10.60.3.250 remote-as 4232570301
  neighbor 10.60.3.250 description LAN peer for CPEC:INET:2112 VRF
  neighbor 10.60.3.250 activate
  neighbor 10.60.3.250 route-map BLOCK-ALL out
  neighbor 192.168.1.18 remote-as 12076
  neighbor 192.168.1.18 description PRIMARY ER peer for Tenant 2 to Azure
  neighbor 192.168.1.18 ebgp-multihop 5
  neighbor 192.168.1.18 activate
  neighbor 192.168.1.18 soft-reconfiguration inbound
  neighbor 192.168.1.18 route-map VNET-ONLY out
  neighbor 192.168.1.22 remote-as 12076
  neighbor 192.168.1.22 description BACKUP ER peer for Tenant 2 to Azure
  neighbor 192.168.1.22 ebgp-multihop 5
  neighbor 192.168.1.22 activate
  neighbor 192.168.1.22 soft-reconfiguration inbound
  neighbor 192.168.1.22 route-map VNET-ONLY out
  maximum-paths 8
 exit-address-family
!
ip forward-protocol nd
!
ip as-path access-list 1 permit ^$
ip route vrf Tenant 1 10.1.0.0 255.255.0.0 Tunnel10
ip route vrf Tenant 2 10.1.0.0 255.255.0.0 Tunnel20
!
ip prefix-list BLOCK-ALL seq 5 deny 0.0.0.0/0 le 32
!
route-map BLOCK-ALL permit 10
 match ip address prefix-list BLOCK-ALL
!
route-map VNET-ONLY permit 10
 match as-path 1
!
```

## <a name="test-the-connection"></a>測試連線

建立站對站連線和 ExpressRoute 線路之後測試連線。

執行下列偵測測試：

* 登入 Azure VNet 中的其中一個虛擬機器，然後偵測您在 Azure Stack 中建立的虛擬機器。
* 登入您在 Azure Stack 中建立的其中一個虛擬機器，然後偵測您在 Azure VNet 中建立的虛擬機器。

>[!NOTE]
>為了確定是透過站對站和 ExpressRoute 連線來傳送流量，您必須在兩端都偵測虛擬機器的固定 IP (DIP) 位址，而不是虛擬機器的 VIP 位址。

### <a name="allow-icmp-in-through-the-firewall"></a>允許 ICMP 通過防火牆

根據預設，Windows Server 2016 不允許傳入的 ICMP 封包通過防火牆。 對於用於偵測測試的每個虛擬機器，您需要允許傳入的 ICMP 封包。 若要對 ICMP 建立防火牆規則，請在提升權限的 PowerShell 視窗中執行下列 Cmdlet：

```PowerShell
# Create ICMP firewall rule.
New-NetFirewallRule `
  –DisplayName “Allow ICMPv4-In” `
  –Protocol ICMPv4

```

### <a name="ping-the-azure-stack-virtual-machine"></a>偵測 Azure Stack 虛擬機器

1. 使用租用戶帳戶登入 Azure Stack 使用者入口網站。

1. 尋找您所建立的虛擬機器，然後選取虛擬機器。

1. 選取 [ **連接**]。

1. 從已提高權限的 Windows 或 PowerShell 命令提示字元，輸入 **ipconfig /all**。 記下輸出中傳回的 IPv4 位址。

1. 從 Azure VNet 中的虛擬機器偵測 IPv4 位址。

   在環境範例中，IPv4 位址來自 10.1.1.x/24 子網路。 在您的環境中，此位址可能不同。 但應該會在您為租用戶 VNet 子網路建立的子網路內。

### <a name="view-data-transfer-statistics"></a>檢視資料轉送統計資料

如果想要知道有多少流量通過您的連線，您可以在 Azure Stack 使用者入口網站上找到此資訊。 這也很適合用來了解偵測測試資料是否有通過 VPN 和 ExpressRoute 連線。

1. 使用租用戶帳戶登入 Azure Stack 使用者入口網站，然後選取 [所有資源]。
1. 瀏覽至 VPN 閘道的資源群組，然後選取 [連線] 物件類型。
1. 選取清單中的 [ConnectToAzure] 連線。
1. 在 [連線] > [概觀] 底下，您可以看到 [資料輸入] 和 [資料輸出] 的統計資料。您應該會看到一些非零值。

   ![資料輸入和資料輸出](media/azure-stack-connect-expressroute/DataInDataOut.png)

## <a name="next-steps"></a>後續步驟

[將應用程式部署到 Azure 和 Azure Stack](azure-stack-solution-pipeline.md)
