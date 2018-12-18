---
title: 使用 Nodejs 以清單實體自動標記實體 | Microsoft Docs
description: 了解如何新增清單實體以協助 LUIS 標記字組或片語的差異。
services: cognitive-services
author: diberry
titleSuffix: Azure
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 02/21/2018
ms.author: diberry
ms.openlocfilehash: 42fde2b24f851129e24257bbfe6d65a96e235485
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47036774"
---
# <a name="use-a-list-entity-to-increase-entity-detection"></a>使用清單實體增加實體偵測 
本教學課程將示範如何使用[清單實體](luis-concept-entity-types.md)增加實體偵測。 清單實體不需要標籤，因為其與字詞完全相符。  

在本教學課程中，您了解如何：

> [!div class="checklist"]
* 建立清單實體 
* 新增正規化的值和同義字
* 驗證改善的實體識別

## <a name="prerequisites"></a>必要條件

> [!div class="checklist"]
> * 最新版的 [Node.js](https://nodejs.org)
> * [HomeAutomation LUIS 應用程式](luis-get-started-create-app.md)。 如果您尚未建立「家庭自動化」應用程式，請建立新的應用程式，並新增預先建置的網域 **HomeAutomation**。 訓練和發佈應用程式。 
> * LUIS 應用程式的 [AuthoringKey](luis-concept-keys.md#authoring-key)、[EndpointKey](luis-concept-keys.md#endpoint-key) (如果查詢多次)、應用程式識別碼、版本識別碼和[區域](luis-reference-regions.md)。

> [!Tip]
> 如果您還沒有訂用帳戶，則可以註冊[免費帳戶](https://azure.microsoft.com/free/)。

此教學課程中的所有程式碼皆位於 [LUIS 範例 github 存放庫](https://github.com/Microsoft/LUIS-Samples/tree/master/documentation-samples/tutorial-list-entity)。 

## <a name="use-homeautomation-app"></a>使用 HomeAutomation 應用程式
HomeAutomation 應用程式可讓您控制許多裝置，例如燈和娛樂系統，以及控制環境，例如升溫或降溫。 這些系統有許多不同名稱，其中可能包含製造商名稱、別名、暱稱、首字母縮寫和俚語。 

因為不同文化和人口資料而有許多名稱的其中一個系統就是控溫器。 控溫器可以控制住家或辦公大樓的降溫和升溫系統。

在理想情況下，預先建置的實體 **HomeAutomation.Device** 應能解析下列語句：

|#|語句|識別的實體|分數|
|--|--|--|--|
|1|turn on the ac (開啟冷氣)|HomeAutomation.Device - "ac"|0.8748562|
|2|turn up the heat (調高溫度)|HomeAutomation.Device - "heat"|0.784990132|
|3|make it colder (讓溫度冷一點)|||

前兩個語句會對應至不同裝置。 第三個語句 "make it colder" (讓溫度冷一點) 不會對應到裝置，而是要求結果。 LUIS 不知道字詞 "colder" (冷一點) 表示控溫器就是受到要求的裝置。 在理想情況下，LUIS 應解析所有這些以相同裝置為對象的語句。 

## <a name="use-a-list-entity"></a>使用清單實體
HomeAutomation.Device 實體最適合用於少數裝置或少數名稱差異。 對於辦公大樓或校園，裝置名稱會成長到超出 HomeAutomation.Device 實體的效益。 

**清單實體**十分適用於這類情況，因為辦公大樓和校園中的裝置字詞集合是已知的集合 (即使是大型集合也一樣)。 藉由使用清單實體，LUIS 可以針對控溫器接收集合中的所有可能值，並只會將其解析至單一裝置「控溫器」。 

本教學課程將使用控溫器來建立實體清單。 在本教學課程中，控溫器的替代名稱如下： 

|控溫器的替代名稱|
|--|
| ac |
| a/c|
| a-c|
|heater|
|hot|
|hotter|
|cold|
|colder|

如果 LUIS 需要時常判斷新的替代名稱，則[片語清單](luis-concept-feature.md#how-to-use-phrase-lists)會更適用。

## <a name="create-a-list-entity"></a>建立清單實體
建立 Node.js 檔案，並將下列程式碼複製到其中。 變更 authoringKey、appId、versionId 和 region 值。

   [!code-javascript[Create DevicesList List Entity](~/samples-luis/documentation-samples/tutorial-list-entity/add-entity-list.js "Create DevicesList List Entity")]

使用下列命令來安裝 NPM 相依項目，並執行程式碼來建立清單實體：

```Javascript
npm install && node add-entity-list.js
```

此執行的輸出是清單實體的識別碼：

```Javascript
026e92b3-4834-484f-8608-6114a83b03a6
```
## <a name="train-the-model"></a>訓練模型
訓練 LUIS 是為了讓新清單影響查詢結果。 訓練包含兩個部分的訓練程序，然後會在訓練完成時檢查狀態。 有多個模型的應用程式可能需要幾分鐘的時間來訓練。 下列程式碼會訓練應用程式，然後等候訓練成功。 程式碼會使用「等候並重試」策略以避免發生「要求太多」的 429 錯誤。 

建立 Node.js 檔案，並將下列程式碼複製到其中。 變更 authoringKey、appId、versionId 和 region 值。

   [!code-javascript[Train LUIS](~/samples-luis/documentation-samples/tutorial-list-entity/train.js "Train LUIS")]

使用下列命令以執行程式碼來訓練應用程式：

```Javascript
node train.js
```

此執行的輸出是每次 LUIS 模型訓練的反覆項目狀態。 下列執行只需一次訓練檢查：

```Javascript
1 trained = true
[ { modelId: '2c549f95-867a-4189-9c35-44b95c78b70f',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } },
  { modelId: '5530e900-571d-40ec-9c78-63e66b50c7d4',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } },
  { modelId: '519faa39-ae1a-4d98-965c-abff6f743fe6',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } },
  { modelId: '9671a485-36a9-46d5-aacd-b16d05115415',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } },
  { modelId: '9ef7d891-54ab-48bf-8112-c34dcd75d5e2',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } },
  { modelId: '8e16a660-8781-4abf-bf3d-f296ebe1bf2d',
    details: { statusId: 2, status: 'UpToDate', exampleCount: 45 } } ]

```
## <a name="publish-the-model"></a>發行模型
發行後即可從端點使用清單實體。

建立 Node.js 檔案，並將下列程式碼複製到其中。 變更 endpointKey、appId 和 region 值。 如果您呼叫此檔案時不打算超過配額限制，則可以使用您的 authoringKey。

   [!code-javascript[Publish LUIS](~/samples-luis/documentation-samples/tutorial-list-entity/publish.js "Publish LUIS")]

使用下列命令以執行程式碼來查詢應用程式：

```Javascript
node publish.js
```

下列輸出會包含所有查詢的端點 url。 實際的 JSON 結果會包含實際的 appID。 

```JSON
{ 
  versionId: null,
  isStaging: false,
  endpointUrl: 'https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>',
  region: null,
  assignedEndpointKey: null,
  endpointRegion: 'westus',
  publishedDateTime: '2018-01-29T22:17:38Z' }
}
```

## <a name="query-the-app"></a>查詢應用程式 
從端點查詢應用程式，以證明清單實體可協助 LUIS 判斷裝置類型。

建立 Node.js 檔案，並將下列程式碼複製到其中。 變更 endpointKey、appId 和 region 值。 如果您呼叫此檔案時不打算超過配額限制，則可以使用您的 authoringKey。

   [!code-javascript[Query LUIS](~/samples-luis/documentation-samples/tutorial-list-entity/query.js "Query LUIS")]

使用下列命令以執行程式碼和查詢應用程式：

```Javascript
node train.js
```

此輸出為查詢結果。 由於程式碼已新增 **verbose** 名稱/值配對組到查詢字串中，因此此輸出會包含所有意圖及其分數：

```JSON
{
  "query": "turn up the heat",
  "topScoringIntent": {
    "intent": "HomeAutomation.TurnOn",
    "score": 0.139018849
  },
  "intents": [
    {
      "intent": "HomeAutomation.TurnOn",
      "score": 0.139018849
    },
    {
      "intent": "None",
      "score": 0.120624863
    },
    {
      "intent": "HomeAutomation.TurnOff",
      "score": 0.06746891
    }
  ],
  "entities": [
    {
      "entity": "heat",
      "type": "HomeAutomation.Device",
      "startIndex": 12,
      "endIndex": 15,
      "score": 0.784990132
    },
    {
      "entity": "heat",
      "type": "DevicesList",
      "startIndex": 12,
      "endIndex": 15,
      "resolution": {
        "values": [
          "Thermostat"
        ]
      }
    }
  ]
}
```

特定的**控溫器**裝置會以結果導向的查詢 "turn up the heat" (調高溫度) 來加以識別。 由於原本的 HomeAutomation.Device 實體仍在應用程式中，您也可以看到其結果。 

請嘗試其他兩個語句，您會看到此兩個語句也會傳回控溫器。 

|#|語句|實體|type|value|
|--|--|--|--|--|
|1|turn on the ac (開啟冷氣)| ac | DevicesList | 控溫器|
|2|turn up the heat (調高溫度)|heat| DevicesList |控溫器|
|3|make it colder (讓溫度冷一點)|colder|DevicesList|控溫器|

## <a name="next-steps"></a>後續步驟

您可以建立另一個清單實體，將裝置位置擴展到房間、樓層或辦公大樓。 
