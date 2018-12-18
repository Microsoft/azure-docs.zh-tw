---
title: 知識探索服務 API 中的資料格式 | Microsoft Docs
description: 了解認知服務中知識探索服務 (KES) API 中的資料格式。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: a763505ac6458d68df74ae73e71029b81202ec8b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367906"
---
# <a name="data-format"></a>資料格式
資料檔案會描述要編製索引的物件清單。
檔案中的每一行都會以 [JSON 格式](http://json.org/)搭配 UTF-8 編碼方式，指定物件的屬性值。
除了[結構描述](SchemaFormat.md)中所定義的屬性之外，每個物件都具有選擇性 "logprob" 屬性，而該屬性可指定物件之間的相對對數機率。
當服務傳回物件以便降低機率時，我們可以使用 "logprob" 來表示相符物件的傳回順序。
假設 0 和 1 之間的概率為 p，相對應的對數概率可計算為 log(p)，其中 log() 是自然對數函式。
若未針對 logprob 指定任何值，則會使用預設值 0。

```json
{"logprob":-5.3, "Title":"latent dirichlet allocation", "Year":2003, "Author":{"Name":"david m blei", "Affiliation":"uc berkeley"}, "Author":{"Name":"andrew y ng", "Affiliation":"stanford"}, "Author":{"Name":"michael i jordan", "Affiliation":"uc berkeley"}}
{"logprob":-6.1, "Title":"probabilistic latent semantic indexing", "Year":1999, "Author":{"Name":"thomas hofmann", "Affiliation":"uc berkeley"}}
...
```

對於字串、GUID 及 Blob 屬性，此值會以加上引號的 JSON 字串表示。  對於數值屬性 (Int32、Int64、Double)，此值會以 JSON 數字表示。  對於複合屬性，此值為可指定子屬性值的 JSON 物件。  若要更快建立索引，請藉由降低對數機率來預先排序物件。

一般而言，屬性可能有 0 個或更多值。  如果屬性沒有值，我們只會從 JSON 中卸除它。  如果屬性有 2 個或更多值，我們可以在 JSON 物件中重複屬性值組。  或者，我們可以指派 JSON 陣列，其中包含此屬性的多個值。

```json
{"logprob":0, "Title":"0 keyword"}
{"logprob":0, "Title":"1 keyword", "Keyword":"foo"}
{"logprob":0, "Title":"2 keywords", "Keyword":"foo", "Keyword":"bar"}
{"logprob":0, "Title":"2 keywords (alt)", "Keyword":["foo", "bar"]}
```
