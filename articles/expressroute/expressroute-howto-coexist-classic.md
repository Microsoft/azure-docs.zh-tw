---
title: 設定可以並存的 ExpressRoute 和站對站 VPN 連線：傳統：Azure | Microsoft Docs
description: 本文會引導您針對傳統部署模型設定可以並存的 ExpressRoute 和站對站 VPN 連線。
documentationcenter: na
services: expressroute
author: charwen
manager: carmonm
editor: ''
tags: azure-service-management
ms.assetid: dcf1a5af-a289-466a-b812-0bfedbd2bda0
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/21/2017
ms.author: charwen
ms.openlocfilehash: 09d1649f0ca0cf4ca464d95b29461cad3fe51788
ms.sourcegitcommit: 68aec76e471d677fd9a6333dc60ed098d1072cfc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/18/2017
ms.locfileid: "22710128"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-classic"></a>設定 ExpressRoute 和站對站並存連線 (傳統)
> [!div class="op_single_selector"]
> * [PowerShell - 資源管理員](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell - 傳統](expressroute-howto-coexist-classic.md)
> 
> 

能夠設定站對站 VPN 和 ExpressRoute 有諸多好處。 您可以將站對站 VPN 設定為 ExressRoute 的安全容錯移轉路徑，或使用站對站 VPN 來連接至不是透過 ExpressRoute 連接的網站。 本文中將說明設定這兩個案例的步驟。 本文適用於傳統部署模型。 此組態無法使用於入口網站。

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**關於 Azure 部署模型**

[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

> [!IMPORTANT]
> 在執行下列指示之前，必須事先設定 ExpressRoute 線路。 在執行下列步驟之前，請確定您已遵循[建立 ExpressRoute 線路](expressroute-howto-circuit-classic.md)和[設定路由](expressroute-howto-routing-classic.md)的指南。
> 
> 

## <a name="limits-and-limitations"></a>限制
* **不支援傳輸路由。** 您無法在透過站對站 VPN 連線的區域網路與透過 ExpressRoute 連線的區域網路之間，進行路由傳送 (透過 Azure)。
* **不支援點對站連線。** 您無法啟用連接到 ExpressRoute 之相同 VNet 的點對站 VPN 連線。 點對站 VPN 和 ExpressRoute 不能並存在相同的 VNet。
* **無法在站對站 VPN 閘道上啟用強制通道。** 您只可以「強制」所有網際網路繫結流量透過 ExpressRoute 回到內部部署網路。
* **不支援基本 SKU 閘道。** 您必須對 [ExpressRoute 閘道](expressroute-about-virtual-network-gateways.md)和 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)使用非基本 SKU 閘道。
* **僅支援路由式 VPN 閘道。** 您必須使用路由式 [VPN 閘道](../vpn-gateway/vpn-gateway-about-vpngateways.md)。
* **應該為 VPN 閘道設定靜態路由。** 如果您的區域網路連線到 ExpressRoute 和站對站 VPN，您必須在區域網路中設定靜態路由，才能將站對站 VPN 連線路由傳送到公用網際網路。
* **必須先設定 ExpressRoute 閘道。** 您必須先建立 ExpressRoute 閘道器，才能新增站對站 VPN 閘道。

## <a name="configuration-designs"></a>組態設計
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>設定站對站 VPN 作為 ExpressRoute 的容錯移轉路徑
您可以設定站對站 VPN 連線作為 ExpressRoute 的備用連線。 這僅適用於連結至 Azure 私用對等路徑的虛擬網路。 對於可透過 Azure 公用和 Microsoft 對等存取的服務，沒有 VPN 架構的容錯移轉解決方案。 ExpressRoute 線路一律為主要連結。 只有在 ExpressRoute 線路故障時，資料才能流經站對站 VPN 路徑。 

> [!NOTE]
> 雖然當兩個路由相同時，ExpressRoute 線路偏好透過站對站 VPN，但 Azure 會使用最長的相符前置詞來選擇朝向封包目的地的路由。
> 
> 

