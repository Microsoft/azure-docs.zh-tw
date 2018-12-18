---
title: 使用 Azure 地圖服務新增圖形 | Microsoft Docs
description: 如何在 JavaScript 地圖上新增圖形
author: jingjing-z
ms.author: jinzh
ms.date: 05/07/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 6b78c7f5d7d1b5ad4db9401bf2138502c9bbd397
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46121889"
---
# <a name="add-a-shape-to-a-map"></a>在地圖中新增圖形

本文說明如何在地圖中新增線條、圓形和多邊形。 

<a id="addALine"></a>

## <a name="add-a-line"></a>新增線條

<iframe height='500' scrolling='no' title='在地圖中新增線條' src='//codepen.io/azuremaps/embed/qomaKv/?height=534&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/qomaKv/'>在地圖中新增線條</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構地圖物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

在程式碼的第二個區塊中，會建立一條線。 線條是 LineString 的一項[功能](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature?view=azure-iot-typescript-latest)，會以 LineStringProperties 作為其功能屬性。 使用 `new atlas.data.Feature(new atlas.data.LineString())` 建立線條，並定義其屬性。

線條圖層是線條的陣列。 最後一個程式碼區塊會使用地圖類別的 [addLineStrings](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addlinestrings) 函式，在地圖中新增線條圖層並定義線條圖層的屬性。 請在 [LinestringLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.linestringlayeroptions?view=azure-iot-typescript-latest) 查看線條圖層的屬性。

<a id="addACircle"></a>

## <a name="add-a-circle"></a>新增圓形

<iframe height='500' scrolling='no' title='在地圖中新增圓形' src='//codepen.io/azuremaps/embed/PRmzJX/?height=538&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/PRmzJX/'>在地圖中新增圓形</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構地圖物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

在程式碼的第二個區塊中，會建立一個圓形。 圓形是 [Point](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.point?view=azure-iot-typescript-latest) 的一項[功能](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature?view=azure-iot-typescript-latest)，會以 [CircleProperties](https://docs.microsoft.com/javascript/api/azure-maps-control/models.circleproperties?view=azure-iot-typescript-latest) 作為其功能屬性。 使用 `new atlas.data.Feature(new atlas.data.Point())` 建立圓形，並定義其屬性。

圓形圖層是圓形的陣列。 最後一個程式碼區塊會使用地圖類別的 [addCircle](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcircles) 函式，在地圖中新增圓形圖層並定義圓形圖層的屬性。 請在 [CircleLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.circlelayeroptions?view=azure-iot-typescript-latest) 查看圓形圖層的屬性。

<a id="addAPolygon"></a>

## <a name="add-a-polygon"></a>新增多邊形

<iframe height='500' scrolling='no' title='在地圖中新增多邊形 ' src='//codepen.io/azuremaps/embed/yKbOvZ/?height=543&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/yKbOvZ/'>在地圖中新增多邊形</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構地圖物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

在程式碼的第二個區塊中，會建立一個多邊形。 多邊形是 [Polygon](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.polygon?view=azure-iot-typescript-latest) 的一項[功能](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature?view=azure-iot-typescript-latest)，會以 [PolygonProperties](https://docs.microsoft.com/javascript/api/azure-maps-control/models.polygonproperties?view=azure-iot-typescript-latest) 作為其功能屬性。 使用 `new atlas.data.Feature(new atlas.data.Polygon())` 建立多邊形，並定義其屬性。 在多邊形建構函式中提供多邊形路徑的已排序座標。

多邊形圖層是多邊形的陣列。 最後一個程式碼區塊會使用地圖類別的 [addPolygons](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addpolygons) 函式，在地圖中新增多邊形圖層並定義其屬性。 請在 [PolygonLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.polygonlayeroptions?view=azure-iot-typescript-latest) 查看多邊形圖層的屬性。

## <a name="next-steps"></a>後續步驟

如需更多可新增至地圖的程式碼範例，請參閱下列文章：

> [!div class="nextstepaction"]
> [新增自訂 HTML](./map-add-custom-html.md)

> [!div class="nextstepaction"]
> [顯示搜尋結果](./map-search-location.md)
