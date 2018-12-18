---
title: Azure 串流分析地理空間函式簡介
description: 本文說明 Azure 串流分析作業所使用的地理空間函式。
services: stream-analytics
author: mamccrea
ms.author: mamccrea
manager: kfile
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 09/04/2018
ms.openlocfilehash: 02d1f551c7ec2856bbfce65c5397f454f6b9d5be
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43703271"
---
# <a name="introduction-to-stream-analytics-geospatial-functions"></a>串流分析地理空間函式簡介

Azure 串流分析中的地理空間函式可即時分析串流地理空間資料。 只要撰寫幾行程式碼，就能開發適用於複雜案例的生產等級解決方案。 

可受益於地理空間函式的案例範例包括：

* 汽車共乘
* 車隊管理
* 資產追蹤
* 異地隔離
* 跨基地台進行電話追蹤

串流分析查詢語言具有七個內建的地理空間函式：**CreateLineString**、**Createpoint**、**CreatePolygon**、**ST_DISTANCE**、**ST_OVERLAPS**、**ST_INTERSECTS** 和 **ST_WITHIN**。

## <a name="createlinestring"></a>CreateLineString

`CreateLineString` 函式會接受點，並傳回 GeoJSON LineString，以在地圖上繪製為線條。 您必須有至少兩個點才能建立 LineString。 LineString 點會依序連接。

下列查詢使用 `CreateLineString` 來建立使用三個點的 LineString。 第一個點會透過串流輸入資料來建立，另外兩個點則以手動方式建立。

```SQL 
SELECT  
     CreateLineString(CreatePoint(input.latitude, input.longitude), CreatePoint(10.0, 10.0), CreatePoint(10.5, 10.5))  
FROM input  
```  

### <a name="input-example"></a>輸入範例  
  
|緯度|經度|  
|--------------|---------------|  
|3.0|-10.2|  
|-87.33|20.2321|  
  
### <a name="output-example"></a>輸出範例  

 {"type" : "LineString", "coordinates" : [ [-10.2, 3.0], [10.0, 10.0], [10.5, 10.5] ]}

 {"type" : "LineString", "coordinates" : [ [20.2321, -87.33], [10.0, 10.0], [10.5, 10.5] ]}

