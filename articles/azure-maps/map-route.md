---
title: 使用 Azure 地圖服務顯示指示 | Microsoft Docs
description: 如何在 JavaScript 地圖上顯示兩個位置之間的指示
author: jingjing-z
ms.author: jinzh
ms.date: 08/31/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: codepen
ms.openlocfilehash: 80abd6db9888524aa6a66d9861d8dc2d05188e19
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43781491"
---
# <a name="show-directions-from-a-to-b"></a>顯示從甲地到乙地的指示 

本文說明如何提出路線要求，並在地圖上顯示路線。 

## <a name="understand-the-code"></a>了解程式碼

<iframe height='500' scrolling='no' title='在地圖上顯示從甲地到乙地的指示 (服務模組)' src='//codepen.io/azuremaps/embed/RBZbep/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆<a href='https://codepen.io/azuremaps/pen/RBZbep/'>在地圖上顯示從甲地到乙地的指示 (服務模組)</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構對應物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

第二個程式碼區塊中的行會具現化服務用戶端。

第三個程式碼區塊會初始化地圖上的[串字圖層](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addlinestrings)。

程式碼的第四個區塊會在地圖上建立並新增圖釘，以代表路線的起點和終點。 如需相關指示，您可以查看[在地圖上新增圖釘](map-add-pin.md)。

程式碼的下一個區塊會使用地圖類別的 [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamerabounds) 函式，根據路線的起點和終點設定地圖的週框方塊。

第六個程式碼區塊會建構路線查詢。

程式碼的最後一個區塊會透過 [getRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/services.route?view=azure-iot-typescript-latest#getroutedirections) 方法查詢 Azure 地圖服務路線規劃服務，以取得起點與終點之間的路線。 然後，回應會使用 [getGeoJsonRoutes](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.geojson.geojsonroutedirectionsresponse?view=azure-iot-typescript-latest#getgeojsonroutes) 方法剖析為 GeoJSON 格式。 它會將這些線條全都新增到地圖上以呈現路線。 如需詳細資訊，您可以查看[在地圖上新增線條](./map-add-shape.md#addALine)。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 

* [getRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/services.route?view=azure-iot-typescript-latest#getroutedirections)
* [getGeoJsonRoutes](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.geojson.geojsonroutedirectionsresponse?view=azure-iot-typescript-latest#getgeojsonroutes)
* [串線圖層](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addlinestrings)
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamerabounds)
    * [addLinestrings](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addlinestrings)
    * [addPins](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addpins)

如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [在地圖上顯示路況](./map-show-traffic.md)
* [與地圖互動 – 滑鼠事件](./map-events.md)
