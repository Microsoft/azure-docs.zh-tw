---
title: Azure 受控應用程式建立 UI 定義函式 | Microsoft Docs
description: 描述建構 Azure 受控應用程式的 UI 定義時要使用的函式
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: managed-applications
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/12/2017
ms.author: tomfitz
ms.openlocfilehash: a01a59a7e8c9757cb41d328cd26a34fa219f9152
ms.sourcegitcommit: 688a394c4901590bbcf5351f9afdf9e8f0c89505
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/18/2018
ms.locfileid: "34304499"
---
# <a name="createuidefinition-functions"></a>CreateUiDefinition 函式
本節包含 CreateUiDefinition 所有支援的函式的簽章。

若要使用函式，請使用方括弧括住宣告。 例如︰

```json
"[function()]"
```

您可以參考字串和其他函式做為函式的參數，但必須以單引號括住字串。 例如︰

```json
"[fn1(fn2(), 'foobar')]"
```

如果適用，您可以使用點運算子參考函式輸出的屬性。 例如︰

```json
"[func().prop1]"
```

## <a name="referencing-functions"></a>參考函式
這些函式可以用來參考屬性的輸出或 CreateUiDefinition 的內容。

### <a name="basics"></a>basics
傳回在 Basics 步驟中定義之元素的輸出值。

下列範例會傳回 Basics 步驟中名為 `foo` 之元素的輸出︰

```json
"[basics('foo')]"
```

### <a name="steps"></a>steps
傳回在指定步驟中定義之元素的輸出值。 若要取得 Basics 步驟中元素的輸出值，請改用 `basics()`。

下列範例會傳回名為 `foo` 之步驟中，名為 `bar` 之元素的輸出︰

```json
"[steps('foo').bar]"
```

### <a name="location"></a>location
傳回在 Basics 步驟或目前內容中選取的位置。

下列範例可能會傳回 `"westus"`：

```json
"[location()]"
```

## <a name="string-functions"></a>字串函數
這些函式只能搭配 JSON 字串使用。

### <a name="concat"></a>concat
串連一或多個字串。

例如，如果 `element1` 的輸出值為 `"bar"`，則此範例會傳回字串 `"foobar!"`：

```json
"[concat('foo', steps('step1').element1), '!']"
```

### <a name="substring"></a>substring
傳回指定字串的子字串。 子字串從指定的索引處開始，而且有指定的長度。

下列範例會傳回 `"ftw"`：

```json
"[substring('azure-ftw!!!1one', 6, 3)]"
```

### <a name="replace"></a>取代
傳回將目前字串中所有指定的字串取代成另一個字串的字串。

下列範例會傳回 `"Everything is awesome!"`：

```json
"[replace('Everything is terrible!', 'terrible', 'awesome')]"
```

### <a name="guid"></a>GUID
產生全域唯一字串 (GUID)。

下列範例可能會傳回 `"c7bc8bdc-7252-4a82-ba53-7c468679a511"`：

```json
"[guid()]"
```

### <a name="tolower"></a>toLower
傳回轉換成小寫的字串。

下列範例會傳回 `"foobar"`：

```json
"[toLower('FOOBAR')]"
```

### <a name="toupper"></a>toUpper
傳回轉換成大寫的字串。

下列範例會傳回 `"FOOBAR"`：

```json
"[toUpper('foobar')]"
```

## <a name="collection-functions"></a>集合函式
這些函式可以搭配集合使用，例如 JSON 字串、陣列和物件。

### <a name="contains"></a>contains
如果字串包含指定的子字串、陣列包含指定的值，或是物件包含指定的索引鍵，則傳回 `true`。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `true`：

```json
"[contains('foobar', 'foo')]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `false`：

```json
"[contains(steps('foo').element1, 4)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```

下列範例會傳回 `true`：

```json
"[contains(steps('foo').element1, 'key1')]"
```

### <a name="length"></a>length
傳回字串中的字元數目、陣列中的值數目或物件中的索引鍵數目。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `6`：

```json
"[length('foobar')]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `3`：

```json
"[length(steps('foo').element1)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```

