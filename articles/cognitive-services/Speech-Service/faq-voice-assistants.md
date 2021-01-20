---
title: 語音助理的常見問題
titleSuffix: Azure Cognitive Services
description: 取得使用自訂命令或 Direct Line 語音通道的語音助理最熱門問題的解答。
services: cognitive-services
author: trrwilson
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/05/2019
ms.author: travisw
ms.openlocfilehash: 4b0bbb982ed48dc052b1a15514ad36b1d69b62b5
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98599425"
---
# <a name="voice-assistants-frequently-asked-questions"></a>語音助理的常見問題

如果您在此檔中找不到問題的解答，請參閱 [其他支援選項](../cognitive-services-support-options.md?context=%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext%253fcontext%253d%2fazure%2fcognitive-services%2fspeech-service%2fcontext%2fcontext)。

## <a name="general"></a>一般

**問：什麼是語音助理？**

**答：** 就像 Cortana 一樣，語音助理是一個解決方案，可接聽使用者的語音語句、分析這些語句的內容，以回應語句的意圖，然後針對通常包含語音元件的使用者提供回應。 這是與系統互動的「語音輸入、語音輸出」體驗。 語音助理作者會使用語音 SDK 中的來建立裝置上的應用程式， `DialogServiceConnector` 以與使用 [自訂命令](custom-commands.md) 或 Bot Framework 的 [Direct Line 語音](direct-line-speech.md) 通道所建立的小幫手進行通訊。 這些助理可以使用自訂關鍵字、自訂語音和自訂語音，以提供針對您的品牌或產品量身打造的體驗。

**問：我應該使用自訂命令或 Direct Line 語音？有何差異？**

**答：** [自訂命令](custom-commands.md) 是一組較低複雜性的工具，可輕鬆地建立及裝載非常適合工作完成案例的小幫手。 [Direct Line Speech](direct-line-speech.md) 提供更豐富、更精密的功能，可啟用健全的對話式案例。 如需詳細資訊，請參閱 [助理解決方案的比較](voice-assistants.md#choosing-an-assistant-solution) 。

**問：如何開始使用？**

**答：** 開始建立自訂命令 (預覽) 應用程式或基本 Bot Framework Bot 的最佳方式。

- [ (預覽版) 應用程式建立自訂命令](./quickstart-custom-commands-application.md)
- [建立基本 Bot Framework Bot](/azure/bot-service/bot-builder-tutorial-basic-deploy?view=azure-bot-service-4.0)
- [將 bot 連線至 Direct Line 語音通道](/azure/bot-service/bot-service-channel-connect-directlinespeech)

## <a name="debugging"></a>偵錯

**問：我的通道秘密在哪裡？**

**答：** 如果您已使用 Direct Line Speech preview 版本，或您正在閱讀相關的檔，您可能會想要在 Direct Line 語音通道註冊頁面上找到秘密金鑰。 `DialogServiceConfig`語音 SDK 中的 1.7 factory 方法 `FromBotSecret` 也需要此值。

Direct Line Speech 的最新版本可簡化從裝置聯繫 bot 的程式。 在 [通道註冊] 頁面上，頂端的下拉式清單會將您的 Direct Line 語音通道註冊與語音資源產生關聯。 一旦建立關聯之後， `BotFrameworkConfig::FromSubscription` 就會包含一個 factory 方法，以將設定 `DialogServiceConnector` 為與您的訂用帳戶相關聯的 bot。

如果您仍在將用戶端應用程式從1.7 版遷移至1.8 版，則 `DialogServiceConfig::FromBotSecret` 可以繼續使用非空白的非 null 值作為通道秘密參數，例如您使用的先前秘密。 使用與較新通道註冊相關聯的語音訂用帳戶時，只會忽略它。 請注意，此值 _必須_ 為非 null 和非空白，因為這些值會在服務端關聯相關之前，在裝置上進行檢查。

如需更詳細的指南，請參閱逐步解說通道註冊的 [教學課程一節](tutorial-voice-enable-your-bot-speech-sdk.md#register-the-direct-line-speech-channel) 。

**問：當連接時，我收到401錯誤，沒有任何作用。我知道我的語音訂用帳戶金鑰有效。這是怎麼回事？**

**答：** 在 Azure 入口網站管理您的訂用帳戶時，請確定您使用的是 **語音** 資源 (Microsoft. CognitiveServicesSpeechServices、"Speech" ) 而 _非_**認知服務** 資源 (CognitiveServicesAllInOne "所有認知服務" ) 。 此外，請查看語音 [助理的語音服務區域支援](regions.md#voice-assistants)。

![正確的 direct line speech 訂用帳戶](media/voice-assistants/faq-supported-subscription.png "相容語音訂用帳戶的範例")

**問：我從我那裡取得了辨識文字 `DialogServiceConnector` ，但我看到的是「1011」錯誤，而不是來自 bot 的任何內容。為什麼？**

**答：** 此錯誤表示您的助理與語音助理服務之間發生通訊問題。

- 針對自訂命令，請確定已發佈您的自訂命令應用程式
- 針對 Direct Line 語音，請確定您已將 [bot 連線到 Direct Line 語音通道](/azure/bot-service/bot-service-channel-connect-directlinespeech)、將 [串流通訊協定支援新增](/azure/bot-service/directline-speech-bot) 至您的 bot (並) 相關的 Web 通訊端支援，然後檢查您的 bot 是否回應來自通道的連入要求。

**問：此程式碼仍無法運作，且（或）在使用時遇到不同的錯誤 `DialogServiceConnector` 。我該怎麼做？**

**答：** 以檔案為基礎的記錄提供更詳細的資料，並可協助加速支援要求。 若要啟用此功能，請參閱 [如何使用檔案記錄](how-to-use-logging.md)。

## <a name="next-steps"></a>後續步驟

- [疑難排解](troubleshooting.md)
- [版本資訊](releasenotes.md)