---
title: 包含檔案
description: 包含檔案
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.component: anomaly-finder
ms.topic: include
ms.date: 04/13/2018
ms.author: chliang
ms.custom: include file
ms.openlocfilehash: a49df0c18ef9db9d0d41ca2e714474e6386ae7d1
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48904515"
---
<a name="definitions"></a>
## <a name="definitions"></a>定義

<a name="point"></a>
### <a name="point"></a>Point

|名稱|說明|結構描述|
|---|---|---|
|**Timestamp**  <br>*選用*|資料點的時間戳記。 請確定與午夜對應，並使用 UTC 日期時間字串，例如 2017-08-01T00:00:00Z。|字串 (日期-時間)|
|**值**  <br>*選用*|資料量值。|數字 (雙精度)|


<a name="request"></a>
### <a name="request"></a>要求

|名稱|說明|結構描述|
|---|---|---|
|**期間**  <br>*選用*|資料點的期間。 如果值為 null 或未顯示，API 將會自動決定該期間。|數字 (雙精度)|
|**點**  <br>*選用*|時間序列資料點。 資料應該依照時間戳記遞增排序，以便與異常結果對應。 如果資料未正確排序或時間戳記重複，API 會正確偵測到異常點，但您也可能無法確實對應到輸入回傳的資料點。 在這種情況下，將在回應中新增警告訊息。|< [點](#point) > 陣列|


<a name="response"></a>
### <a name="response"></a>Response

|名稱|說明|結構描述|
|---|---|---|
|**ExpectedValues**  <br>*選用*|學習型模型的預測值。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應期望值和原始值。|< 數字 (雙精度) > 陣列|
|**IsAnomaly**  <br>*選用*|資料點是否比預測值要高或低，為異常的結果 (尖峰或下降)。 true 表示資料點異常，false 表示資料點非異常。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應期望值和原始值。|< 布林值 > 陣列|
|**IsAnomaly_Neg**  <br>*選用*|資料點是否為負異常 (下降)。 true 表示異常的方向為負向。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應期望值和原始值。|< 布林值 > 陣列|
|**IsAnomaly_Pos**  <br>*選用*|資料點是否為正異常 (尖峰)。 true 表示異常的方向為正向。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應預期值和原始值。|< 布林值 > 陣列|
|**LowerMargin**  <br>*選用*|(ExpectedValue - LowerMargin) 決定資料點仍屬於正常範圍的下限。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應期望值和原始值。|< 數字 (雙精度) > 陣列|
|**期間**  <br>*選用*|API 用來偵測異常點的期間。|數字 (浮點數)|
|**UpperMargin**  <br>*選用*|ExpectedValue 和 UpperMargin 的總和決定資料點仍屬於正常範圍的上限。 如果輸入資料點是依照時間戳記遞增排序，陣列的索引可以用於對應期望值和原始值。|< 數字 (雙精度) > 陣列|
|**WarningText**  <br>*選用*|如果提供的輸入資料點並未遵照 API 要求的規則，並且 API 仍然可以偵測該資料，則 API 將分析資料，並在此欄位中附加警告資訊。|字串|



