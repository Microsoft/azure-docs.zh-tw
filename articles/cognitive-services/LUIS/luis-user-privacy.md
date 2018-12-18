---
title: 匯出和刪除客戶資料 - Language Understanding - LUIS
titleSuffix: Azure Cognitive Services
description: Language Understanding Intelligent Service (LUIS) 可保存客戶內容以執行此服務，但 LUIS 使用者具有檢視、匯出和刪除其資料的完整控制權。 此作業可透過 LUIS Web 入口網站或 LUIS 程式設計 API 來完成。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: conceptual
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: bfd168d5c56bc800831c402a23bb1a506fa15221
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883885"
---
# <a name="export-and-delete-your-customer-data-in-language-understanding-luis-in-cognitive-services"></a>在認知服務的 Language Understanding (LUIS) 中匯出和刪除客戶資料

## <a name="summary-of-customer-data-request-features"></a>客戶資料要求功能的摘要
Language Understanding Intelligent Service (LUIS) 可保存客戶內容以執行此服務，但 LUIS 使用者具有檢視、匯出和刪除其資料的完整控制權。 此作業可透過 LUIS Web [入口網站](luis-reference-regions.md)或 [LUIS 程式設計 API](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f) 來完成。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

客戶的內容會加密並儲存在 Microsoft 區域 Azure 儲存體中，且包含：

- 註冊時收集的使用者帳戶內容
- 建置模型所需的訓練資料 (也就是目的和實體)
- 在執行階段記錄以利改善使用者模型的使用者查詢
  - 使用者可藉由將 `&log=false`附加至要求以關閉查詢記錄，詳情請見[這裡](luis-resources-faq.md#how-can-i-disable-the-logging-of-utterances)

## <a name="deleting-customer-data"></a>刪除客戶資料
LUIS 使用者有完整的控制權可刪除任何使用者內容 (透過 LUIS Web 入口網站或 LUIS 程式設計 API)。 下表顯示這兩者的輔助連結：

| | **使用者帳戶** | **應用程式** | **語句** | **使用者查詢** |
| --- | --- | --- | --- | --- |
| **入口網站** | [連結](luis-how-to-account-settings.md) | [連結](luis-how-to-start-new-app.md#delete-app) | [連結](luis-how-to-start-new-app.md#delete-app) | [連結](luis-how-to-start-new-app.md#delete-app) |
| **API** | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c4c) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c39) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0b) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/58b6f32139e2bb139ce823c9) |


## <a name="exporting-customer-data"></a>匯出客戶資料
LUIS 使用者有完整的控制權可檢視入口網站上的資料，但必須透過 LUIS 程式設計 API 才可加以匯出。 下表顯示透過 LUIS 程式設計 API 進行資料匯出的輔助連結：

| | **使用者帳戶** | **應用程式** | **語句** | **使用者查詢** |
| --- | --- | --- | --- | --- |
| **API** | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c48) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c0a) | [連結](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c36) |


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [LUIS 區域參考](./luis-reference-regions.md)
