---
title: 意圖辨識範例 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 以下為意圖辨識範例。
services: cognitive-services
author: wolfma61
manager: onano
ms.service: cognitive-services
ms.technology: Speech
ms.topic: article
ms.date: 07/16/2018
ms.author: wolfma
ms.openlocfilehash: 3e9afc990d6bfa73eb045e7ed76dfd194df309c6
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39213193"
---
# <a name="sample-for-intent-recognition"></a>意圖辨識範例

請先取得訂用帳戶金鑰。 相較於認知服務語音 SDK 支援的其他服務，意圖辨識服務需要特定的訂用帳戶金鑰。 您可以在[這裡](https://www.luis.ai)找到更多的意圖辨識技術，以及如何取得訂用帳戶金鑰的資訊。 使用您自己的 Language Understanding 訂用帳戶金鑰、[訂用帳戶區域](regions.md)，以及您意圖模型的應用程式識別碼取代範例中的適當項目。

## <a name="top-level-declarations"></a>最上層宣告

以下所有範例都應該具備下列最上層宣告：

[!code-csharp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#toplevel)]

[!code-cpp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/intent_recognition_samples.cpp#toplevel)]

[!code-java[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/IntentRecognitionSamples.java#toplevel)]

## <a name="intent-recognition-using-microphone"></a>使用麥克風的意圖辨識

下方程式碼片段示範如何以預設語言 (`en-US`) 辨識麥克風語音輸入的意圖。

[!code-csharp[Intent Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentRecognitionWithMicrophone)]

[!code-cpp[Intent Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/intent_recognition_samples.cpp#IntentRecognitionWithMicrophone)]

[!code-java[Intent Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/IntentRecognitionSamples.java#IntentRecognitionWithMicrophone)]

## <a name="intent-recognition-using-microphone-in-a-specified-language"></a>辨識麥克風傳來之指定語言音訊的意圖

下列程式碼片段顯示如何辨識麥克風輸入中的指定語言音訊的意圖，在本例中為德文 (`de-de`)。

[!code-csharp[Intent Recognition Using Microphone In A Specified Language](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentRecognitionWithLanguage)]

[!code-cpp[Intent Recognition Using Microphone In A Specified Language](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/intent_recognition_samples.cpp#IntentRecognitionWithLanguage)]

[!code-java[Intent Recognition Using Microphone In A Specified Language](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/IntentRecognitionSamples.java#IntentRecognitionWithLanguage)]

## <a name="intent-recognition-from-a-file-using-events"></a>使用事件，從檔案進行意圖辨識

程式碼片段顯示如何以連續的方式用預設語言 (`en-US`) 辨識意圖。 此程式碼允許存取其他資訊，例如中繼結果。 輸入取自音訊檔案，支援的格式是取樣率為 16 KHz 的單聲道 (mono) WAV / PCM。

[!code-csharp[Intent Recognition Using Events From a File](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/intent_recognition_samples.cs#intentContinuousRecognitionWithFile)]

[!code-cpp[Intent Recognition Using Events From a File](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/intent_recognition_samples.cpp#IntentContinuousRecognitionWithFile)]

[!code-java[Intent Recognition Using Events From a File](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/IntentRecognitionSamples.java#IntentContinuousRecognitionWithFile)]

[!include[Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>後續步驟

- [語音辨識](./speech-to-text-sample.md)

- [翻譯](./translation.md)
