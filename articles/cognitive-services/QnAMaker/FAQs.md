---
title: 常見問題集 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 列出 QnA Maker 服務的的常見問題集
services: cognitive-services
author: tulasim88
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: article
ms.date: 09/12/2018
ms.author: tulasim
ms.openlocfilehash: efefd595c43d7f46ff1ead91577d070cf8fb90e4
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47164611"
---
# <a name="frequently-asked-questions"></a>常見問題集

## <a name="why-is-my-urlsfiles-is-not-extracting-question-answer-pairs"></a>為何我的 URL/檔案不會擷取問答組？

有可能是因為 QnA Maker 無法從有效的常見問題集 URL 自動擷取某些問答 (QnA) 內容。 在這種情況下，您可以將 QnA 內容貼到 .txt 檔案中，並確認工具是否可加以擷取。 或者，您也可以藉由編輯將內容新增至知識庫。

## <a name="how-large-a-knowledge-base-can-i-create"></a>我可建立多大的知識庫？

知識庫的大小取決於您在建立 QnA Maker 服務時所選擇的 Azure 搜尋服務 SKU。 如需詳細資訊，請參閱[這裡](./Tutorials/choosing-capacity-qnamaker-deployment.md)。

## <a name="why-do-i-not-see-anything-in-the-drop-down-for-when-i-try-to-create-a-new-knowledge-base"></a>當我嘗試建立新的知識庫，為何我在下拉式清單中未看到任何項目？

您尚未在 Azure 中建立任何 QnA Maker 服務。 請參閱[這裡](./How-To/set-up-qnamaker-service-azure.md)以了解如何執行這些作業。

## <a name="how-do-i-share-a-knowledge-base-with-other"></a>如何與他人共用知識庫？

共用會在 QnA Maker 服務層級上運作，也就是說，將會共用服務中的所有知識庫。 請參閱[這裡](./How-To/collaborate-knowledge-base.md)以了解如何在知識庫上共同作業。

## <a name="how-can-i-change-the-default-message-when-no-good-match-is-found"></a>如何變更找不到適當相符項目時的預設訊息？

預設訊息是您 App Service 中的部分設定。
- 在 Azure 入口網站中移至您的 QnA App Service 資源

![qnamaker appservice](./media/qnamaker-faq/qnamaker-resource-list-appservice.png)
- 按一下 [設定] 選項

![qnamaker appservice 設定](./media/qnamaker-faq/qnamaker-appservice-settings.png)
- 變更 **DefaultAnswer** 設定的值
- 重新啟動 App Service

![qnamaker appservice 重新啟動](./media/qnamaker-faq/qnamaker-appservice-restart.png)

## <a name="why-is-my-sharepoint-link-not-getting-extracted"></a>為什麼我的 SharePoint 連結無法供擷取？

此工具只會剖析公用 URL，且目前並不支援已驗證的資料來源。 或者，您可以下載檔案，並使用檔案上傳選項來擷取問題和解答。


## <a name="the-updates-that-i-made-to-my-knowledge-base-are-not-reflected-on-publish-why-not"></a>我對知識庫所做的更新並未在發佈時反映。 為什麼？

無論是資料表更新、測試或設定，每項編輯作業都必須先儲存才能加以發佈。 在每次編輯作業完成後，請務必按一下 [儲存並訓練] 按鈕。

## <a name="when-should-i-refresh-my-endpoint-keys"></a>何時應重新整理端點金鑰？

若您懷疑端點金鑰已遭入侵，請加以重新整理。

## <a name="does-the-knowledge-base-support-rich-data-or-multimedia"></a>知識庫是否支援豐富的資料或多媒體？

知識庫支援 Markdown。 然而，URL 的自動擷取限制了從 HTML 轉換至 Markdown 的功能。 您可直接在資料表中修改內容或使用豐富內容上傳知識庫，即可使用完備的 Markdown。

目前並不支援多媒體 (例如影像和視訊)。

## <a name="does-qna-maker-support-non-english-languages"></a>QnA Maker 是否支援非英文的語言？

請參閱更多關於[支援的語言](./Overview/languages-supported.md)的詳細資料。

如果您有多種語言的內容，請務必為每一種語言建立個別的服務。

## <a name="do-i-need-to-use-bot-framework-in-order-to-use-qna-maker"></a>我是否需要使用 Bot Framework 才能使用 QnA Maker？

否，您不需要將 Bot Framework 用於 QnA Maker。 不過，Azure Bot 服務會以數個範本之一的形式提供 QnA Maker。 Bot Service 可透過 Microsoft Bot Framework 快速開發智慧型 Bot，並且可在無伺服器環境中執行。

## <a name="how-can-i-create-a-bot-with-qna-maker"></a>如何使用 QnA Maker 建立 Bot？

遵循[這份](./Tutorials/create-qna-bot.md)文件中的指示，使用 Azure Bot 建立您的 Bot。

## <a name="how-do-i-embed-the-qna-maker-service-in-my-website"></a>我該如何在我的網站中內嵌 QnA Maker 服務？

依照以下步驟，將 QnA Maker 服務作為網站聊天控制項內嵌至您的網站中：

1. 依照[這裡](./Tutorials/create-qna-bot.md)的指示建立您的常見問題集 Bot。
2. 依照[這裡](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat)的步驟啟用網路聊天


