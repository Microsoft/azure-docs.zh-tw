---
title: 知識探索服務 API 中的語意解譯 | Microsoft Docs
description: 了解如何在認知服務的知識探索服務 (KES) API 中使用語意解譯。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: 022188464eb7269b69f96a058b444167b587387c
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368127"
---
# <a name="semantic-interpretation"></a>語意解譯
語意解譯會讓語意輸出與通過文法的每個已解譯路徑產生關聯。  特別是，服務會針對解譯周遊的 `tag` 元素，評估其中的陳述式順序，以計算最終輸出。  

陳述式可能是指派給另一個變數的常值或變數。  也可以將函式的輸出 0 或多個參數指派給變數。  每個函式參數皆可使用常值或變數來指定。  如果函式未傳回任何輸出，則會略過指派。

```xml
<tag>x = 1; y = x;</tag>
<tag>q = All(); q = And(q, q2);</tag>
<tag>AssertEquals(x, 1);</tag>
```

指定變數時使用的名稱識別碼必須以字母開頭，並且只能包含字母 (A-Z)、數字 (0-9) 和底線 (\_)。  其類型會從指派給它的常值或函式輸出值中間接推斷。 

以下是目前所支援的資料類型清單：

|類型|說明|範例|
|----|----|----|
|字串|0 或更多字元的序列|"Hello World!"<br/>""|
|Bool|布林值|true<br/>false|
|Int32|32 位元帶正負號的整數。  -2.1e9 至 2.1e9|123<br/>-321|
|Int64|64 位元帶正負號的整數。 -9.2e18 和 9.2e18|9876543210|
|兩倍|雙精確度浮點。 1.7e+/-308 (15 位數)|123.456789<br/>1.23456789e2|
|Guid|全域唯一識別碼|"602DD052-CC47-4B23-A16A-26B52D30C05B"|
|查詢|指定索引中資料物件子集的查詢運算式|All()<br/>And(*q1*, *q2*)|

<a name="semantic-functions"></a>
## <a name="semantic-functions"></a>語意函式
有一組語義函式是內建的。  這些函式可允許複雜查詢的建構，並對文法解譯提供內容感知控制。

### <a name="and-function"></a>And 函式
`query = And(query1, query2);`

傳回的查詢是由兩個輸入查詢的交集所組成。

### <a name="or-function"></a>Or 函式
`query = Or(query1, query2);`

傳回的查詢是由兩個輸入查詢的集合聯集所組成。

### <a name="all-function"></a>All 函式
`query = All();`

傳回包含所有資料物件的查詢。

在下列範例中，我們會以 1 或多個關鍵字的交集為基礎，使用 All() 函式來反覆建立查詢。

```
<tag>query = All();</tag>
<item repeat="1-">
  <attrref uri="academic#Keyword" name="keyword">
  <tag>query = And(query, keyword);</tag>
</item>
```

### <a name="none-function"></a>None 函式
`query = None();`

傳回沒有資料物件的查詢。

在下列範例中，我們會以 1 或多個關鍵字的集合聯集為基礎，使用 None() 函式來反覆建立查詢。

```
<tag>query = None();</tag>
<item repeat="1-">
  <attrref uri="academic#Keyword" name="keyword">
  <tag>query = Or(query, keyword);</tag>
</item>
```

### <a name="query-function"></a>Query 函式
```
query = Query(attrName, value)
query = Query(attrName, value, op)
```

傳回的查詢中僅包含根據指定 op 作業 (預設為 "eq")，其 attrName 屬性符合 value 值的資料物件。  一般會使用 `attrref` 元素根據相符的輸入查詢字串來建立查詢。  如果值已指定，或透過其他方式取得，Query() 函式可用來建立符合此值的查詢。

在下列範例中，我們會使用 Query() 函式來實作從某十年間指定學術刊物的支援。

```xml
written in the 90s
<tag>
  beginYear = Query("academic#Year", 1990, "ge");
  endYear = Query("academic#Year", 2000, "lt");
  year = And(beginYear, endYear);
</tag>
```

<a name="composite-function"/>
### <a name="composite-function"></a>Composite 函式
`query = Composite(innerQuery);`

其所傳回的查詢會封裝 innerQuery，而構成此 innerQuery 的值是與常見複合屬性 attr 的子屬性作比對後，所得到的相符值。  任何相符資料物件的複合屬性 attr 至少要有一個個別滿足 innerQuery 的值，才會封裝。  請注意，針對複合屬性子屬性所建立的查詢必須先使用 Composite() 函式加以封裝，才能與其他查詢結合。

例如，下列查詢會傳回 "harry shum" 任職於 "microsoft" 時所撰寫的學術刊物：
```
Composite(And(Query("academic#Author.Name", "harry shum"), 
              Query("academic#Author.Affiliation", "microsoft")));
```

另一方面，下列查詢則會傳回其中一位作者是 "harry shum" 以及其中一個所屬單位是 "microsoft" 的學術刊物：
```
And(Composite(Query("academic#Author.Name", "harry shum"), 
    Composite(Query("academic#Author.Affiliation", "microsoft")));
```

### <a name="getvariable-function"></a>GetVariable 函式
`value = GetVariable(name, scope);`

傳回 name 變數的值，其定義於指定的「範圍」下方。  name 識別碼必須以字母開頭，並且只能包含字母 (A-Z)、數字 (0-9) 和底線 (_)。  「範圍」可以設定為 "request" 或 "system"。  請注意，不同範圍下定義的變數彼此相異，其中包括透過語意函式輸出定義的變數。

目前解譯要求中的所有解譯會共用要求範圍變數。  這些變數可以用來控制透過文法的解譯搜尋。

系統變數會由服務預先定義，並可用來擷取系統目前狀態的各種相關統計資料。  以下是目前支援的系統變數集合：

|Name|類型|說明|
|----|----|----|
|IsAtEndOfQuery|Bool|如果目前的解譯符合所有輸入查詢文字，則成立|
|IsBeyondEndOfQuery|Bool|如果目前的解譯有超出輸入查詢文字的完成建議，則成立|

### <a name="setvariable-function"></a>SetVariable 函式
`SetVariable(name, value, scope);`

將 value 指派給指定「範圍」下的 name 變數。  name 識別碼必須以字母開頭，並且只能包含字母 (A-Z)、數字 (0-9) 和底線 (_)。  目前，「範圍」的唯一有效值為 "request"。  沒有任何可設定的系統變數。

目前解譯要求中的所有解譯會共用要求範圍變數。  這些變數可以用來控制透過文法的解譯搜尋。

### <a name="assertequals-function"></a>AssertEquals 函式
`AssertEquals(value1, value2);`

如果 value1 和 value2 是相等的，則此函式會成功，而且沒有副作用。  否則，函式會失敗並拒絕解譯。

### <a name="assertnotequals-function"></a>AssertNotEquals 函式
`AssertNotEquals(value1, value2);`

如果 value1 和 value2 不相等，則此函式會成功，而且沒有副作用。  否則，函式會失敗並拒絕解譯。


