---
title: 確認連線：Azure ExpressRoute 疑難排解指南 | Microsoft Docs
description: 此頁面提供 ExpressRoute 路線的端對端連線確認和疑難排解的指示。
documentationcenter: na
services: expressroute
author: rambk
manager: tracsman
editor: ''
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/26/2017
ms.author: cherylmc
ms.openlocfilehash: 10d4779d05d95822ffd487db1ce8992d199c495f
ms.sourcegitcommit: 6eb14a2c7ffb1afa4d502f5162f7283d4aceb9e2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/25/2018
ms.locfileid: "36753435"
---
# <a name="verifying-expressroute-connectivity"></a>確認 ExpressRoute 連線
ExpressRoute 透過連線提供者所提供的私人連線將內部部署網路擴充至 Microsoft 雲端，涉及三個不同的網路區域：

-   客戶網路
-   提供者網路
-   Microsoft 資料中心

本文件的目的是協助使用者識別連線有沒有問題、在哪一個區域，藉此尋找適當的小組來解決問題。 如果需要 Microsoft 支援服務解決問題，請利用 [Microsoft 支援服務][Support]開啟支援票證。

> [!IMPORTANT]
> 本文件旨在協助診斷並修正簡單的問題。 它無法取代 Microsoft 支援服務。 如果無法使用本文提供的指引來解決問題，請利用 [Microsoft 支援服務][Support]開啟支援票證。
>
>

## <a name="overview"></a>概觀
下圖顯示使用 ExpressRoute 從客戶網路連至 Microsoft 網路的邏輯連線。
[![1]][1]

在上圖中，數字指出重要的網路點。 本文提及不同網路點會附上其圖中的編號。

網路點 3 和 4 可能互換 (第 2 層裝置)，取決於 ExpressRoute 連線模型 (雲端 Exchange 共置、點對點乙太網路連線、或任何對任何 (IPVPN))。 圖中的重要網路點分別是︰

1.  客戶計算裝置 (例如，伺服器或電腦)
2.  CE︰客戶邊緣路由器 
3.  PE (面對 CE)︰面對客戶邊緣路由器的提供者邊緣路由器/交換器。 在本文件中稱為 PE-CE。
4.  PE (面對 MSEE)︰面對 MSEE 的提供者邊緣路由器/交換器。 在本文件中稱為 PE-MSEE。
5.  MSEE：Microsoft Enterprise Edge (MSEE) ExpressRoute 路由器
6.  虛擬網路 (VNet) 閘道器
7.  Azure VNet 上的計算裝置

如果使用雲端 Exchange 共置或點對點乙太網路連線的連線模型，客戶邊緣路由器 (2) 會建立與 MSEE (5) 的 BGP 對等互連。 網路點 3 和 4 仍會存在，但會化做透明的第 2 層裝置。

如果使用任何對任何 (IPVPN) 連線模型，PE (面對 MSEE)(4) 會建立與 MSEE (5) 的 BGP 對等互連。 然後，路由會透過 IPVPN 服務網路提供者的網路傳播回客戶網路。

>[!NOTE]
>為了讓 ExpressRoute 有高可用性，Microsoft 需要在 MSEE (5) 和 PE-MSEE (4) 之間有一組備援的 BGP 工作階段。 我們也鼓勵您在客戶網路和 PE-CE 之間有一組備援的網路路徑。 不過，在任何對任何 (IPVPN) 連線模型中，單一 CE 裝置 (2) 可能會連線到一或多個 PE (3)。
>
>

