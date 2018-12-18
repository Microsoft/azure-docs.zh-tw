---
title: 快速入門：產生縮圖 - REST、Node.js - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 Node.js，從影像產生縮圖。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: 940d5f87e8e411d520c3d30f3e9f08e982162748
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338585"
---
# <a name="quickstart-generate-a-thumbnail-using-the-rest-api-and-nodejs-in-computer-vision"></a>快速入門：使用電腦視覺中的 REST API 和 Node.js 產生縮圖

在本快速入門中，您會使用電腦視覺的 REST API 從影像產生縮圖。 您可以使用[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法產生影像的縮圖。 您可指定高度和寬度，可以與輸入影像的外觀比例不同。 「電腦視覺」會使用智慧型裁剪，以智慧方式識別感興趣的區域，並且根據該區域產生裁剪的座標。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

- 您必須安裝 [Node.js](https://nodejs.org) 4.x 或更新版本。
- 您必須安裝 [npm](https://www.npmjs.com/)。
- 您必須具有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample"></a>建立並執行範例

若要建立並執行範例，請執行下列步驟：

1. 安裝 npm [`request`](https://www.npmjs.com/package/request) 套件。
   1. 以系統管理員身分開啟 [命令提示字元] 視窗。
   1. 執行以下命令：

      ```console
      npm install request
      ```

   1. 成功安裝套件之後，關閉命令提示字元視窗。

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscriptionKey` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將 `uriBase` 的值取代為 [Get Thumbnail](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb) 方法的端點 URL。
    1. (選擇性) 將 `imageUrl` 的值取代為您要分析之不同影像的 URL。
1. 將程式碼儲存為副檔名為 `.js` 的檔案。 例如： `get-thumbnail.js`。
1. 開啟命令提示字元視窗。
1. 出現提示時，使用 `node` 命令來執行該檔案。 例如： `node get-thumbnail.js`。

```nodejs
'use strict';

const request = require('request');

// Replace <Subscription Key> with your valid subscription key.
const subscriptionKey = '<Subscription Key>';

// You must use the same location in your REST call as you used to get your
// subscription keys. For example, if you got your subscription keys from
// westus, replace "westcentralus" in the URL below with "westus".
const uriBase =
    'https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail';

const imageUrl =
    'https://upload.wikimedia.org/wikipedia/commons/9/94/Bloodhound_Puppy.jpg';

// Request parameters.
const params = {
    'width': '100',
    'height': '100',
    'smartCropping': 'true'
};

const options = {
    uri: uriBase,
    qs: params,
    body: '{"url": ' + '"' + imageUrl + '"}',
    headers: {
        'Content-Type': 'application/json',
        'Ocp-Apim-Subscription-Key' : subscriptionKey
    }
};

request.post(options, (error, response, body) => {
  if (error) {
    console.log('Error: ', error);
    return;
  }
});
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以二進位資料的形式傳回，代表縮圖的影像資料。 如果要求失敗，則回應會顯示在主控台視窗中。 如果要求失敗，回應會包含錯誤碼和訊息以協助判斷問題所在。

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人與地標、建立縮圖，以及擷取印刷與手寫文字的電腦視覺 API。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
