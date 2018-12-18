---
title: 從虛擬網路到 SAP HANA on Azure (大型執行個體) 的連線設定 | Microsoft Docs
description: 從虛擬網路到 SAP HANA on Azure (大型執行個體) 的連線設定。
services: virtual-machines-linux
documentationcenter: ''
author: RicksterCDN
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: rclaus
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 5efdda485e4e1f5013948c6636b267f0d388f4d5
ms.sourcegitcommit: 2d961702f23e63ee63eddf52086e0c8573aec8dd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44167583"
---
# <a name="connecting-a-vnet-to-hana-large-instance-expressroute"></a>將 VNet 連接到 HANA 大型執行個體 ExpressRoute

由於您已定義所有 IP 位址範圍，並且現在已取得 Microsoft 傳回的資料，因此可以開始將以前建立的 VNet 連接到「HANA 大型執行個體」。 建立 Azure VNet 之後，必須在該 VNet 上建立 ExpressRoute 閘道，以將 VNet 連結到會連接到「大型執行個體」戳記上客戶租用戶的 ExpressRoute 線路。

> [!NOTE] 
> 此步驟最多可能需要 30 分鐘才能完成，因為會在指定的 Azure 訂用帳戶中建立新閘道，然後連接到指定的 Azure VNet。

如果閘道已經存在，請檢查它是否是 ExpressRoute 閘道。 如果不是，就必須刪除該閘道，然後重新建立成 ExpressRoute 閘道。 如果已經建立 ExpressRoute 閘道，則由於 Azure VNet 已經連接到用於內部部署環境連線的 ExpressRoute 線路，因此請繼續進行下面的＜連結 VNet＞小節。

- 使用 (新版) [Azure 入口網站](https://portal.azure.com/) 或 PowerShell 來建立連接到您 VNet 的 ExpressRoute VPN 閘道。
  - 如果您使用 Azure 入口網站，請新增新的 [虛擬網路閘道]，然後選取 [ExpressRoute] 作為閘道類型。
  - 如果您改為選擇 PowerShell，則請先下載並使用 [Azure PowerShell SDK](https://azure.microsoft.com/downloads/)，以確保獲得最佳體驗。 下列命令會建立 ExpressRoute 閘道。 前面有 _$_ 的文字是使用者定義的變數，必須更新成您的特定資訊。

```PowerShell
# These Values should already exist, update to match your environment
$myAzureRegion = "eastus"
$myGroupName = "SAP-East-Coast"
$myVNetName = "VNet01"

# These values are used to create the gateway, update for how you wish the GW components to be named
$myGWName = "VNet01GW"
$myGWConfig = "VNet01GWConfig"
$myGWPIPName = "VNet01GWPIP"
$myGWSku = "HighPerformance" # Supported values for HANA Large Instances are: HighPerformance or UltraPerformance

# These Commands create the Public IP and ExpressRoute Gateway
$vnet = Get-AzureRmVirtualNetwork -Name $myVNetName -ResourceGroupName $myGroupName
$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
New-AzureRmPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName `
-Location $myAzureRegion -AllocationMethod Dynamic
$gwpip = Get-AzureRmPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName
$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name $myGWConfig -SubnetId $subnet.Id `
-PublicIpAddressId $gwpip.Id

New-AzureRmVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName -Location $myAzureRegion `
-IpConfigurations $gwipconfig -GatewayType ExpressRoute `
-GatewaySku $myGWSku -VpnType PolicyBased -EnableBgp $true
```

在此範例中，使用了 HighPerformance 閘道 SKU。 您的選項為 HighPerformance 或 UltraPerformance，因為 SAP HANA on Azure (大型執行個體) 僅支援這些閘道 SKU。

> [!IMPORTANT]
> 對於類型 II 類別 SKU 的 HANA 大型執行個體，必須使用 UltraPerformance 閘道 SKU。

**連結 VNet**

既然 Azure VNet 已有 ExpressRoute 閘道，您便可以使用 Microsoft 所提供的授權資訊，將 ExpressRoute 閘道連接到為此連線建立的 SAP HANA on Azure (大型執行個體) ExpressRoute 線路。 您可以使用 Azure 入口網站或 PowerShell 來執行這個步驟。 建議使用入口網站，不過，以下提供 PowerShell 指示。 

- 您需針對每個連線使用不同的 AuthGUID，來為每個 VNet 閘道執行下列命令。 下列指令碼顯示的前兩個項目來自 Microsoft 所提供的資訊。 此外，AuthGUID 對每個 VNet 及其閘道來說都是特定的。 這意謂著如果您想要新增另一個 Azure VNet，就必須為將 HANA 大型執行個體連接到 Azure 的 ExpressRoute 線路取得另外一個 AuthID。 

```PowerShell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway Information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzureRmVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzureRmVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID
```

如果您想要將閘道連接到與訂用帳戶關聯的多個 ExpressRoute 線路，可能會需要執行此步驟多次。 例如，您可能要將相同的 VNet 閘道連線到 ExpressRoute 線路，而此線路會將 VNet 連線到內部部署網路。

**後續步驟**

- 請參閱 [HLI 的其他網路需求](hana-additional-network-requirements.md)。