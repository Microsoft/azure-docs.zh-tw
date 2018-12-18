---
title: 將 Azure ExpressRoute 上的公用對等互連移至 Microsoft 對等互連 | Microsoft Docs
description: 本文會示範在 ExpressRoute 中將公用對等互連移至 Microsoft 對等互連的步驟。
services: expressroute
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/12/2018
ms.author: cherylmc
ms.openlocfilehash: f34fabc95d5b56edc6e37c323bebf60bd98c8b90
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/03/2018
ms.locfileid: "30314294"
---
# <a name="move-a-public-peering-to-microsoft-peering"></a>將公用互連移至 Microsoft 對等互連

ExpressRoute 支援使用 Microsoft 對等互連與路由篩選進行 Azure PaaS 服務，例如 Azure 儲存體和 Azure SQL Database。 現在存取 Microsoft PaaS 和 SaaS 服務只需要一個路由網域。 您可以使用路由篩選，選擇性地公告您想要使用的 Azure 區域 PaaS 服務首碼。

本文可協助您將公用對等設定移至 Microsoft 對等互連，不需要停機。 如需路由網域和對等互連的詳細資訊，請參閱 [ExpressRoute 線路及路由網域](expressroute-circuit-peerings.md)。


## <a name="before"></a>開始之前

* 若要連接到 Microsoft 對等互連，您必須設定與管理 NAT。 您的連線服務提供者可能會將 NAT 當成受控服務加以設定及管理。 如果您打算存取 Microsoft 對等互連上的 Azure PaaS 和 Azure SaaS 服務，請務必正確調整 NAT IP 集區的大小。 如需 NAT for ExpressRoute 的詳細資訊，請參閱 [Microsoft 對等互連的 NAT 需求](expressroute-nat.md#nat-requirements-for-microsoft-peering)。

* 如果您使用公用對等互連，且已經有用於存取 [Azure 儲存體](../storage/common/storage-network-security.md)或 [Azure SQL Database](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md) 的公用 IP 位址 IP 網路規則，必須確定 Microsoft 對等互連搭配設定的 NAT IP 集區有包含在 Azure 儲存體帳戶或 Azure SQL 帳戶的公用 IP 位址清單中。

* 為了在移至 Microsoft 對等互連時不用停機，請依本文中的出現順序操作步驟。

## <a name="create"></a>1.建立 Microsoft 對等互連

如果尚未建立 Microsoft 對等互連，請使用下列任一文章建立 Microsoft 對等互連。 若您的連線提供者提供受控第 3 層服務，您可以要求連線提供者為您的線路啟用 Microsoft 對等互連。

  * [建立使用 Azure 入口網站的 Microsoft 對等互連](expressroute-howto-routing-portal-resource-manager.md#msft)
  * [建立使用 Azure Powershell 的 Microsoft 對等互連](expressroute-howto-routing-arm.md#msft)
  * [建立使用 Azure CLI 的 Microsoft 對等互連](howto-routing-cli.md#msft)

## <a name="validate"></a>2.驗證 Microsoft 對等互連是否啟用

確認已啟用 Microsoft 對等互連，且已公告公用首碼為已設定狀態。

  * [Azure 入口網站](expressroute-howto-routing-portal-resource-manager.md#getmsft)
  * [Azure PowerShell](expressroute-howto-routing-arm.md#getmsft)
  * [Azure CLI](howto-routing-cli.md#getmsft)

## <a name="routefilter"></a>3.設定路由篩選並附加到線路

根據預設，新的 Microsoft 對等互連不會公告任何首碼，直到路由篩選附加至線路為止。 當您建立路由篩選規則時，您可以指定想要供 Azure PaaS 服務使用的 Azure 區域服務社群清單，如下列的螢幕擷取畫面所示：

![合併公用對等](.\media\how-to-move-peering\public.png)

依照下列任一文章來設定路由篩選：

  * [針對使用 Azure 入口網站的 Microsoft 對等互連設定路由篩選](how-to-routefilter-portal.md)
  * [針對使用 Azure PowerShell 的 Microsoft 對等互連設定路由篩選](how-to-routefilter-powershell.md)
  * [針對使用 Azure CLI 的 Microsoft 對等互連設定路由篩選](how-to-routefilter-cli.md)

## <a name="delete"></a>4.刪除公用對等互連

確認已設定 Microsoft 對等互連，而且已在 Microsoft 對等互連上正確公告您想要使用的首碼之後，您就可以刪除公用對等互連。 若要刪除公用對等互連，使用下列任一文章：

  * [刪除使用 Azure 入口網站的 Azure 公用對等互連](expressroute-howto-routing-portal-resource-manager.md#deletepublic)
  * [刪除使用 Azure PowerShell 的 Azure 公用對等互連](expressroute-howto-routing-arm.md#deletepublic)
  * [刪除使用 CLI 的 Azure 公用對等互連](howto-routing-cli.md#deletepublic)
  
## <a name="view"></a>5.檢視對等互連
  
在 Azure 入口網站中，您可以看到所有 ExpressRoute 線路和對等互連的清單。 如需詳細資訊，請參閱[檢視 Microsoft 對等互連的詳細資料](expressroute-howto-routing-portal-resource-manager.md#getmsft)。

## <a name="next-steps"></a>後續步驟

如需有關 ExpressRoute 的詳細資訊，請參閱 [ExpressRoute 常見問題集](expressroute-faqs.md)。
