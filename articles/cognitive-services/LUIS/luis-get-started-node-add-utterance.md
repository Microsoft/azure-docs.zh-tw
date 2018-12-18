---
title: Node.js 快速入門 - 變更模型及訓練 LUIS 應用程式
titleSuffix: Azure Cognitive Services
description: 在此 Node.js 快速入門中，會將範例語句新增至「家庭自動化」應用程式，並且將應用程式定型。 範例語句是對應到意圖的交談使用者文字。 您可以藉由提供與意圖相關的範例語句，教導 LUIS 哪些種類的使用者提供文字屬於哪種意圖。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: a487f44e164830928367d9f6ea737e793e38c0a8
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036145"
---
# <a name="quickstart-change-model-using-nodejs"></a>快速入門：使用 Node.js 來變更模型

[!INCLUDE [Quickstart introduction for change model](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

[!INCLUDE [Quickstart prerequisites for changing model](../../../includes/cognitive-services-luis-qs-change-model-prereq.md)]
* 採用 NPM 的最新 [**Node.js**](https://nodejs.org/en/download/)。
* 本文的 NPM 相依性：[**request**](https://www.npmjs.com/package/request)、[**request-promise**](https://www.npmjs.com/package/request-promise)、[**fs-extra**](https://www.npmjs.com/package/fs-extra)。  
* [Visual Studio Code](https://code.visualstudio.com/)。

[!INCLUDE [Code is available in LUIS-Samples Github repo](../../../includes/cognitive-services-luis-qs-change-model-luis-repo-note.md)]

## <a name="example-utterances-json-file"></a>範例語句 JSON 檔案

[!INCLUDE [Quickstart explanation of example utterance JSON file](../../../includes/cognitive-services-luis-qs-change-model-json-ex-utt.md)]

## <a name="create-quickstart-code"></a>建立快速入門程式碼 

將 NPM 相依性新增至名為 `add-utterances.js` 的檔案。

   [!code-javascript[NPM Dependencies](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=8-11 "NPM Dependencies")]

將 LUIS 常數新增至此檔案。 複製下列程式碼，並且變更為您的撰寫金鑰、應用程式識別碼和版本識別碼。

   [!code-javascript[LUIS key and IDs](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=13-22 "LUIS key and IDs")]

新增上傳檔案 (包含語句) 的名稱與位置。 

   [!code-javascript[Add upload file](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=24-26 "Add upload file")]

針對 `addUtterance` 函式新增方法和物件。

   [!code-javascript[Add configuration information for adding utterance](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=28-67 "Add configuration information for adding utterance")]

針對 `train` 函式新增方法和物件。

   [!code-javascript[Add configuration information for training LUIS](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=69-101 "Add configuration information for training LUIS")]

新增 `sendUtteranceToApi` 函式以傳送和接收 HTTP 呼叫。 

   [!code-javascript[Send the HTTP request](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=103-119 "Send the HTTP request")]

新增 **main** 程式碼，它會選擇動作。

   [!code-javascript[Main function](~/samples-luis/documentation-samples/quickstarts/change-model/node/add-utterances.js?range=121-143 "Main function")]

### <a name="install-dependencies"></a>安裝相依性

建立 `package.json` 檔案，具有下列文字：

   [!code-json[Package.json](~/samples-luis/documentation-samples/quickstarts/change-model/node/package.json "Package.json")]

在命令列中，從具有 package.json 的目錄中，使用 NPM 安裝相依性：`npm install`。

## <a name="run-code"></a>執行程式碼

使用 Node.js 從命令列執行應用程式。

呼叫 `npm start` 會新增語句、定型及取得定型狀態。

```CMD
> npm start 
```

此命令列會顯示呼叫新增語句 API 的結果。 

[!INCLUDE [Quickstart response from API calls](../../../includes/cognitive-services-luis-qs-change-model-json-results.md)]


## <a name="clean-up-resources"></a>清除資源

當您完成快速入門時，請移除在本快速入門中建立的所有檔案。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [以程式設計方式建置 LUIS 應用程式](luis-tutorial-node-import-utterances-csv.md)