![並存](media/expressroute-howto-coexist-classic/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>設定站對站 VPN 來連線到未透過 ExpressRoute 連接的網站
您可以將網路設定成有些網站透過站對站 VPN 直接連線到 Azure，而有些網站透過 ExpressRoute 來連線。 

![並存](media/expressroute-howto-coexist-classic/scenario2.jpg)

> [!NOTE]
> 您無法將虛擬網路設定為轉送路由器。
> 
> 

## <a name="selecting-the-steps-to-use"></a>選取要使用的步驟
如果要設定可並存的連線，有兩組不同的程序可供選擇。 您所選取的設定程序取決於您是已經有想要連線的現有虛擬網路，還是想要建立新的虛擬網路。

* 我沒有 VNet 且需要建立一個。
  
    如果您還沒有虛擬網路，這個程序將引導您使用傳統部署模型來建立新的虛擬網路，並建立新的 ExpressRoute 和站對站 VPN 連線。 若要設定，請依照 [建立新的虛擬網路和並存的連線](#new)一節中的步驟進行。
* 我已經有一個傳統部署模型 VNet。
  
    您可能已經有虛擬網路，而且使用現有的站對站 VPN 連線或 ExpressRoute 連線。 本文的 [為已經存在的 VNet 設定並存的連線](#add) 一節將引導您刪除閘道，然後建立新的 ExpressRoute 和站對站 VPN 連線。 請注意，建立新的連線時，您必須以非常特定的順序來完成這些步驟。 請勿使用其他文章中的指示建立閘道器和連線。
  
    在此程序中，建立可以並存的連線將會要求您刪除閘道器，然後設定新的閘道器。 這表示當您刪除和重新建立閘道器與連線時，將會有跨設備連線的停機時間，但您不需要將任何 VM 或服務移轉至新的虛擬網路。 您的 VM 和服務仍能透過負載平衡器對外通訊，而您能夠在這兩者設定為這樣做時進行閘道器設定。

## <a name="new"></a>建立新的虛擬網路和並存的連線
此程序會引導您建立 VNet，並建立將並存的站對站和 ExpressRoute 連線。

1. 您必須安裝最新版的 Azure PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 請注意，您將針對此組態使用的 Cmdlet 可能與您熟悉的 Cmdlet 有些微不同。 請務必使用這些指示中指定的 Cmdlet。 
2. 建立虛擬網路的結構描述。 如需關於組態結構描述的詳細資訊，請參閱 [Azure 虛擬網路組態結構描述](https://msdn.microsoft.com/library/azure/jj157100.aspx)。
   
    當您建立結構描述時，請務必使用下列值：
   
   * 虛擬網路的閘道器子網路必須是 /27 或更短的首碼 (例如 /26 或 /25)。
   * 閘道器連線類型為「專用」。
     
             <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
               <AddressSpace>
                 <AddressPrefix>10.17.159.192/26</AddressPrefix>
               </AddressSpace>
               <Subnets>
                 <Subnet name="Subnet-1">
                   <AddressPrefix>10.17.159.192/27</AddressPrefix>
                 </Subnet>
                 <Subnet name="GatewaySubnet">
                   <AddressPrefix>10.17.159.224/27</AddressPrefix>
                 </Subnet>
               </Subnets>
               <Gateway>
                 <ConnectionsToLocalNetwork>
                   <LocalNetworkSiteRef name="MyLocalNetwork">
                     <Connection type="Dedicated" />
                   </LocalNetworkSiteRef>
                 </ConnectionsToLocalNetwork>
               </Gateway>
             </VirtualNetworkSite>
3. 建立和設定 XML 結構描述檔案後，請上傳該檔案。 這樣會建立您的虛擬網路。
   
    使用下列 Cmdlet 來上傳檔案，將該值替換為您自己的值。
   
        Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'
4. <a name="gw"></a>建立 ExpressRoute 閘道。 請務必將 GatewaySKU 指定為 *Standard*、*HighPerformance* 或 *UltraPerformance*，並將 GatewayType 指定為 *DynamicRouting*。
   
    使用以下範例，將該值替換為您自己的值。
   
        New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance
5. 將 ExpressRoute 閘道器連結到 ExpressRoute 電路。 完成這個步驟之後，內部部署網路和 Azure 之間的連線 (透過 ExpressRoute ) 便會建立。
   
        New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET
6. <a name="vpngw"></a>接下來，建立站對站 VPN 閘道。 GatewaySKU 必須是 *Standard*、*HighPerformance* 或 *UltraPerformance*，而 GatewayType 必須是 *DynamicRouting*。
   
        New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance
   
    若要擷取虛擬網路閘道設定 (包括閘道識別碼和公用 IP)，請使用 `Get-AzureVirtualNetworkGateway` Cmdlet。
   
        Get-AzureVirtualNetworkGateway
   
        GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
        GatewayName          : S2SVPN
        LastEventData        :
        GatewayType          : DynamicRouting
        LastEventTimeStamp   : 5/29/2015 4:41:41 PM
        LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
        LastEventID          : 23002
        State                : Provisioned
        VIPAddress           : 104.43.x.y
        DefaultSite          :
        GatewaySKU           : HighPerformance
        Location             :
        VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
        SubnetId             :
        EnableBgp            : False
        OperationDescription : Get-AzureVirtualNetworkGateway
        OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
        OperationStatus      : Succeeded
7. 建立本機的站台 VPN 閘道實體。 此命令不會設定內部部署 VPN 閘道。 相反地，它可讓您提供本機閘道器設定 (例如公用 IP 與內部位址空間)，使 Azure VPN 閘道能夠與其連線。
   
   > [!IMPORTANT]
   > 未在 Netcfg 中定義站對站 VPN 的本機站台。 您必須改為使用此 Cmdlet 來指定本機站台參數。 您無法使用入口網站或 Netcfg 檔來定義參數。
   > 
   > 
   
    使用下列範例，將該值替換為您自己的值。
   
        New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <MyLocalGatewayIp> -AddressSpace <MyLocalNetworkAddress>
   
   > [!NOTE]
   > 如果您的區域網路有多個路由，您可以將它們全部放在陣列中傳遞。  $MyLocalNetworkAddress = @("10.1.2.0/24","10.1.3.0/24","10.2.1.0/24")  
   > 
   > 

    若要擷取虛擬網路閘道設定 (包括閘道識別碼和公用 IP)，請使用 `Get-AzureVirtualNetworkGateway` Cmdlet。 請參閱下列範例。

        Get-AzureLocalNetworkGateway

        GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
        GatewayName          : MyLocalNetwork
        IpAddress            : 23.39.x.y
        AddressSpace         : {10.1.2.0/24}
        OperationDescription : Get-AzureLocalNetworkGateway
        OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
        OperationStatus      : Succeeded


1. 設定本機 VPN 裝置以連線到新的閘道器。 當您設定 VPN 裝置時，請使用於步驟 6 所抓取的資訊。 如需關於 VPN 裝置組態的詳細資訊，請參閱 [VPN 裝置組態](../vpn-gateway/vpn-gateway-about-vpn-devices.md)。
2. 將 Azure 上的站對站 VPN 閘道連結至本機閘道器。
   
    在此範例中，connectedEntityId 是本機的閘道器識別碼，您可以透過執行 `Get-AzureLocalNetworkGateway`找到此識別碼。 您可以使用 `Get-AzureVirtualNetworkGateway` Cmdlet 尋找 virtualNetworkGatewayId。 完成這個步驟之後，區域網路和 Azure 之間的連線 (透過站對站 VPN 連線) 便會建立。

        New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>

## <a name="add"></a>為已經存在的 VNet 設定並存的連線
如果您有現有的虛擬網路，請檢查閘道器子網路大小。 如果閘道器子網路是/28 或/29，您必須先刪除虛擬網路閘道器，並增加閘道器子網路大小。 本節中的步驟將示範如何執行該作業。

如果閘道器子網路是/27 以上且虛擬網路是透過 ExpressRoute 連線，則可以略過下列步驟，並且繼續進行上一節中的 [「步驟 6 - 建立站對站 VPN 閘道」](#vpngw) 。

> [!NOTE]
> 當您刪除現有閘道器時，您在進行此設定時，本機設備將會與虛擬網路中斷連線。
> 
> 

1. 您必須安裝最新版的 Azure 資源管理員 PowerShell Cmdlet。 如需如何安裝 PowerShell Cmdlet 的詳細資訊，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。 請注意，您將針對此組態使用的 Cmdlet 可能與您熟悉的 Cmdlet 有些微不同。 請務必使用這些指示中指定的 Cmdlet。 
2. 刪除現有的 ExpressRoute 或站對站 VPN 閘道。 使用下列 Cmdlet，將該值替換為您自己的值。
   
        Remove-AzureVNetGateway –VnetName MyAzureVNET
3. 匯出虛擬網路的結構描述。 使用下列 PowerShell Cmdlet，將該值替換為您自己的值。
   
        Get-AzureVNetConfig –ExportToFile “C:\NetworkConfig.xml”
4. 編輯網路組態檔結構描述，讓閘道器子網路是 /27 或更短的首碼 (例如 /26 或 /25)。 請參閱下列範例。 
   
   > [!NOTE]
   > 如果您的虛擬網路中沒有足夠的 IP 位址可以增加閘道器子網路大小，您必須新增更多 IP 位址空間。 如需關於組態結構描述的詳細資訊，請參閱 [Azure 虛擬網路組態結構描述](https://msdn.microsoft.com/library/azure/jj157100.aspx)。
   > 
   > 
   
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.17.159.224/27</AddressPrefix>
          </Subnet>
5. 如果您先前的閘道是站對站 VPN，則也必須將連線類型變更為 [專用] 。
   
                 <Gateway>
                  <ConnectionsToLocalNetwork>
                    <LocalNetworkSiteRef name="MyLocalNetwork">
                      <Connection type="Dedicated" />
                    </LocalNetworkSiteRef>
                  </ConnectionsToLocalNetwork>
                </Gateway>
6. 此時，您必須使用沒有閘道器的 VNet。 若要建立新的閘道器並完成連接，您可以繼續進行 [步驟 4 - 建立 ExpressRoute 閘道器](#gw)(您可以在先前的步驟組中找到)。

## <a name="next-steps"></a>後續步驟
如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)

