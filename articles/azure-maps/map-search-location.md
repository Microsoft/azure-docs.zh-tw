---
title: 使用 Azure 地圖服務顯示搜尋結果 | Microsoft Docs
description: 如何使用 Azure 地圖服務執行搜尋要求，然後在 Javascript 地圖上顯示結果
author: jingjing-z
ms.author: jinzh
ms.date: 08/31/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 7d4eb5f9be4a6bcefe4b544d3f97a9b9391c0d81
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43665772"
---
# <a name="show-search-results-on-the-map"></a>在地圖上顯示搜尋結果

本文說明如何搜尋景點，並在地圖上顯示搜尋結果。 

## <a name="understand-the-code"></a>了解程式碼

<iframe height='500' scrolling='no' title='在地圖上顯示搜尋結果 (服務模組)' src='//codepen.io/azuremaps/embed/zLdYEB/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/zLdYEB/'> (英文) 在地圖上顯示搜尋結果 (服務模組)</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a> (英文)。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構地圖物件，並且具現化用戶端服務。 如需相關指示，您可以查看[建立對應](./map-create.md)。

程式碼的第二個區塊會使用模糊搜尋 [Azure 地圖服務模糊搜尋 API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) (英文) 來搜尋景點。 模糊搜尋 API 可以處理任何模糊輸入組合。 然後，模糊搜尋服務的回應會透過 [getGeoJsonSearchResponse](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.geojson.geojsonsearchresponse?view=azure-iot-typescript-latest#geojsonsearchresponse) 方法剖析成 GeoJSON 格式。 這些釘選接著會新增至地圖，以在地圖上顯示景點。

程式碼的最後一個區塊會使用地圖的 [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/models.cameraboundsoptions?view=azure-iot-typescript-latest) (英文) 屬性，為地圖新增觀景窗界限。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 

* [Azure 地圖服務模糊搜尋 API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy)
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [addPins](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addpins)
    
如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [從座標取得資訊](./map-get-information-from-coordinate.md)
* [顯示從甲地到乙地的指示](./map-route.md)
