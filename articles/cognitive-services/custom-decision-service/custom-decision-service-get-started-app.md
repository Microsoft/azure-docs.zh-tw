---
title: 從應用程式呼叫 API - Azure 認知服務 | Microsoft Docs
description: 如何在從智慧型手機應用程式呼叫 API 時開始使用 Azure 自訂決策服務。
services: cognitive-services
author: slivkins
manager: slivkins
ms.service: cognitive-services
ms.topic: article
ms.date: 05/10/2018
ms.author: slivkins
ms.reviewer: marcozo, alekh
ms.openlocfilehash: 2d02b0deaaa701dd0b4818638827c04e2c946558
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370715"
---
# <a name="call-api-from-an-app"></a>從應用程式呼叫 API

從智慧型手機應用程式呼叫 Azure 自訂決策服務 API。 本文說明如何開始使用一些基本選項。

請務必先[註冊應用程式](custom-decision-service-get-started-register.md)。

您會從智慧型手機應用程式對自訂決策服務進行兩次 API 呼叫：呼叫排名 API 以取得您內容的排名清單，以及呼叫獎勵 API 以報告獎勵。 以下是 [cURL](https://en.wikipedia.org/wiki/CURL) 中的範例呼叫。

如需詳細資訊，請參閱參考 [API](custom-decision-service-api-reference.md)。

從排名 API 呼叫開始。 建立 `<request.json>` 檔案，其中包含動作集識別碼。 此識別碼是您在入口網站上輸入之對應 RSS 或 Atom 摘要的名稱：

```json
{"decisions":
     [{ "actionSets":[{"id":{"id":"<actionSetId>"}}] }]}
```

許多動作集可以指定如下：

```json
{"decisions":
    [{ "actionSets":[{"id":{"id":"<actionSetId1>"}},
                     {"id":{"id":"<actionSetId2>"}}] }]}
```

此 JSON 檔案接著會傳送為排名要求的一部分：

```shell
curl -d @<request.json> -X POST https://ds.microsoft.com/api/v2/<appId>/rank --header "Content-Type: application/json"
```

在這裡，`<appId>` 是入口網站上註冊的應用程式名稱。 您應該會收到可在應用程式中轉譯的一組排序過內容項目。 範例傳回如下：

```json
[{ "ranking":[{"id":"actionId3"}, {"id":"actionId1"}, {"id":"actionId2"}],
   "eventId":"<opaque event string>",
   "appId":"<your app id>",
   "rewardAction":"actionId3",
   "actionSets":[{"id":"<actionSetId1>","lastRefresh":"2017-04-29T22:34:25.3401438Z"},
                 {"id":"<actionSetId2>","lastRefresh":"2017-04-30T22:34:25.3401438Z"}]}]
```

傳回的第一個部分包含以其動作識別碼所指定的已排序動作清單。 針對發行項，動作識別碼是 URL。 整體要求也會有系統建立的唯一 `<eventId>`。

稍後，您可以指定是否從此事件觀察到按一下第一個內容項目，即 `<actionId3>`。 您接著可以使用與下列類似的另一個要求，透過獎勵 API 將此 `<eventId>` 上的獎勵回報給自訂決策服務：

```shell
curl -v https://ds.microsoft.com/api/v2/<appId>/reward/<eventId> -X POST
```

最後，您需要提供動作集 API，以傳回自訂決策服務要考量的發行項 (動作) 清單。 實作此 API 作為 RSS 摘要，如這裡所示：

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

在這裡，每個最上層 `<item>` 項目都會描述一個發行項。 `<link>` 是必要項目，而且自訂決策服務用來作為動作識別碼。 若您的發行項超過 15 個，則請指定 `<date>` (標準 RSS 格式)。 會使用 15 個最新發行項。 `<title>` 為選擇性且用來建立發行項的文字相關功能。

## <a name="next-steps"></a>後續步驟

* 完成[教學課程](custom-decision-service-tutorial-news.md)以了解更深入的範例。
* 請參閱參考 [API](custom-decision-service-api-reference.md) 以深入了解提供的功能。