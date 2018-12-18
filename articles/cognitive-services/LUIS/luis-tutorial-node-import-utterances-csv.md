---
title: 使用 Node.js 以程式設計方式建置 LUIS 應用程式 | Microsoft Docs
titleSuffix: Azure
description: 了解如何使用「LUIS 撰寫 API」，從 CSV 格式的既有資料，以程式設計方式建置 LUIS 應用程式。
services: cognitive-services
author: diberry
manager: cjgronlund
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 02/21/2018
ms.author: diberry
ms.openlocfilehash: 42b9800c94171ecbd2dadf30bb2ce2f342063552
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39238500"
---
# <a name="build-a-luis-app-programmatically-using-nodejs"></a>使用 Node.js 以程式設計方式建置 LUIS 應用程式

LUIS 提供一個具備 [LUIS](luis-reference-regions.md) 網站所有功能的程式設計 API。 當您已有既有資料時，這可節省時間，而且以程式設計方式建立 LUIS 應用程式會比手動輸入資訊快速。 

## <a name="prerequisites"></a>必要條件

* 登入 [LUIS](luis-reference-regions.md) 網站，然後在 [帳戶設定] 中尋找您的[撰寫金鑰](luis-concept-keys.md#authoring-key)。 您可以使用此金鑰來呼叫「撰寫 API」。
* 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。
* 本教學課程會從一個假設性公司的使用者要求記錄 CSV 檔開始著手。 在 [這裡](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/IoT.csv)下載。
* 安裝含有 NPM 的最新 Node.js。 請從[這裡](https://nodejs.org/en/download/)下載。
* **[建議使用]** 適用於 IntelliSense 和偵錯的 Visual Studio Code，請從[這裡](https://code.visualstudio.com/)免費下載。

## <a name="map-preexisting-data-to-intents-and-entities"></a>將既有資料對應至意圖和實體
即使您的系統在建立時並未將 LUIS 納入考量，只要它包含與使用者所要執行之各種不同工作對應的文字資料，您便可能得以提供一個現有使用者輸入分類與 LUIS 中意圖的對應。 如果您可以識別出使用者話語中的重要單字或片語，這些單字便可能對應至實體。

開啟 `IoT.csv` 檔案。 它包含使用者對假設性家庭自動化服務的查詢記錄，其中包括查詢分類方式、使用者語句，以及一些含有從它們當中提取之實用資訊的資料行。 

![CSV 檔案](./media/luis-tutorial-node-import-utterances-csv/csv.png) 

您會看到 **RequestType** 資料行可能為意圖，而 **Request** 資料行則顯示範例語句。 其他欄位如果有出現在語句中，則可能為實體。 由於有意圖、實體及範例語句，因此您需要一個簡單的範例應用程式。

## <a name="steps-to-generate-a-luis-app-from-non-luis-data"></a>從非 LUIS 資料產生 LUIS 應用程式的步驟
若要從來源檔案產生一個新的 LUIS 應用程式，您需先剖析來自 CSV 檔案的資料，然後使用「撰寫 API」將此資料轉換成您可以上傳到 LUIS 的格式。 從剖析的資料中，您會收集有關其中所含意圖和實體的資訊。 接著，您會發出 API 呼叫來建立應用程式，然後新增從所剖析資料收集的意圖和實體。 建立 LUIS 應用程式之後，您便可以從所剖析的資料新增範例語句。 您可以在下列程式碼的最後部分看到此流程。 請複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/index.js)此程式碼，並將其儲存成 `index.js`。

   [!code-javascript[Node.js code for calling the steps to build a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/index.js)]


## <a name="parse-the-csv"></a>剖析 CSV

包含 CSV 中語句的資料行項目必須剖析成 LUIS 可理解的 JSON 格式。 此 JSON 格式必須包含識別語句意圖的 `intentName` 欄位。 它也必須包含 `entityLabels` 欄位，如果語句中沒有任何實體，則此欄位可以空白。 

例如，"Turn on the lights" 項目會與此 JSON 對應：

```json
        {
            "text": "Turn on the lights",
            "intentName": "TurnOn",
            "entityLabels": [
                {
                    "entityName": "Operation",
                    "startCharIndex": 5,
                    "endCharIndex": 6
                },
                {
                    "entityName": "Device",
                    "startCharIndex": 12,
                    "endCharIndex": 17
                }
            ]
        }
```

在此範例中，`intentName` 來自 CSV 檔案中 **Request** 資料行標題下的使用者要求，而 `entityName` 則來自其他含有關鍵資訊的資料行。 例如，如果有 **Operation** 或 **Device** 的項目，而該字串也出現在實際的要求中，您便可以將它標記為實體。 下列程式碼示範此剖析程序。 您可以複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/_parse.js)此程式碼，並將其儲存成 `_parse.js`。

   [!code-javascript[Node.js code for parsing a CSV file to extract intents, entities, and labeled utterances](~/samples-luis/examples/build-app-programmatically-csv/_parse.js)]



## <a name="create-the-luis-app"></a>建立 LUIS 應用程式
將資料剖析成 JSON 之後，請將其新增至 LUIS 應用程式。 下列程式碼會建立 LUIS 應用程式。 請複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/_create.js)此程式碼，並將其儲存成 `_create.js`。

   [!code-javascript[Node.js code for creating a LUIS app](~/samples-luis/examples/build-app-programmatically-csv/_create.js)]


