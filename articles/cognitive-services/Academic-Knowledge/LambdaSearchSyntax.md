---
title: 學術知識 API 中的 Lambda 搜尋語法 | Microsoft Docs
description: 了解您可以在 Microsoft 認知服務的學術知識 API 中使用的 Lambda 搜尋語法。
services: cognitive-services
author: alch-msft
manager: kuansanw
ms.service: cognitive-services
ms.component: academic-knowledge
ms.topic: article
ms.date: 03/23/2017
ms.author: alch
ms.openlocfilehash: f486368e1d0258087091acb846a84b125712db40
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35367907"
---
# <a name="lambda-search-syntax"></a>Lambda 搜尋語法

每個 lambda 搜尋查詢字串都會描述一個圖表模式。 查詢必須至少有一個起始節點，並指定我們開始周遊的圖表節點。 若要指定起始節點，請呼叫 *MAG.StartFrom()* 方法並傳入一或多個節點的識別碼，或傳入可指定搜尋條件約束的查詢物件。 *StartFrom()* 方法有三項多載。 全都採用兩個引數，第二個為選擇性引數。 第一個引數可以是長整數、可列舉的長整數集合，或代表 JSON 物件的字串，其語意與在 *json* 搜尋中相同：
```
StartFrom(long cellid, IEnumerable<string> select = null)
StartFrom(IEnumerable<long> cellid, IEnumerable<string> select = null)
StartFrom(string queryObject, IEnumerable<string> select = null)
```

撰寫 lambda 搜尋查詢的程序是從一個節點走到另一個節點。 若要指定要通過的邊緣類型，請使用 *FollowEdge()* 並傳入想要的邊緣類型。 *FollowEdge()* 可採用任意數目的字串引數：
```
FollowEdge(params string[] edgeTypes)
```
> [!NOTE]
> 如果我們不在意要遵循的邊緣類型，只要在兩個節點之間略過 *FollowEdge()* 即可：查詢將通過這兩個節點之間的所有可能邊緣。

我們可以透過 *VisitNode()* 指定要在節點上採取的周遊動作，也就是，是否要停在這個節點並傳回目前路徑作為結果，或繼續探索圖表。  列舉類型 *Action* 可定義兩種動作：*Action.Return* 和 *Action.Continue*。 我們可以將這類列舉值直接傳入 *VisitNode()*，或使用位元 and 運算子 '&' 加以結合。 結合兩個動作時，這表示將會採取這兩個動作。 注意：請勿對動作使用位元 or 運算子 '|'。 這麼做會導致查詢終止，但未傳回任何項目。 略過兩個 *FollowEdge()* 呼叫之間的 *VisitNode()* 會導致查詢在抵達節點之後，無限制地探索圖表。

```
VisitNode(Action action, IEnumerable<string> select = null)
```

對於 *VisitNode()*，我們也可以傳入 *Expression\<Func\<INode, Action\>\>* 類型的 lambda 運算式，其可採用 *INode* 並傳回周遊動作：

```
VisitNode(Expression<Func<INode, Action>> action, IEnumerable<string> select = null)
```

## <a name="inode"></a>*INode* 

*INode* 會提供「唯讀」資料存取介面和節點上的一些內建協助程式函式。 

### <a name="basic-data-access-interfaces"></a>基本資料存取介面

##### <a name="long-cellid"></a>長 CellID

節點的 64 位元識別碼。 

##### <a name="t-getfieldtstring-fieldname"></a>T GetField\<T\>(string fieldName)

取得所指定屬性的值。 *T* 是欄位應該解譯成的所需類型。 如果無法從欄位的類型隱含地轉換所需類型，將會嘗試自動類型轉換。 注意：如果屬性有多重值，則 *GetField\<string\>* 會導致清單序列化成為 Json 字串 ["val1", "val2", ...]。如果屬性不存在，則會擲回例外狀況並中止目前的圖表探索瀏覽。

##### <a name="bool-containsfieldstring-fieldname"></a>bool ContainsField(string fieldName)

告知目前節點中是否存在指定名稱的欄位。

##### <a name="string-getstring-fieldname"></a>string get(string fieldName)

運作方式類似 *GetField\<string\>(fieldName)*。 不過，它不會在找不到欄位時擲回例外狀況，而會改為傳回空的字串 ("")。

##### <a name="bool-hasstring-fieldname"></a>bool has(string fieldName)

告知目前節點中是否存在指定的屬性。 與 *ContainsField(fieldName)* 相同。

##### <a name="bool-hasstring-fieldname-string-value"></a>bool has(string fieldName, string value)

告知屬性是否具有指定的值。 

##### <a name="int-countstring-fieldname"></a>int count(string fieldname)

取得指定屬性的值計數。 如果屬性不存在，則傳回 0。

### <a name="built-in-helper-functions"></a>內建協助程式函式

##### <a name="action-returnifbool-condition"></a>Action return_if(bool condition)

如果條件為 *true*，則傳回 *Action.Return*。 如果條件為 *false* 而且此運算式並未經由位元 and 運算子與其他動作聯結，則會中止圖表探索。

##### <a name="action-continueifbool-condition"></a>Action continue_if(bool condition)

如果條件為 *true*，則傳回 *Action.Continue*。 如果條件為 *false* 而且此運算式並未經由位元 and 運算子與其他動作聯結，則會中止圖表探索。

##### <a name="bool-dicedouble-p"></a>bool dice(double p)

產生大於或等於 0.0 且小於 1.0 的亂數。 如果此數字小於或等於 *p*，此函式會傳回 *true*。

相較於 *json* 搜尋，*lambda* 搜尋更具運算能力：C# lambda 運算式可以直接用來指定查詢模式。 以下有兩個範例。

```
MAG.StartFrom(@"{
    type  : ""ConferenceSeries"",
    match : {
        FullName : ""graph""
    }
}", new List<string>{ "FullName", "ShortName" })
.FollowEdge("ConferenceInstanceIDs")
.VisitNode(v => v.return_if(v.GetField<DateTime>("StartDate").ToString().Contains("2014")),
        new List<string>{ "FullName", "StartDate" })
```

```
MAG.StartFrom(@"{
    type  : ""Affiliation"",
    match : {
        Name : ""microsoft""
    }
}").FollowEdge("PaperIDs")
.VisitNode(v => v.return_if(v.get("NormalizedTitle").Contains("graph") || v.GetField<int>("CitationCount") > 100),
        new List<string>{ "OriginalTitle", "CitationCount" })
```
