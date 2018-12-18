---
title: API - Azure 認知服務 | Microsoft Docs
description: 這是 Azure 自訂決策服務之完整且方便使用的 API 指南，其中的雲端式 API 可透過體驗強化內容相關的決策制定。
services: cognitive-services
author: slivkins
manager: slivkins
ms.service: cognitive-services
ms.topic: article
ms.date: 05/11/2018
ms.author: slivkins
ms.reviewer: marcozo, alekh
ms.openlocfilehash: 403b17e33394016a07a7b33ba1bcbfe6afdcc05b
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370222"
---
# <a name="api"></a>API

Azure 自訂決策服務提供每個決策所需的兩個 API：[排名 API](#ranking-api) 可輸入動作的排名，以及[獎勵 API](#reward-api) 可輸出獎勵。 此外，您可以將指定動作的[動作集 API](#action-set-api-customer-provided) 提供給 Azure 自訂決策服務。 本文說明這三種 API。 以下使用一個典型的案例來示範自訂決策服務何時最佳化發行項的排名。

## <a name="ranking-api"></a>排名 API

排名 API 使用標準 [JSONP](https://en.wikipedia.org/wiki/JSONP) 樣式通訊模式來最佳化延遲，並略過[相同來源原則](https://en.wikipedia.org/wiki/Same-origin_policy)。 後者禁止 JavaScript 擷取頁面來源以外的資料。

將此程式碼片段插入您首頁的 HTML 標頭 (發行項的個人化清單顯示位置)：

```html
// define the "callback function" to render UI
<script> function callback(data) { … } </script>
// "data" provides the ranking of URLs, see example below.

// call to Ranking API
<script src="https://ds.microsoft.com/api/v2/<appId>/rank/<actionSetId>?<parameters>" async></script>
// action set id is the name of the corresponding RSS/Atom feed.

// same call with multiple action sets:
// <script src="https://ds.microsoft.com/api/v2/<appId>/rank/<A1>/<A2>/.../<An>?<parameters>" async></script>
```

> [!IMPORTANT]
> 必須在排名 API 呼叫前定義回呼函數。

> [!TIP]
> 若要改善延遲，請透過 HTTP 而不是 HTTPS 公開排名 API，如 `http://ds.microsoft.com/api/v2/<appId>/rank/*` 中所示。
> 不過，若透過 HTTPS 提供首頁，則必須使用 HTTPS 端點。

不使用參數時，來自排名 API 的 HTTP 回應會是 JSONP 格式字串：

```json
callback({
   "ranking":[{"id":"url1"}, {"id":"url2"}, {"id":"url3"}],
   "eventId":"<opaque event string>",
   "appId":"<your app id>",
   "actionSets":[{"id":"<A1>","lastRefresh":"2017-04-29T22:34:25.3401438Z"},
                 {"id":"<A2>","lastRefresh":"2017-04-30T22:34:25.3401438Z"}]});
```

瀏覽器會接著執行此字串作為 `callback()` 函式的呼叫。

上述範例中的回呼函數參數具有下列結構描述：

- `ranking` 會提供要顯示的 URL 排名。
- 自訂決策服務會在內部使用 `eventId` 來比對此排名與對應的點選。
- `appId` 可讓回呼函數區分相同網頁上執行之自訂決策服務的多個應用程式。
- `actionSets` 會列出用於排名 API 呼叫中的每個動作集，以及上次成功重新整理的 UTC 時間戳記。 自訂決策服務會定期重新整理動作集摘要。 例如，若某些動作集不是最新的，回呼函數可能需要切換回其預設排名。

> [!IMPORTANT]
> 指定的動作集已經過處理並可能剪除，來形成發行項的預設排名。 預設排名會接著重新排序並在 HTTP 回應中傳回。 預設排名的定義如下：
>
> - 在每個動作集內，發行項已剪除至 15 個最新的發行項 (若傳回 15 個以上)。
> - 指定多個動作集時，會依其在 API 呼叫中的相同順序來合併。 發行項的原始排序會保留在每個動作集內。 重複項目會遭到移除，以改用較早複本。
> - 前 `n` 個發行項會與發行項的合併清單分開保留，預設為 `n=20`。

### <a name="ranking-api-with-parameters"></a>使用參數的排名 API

排名 API 允許下列參數：

- `details=1` 和 `details=2` 會插入 `ranking` 中所列每個發行項的其他詳細資料。
- `limit=<n>` 會指定預設排名中的最大發行項數。 `n` 必須介於 `2` 和 `30` 之間 (否則會分別截斷至 `2` 或 `30`)。
- `dnt=1` 會停用使用者 Cookie。

參數可結合成標準查詢字串語法，例如 `details=2&dnt=1`。

> [!IMPORTANT]
> 歐洲的預設設定應該是 `dnt=1`，直到客戶同意 Cookie 橫幅為止。 它也應該是目標為未成年人之網站的預設設定。 如需詳細資訊，請參閱[使用規定](https://www.microsoft.com/cognitive-services/en-us/legal/CognitiveServicesTerms20160804)。

`details=1` 項目若由動作集 API 提供，則會插入每個發行項的 `guid`。 HTTP 回應：

```json
callback({
   "ranking":[{"id":"url1","details":[{"guid":"123"}]},
              {"id":"url2","details":[{"guid":"456"}]},
              {"id":"url3","details":[{"guid":"789"}]}],
   "eventId":"<opaque event string>",
   "appId":"<your app id>",
   "actionSets":[{"id":"<A1>","lastRefresh":"timeStamp1"},
                 {"id":"<A2>","lastRefresh":"timeStamp2"}]});
```

`details=2` 項目會新增 自訂決策服務可能從發行項的 SEO 中繼標籤[功能化程式碼](https://github.com/Microsoft/mwt-ds/tree/master/Crawl)擷取的更多詳細資料：

- 來自 `<meta property="og:title" content="..." />`、`<meta property="twitter:title" content="..." />` 或 `<title>...</title>` 的 `title`
- 來自 `<meta property="og:description" ... />`、`<meta property="twitter:description" content="..." />` 或 `<meta property="description" content="..." />` 的 `description`
- 來自 `<meta property="og:image" content="..." />` 的 `image`
- 來自 `<meta name=”microsoft:ds_id” content="..." />` 的 `ds_id`

HTTP 回應：

```json
callback({
   "ranking":[{"id":"url1","details":<details>},
              {"id":"url2","details":<details>},
              {"id":"url3","details":<details>}],
   "eventId":"<opaque event string>",
   "appId":"<your app id>",
   "actionSets":[{"id":"<A1>","lastRefresh":"timeStamp1"},
                 {"id":"<A2>","lastRefresh":"timeStamp2"}]});
```

`<details>` 項目：

```json
[{"guid":"123"}, {"description":"some text", "ds_id":"234", "image":"ImageUrl1", "title":"some text"}]
```

## <a name="reward-api"></a>獎勵 API

自訂決策服務只會使用熱門位置的點選。 每個點選會解譯為獎勵 1。 缺少點選會解譯為獎勵 0。 點選會使用事件識別碼與對應排名進行比對，這是由[排名 API](#ranking-api) 呼叫所產生。 如有需要，事件識別碼可透過工作階段 Cookie 傳遞。

若要處理熱門位置的點選，請將此程式碼放在您的首頁：

```javascript
$.ajax({
    type: "POST",
    url: '//ds.microsoft.com/api/v2/<appId>/reward/' + data.eventId,
    contentType: "application/json" })
```

在此，`data` 是 `callback()` 函式的引數，如上所述。 在點選處理程式碼中使用 `data` 需要稍加注意。 此[教學課程](custom-decision-service-tutorial-news.md#use-the-apis)中顯示一個範例。

獎勵 API 僅供測試之用，可透過 [cURL](https://en.wikipedia.org/wiki/CURL) 呼叫：

```sh
curl -v https://ds.microsoft.com/api/v2/<appId>/reward/<eventId> -X POST -d 1 -H "Content-Type: application/json"
```

預期的效果為 HTTP 回應 200 (確定)。 您可以看到記錄中此事件的獎勵為 1 (若入口網站上已提供 Azure 儲存體帳戶金鑰)。

## <a name="action-set-api-customer-provided"></a>動作集 API (客戶提供)

在高層級，動作集 API 會傳回發行項 (動作) 清單。 每個發行項是由其 URL 及 (選擇性) 發行項標題和發行日期指定。 您可以在入口網站上指定幾個動作集。 每個動作集應該使用不同的動作集 API 作為相異 URL。

每個動作集 API 可透過兩個方式來實作：作為 RSS 摘要或作為 Atom 摘要。 不論是哪一個都應該符合標準並傳回正確的 XML。 以下是 RSS 範例：

```xml
<rss version="2.0">
<channel>
   <item>
      <title><![CDATA[title (possibly with url) ]]></title>
      <link>url</link>
      <guid>guid (could be a your internal database id)</guid>
      <pubDate>date</pubDate>
    </item>
   <item>
       ....
   </item>
</channel>
</rss>
```

每個最上層 `<item>` 項目會描述一個動作：

- `<link>` 是必要的，可作為動作識別碼。
- 若 `<date>` 小於或等於 15 個項目，則會予以忽略；否則是必要的。
  - 若有 15 個以上的項目，則會使用 15 個最新的項目。
  - 它必須是 RSS 或 Atom 的標準格式，分別為：
    - RSS 為 [RFC 822](https://tools.ietf.org/html/rfc822)：例如，`"Fri, 28 Apr 2017 18:02:06 GMT"`
    - Atom 為 [RFC 3339](https://tools.ietf.org/html/rfc3339)：例如，`"2016-12-19T16:39:57-08:00"`
- `<title>` 為選擇性，可用來產生描述發行項的功能。
- `<guid>` 為選擇性，可透過系統傳遞至回呼函數 (若在排名 API 呼叫中指定 `?details` 參數)。

`<item>` 內的其他項目則會予以忽略。

Atom 摘要版本使用相同的 XML 語法和慣例。

> [!TIP]
> 若您的系統使用其本身的發行項識別碼，則可以使用 `<guid>` 傳遞至回呼函數。