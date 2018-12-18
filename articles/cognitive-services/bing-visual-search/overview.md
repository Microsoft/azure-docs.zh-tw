---
title: 什麼是 Bing 圖像式搜尋？
titleSuffix: Azure Cognitive Services
description: Bing 圖像式搜尋可提供影像的相關詳細資料或深入解析，例如類似影像或購物來源。
services: cognitive-services
author: swhite-msft
manager: cgronlun
ms.service: cognitive-services
ms.technology: bing-visual-search
ms.topic: overview
ms.date: 04/10/2018
ms.author: scottwhi
ms.openlocfilehash: 2ab73281eb050a52b596504ea9cc0e478144ff24
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47226756"
---
# <a name="what-is-bing-visual-search-api"></a>什麼是 Bing 圖像式搜尋 API？

Bing 圖像式搜尋 API 可提供與 Bing.com/images 上顯示的影像詳細資料類似的體驗。 您可以使用圖像式搜尋上傳圖片，並取得影像的相關深入解析，例如視覺上類似的影像、購物來源、包含影像的網頁等。 您也可以不要上傳影像，而是在影像搜尋結果中提供您從影像中取得的深入解析權杖 (請參閱 [Bing 影像 API](../bing-image-search/overview.md))。

圖像式搜尋可識別名人、紀念碑和地標、藝術品、居家裝飾、款式、產品、字元辨識 (OCR) 等等。

以下是圖像式搜尋可讓您探索的深入解析。

- 外觀上類似的影像 &mdash; 外觀上類似於輸入影像的影像清單
- 外觀上類似的產品 &mdash; 所含產品在外觀上與輸入影像中顯示的產品相似的影像清單
- 購物來源 &mdash; 可購買到輸入影像中所含項目的地點清單
- 相關搜尋 &mdash; 由他人建立或根據影像內容而建立的相關搜尋清單
- 包含影像的網頁 &mdash; 包含輸入影像的網頁清單
- 食譜 &mdash; 一份網頁清單，這些網頁包含用來烹調輸入影像中所含菜餚的食譜

除了這些深入解析以外，圖像式搜尋也會傳回從輸入影像衍生的各種字詞 (標記) 集。 這些標記可讓使用者瀏覽影像中出現的概念。 例如，如果輸入影像顯示的是知名運動員，則其中一個標記可能是運動員的名字，另外也可能有「運動」標記。 或者，如果輸入影像顯示的是蘋果派，則可能會有「蘋果派」、「甜派」、「甜點」等標記，讓使用者能夠瀏覽相關的概念。

圖像式搜尋的結果也會包含影像中相關區域的週框。 例如，如果影像包含數個名人，則結果可能會為影像中可辨識的每個名人加上週框。 或者，如果 Bing 在影像中辨識出某項產品或服飾，則結果可能會為可辨識的產品或服飾項目加上週框。

> [!IMPORTANT]
> 如果您使用 /images/details 端點來[取得影像深入解析](../bing-image-search/image-insights.md)，則應更新程式碼以改為使用圖像式搜尋，因為它可提供更完整的深入解析。


## <a name="the-request"></a>要求

下列選項可用來取得影像的相關深入解析。 

