---
title: 在 Azure 地圖服務中新增地圖控制項 | Microsoft Docs
description: 如何將縮放控制項、傾斜角度控制項、旋轉控制項和樣式選擇器新增至 Azure 地圖服務中的地圖。
author: walsehgal
ms.author: v-musehg
ms.date: 08/29/2018
ms.topic: how-to-guides
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 435c6545b69b4457c3e1035fb8125399d4c9c23a
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666126"
---
# <a name="add-map-controls-to-azure-maps"></a>將地圖控制項新增至 Azure 地圖服務

本文將說明如何在地圖上新增地圖控制項。 您也將了解如何建立具有所有控制項和[樣式選擇器](https://docs.microsoft.com/azure/azure-maps/choose-map-style#adding-the-style-picker)的地圖。

## <a name="add-zoom-control"></a>新增縮放控制項

<iframe height='500' scrolling='no' title='新增縮放控制項' src='//codepen.io/azuremaps/embed/WKOQyN/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>請在 <a href='https://codepen.io'>CodePen</a> 上，參閱 Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) 建立的 Pen：<a href='https://codepen.io/azuremaps/pen/WKOQyN/'>新增縮放控制項</a>。
</iframe>

上述程式碼中的第一個程式碼區塊會建立地圖物件。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用 atlas [ZoomControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.zoomcontrol?view=azure-iot-typescript-latest) 建立縮放控制項物件。

縮放控制項會新增可縮放地圖的功能。 第三個區塊會使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將縮放控制項新增至地圖。

## <a name="add-pitch-control"></a>新增傾斜角度控制項

<iframe height='500' scrolling='no' title='新增傾斜角度控制項' src='//codepen.io/azuremaps/embed/xJrwaP/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>請在 <a href='https://codepen.io'>CodePen</a> 上，參閱 Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) 建立的 Pen：<a href='https://codepen.io/azuremaps/pen/xJrwaP/'>新增傾斜角度控制項</a>。
</iframe>

上述第一個程式碼區塊會建立樣式設定為「灰階」的地圖物件。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用 atlas [PitchControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.pitchcontrol?view=azure-iot-typescript-latest) 建立傾斜角度控制項物件。

傾斜角度控制項會新增可變更地圖傾斜角度的功能。 第三個區塊會使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將傾斜角度控制項新增至地圖。

## <a name="add-compass-control"></a>新增指南針控制項

<iframe height='500' scrolling='no' title='新增旋轉控制項' src='//codepen.io/azuremaps/embed/GBEoRb/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>請在 <a href='https://codepen.io'>CodePen</a> 上，參閱 Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) 建立的 Pen：<a href='https://codepen.io/azuremaps/pen/GBEoRb/'>新增旋轉控制項</a>。
</iframe>

上述程式碼中的第一個程式碼區塊會建立地圖物件。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用 atlas [Compass Control](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.compasscontrol?view=azure-iot-typescript-latest#compasscontrol) 建立指南針控制項物件。 並且也會使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將指南針控制項新增至地圖。

## <a name="a-map-with-all-controls"></a>具有所有控制項的地圖

<iframe height='500' scrolling='no' title='具有所有控制項的地圖' src='//codepen.io/azuremaps/embed/qyjbOM/?height=265&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>請在 <a href='https://codepen.io'>CodePen</a> 上，參閱 Azure 地圖服務 (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) 建立的 Pen：<a href='https://codepen.io/azuremaps/pen/qyjbOM/'>具有所有控制項的地圖</a>。
</iframe>

上述程式碼中的第一個程式碼區塊會建立地圖物件。 如需如何建立地圖的相關指示，請參閱[建立地圖](./map-create.md)。

第二個程式碼區塊會使用 atlas [CompassControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.compasscontrol?view=azure-iot-typescript-latest#compasscontrol) 建立指南針控制項物件，並使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將其新增至地圖。

第三個程式碼區塊會使用 atlas [ZoomControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.zoomcontrol?view=azure-iot-typescript-latest) 建立縮放控制項物件，並使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將其新增至地圖。

第四個程式碼區塊會使用 atlas [PitchControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.pitchcontrol?view=azure-iot-typescript-latest) 建立傾斜角度控制項物件，並使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將其新增至地圖。

最後一個程式碼區塊會使用 atlas [StyleControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.stylecontrol?view=azure-iot-typescript-latest#stylecontrol)，新增地圖的樣式選擇器物件，並使用地圖的 [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol) 方法將其新增至地圖。

## <a name="next-steps"></a>後續步驟

深入了解本文使用的類別和方法： 
* [地圖](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)
    * [addControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addcontrol)

* [Atlas](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas?view=azure-iot-typescript-latest)
    * [CompassControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.compasscontrol?view=azure-iot-typescript-latest#compasscontrol)
    * [ZoomControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.zoomcontrol?view=azure-iot-typescript-latest)
    * [PitchControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.pitchcontrol?view=azure-iot-typescript-latest)
    * [StyleControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.stylecontrol?view=azure-iot-typescript-latest#stylecontrol)
    
如需更多可新增至地圖的程式碼範例，請參閱下列文章： 
* [新增圖釘](./map-add-pin.md)
* [新增快顯](./map-add-popup.md)