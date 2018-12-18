---
title: 使用 Azure 地圖服務新增圖釘 | Microsoft Docs
description: 如何在 JavaScript 地圖上新增圖釘
author: jingjing-z
ms.author: jinzh
ms.date: 05/07/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 35b22e28a6a339af6f6f6e8db0655f2ae6de0118
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46122164"
---
# <a name="add-pins-to-the-map"></a>在地圖上新增圖釘

此文章說明如何在地圖上新增圖釘。  

## <a name="understand-the-code"></a>了解程式碼

<iframe height='500' scrolling='no' title='在地圖上新增圖釘' src='//codepen.io/azuremaps/embed/ZrVpEa/?height=504&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/ZrVpEa/'>在地圖上新增圖釘</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構對應物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

程式碼的第二個區塊會建立圖釘並將其新增至地圖上。 圖釘是 [Point](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.point?view=azure-iot-typescript-latest) 的一個[功能](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature?view=azure-iot-typescript-latest)，會以 [PinProperties](https://docs.microsoft.com/javascript/api/azure-maps-control/models.pinproperties?view=azure-iot-typescript-latest) 作為其功能屬性。 使用 `new atlas.data.Feature(new atlas.data.Point())` 建立圖釘，並定義其屬性。 圖釘圖層是圖釘的陣列。 使用地圖類別的 [addPins](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addpins) 函式，在地圖中新增圖釘圖層並定義圖釘圖層的屬性。 請在 [PinLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.pinlayeroptions?view=azure-iot-typescript-latest) 查看圖釘圖層的屬性。

## <a name="next-steps"></a>後續步驟

深入了解此文章使用的類別和方法：

> [!div class="nextstepaction"]
> [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)

請參閱下列文章，以取得更多可新增至地圖的程式碼範例：

> [!div class="nextstepaction"]
> [新增快顯](./map-add-popup.md)

> [!div class="nextstepaction"]
> [新增圖形](./map-add-shape.md)