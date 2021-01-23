---
title: 快速入門：使用 cURL 和 REST 來管理知識庫 - QnA Maker
description: 本快速入門說明如何使用 REST API 來建立、發佈和查詢知識庫。
ms.date: 1/22/2021
ms.topic: quickstart
ms.openlocfilehash: d4c832c4a96ee1419b9e2f0accc1b86e9394ee02
ms.sourcegitcommit: 75041f1bce98b1d20cd93945a7b3bd875e6999d0
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98701277"
---
[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Prerequisites

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

* 最新版的 [cURL](https://curl.haxx.se/)。 快速入門會使用數個命令列參數，其記載於 [cURL 文件](https://curl.haxx.se/docs/manpage.html)中。
* 您必須擁有 [QnA Maker 資源](../How-To/set-up-qnamaker-service-azure.md)，才能使用金鑰和資源名稱。 您已在資源建立期間輸入資源 **名稱**，接下來系統會為您建立金鑰。 資源名稱會用來作為端點的子網域。 若要擷取您的金鑰和資源名稱，請在 Azure 入口網站中針對您的資源選取 [快速入門]  。 資源名稱是端點 URL 的第一個子網域：

    `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0`

> [!CAUTION]
> 下列 BASH 範例會使用 `\` 行接續字元。 如果您的主控台或終端機使用不同的行接續字元，請使用此字元。

# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)

* 最新版的 [cURL](https://curl.haxx.se/)。 快速入門會使用數個命令列參數，其記載於 [cURL 文件](https://curl.haxx.se/docs/manpage.html)中。
* 您必須擁有 [QnA Maker 資源](../How-To/set-up-qnamaker-service-azure.md)，才能使用金鑰和資源名稱。 您已在資源建立期間輸入資源 **名稱**，接下來系統會為您建立金鑰。 資源名稱會用來作為端點的子網域。 若要擷取您的金鑰和資源名稱，請在 Azure 入口網站中針對您的資源選取 [快速入門]  。 資源名稱是端點 URL 的第一個子網域：

    `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1`

> [!CAUTION]
> 下列 BASH 範例會使用 `\` 行接續字元。 如果您的主控台或終端機使用不同的行接續字元，請使用此字元。

---

## <a name="create-a-knowledge-base"></a>建立知識庫

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

若要使用 REST API 和 cURL 建立知識庫，您需要有下列資訊：

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|描述知識庫的 JSON|`-d` 參數|JSON 的[範例](/rest/api/cognitiveservices/qnamaker/knowledgebase/create#examples)|
|JSON 的大小 (以位元組為單位)|`Content-Size` 標頭的 `-h` 參數||

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰，以及 JSON 值和 JSON 大小來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/create \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:107" \
-d '{ name: "QnA Maker FAQ",urls: [ "https://docs.microsoft.com/en-in/azure/cognitive-services/qnamaker/faqs"]}'
```

來自 QnA Maker 的 cURL 回應包含 `operationId`，這是[取得作業狀態](#get-status-of-operation)所需的項目。

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2020-02-27T04:11:22Z",
  "lastActionTimestamp": "2020-02-27T04:11:22Z",
  "userId": "9596077b3e0441eb93d5080d6a15c64b",
  "operationId": "95a4f700-9899-4c98-bda8-5449af9faef8"
}
```

# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)


若要使用 REST API 和 cURL 建立知識庫，您需要有下列資訊：

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|描述知識庫的 JSON|`-d` 參數|JSON 的[範例](/rest/api/cognitiveservices/qnamaker/knowledgebase/create#examples)|
|JSON 的大小 (以位元組為單位)|`Content-Size` 標頭的 `-h` 參數||

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰，以及 JSON 值和 JSON 大小來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/create \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:107" \
-d '{ name: "QnA Maker FAQ",urls: [ "https://docs.microsoft.com/en-in/azure/cognitive-services/qnamaker/faqs"]}'
```

來自 QnA Maker 的 cURL 回應包含 `operationId`，這是[取得作業狀態](#get-status-of-operation)所需的項目。

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2020-02-27T04:11:22Z",
  "lastActionTimestamp": "2020-02-27T04:11:22Z",
  "userId": "9596077b3e0441eb93d5080d6a15c64b",
  "operationId": "95a4f700-9899-4c98-bda8-5449af9faef8"
}
```
---

## <a name="get-status-of-operation"></a>取得作業的狀態

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

當您建立知識庫時，因為作業為非同步，回應會包含用以判斷狀態的資訊。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|作業識別碼|URL 路由|`/operations/REPLACE-WITH-YOUR-OPERATION-ID`|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和作業識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/operations/REPLACE-WITH-YOUR-OPERATION-ID \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

cURL 回應包含狀態。 如果作業狀態為成功，則 `resourceLocation` 包含知識庫識別碼。

```json
{
   "operationState": "Succeeded",
   "createdTimestamp": "2020-02-27T04:54:07Z",
   "lastActionTimestamp": "2020-02-27T04:54:19Z",
   "resourceLocation": "/knowledgebases/fe3971b7-cfaa-41fa-8d9f-6ceb673eb865",
   "userId": "f596077b3e0441eb93d5080d6a15c64b",
   "operationId": "f293f218-d080-48f0-a766-47993e9b26a8"
}
```
# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)


當您建立知識庫時，因為作業為非同步，回應會包含用以判斷狀態的資訊。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|作業識別碼|URL 路由|`/operations/REPLACE-WITH-YOUR-OPERATION-ID`|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和作業識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/operations/REPLACE-WITH-YOUR-OPERATION-ID \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

cURL 回應包含狀態。 如果作業狀態為成功，則 `resourceLocation` 包含知識庫識別碼。

```json
{
   "operationState": "Succeeded",
   "createdTimestamp": "2020-02-27T04:54:07Z",
   "lastActionTimestamp": "2020-02-27T04:54:19Z",
   "resourceLocation": "/knowledgebases/fe3971b7-cfaa-41fa-8d9f-6ceb673eb865",
   "userId": "f596077b3e0441eb93d5080d6a15c64b",
   "operationId": "f293f218-d080-48f0-a766-47993e9b26a8"
}
```

---

## <a name="publish-knowledge-base"></a>發佈知識庫

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

查詢知識庫之前，您必須：
* 發佈知識庫
* 取得執行階段端點金鑰

此工作會發佈知識庫。 取得執行階段端點金鑰是[個別的工作](#get-published-knowledge-bases-runtime-endpoint-key)。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-v \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
--data-raw ''
```

回應狀態為 204，沒有任何結果。 使用 `-v` 命令列參數來查看 cURL 命令的詳細資訊輸出。 這會包含 HTTP 狀態。

# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)

查詢知識庫之前，您必須：
* 發佈知識庫
* 取得執行階段端點金鑰

此工作會發佈知識庫。 取得執行階段端點金鑰是[個別的工作](#get-published-knowledge-bases-runtime-endpoint-key)。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-v \
-X POST \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY" \
--data-raw ''
```

回應狀態為 204，沒有任何結果。 使用 `-v` 命令列參數來查看 cURL 命令的詳細資訊輸出。 這會包含 HTTP 狀態。

---

## <a name="get-published-knowledge-bases-runtime-endpoint-key"></a>取得已發佈知識庫的執行階段端點金鑰

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

查詢知識庫之前，您必須：
* 發佈知識庫
* 取得執行階段端點金鑰

此工作會取得執行階段端點金鑰。 發佈知識庫是[個別的工作](#publish-knowledge-base)。

執行階段端點金鑰適用於使用 QnA Maker 資源的所有知識庫。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/endpointkeys \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```


cURL 回應包含執行階段端點金鑰。 查詢時只要使用其中一個金鑰，即可從知識庫取得答案。

```json
{
  "primaryEndpointKey": "93e88a14-694a-44d5-883b-184a68aa8530",
  "secondaryEndpointKey": "92c98c16-ca31-4294-8626-6c57454a5063",
  "installedVersion": "4.0.5",
  "lastStableVersion": "4.0.6"
}
```
# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)

查詢知識庫之前，您必須：
* 發佈知識庫
* 取得執行階段端點金鑰

此工作會取得執行階段端點金鑰。 發佈知識庫是[個別的工作](#publish-knowledge-base)。

執行階段端點金鑰適用於使用 QnA Maker 資源的所有知識庫。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/endpointkeys \
-X GET \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```


cURL 回應包含執行階段端點金鑰。 查詢時只要使用其中一個金鑰，即可從知識庫取得答案。

```json
{
  "primaryEndpointKey": "93e88a14-694a-44d5-883b-184a68aa8530",
  "secondaryEndpointKey": "92c98c16-ca31-4294-8626-6c57454a5063",
  "installedVersion": "4.0.5",
  "lastStableVersion": "4.0.6"
}
```

---

## <a name="query-for-answer-from-published-knowledge-base"></a>從已發佈的知識庫查詢答案

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

從知識庫取得答案是從個別的執行階段完成，而不是管理知識庫。 因為這是個別的執行時間，所以您必須使用執行階段金鑰進行驗證。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 執行階段金鑰|`Authorization` 標頭的 `-h` 參數|金鑰是字串的一部分，其中包含 `Endpointkey `這個字。 驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|
|描述查詢的 JSON|`-d` 參數|JSON 的[要求本文參數](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#request-body)和[範例](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#examples)|
|JSON 的大小 (以位元組為單位)|`Content-Size` 標頭的 `-h` 參數||

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.azurewebsites.net/qnamaker/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID/generateAnswer \
-X POST \
-H "Authorization: EndpointKey REPLACE-WITH-YOUR-RUNTIME-KEY" \
-H "Content-Type:application/json" \
-H "Content-Size:159" \
-d '{"question": "How are QnA Maker and LUIS used together?","top": 6,"isTest": true,  "scoreThreshold": 20, "strictFilters": [], "userId": "sd53lsY="}'
```

成功的回應包括最佳答案，以及用戶端應用程式 (例如聊天 Bot) 向使用者顯示答案所需的其他資訊。

# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)

從知識庫取得答案是從個別的執行階段完成，而不是管理知識庫。 因為這是個別的執行時間，所以您必須使用執行階段金鑰進行驗證。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|
|描述查詢的 JSON|`-d` 參數|JSON 的[要求本文參數](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#request-body)和[範例](/rest/api/cognitiveservices/qnamakerruntime/runtime/generateanswer#examples)|
|JSON 的大小 (以位元組為單位)|`Content-Size` 標頭的 `-h` 參數||

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID/generateAnswer \
-X POST \
-H "Content-Type:application/json" \
-H "Content-Size:159" \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
-d '{"question": "How are QnA Maker and LUIS used together?","top": 6,"isTest": true,  "scoreThreshold": 20, "strictFilters": [], "userId": "sd53lsY="}'
```

成功的回應包括最佳答案，以及用戶端應用程式 (例如聊天 Bot) 向使用者顯示答案所需的其他資訊。


---

## <a name="delete-knowledge-base"></a>刪除知識庫

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker 正式發行 (穩定版本)](#tab/v1)

知識庫使用完畢後，請加以刪除。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v4.0/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-X DELETE \
-v \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

回應狀態為 204，沒有任何結果。 使用 `-v` 命令列參數來查看 cURL 命令的詳細資訊輸出。 這會包含 HTTP 狀態。

# <a name="qna-maker-managed-preview-release"></a>[受控 QnA Maker (預覽版本)](#tab/v2)

知識庫使用完畢後，請加以刪除。

|資訊|cURL 組態|目的|
|--|--|--|
|QnA Maker 資源名稱|URL|用於建構 URL|
|QnA Maker 資源金鑰|`Ocp-Apim-Subscription-Key` 標頭的 `-h` 參數|驗證 QnA Maker 服務|
|知識庫識別碼|URL 路由|`/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID`|

cURL 命令是從 BASH 殼層執行。 使用您自己的資源名稱、資源金鑰和知識庫識別碼來編輯此命令。

```bash
curl https://REPLACE-WITH-YOUR-RESOURCE-NAME.cognitiveservices.azure.com/qnamaker/v5.0-preview.1/knowledgebases/REPLACE-WITH-YOUR-KNOWLEDGE-BASE-ID \
-X DELETE \
-v \
-H "Ocp-Apim-Subscription-Key: REPLACE-WITH-YOUR-RESOURCE-KEY"
```

回應狀態為 204，沒有任何結果。 使用 `-v` 命令列參數來查看 cURL 命令的詳細資訊輸出。 這會包含 HTTP 狀態。

---

## <a name="additional-resources"></a>其他資源

* [撰寫](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)參考文件
* [執行階段](/rest/api/cognitiveservices/qnamaker4.0/runtime)參考文件
* [使用 cURL 的範例 BASH 指令碼](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/curl/QnAMaker)