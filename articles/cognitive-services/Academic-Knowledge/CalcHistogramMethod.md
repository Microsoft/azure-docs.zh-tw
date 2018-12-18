---
title: 學術知識 API 中的 CalcHistogram 方法 | Microsoft Docs
description: 使用 CalcHistogram 方法來計算 Microsoft 認知服務中一組論文實體的屬性值分佈。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/27/2017
ms.author: alch
ms.openlocfilehash: e0b773fb9791ee638c8cfdbbc9dca40543e50ec0
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367871"
---
# <a name="calchistogram-method"></a>CalcHistogram 方法

**calchistogram** REST API 用來計算一組論文實體的屬性值分佈。          


**REST 端點：**
```
https:// westus.api.cognitive.microsoft.com/academic/v1.0/calchistogram?
``` 
<br>
  
## <a name="request-parameters"></a>要求參數

Name  |值 | 必要？  |說明
-----------|----------|--------|----------
**expr**    |文字字串 | yes  |查詢運算式，用來指定用於計算長條圖的實體。
**model** |文字字串 | 否 |選取您想要查詢的模型名稱。  目前，此值會預設為 latest。
**attributes** | 文字字串 | 否<br>預設值： | 以逗號分隔的清單，可指定回應中包含的屬性值。 屬性名稱區分大小寫。
**count** |數字 | 否<br>預設值：10 |要傳回的結果數目。
**offset**  |數字 | 否<br>預設值：0 |要傳回的第一個結果索引。
<br>
## <a name="response-json"></a>回應 (JSON)
Name | 說明
--------|---------
**expr**  |要求中的 expr 參數。
**num_entities** | 相符實體的總數。
**histograms** |  長條圖的陣列，要求中指定的每個屬性各有一個長條圖。
**histograms[x].attribute** | 用於計算長條圖的屬性名稱。
**histograms[x].distinct_values** | 此屬性的相符實體當中的相異值數目。
**histograms[x].total_count** | 此屬性的相符實體當中的值執行個體總數。
**histograms[x].histogram** | 此屬性的長條圖資料。
**histograms[x].histogram[y].value** |  屬性的值。
**histograms[x].histogram[y].logprob**  |這個屬性值相符實體的總計自然對數機率。
**histograms[x].histogram[y].count**  |這個屬性值的相符實體數目。
**已中止** | 如果要求逾時，則為 true。

 <br>
#### <a name="example"></a>範例：
```
https:// westus.api.cognitive.microsoft.com/academic/v1.0/calchistogram?expr=And(Composite(AA.AuN=='jaime teevan'),Y>2012)&attributes=Y,F.FN&count=4
```
<br>在此範例中，若要針對特定作者產生自 2010 年起每年的出版品計數長條圖，我們可以先使用**解譯** API 搭配查詢字串來產生查詢運算式：*papers by jaime teevan after 2012*。

```
https:// westus.api.cognitive.microsoft.com/academic/v1.0/interpret?query=papers by jaime teevan after 2012
```
<br>第一個解譯中從解譯 API 傳回的運算式為 *And(Composite(AA.AuN=='jaime teevan'),Y>2012)*。
<br>此運算式值會接著傳遞至 **calchistogram** API。 *attributes=Y,F.FN* 參數表示論文計數應該依年度和研究領域分佈，例如：
```
https:// westus.api.cognitive.microsoft.com/academic/v1.0/calchistogram?expr=And(Composite(AA.AuN=='jaime teevan'),Y>2012)&attributes=Y,F.FN&count=4
```
<br>此要求的回應先指出有 37 篇符合查詢運算式的論文。  對於 Year  屬性，有 3 個相異值，如查詢中所指定，在 2012 年後每年一個值 (也就是 2013、2014 和 2015)。  3 個相異值的總計論文計數為 37。  對於每個 Year，長條圖會顯示值、總計自然對數可能性及相符實體計數。     

「研究領域」的長條圖顯示有 34 個不同的研究領域。 因為一篇論文可能與多個研究領域相關聯，所以總計數 (53) 可以大於相符的實體數字。  雖然有 34 個相異值，但回應只包含前 4 個值，因為 *count=4* 參數。

```JSON
{
  "expr": "And(Composite(AA.AuN=='jaime teevan'),Y>2012)",
  "num_entities": 37,
  "histograms": [
    {
      "attribute": "Y",
      "distinct_values": 3,
      "total_count": 37,
      "histogram": [
        {
          "value": 2014,
          "logprob": -15.753,
          "count": 15
        },
        {
          "value": 2013,
          "logprob": -15.805,
          "count": 12
        },
        {
          "value": 2015,
          "logprob": -16.035,
          "count": 10
        }
      ]
    },
    {
      "attribute": "F.FN",
      "distinct_values": 34,
      "total_count": 53,
      "histogram": [
        {
          "value": "crowdsourcing",
          "logprob": -15.258,
          "count": 9
        },
        {
          "value": "information retrieval",
          "logprob": -16.002,
          "count": 4
        },
        {
          "value": "personalization",
          "logprob": -16.226,
          "count": 3
        },
        {
          "value": "mobile search",
          "logprob": -17.228,
          "count": 2
        }
      ]
    }
  ]
}
```
