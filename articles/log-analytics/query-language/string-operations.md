---
title: 在 Azure Log Analytics 查詢中處理字串 | Microsoft Docs
description: 本文提供教學課程來說明如何使用 Analytics 入口網站在 Log Analytics 中撰寫查詢。
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/16/2018
ms.author: bwren
ms.component: na
ms.openlocfilehash: 6ac697fa12b56840e5dc361500f81e2b7e2ce11a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46950245"
---
# <a name="working-with-strings-in-log-analytics-queries"></a>在 Log Analytics 查詢中處理字串


> [!NOTE]
> 您應該先完成[開始使用 Analytics 入口網站](get-started-analytics-portal.md)與[開始使用查詢](get-started-queries.md)，再完成此教學課程。

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

此文章說明如何編輯、比較、搜尋字串，以及如何在字串上執行各種其他作業。 

字串中的每個字元都有索引編號 (根據其位置)。 第一個字元位於索引 0、第二個字元是 1，依此類推。 不同的字串函式都會使用索引編號，如以下各節所示。 在下列範例中，許多範例都使用 **print** 命令來示範字串，而未使用特定資料來源。


## <a name="strings-and-escaping-them"></a>字串與字串逸出
字串值是被單引號字元或雙引號字元括住。 反斜線 (\) 是用來逸出其後的字元，例如 \t 代表 tab、\n 代表新行，而 \" 代表引號字元本身。

```Kusto
print "this is a 'string' literal in double \" quotes"
```

若要防止 "\\" 被視為逸出字元，請將 \"\@\" 新增為字串的前置詞：

```Kusto
print @"C:\backslash\not\escaped\with @ prefix"
```


## <a name="string-comparisons"></a>字串比較

運算子       |說明                         |區分大小寫|範例 (結果為 `true`)
---------------|------------------------------------|--------------|-----------------------
`==`           |Equals                              |是           |`"aBc" == "aBc"`
`!=`           |Not Equals                          |是           |`"abc" != "ABC"`
`=~`           |Equals                              |否            |`"abc" =~ "ABC"`
`!~`           |不等於                          |否            |`"aBc" !~ "xyz"`
`has`          |右側是左側中的完整詞彙 |否|`"North America" has "america"`
`!has`         |右側不是左側中的完整詞彙       |否            |`"North America" !has "amer"` 
`has_cs`       |右側是左側中的完整詞彙 |是|`"North America" has_cs "America"`
`!has_cs`      |右側不是左側中的完整詞彙       |是            |`"North America" !has_cs "amer"` 
`hasprefix`    |右側是左側中的詞彙前置詞         |否            |`"North America" hasprefix "ame"`
`!hasprefix`   |右側不是左側中的詞彙前置詞     |否            |`"North America" !hasprefix "mer"` 
`hasprefix_cs`    |右側是左側中的詞彙前置詞         |是            |`"North America" hasprefix_cs "Ame"`
`!hasprefix_cs`   |右側不是左側中的詞彙前置詞     |是            |`"North America" !hasprefix_cs "CA"` 
`hassuffix`    |右側是左側中的詞彙後置詞         |否            |`"North America" hassuffix "ica"`
`!hassuffix`   |右側不是左側中的詞彙後置詞     |否            |`"North America" !hassuffix "americ"
`hassuffix_cs`    |右側是左側中的詞彙後置詞         |是            |`"North America" hassuffix_cs "ica"`
`!hassuffix_cs`   |右側不是左側中的詞彙後置詞     |是            |`"North America" !hassuffix_cs "icA"
`contains`     |右側是左側的子項目  |否            |`"FabriKam" contains "BRik"`
`!contains`    |u.4hk4右側未出現在左側中           |否            |`"Fabrikam" !contains "xyz"`
`contains_cs`   |右側是左側的子項目  |是           |`"FabriKam" contains_cs "Kam"`
`!contains_cs`  |u.4hk4右側未出現在左側中           |是           |`"Fabrikam" !contains_cs "Kam"`
`startswith`   |右側是左側的初始項目|否            |`"Fabrikam" startswith "fab"`
`!startswith`  |右側不是左側的初始項目|否        |`"Fabrikam" !startswith "kam"`
`startswith_cs`   |右側是左側的初始項目|是            |`"Fabrikam" startswith_cs "Fab"`
`!startswith_cs`  |右側不是左側的初始項目|是        |`"Fabrikam" !startswith_cs "fab"`
`endswith`     |右側是左側的結尾項目|否             |`"Fabrikam" endswith "Kam"`
`!endswith`    |右側不是左側的結尾項目|否         |`"Fabrikam" !endswith "brik"`
`endswith_cs`     |右側是左側的結尾項目|是             |`"Fabrikam" endswith "Kam"`
`!endswith_cs`    |右側不是左側的結尾項目|是         |`"Fabrikam" !endswith "brik"`
`matches regex`|左側包含右側的相符項        |是           |`"Fabrikam" matches regex "b.*k"`
`in`           |等於其中一個元素       |是           |`"abc" in ("123", "345", "abc")`
`!in`          |不等於任何元素   |是           |`"bca" !in ("123", "345", "abc")`


