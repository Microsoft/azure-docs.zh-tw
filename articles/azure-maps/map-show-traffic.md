---
title: 使用 Azure 地圖服務顯示路況 | Microsoft Docs
description: 如何在 Javascript 地圖上顯示路況資料
author: jingjing-z
ms.author: jinzh
ms.date: 05/07/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 2499822db587dbf47dccedf6039d0fb5823c58b5
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746073"
---
# <a name="show-traffic-on-the-map"></a>在地圖上顯示路況

本文會示範如何在地圖上顯示路況和事件資訊。 

## <a name="understand-the-code"></a>了解程式碼

<iframe height='456' scrolling='no' title='在地圖上顯示路況' src='//codepen.io/azuremaps/embed/WMLRPw/?height=456&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>在 <a href='https://codepen.io'>CodePen</a> 上查看 Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) 製作的<a href='https://codepen.io/azuremaps/pen/WMLRPw/'>在地圖上顯示路況</a>Pen。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構對應物件。 如需相關指示，您可以查看[建立對應](map-create.md)。

程式碼的第二個區塊會使用地圖類別的 [setTraffic](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#settraffic) 函式，來轉譯地圖上的交通流量和事件。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [setTraffic](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#settraffic)

如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [與地圖互動 – 滑鼠事件](./map-events.md)
* [建置可存取的地圖](./map-accessibility.md)

請查看我們的[程式碼範例頁面](http://aka.ms/AzureMapsSamples)以取得更多地圖案例。
