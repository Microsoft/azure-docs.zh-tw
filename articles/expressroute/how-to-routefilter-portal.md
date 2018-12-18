---
title: 設定 Azure ExpressRoute Microsoft 對等互連的路由篩選：入口網站 | Microsoft Docs
description: 本文說明如何使用 Azure 入口網站針對 Microsoft 對等互連設定路由篩選
documentationcenter: na
services: expressroute
author: ganesr
manager: rossort
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/26/2017
ms.author: ganesr
ms.openlocfilehash: ab0cd45334581f6f5c6dd6e86939e018df5de074
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29387722"
---
# <a name="configure-route-filters-for-microsoft-peering-azure-portal"></a>針對 Microsoft 對等互連設定路由篩選：Azure 入口網站
> [!div class="op_single_selector"]
> * [HttpTrigger Nodejs 函數](how-to-routefilter-portal.md)
> * [Azure PowerShell](how-to-routefilter-powershell.md)
> * [Azure CLI](how-to-routefilter-cli.md)
> 

路由篩選是透過 Microsoft 對等互連使用支援服務子集的方式。 這篇文章中的步驟可協助您設定和管理 ExpressRoute 線路的路由篩選。

透過 Microsoft 對等互連，既可以存取 Dynamics 365 服務和 Office 365 服務 (例如 Exchange Online、SharePoint Online 和商務用 Skype)，也可以存取 Azure 服務 (例如儲存體和 SQL DB)。 當 Microsoft 對等互連在 ExpressRoute 線路中設定時，與這些服務相關的所有前置詞都會透過建立的 BGP 工作階段進行公告。 BGP 社群值附加至每個前置詞，來識別透過前置詞提供的服務。 如需 BGP 社群值和它們對應之服務的清單，請參閱 [BGP 社群](expressroute-routing.md#bgp)。

如果您需要連線到所有服務，則會透過 BGP 公告大量前置詞。 這會大幅增加網路內路由器維護的路由資料表大小。 如果您計劃僅使用透過 Microsoft 對等互連提供的服務子集，您可以用兩種方式減少路由資料表大小。 您可以：

- 在 BGP 社群上套用路由篩選，以篩選不想要的前置詞。 這是標準網路做法，在許多網路經常使用。

- 定義路由篩選，並將其套用到 ExpressRoute 線路。 路由篩選是新的資源，可讓您選取計劃透過 Microsoft 對等互連使用的服務清單。 ExpressRoute 路由器只會傳送屬於路由篩選中所識別之服務的前置詞清單。

### <a name="about"></a>關於路由篩選

當 Microsoft 對等互連在 ExpressRoute 線路上設定時，Microsoft 邊緣路由器會建立一組 BGP 工作階段與邊緣路由器 (您或您的連線提供者)。 沒有路由會公告至您的網路。 若要讓路由公告至您的網路，您必須建立與路由篩選的關聯。

路由篩選可讓您識別想要透過 ExpressRoute 線路的 Microsoft 對等互連使用的服務。 它基本上是 BGP 社群值的允許清單。 一旦定義路由篩選資源，並且連結至 ExpressRoute 線路，對應到 BGP 社群值的所有前置詞都會公告至您的網路。

您必須具有透過 ExpressRoute 使用 Office 365 服務的授權，才能在上面連結路由篩選與 Office 365 服務。 如果您未獲授權透過 ExpressRoute 使用 Office 365 服務，連結路由篩選的作業會失敗。 如需授權程序的詳細資訊，請參閱 [Azure ExpressRoute for Office 365](https://support.office.com/article/Azure-ExpressRoute-for-Office-365-6d2534a2-c19c-4a99-be5e-33a0cee5d3bd)。 Dynamics 365 服務的連線不需要任何先前的授權。

> [!IMPORTANT]
> 在 2017 年 8 月 1 日以前設定之 ExpressRoute 線路的 Microsoft 對等互連會透過 Microsoft 對等互連公告所有服務前置詞，即使未定義路由篩選也一樣。 在 2017 年 8 月 1 日當日或以後設定之 ExpressRoute 線路的 Microsoft 對等互連不會公告任何前置詞，直到路由篩選連結至線路為止。
> 
> 

### <a name="workflow"></a>工作流程

若要能夠成功透過 Microsoft 對等互連連線到服務，您必須完成下列設定步驟：

- 您必須具有已佈建 Microsoft 對等互連的使用中 ExpressRoute 線路。 您可以使用下列指示來完成這些工作：
  - 繼續之前，請[建立 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md)，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建及已啟用的狀態。
  - [建立 Microsoft 對等互連](expressroute-howto-routing-portal-resource-manager.md)，如果您直接管理 BGP 工作階段。 或者，讓您的連線提供者為您的線路佈建 Microsoft 對等互連。

-  您必須建立及設定路由篩選。
    - 識別您透過 Microsoft 對等互連使用的服務
    - 識別與服務相關聯的 BGP 社群值清單
    - 建立規則以允許與 BGP 社群值相符的前置詞清單

-  您必須將路由篩選連結至 ExpressRoute 線路。

## <a name="before-you-begin"></a>開始之前

開始設定之前，請確定符合下列準則：

 - 開始設定之前，請檢閱[必要條件](expressroute-prerequisites.md)和[工作流程](expressroute-workflows.md)。

 - 您必須擁有作用中的 ExpressRoute 線路。 繼續之前，請遵循指示來 [建立 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md) ，並由您的連線提供者來啟用該線路。 ExpressRoute 線路必須處於已佈建及已啟用的狀態。

 - 您必須擁有使用中 Microsoft 對等互連。 請遵循[建立和修改對等互連設定](expressroute-howto-routing-portal-resource-manager.md)的指示


## <a name="prefixes"></a>步驟 1：取得前置詞和 BGP 社群值的清單

### <a name="1-get-a-list-of-bgp-community-values"></a>1.取得 BGP 社群值的清單

與服務相關的 BGP 社群值可透過 [ExpressRoute 路由需求](expressroute-routing.md) 頁面中提供的 Microsoft 對等互連存取。

### <a name="2-make-a-list-of-the-values-that-you-want-to-use"></a>2.製作您想要使用的值清單

製作您想要在路由篩選中使用的 BGP 社群值清單。 例如，Dynamics 365 服務的 BGP 社群值是 12076:5040。

## <a name="filter"></a>步驟 2：建立路由篩選和篩選規則

路由篩選只能有一個規則，且規則的類型必須是 'Allow'。 此規則可以具有與其相關聯的 BGP 社群值清單。

### <a name="1-create-a-route-filter"></a>1.建立路由篩選
您可以選取建立新資源的選項來建立路由篩選。 按一下 [建立資源]  >  [網路]  >  [RouteFilter]，如下圖所示：

![建立路由篩選](.\media\how-to-routefilter-portal\CreateRouteFilter1.png)

您必須在資源群組中放置路由篩選。 

![建立路由篩選](.\media\how-to-routefilter-portal\CreateRouteFilter.png)

### <a name="2-create-a-filter-rule"></a>2.建立篩選規則

您可以為路由篩選選取管理規則索引標籤，以新增及更新規則。

![建立路由篩選](.\media\how-to-routefilter-portal\ManageRouteFilter.png)


您可以從下拉式清單選取您想要連線的服務，並且在完成時儲存規則。

![建立路由篩選](.\media\how-to-routefilter-portal\AddRouteFilterRule.png)


## <a name="attach"></a>步驟 3：將路由篩選連接到 ExpressRoute 線路

可以選取 [新增線路] 按鈕，並從下拉式清單選取 [ExpressRoute] 線路，以將路由篩選附加至線路。

![建立路由篩選](.\media\how-to-routefilter-portal\AddCktToRouteFilter.png)

如果連線提供者為您的 ExpressRoute 線路設定對等互連，請先重新整理 ExpressRoute 線路刀鋒視窗的線路，再選取 [新增線路] 按鈕。

![建立路由篩選](.\media\how-to-routefilter-portal\RefreshExpressRouteCircuit.png)

## <a name="tasks"></a>常見工作

### <a name="getproperties"></a>若要取得路由篩選的屬性

當您在入口網站中開啟資源時，可以檢視路由篩選的屬性。

![建立路由篩選](.\media\how-to-routefilter-portal\ViewRouteFilter.png)


### <a name="updateproperties"></a>若要更新路由篩選的屬性

可以藉由選取 [管理規則] 按鈕，更新附加至線路之 BGP 社群值的清單。


![建立路由篩選](.\media\how-to-routefilter-portal\ManageRouteFilter.png)

![建立路由篩選](.\media\how-to-routefilter-portal\AddRouteFilterRule.png) 


### <a name="detach"></a>若要從 ExpressRoute 線路取消連結路由篩選

若要從路由篩選器將電路中斷連結，請以滑鼠右鍵按一下電路並按一下「取消關聯」。

![建立路由篩選](.\media\how-to-routefilter-portal\DetachRouteFilter.png) 


### <a name="delete"></a>若要刪除路由篩選

可以選取 [刪除] 按鈕來刪除路由篩選。 

![建立路由篩選](.\media\how-to-routefilter-portal\DeleteRouteFilter.png) 

## <a name="next-steps"></a>後續步驟

如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。