下列範例會傳回 `2`：

```json
"[length(steps('foo').element1)]"
```

### <a name="empty"></a>empty
如果字串、陣列或物件為 null 或空白，則傳回 `true`。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `true`：

```json
"[empty('')]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `false`：

```json
"[empty(steps('foo').element1)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```

下列範例會傳回 `false`：

```json
"[empty(steps('foo').element1)]"
```

#### <a name="example-4-null-and-undefined"></a>範例 4：null 和未定義
假設 `element1` 為 `null` 或未定義。 下列範例會傳回 `true`：

```json
"[empty(steps('foo').element1)]"
```

### <a name="first"></a>first
傳回指定字串的第一個字元、指定陣列的第一個值，或指定物件的第一個索引鍵和值。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `"f"`：

```json
"[first('foobar')]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `1`：

```json
"[first(steps('foo').element1)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```
下列範例會傳回 `{"key1": "foobar"}`：

```json
"[first(steps('foo').element1)]"
```

### <a name="last"></a>last
傳回指定字串的最後一個字元、指定陣列的最後一個值，或指定物件的最後一個索引鍵和值。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `"r"`：

```json
"[last('foobar')]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `2`：

```json
"[last(steps('foo').element1)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```

下列範例會傳回 `{"key2": "raboof"}`：

```json
"[last(steps('foo').element1)]"
```

### <a name="take"></a>take
傳回從字串開頭指定數目的連續字元、從陣列開頭指定數目的連續值，或從物件開頭指定數目的連續索引鍵和值。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `"foo"`：

```json
"[take('foobar', 3)]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `[1, 2]`：

```json
"[take(steps('foo').element1, 2)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```

下列範例會傳回 `{"key1": "foobar"}`：

```json
"[take(steps('foo').element1, 1)]"
```

### <a name="skip"></a>skip
略過集合中指定數目的元素，然後傳回其餘的元素。

#### <a name="example-1-string"></a>範例 1︰字串
下列範例會傳回 `"bar"`：

```json
"[skip('foobar', 3)]"
```

#### <a name="example-2-array"></a>範例 2︰陣列
假設 `element1` 傳回 `[1, 2, 3]`。 下列範例會傳回 `[3]`：

```json
"[skip(steps('foo').element1, 2)]"
```

#### <a name="example-3-object"></a>範例 3︰物件
假設 `element1` 傳回：

```json
{
  "key1": "foobar",
  "key2": "raboof"
}
```
下列範例會傳回 `{"key2": "raboof"}`：

```json
"[skip(steps('foo').element1, 1)]"
```

## <a name="logical-functions"></a>邏輯函式
這些函式可在條件中使用。 有些函式可能不支援所有 JSON 資料類型。

### <a name="equals"></a>equals
如果這兩個參數有相同的類型和值，則傳回 `true`。 此函式支援所有 JSON 資料類型。

下列範例會傳回 `true`：

```json
"[equals(0, 0)]"
```

下列範例會傳回 `true`：

```json
"[equals('foo', 'foo')]"
```

下列範例會傳回 `false`：

```json
"[equals('abc', ['a', 'b', 'c'])]"
```

### <a name="less"></a>less
如果第一個參數小於第二個參數，則傳回 `true`。 此函式只支援數值和字串類型的參數。

下列範例會傳回 `true`：

```json
"[less(1, 2)]"
```

下列範例會傳回 `false`：

```json
"[less('9', '10')]"
```

### <a name="lessorequals"></a>lessOrEquals
如果第一個參數小於或等於第二個參數，則傳回 `true`。 此函式只支援數值和字串類型的參數。

下列範例會傳回 `true`：

```json
"[lessOrEquals(2, 2)]"
```

### <a name="greater"></a>greater
如果第一個參數大於第二個參數，則傳回 `true`。 此函式只支援數值和字串類型的參數。

下列範例會傳回 `false`：

```json
"[greater(1, 2)]"
```

下列範例會傳回 `true`：

```json
"[greater('9', '10')]"
```

