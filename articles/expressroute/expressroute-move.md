---
title: 將 ExpressRoute 線路從傳統部署模型移至 Resource Manager | Microsoft Docs
description: 此頁面提供銜接傳統與 Resource Manager 部署模型的相關概觀。
documentationcenter: na
services: expressroute
author: ganesr
manager: carmonm
editor: ''
ms.assetid: bdf01217-1a98-4ec0-a08e-d84fd37f78af
ms.service: expressroute
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/02/2017
ms.author: ganesr
ms.openlocfilehash: 7f8386b518ada850fc03e23c5cae3b159b3b213e
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "22710060"
---
# <a name="moving-expressroute-circuits-from-the-classic-to-the-resource-manager-deployment-model"></a>將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型
本文提供將 Azure ExpressRoute 線路從傳統部署模型移至 Azure Resource Manager 部署模型的概觀。

您可以使用單一 ExpressRoute 線路來連接到在傳統和 Resource Manager 部署模型中部署的虛擬網路。 無論 ExpressRoute 線路的建立方式為何，現在都可以連結至兩種部署模型中的虛擬網路。

![連結至兩種部署模型中虛擬網路的 ExpressRoute 線路。](./media/expressroute-move/expressroute-move-1.png)

## <a name="expressroute-circuits-that-are-created-in-the-classic-deployment-model"></a>在傳統部署模型中建立的 ExpressRoute 線路
在傳統部署模型中建立的 ExpressRoute 線路必須先移至 Resource Manager 部署模型，才能連接至傳統和 Resource Manager 部署模型。 移動連線時，不會發生連線中斷的情況。 會保留傳統部署模型中的所有線路與虛擬網路的連結 (在相同訂用帳戶內和跨越訂用帳戶)。

順利完成移動後，ExpressRoute 線路的外觀、執行及操作方式與在 Resource Manager 部署模型中建立的 ExpressRoute 線路完全相似。 您現在可以在 Resource Manager 部署模型中建立虛擬網路的連線。

在 ExpressRoute 線路移至 Resource Manager 部署模型後，您可以利用 Resource Manager 部署模型來管理 ExpressRoute 線路的生命週期。 這表示您只能在 Resource Manager 部署模型中執行某些作業，如新增/更新/刪除對等、更新線路屬性 (如頻寬、SKU 和計費類型)，以及刪除線路。 如需進一步得知如何管理這兩種部署模型的存取權，請先參閱下方的章節，以了解於 Resource Manager 部署模型中所建立的線路。

您不必牽連您的連線提供者，即可執行移動。

## <a name="expressroute-circuits-that-are-created-in-the-resource-manager-deployment-model"></a>在 Resource Manager 部署模型中建立的 ExpressRoute 線路
您可讓在 Resource Manager 部署模型中建立的 ExpressRoute 線路能夠從這兩種部署模型進行存取。 您訂用帳戶中的 ExpressRoute 線路均可從這兩種部署模型進行存取。

* 根據預設，在 Resource Manager 部署模型中建立的 ExpressRoute 線路無法存取傳統部署模型。
* 根據預設，無法從這兩種部署模型存取從傳統部署模型移至 Resource Manager 部署模型的 ExpressRoute 線路。
* 無論是建立於 Resource Manager 部署模型或傳統部署模型中，ExpressRoute 線路永遠都可以存取 Resource Manager 部署模型。 這表示您可以依照 [如何連結虛擬網路](expressroute-howto-linkvnet-arm.md)中的指示執行，為建立於 Resource Manager 部署模型中的虛擬網路建立連線。
* 傳統部署模型的存取權是由 ExpressRoute 線路中的 **allowClassicOperations** 參數所控制。

> [!IMPORTANT]
> 會套用 [服務限制](../azure-subscription-service-limits.md) 頁面上記載的所有配額。 例如，標準線路最多可以有 10 個跨傳統與 Resource Manager 部署模型的虛擬網路連結/連線。
> 
> 

## <a name="controlling-access-to-the-classic-deployment-model"></a>控制傳統部署模型的存取權
設定 ExpressRoute 線路的 **allowClassicOperations** 參數，即可讓單一 ExpressRoute 線路連結至這兩種部署模型中的虛擬網路。

將 **allowClassicOperations** 設定為 TRUE，可讓您從這兩種部署模型中的虛擬網路連結到 ExpressRoute 線路。 遵循 [如何連結傳統部署模型中的虛擬網路](expressroute-howto-linkvnet-classic.md)上的指導方針，即可連結至傳統部署模型中的虛擬網路。 遵循 [如何連結 Resource Manager 部署模型中的虛擬網路](expressroute-howto-linkvnet-arm.md)上的指導方針，即可連結至 Resource Manager 部署模型中的虛擬網路。

