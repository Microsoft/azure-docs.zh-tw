---
title: 使用 Azure 虛擬 WAN 建立與 Azure 和內部部署環境的 ExpressRoute 連線 | Microsoft Docs
description: 在本教學課程中，深入了解如何使用 Azure 虛擬 WAN 建立與 Azure 和內部部署環境的 ExpressRoute 連線。
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: tutorial
ms.date: 10/5/2018
ms.author: cherylmc
Customer intent: As someone with a networking background, I want to connect my corporoate on-premises network(s) to my VNets using Virtual WAN and ExpressRoute.
ms.openlocfilehash: 10b73843e2ca2e0ee0af766ef1ad7ee9cbf61f94
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854591"
---
# <a name="tutorial-create-an-expressroute-association-using-azure-virtual-wan-preview"></a>教學課程：使用 Azure 虛擬 WAN 來建立 ExpressRoute 關聯 (預覽)

本教學課程會示範如何使用虛擬 WAN，透過使用 ExpressRoute 線路和關聯來與 Azure 中的資源連線。 如需有關虛擬 WAN 的詳細資訊，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立 vWAN
> * 建立中樞
> * 尋找線路並將其關聯至中樞
> * 將線路關聯至中樞
> * 將 VNet 連線至中樞
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

> [!IMPORTANT]
> 此公開預覽版是在沒有服務等級協定的情況下提供，不得用於生產工作負載。 可能不支援特定功能、可能已經限制功能，或者可能無法在所有 Azure 位置提供使用。 如需詳細資訊，請參閱 [Microsoft Azure 預覽專用的補充使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。
>

## <a name="before-you-begin"></a>開始之前

[!INCLUDE [Before you begin](../../includes/virtual-wan-tutorial-vwan-before-include.md)]

## <a name="register"></a>註冊此功能

設定虛擬 WAN 之前，您必須先在預覽版中註冊訂用帳戶。 否則，您將無法在入口網站中使用虛擬 WAN。 若要註冊，您可以傳送電子郵件與訂用帳戶識別碼至 **azurevirtualwan@microsoft.com**。 當訂用帳戶註冊完成之後，您會收到電子郵件。

**預覽考量：**

* 區域可用性：美國中西部
* 必須在支援 [ExpressRoute Global Reach](https://docs.microsoft.com/azure/expressroute/expressroute-faqs#where-is-expressroute-global-reach-supported) 的國家/地區中啟用 ExpressRoute 線路

## <a name="vnet"></a>1.建立虛擬網路

[!INCLUDE [Create a virtual network](../../includes/virtual-wan-tutorial-vnet-include.md)]

## <a name="openvwan"></a>2.建立虛擬 WAN

透過瀏覽器瀏覽至 [Azure 入口網站 (預覽)](http://aka.ms/azurevirtualwanpreviewfeatures)，並使用您的 Azure 帳戶登入。

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-vwan-include.md)]

### <a name="getting-started-page"></a>開始使用頁面

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-gettingstarted-include.md)]

## <a name="hub"></a>3.建立中樞

[!INCLUDE [Create a virtual WAN](../../includes/virtual-wan-tutorial-hub-include.md)]

## <a name="hub"></a>4.尋找線路並將其關聯至中樞

1. 選取您的 vWAN，並且在 [虛擬 WAN 架構] 底下選取 [ExpressRoute 線路]
2. 如果 ExpressRoute 線路位在與您 vWAN 相同的訂用帳戶中，請從訂用帳戶按一下 [選取 ExpressRoute 線路] 
3. 使用下拉式清單，選取您要關聯至中樞的 ExpressRoute。
4. 如果 ExpressRoute 線路不是在相同的訂用帳戶中，或是您已提供[授權金鑰和對等互連識別碼](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md)，請選取 [尋找線路來兌換授權金鑰]
5. 輸入下列詳細資料：
* **授權金鑰** - 由線路擁有者產生，如上所述
* **對等互連線路 URI** - 由線路擁有者所提供的線路 URI，是線路的唯一識別碼
* **路由權重** - 當有來自不同對等互連位置的多個線路連線至相同中樞時，[路由權重](../expressroute/expressroute-optimize-routing.md)可讓您使用偏好的特定路徑
6. 按一下 [尋找線路]，在找到時選取線路
7. 從下拉式清單中選取 1 或多個中樞，然後按一下 [儲存]

## <a name="vnet"></a>5.將 VNet 連線至中樞

在此步驟中，您會在中樞和 VNet 之間建立等互連的連線。 為您想要連線的每個 VNet 重複這些步驟。

1. 在您的虛擬 WAN 頁面上，按一下 [虛擬網路連線]。
2. 在 [虛擬網路連線] 頁面上，按一下 [+ 新增連線]。
3. 在 [新增連線] 頁面上，填寫下列欄位︰

    * **名稱** - 為您的連線命名。
    * **中樞** - 選取要與此連線產生關聯的中樞。
    * **訂用帳戶** - 請確認訂用帳戶。
    * **虛擬網路** - 選取要與此中樞連線的虛擬網路。 虛擬網路不能有現有的虛擬網路閘道。


## <a name="viewwan"></a>6.檢視您的虛擬 WAN

1. 瀏覽至虛擬 WAN。
2. 在 [概觀] 頁面中，地圖上的每個點都代表著中樞。 暫留在任一點上，即可檢視中樞健康情況摘要。
3. 在 [中樞與連線] 區段中，您可以檢視中樞狀態、網站、區域、VPN 連線狀態和輸入與輸出位元組。

## <a name="viewhealth"></a>7.檢視資源健康情況

1. 瀏覽至您的 WAN。
2. 在 WAN 頁面上的 [支援 + 疑難排解] 區段中，按一下 [健康情況] 並檢視您的資源。

## <a name="connectmon"></a>8.監視連線

建立連線以監視 Azure VM 和遠端站台之間的通訊。 如需有關如何設定連線監視的資訊，請參閱[監視網路通訊](~/articles/network-watcher/connection-monitor.md)。 來源欄位是 Azure 中的 VM IP，目的地 IP 是位址是網站 IP。

## <a name="cleanup"></a>9.清除資源

您可以使用 [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) 來移除不再需要的資源群組，以及其所包含的所有資源。 將 "myResourceGroup" 取代為您的資源群組名稱，然後執行下列 PowerShell 命令：

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立 vWAN
> * 建立中樞
> * 尋找線路並將其關聯至中樞
> * 將線路關聯至中樞
> * 將 VNet 連線至中樞
> * 檢視您的虛擬 WAN
> * 檢視資源健康情況
> * 監視連線

若要深入了解虛擬 WAN，請參閱[虛擬 WAN 概觀](virtual-wan-about.md)頁面。
