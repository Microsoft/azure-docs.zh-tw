---
title: LUIS 應用程式的企業概念 - Language Understanding
titleSuffix: Azure Cognitive Services
description: 了解大型 LUIS 應用程式的設計概念。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 3782874ac69b299e24cf1555b290a9d2d86b6285
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038335"
---
# <a name="enterprise-strategies-for-a-luis-app"></a>LUIS 應用程式的企業策略
檢閱您企業應用程式的這些設計策略。

## <a name="when-you-expect-luis-requests-beyond-the-quota"></a>當您預期 LUIS 要求會超出配額時
如果 LUIS 應用程式要求率超出允許的[配額率](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/)，請將負載分散至多個具有[相同應用程式定義](#use-multiple-apps-with-same-app-definition)的 LUIS 應用程式，或是建立並[指派多個金鑰](#assign-multiple-luis-keys-to-same-app)給應用程式。 

### <a name="use-multiple-apps-with-same-app-definition"></a>使用多個具有相同應用程式定義的應用程式
請匯出原始 LUIS 應用程式，然後將應用程式匯入回個別的應用程式。 每個應用程式都有自己的應用程式識別碼。 當您發佈時，請不要在所有應用程式都使用相同的金鑰，而是為每個應用程式建立個別的金鑰。 請平衡所有應用程式之間的負載，讓任何單一應用程式都不致於不堪負荷。 新增 [Application Insights](luis-tutorial-bot-csharp-appinsights.md) 以監視使用情況。 

為了在所有應用程式之間都獲得相同的最高分意圖，請確定第一個與第二個意圖之間的意圖預測有足夠的差距，讓 LUIS 不致產生混淆，而能針對語句中的微小變化在應用程式之間提供不同結果。 

請指定單一應用程式作為主要應用程式。 系統建議檢閱的所有語句都應該新增至主要應用程式，然後再移回至所有其他應用程式。 這會完整匯出應用程式，或將所標記的語句從主要應用程式載入至子應用程式。 您可以針對[單一語句](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c08) \(英文\) 或[批次](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09) \(英文\)，從 [LUIS](luis-reference-regions.md) 網站或撰寫 API 完成載入。 

請排定定期 (例如每隔兩週) 的[端點語句檢閱](luis-how-to-review-endoint-utt.md)來進行主動式學習，然後再重新定型並重新發佈。 

### <a name="assign-multiple-luis-keys-to-same-app"></a>將多個 LUIS 金鑰指派給相同的應用程式
如果您 LUIS 應用程式所收到的端點叫用次數超出單一金鑰配額允許的次數，請建立更多金鑰並指派給 LUIS 應用程式。 請建立流量管理員或負載平衡器，來管理所有端點金鑰上的端點查詢。 

## <a name="when-your-monolithic-app-returns-wrong-intent"></a>當您的整合型應用程式傳回錯誤意圖時
如果您的應用程式是用來預測各種使用者語句，請考慮實作[分派模型](#dispatch-tool-and-model)。 分解整合型應用程式可讓 LUIS 成功地專注於意圖間的偵測，而不會在跨父應用程式與子應用程式的意圖間產生混淆。 

請排定定期 (例如每隔兩週) 的[端點語句檢閱](luis-how-to-review-endoint-utt.md)來進行主動式學習，然後再重新定型並重新發佈。 

## <a name="when-you-need-to-have-more-than-500-intents"></a>當您所需的意圖超過 500 個時
例如，假設您要開發一個有超過 500 個意圖的辦公室助理。 如果有 200 個意圖與安排會議有關、200 個與提醒有關、200 個與取得同事相關資訊有關，以及 200 個用於傳送電子郵件，則請將這些意圖分組，讓每個群組都在單一應用程式中，然後建立一個包含每個意圖的最上層應用程式。 請使用[分派工具和架構](#dispatch-tool-and-model)來建置最上層應用程式。 然後將您的 Bot 變更成使用階層式呼叫，如[分派教學課程][dispatcher-application-tutorial]中所示。 

## <a name="when-you-need-to-combine-several-luis-and-qna-maker-apps"></a>當您需要將數個 LUIS 與 QnA Maker 應用程式結合時
如果您有數個需要回應 Bot 的 LUIS 和 QnA Maker 應用程式，請使用[分派工具](#dispatch-tool-and-model)來建置最上層應用程式。 然後將您的 Bot 變更成使用階層式呼叫，如[分派教學課程][dispatcher-application-tutorial]中所示。 

## <a name="dispatch-tool-and-model"></a>分派工具和模型
請使用 [Dispatch][dispatch-tool] 命令列工具 (可在 [BotBuilder-tools](https://github.com/Microsoft/botbuilder-tools) 中找到) 將多個 LUIS 和/或 QnA Maker 應用程式結合成一個父 LUIS 應用程式。 此方法可讓您擁有一個包含個別應用程式中所有主體和不同子主體定義域的父定義域。 

![分派架構的概念圖](./media/luis-concept-enterprise/dispatch-architecture.png)

父代網域會在 LUIS 的應用程式清單中以名為 `Dispatch` 的版本註明。 

Chatbot 會接收語句，然後將其傳送給父 LUIS 應用程式來進行預測。 來自父應用程式的最高分預測意圖會決定下一個要呼叫的子 LUIS 應用程式。 Chatbot 會將語句傳送給子應用程式來進行更具體的預測。

請從 Bot Builder v4 [發送器應用程式教學課程][dispatcher-application-tutorial]了解如何建立此呼叫階層。  

### <a name="intent-limits-in-dispatch-model"></a>分派模型中的意圖限制
分派應用程式最多可有 500 個分派來源，相當於 500 個意圖。 

## <a name="next-steps"></a>後續步驟

* 了解如何[測試批次](luis-how-to-batch-test.md)

[dispatcher-application-tutorial]: https://aka.ms/bot-dispatch
[dispatch-tool]: https://aka.ms/dispatch-tool