---
title: 快速入門：發佈知識庫 - REST、Node.js - QnA Maker
titleSuffix: Azure Cognitive Services
description: 本快速入門會逐步引導您以程式設計方式發佈知識庫 (KB)。 發佈作業會將最新版的知識庫推送到專用 Azure 搜尋服務索引，並建立端點以供應用程式或聊天機器人呼叫。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 10/02/2018
ms.author: diberry
ms.openlocfilehash: c70b90a6e465c72193f63afd7ab9106579e2c634
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48886605"
---
# <a name="quickstart-publish-a-qna-maker-knowledge-base-in-nodejs"></a>快速入門：以 Node.js 發佈 QnA Maker 知識庫

本快速入門會逐步引導您以程式設計方式發佈知識庫 (KB)。 發佈作業會將最新版的知識庫推送到專用 Azure 搜尋服務索引，並建立端點以供應用程式或聊天機器人呼叫。

本快速入門會呼叫 QnA Maker API：
* [發佈](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75fe) - 此 API 不需要在要求主體中有任何資訊。

## <a name="prerequisites"></a>必要條件

* [Node.js 6+](https://nodejs.org/en/download/)
* 您必須有 [QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)。 若要擷取您的金鑰，請選取儀表板中 [資源管理] 下方的 [金鑰]。 
* QnA Maker 知識庫 (KB) 識別碼可以在 kbid 查詢字串參數中的 URL 找到，如下所示。

    ![QnA Maker 知識庫識別碼](../media/qnamaker-quickstart-kb/qna-maker-id.png)

如果您還沒有知識庫，可以建立要用於本快速入門的範例知識庫：[建立新的知識庫](create-new-kb-nodejs.md)。

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-nodejs-repo-note.md)]

## <a name="create-a-knowledge-base-nodejs-file"></a>建立知識庫 Node.js 檔案

建立名為 `publish-knowledge-base.js` 的檔案。

## <a name="add-required-dependencies"></a>新增必要的相依性

在 `publish-knowledge-base.js` 的頂端，新增下列幾行以將必要的相依性新增至專案：

[!code-nodejs[Add the dependencies](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=1-4 "Add the dependencies")]

## <a name="add-required-constants"></a>新增必要的常數

在上述必要的相依性後面，新增必要的常數以存取 QnA Maker。 以您自己的 QnA Maker 金鑰取代 `subscriptionKey` 變數的值。 

[!code-nodejs[Add required constants](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=10-17 "Add required constants")]

## <a name="add-knowledge-base-id"></a>新增知識庫識別碼

在先前的常數後面，新增知識庫識別碼並且將其新增至路徑：

[!code-nodejs[Add knowledge base ID](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=19-23 "Add knowledge base ID")]

## <a name="add-supporting-functions"></a>新增支援的函式

接下來，新增下列支援的函式。

1. 新增下列函式，以可讀取的格式列印出 JSON：

   [!code-nodejs[Add supporting functions, step 1](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=25-28 "Add supporting functions, step 1")]

2. 新增下列函式，來管理要取得建立作業狀態的 HTTP 回應：

   [!code-nodejs[Add supporting functions, step 2](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=30-52 "Add supporting functions, step 2")]

## <a name="add-the-publishkb-function-and-main-function"></a>新增 publish_kb 函式和 main 函式

下列程式碼會對 QnA Maker API 提出 HTTPS 要求，以發佈 KB 並接收回應：

[!code-nodejs[Add POST request to publish KB](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=54-71 "Add POST request to publish KB")]

[!code-nodejs[Add the publish_kb function](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=73-91 "Add the publish_kb function and main function")]

## <a name="add-the-main-function"></a>新增 main 函式

新增下列函式以管理要求和回應：

[!code-nodejs[Add the main function](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/publish-knowledge-base/publish-knowledge-base.js?range=94-97 "Add the main function")]

## <a name="run-the-program"></a>執行程式

建置並執行程式。 程式會自動將要求傳送至 QnA Maker API 以發佈 KB，然後在主控台視窗中輸出回應。

發佈知識庫之後，您就可以從具有用戶端應用程式或聊天機器人的端點來加以查詢。 

```bash
node publish-knowledge-base.js
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)