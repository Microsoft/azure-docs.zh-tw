---
title: 教學課程：文章個人化 - 自訂決策服務
titlesuffix: Azure Cognitive Services
description: 內容相關決策制訂的文章個人化教學課程。
services: cognitive-services
author: slivkins
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-decision-service
ms.topic: tutorial
ms.date: 05/08/2018
ms.author: slivkins
ms.openlocfilehash: b142fe2051c017d0c0ec3c4cac6aaedd563f6cd7
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46366330"
---
# <a name="tutorial-article-personalization-for-contextual-decision-making"></a>教學課程：內容相關決策制訂的文章個人化教學課程

本教學課程著重於網站首頁上精選文章的個人化。 例如，自訂決策服務會影響首頁上多個清單。 假設該頁面是僅提供政治和體育消息的新聞網站， 網站上會顯示三種等級的文章清單：政治、體育和最新消息。

## <a name="applications-and-action-sets"></a>應用程式和動作集合

以下是如何讓您的情況符合架構。 假設有三個應用程式，每個均會列出經過最佳化的清單：app-politics、app-sports、app-recent。 若要為每個應用程式指定候選文章，有兩個動作集合：一個用於政治，另一個用於體育。 app-recent 的動作集合會自動成為其他兩組的集合聯集。

> [!TIP]
> 自訂決策服務的應用程式之間可共用動作集合。

## <a name="prepare-action-set-feeds"></a>準備動作集合摘要

自訂決策服務會透過客戶提供的 RSS 或 Atom 摘要取用動作集合。 您提供兩個摘要：一個適用於政治，另一個適用於體育。 假設皆由 `http://www.domain.com/feeds/<feed-name>` 提供。

每個摘要會提供一份文章清單。 在 RSS 中，每個摘要皆由 `<item>` 元素指定，如下所示：

```xml
<rss version="2.0"><channel>
   <item>
      <title><![CDATA[article title]]></title>
      <link>"article url"</link>
      <pubDate>publication date</pubDate>
    </item>
</channel></rss>
```

文章的順序很重要。 該元素可指定預設順位，盡可能猜測文章最佳的排序方式。 接著，會在[儀表板](#performance-dashboard)上使用預設順位來比較效果。

如需摘要格式的更多資訊，請參閱 [API 參考](custom-decision-service-api-reference.md#action-set-api-customer-provided)。

## <a name="register-a-new-app"></a>註冊新的應用程式

1. 使用您的 [Microsoft 帳戶](https://account.microsoft.com/account)登入。 在功能區中，按一下 [我的入口網站]。

2. 若要註冊新的應用程式，請按一下 [新應用程式] 按鈕。

    ![自訂決策服務入口網站](./media/custom-decision-service-tutorial/portal.png)

3. 在**應用程式識別碼**文字方塊中輸入應用程式的唯一名稱。 如果此名稱已由另一位客戶使用中，系統會要求您選取其他應用程式識別碼。 選取 [進階] 核取方塊，然後為您的 Azure 儲存體帳戶輸入 [[連接字串]](../../storage/common/storage-configure-connection-string.md)。 通常，您會為所有應用程式使用相同的儲存體帳戶。

    ![新應用程式對話方塊](./media/custom-decision-service-tutorial/new-app-dialog.png)

    在上述情況下註冊所有三個應用程式後，它們會列出：

    ![應用程式清單](./media/custom-decision-service-tutorial/apps.png)

    您可以按一下 [應用程式] 按鈕以返回此清單。

4. 在 [新應用程式] 對話方塊中，指定動作摘要。 您也可以按一下 [摘要] 按鈕，然後按一下 [新摘要] 按鈕，以指定動作摘要。 輸入新摘要的 [名稱]，然後輸入提供該摘要的 [URL]，並輸入 [重新整理時間]。 重新整理時間可指定自訂決策服務應該重新整理摘要的頻率。

    ![新摘要對話方塊](./media/custom-decision-service-tutorial/new-feed-dialog.png)

    無論是在何處指定動作摘要，這些動作摘要皆可供任何應用程式使用。 在案例中指定兩個動作摘要後，即會列出：

    ![摘要清單](./media/custom-decision-service-tutorial/feeds.png)

    您可以按一下 [摘要] 按鈕以返回此清單。

## <a name="use-the-apis"></a>使用 API

自訂決策服務會透排名 API 來排列文章順位。 若要使用此 API，請將下列程式碼插入首頁的 HTML 開頭：

```html
<!-- Define the "callback function" to render UI -->
<script> function callback(data) { … } </script>

<!--  call Ranking API after callback() is defined, separately for each app -->
<script src="https://ds.microsoft.com/api/v2/app-politics/rank/feed-politics" async></script>
<script src="https://ds.microsoft.com/api/v2/app-sports/rank/feed-sports" async></script>
<script src="https://ds.microsoft.com/api/v2/app-recent/rank/feed-politics/feed-sports" async></script>
<!-- NB: action feeds for 'app-recent' are listed one after another. -->
```

來自排名 API 的 HTTP 回應會是 JSONP 格式字串。 對於 app-politics，例如，字串如下所示：

```json
callback({
   "ranking":[{"id":"url1"}, {"id":"url2"}, {"id":"url3"}],
   "eventId":"<opaque event string>",
   "appId":"app-politics",
   "actionSets":[{"id":"feed-politics","lastRefresh":"date"}] });
```

瀏覽器會接著執行此字串作為 `callback()` 函式的呼叫。 `callback()` 函式中的 `data` 引數包含應用程式識別碼，以及要呈現的 URL 順位。 特別是，`callback()` 應會使用 `data.appId` 區別這三個應用程式。 自訂決策服務會在內部使用 `eventId` 來比對提供的順位與對應的點選 (若有)。

> [!TIP]
> `callback()` 可能會藉由使用 `lastRefresh` 欄位檢查每個動作摘要的時效性。 如果提供的摘要不夠新，則 `callback()` 可能會忽略提供的順位，直接呼叫此摘要，並使用由摘要提供的預設順位。

如需更多規格和排名 API 提供之其他選項的相關資訊，請參閱 [API 參考](custom-decision-service-api-reference.md)。

呼叫獎勵 API，可傳回使用者的最優先的文章選擇。 收到優先文章選擇之後，會在首頁叫用下列程式碼：

```javascript
$.ajax({
    type: "POST",
    url: '//ds.microsoft.com/api/v2/<appId>/reward/<eventId>',
    contentType: "application/json" })
```

在點擊處理程式碼中使用 `appId` 和 `eventId` 需要特別小心。 例如，您可以實作 `callback()` 函數，如下所示：

```javascript
function callback(data) {
    $.map(data.ranking, function (element) {
        //custom rendering function given content info
        render({appId:data.appId,
                article:element,
                onClick: element.id != data.rewardAction ? null :
                   function() {
                       $.ajax({
                       type: "POST",
                       url: '//ds.microsoft.com/api/v2/' + data.appId + '/reward/' + data.eventId,
                       contentType: "application/json" })}
                });
}}
```

在此範例中，請實作 `render()` 函數，以針對指定的應用程式轉譯指定的文章。 此函數會輸入應用程式識別碼和文章 (使用排名 API 的格式)。 `onClick` 參數是應從 `render()` 呼叫處理點擊的函數。 它會檢查該點擊是否位於最上方的位置。 然後使用適當的應用程式識別碼和事件識別碼呼叫獎勵 API。

## <a name="next-steps"></a>後續步驟

* 請參閱 [API 參考](custom-decision-service-api-reference.md)以深入了解提供的功能。