## <a name="countof"></a>countof

計算子字串在字串中的出現次數。 可以比對純文字或使用規則運算式。 純文字字串比對可能會重疊，而規則運算式比對則不會。

### <a name="syntax"></a>語法
```
countof(text, search [, kind])
```

### <a name="arguments"></a>引數：
- `text` - 輸入字串 
- `search` - 要比對內部文字的純文字或規則運算式。
- `kind` - _normal_ | _regex_ (預設值：normal)。

### <a name="returns"></a>傳回

搜尋字串可在容器中相符的次數。 純文字字串比對可能會重疊，而規則運算式比對則不會。

### <a name="examples"></a>範例

#### <a name="plain-string-matches"></a>純文字比對

```Kusto
print countof("The cat sat on the mat", "at");  //result: 3
print countof("aaa", "a");  //result: 3
print countof("aaaa", "aa");  //result: 3 (not 2!)
print countof("ababa", "ab", "normal");  //result: 2
print countof("ababa", "aba");  //result: 2
```

#### <a name="regex-matches"></a>規則運算式比對

```Kusto
print countof("The cat sat on the mat", @"\b.at\b", "regex");  //result: 3
print countof("ababa", "aba", "regex");  //result: 1
print countof("abcabc", "a.c", "regex");  // result: 2
```


## <a name="extract"></a>extract

從給定字串取得規則運算式的相符項目。 (選擇性) 也可以將擷取的字串轉換為指定型別。

### <a name="syntax"></a>語法

```Kusto
extract(regex, captureGroup, text [, typeLiteral])
```

### <a name="arguments"></a>引數

- `regex` - 規則運算式。
- `captureGroup` - 指出要擷取之擷取群組的正整數常數。 0 代表整個比對、1 代表規則運算式中第一個 '('括號')' 所比對的值，2 或以上的數字代表後續的括號。
- `text` - 要搜尋的字串。
- `typeLiteral` - 選擇性的型別常值 (例如 typeof(long))。 如果提供，所擷取的子字串會轉換為此類型。

### <a name="returns"></a>傳回
針對指定擷取群組 captureGroup 比對的子字串，選擇性地轉換為 typeLiteral。
如果沒有相符項目或型別轉換失敗，則傳回 Null。

### <a name="examples"></a>範例

下列範例會從活動訊號記錄擷取 *ComputerIP* 的最後一個八位元：
```Kusto
Heartbeat
| where ComputerIP != "" 
| take 1
| project ComputerIP, last_octet=extract("([0-9]*$)", 1, ComputerIP) 
```

下列範例會擷取最後一個八位元並將它轉換為 *real* 型別 (數字) 並計算下一個 IP 值
```Kusto
Heartbeat
| where ComputerIP != "" 
| take 1
| extend last_octet=extract("([0-9]*$)", 1, ComputerIP, typeof(real)) 
| extend next_ip=(last_octet+1)%255
| project ComputerIP, last_octet, next_ip
```

在下面的範例中，會搜尋字串 *Trace* 以尋找「時間長度」定義。 相符項目會轉換為 *real* 並乘以時間常數 (1 秒) *，這會將「時間長度」轉換為型別時間戳記*。
```Kusto
let Trace="A=12, B=34, Duration=567, ...";
print Duration = extract("Duration=([0-9.]+)", 1, Trace, typeof(real));  //result: 567
print Duration_seconds =  extract("Duration=([0-9.]+)", 1, Trace, typeof(real)) * time(1s);  //result: 00:09:27
```


## <a name="isempty-isnotempty-notempty"></a>isempty、isnotempty、notempty

- 若引數是空字串或 Null，*isempty* 會傳回 true (另請參閱 *isnull*)。
- 若引數不是空字串或 Null，*isnotempty* 會傳回 true (另請參閱 *isnotnull*)。 別名：*notempty*.

### <a name="syntax"></a>語法

```Kusto
isempty(value)
isnotempty(value)
```

### <a name="examples"></a>範例

```Kusto
print isempty("");  // result: true

print isempty("0");  // result: false

print isempty(0);  // result: false

print isempty(5);  // result: false

Heartbeat | where isnotempty(ComputerIP) | take 1  // return 1 Heartbeat record in which ComputerIP isn't empty
```


## <a name="parseurl"></a>parseurl

