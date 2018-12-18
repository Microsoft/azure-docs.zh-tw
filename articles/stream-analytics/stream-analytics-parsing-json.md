---
title: 在 Azure 串流分析中剖析 JSON 和 AVRO
description: 本文說明如何對複雜資料類型 (如 陣列、JSON、CSV 格式的資料) 進行作業。
services: stream-analytics
ms.service: stream-analytics
author: jasonwhowell
ms.author: jasonh
manager: kfile
ms.topic: conceptual
ms.date: 08/03/2018
ms.openlocfilehash: 208dfd1d78437e4ae8270d599f6365ad70a708c7
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39580137"
---
# <a name="parse-json-and-avro-data-in-azure-stream-analytics"></a>在 Azure 串流分析中剖析 JSON 和 Avro 資料

Azure 串流分析可處理資料格式為 CSV、JSON 和 Avro 的事件。 JSON 和 Avro 資料都可以包含巢狀物件 (資料列) 和陣列等複雜類型。 
  
## <a name="array-data-types"></a>陣列資料類型  
陣列資料類型是已排序的值集合。 以下詳述陣列值的一些典型作業。 這些範例假設輸入事件具有名為 "arrayField" 的屬性，其資料類型為陣列。

這些範例使用函式 [GetArrayElement](https://msdn.microsoft.com/azure/stream-analytics/reference/getarrayelement-azure-stream-analytics)、[GetArrayElements](https://msdn.microsoft.com/azure/stream-analytics/reference/getarrayelements-azure-stream-analytics)、[GetArrayLength](https://msdn.microsoft.com/azure/stream-analytics/reference/getarraylength-azure-stream-analytics)，以及 [APPLY](https://msdn.microsoft.com/azure/stream-analytics/reference/apply-azure-stream-analytics) 運算子。

## <a name="examples"></a>範例  
 選取在指定索引的陣列元素 (選取第一個陣列元素)：  
  
```SQL 
SELECT   
    GetArrayElement(arrayField, 0) AS firstElement  
FROM input  
```  
  
 選取陣列長度：  
  
```SQL  
SELECT   
    GetArrayLength(arrayField) AS arrayLength  
FROM input  
```  
  
以個別事件選取所有陣列元素。 [APPLY](https://msdn.microsoft.com/azure/stream-analytics/reference/apply-azure-stream-analytics) 運算子和 [GetArrayElements](https://msdn.microsoft.com/azure/stream-analytics/reference/getarrayelements-azure-stream-analytics) 內建函式一起擷取所有陣列元素且視為個別事件：  
  
```SQL  
SELECT   
    arrayElement.ArrayIndex,  
    arrayElement.ArrayValue  
FROM input as event  
CROSS APPLY GetArrayElements(event.arrayField) AS arrayElement  
```  
  
## <a name="record-data-types"></a>資料列資料類型  
當輸入資料流中是使用資料列資料類型時，此資料類型會用來代表 JSON 和 Avro。 這些範例示範讀取 JSON 格式輸入事件的範例感應器。 以下是單一事件的範例：
  
```json  
{  
    "DeviceId" : "12345",  
    "Location" : 
    {
        "Lat": 47,
        "Long": 122 
    },  
    "SensorReadings" :  
    {  
        "Temperature" : 80,  
        "Humidity" : 70,  
        "CustomSensor01" : 5,  
        "CustomSensor02" : 99  
    }  
}  
```  
  
## <a name="examples"></a>範例  
使用點標記法 (.) 存取巢狀欄位。 例如，此查詢選取上述 JSON 資料中 Location 屬性下的 Lat 和 Long 座標： 
  
```SQL  
SELECT  
    DeviceID,  
    Location.Lat,  
    Location.Long  
FROM input  
```  

如果屬性名稱為未知，請使用 [GetRecordPropertyValue](https://msdn.microsoft.com/azure/stream-analytics/reference/getrecordpropertyvalue-azure-stream-analytics) 函式。 例如，假設範例資料流需要與包含每個裝置感應器閾值的參考資料聯結：  

```json  
{  
    "DeviceId" : "12345",  
    "SensorName" :  "Temperature",
    "Value" : 75
}  
```  
  
```SQL  
SELECT  
    input.DeviceID,  
    thresholds.SensorName  
FROM input  
JOIN thresholds  
ON  
    input.DeviceId = thresholds.DeviceId  
WHERE  
    GetRecordPropertyValue(input.SensorReadings, thresholds.SensorName) > thresholds.Value  
```  
  
若要將資料列欄位轉換成個別的事件，請搭配使用 [APPLY](https://msdn.microsoft.com/azure/stream-analytics/reference/apply-azure-stream-analytics) 運算子和 [GetRecordProperties](https://msdn.microsoft.com/azure/stream-analytics/reference/getrecordproperties-azure-stream-analytics) 函式。 例如，若要將範例資料流轉換成包含個別感應器讀取之資料的事件，您可以使用此查詢：  
  
```SQL  
SELECT   
    event.DeviceID,  
    sensorReading.PropertyName,  
    sensorReading.PropertyValue  
FROM input as event  
CROSS APPLY GetRecordProperties(event.SensorReadings) AS sensorReading  
```  

您可以使用 '*' 萬用字元選取巢狀資料列的所有屬性。 請思考下列範例：  

```SQL  
SELECT input.SensorReadings.*  
FROM input  
```  

結果如下：  

```json  
{  
    "Temperature" : 80,  
    "Humidity" : 70,  
    "CustomSensor01" : 5,  
    "CustomSensor022" : 99  
}  
```  
  
## <a name="see-also"></a>另請參閱  
 [Azure 串流分析中的資料類型](https://msdn.microsoft.com/azure/stream-analytics/reference/data-types-azure-stream-analytics) \(英文\)  