## <a name="add-intents"></a>新增對應方式
有了應用程式之後，您必須將意圖新增至應用程式。 下列程式碼會建立 LUIS 應用程式。 請複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/_intents.js)此程式碼，並將其儲存成 `_intents.js`。

   [!code-javascript[Node.js code for creating a series of intents](~/samples-luis/examples/build-app-programmatically-csv/_intents.js)]


## <a name="add-entities"></a>新增實體
下列程式碼會將實體新增至 LUIS 應用程式。 請複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/_entities.js)此程式碼，並將其儲存成 `_entities.js`。

   [!code-javascript[Node.js code for creating entities](~/samples-luis/examples/build-app-programmatically-csv/_entities.js)]
   


## <a name="add-utterances"></a>新增表達方式
在 LUIS 應用程式中定義實體和意圖之後，您可以新增語句。 下列程式碼會使用 [Utterances_AddBatch](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09) API，這可讓您一次最多新增 100 個語句。  請複製或[下載](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/build-app-programmatically-csv/_upload.js)此程式碼，並將其儲存成 `_upload.js`。

   [!code-javascript[Node.js code for adding utterances](~/samples-luis/examples/build-app-programmatically-csv/_upload.js)]


## <a name="run-the-code"></a>執行程式碼


### <a name="install-nodejs-dependencies"></a>安裝 Node.js 相依性
在終端機/命令列中，從 NPM 安裝 Node.js 相依性。

````
> npm install
````

### <a name="change-configuration-settings"></a>變更組態設定
若要使用此應用程式，您必須將 index.js 檔案中的值變更成自己的端點金鑰，然後提供想要讓應用程式擁有的名稱。 您也可以設定應用程式的文化特性，或變更版本號碼。

開啟 index.js 檔案，然後變更位於檔案頂端的這些值。


````JavaScript
// Change these values
const LUIS_programmaticKey = "YOUR_PROGRAMMATIC_KEY";
const LUIS_appName = "Sample App";
const LUIS_appCulture = "en-us"; 
const LUIS_versionId = "0.1";
````
### <a name="run-the-script"></a>執行指令碼
使用 Node.js 從終端機/命令列執行指令碼。

````
> node index.js
````
或
````
> npm start
````

### <a name="application-progress"></a>應用程式進度
當應用程式正在執行時，命令列會顯示進度。 命令列輸出會包含來自 LUIS 之回應的格式。

````
> node index.js
intents: ["TurnOn","TurnOff","Dim","Other"]
entities: ["Operation","Device","Room"]
parse done
JSON file should contain utterances. Next step is to create an app with the intents and entities it found.
Called createApp, created app with ID 314b306c-0033-4e09-92ab-94fe5ed158a2
Called addIntents for intent named TurnOn.
Called addIntents for intent named TurnOff.
Called addIntents for intent named Dim.
Called addIntents for intent named Other.
Results of all calls to addIntent = [{"response":"e7eaf224-8c61-44ed-a6b0-2ab4dc56f1d0"},{"response":"a8a17efd-f01c-488d-ad44-a31a818cf7d7"},{"response":"bc7c32fc-14a0-4b72-bad4-d345d807f965"},{"response":"727a8d73-cd3b-4096-bc8d-d7cfba12eb44"}]
called addEntity for entity named Operation.
called addEntity for entity named Device.
called addEntity for entity named Room.
Results of all calls to addEntity= [{"response":"6a7e914f-911d-4c6c-a5bc-377afdce4390"},{"response":"56c35237-593d-47f6-9d01-2912fa488760"},{"response":"f1dd440c-2ce3-4a20-a817-a57273f169f3"}]
retrying add examples...

Results of add utterances = [{"response":[{"value":{"UtteranceText":"turn on the lights","ExampleId":-67649},"hasError":false},{"value":{"UtteranceText":"turn the heat on","ExampleId":-69067},"hasError":false},{"value":{"UtteranceText":"switch on the kitchen fan","ExampleId":-3395901},"hasError":false},{"value":{"UtteranceText":"turn off bedroom lights","ExampleId":-85402},"hasError":false},{"value":{"UtteranceText":"turn off air conditioning","ExampleId":-8991572},"hasError":false},{"value":{"UtteranceText":"kill the lights","ExampleId":-70124},"hasError":false},{"value":{"UtteranceText":"dim the lights","ExampleId":-174358},"hasError":false},{"value":{"UtteranceText":"hi how are you","ExampleId":-143722},"hasError":false},{"value":{"UtteranceText":"answer the phone","ExampleId":-69939},"hasError":false},{"value":{"UtteranceText":"are you there","ExampleId":-149588},"hasError":false},{"value":{"UtteranceText":"help","ExampleId":-81949},"hasError":false},{"value":{"UtteranceText":"testing the circuit","ExampleId":-11548708},"hasError":false}]}]
upload done
````




## <a name="open-the-luis-app"></a>開啟 LUIS 應用程式
當指令碼完成之後，您可以登入 [LUIS](luis-reference-regions.md)，然後在 [我的應用程式] 底下查看您建立的 LUIS 應用程式。 您應該能夠在 [TurnOn]、[TurnOff] 及 [None] 意圖底下看到您所新增的語句。

![TurnOn 意圖](./media/luis-tutorial-node-import-utterances-csv/imported-utterances-661.png)


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [在 LUIS 網站中進行應用程式測試和定型](luis-interactive-test.md)

## <a name="additional-resources"></a>其他資源

此範例應用程式會使用下列 LUIS API：
- [建立應用程式](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36)
- [新增意圖](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0c)
- [新增實體](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0e) 
- [新增語句](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09)