將 URL 分割為多個部分 (通訊協定、主機、連接埠等)，並傳回包含字串部分的字典物件。

### <a name="syntax"></a>語法

```
parseurl(urlstring)
```

### <a name="examples"></a>範例

```Kusto
print parseurl("http://user:pass@contoso.com/icecream/buy.aspx?a=1&b=2#tag")
```

結果將會是：
```
{
    "Scheme" : "http",
    "Host" : "contoso.com",
    "Port" : "80",
    "Path" : "/icecream/buy.aspx",
    "Username" : "user",
    "Password" : "pass",
    "Query Parameters" : {"a":"1","b":"2"},
    "Fragment" : "tag"
}
```


## <a name="replace"></a>取代

用另一個字串取代所有規則運算式相符項目。 

### <a name="syntax"></a>語法

```
replace(regex, rewrite, input_text)
```

### <a name="arguments"></a>引數

- `regex` - 要據以比對的規則運算式。 它可以在 '('括號')' 中包含擷取群組。
- `rewrite` - 針對由進行比對之規則運算式所進行之比對的取代規則運算式。 使用 \0 來代表整個相符項目、\1 來代表第一個擷取群組，\2 和以上的數字來代表後續的擷取群組。
- `input_text` - 要在其中搜尋的輸入字串。

### <a name="returns"></a>傳回
以重寫的評估取代規則運算式之所有相符項目後的文字。 相符項目不會重疊。

### <a name="examples"></a>範例

```Kusto
SecurityEvent
| take 1
| project Activity 
| extend replaced = replace(@"(\d+) -", @"Activity ID \1: ", Activity) 
```

可以具有下列結果：
活動                                        |取代後
------------------------------------------------|----------------------------------------------------------
4663 - 已嘗試存取物件  |活動識別碼 4663：已嘗試存取物件。


## <a name="split"></a>split

根據指定的分隔符號分割給定字串，並傳回結果子字串的陣列。

### <a name="syntax"></a>語法
```
split(source, delimiter [, requestedIndex])
```

### <a name="arguments"></a>引數：

- `source` - 將根據指定的分隔符號分割的字串。
- `delimiter` - 將會用來分割來源字串的分隔符號。
- `requestedIndex` - 選擇性的基底為零的索引。 若已提供，傳回的字串陣列將只保有該項目 (若存在)。


### <a name="examples"></a>範例

```Kusto
print split("aaa_bbb_ccc", "_");    // result: ["aaa","bbb","ccc"]
print split("aa_bb", "_");          // result: ["aa","bb"]
print split("aaa_bbb_ccc", "_", 1); // result: ["bbb"]
print split("", "_");               // result: [""]
print split("a__b", "_");           // result: ["a","","b"]
print split("aabbcc", "bb");        // result: ["aa","cc"]
```

## <a name="strcat"></a>strcat

串連字串引數 (支援 1-16 個引數)。

### <a name="syntax"></a>語法
```
strcat("string1", "string2", "string3")
```

### <a name="examples"></a>範例
```Kusto
print strcat("hello", " ", "world") // result: "hello world"
```


## <a name="strlen"></a>strlen

傳回字串長度。

### <a name="syntax"></a>語法
```
strlen("text_to_evaluate")
```

### <a name="examples"></a>範例
```Kusto
print strlen("hello")   // result: 5
```


## <a name="substring"></a>substring

從指定索引開始擷取給定來源字串中的子字串。 (選擇性) 您可以指定所要求子字串的長度。

### <a name="syntax"></a>語法
```
substring(source, startingIndex [, length])
```

### <a name="arguments"></a>引數：

- `source` - 要從中擷取子字串的來源字串。
- `startingIndex` - 所要求子字串以零為基底的起始字元位置。
- `length` - 可用來指定所要求傳回子字串長度的選擇性參數。

### <a name="examples"></a>範例
```Kusto
print substring("abcdefg", 1, 2);   // result: "bc"
print substring("123456", 1);       // result: "23456"
print substring("123456", 2, 2);    // result: "34"
print substring("ABCD", 0, 2);  // result: "AB"
```


## <a name="tolower-toupper"></a>tolower、toupper

將給定字串轉換為全小寫或全大寫。

### <a name="syntax"></a>語法
```
tolower("value")
toupper("value")
```

### <a name="examples"></a>範例
```Kusto
print tolower("HELLO"); // result: "hello"
print toupper("hello"); // result: "HELLO"
```



## <a name="next-steps"></a>後續步驟
繼續進階教學課程：
* [彙總函式](aggregations.md)
* [進階彙總](advanced-aggregations.md)
* [圖表](charts.md)
* [使用 JSON 與資料結構](json-data-structures.md)
* [進階查詢撰寫](advanced-query-writing.md)
* [聯結 - 跨分析](joins.md)