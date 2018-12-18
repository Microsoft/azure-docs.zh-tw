---
title: 快速入門：更新知識庫 - REST，Node.js - QnA Maker
titleSuffix: Azure Cognitive Services
description: 本快速入門會以程式設計方式逐步引導您更新現有的 QnA Maker 知識庫 (KB)。  此 JSON 可讓您藉由新增資料來源、變更資料來源或刪除資料來源來更新 KB。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 10/02/2018
ms.author: diberry
ms.openlocfilehash: 3bbc55b3bb064b2cf4b140a395e99209b71a5ce1
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816205"
---
# <a name="quickstart-update-a-qna-maker-knowledge-base-in-nodejs"></a>快速入門：以 Node.js 更新 QnA Maker 知識庫

本快速入門會以程式設計方式逐步引導您更新現有的 QnA Maker 知識庫 (KB)。  此 JSON 可讓您藉由新增資料來源、變更資料來源或刪除資料來源來更新 KB。

此 API 就相當於編輯，然後使用 QnA Maker 入口網站中的 [儲存並定型] 按鈕。

本快速入門會呼叫 QnA Maker API：
* [更新](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) - 知識庫的模型是在 JSON (在 API 要求的本體中傳送) 中定義的。 
* [取得作業詳細資料](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/operations_getoperationdetails)

## <a name="prerequisites"></a>必要條件

* [Node.js 6+](https://nodejs.org/en/download/)
* 您必須有 [QnA Maker 服務](../How-To/set-up-qnamaker-service-azure.md)。 若要擷取您的金鑰，請選取儀表板中 [資源管理] 下方的 [金鑰]。 
* QnA Maker 知識庫 (KB) 識別碼可以在 kbid 查詢字串參數中的 URL 找到，如下所示。

    ![QnA Maker 知識庫識別碼](../media/qnamaker-quickstart-kb/qna-maker-id.png)

如果您還沒有知識庫，可以建立要用於本快速入門的範例知識庫：[建立新的知識庫](create-new-kb-nodejs.md)。

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-nodejs-repo-note.md)]

## <a name="create-a-knowledge-base-nodejs-file"></a>建立知識庫 Node.js 檔案

建立名為 `update-knowledge-base.js` 的檔案。

## <a name="add-the-required-dependencies"></a>新增必要的相依性

在 `update-knowledge-base.js` 的頂端，新增下列幾行以將必要的相依性新增至專案：

[!code-nodejs[Add the dependencies](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=1-4 "Add the dependencies")]

## <a name="add-required-constants"></a>新增必要的常數
在上述必要的相依性後面，新增必要的常數以存取 QnA Maker。 以您自己的 QnA Maker 金鑰取代 `subscriptionKey` 變數的值。 

[!code-nodejs[Add required constants](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=10-17 "Add required constants")]

## <a name="add-knowledge-base-id"></a>新增知識庫識別碼

在先前的常數後面，新增知識庫識別碼並且將其新增至路徑：

[!code-nodejs[Add knowledge base ID](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=19-23 "Add knowledge base ID")]

## <a name="add-the-kb-update-model-definition"></a>新增 KB 更新模型定義

在常數後面新增下列 KB 更新定義。 更新定義有三個區段：

* 新增
* update
* delete

每個區段都可在對 API 發出的相同單一要求中使用。 

[!code-nodejs[Add the KB update definition](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=25-53 "Add the KB update definition")]


## <a name="add-supporting-functions"></a>新增支援的函式

接下來，新增下列支援的函式。

1. 新增下列函式，以可讀取的格式列印出 JSON：

   [!code-nodejs[Add supporting functions, step 1](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=55-58 "Add supporting functions, step 1")]

2. 新增下列函式，來管理要取得建立作業狀態的 HTTP 回應：

   [!code-nodejs[Add supporting functions, step 2](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=60-82 "Add supporting functions, step 2")]

## <a name="add-patch-request-to-update-kb"></a>新增 PATCH 要求來更新 KB

新增下列函式，以提出要更新知識庫的 HTTP PATCH 要求。 `Ocp-Apim-Subscription-Key` 是 QnA Maker 服務金鑰，用於驗證。

[!code-nodejs[Add PATCH request to update KB](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=84-111 "Add PATCH request to update KB")]

此 API 呼叫會傳回包含作業識別碼的 JSON 回應。 如果作業未完成，則需要作業識別碼才能要求狀態。

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-10-02T01:23:00Z",
  "lastActionTimestamp": "2018-10-02T01:23:00Z",
  "userId": "335c3841df0b42cdb00f53a49d51a89c",
  "operationId": "e7be3897-88ff-44e5-a06c-01df0e05b78c"
}
```

## <a name="add-get-request-to-determine-operation-status"></a>新增 GET 要求以判斷作業狀態

檢查作業的狀態。
    
[!code-nodejs[Add GET request to determine operation status](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=113-137 "Add GET request to determine operation status")]

此 API 呼叫會傳回包含作業狀態的 JSON 回應： 

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:22:53Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

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

## <a name="add-updatekb-method"></a>新增 update_kb 方法

下列方法會更新 KB，並重複檢查狀態。 因為 KB 建立可能需要一些時間，您必須重複呼叫來檢查狀態，直到狀態為成功或失敗。

[!code-nodejs[Add update_kb method](~/samples-qnamaker-nodejs/documentation-samples/quickstarts/update-knowledge-base/update-knowledge-base.js?range=139-169 "Add update_kb method")]

## <a name="run-the-program"></a>執行程式

在命令列中輸入下列命令，以執行程式。 程式會將要求傳送至 QnA Maker API 以更新 KB，然後會每隔 30 秒輪詢結果。 每個回應都會列印到主控台視窗。

```bash
node update-knowledge-base.js
```

一旦更新您的知識庫，您可以在 QnA Maker 入口網站的[我的知識庫](https://www.qnamaker.ai/Home/MyServices)頁面中檢視。 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)