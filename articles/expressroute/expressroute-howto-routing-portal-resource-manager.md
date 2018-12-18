---
title: 如何設定 ExpressRoute 線路的路由 (對等互連)：Resource Manager：Azure | Microsoft Docs
description: 本文將逐步引導您為 ExpressRoute 線路建立和佈建私用、公用及 Microsoft 對等。 本文也示範如何檢查狀態、更新或刪除線路的對等。
documentationcenter: na
services: expressroute
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: 8c2a7ed2-ae5c-4e49-81f6-77cf9f2b2ac9
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/12/2018
ms.author: cherylmc
ms.openlocfilehash: f0f0a31abc4e2d3114d71729c6c447c569295290
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30314025"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit"></a>建立和修改 ExpressRoute 線路的對等互連

本文將協助您使用 Azure 入口網站，在 Resource Manager 部署模型中建立和管理 ExpressRoute 線路的路由設定。 您還可以檢查狀態、更新，或是刪除與取消佈建 ExpressRoute 線路的對等互連。 如果您想要對線路使用不同的方法，可選取下列清單中的文章：

> [!div class="op_single_selector"]
> * [Azure 入口網站](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [視訊 - 私用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [視訊 - 公用對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [視訊 - Microsoft 對等互連](http://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (傳統)](expressroute-howto-routing-classic.md)
> 


## <a name="configuration-prerequisites"></a>組態必要條件

* 開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)頁面、[路由需求](expressroute-routing.md)頁面和[工作流程](expressroute-workflows.md)頁面。
* 您必須擁有作用中的 ExpressRoute 線路。 繼續之前，請遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md) ，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建和已啟用狀態，您才能執行下一節中的 Cmdlet。
* 如果您打算使用共用金鑰/MD5 雜湊，請務必將它使用於通道的兩端，並將字元數目限制為最多 25 個。

這些指示只適用於由提供第 2 層連線服務的服務提供者所建立的線路。 如果您使用的服務提供者提供受控第 3 層服務 (通常是 IPVPN，如 MPLS)，連線提供者會為您設定和管理路由。 

> [!IMPORTANT]
> 我們目前不會透過服務管理入口網站來公告服務提供者所設定的對等。 我們正努力在近期推出這項功能。 設定 BGP 對等互連之前，請洽詢您的服務提供者。
> 
> 

您可以為 ExpressRoute 線路設定一個、兩個或全部三個對等 (Azure 私用、Azure 公用和 Microsoft)。 您可以依自己選擇的任何順序設定對等。 不過，您必須確定一次只完成一個對等的設定。 如需路由網域和對等互連的詳細資訊，請參閱 [ExpressRoute 路由網域](expressroute-circuit-peerings.md)。

## <a name="msft"></a>Microsoft 對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Microsoft 對等互連設定。

> [!IMPORTANT]
> 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連，會透過 Microsoft 對等互連公告所有服務首碼，即使未定義路由篩選也一樣。 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連，不會公告任何首碼，直到路由篩選連結至線路為止。 如需詳細資訊，請參閱[設定 Microsoft 對等互連的路由篩選](how-to-routefilter-powershell.md)。
> 
> 

### <a name="to-create-microsoft-peering"></a>建立 Microsoft 對等

1. 設定 ExpressRoute 電路。 先確定連線提供者已完整佈建電路，再繼續作業。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Microsoft 對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

  ![列出 Microsoft 對等互連](./media/expressroute-howto-routing-portal-resource-manager/listprovisioned.png)
2. 設定線路的 Microsoft 對等。 繼續之前，請確定您擁有下列資訊。

  * 主要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是您所擁有且註冊在 RIR / IRR 中的有效公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * 公告的首碼：您必須提供一份您打算在 BGP 工作階段上公告的所有首碼的清單。 只接受公用 IP 位址首碼。 如果計劃傳送一組首碼，可以傳送以逗號分隔的清單。 這些首碼必須在 RIR / IRR 中註冊給您。
  * **選用：** 客戶 ASN：如果您要公告的首碼未註冊給對等互連 AS 編號，則可以指定它們所註冊的 AS 編號。
  * 路由登錄名稱：您可以指定可供註冊 AS 編號和首碼的 RIR / IRR。
  * **選用：** MD5 雜湊 (如果選擇使用)。
3. 如下列範例所示，您可以選取要設定的對等互連。 選取 Microsoft 對等資料列。

  ![選取 Microsoft 對等互連列](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft1.png)
4. 設定 Microsoft 對等。 下圖顯示設定範例：

  ![設定 Microsoft 對等互連](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft2.png)
5. 指定所有參數後，請儲存組態。

  如果您的線路出現「需要驗證」狀態 (如下圖所示)，您必須開立支援票證，向我們的支援小組出示首碼擁有權的證明。

  ![儲存 Microsoft 對等互連設定](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft5.png)

  如下列範例所示，您可以直接從入口網站開立支援票證：

  ![](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft6.png)


1. 成功接受設定後，會出現類似下圖的畫面：

  ![](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft7.png)

### <a name="getmsft"></a>檢視 Microsoft 對等詳細資訊

選取 Azure 公用對等，即可檢視該對等的屬性。

![](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft3.png)

### <a name="updatemsft"></a>更新 Microsoft 對等組態

您可以選取對等的資料列並修改對等屬性。

![](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft7.png)

### <a name="deletemsft"></a>刪除 Microsoft 對等

如下圖所示，您可以選取刪除圖示來移除對等互連設定：

![](./media/expressroute-howto-routing-portal-resource-manager/rmicrosoft4.png)

## <a name="private"></a>Azure 私人對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 私用對等互連設定。

### <a name="to-create-azure-private-peering"></a>建立 Azure 私用對等

1. 設定 ExpressRoute 電路。 先確定連線提供者已完整佈建電路，再繼續作業。 若您的連線提供者是提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 私人對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

  ![list](./media/expressroute-howto-routing-portal-resource-manager/listprovisioned.png)
2. 設定線路的 Azure 私用對等。 繼續執行接下來的步驟之前，請確定您有下列項目：

  * 主要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 次要連結的 /30 子網路。 子網路不能在保留給虛擬網路的任何位址空間中。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。 您可以將私用 AS 編號用於此對等。 請確定您不是使用 65515。
  * **選用：** MD5 雜湊 (如果選擇使用)。
3. 如下列範例所示，選取 Azure 私用對等互連列：

  ![私用](./media/expressroute-howto-routing-portal-resource-manager/rprivate1.png)
4. 設定私用對等。 下圖顯示設定範例：

  ![設定私用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rprivate2.png)
5. 指定所有參數後，請儲存組態。 成功接受設定後，會出現類似下列範例的畫面：

  ![儲存私用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rprivate3.png)

### <a name="getprivate"></a>檢視 Azure 私用對等詳細資訊

選取 Azure 私用對等，即可檢視該對等的屬性。

![檢視私用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rprivate3.png)

### <a name="updateprivate"></a>更新 Azure 私用對等組態

您可以選取對等的資料列並修改對等屬性。

![更新私用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rprivate2.png)

### <a name="deleteprivate"></a>刪除 Azure 私用對等

如下圖所示，您可以選取刪除圖示來移除對等互連設定：

![刪除私用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rprivate4.png)

## <a name="public"></a>Azure 公用對等互連

本節將協助您為 ExpressRoute 線路建立、取得、更新和刪除 Azure 公用對等互連設定。

### <a name="to-create-azure-public-peering"></a>建立 Azure 公用對等

1. 設定 ExpressRoute 電路。 先確定連線提供者已完整佈建電路，再繼續作業。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您啟用 Azure 公用對等互連。 在此情況下，您不需要遵循後續幾節所列的指示。 不過，如果您的連線提供者不管理路由，請在建立線路之後繼續使用後續步驟進行設定。

  ![列出公用對等互連](./media/expressroute-howto-routing-portal-resource-manager/listprovisioned.png)
2. 設定線路的 Azure 公用對等。 繼續執行接下來的步驟之前，請確定您有下列項目：

  * 主要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 次要連結的 /30 子網路。 這必須是有效的公用 IPv4 首碼。
  * 供建立此對等的有效 VLAN ID。 請確定線路有沒有其他對等使用相同的 VLAN ID。
  * 對等的 AS 編號。 您可以使用 2 位元組和 4 位元組 AS 編號。
  * **選用：** MD5 雜湊 (如果選擇使用)。
3. 如下圖所示，選取 Azure 公用對等互連列：

  ![選取公用對等互連列](./media/expressroute-howto-routing-portal-resource-manager/rpublic1.png)
4. 設定公用對等。 下圖顯示設定範例：

  ![設定公用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rpublic2.png)
5. 指定所有參數後，請儲存組態。 成功接受設定後，會出現類似下列範例的畫面：

  ![儲存公用對等互連設定](./media/expressroute-howto-routing-portal-resource-manager/rpublic3.png)

### <a name="getpublic"></a>檢視 Azure 公用對等詳細資訊

選取 Azure 公用對等，即可檢視該對等的屬性。

![檢視公用對等互連屬性](./media/expressroute-howto-routing-portal-resource-manager/rpublic3.png)

### <a name="updatepublic"></a>更新 Azure 公用對等組態

您可以選取對等的資料列並修改對等屬性。

![選取公用對等互連列](./media/expressroute-howto-routing-portal-resource-manager/rpublic2.png)

### <a name="deletepublic"></a>刪除 Azure 公用對等

如下列範例所示，您可以選取刪除圖示來移除對等互連設定：

![刪除公用對等互連](./media/expressroute-howto-routing-portal-resource-manager/rpublic4.png)

## <a name="next-steps"></a>後續步驟

下一步，[將 VNet 連結到 ExpressRoute 線路](expressroute-howto-linkvnet-portal-resource-manager.md)
* 如需 ExpressRoute 工作流程的詳細資訊，請參閱 [ExpressRoute 工作流程](expressroute-workflows.md)。
* 如需線路對等的詳細資訊，請參閱 [ExpressRoute 線路和路由網域](expressroute-circuit-peerings.md)。
* 如需使用虛擬網路的詳細資訊，請參閱 [虛擬網路概觀](../virtual-network/virtual-networks-overview.md)。
