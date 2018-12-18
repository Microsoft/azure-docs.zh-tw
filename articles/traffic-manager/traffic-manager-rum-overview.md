---
title: Azure 流量管理員中的實際使用者度量 | Microsoft Docs
description: 流量管理員中實際使用者度量的簡介
services: traffic-manager
documentationcenter: traffic-manager
author: KumudD
manager: timlt
editor: ''
tags: ''
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: 4e8d808d65c9898d230455d128e3ffc50db303d6
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30178108"
---
# <a name="traffic-manager-real-user-measurements-overview"></a>流量管理員實際使用者度量概觀

當您設定流量管理員設定檔使用效能路由方式時，服務會尋找 DNS 查詢要求的來源位置，進行路由決策以將這些要求者導向至給予他們最低延遲的 Azure 區域。 這個作業會透過利用流量管理員為不同使用者網路維護的網路延遲智慧來完成。

實際使用者度量可讓您從使用者使用的用戶端應用程式測量 Azure 區域的網路延遲度量，並且讓流量管理員在進行路由決策時一併考量該資訊。 藉由選擇使用實際使用者度量，您可以增加路由要求的精確度，這些要求是來自您的使用者所在的網路。 

## <a name="how-real-user-measurements-work"></a>實際使用者度量的運作方式

實際使用者度量的運作方式是從使用所在的使用者網路中看到時，給予 Azure 區域的用戶端應用程式測量延遲。 舉例來說，如果您有讓不同位置之使用者存取的網頁 (例如，北美區域)，您可以在使用效能路由方法將它們導向裝載您的伺服器的最佳 Azure 區域時，利用實際使用者度量的功能。

一開始會將 Azure 提供的 JavaScript (其中具有獨特索引鍵) 內嵌至您的網頁。 完成之後，每當使用者造訪該網頁，JavaScript 就會查詢流量管理員來取得它應該測量之 Azure 區域的相關資訊。 服務會傳回一組端點給指令碼，然後藉由下載裝載在這些 Azure 區域的單一像素映像，並且註明要求傳送的時間與收到第一個位元組的時間之間的延遲，來連續測量這些區域。 這些度量會接著回報給「流量管理員」服務。

經過一段時間，這個作業會在許多網路之間發生許多次，讓流量管理員取得更多使用者所在之網路的延遲特性相關精確資訊。 這項資訊開始納入流量管理員進行的路由決策中。 如此一來，就會根據傳送的實際使用者度量，增加這些決策的精確度。

當您使用實際使用者度量時，您會根據傳送給流量管理員的度量數目收到帳單。 如需價格的詳細資訊，請瀏覽[流量管理員定價分頁](https://azure.microsoft.com/pricing/details/traffic-manager/)。

## <a name="next-steps"></a>後續步驟
- 了解如何使用[具有網頁的實際使用者度量](traffic-manager-create-rum-web-pages.md)
- 了解 [流量管理員的運作方式](traffic-manager-overview.md)
- 深入了解 [Mobile Center](https://docs.microsoft.com/mobile-center/)
- 深入了解流量管理員支援的 [流量路由方法](traffic-manager-routing-methods.md)
- 了解如何 [建立流量管理員設定檔](traffic-manager-create-profile.md)

