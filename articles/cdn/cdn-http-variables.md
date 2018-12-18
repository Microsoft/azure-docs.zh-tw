---
title: Azure CDN 規則引擎的 HTTP 變數 | Microsoft Docs
description: HTTP 變數可讓您擷取 HTTP 要求和回應中繼資料。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/09/2018
ms.author: magattus
ms.openlocfilehash: 39084fd8408a123e8152ad96fa92025fd04ed42b
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092808"
---
# <a name="http-variables-for-azure-cdn-rules-engine"></a>Azure CDN 規則引擎的 HTTP 變數
HTTP 變數能提供擷取 HTTP 要求和回應中繼資料的方法。 此中繼資料可以接著用來動態調整要求或回應。 HTTP 變數的使用，僅限於下列規則引擎功能：

- [快取索引鍵重寫](cdn-rules-engine-reference-features.md#cache-key-rewrite)
- [修改用戶端要求標頭](cdn-rules-engine-reference-features.md#modify-client-request-header)
- [修改用戶端回應標頭](cdn-rules-engine-reference-features.md#modify-client-response-header)
- [URL 重新導向](cdn-rules-engine-reference-features.md#url-redirect)
- [URL 重寫](cdn-rules-engine-reference-features.md#url-rewrite)

## <a name="definitions"></a>定義
下表描述支援的 HTTP 變數。 在無法針對特定要求提供地理中繼資料 (例如郵遞區號) 時，系統將會傳回空白值。


| 名稱 | 變數 | 說明 | 範例值 |
| ---- | -------- | ----------- | ------------ |
| ASN (要求者) | %{geo_asnum} | 表示要求者的 AS 號碼。 <br /><br />**已淘汰：**%{virt_dst_asnum}。 <br />此變數已由 %{geo_asnum} 取代。 雖然使用此已淘汰變數的規則仍然可以運作，您應該更新規則以使用新的變數。 | AS15133 |
| 城市 (要求者) | %{geo_city} | 表示要求者的城市。 | 洛杉磯 |
| 大陸 (要求者) | %{geo_continent} | 透過縮寫表示要求者的大陸。 <br />有效值為： <br />AF：非洲<br />AS：亞洲<br />EU：歐洲<br />NA：北美洲<br />OC：大洋洲<br />SA：南美洲<br /><br />**已淘汰：**%{virt_dst_continent}。 <ber />此變數已由 %{geo_continent} 取代。 <br />雖然使用此已淘汰變數的規則仍然可以運作，您應該更新規則以使用新的變數。| N/A |
| Cookie 值 | %{cookie_Cookie} | 傳回 Cookie 字詞所識別 Cookie 索引鍵的對應值。 | 範例用法： <br />%{cookie__utma}<br /><br />範例值：<br />111662281.2.10.1222100123 |
| 國家 (要求者) | %{geo_country} | 透過國碼 (地區碼) 表示要求者的國家/地區。 <br />**已淘汰：**%{virt_dst_country}。 <br /><br />此變數已由 %{geo_country} 取代。 雖然使用此已淘汰變數的規則仍然可以運作，您應該更新規則以使用新的變數。 | US |
| 指定市場地區 (Requester) | %{geo_dma_code} |依地區碼表示要求者的媒體市場。 <br /><br />此欄位僅適用於源自美國的要求。| 745 |
| HTTP 要求方法 | %{request_method} | 表示 HTTP 要求方法。 | GET |
| HTTP 狀態碼 | %{status} | 表示回應的 HTTP 狀態碼。 | 200 |
| IP 位址 (要求者) | %{virt_dst_addr} | 表示要求者的 IP 位址。 | 192.168.1.1 |
| 緯度 (要求者) | %{geo_latitude} | 表示要求者的緯度。 | 34.0995 |
| 經度 (要求者) | %{geo_longitude} | 表示要求者的經度。 | -118.4143 |
| 都會統計資料區域 (要求者) | %{geo_metro_code} | 表示要求者的都會區域。 <br /><br />此欄位僅適用於源自美國的要求。<br />| 745 |
| 連接埠 (要求者) | %{virt_dst_port} | 表示要求者的暫時連接埠。 | 55885 |
| 郵遞區號 (要求者) | %{geo_postal_code} | 表示要求者的郵遞區號。 | 90210 |
| 找到查詢字串 | %{is_args} | 此變數的值會根據要求是否包含查詢字串而有所不同。<br /><br />- 找到查詢字串：?<br />- 沒有查詢字串：NULL | ? |
| 找到查詢字串參數 | %{is_amp} | 此變數的值會根據要求是否包含至少一個查詢字串參數而有所不同。<br /><br />- 找到參數：&<br />- 沒有參數：NULL | & |
| 查詢字串參數值 | %{arg_&lt;parameter&gt;} | 傳回 &lt;parameter&gt; 字詞所識別查詢字串參數的對應值。 | 範例用法： <br />%{arg_language}<br /><br />範例查詢字串參數： <br />?language=en<br /><br />範例值：en |
| 查詢字串值 | %{query_string} | 表示定義於要求 URL 中的完整查詢字串值。 |key1=val1&key2=val2&key3=val3 |
| 推薦者網域 | %{referring_domain} | 表示定義於推薦者要求標頭中的網域。 | www.google.com |
| 地區 (要求者) | %{geo_region} | 透過英數字元縮寫表示要求者的地區 (例如州或省)。 | CA |
| 要求標頭值 | %{http_RequestHeader} | 傳回 RequestHeader 字詞所識別要求標頭的對應值。 <br /><br />如果要求標頭包含破折號 (例如 User-Agent)，則會以底線來取代 (例如 User_Agent)。| 範例用法：%{http_Connection}<br /><br />範例值：Keep-Alive | 
| 要求主機 | %{host} | 表示定義於要求 URL 中的主機。 | www.mydomain.com |
| 要求通訊協定 | %{request_protocol} | 表示要求通訊協定。 | HTTP/1.1 |
| 要求配置 | %{scheme} | 表示要求配置。 |http |
| 要求 URI (相對) | %{request_uri} | 表示定義於要求 URI 中的相對路徑 (包括查詢字串)。 | /marketing/foo.js?loggedin=true |
| 要求 URI (相對，不含查詢字串) | %{uri} | 表示被要求內容的相對路徑。 <br /><br/>重要資訊：<br />- 此相對路徑不包含查詢字串。<br />- 此相對路徑會反映 URL 重寫。 URL 將會在下列情況下被重寫：<br />  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- URL 重寫功能：此功能會對定義於要求 URI 中的相對路徑進行重寫。<br />    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- 邊緣 CNAME URL：此類型的要求會被重寫至相對應的 CDN URL。 |/800001/corigin/rewrittendir/foo.js |
| 要求 URI | %{request} | 描述要求。 <br />語法：&lt;HTTP 方法&gt; &lt;相對路徑&gt; &lt;HTTP 通訊協定&gt; | GET /marketing/foo.js?loggedin=true HTTP/1.1 |
| 回應標頭值 | %{resp_&lt;ResponseHeader&gt;} | 傳回 &lt;ResponseHeader&gt; 字詞所識別回應標頭的對應值。 <br /><br />如果回應標頭包含破折號 (例如 User-Agent)，則會以底線來取代 (例如 User_Agent)。 | 範例用法：%{resp_Content_Length}<br /><br />範例值：100 |

## <a name="usage"></a>使用量
下表說明指定 HTTP 變數的正確語法。


| 語法 | 範例 | 說明 |
| ------ | -------- | ---------- |
| %{&lt;HTTPVariable&gt;} | %{host} | 使用此語法以取得對應至指定 &lt;HTTPVariable&gt; 的完整值。 |
| %{&lt;HTTPVariableDelimiter&gt;} | %{host,} | 使用此語法針對對應至指定 &lt;HTTPVariableDelimiter&gt; 的完整值設定大小寫。 |
| %{&lt;HTTPVariableDelimiterExpression&gt;} | %{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80} | 使用針對 &lt;HTTPVariableDelimiterExpression&gt; 的規則運算式來取代、刪除或操作 HTTP 變數的值。 |

HTTP 變數名稱僅支援字母字元和底線。 系統會將不支援的字元轉換為底線。

## <a name="delimiter-reference"></a>分隔符號參考
可以在 HTTP 變數之後指定分隔符號，以達成下列任一效果：

- 轉換變數關聯值。

     範例：將整個值轉換成小寫。

- 刪除變數關聯值。

- 操作變數關聯值。

     範例：使用規則運算式來變更 HTTP 變數關聯值。

下表會說明這些分隔符號。

| 分隔符號 | 說明 |
| --------- | ----------- |
| := | 表示將會在變數滿足下列其中一個條件時，將預設值指派給變數： <br />- 遺漏 <br />- 設定為 NULL。 |
| :+ | 表示在變數有指派值的情況下，將預設值指派給變數。 |
| ： | 表示將會展開變數指派值的子字串。 |
| # | 表示若在變數關聯值的開頭找到此分隔符號之後所指定的模式時，應該將該模式刪除。 |
| % | 表示若在變數關聯值的尾端找到在此分隔符號之後所指定的模式時，應該將該模式刪除。 <br />只有在使用 % 符號作為分隔符號時才適用此定義。 |
| / | 分隔 HTTP 變數或模式。 |
| // | 尋找並取代指定模式的所有執行個體。 |
| /= | 尋找，複製並重寫指定模式的所有發生案例。 |
| ， | 將 HTTP 變數關聯值轉換成小寫。 |
| ^ | 將 HTTP 變數關聯值轉換成大寫。 |
| ,, | 將 HTTP 變數關聯值中指定字元的所有執行個體轉換成小寫。 |
| ^^ | 將 HTTP 變數關聯值中指定字元的所有執行個體轉換成大寫。 |

## <a name="exceptions"></a>例外狀況
下表說明系統不會將指定文字視為 HTTP 變數的情況。

| 條件 | 說明 | 範例 |
| --------- | ----------- | --------|
| 逸出 % 符號 | 百分比符號可以使用反斜線來逸出。 <br />右側的範例值將會被系統視為常值而非 HTTP 變數。| \%{host} |
| 未知的變數 | 系統針對未知的變數一律會傳回空字串。 | %{unknownvariable} |
| 無效的字元或語法 | 包含無效的字元或語法的變數會被系統視為常值。 <br /><br />範例 #1：指定的值包含無效的字元 (例如 -)。 <br /><br />範例 #2：指定的值包含兩組大括號。 <br /><br />範例 #3：指定的值遺漏右大括號。<br /> | 範例 #1：%{resp_user-agent} <br /><br />範例 #2：%{{host}} <br /><br />範例 #3：%{host |
| 遺漏變數名稱 | 在沒有指定變數的情況下，一律會傳回 NULL 值。 | %{} |
| 尾端字元 | 位於變數尾端的字元一律會被視為常值。 <br />右側的範例值包含尾端的大括號，系統將會把它視為常值。 | %{host}} |

## <a name="setting-default-header-values"></a>設定預設標頭值
當標頭符合下列任一條件時，便可以將預設值指派給標頭：
- 遺漏/未設定
- 設定為 NULL。

下表說明如何定義預設值。

| 條件 | 語法 | 範例 | 說明 |
| --------- | ------ | --------| ----------- |
| 在標頭符合下列任一條件時，將標頭設定為預設值： <br /><br />- 遺漏標頭 <br /><br />- 標頭值已設為 NULL。| %{Variable:=Value} | %{http_referer:=unspecified} | 在推薦者標頭遺漏或設定為 NULL 的情況下，只會被設為 *unspecified*。 在已設定的情況下，系統將不會採取任何動作。 |
| 在遺漏標頭的情況下，將標頭設定為預設值。 | %{Variable=Value} | %{http_referer=unspecified} | 在推薦者標頭遺漏的情況下，只會被設為 *unspecified*。 在已設定的情況下，系統將不會採取任何動作。 |
| 在標頭不符合下列任一條件時，將標頭設定為預設值： <br /><br />- 遺漏<br /><br /> - 設定為 NULL。 | %{Variable:+Value} | %{http_referer:+unspecified} | 在推薦者標頭有指派值的情況下，只會被設為 *unspecified*。 在遺漏或設定為 NULL 的情況下，系統將不會採取任何動作。 |

## <a name="manipulating-variables"></a>操作變數
變數可以透過下列方式進行操作：

- 展開子字串
- 移除模式

### <a name="substring-expansion"></a>子字串展開
根據預設，變數會展開成其完整的值。 使用下列語法來僅展開變數值的子字串。

`%<Variable>:<Offset>:<Length>}`

重要資訊：

- 指派至 Offset 字詞的值會決定子字串的起始字元：

     - 正值：子字串的起始字元是從字串的第一個字元來計算。
     - 零：子字串的起始字元是字串中的第一個字元。
     - 負值：子字串的起始字元是從字串的最後一個字元來計算。

- 子字串的長度是由 *Length*字詞所決定：

     - 省略：省略 Length 字詞彙允許子字串包含起始字元及字串結尾之間的所有字元。
     - 正值：決定子字串從起始字元向右計算的長度。
     - 負值：決定子字串從起始字元向左計算的長度。

#### <a name="example"></a>範例：

下列範例需仰賴下列範例要求 URL：

https:\//cdn.mydomain.com/folder/marketing/myconsultant/proposal.html

下列字串示範操作變數的各種不同方法：

https:\//www%{http_host:3}/mobile/%{request_uri:7:10}/%{request_uri:-5:-8}.htm

根據範例要求 URL，上述變數操作將會產生下列值：

https:\//www.mydomain.com/mobile/marketing/proposal.htm


### <a name="pattern-removal"></a>模式移除
可以從變數值的開頭或結尾移除符合指定模式的文字。

| 語法 | 動作 |
| ------ | ------ |
| %{Variable#Pattern} | 在於變數值的開頭找到指定模式時移除文字。 |
| %{Variable%Pattern} | 在於變數值的結尾找到指定模式時移除文字。 |

#### <a name="example"></a>範例：

在此範例案例中，*request_uri* 變數是設為：

`/800001/myorigin/marketing/product.html?language=en-US`

下表示範此語法的運作方式。

| 範例語法 | 結果 |
| ------------- | ------- |
| %{request_uri#/800001}/customerorigin | /customerorigin/myorigin/marketing/product.html?language=en-US | 由於變數是以該模式作為開頭，因此會被取代。 |
| %{request_uri%html}htm | /800001/myorigin/marketing/product.html?language=en-US | 由於變數沒有以該模式作為結尾，因此沒有任何變更。|

### <a name="find-and-replace"></a>尋找並取代
下表描述尋找並取代的語法。

| 語法 | 動作 |
| ------ | ------ |
| %{Variable/Find/Replace} | 尋找並取代指定模式的首個發生案例。 |
| %{Variable//Find/Replace} | 尋找並取代指定模式的所有發生案例。 |
| %{Variable^} |將整個值轉換成大寫。 |
| %{Variable^Find} | 將指定模式的首個發生案例轉換成大寫。 |
| %{Variable,} | 將整個值轉換成小寫。 |
| %{Variable,Find} | 將指定模式的首個發生案例轉換成小寫。 |

### <a name="find-and-rewrite"></a>尋找並重寫
尋找並取代的變化型態，會在重寫時使用符合指定模式的文字。 下表描述尋找並重寫的語法。

| 語法 | 動作 |
| ------ | ------ |
| %{Variable/=Find/Rewrite} | 尋找，複製並重寫指定模式的所有發生案例。 |
| %{Variable/^Find/Rewrite} | 在指定模式於變數開頭發生時，尋找，複製並重寫該指定模式。 |
| %{Variable/$Find/Rewrite} | 在指定模式於變數結尾發生時，尋找，複製並重寫該指定模式。 |
| %{Variable/Find} | 尋找並刪除指定模式的所有發生案例。 |

重要資訊：

- 透過指定後面接著整數的貨幣符號 (例如 $1) 來展開符合指定模式的文字。

- 可以指定多個模式。 指定模式的順序會決定指派給模式的整數。 在下列範例中，第一個模式符合 "www."，第二個模式符合第二層網域，而第三個模式則符合最上層網域：

    `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$2.$3:80}`

- 重寫的值可以包含任意組合的文字和這些預留位置。

    在上述範例中，主機名稱會重寫為 `cdn.$2.$3:80` (例如，cdn.mydomain.com:80)。

- 模式預留位置 (例如 $1) 的大小寫可以透過下列旗標來修改：
     - U：將展開的值設為大寫。

         範例語法：

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$U2.$3:80}`

     - L：將展開的值設為小寫。

         範例語法：

         `%{host/=^www\.([^\.]+)\.([^\.:]+)/cdn.$L2.$3:80}`

- 必須在模式之前指定運算子。 指定的運算子會決定模式擷取行為：

     - `=`：表示指定模式的所有發生案例都必須被截取並重寫。
     - `^`：表示僅會擷取以指定模式作為開頭的文字。
     - `$`：表示僅會擷取以指定模式作為結尾的文字。
 
- 如果您省略 */Rewrite* 值，系統將會刪除符合模式的文字。


