---
title: 從傳統 VPN 閘道到 Resource Manager 移轉 |Microsoft Docs
description: 此頁面提供傳統 VPN 閘道到 Resource Manager 移轉的概觀。
documentationcenter: na
services: vpn-gateway
author: amsriva
manager: rossort
editor: amsriva
ms.assetid: caa8eb19-825a-4031-8b49-18fbf3ebc04e
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/02/2017
ms.author: amsriva
ms.openlocfilehash: 1164fc24355657af22b6befaad74685ebbc2b5cb
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23127096"
---
# <a name="vpn-gateway-classic-to-resource-manager-migration"></a>從傳統 VPN 閘道到 Resource Manager 移轉
VPN 閘道現在可以用來從傳統移轉至 Resource Manager 部署模型。 您可以進一步了解 Azure Resource Manager [功能和優點](../azure-resource-manager/resource-group-overview.md)。 在本文中，我們會詳細說明如何從傳統部署移轉至較新的 Resource Manager 為基礎模型。 

VPN 閘道會在從傳統至 Resource Manager 的移轉過程中做為 VNet 移轉。 此移轉是一次一個 VNet。 沒有其他工具或移轉的必要條件需求。 移轉步驟與現有的 VNet 移轉相同，且會記載在 [IaaS 資源移轉網頁](../virtual-machines/windows/migration-classic-resource-manager-ps.md)。 在移轉期間沒有任何資料路徑停機時間，因此現有的工作負載在移轉期間會繼續運作而不會中斷內部部署連線。 與 VPN 閘道相關聯的公用 IP 位址在移轉程序期間不會變更。 這表示一旦移轉完成後，您將不需要重新設定內部部署路由器。  

Resource Manager 中的模型不同於傳統模型，是由虛擬網路閘道、區域網路閘道和連線資源所組成。 這些分別代表 VPN 閘道本身、代表內部部署位址空間的本機站台和兩者之間的連線能力。 移轉完成後，您的閘道不能使用傳統模型，且虛擬網路閘道、區域網路閘道及連線物件上的所有管理作業必須使用 Resource Manager 模式執行。

## <a name="supported-scenarios"></a>支援的案例
傳統至 Resource Manager 的移轉涵蓋最常見的 VPN 連線案例。 支援的案例包括 -

* 點對站台連線能力
* 站台對站台與連線到內部部署位置之 VPN 閘道的連線能力
* 使用 VPN 閘道的兩個 Vnet 之間之 VNet 對 VNet 連線能力
* 多個 Vnet 連接到相同內部部署位置
* 多站台連線能力
* 強制通道已啟用 VNets

不支援的案例包括 -  

* 目前不支援 VNet 隨附 ExpressRoute 閘道與 VPN 閘道。
* 傳輸 VM 擴充功能連線到內部部署伺服器的案例。 傳輸 VPN 連線能力限制如下所述。

> [!NOTE]
> Resource Manager 模型中的 CIDR 驗證比傳統模型中的更為嚴格。 在移轉之前，先確保所提供的傳統位址範圍符合開始移轉之前的有效 CIDR 格式。 CIDR 可以使用任何常見 CIDR 驗證器進行驗證。 移轉時具有無效 CIDR 範圍的 VNet 或本機網站可能會導致失敗的狀態。
> 
> 

## <a name="vnet-to-vnet-connectivity-migration"></a>VNet 對 VNet 連線能力移轉
可藉由建立已連接 VNet 的本機站台表示法來達到傳統 VNet 對 VNet 連線能力。 會要求客戶建立兩個表示兩個需要連接在一起的 Vnet 本機網站。 這些就會使用 IPsec 通道連接至對應 Vnet，來建立兩個 Vnet 之間的連線能力。 此模型具有可管理性的挑戰，因為一個 VNet 中的任何位址範圍變更也必須維持在對應的本機站台表示法。 Resource Manager 模型中不再需要此因應措施。 可以使用連線資源中的 'Vnet2Vnet' 連線類型，直接達成兩個 Vnet 之間的連線。 

![VNet 對 VNet 移轉的螢幕擷取畫面。](./media/vpn-gateway-migration/migration1.png)

在 VNet 移轉期間，我們偵測到目前 VNet 的 VPN 閘道連線實體是另一個 VNet，並確定這兩個 Vnet 移轉完成後，您就不會再看見兩個本機站台代表其他 VNet。 這兩個 VPN 閘道、兩個本機站台和它們之間兩個連線的傳統模式轉換為具有兩個 VPN 閘道和兩個 Vnet2Vnet 類型連接的 Resource Manager 模型。

## <a name="transit-vpn-connectivity"></a>傳輸 VPN 連線能力
您可以在拓撲中設定 VPN 閘道，使 VNet 的內部部署連線能力可藉由連線到直接連線到內部部署的另一個 VNet 來達成。 這是傳輸 VPN 連線能力，其中第一個 VNet 中的執行個體會透過傳輸到直接連接至內部部署之已連接 VNet 中的 VPN 閘道連接到內部部署資源。 若要以傳統部署模型達成這項設定，您必須建立已彙總代表已連接 VNet 和內部部署位址空間的前置詞之本機站台。 接著，這個具像本機站台會連接至 VNet 以達成傳輸連線能力。 這個傳統模型也有類似的管理性挑戰，因為內部部署位址範圍中的任何變更也必須維持代表 VNet 和內部部署彙總的本機網站。 導入 Resource Manager 支援閘道器中的 BGP 支援可簡化管理能力，因為已連線閘道器可以從內部部署了解路由，而無須手動修改前置詞。

![傳輸路由案例的螢幕擷取畫面。](./media/vpn-gateway-migration/migration2.png)

因為我們不需要本機站台即可將 VNet 轉換至 VNet 連線能力，傳輸案例會中斷間接連線到內部部署的 Vnet 內部部署連線能力。 在完成移轉後，可以下列兩種方式降低連線中斷 - 

* 啟用 VPN 閘道上連接在一起及連接至內部部署的 BGP。 啟用 BGP 還原的連線能力而不需要任何其他組態變更，因為路由是在 VNet 閘道之間了解及通告。 請注意，BGP 選項僅在標準和更高的 SKU 才可使用。
* 建立從受影響的 VNet 到代表內部部署位置之區域網路閘道的明確連接。 這也會需要變更內部部署路由器上的設定來建立及設定 IPsec 通道。

## <a name="next-steps"></a>後續步驟
了解 VPN 閘道移轉支援之後，請移至[平台支援的 IaaS 資源移轉 (從傳統移轉至 Resource Manager)](../virtual-machines/windows/migration-classic-resource-manager-ps.md) 以開始使用。

