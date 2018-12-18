---
title: 了解 LUIS 應用程式中的意圖
titleSuffix: Azure Cognitive Services
description: 意圖代表使用者想要執行的工作或動作。 它是使用者語句中表達的目的或目標。 請定義一組與使用者想要在您應用程式中執行之動作對應的意圖。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: cfad122b44944cfb637d8ef98931410d60c81ab8
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47041156"
---
# <a name="intents-in-luis"></a>LUIS 中的意圖

意圖代表使用者想要執行的工作或動作。 它是使用者[語句](luis-concept-utterance.md)中表達的目的或目標。

請定義一組與使用者想要在您應用程式中執行之動作對應的意圖。 例如，旅遊應用程式會定義數個意圖：

旅遊應用程式意圖   |   範例語句   | 
------|------|
 BookFlight     |   「幫我預訂下週到里約的班機」 <br/> 「24 日讓我飛到里約」 <br/> 「我需要一張下週日到里約熱內盧的機票」    |
 Greeting     |   「嗨」 <br/>「您好」 <br/>「早安」  |
 CheckWeather | 「波士頓的天氣如何？」 <br/> 「顯示本週的預測」 |
 None         | 「給我餅乾食譜」<br>「湖人隊贏了嗎？」 |

所有應用程式都隨附預先定義的意圖 "[None](#none-intent-is-fallback-for-app)"，這是後援意圖。 

## <a name="prebuilt-domains-provide-intents"></a>預先建置的定義域會提供意圖
除了您定義的意圖之外，還可以使用來自其中一個預先建置之定義域的預先建置意圖。 如需詳細資訊，請參閱[使用 LUIS 應用程式中的預先建置定義域](luis-how-to-use-prebuilt-domains.md)，以了解如何從預先建置的定義域自訂意圖，以在您的應用程式中使用。

