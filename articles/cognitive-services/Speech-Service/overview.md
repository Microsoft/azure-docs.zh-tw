---
title: 什麼是語音服務？
titleSuffix: Azure Cognitive Services
description: 語音服務會將語音轉文字、文字轉語音及語音翻譯整合至單一 Azure 訂用帳戶。 使用語音 SDK、語音裝置 SDK 或 REST API 將語音新增至您的應用程式、工具和裝置。
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: overview
ms.date: 11/23/2020
ms.author: trbye
ms.openlocfilehash: d3d9f41876cf1310fe25a275624f609031c05b00
ms.sourcegitcommit: fc401c220eaa40f6b3c8344db84b801aa9ff7185
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/20/2021
ms.locfileid: "98601890"
---
# <a name="what-is-the-speech-service"></a>什麼是語音服務？

語音服務會將語音轉文字、文字轉語音及語音翻譯整合至單一 Azure 訂用帳戶。 藉由[語音 CLI](spx-overview.md)、[語音 SDK](./speech-sdk.md)、[語音裝置 SDK](./speech-devices-sdk-quickstart.md?pivots=platform-android)、[Speech Studio](https://speech.microsoft.com/) 或 [REST API](#reference-docs)，可輕易地透過語音來啟用您的應用程式、工具和裝置。

> [!IMPORTANT]
> 語音服務已取代 Bing 語音 API 和翻譯工具語音。 如需移轉說明，請參閱「移轉」一節。

下列功能是語音服務的一部分。 請使用此資料表中的連結，深入了解每項功能的常見使用案例，或瀏覽 API 參考。

| 服務 | 功能 | 描述 | SDK | REST |
|---------|---------|-------------|-----|------|
| [語音轉文字](speech-to-text.md) | 即時語音轉換文字 | 語音轉換文字可將音訊串流或本機檔案即時轉譯或翻譯為的文字，讓您的應用程式、工具或裝置可以取用或顯示。 若搭配 [Language Understanding (LUIS)](../luis/index.yml) 使用語音轉文字，即可從轉譯的語音衍生使用者意圖，以及根據語音命令執行動作。 | [是](./speech-sdk.md) | [是](#reference-docs) |
| | [批次語音轉換文字](batch-transcription.md) | 批次語音轉換文字可針對 Azure Blob 儲存體中的大量語音音訊資料，啟用非同步語音轉換文字轉譯。 除了將語音音訊轉換為文字之外，批次語音轉換文字也可以進行自動分段標記和情感分析。 | 否 | [是](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) |
| | [多裝置交談](multi-device-conversation.md) | 透過便利的轉譯和翻譯支援，在一個對話中連接多個裝置或用戶端以傳送以語音或文字為基礎的訊息| 是 | 否 |
| | [對話轉譯](./conversation-transcription.md) | 啟用即時語音辨識、說話者識別和自動分段標記功能。 非常適合利用辨識說話者的能力來轉譯面對面會議。 | 是 | 否 |
| | [建立自訂語音模型](#customize-your-speech-experience) | 如果您在獨特的環境中使用語音轉文字進行辨識及轉譯，您可以建立並定型自訂原音、語言和發音模型，以處理環境噪音或業界專有詞彙。 | 否 | [是](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) |
| [文字轉換語音](text-to-speech.md) | 文字轉換語音 | 文字轉語音會使用[語音合成標記語言 (SSML)](speech-synthesis-markup.md) 將輸入文字轉換為仿真人的合成語音。 可選擇標準語音和類神經語音 (請參閱[語言支援](language-support.md))。 | [是](./speech-sdk.md) | [是](#reference-docs) |
| | [建立自訂語音](#customize-your-speech-experience) | 建立您品牌或產品專有的自訂聲音音調。 | 否 | [是](#reference-docs) |
| [語音翻譯](speech-translation.md) | 語音翻譯 | 語音翻譯可讓您在應用程式、工具和裝置上使用即時且多語言的語音翻譯。 此服務可用於語音轉語音及語音轉文字翻譯。 | [是](./speech-sdk.md) | 否 |
| [語音助理](voice-assistants.md) | 語音助理 | 使用語音服務的語音助理能賦予開發人員建立自然、擬人的對話介面，供其應用程式和體驗之用。 語音助理服務能在裝置和助理實作之間提供迅速且可靠的互動；該助理實作會使用 Bot Framework 的 Direct Line 語音頻道，或是整合的自訂命令服務來完成工作。 | [是](voice-assistants.md) | 否 |
| [說話者辨識](speaker-recognition-overview.md) | 說話者驗證與識別 | 說話者辨識服務提供以唯一語音特性驗證和識別說話者的演算法。 說話者辨識是用來回答「誰在說話？」的問題。 | 是 | [是](/rest/api/speakerrecognition/) |


[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

## <a name="try-the-speech-service-for-free"></a>免費試用語音服務

在下列步驟中，您同時需要 Microsoft 帳戶和 Azure 帳戶。 如果您沒有 Microsoft 帳戶，則可以在 [Microsoft 帳戶入口網站](https://account.microsoft.com/account)免費註冊。 選取 [使用 Microsoft 帳戶登入]，然後在系統要求您登入時選取 [建立 Microsoft 帳戶]。 依照步驟來建立及驗證新的 Microsoft 帳戶。

有了 Microsoft 帳戶之後，請移至 [Azure 註冊頁面](https://azure.microsoft.com/free/ai/)、選取 [免費開始]，然後使用 Microsoft 帳戶建立新的 Azure 帳戶。 以下是[如何註冊 Azure 免費帳戶](https://www.youtube.com/watch?v=GWT2R1C_uUU)的影片。

> [!NOTE]
> 當您註冊免費 Azure 帳戶時，其隨附 200 美元的服務點數，您可以將此點數應用於有效時間長達 30 天的付費語音服務訂用帳戶。 如果點數用完或在 30 天結束時過期，Azure 服務就會停用。 若要繼續使用 Azure 服務，則必須將您的帳戶升級。 如需詳細資訊，請參閱[如何升級您的 Azure 免費帳戶](../../cost-management-billing/manage/upgrade-azure-subscription.md)。 
>
> 語音服務有兩個服務層級：免費 (f0) 和訂用帳戶 (s0)，其各有不同的限制和權益。 如果您使用免費的低容量語音服務層級，則即使在免費試用或服務點數到期後，您仍可保留此免費訂用帳戶。 如需詳細資訊，請參閱[認知服務定價 - 語音服務](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)。

### <a name="create-the-azure-resource"></a>建立 Azure 資源

若要將語音服務資源 (免費或付費層) 新增至您的 Azure 帳戶：

1. 使用您的 Microsoft 帳戶，登入 [Azure 入口網站](https://portal.azure.com/)。

1. 選取入口網站左上方的 [建立資源]。 如果您沒有看到 [建立資源]，則隨時都可以在畫面左上角選取摺疊的功能表來找到。

1. 在 [新增] 視窗中，於搜尋方塊中輸入「語音」，然後按 ENTER。

1. 在搜尋結果中，選取 [Speech]。

   ![語音搜尋結果](media/index/speech-search.png)

1. 選取 [建立]，然後：

   - 為您的新資源提供唯一的名稱。 此名稱可協助您區分繫結至相同服務的多個訂用帳戶。
   - 選擇與新資源相關聯的 Azure 訂用帳戶來決定費用的計費方式。 這是[如何在 Azure 入口網站中建立 Azure 訂用帳戶](../../cost-management-billing/manage/create-subscription.md#create-a-subscription-in-the-azure-portal)的簡介。
   - 選擇將使用此資源的[區域](regions.md)。 Azure 是在全球各地許多地區正式推出的全域雲端平台。 若要取得最佳效能，請選取最接近您或您應用程式執行位置的區域。 語音服務的可用性會因不同區域而有所差異。 請確定您是在支援的區域中建立資源。 請參閱[語音服務的區域支援](./regions.md#speech-to-text-text-to-speech-and-translation)。
   - 選擇免費 (F0) 或付費 (S0) 的定價層。 如需每一層的定價和使用量配額完整資訊，請選取 [檢視完整定價詳細資料]，或是參閱[語音服務定價](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/)。 如需了解資源的限制，請參閱 [Azure 認知服務限制](../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cognitive-services-limits)。
   - 為此語音訂用帳戶建立新的資源群組，或將該訂用帳戶指派給現有的資源群組。 資源群組可協助組織各種 Azure 訂用帳戶。
   - 選取 [建立]  。 這會帶您前往部署概觀並顯示部署進度訊息。  
<!--
> [!NOTE]
> You can create an unlimited number of standard-tier subscriptions in one or multiple regions. However, you can create only one free-tier subscription. Model deployments on the free tier that remain unused for 7 days will be decommissioned automatically.
-->
部署新的語音資源需要幾分鐘的時間。 

### <a name="find-keys-and-region"></a>尋找金鑰和區域

若要尋找已完成部署的金鑰和區域，請遵循下列步驟：

1. 使用您的 Microsoft 帳戶，登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [所有資源]，然後選取認知服務資源的名稱。

3. 在左側窗格中，於 **資源管理** 下選取 [金鑰和端點]。

每個訂用帳戶都有兩個金鑰，您可以在應用程式中使用任一個金鑰。 若要將金鑰複製/貼到您的程式碼編輯器或其他位置，請選取每個金鑰旁的 [複製] 按鈕，切換視窗將剪貼簿內容貼到所需的位置。

此外，請複製 `LOCATION` 值，也就是您的區域識別碼 (例如 `westus`、`westeurope`) 以呼叫 SDK。

> [!IMPORTANT]
> 這些訂用帳戶金鑰可用來存取您的認知服務 API。 請勿共用您的金鑰。 安全地加以儲存，例如使用 Azure Key Vault。 我們也建議您定期重新產生這些金鑰。 進行 API 呼叫時，只需要一個金鑰。 重新產生第一個金鑰時，您可以使用第二個金鑰繼續存取服務。

## <a name="complete-a-quickstart"></a>完成快速入門

我們會以最熱門的程式設計語言提供快速入門，目的是教您基本的設計模式，並讓您能在 10 分鐘內執行程式碼。 請參閱下列清單，以取得每項功能的快速入門。

* [語音轉換文字快速入門](get-started-speech-to-text.md)
* [文字轉換語音快速入門](get-started-text-to-speech.md)
* [語音翻譯快速入門](./get-started-speech-translation.md)
* [意圖辨識快速入門](quickstarts/intent-recognition.md)
* [說話者辨識快速入門](./get-started-speaker-recognition.md)

在您有機會開始使用語音服務之後，請嘗試我們的教學課程，其可說明如何解決各種案例。

- [教學課程：使用語音 SDK 和 LUIS 從語音辨識意圖 (C#)](how-to-recognize-intents-from-speech-csharp.md)
- [教學課程：使用語音 SDK 和 C# 透過聲音啟用 Bot](tutorial-voice-enable-your-bot-speech-sdk.md)
- [教學課程：建置 Flask 應用程式以翻譯文字、分析情緒並將翻譯的文字合成語音，REST](../translator/tutorial-build-flask-app-translation-synthesis.md?bc=%2fazure%2fcognitive-services%2fspeech-service%2fbreadcrumb%2ftoc.json%252c%2fen-us%2fazure%2fbread%2ftoc.json&toc=%2fazure%2fcognitive-services%2fspeech-service%2ftoc.json%252c%2fen-us%2fazure%2fcognitive-services%2fspeech-service%2ftoc.json)

## <a name="get-sample-code"></a>取得範例程式碼

您可以在 GitHub 上取得語音服務的程式碼範例。 這些範例包含常見案例，例如：從檔案或資料流讀取音訊、連續辨識、一次性辨識及使用自訂模型。 使用下列連結來檢視 SDK 和 REST 範例：

- [語音轉文字、文字轉語音及語音翻譯範例 (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [批次轉譯範例 (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [文字轉語音範例 (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)
- [語音助理範例 (SDK)](https://aka.ms/csspeech/samples) \(英文\)

## <a name="customize-your-speech-experience"></a>自訂語音體驗

語音服務可順利地與內建模型搭配使用，不過，您可以進一步自訂及調整體驗，以搭配您的產品或環境。 從原音模型調整到專屬於自身品牌的獨特聲音音調，都是自訂選項的範圍。

其他產品則會提供專為醫療保健或保險等特定用途而調整的語音模型，但可供所有人平等地使用。 Azure 語音中的自訂會成為 *您的獨特* 競爭優勢一部分，而其他使用者或客戶則無法使用。 換句話說，您的模型是私人的，而且只會針對您的使用案例進行自訂調整。

| 語音服務 | 平台 | 描述 |
| -------------- | -------- | ----------- |
| 語音轉文字 | [客製化的語音](https://aka.ms/customspeech) | 依據您的需求和可用的資料，自訂語音辨識模型。 克服像是語音模式、詞彙及背景雜音等語音辨識的阻礙。 |
| 文字轉語音 | [自訂語音](https://aka.ms/customvoice) | 使用您的談話資料，為您的文字轉語音應用程式建置可辨識且獨一無二的語音。 您可以藉由調整一組語音參數，來進一步微調語音輸出。 |

## <a name="deploy-on-premises-using-docker-containers"></a>使用 Docker 容器在內部部署環境進行部署

[使用語音服務容器](speech-container-howto.md)在內部部署環境中部署 API 功能。 這些 Docker 容器可讓服務更加契合您的資料，以實現合規性、安全性或其他操作性原因。 語音服務提供下列容器：

* 標準語音轉換文字
* 自訂語音轉換文字
* 標準文字轉換語音
* 類神經文字轉換語音
* 自訂文字轉換語音 (預覽)
* 語音語言偵測 (預覽)

## <a name="reference-docs"></a>參考文件

- [語音 SDK](./speech-sdk.md)
- [語音裝置 SDK](speech-devices-sdk.md)
- [REST API：語音轉文字](rest-speech-to-text.md)
- [REST API：文字轉語音](rest-text-to-speech.md)
- [REST API：批次轉譯與自訂](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)


## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [開始使用語音轉換文字](./get-started-speech-to-text.md)
> [開始使用文字轉換語音](get-started-text-to-speech.md)