---
title: 使用 Azure 地圖服務顯示座標的相關資訊 | Microsoft Docs
description: 如何在使用者選取座標時於地圖上顯示地址的相關資訊
author: jingjing-z
ms.author: jinzh
ms.date: 08/31/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 331e687c40f21b0bf6074239969848c632682773
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43667476"
---
# <a name="get-information-from-a-coordinate"></a>從座標取得資訊

本文說明如何進行反向地址搜尋，並在點按滑鼠時於快顯中顯示所點選位置的地址。

## <a name="understand-the-code"></a>了解程式碼

<iframe height='500' scrolling='no' title='從座標取得資訊 (服務模組)' src='//codepen.io/azuremaps/embed/ejEYMZ/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/ejEYMZ/'>從座標取得資訊 (服務模組)</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

在上述程式碼中，程式碼的第一個區塊會建構對應物件。 如需相關指示，您可以查看[建立對應](./map-create.md)。

第二個程式碼區塊中的行會具現化服務用戶端。

第三個程式碼區塊會將滑鼠游標的樣式更新為指標。

第四個程式碼區塊會建立一個快顯視窗。 如需相關指示，您可以查看[在地圖上新增快顯](./map-add-popup.md)。

最後一個程式碼區塊會新增滑鼠點選的事件接聽程式。 按下滑鼠時，會使用所按下點的座標建立搜尋查詢。 然後，它會使用地圖的 [getSearchAddressReverse](https://docs.microsoft.com/javascript/api/azure-maps-rest/services.search?view=azure-iot-typescript-latest#getsearchaddressreverse) 端點查詢座標的位址。

對於成功的回應，它會收集所點選位置的地址，並透過快顯類別的 [setPopupOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#setpopupoptions) 函式定義快顯內容和位置。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [反向地址搜尋](https://docs.microsoft.com/rest/api/maps/search/getsearchaddressreverse)
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [addEventListener](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addeventlistener)
* [Popup](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest)
    * [setPopupOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#setpopupoptions)
    * [open](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#open)
    * [close](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#close)

如需更多可新增至地圖的程式碼範例，請參閱下列文章：
* [顯示從甲地到乙地的指示](./map-route.md)
* [顯示流量](./map-show-traffic.md)