- 將您在先前執行呼叫時從影像中取得的深入解析權杖，傳送至其中一個 [Bing 影像 API](https://docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference) 端點
- 傳送影像的 URL
- 上傳影像 (二進位)


如果您將影像權杖或 URL 傳送至圖像式搜尋，以下會顯示您必須包含在 POST 本文中的 JSON 物件。 

```json
{
    "imageInfo" : {
        "url" : "",
        "imageInsightsToken" : "",
        "cropArea" : {
            "top" : 0.1,
            "left" : 0.5,
            "right" : 0.9,
            "bottom" : 0.9
        }
    },
    "knowledgeRequest" : {
      "filters" : {
        "site" : ""
      }
    }
}
```

`imageInfo` 物件必須包含 `url` 或 `imageInsightsToken` 欄位，但不可兩個欄位都包含。 將 `url` 欄位設為網際網路可存取影像的 URL。 支援的影像大小上限為 1 MB。

`imageInsightsToken` 必須設為深入解析權杖。 若要取得深入解析權杖，請呼叫 Bing 影像 API。 回應會包含 `Image` 物件的清單。 每個 `Image` 物件各有一個 `imageInsightsToken` 欄位，其中包含此權杖。

`cropArea` 是選擇性欄位。 裁剪區域會指定相關區域的頂端、左上角、底部、右上角。 請指定 0.0 到 1.0 之間的值。 此值是整體寬度或高度的百分比。 例如，上述範例將影像的右半邊標示為相關區域。 如果您想要將深入解析要求限定於相關區域，請包含該區域。

`filters` 物件包含一個網站篩選條件 (請參閱 `site` 欄位)，可用來將類似影像和類似產品的結果限定於特定網域。 例如，如果影像顯示的是 Surface Book，則您可以將 `site` 設為 www.microsoft.com。 

如果您想要深入了解影像的本機複本，請以二進位資料的形式上傳該影像。

如需在 POST 的本文中包含這些選項的詳細資訊，請參閱[內容表單類型](#content-form-types)。


### <a name="endpoint"></a>端點

圖像式搜尋端點是：https:\/\/api.cognitive.microsoft.com/bing/v7.0/images/visualsearch。

要求必須以 HTTP POST 要求的形式傳送。 


### <a name="query-parameters"></a>查詢參數

以下是您的要求所應指定的查詢參數。 您至少應該包含 `mkt` 查詢參數。

|名稱|值|類型|必要|  
|----------|-----------|----------|--------------|  
|<a name="cc" />cc|產生結果的國家/地區所具備的 2 字元國碼 (地區碼)。<br /><br /> 若您設定此參數，則您也必須指定 [Accept-Language](#acceptlanguage) 標頭。 Bing 會使用它從語言清單中找到的第一個支援的語言，然後將其與您所指定的國碼 (地區碼) 結合，以決定要傳回結果的市場。 如果語言清單中未包含支援的語言，Bing 會就近尋找支援要求的語言和市場。 或者，它可能會將彙總或預設的市場用於結果，而不指定市場。<br /><br /> 只有在指定了多個語言時，才需要使用此查詢參數和 `Accept-Language` 查詢參數，否則，您應使用 `mkt` 和 `setLang` 查詢參數。<br /><br /> 此參數和 [mkt](#mkt) 查詢參數彼此互斥 &mdash; 請勿同時指定。|字串|否|  
|<a name="mkt" />mkt|產生結果的市場。 <br /><br /> **注意：** 建議您一律指定市場 (如果已知)。 指定市場可協助 Bing 路由傳送要求，並傳回適當的最佳回應。<br /><br /> 此參數和 [cc](#cc) 查詢參數彼此互斥 &mdash; 請勿同時指定。|字串|是|  
|<a name="safesearch" />safeSearch|用來篩選成人內容的篩選條件。 以下是可能的篩選值 (不區分大小寫)。<br /><ul><li>關閉 &mdash; 傳回含有成人文字或影像的網頁。<br /><br/></li><li>中度 &mdash; 傳回含有成人文字、但不含成人影像的網頁。<br /><br/></li><li>嚴格 &mdash; 不傳回含有成人文字或影像的網頁。</li></ul><br /> 預設值為「中度」。<br /><br /> **注意：** 如果要求來自於 Bing 的成人內容原則必須將 `safeSearch` 設為「嚴格」的市場，Bing 將會忽略 `safeSearch` 值並使用「嚴格」。<br/><br/>**注意：** 如果您使用 `site:` 查詢運算子，則無論 `safeSearch` 查詢參數設定為何，回應都有可能包含成人內容。 只有在您了解網站上的內容，而且您的案例支援成人內容的可能性時，才可使用 `site:`。 |字串|否|  
|<a name="setlang" />setLang|用於使用者介面字串的語言。 請使用 ISO 639-1 2 字母語言代碼指定語言。 例如，英文的語言代碼是 EN。 預設值為 EN (英文)。<br /><br /> 語言雖然是選擇性的，但您應一律加以指定。 一般而言，除非使用者想要以不同的語言顯示使用者介面字串，否則您都會將 `setLang` 設定為 `mkt` 所指定的相同語言。<br /><br /> 此參數和 [Accept-Language](#acceptlanguage) 標頭彼此互斥 &mdash; 請勿同時指定。<br /><br /> 使用者介面字串是在使用者介面中作為標籤的字串。 JSON 回應物件中有幾個使用者介面字串。 同樣地，回應物件中 Bing.com 屬性的任何連結都會套用指定的語言。|字串|否| 

### <a name="headers"></a>headers

以下是您的要求所應指定的標頭。 必要的標頭只有 Content-Type 和 Ocp-Apim-Subscription-Key 標頭，但您也應加入 User-Agent、X-MSEdge-ClientID、X-MSEdge-ClientIP 和 X-Search-Location。


|頁首|說明|  
|------------|-----------------|  
|<a name="acceptlanguage" />Accept-Language|選擇性的要求標頭。<br /><br /> 要用於使用者介面字串語言的逗號分隔清單。 清單採用喜好設定的遞減順序。 如需詳細資訊 (包括預期的格式)，請參閱 [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)。<br /><br /> 此標頭和 [setLang](#setlang) 查詢參數彼此互斥 &mdash; 請勿同時指定。<br /><br /> 若您設定此標頭，則您也必須指定 [cc](#cc) 查詢參數。 若要決定要傳回結果的市場，Bing 會使用它從清單中找到的第一個支援的語言，然後將其與 `cc` 參數值結合。 如果清單中未包含支援的語言，Bing 會就近尋找支援要求的語言和市場，或將彙總或預設的市場用於結果。 若要判斷 Bing 所使用的市場，請參閱 BingAPIs-Market 標頭。<br /><br /> 只有在指定了多種語言時，才需要使用此標頭和 `cc` 查詢參數。 否則，請使用 [mkt](#mkt) 和 [setLang](#setlang) 查詢參數。<br /><br /> 使用者介面字串是在使用者介面中作為標籤的字串。 JSON 回應物件中有幾個使用者介面字串。 回應物件中 Bing.com 屬性的任何連結都會套用指定的語言。|  
|<a name="contenttype" />Content-Type|必要的要求標頭。<br /><br />必須設為 multipart/form-data，並且加入界限參數 (例如 multipart/form-data; boundary=\<boundary string\>)。 如需詳細資訊，請參閱[內容表單類型](#content-form-types)。
|<a name="market" />BingAPIs-Market|回應標頭。<br /><br /> 要求所使用的市場。 格式為 \<languageCode\>-\<countryCode\>。 例如：en-US。|  
|<a name="traceid" />BingAPIs-TraceId|回應標頭。<br /><br /> 包含要求詳細資料記錄項目的識別碼。 發生錯誤時，會擷取這個識別碼。 如果您無法判定並解決問題，請將此識別碼與其他資訊一併提供給支援小組。|  
|<a name="subscriptionkey" />Ocp-Apim-Subscription-Key|必要的要求標頭。<br /><br /> 您在[認知服務](https://www.microsoft.com/cognitive-services/)中註冊此服務時收到的訂用帳戶金鑰。|  
|<a name="pragma" />Pragma|選擇性的要求標頭<br /><br /> 根據預設，Bing 會傳回快取的內容 (如果有的話)。 若要防止 Bing 傳回快取的內容，請將 Pragma 標頭設定為 no-cache (例如，Pragma: no-cache)。
|<a name="useragent" />User-Agent|選擇性的要求標頭。<br /><br /> 提出要求的使用者代理程式。 Bing 會利用使用者代理程式為行動使用者提供最佳體驗。 雖然是選擇性的，但我們仍建議您一律指定此標頭。<br /><br /> 「使用者代理程式」應為任何常用瀏覽器所傳送的相同字串。 如需使用者代理程式的相關資訊，請參閱 [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)。<br /><br /> 以下是使用者代理程式字串的範例。<br /><ul><li>Windows Phone&mdash;Mozilla/5.0 (compatible; MSIE 10.0; Windows Phone 8.0; Trident/6.0; IEMobile/10.0; ARM; Touch; NOKIA; Lumia 822)<br /><br /></li><li>Android&mdash;Mozilla/5.0 (Linux; U; Android 2.3.5; en-us; SCH-I500 Build/GINGERBREAD) AppleWebKit/533.1 (KHTML; like Gecko) Version/4.0 Mobile Safari/533.1<br /><br /></li><li>iPhone&mdash;Mozilla/5.0 (iPhone; CPU iPhone OS 6_1 like Mac OS X) AppleWebKit/536.26 (KHTML; like Gecko) Mobile/10B142 iPhone4;1 BingWeb/3.03.1428.20120423<br /><br /></li><li>PC&mdash;Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; Touch; rv:11.0) like Gecko<br /><br /></li><li>iPad&mdash;Mozilla/5.0 (iPad; CPU OS 7_0 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/7.0 Mobile/11A465 Safari/9537.53</li></ul>|
|<a name="clientid" />X-MSEdge-ClientID|選擇性要求和回應標頭。<br /><br /> Bing 使用此標頭在 Bing API 所有呼叫之間為使用者提供一致的行為。 Bing 經常推出新功能和改善項目，且會以用戶端識別碼作為不同測試版指派流量的金鑰。 如果在多個要求中，未讓使用者使用相同的用戶端識別碼，Bing 可能會將使用者指派至多個衝突的測試版。 若指派給多個衝突的測試版，可能會導致使用者體驗不一致。 例如若第二個要求與第一個要求指派的測試版不同，可能會產生意料外的體驗。 此外，Bing 可以使用用戶端識別碼將 Web 結果調整為該用戶端識別碼的搜尋記錄，為使用者提供更豐富的體驗。<br /><br /> Bing 也會使用此標頭分析用戶端識別碼產生的活動，協助改善結果的順位。 相關性改進功能有助於 Bing API 提供更高品質的結果，進而為 API 取用者提供更高的點擊率。<br /><br /> **重要事項：** 雖然是選擇性的，但您仍應將此標頭視為必要項目。 為相同的使用者和裝置組合跨多個要求保存用戶端識別碼，可讓 1) API 取用者獲得一致的使用者體驗，以及 2) 透過 Bing API 更理想的結果品質獲得較高的點擊率。<br /><br /> 以下是適用於此標頭的基本使用規則。<br /><ul><li>在裝置上使用您應用程式的每個使用者必須具有 Bing 產生的唯一用戶端識別碼。<br /><br/>如果您未在要求中包含此標頭，Bing 會產生一個識別碼，並於 X-MSEdge-ClientID 回應標頭傳回該識別碼。 不應在要求中包含此標頭的唯一時機是使用者初次在該裝置上使用您的應用程式時。<br /><br/></li><li>**注意：** 您必須確保此用戶端識別碼無法連結至任何已驗證的使用者帳戶資訊。</li><li>您的應用程式每次為該裝置上的該名使用者提出 Bing API 請求時，均需使用此用戶端識別碼。<br /><br/></li><li>保存用戶端識別碼。 若要在瀏覽器應用程式中保存識別碼，請使用永續性的 HTTP Cookie，以確保在所有工作階段均使用該識別碼。 請勿使用工作階段 Cookie。 若為其他應用程式 (例如行動裝置應用程式)，請使用裝置本身的永久儲存裝置保存識別碼。<br /><br/>使用者下次在該裝置上使用您的應用程式時，會取得您保存的用戶端識別碼。</li></ul><br /> **注意：** Bing 回應不一定會包含此標頭。 如果回應包含此標頭，請擷取用戶端識別碼，並將其用於該裝置上使用者的所有後續 Bing 要求。<br /><br /> **注意：** 如果您包含 X-MSEdge-ClientID，則不得在要求中加入 Cookie。|  
|<a name="clientip" />X-MSEdge-ClientIP|選擇性要求標頭。<br /><br /> 用戶端裝置的 IPv4 或 IPv6 位址。 此 IP 位址可用來探索使用者的位置。 Bing 會使用位置資訊來判斷安全搜尋行為。<br /><br /> **注意：** 雖然是選擇性的，但我們仍建議您一律指定此標頭和 X-Search-Location 標頭。<br /><br /> 請勿混淆位址 (例如，藉由將最後一個八位元變更為 0)。 混淆位址會導致位置不在裝置的實際位置附近，這可能會造成 Bing 產生錯誤結果。|  
|<a name="location" />X-Search-Location|選擇性的要求標頭。<br /><br /> 以分號分隔的索引鍵/值組清單，用以說明用戶端的地理位置。 Bing 會使用位置資訊來判斷安全的搜尋行為，以及傳回相關的區域內容。 請將索引鍵/值組指定為 \<key\>:\<value\>。 以下是您用來指定使用者位置的索引鍵。<br /><br /><ul><li>lat &mdash; 必要項目。 用戶端所在位置的緯度，以度為單位。 緯度必須大於或等於 -90.0 且小於或等於 +90.0。 負數值表示南半球的緯度，正數值表示北半球的緯度。<br /><br /></li><li>long &mdash; 必要項目。 用戶端所在位置的經度，以度為單位。 經度必須大於或等於 -180.0 且小於或等於 +180.0。 負數值表示東半球的經度，正數值表示西半球的經度。<br /><br /></li><li>re &mdash; 必要項目。 以公尺為單位的半徑，指定座標的水平精確度。 請傳遞裝置的位置服務所傳回的值。 常見的值可能是 22m (用於 GPS/Wi-Fi)、380m (用於無線訊號基地台三角網定位)，和 18,000m (用於保留 IP 查閱)。<br /><br /></li><li>ts &mdash; 選擇性。 用戶端位於該位置的 UTC UNIX 時間戳記。 (UNIX 時間戳記是自 1970 年 1 月 1 日起經過的秒數)。<br /><br /></li><li>head &mdash; 選擇性。 用戶端移動的相對走向或方向。 指定 0 到 360 度的的移動方向為，計算相對於正北的順時針偏移角度。 只有在 `sp` 索引鍵為非零值時，才需要指定此索引鍵。<br /><br /></li><li>sp &mdash; 選擇性。 用戶端裝置移動的水平速度，以公尺/秒為單位。<br /><br /></li><li>alt &mdash; 選擇性。 用戶端裝置的海拔高度，以公尺為單位。<br /><br /></li><li>are &mdash; 選擇性。 以公尺為單位的半徑，指定座標的垂直精確度。 只有在指定了 `alt` 索引鍵時，才需要指定此索引鍵。<br /><br /></li></ul> **注意：** 雖然有很多索引鍵是選擇性的，但您提供的資訊愈詳細，位置結果就愈精確。<br /><br /> **注意：** 使用者的地理位置雖然是選擇性的，但建議您一律加以指定。 如果用戶端的 IP 位址未精確反映使用者的實體位置 (例如，如果用戶端使用 VPN)，提供位置就益發重要。 若要獲得最佳結果，您應加入此標頭和 X-MSEdge-ClientIP 標頭，但加入此標頭是最基本的要求。|

> [!NOTE] 
> 請記住，使用條款會要求您遵守所有適用法規，包括使用這些標頭的相關法規。 例如，特定轄區 (例如歐洲) 會要求必須先取得使用者同意，才可在使用者裝置上放置特定追蹤裝置。


<a name="content-form-types" />

### <a name="content-form-types"></a>內容表單類型

每個要求都必須包含 Content-Type 標頭。 標頭必須設為：multipart/form-data; boundary=\<boundary string\>，其中，\<boundary string\> 是唯一且不透明的字串，用以識別表單資料的界限。 例如 boundary=boundary_1234-abcd。


若您將影像權杖或 URL 傳送至圖像式搜尋，以下會顯示您必須包含在 POST 本文中的表單資料。 表單資料必須包含 Content-Disposition 標頭，而且其 `name` 參數必須設定為 "knowledgeRequest"。 如需 `imageInfo` 物件的詳細資料，請參閱[要求](#the-request)。


```
--boundary_1234-abcd
Content-Disposition: form-data; name="knowledgeRequest"

{
    "imageInfo" : {
        "url" : "https://contoso.com/2018/05/fashion/red.jpg"
    }
}

--boundary_1234-abcd--
```

如果您上傳本機影像，以下會顯示您必須包含在 POST 本文中的表單資料。 表單資料必須包含 Content-Disposition 標頭。 其 `name` 參數必須設定為 "image"，而 `filename` 參數可以設定為任何字串。 Content-Type 標頭可設為任何常用的影像 MIME 類型。 表單的內容是影像的二進位檔案。 您可以上傳的影像大小上限為 1 MB。 最大寬度或高度應該是 1,500 像素或更少。


```
--boundary_1234-abcd
Content-Disposition: form-data; name="image"; filename="myimagefile.jpg"
Content-Type: image/jpeg

ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...

--boundary_1234-abcd--
```

以下顯示如何為上傳的影像指定相關區域。

```
--boundary_1234-abcd
Content-Disposition: form-data; name="knowledgeRequest"

{
    "imageInfo" : {
        "cropArea" : {
            "top" : 0.2,
            "left" : 0.3,
            "bottom" : 0.7,
            "right" : 0.6
        }
    }
}

--boundary_1234-abcd
Content-Disposition: form-data; name="image"; filename="image"
Content-Type: image/jpeg


ÿØÿà JFIF ÖÆ68g-¤CWŸþ29ÌÄøÖ‘º«™æ±èuZiÀ)"óÓß°Î= ØJ9á+*G¦...

--boundary_1234-abcd--
```



### <a name="example-request"></a>範例要求

以下顯示傳遞影像權杖和相關區域的完整影像深入解析要求。 您可以從先前對 /images/search 的呼叫中取得深入解析權杖。


```  
POST https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch?mkt=en-us HTTP/1.1  
Content-Type: multipart/form-data; boundary=boundary_1234-abcd
Ocp-Apim-Subscription-Key: 123456789ABCDE  
X-MSEdge-ClientIP: 999.999.999.999  
X-Search-Location: lat:47.60357;long:-122.3295;re:100  
X-MSEdge-ClientID: <blobFromPriorResponseGoesHere>  
Host: api.cognitive.microsoft.com 

--boundary_1234-abcd
Content-Disposition: form-data; name="knowledgeRequest"

{
    "imageInfo" : {
        "imageInsightsToken" : "mid_D6426898706EC7..."
        "cropArea" : {
            "top" : 0.1,
            "left" : 0.2,
            "bottom" : 0.7,
            "right" : 0.5
        }
    }
}

--boundary_1234-abcd--
```


## <a name="the-response"></a>回應

如果影像有可用的深入解析，則回應會有一或多個包含深入解析的 `tags`。 `image` 欄位會包含輸入影像的深入解析權杖。

```json
{
  "_type" : "ImageKnowledge",
  "tags" : [
    {...},
    {...},
    {...},
    {...},
    {...}
  ],
  "image" : {
    "imageInsightsToken" : "bcid_AF8C9CA409421B..."
  }
}
```

`tags` 欄位會包含顯示名稱和動作 (深入解析) 的清單。 其中一個標記會包含設為空字串的 `displayName` 欄位。 此標記包含預設的深入解析，例如包含影像、外觀上類似的影像，以及可購買到影像中所含項目之購物地點的網頁。 由於整個影像都是相關區域，因此預設深入解析標記不會為相關區域加上週框。


```json
{
  "_type" : "ImageKnowledge",
  "tags" : [
    {
      "displayName" : "",
      "actions" : [
        {...},
        {...},
        {...},
        {...}
      ]
    },
    {...},
    {...},
    {...},
    {...}
  ],
  "image" : {
    "imageInsightsToken" : "bcid_AF8C9CA409421B..."
  }
}
```

如需預設深入解析的清單，請參閱[預設深入解析](./default-insights-tag.md)。



其餘標記會包含使用者可能感興趣的其他深入解析。 例如，如果影像中包含文字，則其中一個標記可能會有 TextResults 深入解析，其中包含已辨識的文字。 或者，如果 Bing 在影像中辨識出某個實體 (人物、地點或物品)，則會有識別該實體的標記。 圖像式搜尋也會傳回從輸入影像衍生的各種字詞 (標記) 集。 這些標記可讓使用者瀏覽影像中出現的概念。 例如，如果輸入影像顯示的是知名運動員，則其中一個標記可能是「運動」，其中包含運動影像的連結。

每個標記都包含您可用來分類深入解析的顯示名稱、可對適用深入解析的相關區域進行識別的週框、深入解析本身，以及影像的縮圖。 例如，如果影像顯示的是穿著運動服的人，則其中一個標記可能會包含週框，用以界定運動服，以及包含 VisualSearch 和 ProductVisualSearch 深入解析。 而其他標記可能會有 ImageResults 深入解析，其中包含/影像/搜尋 API 要求的 URL，用以取得在主題上具有相關性的影像，或是包含 Bing.com 搜尋 URL 以將使用者導向至 Bing.com 影像搜尋結果。

預設深入解析標記以外的所有標記都會包含週框，用以識別影像中的相關區域。 例如，如果影像包含多個可辨識的人，標記可能會為每個人加上週框，或者，如果影像包含可辨識的服飾項目，則標記可能會為每個可辨識的服飾項目加上週框。 您可以使用週框在影像上建立作用點，而在點選這些作用點時提供與該影像區域中的內容有關的詳細資料。 您不應在影像中為識別整個影像的週框加上作用點。

### <a name="text-recognition"></a>文字辨識

如果影像中包含服務可辨識的文字，則其中一個標記將會包含 TextResults 深入解析 (動作)。 該深入解析的 `displayName` 會包含已辨識的文字。 

```json
    {
        "image" : {
            "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=%23%23Text..."
        },
        "displayName" : "##TextRecognition",
        "boundingBox" : {
            "queryRectangle" : {
                "topLeft" : {"x" : 0, "y" : 0},
                "topRight" : {"x" : 1, "y" : 0},
                "bottomRight" : {"x" : 1, "y" : 1},
                "bottomLeft" : {"x" : 0, "y" : 1}
            },
            "displayRectangle" : {
                "topLeft" : {"x" : 0, "y" : 0},
                "topRight" : {"x" : 1, "y" : 0},
                "bottomRight" : {"x" : 1, "y" : 1},
                "bottomLeft" : {"x" : 0, "y" : 1}
            }
        },
        "actions" : [{
            "displayName" : "WALK BIKE ACROSS BRIDGE",
            "actionType" : "TextResults"
        }],
        "sources" : ["OCR"]
    }
```

標記的 `displayName` 欄位包含 ##TextRecognition，因此請勿將其當作 UX 中的類別標題使用。 這樣會產生任何以 ## 開頭的顯示名稱。 此時請改用動作的顯示名稱。


文字辨識也可以辨識名片上的連絡資訊，例如電話號碼和電子郵件地址。 週框會識別連絡資訊在名片上的位置。 

```json
    {
      "image" : {
        "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=%23%23TextRecognition..."
      },
      "displayName" : "##TextRecognition",
      "boundingBox" : {
        "queryRectangle" : {
          "topLeft" : {"x" : 0.635, "y" : 0},
          "topRight" : {"x" : 0.77, "y" : 0},
          "bottomRight" : {"x" : 0.77, "y" : 0.4873333},
          "bottomLeft" : {"x" : 0.635, "y" : 0.4873333}
        },
        "displayRectangle" : {
          "topLeft" : {"x" : 0.635, "y" : 0},
          "topRight" : {"x" : 0.77, "y" : 0},
          "bottomRight" : {"x" : 0.77, "y" : 0.4873333},
          "bottomLeft" : {"x" : 0.635, "y" : 0.4873333}
        }
      },
      "actions" : [
        {
          "url" : "tel:888%20555%201212",
          "actionType" : "Uri"
        }
      ],
      "sources" : ["OCR"]
    },
    {
      "image" : {
        "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=%23%23TextRecognition..."
      },
      "displayName" : "##TextRecognition",
      "boundingBox" : {
        "queryRectangle" : {
          "topLeft" : {"x" : 0.63, "y" : 0},
          "topRight" : {"x" : 0.866, "y" : 0},
          "bottomRight" : {"x" : 0.866, "y" : 0.5553334},
          "bottomLeft" : {"x" : 0.63, "y" : 0.5553334}
        },
        "displayRectangle" : {
          "topLeft" : {"x" : 0.63, "y" : 0},
          "topRight" : {"x" : 0.866, "y" : 0},
          "bottomRight" : {"x" : 0.866, "y" : 0.5553334},
          "bottomLeft" : {"x" : 0.63, "y" : 0.5553334}
        }
      },
      "actions" : [
        {
          "url" : "mailto:someone@outlook.com",
          "actionType" : "Uri"
        }
      ],
      "sources" : ["OCR"]
    },
    {
      "image" : {
        "thumbnailUrl" : "https:\/\/tse3.mm.bing.net\/th?q=%23%23TextRecognition..."
      },
      "displayName" : "##TextRecognition",
      "boundingBox" : {
        "queryRectangle" : {
          "topLeft" : {"x" : 0, "y" : 0},
          "topRight" : {"x" : 1, "y" : 0},
          "bottomRight" : {"x" : 1, "y" : 1},
          "bottomLeft" : {"x" : 0, "y" : 1}
        },
        "displayRectangle" : {
          "topLeft" : {"x" : 0, "y" : 0},
          "topRight" : {"x" : 1, "y" : 0},
          "bottomRight" : {"x" : 1, "y" : 1},
          "bottomLeft" : {"x" : 0, "y" : 1}
        }
      },
      "actions" : [
        {
          "displayName" : "CHARLENE WHITNEY Graphic Designer 888 555 1212 someone@outlook.com www.contoso.com",
          "actionType" : "TextResults"
        }
      ],
      "sources" : ["OCR"]
    }
```

如果影像包含可辨識的實體，例如人物、地點或物品，則其中一個標記可能會包含「實體」深入解析。 

```json
    {
      "image" : {
        "thumbnailUrl" : "https:\/\/tse4.mm.bing.net\/th?q=Statue+of+Liberty..."
      },
      "displayName" : "Statue of Liberty",
      "boundingBox" : {
        "queryRectangle" : {
          "topLeft" : {"x" : 0.40625, "y" : 0.1757813},
          "topRight" : {"x" : 0.6171875, "y" : 0.1757813},
          "bottomRight" : {"x" : 0.6171875, "y" : 0.3867188},
          "bottomLeft" : {"x" : 0.40625, "y" : 0.3867188}
        },
        "displayRectangle" : {
          "topLeft" : {"x" : 0.40625, "y" : 0.1757813},
          "topRight" : {"x" : 0.6171875, "y" : 0.1757813},
          "bottomRight" : {"x" : 0.6171875, "y" : 0.3867188},
          "bottomLeft" : {"x" : 0.40625, "y" : 0.3867188}
        }
      },
      "actions" : [
        {
          "_type" : "ImageEntityAction",
          "webSearchUrl" : "https:\/\/www.bing.com\/search?q=Statue+of+Liberty",
          "displayName" : "Statue of Liberty",
          "actionType" : "Entity",
        }
      ]
    }
```



## <a name="next-steps"></a>後續步驟

若要快速開始使用您的第一個要求，請參閱 [C#](quickstarts/csharp.md) | [JAVA](quickstarts/java.md) | [node.js](quickstarts/nodejs.md) | [Python](quickstarts/python.md)。

試用 API。 移至[圖像式搜尋 API 測試主控台](https://dev.cognitive.microsoft.com/docs/services/878c38e705b84442845e22c7bff8c9ac)。 


熟悉[圖像式搜尋 API](https://aka.ms/bingvisualsearchreferencedoc) 參考。 此參考包含端點、標頭，以及您用來要求搜尋結果的查詢參數清單。 它也包含回應物件的定義。 

務必閱讀 [Bing 使用和顯示需求](./use-and-display-requirements.md)，您才不會違反任何有關使用搜尋結果的規則。


