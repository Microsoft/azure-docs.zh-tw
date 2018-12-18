---
title: 知識探索服務 API 中的文法格式 | Microsoft Docs
description: 深入了解認知服務中的知識探索服務 (KES) API 文法格式。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: b64025be2f5a9708162da475c1f037d7f253d2c6
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37865748"
---
# <a name="grammar-format"></a>文法格式
文法是一個 XML 檔案，其會指定服務可以解譯的一組加權自然語言查詢，以及將這些自然語言查詢轉譯為語意查詢運算式的方式。  文法的語法是以 [SRGS](http://www.w3.org/TR/speech-grammar/) 為基礎，也就是語音辨識文法的 W3C 標準，其包含支援資料索引整合和語意函式的擴充功能。

下列將描述文法中可用的每個語法元素。  請參閱[此範例](#example)以了解在內容中決定這些元素用法的完整文法。

### <a name="grammar-element"></a>grammar 元素 
`grammar` 元素是文法規格 XML 中的最上層元素。  必要屬性 `root` 會為定義文法起點的根規則定義名稱。

```xml
<grammar root="GetPapers">
```

### <a name="import-element"></a>import 元素
`import` 元素會從外部檔案匯入結構描述定義以啟用屬性參考。 此元素必須是最上層 `grammar` 元素的子系，並且在任何 `attrref` 元素之前出現。 必要屬性 `schema` 會指定結構描述檔案的名稱，結構描述檔案與文法 XML 檔案位於相同目錄。 必要元素 `name` 會指定結構描述別名，後續 `attrref` 元素在參考此結構描述中定義的屬性時會用到此別名。

```xml
  <import schema="academic.schema" name="academic"/>
```

### <a name="rule-element"></a>rule 元素
`rule` 元素會定義文法規則，此結構化單位會指定一組系統可解譯的查詢運算式。  此元素必須是最上層 `grammar` 元素的子系。  必要屬性 `id` 會指定規則的名稱，並由 `grammar` 或 `ruleref` 元素進行參考。

`rule` 元素會定義一組合法的擴充。  文字權杖會直接與輸入查詢進行比對。  `item` 元素會指定重複次數和改變解譯機率。  `one-of` 元素代表替代選項。  `ruleref` 元素會允許從較簡單的擴充中建構較複雜的擴充。  `attrref` 元素會允許從索引中比對屬性值。  `tag` 元素會指定解譯的語意及修改解譯機率。

```xml
<rule id="GetPapers">...</rule>
```

### <a name="example-element"></a>example 元素
選用元素 `example` 會指定其中 `rule` 定義可接受的範例片語。  這可用於文件和/或自動化測試。

```xml
<example>papers about machine learning by michael jordan</example>
```

### <a name="item-element"></a>item 元素
`item` 元素會群組文法建構的序列。  這可以用來表示擴充序列的重複次數，或指定與 `one-of` 元素搭配的替代項目。

當 `item` 元素不是 `one-of` 元素的子系時，其可藉由將 `repeat` 屬性指派給計數值來為括住的序列指定重複次數。  計數值 "n" (其中 n 是一個整數) 表示序列必須出現剛好 n 次。  計數值 "m-n" 表示可允許序列出現 m 到 n 次 (含)。  計數值 "m-" 會指定序列必須至少出現 m 次。  針對每個超過最小值的額外重複，選用屬性 `repeat-logprob` 可用來改變其解譯機率。

```xml
<item repeat="1-" repeat-logprob="-10">...</item>
```

當 `item` 元素顯示為 `one-of` 元素的子系時，這些元素會定義一組擴充的替代項目。  在此用法中，選用屬性 `logprob` 會指定不同選擇間的相對對數概率。  假設 0 和 1 之間的概率為 p，相對應的對數概率可計算為 log(p)，其中 log() 是自然對數函式。  如果未指定，`logprob` 的預設值為 0，並且不會改變解譯機率。  請注意，對數概率永遠為負數的浮點值或 0。

```xml
<one-of>
  <item>by</item>
  <item logprob="-0.5">written by</item>
  <item logprob="-1">authored by</item>
</one-of>
```

### <a name="one-of-element"></a>one-of 元素
`one-of` 元素會對其中一個子系 `item` 元素指定替代的擴充。  只有 `item` 元素中可能出現在 `one-of` 元素中。  不同選擇間的相對機率可透過每個子系 `item` 中的 `logprob` 屬性來指定。

```xml
<one-of>
  <item>by</item>
  <item logprob="-0.5">written by</item>
  <item logprob="-1">authored by</item>
</one-of>
```

### <a name="ruleref-element"></a>ruleref 元素
`ruleref` 元素會透過另一個 `rule` 元素的參考來指定有效擴充。  透過 `ruleref` 元素的使用，您可以從較簡單的規則中建立較複雜的運算式。  必要屬性 `uri` 會使用 "#*rulename*" 語法指出所參考 `rule` 的名稱。  若要擷取所參考規則的語意輸出，請使用選用屬性 `name` 來指定獲派語意輸出的變數名稱。
 
```xml
<ruleref uri="#GetPaperYear" name="year"/>
```

### <a name="attrref-element"></a>attrref 元素
`attrref` 元素會參考索引屬性，以便從索引中查看針對屬性值進行的比對。  必要屬性 `uri` 會使用 "schemaName#attrName" 語法來指定索引結構描述名稱和屬性名稱。  前面必須有 `import` 元素來匯入名為 schemaName 的結構描述。  屬性名稱是相對應結構描述中定義的屬性名稱。

除了比對使用者輸入，`attrref` 元素也會傳回結構化的查詢物件作為輸出，此輸出會選取索引中對比輸入值的物件子集。  使用選用屬性 `name` 來指定儲存查詢物件輸出的變數名稱。  撰寫查詢物件時可搭配其他查詢物件，以形成更多複雜的運算式。  請參閱[語意解譯](SemanticInterpretation.md)來了解詳細資訊。  

```xml
<attrref uri="academic#Keyword" name="keyword"/>
```

#### <a name="query-completion"></a>查詢完成 
若要在解譯部分使用者查詢時支援查詢完成，每個參考的屬性都必須在結構描述定義中包含 "starts_with" 作業。  如果提供使用者查詢前置詞，`attrref` 會比對索引中能完成前置詞的所有值，並將每個完成值產生為文法的不同解譯。  

範例：
* 針對查詢前置詞 "dat" 比對 `<attrref uri="academic#Keyword" name="keyword"/>` 會產生一個有關 "database" (資料庫) 的論文解譯、一個有關 "data mining" (資料採礦) 的論文解譯等等。
* 針對查詢前置詞 "200" 比對 `<attrref uri="academic#Year" name="year"/>` 會產生一個 "2000" 年的論文解譯、一個 "2001" 年的論文解譯等等。

#### <a name="matching-operations"></a>比對作業
除了完全相符的比對外，選取屬性類型也可透過選用屬性 `op` 來支援前置詞和不相等比對。  如果索引中沒有物件有符合的值，文法路徑會遭到封鎖，而服務將不會產生任何透過此文法路徑周遊的解譯。   `op` 屬性的預設值為 "eq"。

```xml
in <attrref uri="academic#Year" name="year"/>
before <attrref uri="academic#Year" op="lt" name="year"/
```

下表列出每個屬性類型支援的 `op` 值。  其用法需要將對應的索引作業包含在結構描述屬性定義中。

| 屬性類型 | Op 值 | 說明 | 索引作業
|----|----|----|----|
| 字串 | eq | 字串完全相符的比對 | equals |
| 字串 | starts_with | 字串前置詞的比對 | starts_with |
| Int32、Int64、Double | eq |  數字相等的比對 | equals |
| Int32、Int64、Double | lt、le、gt、ge | 數字不相等的比對 (<、<=、>、>=) | is_between |
| Int32、Int64、Double | starts_with | 對十進位表示法中的值進行前置詞比對 | starts_with |

範例：
* `<attrref uri="academic#Year" op="lt" name="year"/>` 會比對輸入的字串 "2000"，並傳回 2000 年之前 (不含 2000 年) 發行的所有論文。
* `<attrref uri="academic#Year" op="lt" name="year"/>` 不會比對輸入的字串 "20"，因為索引中沒有 20 年之前發行的論文。
* `<attrref uri="academic#Keyword" op="starts_with" name="keyword"/>` 會比對輸入的字串 "dat"，並傳回有關 "database" (資料庫)、"data mining" (資料採礦) 等單一解譯論文。這是罕見的使用案例。
* `<attrref uri="academic#Year" op="starts_with" name="year"/>` 會比對輸入的字串 "20"，並傳回在 200-299、2000-2999 等年度發行的單一解譯論文。這是罕見的使用案例。

### <a name="tag-element"></a>tag 元素
`tag` 元素會指定路徑如何通過文法來進行解譯。  其包含一系列以分號結束的陳述式。  陳述式可能是指派給另一個變數的常值或變數。  也可以將函式的輸出 0 或多個參數指派給變數。  每個函式參數皆可使用常值或變數來指定。  如果函式未傳回任何輸出，則會略過指派。  變數範圍在內含規則中。

```xml
<tag>x = 1; y = x;</tag>
<tag>q = All(); q = And(q, q2);</tag>
<tag>AssertEquals(x, 1);</tag>
```

文法中的每個 `rule` 都有預先定義且名為 "out" 的變數，代表規則的語意輸出。  其值是藉由評估每個語意陳述式而計算出來的，周遊語意陳述式的路徑會通過比對使用者查詢輸入的 `rule`。  在評估結束時，即是將規則的語意輸出值指派給 "out" 變數。  依據文法來解譯使用者查詢的語意輸出是根規則語意輸出。

某些陳述式可能會藉由引入加法的對數概率位移，來改變解譯路徑的機率。  如果未滿足指定的條件，某些陳述式可能會全然拒絕解譯。

如需支援的語意函式清單，請參閱[語意函式](SemanticInterpretation.md#semantic-functions)。

## <a name="interpretation-probability"></a>解譯機率
通過文法的解譯路徑機率是，所有 `<item>` 元素及在過程中所遇到語意函數的累積對數概率。  其說明比對特定輸入序列時的相對可能性。

假設 0 和 1 之間的概率為 p，相對應的對數概率可計算為 log(p)，其中 log() 是自然對數函式。  使用對數概率可讓系統透過簡單的加法，來累積解譯路徑的聯合機率。  也可避免此類聯合機率計算常見的浮點反向溢位。  請注意，根據設計，對數概率永遠是負數的浮點值或 0，其中較大的值表示較高的可能性。

<a name="example"></a>
## <a name="example"></a>範例
下列是學術刊物領域的 XML 範例，用於示範文法的各種元素：

```xml
<grammar root="GetPapers">

  <!-- Import academic data schema-->
  <import schema="academic.schema" name="academic"/>
  
  <!-- Define root rule-->
  <rule id="GetPapers">
    <example>papers about machine learning by michael jordan</example>
    
    papers
    <tag>
      yearOnce = false;
      isBeyondEndOfQuery = false;
      query = All();
    </tag>
  
    <item repeat="1-" repeat-logprob="-10">
      <!-- Do not complete additional attributes beyond end of query -->
      <tag>AssertEquals(isBeyondEndOfQuery, false);</tag>
        
      <one-of>
        <!-- about <keyword> -->
        <item logprob="-0.5">
          about <attrref uri="academic#Keyword" name="keyword"/>
          <tag>query = And(query, keyword);</tag>
        </item>
        
        <!-- by <authorName> [while at <authorAffiliation>] -->
        <item logprob="-1">
          by <attrref uri="academic#Author.Name" name="authorName"/>
          <tag>authorQuery = authorName;</tag>
          <item repeat="0-1" repeat-logprob="-1.5">
            while at <attrref uri="academic#Author.Affiliation" name="authorAffiliation"/>
            <tag>authorQuery = And(authorQuery, authorAffiliation);</tag>
          </item>
          <tag>
            authorQuery = Composite(authorQuery);
            query = And(query, authorQuery);
          </tag>
        </item>
        
        <!-- written (in|before|after) <year> -->
        <item logprob="-1.5">
          <!-- Allow this grammar path to be traversed only once -->
          <tag>
            AssertEquals(yearOnce, false);
            yearOnce = true;
          </tag>
          <ruleref uri="#GetPaperYear" name="year"/>
          <tag>query = And(query, year);</tag>
        </item>
      </one-of>

      <!-- Determine if current parse position is beyond end of query -->
      <tag>isBeyondEndOfQuery = GetVariable("IsBeyondEndOfQuery", "system");</tag>
    </item>
    <tag>out = query;</tag>
  </rule>
  
  <rule id="GetPaperYear">
    <tag>year = All();</tag>
    written
    <one-of>
      <item>
        in <attrref uri="academic#Year" name="year"/>
      </item>
      <item>
        before
        <one-of>
          <item>[year]</item>
          <item><attrref uri="academic#Year" op="lt" name="year"/></item>
        </one-of>
      </item>
      <item>
        after
        <one-of>
          <item>[year]</item>
          <item><attrref uri="academic#Year" op="gt" name="year"/></item>
        </one-of>
      </item>
    </one-of>
    <tag>out = year;</tag>
  </rule>
</grammar>
```