## <a name="return-all-intents-scores"></a>傳回所有意圖的分數
您會指派語句給單一意圖。 當 LUIS 在端點上收到語句時，會針對該語句傳回一個最高分意圖。 如果您想要該語句所有意圖的分數，則可以在 API [端點呼叫](https://aka.ms/v1-endpoint-api-docs)的查詢字串上提供 `verbose=true` 旗標。 

## <a name="intent-compared-to-entity"></a>意圖與實體的比較
意圖代表 Chatbot 應該為使用者執行的動作，且所根據的是整個語句。 實體代表語句內所包含的單字或片語。 一個語句只能有一個最高分的意圖，但可以有許多個實體。 

<a name="how-do-intents-relate-to-entities"></a> 當使用者的「意圖」會觸發用戶端應用程式中的動作 (例如對 checkweather() 函式的呼叫) 時，請建立意圖。 然後建立實體來代表執行該動作所需的參數。 

|範例意圖   | 實體 | 範例語句中的實體   | 
|------------------|------------------------------|------------------------------|
| CheckWeather | { "type": "location", "entity": "seattle" }<br>{ "type": "builtin.datetimeV2.date","entity": "tomorrow","resolution":"2018-05-23" } | What's the weather like in `Seattle` `tomorrow`? |
| CheckWeather | { "type": "date_range", "entity": "this weekend" } | Show me the forecast for `this weekend` | 

## <a name="custom-intents"></a>自訂意圖

相似意圖的[語句](luis-concept-utterance.md)會與單一意圖對應。 您意圖中的語句可以使用應用程式中的任何[實體](luis-concept-entity-types.md)，因為實體並非意圖特定的。 

## <a name="prebuilt-domain-intents"></a>預先建置的定義域意圖

[預先建置的定義域](luis-how-to-use-prebuilt-domains.md)含有與語句相關的意圖。  

## <a name="none-intent-is-fallback-for-app"></a>None 意圖是應用程式的後援
**None** 是一個全部擷取或後援意圖。 它可用來教導 LUIS 在應用程式定義域 (主題區域) 中不重要的語句。 **None** 意圖應該佔應用程式中語句總數的百分之 10 到 20。 請勿將它保留空白。 

### <a name="none-intent-helps-conversation-direction"></a>None 意圖可協助引導交談方向
將語句預測為 None 意圖並以該預測傳回給 Chatbot 時，Bot 可以提出更多問題，並提供功能表將使用者導向到 Chatbot 中的有效選項。 

### <a name="no-utterances-in-none-intent-skews-predictions"></a>None 意圖中若無語句會扭曲預測
如果您沒有為 **None** 意圖新增任何語句，LUIS 就會強制讓定義域外的某個語句變成其中一個定義域意圖。 這會教導 LUIS 錯誤的語句意圖而扭曲預測分數。 

### <a name="add-utterances-to-the-none-intent"></a>將語句新增至 None 意圖
系統會建立 **None** 但刻意保留空白。 請以您定義域外的語句填入它。 一個良好的 **None** 語句係指完全在應用程式及應用程式所服務之產業外的語句。 例如，旅遊應用程式不應該將任何可能與旅遊相關的語句 (例如預訂、計費、食物、住宿、貨物、機上娛樂) 用於 **None**。 

哪種類型的語句是保留給 None 意圖的？ 請從您 Bot 明確不該回答的語句開始著手，例如「哪種類型的恐龍有藍色牙齒？」 這是非常明確與旅遊應用程式遠遠不相干的問題。 

### <a name="none-is-a-required-intent"></a>None 是必要意圖
**None** 意圖是必要意圖，您無法將其刪除或重新命名。

## <a name="negative-intentions"></a>負面意圖 
如果您想要判斷負面或正面意圖，例如「我**想要**一輛車」和「我**不**想要一輛車」，您可以建立兩個意圖 (一個正面，一個負面)，然後為每個意圖新增適當的語句。 或者，您也可以建立單一意圖，並標示兩個不同的正面和負面字詞作為實體。  

## <a name="intent-balance"></a>意圖平衡
應用程式定義域應該讓各個意圖之間的語句平衡。 請勿讓一個意圖有 10 個語句，而另一個意圖有 500 個語句。 這樣會造成不平衡。 如果您有這種情況，請檢閱有 500 個語句的意圖，看看是否可以將許多意圖都重新組織成[模式](luis-concept-patterns.md)。 

此平衡並不包括 **None** 意圖。 該意圖所包含的語句應該佔應用程式中總語句的 10%。

## <a name="intent-limits"></a>意圖限制
請檢閱[限制](luis-boundaries.md#model-boundaries)，以了解您可以將多少個意圖新增到模型中。 

### <a name="if-you-need-more-than-the-maximum-number-of-intents"></a>如果您所需的意圖超出意圖數目上限 
請先想想您的系統是否使用太多意圖。 

### <a name="can-multiple-intents-be-combined-into-single-intent-with-entities"></a>是否可以將多個意圖結合成含有實體的單一意圖 
太相似的意圖會讓 LUIS 更難以區分它們。 意圖應該要有足夠的差異，才能擷取使用者所要求的主要工作，但它們並不需要擷取您程式碼所採取的每個路徑。 例如，BookFlight 和 FlightCustomerService 可能是旅遊應用程式中的個別意圖，但 BookInternationalFlight 和 BookDomesticFlight 則太相似。 如果您的系統需要區分它們，請使用實體或其他邏輯，而不要使用意圖。 

### <a name="dispatcher-model"></a>發送器模型
深入了解如何使用[分派模型](luis-concept-enterprise.md#when-you-need-to-combine-several-luis-and-qna-maker-apps)將 LUIS 與 QnA Maker 應用程式結合。 

### <a name="request-help-for-apps-with-significant-number-of-intents"></a>針對含有大量意圖的應用程式要求協助
如果減少意圖數目或將意圖分割成多個應用程式對您並不適用，請與支援人員連絡。 如果您的 Azure 訂用帳戶包含支援服務，請與 [Azure 技術支援人員](https://azure.microsoft.com/support/options/)連絡。 

## <a name="next-steps"></a>後續步驟

* 深入了解[實體](luis-concept-entity-types.md)，這些是與意圖相關的重要單字
* 了解如何在 LUIS 應用程式中[新增及管理意圖](luis-how-to-add-intents.md)。
* 檢閱意圖[最佳做法](luis-concept-best-practices.md)