若要深入了解，請瀏覽 [CreateLineString](https://msdn.microsoft.com/azure/stream-analytics/reference/createlinestring) 參考。

## <a name="createpoint"></a>CreatePoint

`CreatePoint` 函式會接受經緯度，並傳回 GeoJSON 點，以繪製在地圖上。 經緯度必須是**浮點數**資料類型。

下列查詢範例使用 `CreatePoint` 來建立點，這個點所使用的經緯度來自串流輸入資料。

```SQL 
SELECT  
     CreatePoint(input.latitude, input.longitude)  
FROM input 
```  

### <a name="input-example"></a>輸入範例  
  
|緯度|經度|  
|--------------|---------------|  
|3.0|-10.2|  
|-87.33|20.2321|  
  
### <a name="output-example"></a>輸出範例
  
 {"type" : "Point", "coordinates" : [-10.2, 3.0]}  
  
 {"type" : "Point", "coordinates" : [20.2321, -87.33]}  

若要深入了解，請瀏覽 [CreatePoint](https://msdn.microsoft.com/azure/stream-analytics/reference/createpoint) 參考。

## <a name="createpolygon"></a>CreatePolygon

`CreatePolygon` 函式會接受點，並傳回 GeoJSON 多邊形記錄。 點的順序必須依照右手定則方向，也就是逆時針方向。 想像一下以這些點所宣告的順序從一個點走到另一個點。 在整個過程中，此多邊形的中心會在您的左側。

下列查詢範例使用 `CreatePolygon` 從三個點建立多邊形。 前兩個點以手動方式建立，最後一個點則從輸入資料建立。

```SQL 
SELECT  
     CreatePolygon(CreatePoint(input.latitude, input.longitude), CreatePoint(10.0, 10.0), CreatePoint(10.5, 10.5), CreatePoint(input.latitude, input.longitude))  
FROM input  
```  

### <a name="input-example"></a>輸入範例  
  
|緯度|經度|  
|--------------|---------------|  
|3.0|-10.2|  
|-87.33|20.2321|  
  
### <a name="output-example"></a>輸出範例  

 {"type" : "Polygon", "coordinates" : [[ [-10.2, 3.0], [10.0, 10.0], [10.5, 10.5], [-10.2, 3.0] ]]}
 
 {"type" : "Polygon", "coordinates" : [[ [20.2321, -87.33], [10.0, 10.0], [10.5, 10.5], [20.2321, -87.33] ]]}

若要深入了解，請瀏覽 [CreatePolygon](https://msdn.microsoft.com/azure/stream-analytics/reference/createpolygon) 參考。


## <a name="stdistance"></a>ST_DISTANCE
`ST_DISTANCE` 函式會傳回兩點之間的距離 (以公尺為單位)。 

下列查詢使用 `ST_DISTANCE` 以在加油站與車輛的距離小於 10 公里時產生事件。

```SQL
SELECT Cars.Location, Station.Location 
FROM Cars c  
JOIN Station s ON ST_DISTANCE(c.Location, s.Location) < 10 * 1000
```

若要深入了解，請瀏覽 [ST_DISTANCE](https://msdn.microsoft.com/azure/stream-analytics/reference/st-distance) 參考。

## <a name="stoverlaps"></a>ST_OVERLAPS
`ST_OVERLAPS` 函式會比較兩個多邊形。 如果多邊形重疊，函數會傳回 1。 如果多邊形未重疊，則函數會傳回 0。 

下列查詢使用 `ST_OVERLAPS` 以在大樓位於可能淹水的區域內時產生事件。

```SQL
SELECT Building.Polygon, Building.Polygon 
FROM Building b 
JOIN Flooding f ON ST_OVERLAPS(b.Polygon, b.Polygon) 
```

下列查詢範例會在暴風雨朝車輛方向行進時產生事件。

```SQL
SELECT Cars.Location, Storm.Course
FROM Cars c, Storm s
JOIN Storm s ON ST_OVERLAPS(c.Location, s.Course)
```

若要深入了解，請瀏覽 [ST_OVERLAPS](https://msdn.microsoft.com/azure/stream-analytics/reference/st-overlaps) 參考。

## <a name="stintersects"></a>ST_INTERSECTS
`ST_INTERSECTS` 函式會比較兩個 LineString。 如果 LineString 相交，函式會傳回 1。 如果 LineString 未相交，則函式會傳回 0。

下列查詢範例使用 `ST_INTERSECTS` 來判斷柏油路是否與泥土路相交。

```SQL 
SELECT  
     ST_INTERSECTS(input.pavedRoad, input.dirtRoad)  
FROM input  
```  

### <a name="input-example"></a>輸入範例  
  
|datacenterArea|stormArea|  
|--------------------|---------------|  
|{“type”:”LineString”, “coordinates”: [ [-10.0, 0.0], [0.0, 0.0], [10.0, 0.0] ]}|{“type”:”LineString”, “coordinates”: [ [0.0, 10.0], [0.0, 0.0], [0.0, -10.0] ]}|  
|{“type”:”LineString”, “coordinates”: [ [-10.0, 0.0], [0.0, 0.0], [10.0, 0.0] ]}|{“type”:”LineString”, “coordinates”: [ [-10.0, 10.0], [0.0, 10.0], [10.0, 10.0] ]}|  
  
### <a name="output-example"></a>輸出範例  

 1  
  
 0  

若要深入了解，請瀏覽 [ST_INTERSECTS](https://msdn.microsoft.com/azure/stream-analytics/reference/st-intersects) 參考。

## <a name="stwithin"></a>ST_WITHIN
`ST_WITHIN` 函式可判斷某個點或多邊形是否在某個多邊形內。 如果該多邊形包含點或多邊形，函式會傳回 1。 如果點或多邊形未在所宣告的多邊形內，則函式會傳回 0。

下列查詢範例使用 `ST_WITHIN` 來判斷交貨目的地所在點是否位於指定的倉儲多邊形內。

```SQL 
SELECT  
     ST_WITHIN(input.deliveryDestination, input.warehouse)  
FROM input 
```  

### <a name="input-example"></a>輸入範例  
  
|deliveryDestination|warehouse|  
|-------------------------|---------------|  
|{“type”:”Point”, “coordinates”: [76.6, 10.1]}|{“type”:”Polygon”, “coordinates”: [ [0.0, 0.0], [10.0, 0.0], [10.0, 10.0], [0.0, 10.0], [0.0, 0.0] ]}|  
|{“type”:”Point”, “coordinates”: [15.0, 15.0]}|{“type”:”Polygon”, “coordinates”: [ [10.0, 10.0], [20.0, 10.0], [20.0, 20.0], [10.0, 20.0], [10.0, 10.0] ]}|  
  
### <a name="output-example"></a>輸出範例  

 0  
  
 1  

若要深入了解，請瀏覽 [ST_WITHIN](https://msdn.microsoft.com/azure/stream-analytics/reference/st-within) 參考。

## <a name="next-steps"></a>後續步驟

* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)