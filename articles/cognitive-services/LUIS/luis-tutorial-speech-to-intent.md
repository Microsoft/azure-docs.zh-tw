---
title: 使用語音 C# SDK 搭配 LUIS
titleSuffix: Azure Cognitive Services
description: 語音服務可讓您使用單一要求接收音訊並傳回 LUIS 預測 JSON 物件。 在本文中，您將在 Visual Studio 中下載與使用 C# 專案，對麥克風說話，並接收 LUIS 預測資訊。 該專案會使用語音 NuGet 封裝 (已隨附提供參考)。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: tutorial
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: f98d640f032fed5f91df8e9d4fb55d3f20550339
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883919"
---
# <a name="integrate-speech-service"></a>整合語音服務
[語音服務](https://docs.microsoft.com/azure/cognitive-services/Speech-Service/)可讓您使用單一要求接收音訊並傳回 LUIS 預測 JSON 物件。 在本文中，您將在 Visual Studio 中下載與使用 C# 專案，對麥克風說話，並接收 LUIS 預測資訊。 該專案會使用語音 [NuGet](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech/) 封裝 (已隨附提供參考)。 

為了操作本文的作業，您需要免費的 [LUIS][LUIS] 網站帳戶才能匯入應用程式。

## <a name="create-luis-endpoint-key"></a>建立 LUIS 端點金鑰
在 Azure 入口網站中，[建立 ](luis-how-to-azure-subscription.md#create-luis-endpoint-key)**Language Understanding** (LUIS) 金鑰。 

## <a name="import-human-resources-luis-app"></a>匯入 Human Resources LUIS 應用程式
本文中的意圖和語句，皆來自位於 [LUIS-Samples](https://github.com/Microsoft/LUIS-Samples) Github 存放庫中的 Human Resources LUIS 應用程式。 下載 [HumanResources.json](https://github.com/Microsoft/LUIS-Samples/blob/master/documentation-samples/tutorials/HumanResources.json) 檔案、以 `.json` 副檔名儲存該檔案，然後將它[匯入](luis-how-to-start-new-app.md#import-new-app) LUIS。 

此應用程式具有與人力資源領域相關的意圖、實體和語句。 語句範例包括：

|範例語句|
|--|
|Who is John Smith's manager?|
|Who does John Smith manage?|
|Where is Form 123456?|
|Do I have any paid time off?|


## <a name="add-keyphrase-prebuilt-entity"></a>新增 KeyPhrase 預先建置的實體
匯入應用程式後，選取 [實體]，然後再選取 [管理預先建立的實體]。 新增 **KeyPhrase** 實體。 KeyPhrase 實體會從語句中擷取金鑰主題。

## <a name="train-and-publish-the-app"></a>訓練和發佈應用程式
1. 在頂端右側的導覽列中，選取 [訓練] 按鈕以訓練 LUIS 應用程式。

2. 選取右上列中的 [管理]，然後選取左側導覽列中的 [金鑰和端點]。 

3. 在 [金鑰和端點] 頁面上，指派在[建立 LUIS 端點金鑰](#create-luis-endpoint-key)一節中建立的 LUIS 金鑰。

  在此頁面上，收集在[建立 LUIS 端點金鑰](#create-luis-endpoint-key)一節中所建立 LUIS 金鑰的應用程式識別碼、發佈區域和訂閱識別碼。 您必須修改程式碼，以在本文後半部使用這些值。 
  
  **請勿**使用免費的入門版金鑰進行此練習。 只有在 Azure 入口網站中建立的 **Language Understanding** 金鑰適用於此練習。 

  https://**REGION**.api.cognitive.microsoft.com/luis/v2.0/apps/**APPID**?subscription-key=**LUISKEY**&q=


4. 選取右上列中的 [發佈] 按鈕，發佈 LUIS 應用程式。 

## <a name="audio-device"></a>音訊裝置
本文將需要您使用電腦上的音訊裝置。 像是有麥克風的耳機，或是內建的音訊裝置。 檢查音訊輸入量，以了解您得音量是否應該比平常說話大聲，才能讓音訊裝置偵測到您的語音。 

## <a name="download-the-luis-sample-project"></a>下載 LUIS Sample 專案
 複製或下載 [LUIS-Samples](https://github.com/Microsoft/LUIS-Samples) 存放庫。 透過 Visual Studio 開啟[語音轉意圖專案](https://github.com/Microsoft/LUIS-Samples/tree/master/documentation-samples/tutorial-speech-intent-recognition)，然後還原 NuGet 封裝。 VS 解決方案檔案位於 .\LUIS-Samples-master\documentation-samples\tutorial-speech-intent-recognition\csharp\csharp_samples.sln 中。

已隨附語音 SDK 供參考。 

[![](./media/luis-tutorial-speech-to-intent/nuget-package.png "顯示 Microsoft.CognitiveServices.Speech NuGet 封裝的 Visual Studio 2017 螢幕擷取畫面")](./media/luis-tutorial-speech-to-intent/nuget-package.png#lightbox)

## <a name="modify-the-c-code"></a>修改 C# 程式碼
開啟 `Program.cs` 檔案，並變更下列變數：

|變數名稱|目的|
|--|--|
|LUIS_assigned_endpoint_key|從 [發佈] 頁面對應到端點 URL 的指派 subscription-key 值|
|LUIS_endpoint_key_region|對應到端點 URL 的第一個子網域，例如 `westus`|
|LUIS_app_ID|對應到 **apps/** 之後的端點 URL 的路由|

`Program.cs` 檔案已經對應 Human Resources 意圖。

建置並執行應用程式。 

## <a name="test-code-with-utterance"></a>使用語句測試程式碼
對麥克風說「Redmond 的合格牙醫有誰？」。

[!code-console[Command line response from spoken utterance](~/samples-luis/documentation-samples/tutorial-speech-intent-recognition/console-output.txt "Command line response from spoken utterance")]

系統找到正確意圖 **GetEmployeeBenefits**，信賴度 85%。 系統傳回 keyPhrase 實體。 

語音 SDK 會傳回整個 LUIS 回應。 

## <a name="clean-up-resources"></a>清除資源
若不再需要 LUIS HumanResources 應用程式，請予以刪除。 若要這樣做，請選取應用程式，然後在清單上方的內容相關工具列中，選取 [刪除]。 在 [Delete app?] \(刪除應用程式?\) 快顯對話方塊上，選取 [Ok] \(確定\)。

用完範例程式碼後，請記得刪除 LUIS-Samples 目錄。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [將 LUIS 與 BOT 整合](luis-csharp-tutorial-build-bot-framework-sample.md)

[LUIS]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions#luis-website