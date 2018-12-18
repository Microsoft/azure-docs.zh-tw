---
title: 建立與修改 ExpressRoute 線路：Azure 入口網站 | Microsoft Docs
description: 本文說明如何建立、佈建、驗證、更新、刪除和取消佈建 ExpressRoute 線路。
documentationcenter: na
services: expressroute
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: 68d59d59-ed4d-482f-9cbc-534ebb090613
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/20/2017
ms.author: cherylmc;ganesr
ms.openlocfilehash: be97f441d6eb2e97d85d598b0ed66718aec97b14
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29389507"
---
# <a name="create-and-modify-an-expressroute-circuit"></a>建立和修改 ExpressRoute 線路
> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [影片 - Azure 入口網站](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-circuit-classic.md)
>

此文章說明如何使用 Azure 入口網站和 Azure Resource Manager 部署模型建立 Azure ExpressRoute 線路。 下列步驟也會示範如何檢查線路的狀態、對它進行更新，或是對它進行刪除及取消佈建。


## <a name="before-you-begin"></a>開始之前
* 開始設定之前，請檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。
* 確定您可以存取 [Azure 入口網站](https://portal.azure.com)。
* 確定您具有建立新網路資源的權限。 如果您沒有正確的權限，請連絡您的帳戶管理員。
* 您可以在開始前先[觀賞影片](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-an-expressroute-circuit)，以便更加了解步驟。

## <a name="create"></a>建立和佈建 ExpressRoute 線路
### <a name="1-sign-in-to-the-azure-portal"></a>1.登入 Azure 入口網站
透過瀏覽器瀏覽至 [Azure 入口網站](http://portal.azure.com) ，並使用您的 Azure 帳戶登入。

### <a name="2-create-a-new-expressroute-circuit"></a>2.建立新的 ExpressRoute 線路
> [!IMPORTANT]
> ExpressRoute 線路會從發出服務金鑰時開始收費。 請確定在連線提供者準備好佈建線路之後，再執行這項作業。
> 
> 

1. 您可以選取建立新資源的選項來建立 ExpressRoute 線路。 按一下 [建立資源] > [網路] > [ExpressRoute]，如下圖所示：

  ![建立 ExpressRoute 線路](./media/expressroute-howto-circuit-portal-resource-manager/createcircuit1.png)
2. 按一下 [ExpressRoute] 之後，將會看到 [建立 ExpressRoute 線路] 頁面。 在此頁面中填入值時，請確認所指定的 SKU 層 ([標準] 或 [進階]) 以及資料計量計費模型 ([無限制] 或 [計量付費]) 正確無誤。

  ![設定 SKU 層和資料計量](./media/expressroute-howto-circuit-portal-resource-manager/createcircuit.png)

  *  決定要啟用 ExpressRoute 標準或 ExpressRoute 進階附加元件。 您可以指定 [標準] 來取得標準 SKU，或指定 [進階] 來取得進階附加元件。
  *  決定計費類型。 您可以指定 [計量付費] 以採用計量付費數據傳輸方案，選取 [無限制] 以採用無限制的資料方案。 請注意，您可以將計費類型從 [計量付費] 變更為 [無限制]，但無法從 [無限制] 變更為 [計量付費]。
  * [對等位置] 是您與 Microsoft 對等互連的實體位置。

    > [!IMPORTANT]
    > [對等位置] 表示您與 Microsoft 對等互連的[實體位置](expressroute-locations.md)。 這**不會**連結到「位置」屬性，這是指 Azure 網路資源提供者所在的地理位置。 儘管它們並無關聯，但最好還是選擇地理位置靠近線路對等位置的網路資源提供者。
    >
    >

### <a name="3-view-the-circuits-and-properties"></a>3.檢視線路和屬性
**檢視所有線路**

您可以選取左側功能表上的 [所有資源] 來檢視您建立的所有線路。

![檢視線路](./media/expressroute-howto-circuit-portal-resource-manager/listresource.png)

**檢視屬性**

您可選取線路檢視其屬性。 在線路的 [概觀] 頁面上，服務金鑰會顯示於服務金鑰欄位。 您必須複製線路的服務金鑰，並將其向下傳遞至服務提供者以完成佈建流程。 此線路服務金鑰係專屬於您的線路。

![檢視屬性](./media/expressroute-howto-circuit-portal-resource-manager/servicekey1.png)

### <a name="4-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>4.將服務金鑰傳送給連線提供者以進行佈建
在此頁面上，[提供者狀態] 會提供服務提供者目前佈建狀態的相關資訊。 [線路狀態] 提供 Microsoft 端的狀態。 如需線路佈建狀態的詳細資訊，請參閱 [工作流程](expressroute-workflows.md#expressroute-circuit-provisioning-states) 文章。

當您建立新的 ExpressRoute 線路時，線路會是下列狀態：

提供者狀態︰未佈建<BR>
線路狀態：已啟用

![起始佈建程序](./media/expressroute-howto-circuit-portal-resource-manager/status.png)

當連線提供者正在為您啟用線路時，線路會變更為下列狀態：

提供者狀態︰正在佈建<BR>
線路狀態：已啟用

若要能夠使用 ExpressRoute 線路，它必須處於下列狀態：

提供者狀態︰已佈建<BR>
線路狀態：已啟用

### <a name="5-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>5.定期檢查線路金鑰的狀態與狀況
選取感興趣的線路，即可檢視該線路的屬性。 檢查 [提供者狀態]，確定它已變成 [已佈建] 再繼續。

![線路和提供者狀態](./media/expressroute-howto-circuit-portal-resource-manager/provisioned.png)

### <a name="6-create-your-routing-configuration"></a>6.建立路由組態
如需逐步指示，請參閱 [ExpressRoute 線路路由組態](expressroute-howto-routing-portal-resource-manager.md)一文以建立和修改線路對等。

> [!IMPORTANT]
> 這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者是提供受控第 3 層服務 (通常是 IP VPN，如 MPLS)，您的連線提供者會為您設定和管理路由。
> 
> 

### <a name="7-link-a-virtual-network-to-an-expressroute-circuit"></a>7.將虛擬網路連結到 ExpressRoute 線路
接下來，將虛擬網路連結到 ExpressRoute 線路。 當使用 Resource Manager 部署模型時，使用[將虛擬網路連結到 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)文章。

## <a name="status"></a>取得 ExpressRoute 線路的狀態
您可選取線路並檢視 [概觀] 頁面，以檢視該線路的狀態。 

## <a name="modify"></a>修改 ExpressRoute 線路
您可以修改 ExpressRoute 線路的某些屬性，而不會影響連線。 您可以在 [組態] 頁面中修改頻寬、SKU、計費模型並允許傳統作業。 如需限制的相關資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。 

您可以執行下列工作，而無須中途停機：

* 啟用或停用 ExpressRoute 線路的 ExpressRoute 進階附加元件。
* 只要連接埠有可用的容量，即增加 ExpressRoute 線路的頻寬。 不支援將線路的頻寬降級。 
* 將計量方案從 [已計量資料] 變更為 [無限制資料]。 不支援將計量方案從 [無限制資料] 變更為 [已計量資料]。
* 您可以啟用和停用 [允許傳統作業] 。

> [!IMPORTANT]
> 如果現有的連接埠上沒有足夠的容量，您可能必須重新建立 ExpressRoute 線路。 如果該位置已無額外的容量，您無法升級線路。
>
> 雖然您可以順暢升級頻寬，但降低 ExpressRoute 線路的頻寬時必須中斷運作。 頻寬降級需要取消佈建 ExpressRoute 線路，然後重新佈建新的 ExpressRoute 線路。
> 
> 若您使用的資源超出標準線路所允許的數量，可能會無法停用進階附加元件作業。
> 
> 

若要修改 ExpressRoute 線路，請按一下 [組態]。

![修改線路](./media/expressroute-howto-circuit-portal-resource-manager/modifycircuit.png)




## <a name="delete"></a>取消佈建和刪除 ExpressRoute 線路
您可以選取**刪除**圖示，刪除 ExpressRoute 線路。 請注意下列資訊︰

* 您必須取消連結 ExpressRoute 線路的所有虛擬網路。 如果此操作失敗，請檢查您是否有任何虛擬網路連結至線路。
* 如果 ExpressRoute 線路服務提供者佈建狀態為 **Provisioning** 或 **Provisioned**，您就必須與服務提供者一起合作，取消佈建他們那邊的線路。 我們會繼續保留資源並向您收取費用，直到線路服務提供者完成取消佈建並通知我們。
* 若服務提供者已取消佈建線路 (服務提供者佈建狀態設定為 [未佈建])，則可以刪除線路。 這樣會停止針對線路計費。

## <a name="next-steps"></a>後續步驟
建立線路後，繼續執行下列步驟：

* [建立和修改 ExpressRoute 線路的路由](expressroute-howto-routing-portal-resource-manager.md)
* [將虛擬網路連結至 ExpressRoute 線路](expressroute-howto-linkvnet-arm.md)
