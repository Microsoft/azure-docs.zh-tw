---
title: 在 Azure 地圖服務中擴充 GeoJSON 幾何 | Microsoft Docs
description: 了解如何在 Azure 地圖服務中擴充 GeoJSON 幾何
author: sataneja
ms.author: sataneja
ms.date: 05/17/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.openlocfilehash: 319f9cba23d088553f361b6a0d648bbde94e0743
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38968556"
---
# <a name="extending-geojson-geometries"></a>擴充 GeoJSON 幾何

Azure 提供許多強大的 API，能夠在地理特徵之內/依循地理特徵進行搜尋。
這些 API 採用 [GeoJSON 規格][1]進行標準化，統一呈現地理特徵 (例如：州界、路線)。  

[GeoJSON 規格][1] 僅支援下列幾何：

* GeometryCollection
* LineString
* MultiLineString
* MultiPoint
* MultiPolygon
* Point
* Polygon

某些 Azure 地圖服務 API (例如：[幾何內搜尋](https://docs.microsoft.com/rest/api/maps/search/postsearchinsidegeometry)) 接受不屬於 [GeoJSON 規格][1]的幾何，例如「Circle」。

本文詳細說明 Azure 地圖服務如何延伸 [GeoJSON 規格][1]來代表特定的幾何。

### <a name="circle"></a>Circle

[GeoJSON 規格][1]不支援 `Circle` 幾何。我們使用 `GeoJSON Feature` 物件代表圓形。

使用 `GeoJSON Feature` 物件代表的 `Circle` 幾何 __必須__ 包含下列項目：

1. 中心
   >圓形的中心是以 `GeoJSON Point` 型別表示。

2. 半徑
   >圓形的 `radius` 是以 `GeoJSON Feature` 的屬性表示。 半徑值的單位是 _公尺_ ，而且必須屬於型別 `double`。

3. SubType
   >圓形幾何也必須包含 `subType` 屬性。 這個屬性必須屬於 `GeoJSON Feature` 的屬性，它的值應該是 _Circle_


#### <a name="example"></a>範例

以下是中心在 (緯度：47.639754，經度：-122.126986) 且半徑等於 100 公尺的圓形，使用 `GeoJSON Feature` 物件：

```json            
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "subType": "Circle",
        "radius": 100
    }
}          
```

[1]: https://tools.ietf.org/html/rfc7946