將 **allowClassicOperations** 設定為 FALSE，就會封鎖傳統部署模型中線路的存取。 不過，會保留傳統部署模型中的所有虛擬網路連結。 在此情況下，ExpressRoute 線路不會顯示在傳統部署模型中。

## <a name="supported-operations-in-the-classic-deployment-model"></a>傳統部署模型中支援的作業
當 **allowClassicOperations** 設定為 TRUE 時，ExpressRoute 線路支援下列傳統作業。

* 取得 ExpressRoute 線路資訊
* 建立/更新/取得/刪除對傳統虛擬網路的虛擬網路連結
* 建立/更新/取得/刪除跨訂用帳戶連線的虛擬網路連結授權

當 **allowClassicOperations** 設定為 TRUE 時，您無法執行下列傳統作業：

* 建立/更新/取得/刪除 Azure 私人、Azure 公用和 Microsoft 對等的邊界閘道協定 (BGP) 對等
* 刪除 ExpressRoute 線路

## <a name="communication-between-the-classic-and-the-resource-manager-deployment-models"></a>傳統與 Resource Manager 部署模型之間的通訊
ExpressRoute 線路的作用就像是傳統與 Resource Manager 部署模型之間的橋樑。 如果傳統部署模型的虛擬網路中的虛擬機器與 Resource Manager deployment 部署模型的虛擬網路中的虛擬機器連結至相同的 ExpressRoute 線路，這兩種虛擬機器之間的流量會流經 ExpressRoute。

彙總輸送量受限於虛擬網路閘道的輸送量容量。 在這類情況下，流量不會進入連線提供者的網路或您的網路。 虛擬網路之間的流量將會完全包含在 Microsoft 網路內。

## <a name="access-to-azure-public-and-microsoft-peering-resources"></a>Azure 公用和 Microsoft 對等資源的存取權
您可以繼續存取通常可透過 Azure 公用對等和 Microsoft 對等存取的資源，而不需要中斷。  

## <a name="whats-supported"></a>支援的項目
本節說明 ExpressRoute 線路會支援的功能：

* 您可以使用單一 ExpressRoute 線路來存取在傳統和 Resource Manager 部署模型中部署的虛擬網路。
* 您可以將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型。 移動之後，ExpressRoute 線路的外觀、操作及執行方式會與在 Resource Manager 部署模型中建立的任何其他 ExpressRoute 線路相似。
* 您只可以移動 ExpressRoute 線路。 線路連結、虛擬網路和 VPN 閘道無法透過這項作業移動。
* 在 ExpressRoute 線路移至 Resource Manager 部署模型後，您可以利用 Resource Manager 部署模型來管理 ExpressRoute 線路的生命週期。 這表示您只能在 Resource Manager 部署模型中執行某些作業，如新增/更新/刪除對等、更新線路屬性 (如頻寬、SKU 和計費類型)，以及刪除線路。
* ExpressRoute 線路的作用就像是傳統與 Resource Manager 部署模型之間的橋樑。 如果傳統部署模型的虛擬網路中的虛擬機器與 Resource Manager deployment 部署模型的虛擬網路中的虛擬機器連結至相同的 ExpressRoute 線路，這兩種虛擬機器之間的流量會流經 ExpressRoute。
* 傳統和 Resource Manager 部署模型中都支援跨訂用帳戶連線。
* 在您將 ExpressRoute 線路從傳統模型移至 Azure Resource Manager 模型之後，您可以[移轉已連結至 ExpressRoute 線路的虛擬網路](expressroute-migration-classic-resource-manager.md)。

## <a name="whats-not-supported"></a>不支援的內容
本節說明 ExpressRoute 線路不會支援的功能：

* 從傳統部署模型管理 ExpressRoute 線路的生命週期。
* 傳統部署模型的角色型存取控制 (RBAC) 支援。 您無法對傳統部署模型中的線路執行 RBAC 控制。 訂用帳戶的所有管理員/共同管理員都可以將虛擬網路連結至線路或取消其連結。

## <a name="configuration"></a>組態
請遵循 [將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型](expressroute-howto-move-arm.md)中所描述的指示。

## <a name="next-steps"></a>後續步驟
* [將已連結至 ExpressRoute 線路的虛擬網路從傳統模型移轉至 Azure Resource Manager 模型](expressroute-migration-classic-resource-manager.md)
* 如需工作流程資訊，請參閱 [ExpressRoute 線路佈建工作流程和線路狀態](expressroute-workflows.md)。
* 若要設定 ExpressRoute 連線：
  
  * [建立 ExpressRoute 線路](expressroute-howto-circuit-arm.md)
  * [設定路由](expressroute-howto-routing-arm.md)
  * [將虛擬網路連結到 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)

