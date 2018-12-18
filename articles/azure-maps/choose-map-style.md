---
title: Azure 地圖服務中的地圖樣式功能 | Microsoft Docs
description: 了解與「Azure 地圖服務樣式」相關的功能。
author: walsehgal
ms.author: v-musehg
ms.date: 08/31/2018
ms.topic: how-to-guides
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: b95e3c325160d1c0b775320e25fdd40514b02b9f
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666866"
---
# <a name="choose-a-map-style-in-azure-maps"></a>選擇 Azure 地圖服務中的地圖樣式
「Azure 地圖服務」有四種不同的地圖樣式可選擇。 如需地圖樣式的詳細資訊，請參閱 [Azure 地圖服務中支援的地圖樣式](./supported-map-styles.md)。 本文說明如何使用與樣式相關的功能在地圖負載上設定樣式、設定新樣式，以及使用樣式選擇器控制項。

## <a name="setting-style-on-map-load"></a>設定地圖負載上的樣式

<iframe height='500' scrolling='no' title='設定地圖負載上的樣式' src='//codepen.io/azuremaps/embed/WKOQRq/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆<a href='https://codepen.io/azuremaps/pen/WKOQRq/'>在地圖負載上設定樣式</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

上述程式碼會建立樣式設定為「灰階」的地圖物件。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

## <a name="updating-the-style"></a>更新樣式

<iframe height='500' scrolling='no' title='更新樣式' src='//codepen.io/azuremaps/embed/yqXYzY/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆<a href='https://codepen.io/azuremaps/pen/yqXYzY/'>更新樣式</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

上述程式碼中的第一個程式碼區塊會建立地圖物件，但不會預先設定樣式。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用地圖的 [setStyle](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setstyle) 方法將地圖樣式設為附屬。

## <a name="adding-the-style-picker"></a>新增樣式選擇器

<iframe height='500' scrolling='no' title='新增樣式選擇器' src='//codepen.io/azuremaps/embed/OwgyvG/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆<a href='https://codepen.io/azuremaps/pen/OwgyvG/'>新增樣式選擇器</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

上述程式碼中的第一個程式碼區塊會建立地圖物件，但不會預先設定樣式。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用地圖集 [StyleControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.stylecontrol?view=azure-iot-typescript-latest#stylecontrol) 建構函式，來建構樣式選取器。

樣式選取器可啟用地圖的樣式選擇功能。 第三個程式碼區塊使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法，將樣式選取器新增至地圖。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [setStyle](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setstyle) (英文)
    * [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) (英文)

* [Atlas](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas?view=azure-iot-typescript-latest) (英文)
    * [StyleControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.stylecontrol?view=azure-iot-typescript-latest#stylecontrol) (英文)
    
如需更多可新增至地圖的程式碼範例，請參閱下列文章：
* [新增地圖控制項](./map-add-controls.md)
* [新增圖釘](./map-add-pin.md)