### <a name="greaterorequals"></a>greaterOrEquals
如果第一個參數大於或等於第二個參數，則傳回 `true`。 此函式只支援數值和字串類型的參數。

下列範例會傳回 `true`：

```json
"[greaterOrEquals(2, 2)]"
```

### <a name="and"></a>和
如果所有參數都評估為 `true`，則傳回 `true`。 此函式只支援布林值類型的兩個或多個參數。

下列範例會傳回 `true`：

```json
"[and(equals(0, 0), equals('foo', 'foo'), less(1, 2))]"
```

下列範例會傳回 `false`：

```json
"[and(equals(0, 0), greater(1, 2))]"
```

### <a name="or"></a>或
如果至少一個參數評估為 `true`，則傳回 `true`。 此函式只支援布林值類型的兩個或多個參數。

下列範例會傳回 `true`：

```json
"[or(equals(0, 0), equals('foo', 'foo'), less(1, 2))]"
```

下列範例會傳回 `true`：

```json
"[or(equals(0, 0), greater(1, 2))]"
```

### <a name="not"></a>否
如果參數評估為 `false`，則傳回 `true`。 此函式只支援布林值類型的參數。

下列範例會傳回 `true`：

```json
"[not(false)]"
```

下列範例會傳回 `false`：

```json
"[not(equals(0, 0))]"
```

### <a name="coalesce"></a>coalesce
傳回第一個非 null 參數的值。 此函式支援所有 JSON 資料類型。

假設 `element1` 和 `element2` 為未定義。 下列範例會傳回 `"foobar"`：

```json
"[coalesce(steps('foo').element1, steps('foo').element2, 'foobar')]"
```

## <a name="conversion-functions"></a>轉換函式
這些函式可用來轉換 JSON 資料類型與編碼之間的值。

### <a name="int"></a>int
將參數轉換成整數。 此函式支援數值和字串類型的參數。

下列範例會傳回 `1`：

```json
"[int('1')]"
```

下列範例會傳回 `2`：

```json
"[int(2.9)]"
```

### <a name="float"></a>float
將參數轉換成浮點數。 此函式支援數值和字串類型的參數。

下列範例會傳回 `1.0`：

```json
"[float('1.0')]"
```

下列範例會傳回 `2.9`：

```json
"[float(2.9)]"
```

### <a name="string"></a>字串
將參數轉換成字串。 此函式支援所有 JSON 資料類型的參數。

下列範例會傳回 `"1"`：

```json
"[string(1)]"
```

下列範例會傳回 `"2.9"`：

```json
"[string(2.9)]"
```

下列範例會傳回 `"[1,2,3]"`：

```json
"[string([1,2,3])]"
```

下列範例會傳回 `"{"foo":"bar"}"`：

```json
"[string({\"foo\":\"bar\"})]"
```

### <a name="bool"></a>布林
將參數轉換成布林值。 此函式支援數值、字串和布林值類型的參數。 類似於 JavaScript 中的布林值，`0` 或 `'false'` 以外的任何值都會傳回 `true`。

下列範例會傳回 `true`：

```json
"[bool(1)]"
```

下列範例會傳回 `false`：

```json
"[bool(0)]"
```

下列範例會傳回 `true`：

```json
"[bool(true)]"
```

下列範例會傳回 `true`：

```json
"[bool('true')]"
```

### <a name="parse"></a>parse
將參數轉換成原生類型。 換句話說，此函式與 `string()` 相反。 此函式只支援字串類型的參數。

下列範例會傳回 `1`：

```json
"[parse('1')]"
```

下列範例會傳回 `true`：

```json
"[parse('true')]"
```

下列範例會傳回 `[1,2,3]`：

```json
"[parse('[1,2,3]')]"
```

下列範例會傳回 `{"foo":"bar"}`：

```json
"[parse('{\"foo\":\"bar\"}')]"
```

### <a name="encodebase64"></a>encodeBase64
將參數編碼為 base-64 編碼字串。 此函式只支援字串類型的參數。

下列範例會傳回 `"Zm9vYmFy"`：

