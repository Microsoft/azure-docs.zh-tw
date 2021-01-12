---
title: 對應資料流程中的運算式函式
description: 了解對應資料流程中的運算式函式。
author: kromerm
ms.author: makromer
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 01/06/2021
ms.openlocfilehash: d0bebf030a35d5e0cec7e5f9364fddbf090ee1c7
ms.sourcegitcommit: 3af12dc5b0b3833acb5d591d0d5a398c926919c8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2021
ms.locfileid: "98072253"
---
# <a name="data-transformation-expressions-in-mapping-data-flow"></a>對應資料流程中的資料轉換運算式

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

## <a name="expression-functions"></a>運算式函式

在 Data Factory 中，使用對應資料流程功能的運算式語言來設定資料轉換。
___
### <code>abs</code>
<code><b>abs(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
數字的絕對值。  
* ``abs(-20) -> 20``  
* ``abs(10) -> 10``  
___   
### <code>acos</code>
<code><b>acos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算反余弦值。  
* ``acos(1) -> 0.0``  
___
### <code>add</code>
<code><b>add(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
新增一對字串或數字。 將日期新增至天數。 將持續時間新增至時間戳記。 對某個陣列附加類似類型的另一個陣列。 與 + 運算子相同。  
* ``add(10, 20) -> 30``  
* ``10 + 20 -> 30``  
* ``add('ice', 'cream') -> 'icecream'``  
* ``'ice' + 'cream' + ' cone' -> 'icecream cone'``  
* ``add(toDate('2012-12-12'), 3) -> toDate('2012-12-15')``  
* ``toDate('2012-12-12') + 3 -> toDate('2012-12-15')``  
* ``[10, 20] + [30, 40] -> [10, 20, 30, 40]``  
* ``toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') + (days(1) + hours(2) - seconds(10)) -> toTimestamp('2019-02-04 07:19:18.871', 'yyyy-MM-dd HH:mm:ss.SSS')``  
___
### <code>addDays</code>
<code><b>addDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to add&gt;</i> : integral) => datetime</b></code><br/><br/>
在日期或時間戳記中增加天數。 與日期的 + 運算子相同。  
* ``addDays(toDate('2016-08-08'), 1) -> toDate('2016-08-09')``  
___
### <code>addMonths</code>
<code><b>addMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to add&gt;</i> : integral, [<i>&lt;value3&gt;</i> : string]) => datetime</b></code><br/><br/>
在日期或時間戳記中增加月數。 您可以選擇性地傳遞時區。  
* ``addMonths(toDate('2016-08-31'), 1) -> toDate('2016-09-30')``  
* ``addMonths(toTimestamp('2016-09-30 10:10:10'), -1) -> toTimestamp('2016-08-31 10:10:10')``  
___
### <code>and</code>
<code><b>and(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
邏輯 AND 運算子。 與 && 相同。  
* ``and(true, false) -> false``  
* ``true && false -> false``  
___
### <code>asin</code>
<code><b>asin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算反正弦值。  
* ``asin(0) -> 0.0``  
___
### <code>atan</code>
<code><b>atan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算反正切值。  
* ``atan(0) -> 0.0``  
___
### <code>atan2</code>
<code><b>atan2(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
傳回平面的正 X 軸和座標所指定的點之間的角度（以弧度為單位）。  
* ``atan2(0, 0) -> 0.0``  
___
### <code>case</code>
<code><b>case(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, <i>&lt;false_expression&gt;</i> : any, ...) => any</b></code><br/><br/>
根據替代條件套用一個值或另一個值。 如果輸入的數目為偶數，則另一個是最後一個條件的預設值為 Null。  
* ``case(10 + 20 == 30, 'dumbo', 'gumbo') -> 'dumbo'``  
* ``case(10 + 20 == 25, 'bojjus', 'do' < 'go', 'gunchus') -> 'gunchus'``  
* ``isNull(case(10 + 20 == 25, 'bojjus', 'do' > 'go', 'gunchus')) -> true``  
* ``case(10 + 20 == 25, 'bojjus', 'do' > 'go', 'gunchus', 'dumbo') -> 'dumbo'``  
___
### <code>cbrt</code>
<code><b>cbrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算數位的立方根。  
* ``cbrt(8) -> 2.0``  
___
### <code>ceil</code>
<code><b>ceil(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
傳回不小於數位的最小整數。  
* ``ceil(-0.1) -> 0``  
___
### <code>coalesce</code>
<code><b>coalesce(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
從一組輸入傳回第一個非 null 值。 所有輸入的類型都應該相同。  
* ``coalesce(10, 20) -> 10``  
* ``coalesce(toString(null), toString(null), 'dumbo', 'bo', 'go') -> 'dumbo'``  
___
### <code>collect</code>
<code><b>collect(<i>&lt;value1&gt;</i> : any) => array</b></code><br/><br/>
將匯總群組中的所有運算式值收集到陣列中。 在此過程中，可以收集結構並將其轉換成替代結構。 項目數會等於該群組中的資料列數，而且可包含 Null 值。 收集的專案數目應該很小。  
* ``collect(salesPerson)``
* ``collect(firstName + lastName))``
* ``collect(@(name = salesPerson, sales = salesAmount) )``
___
### <code>columnNames</code>
<code><b>columnNames(<i>&lt;value1&gt;</i> : string) => array</b></code><br/><br/>
取得資料流的所有輸出資料行。 您可以傳遞選擇性資料流名稱作為第二個引數。  
* ``columnNames()``
* ``columnNames('DeriveStream')``

___
### <code>columns</code>
<code><b>columns([<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
取得資料流的所有輸出資料行。 您可以傳遞選擇性資料流名稱作為第二個引數。   
* ``columns()``
* ``columns('DeriveStream')``
___
### <code>compare</code>
<code><b>compare(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => integer</b></code><br/><br/>
比較相同類型的兩個值。 如果 value1 < value2，則會傳回負整數，如果 value1 = = value2，則會傳回0，如果 value1 > value2，則會傳回正值。  
* ``(compare(12, 24) < 1) -> true``  
* ``(compare('dumbo', 'dum') > 0) -> true``  
___
### <code>concat</code>
<code><b>concat(<i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
將任意數目的字串串連在一起。 與具有字串的 + 運算子相同。  
* ``concat('dataflow', 'is', 'awesome') -> 'dataflowisawesome'``  
* ``'dataflow' + 'is' + 'awesome' -> 'dataflowisawesome'``  
* ``isNull('sql' + null) -> true``  
___
### <code>concatWS</code>
<code><b>concatWS(<i>&lt;separator&gt;</i> : string, <i>&lt;this&gt;</i> : string, <i>&lt;that&gt;</i> : string, ...) => string</b></code><br/><br/>
以分隔符號將任意數目的字串串連在一起。 第一個參數是分隔符號。  
* ``concatWS(' ', 'dataflow', 'is', 'awesome') -> 'dataflow is awesome'``  
* ``isNull(concatWS(null, 'dataflow', 'is', 'awesome')) -> true``  
* ``concatWS(' is ', 'dataflow', 'awesome') -> 'dataflow is awesome'``  
___
### <code>contains</code>
<code><b>contains(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => boolean</b></code><br/><br/>
如果所提供陣列中的任何元素在所提供的述詞中評估為 true，則傳回 true。 Contains 需要述詞函式中一個元素的參考，做為 #item。  
* ``contains([1, 2, 3, 4], #item == 3) -> true``  
* ``contains([1, 2, 3, 4], #item > 5) -> false``  
___
### <code>cos</code>
<code><b>cos(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算余弦值。  
* ``cos(10) -> -0.8390715290764524``  
___
### <code>cosh</code>
<code><b>cosh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算值的雙曲余弦值。  
* ``cosh(0) -> 1.0``  
___
### <code>crc32</code>
<code><b>crc32(<i>&lt;value1&gt;</i> : any, ...) => long</b></code><br/><br/>
以指定的位元長度為不同基本資料類型的一組資料行計算 CRC32 雜湊，值只能是 0(256)、224、256、384、512。 可以用來計算資料列的指紋。  
* ``crc32(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 3630253689L``  
___
### <code>currentDate</code>
<code><b>currentDate([<i>&lt;value1&gt;</i> : string]) => date</b></code><br/><br/>
在此作業開始執行時取得目前的日期。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 [https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). 
* ``currentDate() == toDate('2250-12-31') -> false``  
* ``currentDate('PST')  == toDate('2250-12-31') -> false``  
* ``currentDate('America/New_York')  == toDate('2250-12-31') -> false``  
___
### <code>currentTimestamp</code>
<code><b>currentTimestamp() => timestamp</b></code><br/><br/>
取得作業開始以本地時區執行時的目前時間戳記。  
* ``currentTimestamp() == toTimestamp('2250-12-31 12:12:12') -> false``  
___
### <code>currentUTC</code>
<code><b>currentUTC([<i>&lt;value1&gt;</i> : string]) => timestamp</b></code><br/><br/>
取得 UTC 格式的目前時間戳記。 如果您想要將目前的時間轉譯為與您的叢集時區不同的時區，您可以用 ' GMT '、' PST '、' UTC '、' 北美洲/開曼 ' 的形式傳遞選擇性的時區。 其預設值為目前的時區。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 [https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html](https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html). 將 UTC 時間轉換成不同的時區使用 `fromUTC()` 。  
* ``currentUTC() == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``currentUTC() != toTimestamp('2050-12-12 19:18:12') -> true``  
* ``fromUTC(currentUTC(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  
___
### <code>dayOfMonth</code>
<code><b>dayOfMonth(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得指定日期的當月日期。  
* ``dayOfMonth(toDate('2018-06-08')) -> 8``  
___
### <code>dayOfWeek</code>
<code><b>dayOfWeek(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得指定日期的星期日期。 1-星期日、2-星期一 ...、7-星期六。  
* ``dayOfWeek(toDate('2018-06-08')) -> 6``  
___
### <code>dayOfYear</code>
<code><b>dayOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得指定日期的年中的日期。  
* ``dayOfYear(toDate('2016-04-09')) -> 100``  
___
### <code>days</code>
<code><b>days(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
天數的持續時間（以毫秒為單位）。  
* ``days(2) -> 172800000L``  
___
### <code>degrees</code>
<code><b>degrees(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
將弧度轉換為度。  
* ``degrees(3.141592653589793) -> 180``  
___
### <code>divide</code>
<code><b>divide(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
將數字配對相除。 與運算子相同 `/` 。  
* ``divide(20, 10) -> 2``  
* ``20 / 10 -> 2``  
___
### <code>endsWith</code>
<code><b>endsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
檢查字串是否以提供的字串結尾。  
* ``endsWith('dumbo', 'mbo') -> true``  
___
### <code>equals</code>
<code><b>equals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「等於」比較運算子。 與 = = 運算子相同。  
* ``equals(12, 24) -> false``  
* ``12 == 24 -> false``  
* ``'bad' == 'bad' -> true``  
* ``isNull('good' == toString(null)) -> true``  
* ``isNull(null == null) -> true``  
___
### <code>equalsIgnoreCase</code>
<code><b>equalsIgnoreCase(<i>&lt;value1&gt;</i> : string, <i>&lt;value2&gt;</i> : string) => boolean</b></code><br/><br/>
忽略大小寫的「等於」比較運算子。 與 <=> 運算子相同。  
* ``'abc'<=>'Abc' -> true``  
* ``equalsIgnoreCase('abc', 'Abc') -> true``  
___
### <code>factorial</code>
<code><b>factorial(<i>&lt;value1&gt;</i> : number) => long</b></code><br/><br/>
計算數位的階乘。  
* ``factorial(5) -> 120``  
___
### <code>false</code>
<code><b>false() => boolean</b></code><br/><br/>
一律傳回 false 值。 `syntax(false())`如果有一個名為 ' false ' 的資料行，請使用函數。  
* ``(10 + 20 > 30) -> false``  
* ``(10 + 20 > 30) -> false()``
___
### <code>floor</code>
<code><b>floor(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
傳回不大於數位的最大整數。  
* ``floor(-0.1) -> -1``  
___
### <code>fromBase64</code>
<code><b>fromBase64(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
以 base64 編碼指定的字串。  
* ``fromBase64('Z3VuY2h1cw==') -> 'gunchus'``  
___
### <code>fromUTC</code>
<code><b>fromUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
從 UTC 轉換成時間戳記。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的形式選擇性傳遞時區。 其預設值為目前的時區。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``fromUTC(currentTimestamp()) == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``fromUTC(currentTimestamp(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  
___
### <code>greater</code>
<code><b>greater(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「大於」比較運算子。 與 > 運算子相同。  
* ``greater(12, 24) -> false``  
* ``('dumbo' > 'dum') -> true``  
* ``(toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS') > toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> true``  
___
### <code>greaterOrEqual</code>
<code><b>greaterOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「大於或等於」比較運算子。 與 >= 運算子相同。  
* ``greaterOrEqual(12, 12) -> true``  
* ``('dumbo' >= 'dum') -> true``  
___
### <code>greatest</code>
<code><b>greatest(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
傳回值清單中的最大值作為輸入 (跳過 Null 值)。 如果所有輸入皆為 null，則傳回 null。  
* ``greatest(10, 30, 15, 20) -> 30``  
* ``greatest(10, toInteger(null), 20) -> 20``  
* ``greatest(toDate('2010-12-12'), toDate('2011-12-12'), toDate('2000-12-12')) -> toDate('2011-12-12')``  
* ``greatest(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS'), toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')) -> toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')``  
___
### <code>hasColumn</code>
<code><b>hasColumn(<i>&lt;column name&gt;</i> : string, [<i>&lt;stream name&gt;</i> : string]) => boolean</b></code><br/><br/>
依名稱檢查資料流中的資料行值。 您可以傳遞選擇性資料流名稱作為第二個引數。 在設計階段已知的資料行名稱，應該只依據其名稱來處理。 不支援計算的輸入，但您可使用參數替換。  
* ``hasColumn('parent')``  
___
### <code>hour</code>
<code><b>hour(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
取得時間戳記的小時值。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``hour(toTimestamp('2009-07-30 12:58:59')) -> 12``  
* ``hour(toTimestamp('2009-07-30 12:58:59'), 'PST') -> 12``  
___
### <code>hours</code>
<code><b>hours(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
小時數的持續時間（以毫秒為單位）。  
* ``hours(2) -> 7200000L``  
___
### <code>iif</code>
<code><b>iif(<i>&lt;condition&gt;</i> : boolean, <i>&lt;true_expression&gt;</i> : any, [<i>&lt;false_expression&gt;</i> : any]) => any</b></code><br/><br/>
根據條件套用一個值或另一個值。 如果未指定其他值，則會將其視為 Null。 這兩個值都必須相容 (數值、字串 ... ) 。 * ``iif(10 + 20 == 30, 'dumbo', 'gumbo') -> 'dumbo'``  
* ``iif(10 > 30, 'dumbo', 'gumbo') -> 'gumbo'``  
* ``iif(month(toDate('2018-12-01')) == 12, 345.12, 102.67) -> 345.12``  
___
### <code>iifNull</code>
<code><b>iifNull(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : any], ...) => any</b></code><br/><br/>
檢查第一個參數是否為 Null。 如果不是 Null，則會傳回第一個參數。 如果是 Null，則會傳回第二個參數。 如果指定了三個參數，則行為會與 iif(isNull(value1), value2, value3) 相同，而如果第一個值不是 null，則會傳回第三個參數。  
* ``iifNull(10, 20) -> 10``  
* ``iifNull(null, 20, 40) -> 20``  
* ``iifNull('azure', 'data', 'factory') -> 'factory'``  
* ``iifNull(null, 'data', 'factory') -> 'data'``  
___
### <code>in</code>
<code><b>in(<i>&lt;array of items&gt;</i> : array, <i>&lt;item to find&gt;</i> : any) => boolean</b></code><br/><br/>
檢查項目是否在陣列中。  
* ``in([10, 20, 30], 10) -> true``  
* ``in(['good', 'kid'], 'bad') -> false``  
___
### <code>initCap</code>
<code><b>initCap(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
將每個單字的第一個字母轉換成大寫。 文字會被識別為以空格分隔。  
* ``initCap('cool iceCREAM') -> 'Cool Icecream'``  
___
### <code>instr</code>
<code><b>instr(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string) => integer</b></code><br/><br/>
尋找子字串在字串中的位置 (從 1 開始)。 如果找不到，則傳回0。  
* ``instr('dumbo', 'mbo') -> 3``  
* ``instr('microsoft', 'o') -> 5``  
* ``instr('good', 'bad') -> 0``  
___
### <code>isDelete</code>
<code><b>isDelete([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否有刪除標示。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isDelete()``  
* ``isDelete(1)``  
___
### <code>isError</code>
<code><b>isError([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否標示為錯誤。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isError()``  
* ``isError(1)``  
___
### <code>isIgnore</code>
<code><b>isIgnore([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否標示為要忽略。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isIgnore()``  
* ``isIgnore(1)``  
___
### <code>isInsert</code>
<code><b>isInsert([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否有插入標示。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isInsert()``  
* ``isInsert(1)``  
___
### <code>isMatch</code>
<code><b>isMatch([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查在查閱時是否比對資料列。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isMatch()``  
* ``isMatch(1)``  
___
### <code>isNull</code>
<code><b>isNull(<i>&lt;value1&gt;</i> : any) => boolean</b></code><br/><br/>
檢查值是否為 Null。  
* ``isNull(NULL()) -> true``  
* ``isNull('') -> false``  
___
### <code>isUpdate</code>
<code><b>isUpdate([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否有更新標示。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isUpdate()``  
* ``isUpdate(1)``  
___
### <code>isUpsert</code>
<code><b>isUpsert([<i>&lt;value1&gt;</i> : integer]) => boolean</b></code><br/><br/>
檢查資料列是否有插入標示。 對於取用多個輸入資料流的轉換，您可傳遞資料流 (從 1 開始) 的索引。 資料流程索引應該是1或2，而預設值是1。  
* ``isUpsert()``  
* ``isUpsert(1)``  
___
### <code>lastDayOfMonth</code>
<code><b>lastDayOfMonth(<i>&lt;value1&gt;</i> : datetime) => date</b></code><br/><br/>
取得指定日期的當月最後一個日期。  
* ``lastDayOfMonth(toDate('2009-01-12')) -> toDate('2009-01-31')``  
___
### <code>least</code>
<code><b>least(<i>&lt;value1&gt;</i> : any, ...) => any</b></code><br/><br/>
「小於或等於」比較運算子。 與 <= 運算子相同。  
* ``least(10, 30, 15, 20) -> 10``  
* ``least(toDate('2010-12-12'), toDate('2011-12-12'), toDate('2000-12-12')) -> toDate('2000-12-12')``  
___
### <code>left</code>
<code><b>left(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
從索引 1 處開始擷取含特定字元數的子字串。 與 SUBSTRING (str，1，n) 相同。  
* ``left('bojjus', 2) -> 'bo'``  
* ``left('bojjus', 20) -> 'bojjus'``  
___
### <code>length</code>
<code><b>length(<i>&lt;value1&gt;</i> : string) => integer</b></code><br/><br/>
傳回字串的長度。  
* ``length('dumbo') -> 5``  
___
### <code>lesser</code>
<code><b>lesser(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「小於」比較運算子。 與 < 運算子相同。  
* ``lesser(12, 24) -> true``  
* ``('abcd' < 'abc') -> false``  
* ``(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') < toTimestamp('2019-02-05 08:21:34.890', 'yyyy-MM-dd HH:mm:ss.SSS')) -> true``  
___
### <code>lesserOrEqual</code>
<code><b>lesserOrEqual(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「小於或等於」比較運算子。 與 <= 運算子相同。  
* ``lesserOrEqual(12, 12) -> true``  
* ``('dumbo' <= 'dum') -> false``  
___
### <code>levenshtein</code>
<code><b>levenshtein(<i>&lt;from string&gt;</i> : string, <i>&lt;to string&gt;</i> : string) => integer</b></code><br/><br/>
取得兩個字串之間的 levenshtein 距離。  
* ``levenshtein('boys', 'girls') -> 4``  
___
### <code>like</code>
<code><b>like(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
此模式是字面上相符的字串。 下列特殊符號則屬例外： _ 符合輸入中的任何一個字元 (類似於 在 ```posix``` 正則運算式) % 符合輸入 (中的零或多個字元，類似于 ```posix``` 正則運算式) 。
逸出字元為 ''。 如果特殊符號或另一個逸出字元前面有逸出字元，就會逐字比對下列字元。 逸出任何其他字元的動作是無效的。  
* ``like('icecream', 'ice%') -> true``  
___
### <code>locate</code>
<code><b>locate(<i>&lt;substring to find&gt;</i> : string, <i>&lt;string&gt;</i> : string, [<i>&lt;from index - 1-based&gt;</i> : integral]) => integer</b></code><br/><br/>
尋找子字串在字串中從特定位置起算的位置 (從 1 開始)。 如果省略位置，則視為字串的開頭。 如果找不到，則傳回0。  
* ``locate('mbo', 'dumbo') -> 3``  
* ``locate('o', 'microsoft', 6) -> 7``  
* ``locate('bad', 'good') -> 0``  
___
### <code>log</code>
<code><b>log(<i>&lt;value1&gt;</i> : number, [<i>&lt;value2&gt;</i> : number]) => double</b></code><br/><br/>
計算對數值。 如果有使用，則可以提供選擇性的基底，否則為歐拉號碼。  
* ``log(100, 10) -> 2``  
___
### <code>log10</code>
<code><b>log10(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
根據10基底計算記錄值。  
* ``log10(100) -> 2``  
___
### <code>lower</code>
<code><b>lower(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Lowercases 字串。  
* ``lower('GunChus') -> 'gunchus'``  
___
### <code>lpad</code>
<code><b>lpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
從左側為字串填補提供的填補內容，直到達到特定長度。 如果字串等於或大於長度，則會修剪為長度。  
* ``lpad('dumbo', 10, '-') -> '-----dumbo'``  
* ``lpad('dumbo', 4, '-') -> 'dumb'``  
* ``lpad('dumbo', 8, '<>') -> '<><dumbo'``  
___
### <code>ltrim</code>
<code><b>ltrim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
從左側修剪開頭字元的字串。 若未指定第二個參數，則會修剪空白字元。 否則，它會修剪第二個參數中指定的任何字元。  
* ``ltrim('  dumbo  ') -> 'dumbo  '``  
* ``ltrim('!--!du!mbo!', '-!') -> 'du!mbo!'``  
___
### <code>md5</code>
<code><b>md5(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
為不同基本資料類型的一組資料行計算 MD5 摘要，並傳回 32 個字元的十六進位字串。 可以用來計算資料列的指紋。  
* ``md5(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> '4ce8a880bd621a1ffad0bca905e1bc5a'``  
___
### <code>millisecond</code>
<code><b>millisecond(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
取得日期的毫秒值。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``millisecond(toTimestamp('2009-07-30 12:58:59.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> 871``  
___
### <code>milliseconds</code>
<code><b>milliseconds(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
毫秒數的持續時間（以毫秒為單位）。  
* ``milliseconds(2) -> 2L``  
___
### <code>minus</code>
<code><b>minus(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
減去數字。 從日期減去天數。 從時間戳記減去持續時間。 減去兩個時間戳記來取得差異 (以毫秒為單位)。 與-運算子相同。  
* ``minus(20, 10) -> 10``  
* ``20 - 10 -> 10``  
* ``minus(toDate('2012-12-15'), 3) -> toDate('2012-12-12')``  
* ``toDate('2012-12-15') - 3 -> toDate('2012-12-12')``  
* ``toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS') + (days(1) + hours(2) - seconds(10)) -> toTimestamp('2019-02-04 07:19:18.871', 'yyyy-MM-dd HH:mm:ss.SSS')``  
* ``toTimestamp('2019-02-03 05:21:34.851', 'yyyy-MM-dd HH:mm:ss.SSS') - toTimestamp('2019-02-03 05:21:36.923', 'yyyy-MM-dd HH:mm:ss.SSS') -> -2072``  
___
### <code>minute</code>
<code><b>minute(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
取得時間戳記的分鐘值。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``minute(toTimestamp('2009-07-30 12:58:59')) -> 58``  
* ``minute(toTimestamp('2009-07-30 12:58:59'), 'PST') -> 58``  
___
### <code>minutes</code>
<code><b>minutes(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
分鐘數的持續時間（以毫秒為單位）。  
* ``minutes(2) -> 120000L``  
___
### <code>mod</code>
<code><b>mod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
數字配對的模數。 與% 運算子相同。  
* ``mod(20, 8) -> 4``  
* ``20 % 8 -> 4``  
___
### <code>month</code>
<code><b>month(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得日期或時間戳記的月份值。  
* ``month(toDate('2012-8-8')) -> 8``  
___
### <code>monthsBetween</code>
<code><b>monthsBetween(<i>&lt;from date/timestamp&gt;</i> : datetime, <i>&lt;to date/timestamp&gt;</i> : datetime, [<i>&lt;roundoff&gt;</i> : boolean], [<i>&lt;time zone&gt;</i> : string]) => double</b></code><br/><br/>
取得兩個日期之間的月數。 您可以將計算四捨五入。您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``monthsBetween(toTimestamp('1997-02-28 10:30:00'), toDate('1996-10-30')) -> 3.94959677``  
___
### <code>multiply</code>
<code><b>multiply(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
將數字配對相乘。 與 * 運算子相同。  
* ``multiply(20, 10) -> 200``  
* ``20 * 10 -> 200``  
___
### <code>negate</code>
<code><b>negate(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
將數字轉換為負數。 將正數變成負數，反之亦然。  
* ``negate(13) -> -13``  
___
### <code>nextSequence</code>
<code><b>nextSequence() => long</b></code><br/><br/>
傳回下一個唯一序列。 此數位只會在資料分割內是連續的，而且前面會加上 partitionId。  
* ``nextSequence() == 12313112 -> false``  
___
### <code>normalize</code>
<code><b>normalize(<i>&lt;String to normalize&gt;</i> : string) => string</b></code><br/><br/>
標準化字串值以分隔重音 unicode 字元。  
* ``regexReplace(normalize('bo²s'), `\p{M}`, '') -> 'boys'``  
___
### <code>not</code>
<code><b>not(<i>&lt;value1&gt;</i> : boolean) => boolean</b></code><br/><br/>
邏輯負運算子。  
* ``not(true) -> false``  
* ``not(10 == 20) -> true``  
___
### <code>notEquals</code>
<code><b>notEquals(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => boolean</b></code><br/><br/>
「不等於」比較運算子。 與！ = 運算子相同。  
* ``12 != 24 -> true``  
* ``'bojjus' != 'bo' + 'jjus' -> false``  
___
### <code>notNull</code>
<code><b>notNull(<i>&lt;value1&gt;</i> : any) => boolean</b></code><br/><br/>
檢查值是否不是 Null。  
* ``notNull(NULL()) -> false``  
* ``notNull('') -> true``  
___
### <code>null</code>
<code><b>null() => null</b></code><br/><br/>
傳回 NULL 值。 `syntax(null())`如果有名為 ' null ' 的資料行，請使用函數。 使用的任何作業都會產生 Null。  
* ``isNull('dumbo' + null) -> true``  
* ``isNull(10 * null) -> true``  
* ``isNull('') -> false``  
* ``isNull(10 + 20) -> false``  
* ``isNull(10/0) -> true``  
___
### <code>or</code>
<code><b>or(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
邏輯 OR 運算子。 與 | | 相同。  
* ``or(true, false) -> true``  
* ``true || false -> true``  
___
### <code>pMod</code>
<code><b>pMod(<i>&lt;value1&gt;</i> : any, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
數字配對的正模數。  
* ``pmod(-20, 8) -> 4``  
___
### <code>partitionId</code>
<code><b>partitionId() => integer</b></code><br/><br/>
傳回輸入資料列所在的目前資料分割識別碼。  
* ``partitionId()``  
___
### <code>power</code>
<code><b>power(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
將一個數位引發至另一個數位。  
* ``power(10, 2) -> 100``  
___
### <code>random</code>
<code><b>random(<i>&lt;value1&gt;</i> : integral) => long</b></code><br/><br/>
傳回指定資料分割內選擇性種子的亂數字。 種子必須是固定值，而且會與 partitionId 搭配使用，以產生隨機值 * ``random(1) == 1 -> false``
___
### <code>regexExtract</code>
<code><b>regexExtract(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, [<i>&lt;match group 1-based index&gt;</i> : integral]) => string</b></code><br/><br/>
擷取指定規則運算式模式的相符子字串。 最後一個參數會識別比對群組，如果省略則預設為 1。 使用 ' <regex> ' (後引號) 來比對字串，而不進行任何轉義。  
* ``regexExtract('Cost is between 600 and 800 dollars', '(\\d+) and (\\d+)', 2) -> '800'``  
* ``regexExtract('Cost is between 600 and 800 dollars', `(\d+) and (\d+)`, 2) -> '800'``  
___
### <code>regexMatch</code>
<code><b>regexMatch(<i>&lt;string&gt;</i> : string, <i>&lt;regex to match&gt;</i> : string) => boolean</b></code><br/><br/>
檢查字串是否符合指定的規則運算式模式。 使用 ' <regex> ' (後引號) 來比對字串，而不進行任何轉義。  
* ``regexMatch('200.50', '(\\d+).(\\d+)') -> true``  
* ``regexMatch('200.50', `(\d+).(\d+)`) -> true``  
___
### <code>regexReplace</code>
<code><b>regexReplace(<i>&lt;string&gt;</i> : string, <i>&lt;regex to find&gt;</i> : string, <i>&lt;substring to replace&gt;</i> : string) => string</b></code><br/><br/>
以指定字串中的另一個子字串取代所有出現的 RegEx 模式，使用 ' <regex> ' (後引號) 來比對字串，而不進行轉義。  
* ``regexReplace('100 and 200', '(\\d+)', 'bojjus') -> 'bojjus and bojjus'``  
* ``regexReplace('100 and 200', `(\d+)`, 'gunchus') -> 'gunchus and gunchus'``  
___
### <code>regexSplit</code>
<code><b>regexSplit(<i>&lt;string to split&gt;</i> : string, <i>&lt;regex expression&gt;</i> : string) => array</b></code><br/><br/>
根據以 RegEx 為基礎的分隔符號來分割字串，並傳回字串陣列。  
* ``regexSplit('bojjusAgunchusBdumbo', `[CAB]`) -> ['bojjus', 'gunchus', 'dumbo']``  
* ``regexSplit('bojjusAgunchusBdumboC', `[CAB]`) -> ['bojjus', 'gunchus', 'dumbo', '']``  
* ``(regexSplit('bojjusAgunchusBdumboC', `[CAB]`)[1]) -> 'bojjus'``  
* ``isNull(regexSplit('bojjusAgunchusBdumboC', `[CAB]`)[20]) -> true``  
___
### <code>replace</code>
<code><b>replace(<i>&lt;string&gt;</i> : string, <i>&lt;substring to find&gt;</i> : string, [<i>&lt;substring to replace&gt;</i> : string]) => string</b></code><br/><br/>
將指定字串中出現的某個子字串全部取代為另一個子字串。 如果省略最後一個參數，則預設為空字串。  
* ``replace('doggie dog', 'dog', 'cat') -> 'catgie cat'``  
* ``replace('doggie dog', 'dog', '') -> 'gie '``  
* ``replace('doggie dog', 'dog') -> 'gie '``  
___
### <code>reverse</code>
<code><b>reverse(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
反轉字串。  
* ``reverse('gunchus') -> 'suhcnug'``  
___
### <code>right</code>
<code><b>right(<i>&lt;string to subset&gt;</i> : string, <i>&lt;number of characters&gt;</i> : integral) => string</b></code><br/><br/>
從右側擷取含特定字元數的子字串。 與 SUBSTRING (str 相同，長度 (str) -n，n) 。  
* ``right('bojjus', 2) -> 'us'``  
* ``right('bojjus', 20) -> 'bojjus'``  
___
### <code>rlike</code>
<code><b>rlike(<i>&lt;string&gt;</i> : string, <i>&lt;pattern match&gt;</i> : string) => boolean</b></code><br/><br/>
檢查字串是否符合指定的規則運算式模式。  
* ``rlike('200.50', `(\d+).(\d+)`) -> true``  
* ``rlike('bogus', `M[0-9]+.*`) -> false``  
___
### <code>round</code>
<code><b>round(<i>&lt;number&gt;</i> : number, [<i>&lt;scale to round&gt;</i> : number], [<i>&lt;rounding option&gt;</i> : integral]) => double</b></code><br/><br/>
以指定的選擇性小數位數和選擇性的捨入模式將數字捨入進位。 如果省略小數位數，則預設為 0。 如果省略捨入模式，則預設為 ROUND_HALF_UP(5)。 舍入的值包括 1-ROUND_UP 2-ROUND_DOWN 3-ROUND_CEILING 4-ROUND_FLOOR 5-ROUND_HALF_UP 6-ROUND_HALF_DOWN 7-ROUND_HALF_EVEN 8 ROUND_UNNECESSARY。  
* ``round(100.123) -> 100.0``  
* ``round(2.5, 0) -> 3.0``  
* ``round(5.3999999999999995, 2, 7) -> 5.40``  
___
### <code>rpad</code>
<code><b>rpad(<i>&lt;string to pad&gt;</i> : string, <i>&lt;final padded length&gt;</i> : integral, <i>&lt;padding&gt;</i> : string) => string</b></code><br/><br/>
從右側為字串填補提供的填補內容，直到達到特定長度。 如果字串等於或大於長度，則會修剪為長度。  
* ``rpad('dumbo', 10, '-') -> 'dumbo-----'``  
* ``rpad('dumbo', 4, '-') -> 'dumb'``  
* ``rpad('dumbo', 8, '<>') -> 'dumbo<><'``  
___
### <code>rtrim</code>rtrim</code>
<code><b>rtrim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
以滑鼠右鍵修剪尾端字元的字串。 若未指定第二個參數，則會修剪空白字元。 否則，它會修剪第二個參數中指定的任何字元。  
* ``rtrim('  dumbo  ') -> '  dumbo'``  
* ``rtrim('!--!du!mbo!', '-!') -> '!--!du!mbo'``  
___
### <code>second</code>
<code><b>second(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => integer</b></code><br/><br/>
取得日期的秒值。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 當地時區會作為預設值。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``second(toTimestamp('2009-07-30 12:58:59')) -> 59``  
___
### <code>seconds</code>
<code><b>seconds(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
秒數的持續時間（以毫秒為單位）。  
* ``seconds(2) -> 2000L``  
___
### <code>sha1</code>
<code><b>sha1(<i>&lt;value1&gt;</i> : any, ...) => string</b></code><br/><br/>
為不同基本資料類型的一組資料行計算 SHA-1 摘要，並傳回 40 個字元的十六進位字串。 可以用來計算資料列的指紋。  
* ``sha1(5, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> '46d3b478e8ec4e1f3b453ac3d8e59d5854e282bb'``  
___
### <code>sha2</code>
<code><b>sha2(<i>&lt;value1&gt;</i> : integer, <i>&lt;value2&gt;</i> : any, ...) => string</b></code><br/><br/>
以指定的位元長度為不同基本資料類型的一組資料行計算 SHA-2 摘要，值只能是 0(256)、224、256、384、512。 可以用來計算資料列的指紋。  
* ``sha2(256, 'gunchus', 8.2, 'bojjus', true, toDate('2010-4-4')) -> 'afe8a553b1761c67d76f8c31ceef7f71b66a1ee6f4e6d3b5478bf68b47d06bd3'``  
___
### <code>sin</code>
<code><b>sin(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算正弦值。  
* ``sin(2) -> 0.9092974268256817``  
___
### <code>sinh</code>
<code><b>sinh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算雙曲正弦值。  
* ``sinh(0) -> 0.0``  
___
### <code>soundex</code>
<code><b>soundex(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
取得 ```soundex``` 字串的程式碼。  
* ``soundex('genius') -> 'G520'``  
___
### <code>split</code>
<code><b>split(<i>&lt;string to split&gt;</i> : string, <i>&lt;split characters&gt;</i> : string) => array</b></code><br/><br/>
根據分隔符號分割字串，並傳回字串陣列。  
* ``split('bojjus,guchus,dumbo', ',') -> ['bojjus', 'guchus', 'dumbo']``  
* ``split('bojjus,guchus,dumbo', '|') -> ['bojjus,guchus,dumbo']``  
* ``split('bojjus, guchus, dumbo', ', ') -> ['bojjus', 'guchus', 'dumbo']``  
* ``split('bojjus, guchus, dumbo', ', ')[1] -> 'bojjus'``  
* ``isNull(split('bojjus, guchus, dumbo', ', ')[0]) -> true``  
* ``isNull(split('bojjus, guchus, dumbo', ', ')[20]) -> true``  
* ``split('bojjusguchusdumbo', ',') -> ['bojjusguchusdumbo']``  
___
### <code>sqrt</code>
<code><b>sqrt(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算數位的平方根。  
* ``sqrt(9) -> 3``  
___
### <code>startsWith</code>
<code><b>startsWith(<i>&lt;string&gt;</i> : string, <i>&lt;substring to check&gt;</i> : string) => boolean</b></code><br/><br/>
檢查字串是否以提供的字串開頭。  
* ``startsWith('dumbo', 'du') -> true``  
___
### <code>subDays</code>
<code><b>subDays(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;days to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
從日期或時間戳記中減去天數。 與日期的-運算子相同。  
* ``subDays(toDate('2016-08-08'), 1) -> toDate('2016-08-07')``  
___
### <code>subMonths</code>
<code><b>subMonths(<i>&lt;date/timestamp&gt;</i> : datetime, <i>&lt;months to subtract&gt;</i> : integral) => datetime</b></code><br/><br/>
從日期或時間戳記中減去月數。  
* ``subMonths(toDate('2016-09-30'), 1) -> toDate('2016-08-31')``  
___
### <code>substring</code>
<code><b>substring(<i>&lt;string to subset&gt;</i> : string, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of characters&gt;</i> : integral]) => string</b></code><br/><br/>
從某個位置擷取特定長度的子字串。 位置從 1 開始。 如果省略長度，則預設為字串的結尾。  
* ``substring('Cat in the hat', 5, 2) -> 'in'``  
* ``substring('Cat in the hat', 5, 100) -> 'in the hat'``  
* ``substring('Cat in the hat', 5) -> 'in the hat'``  
* ``substring('Cat in the hat', 100, 100) -> ''``  
___
### <code>tan</code>
<code><b>tan(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算正切值。  
* ``tan(0) -> 0.0``  
___
### <code>tanh</code>
<code><b>tanh(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
計算雙曲正切值。  
* ``tanh(0) -> 0.0``  
___
### <code>translate</code>
<code><b>translate(<i>&lt;string to translate&gt;</i> : string, <i>&lt;lookup characters&gt;</i> : string, <i>&lt;replace characters&gt;</i> : string) => string</b></code><br/><br/>
將字串中的一組字元取代為另一組字元。 字元有1到1的取代。  
* ``translate('(bojjus)', '()', '[]') -> '[bojjus]'``  
* ``translate('(gunchus)', '()', '[') -> '[gunchus'``  
___
### <code>trim</code>
<code><b>trim(<i>&lt;string to trim&gt;</i> : string, [<i>&lt;trim characters&gt;</i> : string]) => string</b></code><br/><br/>
修剪開頭和結尾字元的字串。 若未指定第二個參數，則會修剪空白字元。 否則，它會修剪第二個參數中指定的任何字元。  
* ``trim('  dumbo  ') -> 'dumbo'``  
* ``trim('!--!du!mbo!', '-!') -> 'du!mbo'``  
___
### <code>true</code>
<code><b>true() => boolean</b></code><br/><br/>
一律傳回 true 值。 `syntax(true())`如果有一個名為 ' true ' 的資料行，請使用函數。  
* ``(10 + 20 == 30) -> true``  
* ``(10 + 20 == 30) -> true()``  
___
### <code>typeMatch</code>
<code><b>typeMatch(<i>&lt;type&gt;</i> : string, <i>&lt;base type&gt;</i> : string) => boolean</b></code><br/><br/>
比對資料行的類型。 只能在模式運算式中使用。數位符合 short、integer、long、double、float 或 decimal、整數相符的 short、integer、long、decimal 符合 double、float、decimal 和 datetime 與 date 或 timestamp 類型。  
* ``typeMatch(type, 'number')``  
* ``typeMatch('date', 'datetime')``  
___
### <code>upper</code>
<code><b>upper(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
Uppercases 字串。  
* ``upper('bojjus') -> 'BOJJUS'``  
___
### <code>uuid</code>
<code><b>uuid() => string</b></code><br/><br/>
傳回產生的 UUID。  
* ``uuid()``  
___
### <code>weekOfYear</code>
<code><b>weekOfYear(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得指定日期的年中的周。  
* ``weekOfYear(toDate('2008-02-20')) -> 8``  
___
### <code>weeks</code>
<code><b>weeks(<i>&lt;value1&gt;</i> : integer) => long</b></code><br/><br/>
周數的持續時間（以毫秒為單位）。  
* ``weeks(2) -> 1209600000L``  
___
### <code>xor</code>
<code><b>xor(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : boolean) => boolean</b></code><br/><br/>
邏輯 XOR 運算子。 與 ^ 運算子相同。  
* ``xor(true, false) -> true``  
* ``xor(true, true) -> false``  
* ``true ^ false -> true``  
___
### <code>year</code>
<code><b>year(<i>&lt;value1&gt;</i> : datetime) => integer</b></code><br/><br/>
取得日期的年份值。  
* ``year(toDate('2012-8-8')) -> 2012``  

## <a name="aggregate-functions"></a>彙總函式
下列函數僅適用于 aggregate、pivot、unpivot 和 window 轉換。
___
### <code>avg</code>
<code><b>avg(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
取得資料行值的平均值。  
* ``avg(sales)``  
___
### <code>avgIf</code>
<code><b>avgIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根據條件取得資料行值的平均值。  
* ``avgIf(region == 'West', sales)``  
___
### <code>count</code>
<code><b>count([<i>&lt;value1&gt;</i> : any]) => long</b></code><br/><br/>
取得值的彙總計數。 如果指定了選擇性資料行 (s) ，它會忽略計數中的 Null 值。  
* ``count(custId)``  
* ``count(custId, custName)``  
* ``count()``  
* ``count(iif(isNull(custId), 1, NULL))``  
___
### <code>countDistinct</code>
<code><b>countDistinct(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : any], ...) => long</b></code><br/><br/>
取得一組資料行之相異值的匯總計數。  
* ``countDistinct(custId, custName)``  
___
### <code>countIf</code>
<code><b>countIf(<i>&lt;value1&gt;</i> : boolean, [<i>&lt;value2&gt;</i> : any]) => long</b></code><br/><br/>
根據條件取得值的彙總計數。 如果指定了選擇性資料行，則會忽略計數中的 Null 值。  
* ``countIf(state == 'CA' && commission < 10000, name)``  
___
### <code>covariancePopulation</code>
<code><b>covariancePopulation(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
取得兩個數據行之間的擴展共變數。  
* ``covariancePopulation(sales, profit)``  
___
### <code>covariancePopulationIf</code>
<code><b>covariancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
根據條件取得兩個數據行的人口共變數。  
* ``covariancePopulationIf(region == 'West', sales)``  
___
### <code>covarianceSample</code>
<code><b>covarianceSample(<i>&lt;value1&gt;</i> : number, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
取得兩個數據行的樣本共變數。  
* ``covarianceSample(sales, profit)``  
___
### <code>covarianceSampleIf</code>
<code><b>covarianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number, <i>&lt;value3&gt;</i> : number) => double</b></code><br/><br/>
根據條件取得兩個數據行的樣本共變數。  
* ``covarianceSampleIf(region == 'West', sales, profit)``  
___
### <code>first</code>
<code><b>first(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
取得資料行群組的第一個值。 如果省略第二個參數 ignoreNulls，則會假設為 false。  
* ``first(sales)``  
* ``first(sales, false)``  
___
### <code>kurtosis</code>
<code><b>kurtosis(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的峰度。  
* ``kurtosis(sales)``  
___
### <code>kurtosisIf</code>
<code><b>kurtosisIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的峰度。  
* ``kurtosisIf(region == 'West', sales)``  
___
### <code>last</code>
<code><b>last(<i>&lt;value1&gt;</i> : any, [<i>&lt;value2&gt;</i> : boolean]) => any</b></code><br/><br/>
取得資料行群組的最後一個值。 如果省略第二個參數 ignoreNulls，則會假設為 false。  
* ``last(sales)``  
* ``last(sales, false)``  
___
### <code>max</code>
<code><b>max(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
取得資料行的最大值。  
* ``max(sales)``  
___
### <code>maxIf</code>
<code><b>maxIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
根據準則，取得資料行的最大值。  
* ``maxIf(region == 'West', sales)``  
___
### <code>mean</code>
<code><b>mean(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
取得資料行的平均值。 與 AVG。  
* ``mean(sales)``  
___
### <code>meanIf</code>
<code><b>meanIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根據條件取得資料行的平均值。 與 avgIf 相同。  
* ``meanIf(region == 'West', sales)``  
___
### <code>min</code>
<code><b>min(<i>&lt;value1&gt;</i> : any) => any</b></code><br/><br/>
取得資料行的最小值。  
* ``min(sales)``  
___
### <code>minIf</code>
<code><b>minIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : any) => any</b></code><br/><br/>
根據準則，取得資料行的最小值。  
* ``minIf(region == 'West', sales)``  
___
### <code>skewness</code>
<code><b>skewness(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的不對稱。  
* ``skewness(sales)``  
___
### <code>skewnessIf</code>
<code><b>skewnessIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據條件取得資料行的不對稱。  
* ``skewnessIf(region == 'West', sales)``  
___
### <code>stddev</code>
<code><b>stddev(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的標準差。  
* ``stdDev(sales)``  
___
### <code>stddevIf</code>
<code><b>stddevIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的標準差。  
* ``stddevIf(region == 'West', sales)``  
___
### <code>stddevPopulation</code>
<code><b>stddevPopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的人口標準差。  
* ``stddevPopulation(sales)``  
___
### <code>stddevPopulationIf</code>
<code><b>stddevPopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的人口標準差。  
* ``stddevPopulationIf(region == 'West', sales)``  
___
### <code>stddevSample</code>
<code><b>stddevSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的樣本標準差。  
* ``stddevSample(sales)``  
___
### <code>stddevSampleIf</code>
<code><b>stddevSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的樣本標準差。  
* ``stddevSampleIf(region == 'West', sales)``  
___
### <code>sum</code>
<code><b>sum(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
取得數值資料行的匯總總和。  
* ``sum(col)``  
___
### <code>sumDistinct</code>
<code><b>sumDistinct(<i>&lt;value1&gt;</i> : number) => number</b></code><br/><br/>
取得數值資料行相異值的匯總總和。  
* ``sumDistinct(col)``  
___
### <code>sumDistinctIf</code>
<code><b>sumDistinctIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根據條件取得數值資料行的彙總總和。 條件可以根據任何資料行。  
* ``sumDistinctIf(state == 'CA' && commission < 10000, sales)``  
* ``sumDistinctIf(true, sales)``  
___
### <code>sumIf</code>
<code><b>sumIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => number</b></code><br/><br/>
根據條件取得數值資料行的彙總總和。 條件可以根據任何資料行。  
* ``sumIf(state == 'CA' && commission < 10000, sales)``  
* ``sumIf(true, sales)``  
___
### <code>variance</code>
<code><b>variance(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的變異數。  
* ``variance(sales)``  
___
### <code>varianceIf</code>
<code><b>varianceIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的變異數。  
* ``varianceIf(region == 'West', sales)``  
___
### <code>variancePopulation</code>
<code><b>variancePopulation(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的擴展變異數。  
* ``variancePopulation(sales)``  
___
### <code>variancePopulationIf</code>
<code><b>variancePopulationIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的擴展變異數。  
* ``variancePopulationIf(region == 'West', sales)``  
___
### <code>varianceSample</code>
<code><b>varianceSample(<i>&lt;value1&gt;</i> : number) => double</b></code><br/><br/>
取得資料行的非偏誤變異數。  
* ``varianceSample(sales)``  
___
### <code>varianceSampleIf</code>
<code><b>varianceSampleIf(<i>&lt;value1&gt;</i> : boolean, <i>&lt;value2&gt;</i> : number) => double</b></code><br/><br/>
根據準則，取得資料行的非偏誤變異數。  
* ``varianceSampleIf(region == 'West', sales)``  

## <a name="array-functions"></a>陣列函數
陣列函數會針對屬於陣列的資料結構執行轉換。 這些包括用來定址陣列元素和索引的特殊關鍵字：

* ```#acc``` 表示您想要在縮減陣列時包含在單一輸出中的值。
* ```#index``` 表示目前的陣列索引，以及陣列索引編號 ```#index2, #index3 ...```
* ```#item``` 表示陣列中目前的元素值。

### <code>array</code>
<code><b>array([<i>&lt;value1&gt;</i> : any], ...) => array</b></code><br/><br/>
建立項目陣列。 所有項目都應該是相同的類型。 如果未指定任何項目，則預設值為空字串陣列。 與 [] 建立運算子相同。  
* ``array('Seattle', 'Washington')``
* ``['Seattle', 'Washington']``
* ``['Seattle', 'Washington'][1]``
* ``'Washington'``
___
### <code>filter</code>
<code><b>filter(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => array</b></code><br/><br/>
從陣列中篩選出不符合所提供述詞的元素。 篩選準則必須參考述詞函式中的一個元素做為 #item。  
* ``filter([1, 2, 3, 4], #item > 2) -> [3, 4]``  
* ``filter(['a', 'b', 'c', 'd'], #item == 'a' || #item == 'b') -> ['a', 'b']``  
___
### <code>find</code>
<code><b>find(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></code><br/><br/>
從符合條件的陣列中尋找第一個專案。 它會採用篩選函數，您可以在其中將陣列中的專案定址為 #item。 針對深層嵌套的對應，您可以使用 #item_n ( # item_1，#item_2 ... ) 標記法來參考父地圖。  
* ``find([10, 20, 30], #item > 10) -> 20``
* ``find(['azure', 'data', 'factory'], length(#item) > 4) -> 'azure'``
* ``find([
      @(
         name = 'Daniel',
         types = [
                   @(mood = 'jovial', behavior = 'terrific'),
                   @(mood = 'grumpy', behavior = 'bad')
                 ]
        ),
      @(
         name = 'Mark',
         types = [
                   @(mood = 'happy', behavior = 'awesome'),
                   @(mood = 'calm', behavior = 'reclusive')
                 ]
        )
      ],
      contains(#item.types, #item.mood=='happy')  /*Filter out the happy kid*/
    )``
* ``
     @(
           name = 'Mark',
           types = [
                     @(mood = 'happy', behavior = 'awesome'),
                     @(mood = 'calm', behavior = 'reclusive')
                   ]
      )
    ``  
___
### <code>map</code>
<code><b>map(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></code><br/><br/> 會使用提供的運算式，將陣列的每個專案對應至新的元素。 Map 預期運算式中有一個元素的參考 function as #item.  
* ``map([1, 2, 3, 4], #item + 2) -> [3, 4, 5, 6]``  
* ``map(['a', 'b', 'c', 'd'], #item + '_processed') -> ['a_processed', 'b_processed', 'c_processed', 'd_processed']``  
___
### <code>mapIndex</code>
<code><b>mapIndex(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => any</b></code><br/><br/> 會使用提供的運算式，將陣列的每個專案對應至新的元素。 Map 預期 expression 函式中一個元素的參考是 #item 和元素的參考 index as #index.  
* ``mapIndex([1, 2, 3, 4], #item + 2 + #index) -> [4, 6, 8, 10]``  
___
### <code>reduce</code>
<code><b>reduce(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : any, <i>&lt;value3&gt;</i> : binaryfunction, <i>&lt;value4&gt;</i> : unaryfunction) => any</b></程式碼><br/><br/> 會累積陣列中的元素。 在第一個 expression 函式中，減少需要累計和一個元素的參考做為 #acc 和 #item，而且它預期產生的值會是要在第二個 expr 中使用的 #resultession function.  
* ``toString(reduce(['1', '2', '3', '4'], '0', #acc + #item, #result)) -> '01234'``  
___
### <code>size</code>
<code><b>size(<i>&lt;value1&gt;</i> : any) => integer</b></程式碼><br/><br/> 尋找 a 的大小rray or map type  
* ``size(['element1', 'element2']) -> 2``
* ``size([1,2,3]) -> 3``
___
### <code>slice</code>
<code><b>slice(<i>&lt;array to slice&gt;</i> : array, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of items&gt;</i> : integral]) => array</b></程式碼><br/><br/> 從位置解壓縮陣列的子集。 位置從 1 開始。 如果省略長度，則預設為 end of the string.  
* ``slice([10, 20, 30, 40], 1, 2) -> [10, 20]``  
* ``slice([10, 20, 30, 40], 2) -> [20, 30, 40]``  
* ``slice([10, 20, 30, 40], 2)[1] -> 20``  
* ``isNull(slice([10, 20, 30, 40], 2)[0]) -> true``  
* ``isNull(slice([10, 20, 30, 40], 2)[20]) -> true``  
* ``slice(['a', 'b', 'c', 'd'], 8) -> []``  
___
### <code>sort</code>
<code><b>sort(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => array</b></程式碼><br/><br/> 會使用提供的述詞函式來排序陣列。 排序需要運算式函式中兩個連續元素的參考做為 #item1 and #item2.  
* ``sort([4, 8, 2, 3], compare(#item1, #item2)) -> [2, 3, 4, 8]``  
* ``sort(['a3', 'b2', 'c1'], iif(right(#item1, 1) >= right(#item2, 1), 1, -1)) -> ['c1', 'b2', 'a3']``* ``
 @(
       name = 'Mark',
       types = [
                 @(mood = 'happy', behavior = 'awesome'),
                 @(mood = 'calm', behavior = 'reclusive')
               ]
  )
``  
___
### <code>map</code>
<code><b>map(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : unaryfunction) => any</b></code><br/><br/>
Maps each element of the array to a new element using the provided expression. Map expects a reference to one element in the expression function as #item.  
* ``map([1, 2, 3, 4], #item + 2) -> [3, 4, 5, 6]``  
* ``map(['a', 'b', 'c', 'd'], #item + '_processed') -> ['a_processed', 'b_processed', 'c_processed', 'd_processed']``  
___
### <code>mapIndex</code>
<code><b>mapIndex(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => any</b></code><br/><br/>
Maps each element of the array to a new element using the provided expression. Map expects a reference to one element in the expression function as #item and a reference to the element index as #index.  
* ``mapIndex([1, 2, 3, 4], #item + 2 + #index) -> [4, 6, 8, 10]``  
___
### <code>reduce</code>
<code><b>reduce(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : any, <i>&lt;value3&gt;</i> : binaryfunction, <i>&lt;value4&gt;</i> : unaryfunction) => any</b></code><br/><br/>
Accumulates elements in an array. Reduce expects a reference to an accumulator and one element in the first expression function as #acc and #item and it expects the resulting value as #result to be used in the second expression function.  
* ``toString(reduce(['1', '2', '3', '4'], '0', #acc + #item, #result)) -> '01234'``  
___
### <code>size</code>
<code><b>size(<i>&lt;value1&gt;</i> : any) => integer</b></code><br/><br/>
Finds the size of an array or map type  
* ``size(['element1', 'element2']) -> 2``
* ``size([1,2,3]) -> 3``
___
### <code>slice</code>
<code><b>slice(<i>&lt;array to slice&gt;</i> : array, <i>&lt;from 1-based index&gt;</i> : integral, [<i>&lt;number of items&gt;</i> : integral]) => array</b></code><br/><br/>
Extracts a subset of an array from a position. Position is 1 based. If the length is omitted, it is defaulted to end of the string.  
* ``slice([10, 20, 30, 40], 1, 2) -> [10, 20]``  
* ``slice([10, 20, 30, 40], 2) -> [20, 30, 40]``  
* ``slice([10, 20, 30, 40], 2)[1] -> 20``  
* ``isNull(slice([10, 20, 30, 40], 2)[0]) -> true``  
* ``isNull(slice([10, 20, 30, 40], 2)[20]) -> true``  
* ``slice(['a', 'b', 'c', 'd'], 8) -> []``  
___
### <code>sort</code>
<code><b>sort(<i>&lt;value1&gt;</i> : array, <i>&lt;value2&gt;</i> : binaryfunction) => array</b></code><br/><br/>
Sorts the array using the provided predicate function. Sort expects a reference to two consecutive elements in the expression function as #item1 and #item2.  
* ``sort([4, 8, 2, 3], compare(#item1, #item2)) -> [2, 3, 4, 8]``  
* ``sort(['a3', 'b2', 'c1'], iif(right(#item1, 1) >= right(#item2, 1), 1, -1)) -> ['c1', 'b2', 'a3']``  


## <a name="conversion-functions"></a>轉換函數

轉換函數可用來轉換資料和資料類型

### <code>toBase64</code>
<code><b>toBase64(<i>&lt;value1&gt;</i> : string) => string</b></code><br/><br/>
以 base64 編碼指定的字串。  
* ``toBase64('bojjus') -> 'Ym9qanVz'``  
___
### <code>toBinary</code>
<code><b>toBinary(<i>&lt;value1&gt;</i> : any) => binary</b></code><br/><br/>
將任何數值/日期/時間戳記/字串轉換為二進位標記法。  
* ``toBinary(3) -> [0x11]``  
___
### <code>toBoolean</code>
<code><b>toBoolean(<i>&lt;value1&gt;</i> : string) => boolean</b></code><br/><br/>
將 ( t '、' true '、' y '、' yes '、' 1 ' ) 的值轉換為 true，並 ( ' f '、' false '、' n '、' no '、' 0 ' ) 設為 false，並將 Null 用於任何其他值。  
* ``toBoolean('true') -> true``  
* ``toBoolean('n') -> false``  
* ``isNull(toBoolean('truthy')) -> true``  
___
### <code>toByte</code>
<code><b>toByte(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => byte</b></code><br/><br/>
將任何數值或字串轉換為位元組值。 可使用選擇性的 Java 十進位格式進行轉換。  
* ``toByte(123)``
* ``123``
* ``toByte(0xFF)``
* ``-1``
* ``toByte('123')``
* ``123``
___
### <code>toDate</code>
<code><b>toDate(<i>&lt;string&gt;</i> : any, [<i>&lt;date format&gt;</i> : string]) => date</b></code><br/><br/>
使用選擇性輸入日期格式，將輸入日期字串轉換為日期。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 如果省略輸入日期格式，則預設格式為 yyyy-[M]M-[d]d。 接受的格式為： [yyyy，yyyy-[M] M，yyyy-[M] M-[d] d，yyyy-[M] M-[d] dT *]。  
* ``toDate('2012-8-18') -> toDate('2012-08-18')``  
* ``toDate('12/18/2012', 'MM/dd/yyyy') -> toDate('2012-12-18')``  
___
### <code>toDecimal</code>
<code><b>toDecimal(<i>&lt;value&gt;</i> : any, [<i>&lt;precision&gt;</i> : integral], [<i>&lt;scale&gt;</i> : integral], [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => decimal(10,0)</b></code><br/><br/>
將任何數值或字串轉換為小數值。 若未指定有效位數和小數位數，則預設為 (10,2)。可使用選用的 Java 十進位格式進行轉換。 >BCP47 語言格式的選擇性地區設定格式，例如 en-us、de、zh-CN。  
* ``toDecimal(123.45) -> 123.45``  
* ``toDecimal('123.45', 8, 4) -> 123.4500``  
* ``toDecimal('$123.45', 8, 4,'$###.00') -> 123.4500``  
* ``toDecimal('Ç123,45', 10, 2, 'Ç###,##', 'de') -> 123.45``  
___
### <code>toDouble</code>
<code><b>toDouble(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => double</b></code><br/><br/>
將任何數值或字串轉換為雙精準數值。 可使用選擇性的 Java 十進位格式進行轉換。 >BCP47 語言格式的選擇性地區設定格式，例如 en-us、de、zh-CN。  
* ``toDouble(123.45) -> 123.45``  
* ``toDouble('123.45') -> 123.45``  
* ``toDouble('$123.45', '$###.00') -> 123.45``  
* ``toDouble('Ç123,45', 'Ç###,##', 'de') -> 123.45``  
___
### <code>toFloat</code>
<code><b>toFloat(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => float</b></code><br/><br/>
將任何數值或字串轉換為浮點值。 可使用選擇性的 Java 十進位格式進行轉換。 截斷任何雙精度浮點數。  
* ``toFloat(123.45) -> 123.45f``  
* ``toFloat('123.45') -> 123.45f``  
* ``toFloat('$123.45', '$###.00') -> 123.45f``  
___
### <code>toInteger</code>
<code><b>toInteger(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => integer</b></code><br/><br/>
將任何數值或字串轉換為整數值。 可使用選擇性的 Java 十進位格式進行轉換。 截斷任何 long、float、double。  
* ``toInteger(123) -> 123``  
* ``toInteger('123') -> 123``  
* ``toInteger('$123', '$###') -> 123``  
___
### <code>toLong</code>
<code><b>toLong(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => long</b></code><br/><br/>
將任何數值或字串轉換為長值。 可使用選擇性的 Java 十進位格式進行轉換。 截斷任何浮點數、雙精度浮點數。  
* ``toLong(123) -> 123``  
* ``toLong('123') -> 123``  
* ``toLong('$123', '$###') -> 123``  
___
### <code>toShort</code>
<code><b>toShort(<i>&lt;value&gt;</i> : any, [<i>&lt;format&gt;</i> : string], [<i>&lt;locale&gt;</i> : string]) => short</b></code><br/><br/>
將任何數值或字串轉換為短值。 可使用選擇性的 Java 十進位格式進行轉換。 截斷任何整數、長值、浮點數、雙精度浮點數。  
* ``toShort(123) -> 123``  
* ``toShort('123') -> 123``  
* ``toShort('$123', '$###') -> 123``  
___
### <code>toString</code>
<code><b>toString(<i>&lt;value&gt;</i> : any, [<i>&lt;number format/date format&gt;</i> : string]) => string</b></code><br/><br/>
將基本資料類型轉換為字串。 針對數字和日期，可以指定格式。 若未指定，則會選擇系統預設值。對於數字會使用 Java 十進位格式。 請參閱 JAVA SimpleDateFormat 以取得所有可能的日期格式;預設格式為 yyyy-mm-dd。  
* ``toString(10) -> '10'``  
* ``toString('engineer') -> 'engineer'``  
* ``toString(123456.789, '##,###.##') -> '123,456.79'``  
* ``toString(123.78, '000000.000') -> '000123.780'``  
* ``toString(12345, '##0.#####E0') -> '12.345E3'``  
* ``toString(toDate('2018-12-31')) -> '2018-12-31'``  
* ``isNull(toString(toDate('2018-12-31', 'MM/dd/yy'))) -> true``  
* ``toString(4 == 20) -> 'false'``  
___
### <code>toTimestamp</code>
<code><b>toTimestamp(<i>&lt;string&gt;</i> : any, [<i>&lt;timestamp format&gt;</i> : string], [<i>&lt;time zone&gt;</i> : string]) => timestamp</b></code><br/><br/>
以指定的選擇性時間戳記格式將字串轉換為時間戳記。 如果省略時間戳記，則會使用預設模式 yyyy-[M] M-[d] d hh： mm： ss [f. ...]。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 Timestamp 支援最高毫秒的精確度，且值為999。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``toTimestamp('2016-12-31 00:12:00') -> toTimestamp('2016-12-31 00:12:00')``  
* ``toTimestamp('2016-12-31T00:12:00', 'yyyy-MM-dd\'T\'HH:mm:ss', 'PST') -> toTimestamp('2016-12-31 00:12:00')``  
* ``toTimestamp('12/31/2016T00:12:00', 'MM/dd/yyyy\'T\'HH:mm:ss') -> toTimestamp('2016-12-31 00:12:00')``  
* ``millisecond(toTimestamp('2019-02-03 05:19:28.871', 'yyyy-MM-dd HH:mm:ss.SSS')) -> 871``  
___
### <code>toUTC</code>
<code><b>toUTC(<i>&lt;value1&gt;</i> : timestamp, [<i>&lt;value2&gt;</i> : string]) => timestamp</b></code><br/><br/>
將時間戳記轉換為 UTC。 您可以用 'GMT'、'PST'、'UTC'、'America/Cayman' 的格式傳遞選擇性的時區。 其預設值為目前的時區。 請參閱 JAVA 的 `SimpleDateFormat` 類別以取得可用的格式。 https://docs.oracle.com/javase/8/docs/api/java/text/SimpleDateFormat.html.  
* ``toUTC(currentTimestamp()) == toTimestamp('2050-12-12 19:18:12') -> false``  
* ``toUTC(currentTimestamp(), 'Asia/Seoul') != toTimestamp('2050-12-12 19:18:12') -> true``  

## <a name="metafunctions"></a>Metafunctions

Metafunctions 主要是在資料流程的中繼資料上運作

### <code>byItem</code>
<code><b>byItem(<i>&lt;parent column&gt;</i> : any, <i>&lt;column name&gt;</i> : string) => any</b></code><br/><br/>
如果有多個相符專案，請在結構或結構陣列中尋找子專案，然後傳回第一個相符專案。 如果沒有相符項目，則會傳回 Null 值。 傳回的值必須是類型轉換動作 ( 的轉換類型？ 日期？ 字串 ... ) 。 在設計階段已知的資料行名稱，應該只由其名稱定址。 不支援計算的輸入，但您可以使用參數替代 * ``byItem( byName('customer'), 'orderItems') ? (itemName as string, itemQty as integer)``
* ````
* ``byItem( byItem( byName('customer'), 'orderItems'), 'itemName') ? string``
* ````
___
### <code>byOrigin</code>
<code><b>byOrigin(<i>&lt;column name&gt;</i> : string, [<i>&lt;origin stream name&gt;</i> : string]) => any</b></code><br/><br/>
依名稱選取原始資料流程中的資料行值。 第二個引數是原始資料流程名稱。 如有多個相符項目，則傳回第一個相符項目。 如果沒有相符項目，則會傳回 Null 值。 傳回的值必須是類型轉換函數之一轉換的類型 (TO_DATE，TO_STRING ... ) 。在設計階段已知的資料行名稱，應該只由其名稱定址。 不支援計算的輸入，但您可使用參數替換。  
* ``toString(byOrigin('ancestor', 'ancestorStream'))``
___
### <code>byOrigins</code>
<code><b>byOrigins(<i>&lt;column names&gt;</i> : array, [<i>&lt;origin stream name&gt;</i> : string]) => any</b></code><br/><br/>
依名稱選取資料流程中的資料行陣列。 第二個引數是其來源的資料流程。 如有多個相符項目，則傳回第一個相符項目。 如果沒有相符項目，則會傳回 Null 值。 傳回的值必須是類型轉換函式所轉換的型別 (TO_DATE，TO_STRING ... 在設計階段所知道的資料行名稱 ) 應該只由其名稱定址。 不支援計算的輸入，但您可使用參數替換。
* ``toString(byOrigins(['ancestor1', 'ancestor2'], 'ancestorStream'))``
___
### <code>byName</code>
<code><b>byName(<i>&lt;column name&gt;</i> : string, [<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
依名稱選取資料流中的資料行值。 您可以傳遞選擇性資料流名稱作為第二個引數。 如有多個相符項目，則傳回第一個相符項目。 如果沒有相符項目，則會傳回 Null 值。 傳回的值必須是由其中一個類型轉換函數 (TO_DATE、TO_STRING ...) 所轉換的類型。在設計階段已知的資料行名稱，應該只依據其名稱來處理。 不支援計算的輸入，但您可使用參數替換。  
* ``toString(byName('parent'))``  
* ``toLong(byName('income'))``  
* ``toBoolean(byName('foster'))``  
* ``toLong(byName($debtCol))``  
* ``toString(byName('Bogus Column'))``  
* ``toString(byName('Bogus Column', 'DeriveStream'))``  
___
### <code>byNames</code>
<code><b>byNames(<i>&lt;column names&gt;</i> : array, [<i>&lt;stream name&gt;</i> : string]) => any</b></code><br/><br/>
在資料流中依名稱選取資料行陣列。 您可以傳遞選擇性資料流名稱作為第二個引數。 如有多個相符項目，則傳回第一個相符項目。 如果資料行沒有相符項目，則整個輸出會是 NULL 值。 傳回的值需要類型轉換函式 (toDate、toString、...)。在設計階段已知的資料行名稱，應該只依據其名稱來處理。 不支援計算的輸入，但您可使用參數替換。
* ``toString(byNames(['parent', 'child']))``
* ``byNames(['parent']) ? string``
* ``toLong(byNames(['income']))``
* ``byNames(['income']) ? long``
* ``toBoolean(byNames(['foster']))``
* ``toLong(byNames($debtCols))``
* ``toString(byNames(['a Column']))``
* ``toString(byNames(['a Column'], 'DeriveStream'))``
* ``byNames(['orderItem']) ? (itemName as string, itemQty as integer)``
___
### <code>byPosition</code>
<code><b>byPosition(<i>&lt;position&gt;</i> : integer) => any</b></code><br/><br/>
依其在資料流中的相對位置 (從 1 開始) 來選取資料行值。 如果位置超出範圍，則會傳回 Null 值。 傳回的值必須是類型轉換函式的轉換類型 (TO_DATE，TO_STRING ... ) 計算輸入不受支援，但您可以使用參數替代。  
* ``toString(byPosition(1))``  
* ``toDecimal(byPosition(2), 10, 2)``  
* ``toBoolean(byName(4))``  
* ``toString(byName($colName))``  
* ``toString(byPosition(1234))``  

## <a name="cached-lookup-functions"></a>快取的查閱函數
下列函式只有在您已包含快取的接收時，才可使用快取的查詢。
___
### <code>lookup</code>
<code><b>lookup(key, key2, ...) => complex[]</b></code><br/><br/>
使用與快取接收中的索引鍵相符的指定索引鍵，從快取接收中查閱第一個資料列。
* ``cacheSink#lookup(movieId)``  
___
### <code>mlookup</code>
<code><b>mlookup(key, key2, ...) => complex[]</b></code><br/><br/>
使用與快取接收中的索引鍵相符的指定索引鍵，從快取接收中查閱所有相符的資料列。
* ``cacheSink#mlookup(movieId)``  
___
### <code>output</code>
<code><b>output() => any</b></code><br/><br/>
傳回快取接收結果的第一個資料列 * ``cacheSink#output()``  
___
### <code>outputs</code>
<code><b>output() => any</b></code><br/><br/>
傳回快取接收結果的整個輸出資料列集 * ``cacheSink#outputs()``
___

## <a name="window-functions"></a>視窗函式
下列函數僅適用于視窗轉換。
___
### <code>cumeDist</code>
<code><b>cumeDist() => integer</b></code><br/><br/>
CumeDist 函式會計算某個值與分割區中所有值的相對位置。 依照分割區的排序，將目前資料列 (含此列) 之前的列數，除以視窗分割區中的總列數，即為結果。 排序中的任何相等值都會評估為相同的位置。  
* ``cumeDist()``  
___
### <code>denseRank</code>
<code><b>denseRank() => integer</b></code><br/><br/>
在視窗的 order by 子句中指定的值群組中，計算某個值的順位。 依照分割區的排序，將目前資料列 (含此列) 之前的列數加一，即為結果。 這些值不會在序列中產生間距。 即使資料未排序，且在值中尋找變更，密集排名仍可運作。  
* ``denseRank()``  
___
### <code>lag</code>
<code><b>lag(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look before&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
取得第一個參數對目前資料列之前的 n 個資料列評估的值。 第二個參數是要回溯查看的資料列數目，預設值為 1。 如果沒有任何資料列，除非指定了預設值，否則會傳回 null 值。  
* ``lag(amount, 2)``  
* ``lag(amount, 2000, 100)``  
___
### <code>lead</code>
<code><b>lead(<i>&lt;value&gt;</i> : any, [<i>&lt;number of rows to look after&gt;</i> : number], [<i>&lt;default value&gt;</i> : any]) => any</b></code><br/><br/>
取得第一個參數對目前資料列之後的 n 個資料列評估的值。 第二個參數是要正向查看的資料列數目，預設值為 1。 如果沒有任何資料列，除非指定了預設值，否則會傳回 null 值。  
* ``lead(amount, 2)``  
* ``lead(amount, 2000, 100)``  
___
### <code>nTile</code>
<code><b>nTile([<i>&lt;value1&gt;</i> : integer]) => integer</b></code><br/><br/>
函式會將 ```NTile``` 每個視窗分割區的資料列分割成 `n` 介於1到最多的 bucket `n` 。 貯體值的最大差異為 1。 如果分割區中的資料列數目無法平均分成貯體數，其餘值將會逐一均分給各個貯體，從第一個貯體開始。 ```NTile```函數適用于計算 ```tertiles``` 、分量、十分位數和其他常見的摘要統計資料。 此函式會在初始化期間計算兩個變數：一般貯體的大小會新增一個額外的資料列。 這兩個變數都是以目前分割區的大小為基礎。 在計算過程中，此函式會追蹤目前的資料列數目、目前的貯體數目，和貯體將有所變更的資料列號碼 (bucketThreshold)。 當目前的資料列數目達到貯體閾值時，貯體值將會加一，而閾值會依貯體大小增加 (如果目前的貯體已填補，則額外再加一)。  
* ``nTile()``  
* ``nTile(numOfBuckets)``  
___
### <code>rank</code>
<code><b>rank() => integer</b></code><br/><br/>
在視窗的 order by 子句中指定的值群組中，計算某個值的順位。 依照分割區的排序，將目前資料列 (含此列) 之前的列數加一，即為結果。 這些值會在序列中產生間距。 即使資料未排序，並在值中尋找變更，排名仍可運作。  
* ``rank()``  
___
### <code>rowNumber</code>
<code><b>rowNumber() => integer</b></code><br/><br/>
為視窗中的資料列指派以1為開頭的連續資料列編號。  
* ``rowNumber()``  

## <a name="next-steps"></a>後續步驟

[了解如何使用運算式產生器](concepts-data-flow-expression-builder.md)。
