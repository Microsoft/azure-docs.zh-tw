---
title: Azure 流量管理員中的流量檢視 | Microsoft Docs
description: 流量管理員流量檢視簡介
services: traffic-manager
documentationcenter: traffic-manager
author: KumudD
manager: jeconnoc
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
ms.openlocfilehash: 7ce51017fdee92e5589c06b398c9650930d5436d
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30179832"
---
# <a name="traffic-manager-traffic-view"></a>流量管理員流量檢視

流量管理員為您提供 DNS 層級路由，讓您的使用者根據您在建立設定檔時指定的路由方法，被導向至狀況良好的端點。 流量檢視提供流量管理員一個使用者群體 (在 DNS 解析程式資料粒度層級) 及其流量模式的檢視。 當您啟用流量檢視時，系統會處理這項資訊以提供可採取動作的深入資訊。 

藉由使用流量檢視，您可以：
- 了解您的使用者群體所在的位置 (取決於本機 DNS 解析程式層級資料粒度)。
- 檢視源自於這些區域的流量 (當作 Azure Traffic Manager 所處理的 DNS 觀察)。
- 洞悉這些使用者體驗的代表延遲是什麼。
- 深入探討從使用者群體之每個個體到您具有端點之 Azure 區域的特定流量模式。 

例如，您可以使用流量檢視來了解哪些區域有大量流量，但是受更高的延遲所苦。 接下來您可以使用這項資訊來規劃到新的 Azure 區域的使用量擴充，讓這些使用者可以擁有較低的延遲體驗。

## <a name="how-traffic-view-works"></a>流量檢視的運作方式

流量檢視的運作方式是讓流量管理員查看在過去七天內針對已啟用此功能之設定檔所收到的連入查詢。 流量檢視會根據連入查詢資訊擷取 DNS 解析程式的來源 IP，然後用來作為使用者位置的代表。 然後會在 DNS 解析程式層級資料粒度群組在一起，藉由使用由流量管理員維護之 IP 位址的地理資訊，來建立使用者群體區域。 流量管理員接著會查看查詢路由前往的 Azure 區域，並且為這些區域的使用者建構流量流程對應。  
在下一個步驟中，流量管理員會將 Azure 區域對應的使用者群組區域與它為不同使用者網路維護的網路智慧延遲資料表相互關聯，以了解當連接到 Azure 區域時，這些區域的使用者所體驗的平均延遲。 所有這些計算接著會在呈現給您之前，在每個本機 DNS 解析程式 IP 層級上合併。 您可以用各種方式使用該資訊。

>[!NOTE]
>流量檢視中所述的延遲是使用者與已連線 Azure 區域間的代表性延遲，不是 DNS 查詢延遲。 流量檢視對本機 DNS 解析程式與查詢路由前往的 Azure 區域之間的延遲進行最佳投入量估算，如果資料不足，則傳回的延遲將為 Null。 

## <a name="visual-overview"></a>視覺效果概觀

當您瀏覽至流量管理員頁面中的 [流量檢視] 區段時，您會看到覆蓋流量檢視深入分析的地理區域地圖。 地圖會提供流量管理員設定檔的使用者群體和端點相關資訊。

### <a name="user-base-information"></a>使用者群體資訊

可使用位置資訊的本機 DNS 解析程式會顯示在地圖中。 DNS 解析程式的顏色表示使用者所經歷的延遲，使用者使用該 DNS 解析程式進行流量管理員查詢。

如果您將滑鼠停留在地圖中的 DNS 解析程式位置，則會顯示：
- DNS 解析程式的 IP 位址
- 流量管理員從此處看到的 DNS 查詢流量數量
- 路由 DNS 解析程式流量的端點，如同端點及 DNS 解析程式之間的路線 
- 從該位置至端點的平均延遲，以連接這兩者的線條顏色表示

### <a name="endpoint-information"></a>端點資訊

端點所在的 Azure 區域會在地圖中顯示為藍點。 如果您的端點是外部的，而且沒有與其對應的 Azure 區域，則該端點會顯示在地圖的頂端。 按一下任何端點，可查看流量被導向該端點的來源位置 (根據使用的 DNS 解析程式)。 連線會顯示為端點與 DNS 解析程式位置之間的路線，並且會根據該配對之間的代表性延遲標示顏色。 此外，您可以查看端點名稱、執行端點的 Azure 區域和由此流量管理員設定檔導向至此的要求總數。


## <a name="tabular-listing-and-raw-data-download"></a>表格檢視和原始資料下載

您可以在 Azure 入口網站中以表格形式檢視流量檢視資料。 每個 DNS 解析程式 IP / 端點配對都有一個項目會顯示 DNS 解析程式的 IP 位置、端點所在位置 (如果有的話) 的 Azure 區域名稱與地理位置、與 DNS 解析程式到該端點相關聯的要求數量，以及與使用 DNS 的使用者相關聯的代表性延遲 (如果有的話)。 您也可以將流量檢視資料下載為 CSV 檔案，這可作為所選分析流程的一部分使用。

## <a name="billing"></a>計費

當您使用流量檢視時，您會根據用來建立呈現之深入資訊的資料點數目收到帳單。 目前，唯一使用的資料點類型是流量管理員設定檔收到的查詢。 如需價格的詳細資訊，請瀏覽[流量管理員定價分頁](https://azure.microsoft.com/pricing/details/traffic-manager/)。


## <a name="next-steps"></a>後續步驟

- 了解 [流量管理員的運作方式](traffic-manager-overview.md)
- 深入了解流量管理員支援的 [流量路由方法](traffic-manager-routing-methods.md)
- 了解如何 [建立流量管理員設定檔](traffic-manager-create-profile.md)