```json
"[encodeBase64('foobar')]"
```

### <a name="decodebase64"></a>decodeBase64
將參數從 base-64 編碼字串解碼。 此函式只支援字串類型的參數。

下列範例會傳回 `"foobar"`：

```json
"[decodeBase64('Zm9vYmFy')]"
```

### <a name="encodeuricomponent"></a>encodeUriComponent
將參數編碼為 URL 編碼字串。 此函式只支援字串類型的參數。

下列範例會傳回 `"https%3A%2F%2Fportal.azure.com%2F"`：

```json
"[encodeUriComponent('https://portal.azure.com/')]"
```

### <a name="decodeuricomponent"></a>decodeUriComponent
將參數從 URL 編碼字串解碼。 此函式只支援字串類型的參數。

下列範例會傳回 `"https://portal.azure.com/"`：

```json
"[decodeUriComponent('https%3A%2F%2Fportal.azure.com%2F')]"
```

## <a name="math-functions"></a>數學函式
### <a name="add"></a>add
將兩個數字相加，並傳回結果。

下列範例會傳回 `3`：

```json
"[add(1, 2)]"
```

### <a name="sub"></a>sub
將第一個數字減去第二個數字，並傳回結果。

下列範例會傳回 `1`：

```json
"[sub(3, 2)]"
```

### <a name="mul"></a>mul
將兩個數字相乘，並傳回結果。

下列範例會傳回 `6`：

```json
"[mul(2, 3)]"
```

### <a name="div"></a>div
將第一個數字除以第二個數字，並傳回結果。 結果永遠是整數。

下列範例會傳回 `2`：

```json
"[div(6, 3)]"
```

### <a name="mod"></a>mod
將第一個數字除以第二個數字，並傳回餘數。

下列範例會傳回 `0`：

```json
"[mod(6, 3)]"
```

下列範例會傳回 `2`：

```json
"[mod(6, 4)]"
```

### <a name="min"></a>min
傳回兩個數字中較小的數字。

下列範例會傳回 `1`：

```json
"[min(1, 2)]"
```

### <a name="max"></a>max
傳回兩個數字中較大的數字。

下列範例會傳回 `2`：

```json
"[max(1, 2)]"
```

### <a name="range"></a>range
產生指定範圍內的整數序列。

下列範例會傳回 `[1,2,3]`：

```json
"[range(1, 3)]"
```

### <a name="rand"></a>rand
產生指定範圍內的隨機整數。 此函式不會產生密碼編譯安全的隨機數字。

下列範例可能會傳回 `42`：

```json
"[rand(-100, 100)]"
```

### <a name="floor"></a>floor
傳回小於或等於指定數字的最大整數。

下列範例會傳回 `3`：

```json
"[floor(3.14)]"
```

### <a name="ceil"></a>ceil
傳回大於或等於指定數字的最小整數。

下列範例會傳回 `4`：

```json
"[ceil(3.14)]"
```

## <a name="date-functions"></a>日期函式
### <a name="utcnow"></a>utcNow
傳回本機電腦上目前日期和時間之 ISO 8601 格式的字串。

下列範例可能會傳回 `"1990-12-31T23:59:59.000Z"`：

```json
"[utcNow()]"
```

### <a name="addseconds"></a>addSeconds
將指定的時間戳記加上整數秒數。

下列範例會傳回 `"1991-01-01T00:00:00.000Z"`：

```json
"[addSeconds('1990-12-31T23:59:60Z', 1)]"
```

### <a name="addminutes"></a>addMinutes
將指定的時間戳記加上整數分鐘數。

下列範例會傳回 `"1991-01-01T00:00:59.000Z"`：

```json
"[addMinutes('1990-12-31T23:59:59Z', 1)]"
```

### <a name="addhours"></a>addHours
將指定的時間戳記加上整數小時數。

下列範例會傳回 `"1991-01-01T00:59:59.000Z"`：

```json
"[addHours('1990-12-31T23:59:59Z', 1)]"
```

## <a name="next-steps"></a>後續步驟
* 如需 Azure Resource Manager 的簡介，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。

