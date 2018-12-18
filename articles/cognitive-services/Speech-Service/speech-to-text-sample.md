---
title: 語音轉換文字的範例 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 以下是語音轉換文字的範例。
services: cognitive-services
author: wolfma61
manager: onano
ms.service: cognitive-services
ms.technology: Speech
ms.topic: article
ms.date: 06/07/2018
ms.author: wolfma
ms.openlocfilehash: b08b461ceccbdf5e79d7bb4320bdc15d09c4b859
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39114237"
---
# <a name="sample-for-speech-to-text"></a>語音轉換文字的範例

[!include[Get a Subscription Key](../../../includes/cognitive-services-speech-service-get-subscription-key.md)]

## <a name="top-level-declarations"></a>最上層宣告

以下所有範例都應該具備下列最上層宣告：

[!code-csharp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#toplevel)]

[!code-cpp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#toplevel)]

[!code-java[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#toplevel)]

## <a name="speech-recognition-using-the-microphone"></a>使用麥克風的語音辨識

下列程式碼片段示範如何以預設語言 (`en-US`) 辨識來自麥克風的語音輸入。

[!code-csharp[Speech Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#recognitionWithMicrophone)]

[!code-cpp[Speech Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#SpeechRecognitionWithMicrophone)]

[!code-java[Speech Recognition Using Microphone](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#recognitionWithMicrophone)]

## <a name="speech-recognition-using-a-customized-model"></a>使用自訂模型的語音辨識

[自訂語音服務 (CRIS)](https://www.cris.ai/) 允許為您的應用程式自訂 Microsoft 語音轉換文字引擎。 下列程式碼片段示範如何使用 CRIS 模型辨識來自麥克風的語音；在執行它之前，請填入您的 CRIS 訂用帳戶金鑰與您自己的部署識別。

[!code-csharp[Speech Recognition Using a Customized Model](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#recognitionCustomized)]

[!code-cpp[Speech Recognition Using a Customized Model](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#SpeechRecognitionUsingCustomizedModel)]

[!code-java[Speech Recognition Using a Customized Model](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#recognitionCustomized)]

## <a name="continuous-speech-recognition-from-a-file"></a>從檔案持續進行的語音辨識

下列程式碼片段會以預設語言 (`en-US`) 持續辨識來自音訊檔案的語音輸入，支援的格式是取樣率為 16 KHz 的單一頻道 (單聲道) WAV/PCM。

[!include[Sample Audio](../../../includes/cognitive-services-speech-service-sample-audio.md)]

[!code-csharp[Continuous Speech Recognition](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/speech_recognition_samples.cs#recognitionContinuousWithFile)]

[!code-cpp[Continuous Speech Recognition](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/speech_recognition_samples.cpp#SpeechContinuousRecognitionWithFile)]

[!code-java[Continuous Speech Recognition](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/SpeechRecognitionSamples.java#recognitionContinuousWithFile)]

[!include[Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>後續步驟

- [意圖辨識](./intent.md)

- [翻譯](./translation.md)