若要確認 ExpressRoute 路線，需要下列步驟 (附網路點編號)︰
1. [確認路線的佈建和狀態 (5)](#validate-circuit-provisioning-and-state)
2. [確認至少已設定一個 ExpressRoute 對等互連 (5)](#validate-peering-configuration)
3. [確認 Microsoft 與服務之間的提供者之間的 ARP (4 和 5 之間的連結)](#validate-arp-between-microsoft-and-the-service-provider)
4. [確認 BGP 和 MSEE 上的路由 (BGP 在4 到 5 之間，如果連接 VNet 則在 5 至 6 之間) ](#validate-bgp-and-routes-on-the-msee)
5. [檢查流量統計資料 (通過 5 的流量)](#check-the-traffic-statistics)

未來將加入更多確認和檢查，請每個月回來查看！

## <a name="validate-circuit-provisioning-and-state"></a>確認路線的佈建和狀態
不論是何種連線模型，皆必須建立 ExpressRoute 路線，系統才會為佈建的路線產生服務機碼。 佈建 ExpressRoute 路線會在 PE-MSEE (4) 和 MSEE (5) 之間建立備援的第 2 層連線。 如需有關如何建立、修改、佈建、確認 ExpressRoute 路線的詳細資訊，請參閱[建立和修改 ExpressRoute 路線][CreateCircuit]一文。

>[!TIP]
>服務機碼可唯一識別 ExpressRoute 路線。 本文中所述的 Powershell 命令大多需要有這個機碼。 此外，如果您需 Microsoft 或 ExpressRoute 合作夥伴協助進行問題的疑難排解，請提供服務機碼以利輕易識別路線。
>
>

### <a name="verification-via-the-azure-portal"></a>透過 Azure 入口網站進行確認
在 Azure 入口網站中，選取功能表左欄中的 ![2][2]，然後選取 ExpressRoute 路線，便可以檢查 ExpressRoute 路線狀態。 選取 [所有資源] 底下列出的 ExpressRoute 路線，會開啟 ExpressRoute 路線刀鋒視窗。 在刀鋒視窗的 ![3][3] 區段中，會列出ExpressRoute 的基本資料，如以下螢幕擷取畫面所示︰

![4][4]    

在 ExpressRoute [基本資料] 中，[路線狀態] 指出 Microsoft 端路線的狀態。 [提供者狀態] 指出在服務提供者端是否「已佈建/未佈建」路線。 

[路線狀態] 必須是 [已啟用]，且[提供者狀態] 必須是 [已佈建]，ExpressRoute 路線才能運作。

>[!NOTE]
>如果 [路線狀態] 未啟用，請連絡 [Microsoft 支援服務][Support]。 如果 [提供者狀態] 是未佈建，請連絡您的服務提供者。
>
>

### <a name="verification-via-powershell"></a>透過 PowerShell 進行確認
若要列出資源群組中的所有 ExpressRoute 路線，使用下列命令：

    Get-AzureRmExpressRouteCircuit -ResourceGroupName "Test-ER-RG"

>[!TIP]
>您可以透過 Azure 取得您的資源群組名稱。 請參閱本文之前的文章段落，並記下範例螢幕擷取畫面中列出的資源群組名稱。
>
>

若要選取資源群組中的特定 ExpressRoute 路線，使用下列命令：

    Get-AzureRmExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"

範例回應：

    Name                             : Test-ER-Ckt
    ResourceGroupName                : Test-ER-RG
    Location                         : westus2
    Id                               : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt
    Etag                             : W/"################################"
    ProvisioningState                : Succeeded
    Sku                              : {
                                        "Name": "Standard_UnlimitedData",
                                        "Tier": "Standard",
                                        "Family": "UnlimitedData"
                                        }
    CircuitProvisioningState         : Enabled
    ServiceProviderProvisioningState : Provisioned
    ServiceProviderNotes             : 
    ServiceProviderProperties        : {
                                        "ServiceProviderName": "****",
                                        "PeeringLocation": "******",
                                        "BandwidthInMbps": 100
                                        }
    ServiceKey                       : **************************************
    Peerings                         : []
    Authorizations                   : []

若要確認 ExpressRoute 路線是否在運作，請特別注意下列欄位︰

    CircuitProvisioningState         : Enabled
    ServiceProviderProvisioningState : Provisioned

>[!NOTE]
>如果 CircuitProvisioningState 不是 enabled，請連絡 [Microsoft 支援服務][Support]。 如果 ServiceProviderProvisioningState 不是 provisioned，請連絡您的服務提供者。
>
>

### <a name="verification-via-powershell-classic"></a>透過 PowerShell (傳統) 進行確認
若要列出訂用帳戶下的所有 ExpressRoute 路線，使用下列命令：

    Get-AzureDedicatedCircuit

若要選取特定 ExpressRoute 路線，使用下列命令：

    Get-AzureDedicatedCircuit -ServiceKey **************************************

範例回應：

    andwidth                         : 100
    BillingType                      : UnlimitedData
    CircuitName                      : Test-ER-Ckt
    Location                         : westus2
    ServiceKey                       : **************************************
    ServiceProviderName              : ****
    ServiceProviderProvisioningState : Provisioned
    Sku                              : Standard
    Status                           : Enabled

若要確認 ExpressRoute 路線是否在運作，請特別注意下列欄位︰ServiceProviderProvisioningState : Provisioned Status                           : Enabled

>[!NOTE]
>如果 Status 不是 enabled，請連絡 [Microsoft 支援服務][Support]。 如果 ServiceProviderProvisioningState 不是 provisioned，請連絡您的服務提供者。
>
>

## <a name="validate-peering-configuration"></a>確認對等互連組態
服務提供者已完成 ExpressRoute 路之線佈建後，可以透過 MSEE-PR (4) 和 MSEE (5) 之間的 ExpressRoute 路線建立路由組態。 每個 ExpressRoute 線路可以啟用一、二或三個路由內容︰Azure 私人對等互連 (送至 Azure 中私人虛擬網路的流量)、Azure 公用對等互連 (送至 Azure 中公用 IP 位址的流量) 及 Microsoft 對等互連 (送至 Office 365 和 Dynamics 365 的流量)。 如需有關如何建立及修改路由組態的詳細資訊，請參閱[建立和修改 ExpressRoute 路線的路由][CreatePeering]一文。

### <a name="verification-via-the-azure-portal"></a>透過 Azure 入口網站進行確認

>[!NOTE]
>如果第 3 層是由服務提供者所提供，且對等互連在入口網站中為空白，請使用入口網站上的 [重新整理] 按鈕將循環組態重新整理。 此作業將套用在您線路上的正確路由組態。 
>
>

在 Azure 入口網站中，選取功能表左欄中的 ![2][2]，然後選取 ExpressRoute 路線，便可以查看 ExpressRoute 路線的狀態。 選取 [所有資源] 底下列出的 ExpressRoute 路線，會開啟 ExpressRoute 路線刀鋒視窗。 在刀鋒視窗的 ![3][3] 區段中，會列出ExpressRoute 的基本資料，如以下螢幕擷取畫面所示︰

![5][5]

在以上範例中，啟用了 Azure 私人對等互連的路由內容，但沒有啟用 Azure 公用路由內容和 Microsoft 對等互連路由內容。 啟用成功的對等互連內容也會列出主要和次要點對點 (BGP 所需) 子網路。 /30 子網路是用於 MSEE 和 PE-MSEE 的介面 IP 位址。 

>[!NOTE]
>若對等互連沒有啟用，檢查指派的主要和次要子網路是否符合 PE-MSEE 上的設定。 如果不符，若要變更 MSEE 路由器上的組態，請參閱[建立和修改 ExpressRoute 路線的路由][CreatePeering]。
>
>

### <a name="verification-via-powershell"></a>透過 PowerShell 進行確認
若要取得 Azure 私人對等互連的組態詳細資料，使用下列命令︰

    $ckt = Get-AzureRmExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
    Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt

以下是已成功設定私人對等互連的回應範例︰

    Name                       : AzurePrivatePeering
    Id                         : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt/peerings/AzurePrivatePeering
    Etag                       : W/"################################"
    PeeringType                : AzurePrivatePeering
    AzureASN                   : 12076
    PeerASN                    : ####
    PrimaryPeerAddressPrefix   : 172.16.0.0/30
    SecondaryPeerAddressPrefix : 172.16.0.4/30
    PrimaryAzurePort           : 
    SecondaryAzurePort         : 
    SharedKey                  : 
    VlanId                     : 300
    MicrosoftPeeringConfig     : null
    ProvisioningState          : Succeeded

 成功啟用的對等互連內容也會列出主要和次要位址前置詞。 /30 子網路是用於 MSEE 和 PE-MSEE 的介面 IP 位址。

若要取得 Azure 公用對等互連的組態詳細資料，使用下列命令︰

    $ckt = Get-AzureRmExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
    Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt

若要取得 Microsoft 對等互連的組態詳細資料，使用下列命令︰

    $ckt = Get-AzureRmExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
     Get-AzureRmExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt

如未設定對等互連，會顯示錯誤訊息。 未設定所述對等互連 (在此範例中是 Azure 公用互對等連) 的回應範例︰

    Get-AzureRmExpressRouteCircuitPeeringConfig : Sequence contains no matching element
    At line:1 char:1
        + Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering ...
        + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            + CategoryInfo          : CloseError: (:) [Get-AzureRmExpr...itPeeringConfig], InvalidOperationException
            + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.GetAzureExpressRouteCircuitPeeringConfigCommand


<p/>
>[!NOTE]
>若對等互連沒有啟用，檢查指派的主要和次要子網路是否符合連結之 PE-MSEE 上的設定。 也請檢查 MSEE 上是否使用正確的 VlanId、AzureASN、PeerASN，以及這些值是否對應至連結的 PE-MSEE 上使用的項目。 如果選擇 MD5 雜湊，共用的金鑰應該和「MSEE 與 PE-MSEE」對上的金鑰相同。 若要變更 MSEE 路由器上的組態，請參閱 [建立和修改 ExpressRoute 線路的路由][CreatePeering]。  
>
>

### <a name="verification-via-powershell-classic"></a>透過 PowerShell (傳統) 進行確認
若要取得 Azure 私人對等互連的組態詳細資料，使用下列命令︰

    Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"

以下是已成功設定私人對等互連的回應範例︰

    AdvertisedPublicPrefixes       : 
    AdvertisedPublicPrefixesState  : Configured
    AzureAsn                       : 12076
    CustomerAutonomousSystemNumber : 
    PeerAsn                        : ####
    PrimaryAzurePort               : 
    PrimaryPeerSubnet              : 10.0.0.0/30
    RoutingRegistryName            : 
    SecondaryAzurePort             : 
    SecondaryPeerSubnet            : 10.0.0.4/30
    State                          : Enabled
    VlanId                         : 100

啟用成功的對等互連內容也會列出主要和次要位址前置詞。 /30 子網路是用於 MSEE 和 PE-MSEE 的介面 IP 位址。

若要取得 Azure 公用對等互連的組態詳細資料，使用下列命令︰

    Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"

若要取得 Microsoft 對等互連的組態詳細資料，使用下列命令︰

    Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"

>[!IMPORTANT]
>如果第 3 層對等互連是由服務提供者設定，透過入口網站或 PowerShell 設定 ExpressRoute 對等互連可覆寫服務提供者的設定。 重設提供者端的對等互連設定需要服務提供者的支援。 只有當確定服務提供者僅提供第 2 層服務時，才修改 ExpressRoute 對等互連！
>
>

<p/>
>[!NOTE]
>若對等互連沒有啟用，檢查指派的主要和次要對等互連子網路是否符合連結之 PE-MSEE 上的設定。 也請檢查 MSEE 上是否使用正確的 VlanId、AzureAsn、PeerAsn，以及這些值是否對應至連結的 PE-MSEE 上使用的項目。 若要變更 MSEE 路由器上的組態，請參閱 [建立和修改 ExpressRoute 線路的路由][CreatePeering]。
>
>

## <a name="validate-arp-between-microsoft-and-the-service-provider"></a>確認 Microsoft 與服務提供者之間的 ARP
本節使用 PowerShell (傳統) 命令。 如果您一向使用 PowerShell 的 Azure Resource Manager 命令，請確定您具有可以存取訂用帳戶的系統管理員/共同管理員存取權。 如需針對使用 Azure Resource Manager 命令進行疑難排解，請參閱[在 Resource Manager 部署模型中取得 ARP 資料表][ARP]文件。

>[!NOTE]
>若要取得 ARP，可以使用 Azure 入口網站和 Azure Resource Manager PowerShell 命令。 如果使用 Azure Resource Manager PowerShell 命令時發生錯誤，傳統 PowerShell 命令也可以用於 Azure Resource Manager 的 ExpressRoute 路線。
>
>

若要從主要 MSEE 路由器取得 ARP 表格以用於私人對等互連，使用下列命令︰

    Get-AzureDedicatedCircuitPeeringArpInfo -AccessType Private -Path Primary -ServiceKey "*********************************"

在成功的案例中，命令的回應範例如下︰

    ARP Info:

                 Age           Interface           IpAddress          MacAddress
                 113             On-Prem       10.0.0.1           e8ed.f335.4ca9
                   0           Microsoft       10.0.0.2           7c0e.ce85.4fc9

同樣地，您可以查看 Primary/Secondary 路徑中 MSEE 的 ARP 表，用於 Private/Public/Microsoft 對等互連。

以下是對等互連不存在時此命令的回應範例。

    ARP Info:
       
>[!NOTE]
>如果 ARP 表沒有對應到 MAC 位址之介面的 IP 位址，請檢閱下列條件︰
>1. 針對 MSEE-PR 和 MSEE 之間的連結所指派的 /30 子網路的第一個 IP 位址，是用在 MSEE-PR 的介面上。 Azure 一律為 MSEE 使用第二個 IP 位址。
>2. 確認客戶 (C 標籤) 和服務 (S 標籤) VLAN 標籤是否都和「MSEE-PR 與 MSEE」對上的相符。
>
>

## <a name="validate-bgp-and-routes-on-the-msee"></a>確認 MSEE 上的 BGP 和路由
本節使用 PowerShell (傳統) 命令。 如果您一向使用 PowerShell 的 Azure Resource Manager 命令，請確定您具有可以存取訂用帳戶的系統管理員/共同管理員存取權。

>[!NOTE]
>若要取得 BGP 資訊，可以使用 Azure 入口網站和 Azure Resource Manager PowerShell 命令。 如果使用 Azure Resource Manager PowerShell 命令時發生錯誤，傳統 PowerShell 命令也可以用於 Azure Resource Manager 的 ExpressRoute 路線。
>
>

若要取得特定路由內容的路由表 (BGP 芳鄰) 摘要，使用下列命令︰

    Get-AzureDedicatedCircuitPeeringRouteTableSummary -AccessType Private -Path Primary -ServiceKey "*********************************"

以下是回應範例：

    Route Table Summary:

            Neighbor                   V                  AS              UpDown         StatePfxRcd
            10.0.0.1                   4                ####                8w4d                  50

如以上範例所示，此命令在判斷路由內容已建立多久時十實用分。 它也會指出對等互連路由器通告的路由前置詞數目。

>[!NOTE]
>如果狀態是作用中或閒置，檢查指派的主要和次要對等互連子網路是否符合連結之 PE-MSEE 上的設定。 也請檢查 MSEE 上是否使用正確的 VlanId、AzureAsn、PeerAsn，以及這些值是否對應至連結的 PE-MSEE 上使用的項目。 如果選擇 MD5 雜湊，共用的金鑰應該和「MSEE 與 PE-MSEE」對上的金鑰相同。 若要變更 MSEE 路由器上的組態，請參閱[建立和修改 ExpressRoute 路線的路由][CreatePeering]。
>
>

<p/>
>[!NOTE]
>如果無法透過特定對等互連連線到特定目的地，請確認 MSEE 的路由表屬於該特定對等互連內容。 如果路由表中有符合的前置詞 (可能是 NATed IP)，則請確認路徑上有防火牆/NSG/ACL，且防火牆/NSG/ACL 允許流量。
>
>

若要從 Primary 路徑上的 MSEE 取得完整路由用於特定 Private 路由內容，使用下列命令︰

    Get-AzureDedicatedCircuitPeeringRouteTableInfo -AccessType Private -Path Primary -ServiceKey "*********************************"

以下是此命令的成功結果範例︰

    Route Table Info:

             Network             NextHop              LocPrf              Weight                Path
         10.1.0.0/16            10.0.0.1                                       0    #### ##### #####     
         10.2.0.0/16            10.0.0.1                                       0    #### ##### #####
    ...

同樣地，您可以查看 Primary/Secondary 路徑中 MSEE 的路由表，用於 Private/Public/Microsoft 對等互連內容。

以下是對等互連不存在時此命令的回應範例：

    Route Table Info:

## <a name="check-the-traffic-statistics"></a>檢查流量統計資料
若要取得對等互連內容的主要和次要路徑的合併流量統計資料，包括進和出的位元組，使用下列命令︰

    Get-AzureDedicatedCircuitStats -ServiceKey 97f85950-01dd-4d30-a73c-bf683b3a6e5c -AccessType Private

此命令的輸出範例如下：

    PrimaryBytesIn PrimaryBytesOut SecondaryBytesIn SecondaryBytesOut
    -------------- --------------- ---------------- -----------------
         240780020       239863857        240565035         239628474

對等互連不存在時，命令的輸出範例如下︰

    Get-AzureDedicatedCircuitStats : ResourceNotFound: Can not find any subinterface for peering type 'Public' for circuit '97f85950-01dd-4d30-a73c-bf683b3a6e5c' .
    At line:1 char:1
    + Get-AzureDedicatedCircuitStats -ServiceKey 97f85950-01dd-4d30-a73c-bf ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzureDedicatedCircuitStats], CloudException
        + FullyQualifiedErrorId : Microsoft.WindowsAzure.Commands.ExpressRoute.GetAzureDedicatedCircuitPeeringStatsCommand

## <a name="next-steps"></a>後續步驟
如需詳細資訊或協助，請看看下列連結：

- [Microsoft 支援服務][Support]
- [建立和修改 ExpressRoute 路線][CreateCircuit]
- [建立和修改 ExpressRoute 路線的路由][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-expressroute-overview/expressroute-logical-diagram.png "邏輯 Express 路由連線"
[2]: ./media/expressroute-troubleshooting-expressroute-overview/portal-all-resources.png "所有資源圖示"
[3]: ./media/expressroute-troubleshooting-expressroute-overview/portal-overview.png "概觀圖示"
[4]: ./media/expressroute-troubleshooting-expressroute-overview/portal-circuit-status.png "ExpressRoute 基本資料螢幕擷取畫面範例"
[5]: ./media/expressroute-troubleshooting-expressroute-overview/portal-private-peering.png "ExpressRoute 基本資料螢幕擷取畫面範例"

<!--Link References-->
[Support]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[CreateCircuit]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-circuit-portal-resource-manager 
[CreatePeering]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-routing-portal-resource-manager
[ARP]: https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-arp-resource-manager






