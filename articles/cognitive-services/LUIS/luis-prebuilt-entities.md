---
title: Language Understanding (LUIS) 預先建置的實體
titleSuffix: Azure Cognitive Services
description: LUIS 包含一組預先建置的實體，用於辨識常見的資訊類型，例如日期、時間、數字、計量及貨幣。 預先建置的實體支援會因 LUIS 應用程式的文化特性而異。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/06/2018
ms.author: diberry
ms.openlocfilehash: e3bd203c9ab1d6daaae04866cf195b3ca28c3078
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041552"
---
# <a name="prebuilt-entities-to-recognize-common-data-types"></a>預先建置的實體可辨識常見的資料類型

LUIS 包含一組預先建置的實體，用於辨識常見的資訊類型，例如日期、時間、數字、計量及貨幣。 預先建置的實體支援會因 LUIS 應用程式的文化特性而異。 如需 LUIS 所支援預先建置實體的完整清單，包括文化特性支援的實體在內，請參閱[預先建置的實體參考](./luis-reference-prebuilt-entities.md)。

> [!NOTE]
> **builtin.datetime** 已被取代。 它已由 [**builtin.datetimeV2**](luis-reference-prebuilt-datetimev2.md) 取代，可供辨識日期與時間範圍，以及改善辨識語意模糊的日期與時間。

## <a name="add-a-prebuilt-entity"></a>新增預先建置的實體

1. 在 [我的應用程式] 頁面上按一下應用程式名稱來開啟應用程式，然後按一下左側的 [實體]。 
2. 在 [實體] 頁面上，按一下 [管理預先建置的實體]。

3. 在 [新增預先建置的實體] 對話方塊中，按一下想要新增的預先建置實體 (例如 "datetimeV2")。 然後按一下 [儲存] 。

    ![新增預先建置的實體對話方塊](./media/luis-use-prebuilt-entity/add-prebuilt-entity-dialog.png)

## <a name="use-a-prebuilt-number-entity"></a>使用預先建置的 number 實體
當應用程式中包含預先建置的實體時，其預測會包含在已發佈的應用程式中。 預先建置實體的行為已預先定型且**無法**修改。 請遵循下列步驟，以查看預先建置實體的運作方式：

1. 將 **number** 實體新增到應用程式，然後將應用程式[定型](luis-interactive-test.md)和[發佈](luis-how-to-publish-app.md)。
2. 按一下 [發佈應用程式] 頁面中的端點 URL，以在網頁瀏覽器中開啟 LUIS 端點。 
3. 將語句附加到包含數值運算式的 URL。 例如，您可以輸入 `buy two plane ticktets`，然後看見 LUIS 將 `two` 識別為 `builtin.number` 實體，並將 `2`識別為 `resolution` 欄位中的值。 `resolution` 欄位可協助將數字與日期解析為標準格式，更方便用戶端應用程式使用。 

    ![瀏覽器中包含 number 實體的語句](./media/luis-use-prebuilt-entity/browser-query.png)

LUIS 可運用智能辨識非標準格式的數字。 請在語句中試用不同的數值運算式，並查看 LUIS 傳回的結果。

下列範例顯示來自 LUIS 的 JSON 回應，其中包含「兩打」語句的值 24。

```json
{
  "query": "order two dozen tickets for group travel",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 0.905443209
  },
  "entities": [
    {
      "entity": "two dozen",
      "type": "builtin.number",
      "startIndex": 6,
      "endIndex": 14,
      "resolution": {
        "value": "24"
      }
    }
  ]
}
```
## <a name="use-a-prebuilt-datetimev2-entity"></a>使用預先建置的 datetimeV2 實體
預先建置的 **datetimeV2** 實體可辨識日期、時間、日期範圍及持續時間。 請遵循下列步驟，以查看 `datetimeV2` 預先建置實體的運作方式：

1. 將 **datetimeV2** 實體新增到應用程式，然後將應用程式[定型](luis-interactive-test.md)和[發佈](luis-how-to-publish-app.md)。
2. 按一下 [發佈應用程式] 頁面中的端點 URL，以在網頁瀏覽器中開啟 LUIS 端點。 
3. 將語句附加到包含日期範圍的 URL。 例如，您可以輸入 `book a flight tomorrow`，然後看見 LUIS 將 `tomorrow` 識別為 `builtin.datetimeV2.date` 實體，並將明天的日期識別為 `resolution` 欄位中的值。 

下列範例顯示若今天的日期為 2017 年 10 月 31 日，來自 LUIS 的 JSON 回應可能如下所示。

```json
{
  "query": "book a flight tomorrow",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 0.9063408
  },
  "entities": [
    {
      "entity": "tomorrow",
      "type": "builtin.datetimeV2.date",
      "startIndex": 14,
      "endIndex": 21,
      "resolution": {
        "values": [
          {
            "timex": "2017-11-01",
            "type": "date",
            "value": "2017-11-01"
          }
        ]
      }
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"]
> [預先建置的實體參考](./luis-reference-prebuilt-entities.md)
