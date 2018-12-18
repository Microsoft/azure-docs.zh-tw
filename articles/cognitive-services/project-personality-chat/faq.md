---
title: 常見問題集 - 個人化聊天
titlesuffix: Azure Cognitive Services
description: 特質交談常見問題集
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: personality-chat
ms.topic: faq
ms.date: 05/07/2018
ms.author: nolachar
comment: As a bot developer, I want my bot to be able to handle small talk in a consistent tone so that my bot appears more complete and conversational.
ms.openlocfilehash: 274b232f321b3ad739d18a8580b4a178dbde633e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47219816"
---
# <a name="frequently-asked-questions"></a>常見問題集

## <a name="why-doesnt-this-answer-every-question-i-ask-it-like-other-chat-bots"></a>為什麼這不像其他聊天機器人一樣，回答我問它的每個問題？

專案特質交談將會使用一般小型交談來加強機器人，而這些交談會展示特質，以及建立更完整的使用者經驗。 它的設計不是要繼續機器人主要功能無關主題的冗長交談。 雖然它可以回應所有交談，但在測試版中只限制為一般小型交談領域。

## <a name="how-can-i-customize-the-personality-to-suit-my-brand"></a>如何自訂特質以符合我的品牌？

從可用的預設角色中，選取最接近的角色。 您現在可以採用編輯程式庫，並編輯較符合您品牌的回應。 未來，您可以從選擇的特質上傳一組範例語調，並尋找它的最接近角色識別碼版本。 也有方法可以重新訓練和自訂模型。

## <a name="is-this-service-powering-existing-intelligent-agents-such-as-zo"></a>此服務採用 Zo 這類現有智慧型代理程式嗎？

採用 Zo、Cortana 和專案特質交談的服務會共用一些類似的技術，但為不同的堆疊。 它已納入來自 Zo 和 Cortana 體驗的學習。

## <a name="can-this-service-lead-to-bad-customer-experiences"></a>此服務會導致不正確的客戶體驗嗎？

為了提供更豐富的經驗，特質交談可以產生編輯資料集中交談之外的回應，並嘗試解譯所有使用者輸入。 因此，回應在內容中可能看起來不正確。 許多控制項都已在定位，協助防止不適合的回應，並根據 Zo 這類智慧型代理程式的知識建置。 專案特質交談預設為只回應可辨識的使用者意圖。 建議您測試專案特質交談是否適合您的情況。 若您看到任何需要進一步訓練的項目，則十分歡迎提供您的意見反應。 若您未來要與您的客戶一起使用此服務，則建議您考慮匿名化記錄，協助您找出即時使用者互動的問題。