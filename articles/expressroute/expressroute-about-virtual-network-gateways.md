---
title: 關於 Azure ExpressRoute 虛擬網路閘道 | Microsoft Docs
description: 了解 ExpressRoute 的虛擬網路閘道。
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: cherylmc
ms.openlocfilehash: 34d84a27406f0ebabd7bca576ee443da1d0c9bcd
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44348799"
---
# <a name="about-virtual-network-gateways-for-expressroute"></a>關於 ExpressRoute 的虛擬網路閘道
虛擬網路閘道可用來傳送 Azure 虛擬網路和內部部署位置之間的網路流量。 您可以將虛擬網路閘道使用於 ExpressRoute 流量或 VPN 流量。 本文著重於 ExpressRoute 虛擬網路閘道。

## <a name="gateway-types"></a>閘道類型

當您建立虛擬網路閘道時，需要指定數個設定。 其中一個必要設定 '-GatewayType' 會指定是否要對 ExpressRoute 或 VPN 閘道流量使用閘道。 兩種閘道類型如下： 

* **Vpn** - 若要透過公用網際網路傳送已加密的流量，請使用 'Vpn' 閘道類型。 也稱之為 VPN 閘道。 站對站、點對站和 VNet 對 VNet 連線都使用 VPN 閘道。

* **ExpressRoute** - 若要在私人連線上傳送網路流量，請使用 'ExpressRoute' 閘道類型。 這也稱為 ExpressRoute 閘道，而且是您設定 ExpressRoute 時所用的閘道類型。


對於每種閘道類型，每個虛擬網路只能有一個虛擬網路閘道。 例如，您可以有一個使用 -GatewayType Vpn 的虛擬網路閘道，以及一個使用 -GatewayType ExpressRoute 的虛擬網路閘道。

## <a name="gwsku"></a>閘道 SKU
[!INCLUDE [expressroute-gwsku-include](../../includes/expressroute-gwsku-include.md)]

如果您想要將閘道器升級至更強大的閘道 SKU，在大部分情況下，您可以使用 'Resize-AzureRmVirtualNetworkGateway' PowerShell Cmdlet。 這適用於升級至 Standard 和 HighPerformance SKU。 不過，若要升級至 UltraPerformance SKU，您必須重新建立閘道器。 重新建立閘道時會導致停機。

### <a name="aggthroughput"></a>閘道 SKU 預估的效能
下表顯示閘道類型和預估的效能。 此資料表適用於資源管理員與傳統部署模型。

[!INCLUDE [expressroute-table-aggthroughput](../../includes/expressroute-table-aggtput-include.md)]

> [!IMPORTANT]
> 應用程式效能取決於多項因素，例如端對端延遲以及應用程式開啟之流量的數目。 表格中的數字代表應用程式在理想的環境中，理論上可以達成的最高上限。 
> 
>

### <a name="zrgw"></a>區域備援閘道 SKU (預覽)

您也可以在 Azure 可用性區域中部署 ExpressRoute 閘道。 這能夠在實際上和邏輯上將閘道分隔為不同的可用性區域，同時還能在發生區域層級的失敗時，保護內部部署項目與 Azure 的網路連線。

![區域備援 ExpressRoute 閘道](./media/expressroute-about-virtual-network-gateways/zone-redundant.png)

區域備援閘道會使用適用於 ExpressRoute 閘道的特定新式閘道 SKU。 新的 SKU 目前以**公開預覽**的形式提供。

* ErGw1AZ
* ErGw2AZ
* ErGw3AZ

新的閘道 SKU 也支援其他部署選項，以充分符合您的需求。 使用新的閘道 SKU 建立虛擬網路閘道時，您也可選擇在特定區域中部署閘道。 稱之為區域閘道。 當您部署分區閘道時，閘道的所有執行個體都會部署在相同的可用性區域中。 若要註冊預覽版，請參閱[建立區域備援虛擬網路閘道](../../articles/vpn-gateway/create-zone-redundant-vnet-gateway.md)。

## <a name="resources"></a>REST API 和 PowerShell Cmdlet
如需將 REST API 和 PowerShell Cmdlet 使用於虛擬網路閘道組態時的其他技術資源和特定語法需求，請參閱下列頁面︰

| **傳統** | **Resource Manager** |
| --- | --- |
| [PowerShell](https://msdn.microsoft.com/library/mt270335.aspx) |[PowerShell](https://docs.microsoft.com/powershell/module/azurerm.network#networking) |
| [REST API](https://msdn.microsoft.com/library/jj154113.aspx) |[REST API](https://msdn.microsoft.com/library/mt163859.aspx) |

## <a name="next-steps"></a>後續步驟
如需有關可用連線組態的詳細資訊，請參閱 [ExpressRoute 概觀](expressroute-introduction.md) 。

如需建立 ExpressRoute 閘道的詳細資訊，請參閱[為 ExpressRoute 建立虛擬網路閘道](expressroute-howto-add-gateway-resource-manager.md)。
