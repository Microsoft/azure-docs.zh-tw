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
ms.openlocfilehash: fb02496d9c107a2c21acca6c65ef69fdfceb4597
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48904572"
---
<a name="paths"></a>
## <a name="paths"></a>路徑

<a name="anomalydetection-post"></a>
### <a name="detect-anomaly-points-for-the-time-series-data-points-requested"></a>針對所要求的時間序列資料點偵測異常點
```
POST /anomalydetection
```


#### <a name="parameters"></a>參數

|類型|名稱|說明|結構描述|
|---|---|---|---|
|**內文**|**body**  <br>*必要*|時間序列資料點和期限 (如果需要)。|[要求](#request)|


#### <a name="responses"></a>回應

|HTTP 代碼|說明|結構描述|
|---|---|---|
|**200**|成功的作業。|< [回應](#response) > 陣列|
|**400**|無法剖析 JSON 要求。|沒有內容|
|**403**|伺服器不接受您提供的憑證。|沒有內容|
|**405**|不允許的方法。|沒有內容|
|**500**|內部伺服器錯誤。|沒有內容|


#### <a name="consumes"></a>取用

* `application/json`


#### <a name="produces"></a>產生

* `application/json`


#### <a name="tags"></a>標記

* anomalydetection



