---
title: 使用參數化 URL 共用 Azure 時間序列深入解析自訂檢視 | Microsoft Docs
description: 本文說明如何在 Azure 時間序列深入解析中開發參數化 URL，以便輕鬆地共用客戶檢視。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.topic: conceptual
ms.workload: big-data
ms.date: 11/21/2017
ms.openlocfilehash: dbd717d79230c5dc2fc823484376267e7b7b8ab1
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628795"
---
# <a name="share-a-custom-view-using-a-parameterized-url"></a>使用參數化 URL 共用自訂檢視

若要在時間序列深入解析總管中共用自訂檢視，您可以程式設計方式建立自訂檢視的參數化 URL。

時間序列深入解析總管支援 URL 查詢參數，以便直接從 URL 指定檢視。  例如，僅使用 URL，您就可以指定目標環境、搜尋述詞，以及所需的時間範圍。 當使用者按一下自訂的 URL 時，介面會在時間序列深入解析入口網站中提供該資產的直接連結。  適用資料存取原則。 

## <a name="environment-id"></a>環境識別碼

`environmentId=<guid>` 參數可指定目標環境識別碼。  這是資料存取 FQDN 的元件，而您可以在 Azure 入口網站中環境概觀的右上角找到此元件。  這是在 `env.timeseries.azure.com` 之前的所有內容。 範例環境識別碼參數為 `?environmentId=10000000-0000-0000-0000-100000000108`。

## <a name="time"></a>時間

您可以使用參數化 URL 來指定絕對或相對時間值。

### <a name="absolute-time-values"></a>絕對時間值

對於絕對時間值，使用 `from=<integer>` 和 `to=<integer>` 參數。 

`from=<integer>` 是搜尋範圍的開始時間值 (以 JavaScript 毫秒為單位)。

`to=<integer>` 是搜尋範圍的結束時間值 (以 JavaScript 毫秒為單位)。 

若要識別日期的 JavaScript 毫秒，請參閱 [Epoch 和 Unix 時間戳記轉換器](https://www.freeformatter.com/epoch-timestamp-to-date-converter.html)。

### <a name="relative-time-values"></a>相對時間值

對於相對時間值，使用 `relativeMillis=<value>`，其中 value 是後端最近資料中的時間值 (以 JavaScript 毫秒為單位)。

例如，`&relativeMillis=3600000` 可顯示最近 60 分鐘的資料。

接受的值會對應至時間序列深入解析總管 [快速時間] 功能表，並且包含下列項目：

- 1800000 (過去 30 分鐘)
- 3600000 (過去 60 分鐘)
- 10800000 (過去 3 小時)
- 21600000 (過去 6 小時)
- 43200000 (過去 12 小時)
- 86400000 (過去 24 小時)
- 604800000 (過去 7 天)
- 2592000000 (過去 30 小時)

### <a name="optional-parameters"></a>選擇性參數

`timeSeriesDefinitions=<collection of term objects>` 參數可指定時間序列深入解析檢視的字詞，其中：

- "name":"<string>"
  - term 的名稱。
- "splitBy":"<string>"
  - split by 的資料行名稱。
- "measureName":"<string>"
  - measure 的資料行名稱。
- "predicate":"<string>"
  - 用於伺服器端篩選的 where 子句。
-  "useSum":"true"
  - 這是選擇性參數，指定使用總和作為您的量值。  請注意，如果 [事件] 是選取的量值，則預設會選取 [計數]。  如果未選取 [事件]，則預設會選取 [平均值]。  

'multiChartStack=<true/false>' 參數可在圖表中啟用堆疊，而 'multiChartSameScale=<true/false>' 參數可在選用參數內的不同時段啟用相同的 Y 軸刻度。  

- 'multiChartStack=false'
  - 預設會啟用 'True'，所以傳遞 'false' 進行堆疊。
- 'multiChartStack=false&multiChartSameScale=true' 
  - 必須啟用堆疊，才能在不同時段使用相同的 Y 軸刻度。  如果預設為 'false'，則傳遞 'true' 可啟用此功能。  
  
'timeBucketUnit=<Unit>&timeBucketSize=<integer>' 可讓您調整間隔滑桿，以提供更精細或更平滑、更整體的圖表檢視。  
- 'timeBucketUnit=<Unit>&timeBucketSize=<integer>'
  - 單位 = 天、小時、分鐘、秒、毫秒。  單位一律大寫。
  - 對 timeBucketSize 傳遞所需的整數，以定義單位數。  請注意，您最多可以平滑處理 7 天。  
  
'timezoneOffset=<integer>' 參數可讓您針對要以 UTC 位移檢視的圖表設定時區。  
  - 'timezoneOffset=-<integer>'
    - 整數一律以毫秒為單位。  
    - 請注意，這項功能與我們在 TSI 總管中啟用的功能稍有不同，在 TSI 總管中我們可讓您選擇本機 (瀏覽器時間) 或 UTC。  
 
### <a name="examples"></a>範例

例如，若要將時間序列定義新增為 URL 參數，您可以使用下列：

```https
&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},
{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

將這些範例時間序列定義使用於 

- 環境識別碼
- 過去 60 分鐘的資料
- 構成選擇性參數的字詞 (F1PressureID、F2TempStation 和 F3VibrationPL)
 
您可以為檢視建構下列參數化 URL：

```https
https://insights.timeseries.azure.com/samples?environmentId=10000000-0000-0000-0000-100000000108&relativeMillis=3600000&timeSeriesDefinitions=[{"name":"F1PressureId","splitBy":"Id","measureName":"Pressure","predicate":"'Factory1'"},{"name":"F2TempStation","splitBy":"Station","measureName":"Temperature","predicate":"'Factory2'"},{"name":"F3VibrationPL","splitBy":"ProductionLine","measureName":"Vibration","predicate":"'Factory3'"}]
```

如果您已使用時間序列深入解析總管來建立上述 URL 所描述的檢視，它會如下所示：

![時間序列深入解析總管字詞](media/parameterized-url/url1.png)

完整檢視 (包括圖表) 會如下所示：

![圖表檢視](media/parameterized-url/url2.png)

## <a name="next-steps"></a>後續步驟
[使用 C# 查詢資料](time-series-insights-query-data-csharp.md)
