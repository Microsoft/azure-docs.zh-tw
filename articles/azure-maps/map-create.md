---
title: 使用 Azure 地圖服務建立地圖 | Microsoft Docs
description: 如何建立 Javascript 地圖
author: jingjing-z
ms.author: jinzh
ms.date: 05/07/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: c5d48e2e7316f33a565fc61a769a29c00834eed5
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43343364"
---
# <a name="create-a-map"></a>建立地圖

本文說明如何建立地圖。  

## <a name="understand-the-code"></a>了解程式碼

地圖的建構方式有兩種。 您可以藉由指定中心點和縮放層級來設定地圖的觀景窗，也可以藉由指定西南週框點和東北週框點來設定地圖的觀景窗界限。

<a id="setCameraOptions"></a>

### <a name="setting-the-camera"></a>設定觀景窗

<iframe height='310' scrolling='no' title='透過 CameraOptions 建立地圖' src='//codepen.io/azuremaps/embed/qxKBMN/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/qxKBMN/'>透過 CameraOptions 建立地圖</a>，發佈者：Azure LBS (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

上述程式碼會透過 `new atlas.Map()` 建立[地圖物件](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)。 置中與縮放層級等地圖屬性是 [CameraOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.cameraoptions?view=azure-iot-typescript-latest) 的一部分。 CameraOptions 可定義在地圖建構函式中或透過地圖類別的 [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamera) 函式來定義。

<a id="setCameraBoundsOptions"></a>

### <a name="setting-the-camera-bounds"></a>設定觀景窗界限

<iframe height='310' scrolling='no' title='透過 CameraBoundsOptions 建立地圖' src='//codepen.io/azuremaps/embed/ZrRbPg/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>查看畫筆 <a href='https://codepen.io/azuremaps/pen/ZrRbPg/'>透過 CameraBoundsOptions 建立地圖</a>，發佈者：Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)，發佈位置：<a href='https://codepen.io'>CodePen</a>。
</iframe>

上述程式碼會透過 `new atlas.Map()` 建構[地圖物件](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)。 週框方塊等地圖屬性是 [CameraBoundsOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.cameraboundsoptions?view=azure-iot-typescript-latest) 的一部分。 CameraBoundsOptions 可透過地圖類別的 [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamerabounds) 函式來定義。

## <a name="try-out-the-code"></a>試用程式碼 

看一下上述範例程式碼。 您可以在左側的 [JS] 索引標籤上編輯 JavaScript 程式碼，並在右邊的 [結果] 索引標籤上看到地圖視圖變更。 您也可以按一下 [在 CodePen 上編輯] 按鈕，並在 CodePen 中編輯程式碼。 

<a id="relatedReference"></a>

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamera)
    * [setCameraBounds](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamerabounds)
    
如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [選擇地圖樣式](choose-map-style.md)
* [新增地圖控制項](map-add-controls.md)
    

