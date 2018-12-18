---
title: 從瀏覽器呼叫 API - Azure 認知服務 | Microsoft Docs
description: 如何開始使用 Azure 自訂決策服務，藉由直接從瀏覽器進行 API 呼叫來最佳化網頁。
services: cognitive-services
author: slivkins
manager: slivkins
ms.service: cognitive-services
ms.topic: article
ms.date: 05/09/2018
ms.author: slivkins,marcozo,alekh
ms.openlocfilehash: 10236c9d8f70d9b90a896464b4f86a847ee904c2
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370086"
---
# <a name="call-api-from-a-browser"></a>從瀏覽器呼叫 API

本文可協助您直接從瀏覽器對 Azure 自訂決策服務 API 進行呼叫。

請務必先[註冊應用程式](custom-decision-service-get-started-register.md)。

現在就開始吧。 您的應用程式會模型化為有一個標題頁面，並連結到數個文章頁面。 標題頁面使用自訂決策服務來指定其文章頁面的順序。 請將下列程式碼插入至首頁的 HTML 開頭：

```html
// Define the "callback function" to render UI
<script> function callback(data) { … } </script>

// call Ranking API, after callback() is defined
<script src="https://ds.microsoft.com/api/v2/<appId>/rank/<actionSetId>" async></script>
```

`data` 引數包含要呈現的 URL 順位。 如需詳細資訊，請參閱 [API](custom-decision-service-api-reference.md) 參考。

若要處理最上層文章的使用者點選，請在標題頁面中呼叫下列程式碼：

```javascript
// call Reward API to report a click
$.ajax({
    type: "POST",
    url: '//ds.microsoft.com/api/v2/<appId>/reward/' + data.eventId,
    contentType: "application/json" })
```

在這裡，`data` 是 `callback()` 函式的引數。 若需實作範例，請參閱本[教學課程](custom-decision-service-tutorial-news.md#use-the-apis)。

最後，您需要提供動作集 API，以傳回自訂決策服務要考量的文章 (動作) 清單。 實作此 API 作為 RSS 摘要，如這裡所示：

```xml
<rss version="2.0">
<channel>
   <item>
      <title><![CDATA[title (possibly with url) ]]></title>
      <link>url</link>
      <pubDate>Thu, 27 Apr 2017 16:30:52 GMT</pubDate>
    </item>
   <item>
       ....
   </item>
</channel>
</rss>
```

在這裡，每個最上層 `<item>` 項目都會描述一篇文章。 `<link>` 是必要項目，而且自訂決策服務用來作為動作識別碼。 如果您的文章超過 15 篇，則請指定 `<date>` (標準 RSS 格式)。 會使用 15 篇最近的文章。 `<title>` 是選擇性項目，而且用來建立文章的文字相關功能。

## <a name="next-steps"></a>後續步驟

* 完成[教學課程](custom-decision-service-tutorial-news.md)以了解更深入的範例。
* 請參閱參考 [API](custom-decision-service-api-reference.md) 以深入了解提供的功能。