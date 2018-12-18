---
title: 快速入門：建立知識庫 - REST、Node.js - QnA Maker
description: 本快速入門會逐步引導您，以程式設計方式建立範例 QnA Maker 知識庫，該知識庫會出現在認知服務 API 帳戶的 Azure 儀表板中。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 10/02/2018
ms.author: diberry
ms.openlocfilehash: f0375affa547f657ae36de71901298047359cae2
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48884911"
---
# <a name="quickstart-create-a-qna-maker-knowledge-base-in-nodejs"></a>快速入門：以 Node.js 建立 QnA Maker 知識庫

本快速入門會逐步引導您以程式設計方式建立 QnA Maker 知識庫範例。 QnA Maker 會從[資料來源](../Concepts/data-sources-supported.md)中的半結構化內容 (如常見問題集) 自動擷取問題和答案。 知識庫的模型是在 JSON (在 API 要求的本體中傳送) 中定義的。 

本快速入門會呼叫 QnA Maker API：
* [建立 KB](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)
* [取得作業詳細資料](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails)

## <a name="prerequisites"></a>必要條件

* [Node.js 6+](https://nodejs.org/en/download/)
* 您必須有 [QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)。 若要擷取您的金鑰，請選取儀表板中 [資源管理] 下方的 [金鑰]。 

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-nodejs-repo-note.md)]

## <a name="create-a-knowledge-base-nodejs-file"></a>建立知識庫 Node.js 檔案

建立名為 `create-new-knowledge-base.js` 的檔案。

## <a name="add-the-required-dependencies"></a>新增必要的相依性

在 `create-new-knowledge-base.js` 的頂端，新增下列幾行以將必要的相依性新增至專案：

[!code-nodejs[Add the dependencies](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=1-4 "Add the dependencies")]

## <a name="add-the-required-constants"></a>新增必要的常數
在上述必要的相依性後面，新增必要的常數以存取 QnA Maker。 以您自己的 QnA Maker 金鑰取代 `subscriptionKey` 變數的值。

[!code-nodejs[Add the required constants](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=10-19 "Add the required constants")]

## <a name="add-the-kb-model-definition"></a>新增 KB 模型定義

在常數後面新增下列 KB 模型定義。 模型會在定義之後轉換成字串。

[!code-nodejs[Add the KB model definition](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=21-51 "Add the KB model definition")]

## <a name="add-supporting-functions"></a>新增支援的函式

接下來，新增下列支援的函式。

1. 新增下列函式，以可讀取的格式列印出 JSON：

   [!code-nodejs[Add supporting functions, step 1](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=53-56 "Add supporting functions, step 1")]

2. 新增下列函式以管理 HTTP 回應：

   [!code-nodejs[Add supporting functions, step 2](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=58-80 "Add supporting functions, step 2")]

## <a name="add-functions-to-create-kb"></a>新增函式以建立 KB

新增下列函式，來提出要建立知識庫的 HTTP POST 要求。 `Ocp-Apim-Subscription-Key` 是 QnA Maker 服務金鑰，用於驗證。 

[!code-nodejs[POST Request to API](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=82-109 "POST Request to API")]

此 API 呼叫會傳回包含作業識別碼的 JSON 回應。 使用作業識別碼來判斷是否已成功建立 KB。 

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:19:01Z",
  "lastActionTimestamp": "2018-09-26T05:19:01Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "8dfb6a82-ae58-4bcb-95b7-d1239ae25681"
}
```

## <a name="add-functions-to-determine-creation-status"></a>新增函式以判斷建立狀態

新增下列函式來提出 HTTP GET 要求，以檢查作業狀態。 `Ocp-Apim-Subscription-Key` 是 QnA Maker 服務金鑰，用於驗證。 

[!code-nodejs[GET Request to API](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=111-135 "GET Request to API")]

重複呼叫直到成功或失敗： 

```JSON
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:23:08Z",
  "resourceLocation": "/knowledgebases/XXX7892b-10cf-47e2-a3ae-e40683adb714",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

## <a name="add-create-kb-function"></a>新增 create-kb 函式

下列函式是主要函式，並且會建立 KB 並重複檢查狀態。 因為 KB 建立可能需要一些時間，您必須重複呼叫來檢查狀態，直到狀態為成功或失敗。

[!code-nodejs[Add create-kb function](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base.js?range=137-167 "Add create-kb function")]

## <a name="run-the-program"></a>執行程式

在命令列中輸入下列命令，以執行程式。 程式會將要求傳送至 QnA Maker API 以建立 KB，然後會每隔 30 秒輪詢結果。 每個回應都會列印到主控台視窗。

```bash
node create-new-knowledge-base.js
```

一旦建立您的知識庫，您可以在 QnA Maker 入口網站的[我的知識庫](https://www.qnamaker.ai/Home/MyServices) (英文) 頁面中檢視。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)
