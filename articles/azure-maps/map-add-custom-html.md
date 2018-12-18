---
title: 在 Azure 地圖服務中新增自訂 HTML | Microsoft Docs
description: 如何在 JavaScript 地圖上新增自訂 HTML
author: jingjing-z
ms.author: jinzh
ms.date: 05/07/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 5060839be244f55700b40735e598964a0b7b6709
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382040"
---
# <a name="add-custom-html-to-the-map"></a>在地圖上新增自訂 HTML

本文說明如何在地圖中新增自訂 HTML，例如影像檔。 

## <a name="understand-the-code"></a>了解程式碼

<iframe height='466' scrolling='no' title='在地圖上新增自訂 HTML - png' src='//codepen.io/azuremaps/embed/MVoeVw/?height=466&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/MVoeVw/'>在地圖上新增自訂 HTML - png</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構對應物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

程式碼的第二個區塊會透過影像建立 HTML 元素。

程式碼的最後一個區塊會使用地圖類別的 [addHtml](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addhtml) 函式，在地圖的指定位置新增影像。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [addHtml](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addhtml)
    
如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [顯示搜尋結果](./map-search-location.md)
* [從座標取得資訊](./map-get-information-from-coordinate.md)

