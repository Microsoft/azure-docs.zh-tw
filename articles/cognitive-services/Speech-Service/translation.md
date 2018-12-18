---
title: 翻譯的範例 | Microsoft Docs
titleSuffix: Microsoft Cognitive Services
description: 以下是語音翻譯的範例。
services: cognitive-services
author: wolfma61
manager: onano
ms.service: cognitive-services
ms.technology: Speech
ms.topic: article
ms.date: 07/16/2018
ms.author: wolfma
ms.openlocfilehash: 66d26181334a71578f1a94000cb942a6a87398bc
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39070786"
---
# <a name="sample-for-translation"></a>翻譯的範例

[!include[Get a Subscription Key](../../../includes/cognitive-services-speech-service-get-subscription-key.md)]

## <a name="top-level-declarations"></a>最上層宣告

以下所有範例都應該具備下列最上層宣告：

[!code-csharp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/translation_samples.cs#toplevel)]

[!code-cpp[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/translation_samples.cpp#toplevel)]

[!code-java[Top-level declarations](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/TranslationSamples.java#toplevel)]

## <a name="translation-using-the-microphone"></a>使用麥克風的翻譯

下列程式碼片段示範如何將英文的語音輸入翻譯為德文，同時取得已翻譯文字的語音輸出。 它會使用麥克風。

[!code-csharp[Translation Using Microphone](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/translation_samples.cs#TranslationWithMicrophoneAsync)]

[!code-cpp[Translation Using Microphone](~/samples-cognitive-services-speech-sdk/samples/cpp/windows/console/samples/translation_samples.cpp#TranslationWithMicrophone)]

[!code-java [Translation Using Microphone](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/TranslationSamples.java#TranslationWithMicrophoneAsync)]

## <a name="translation-using-file-input"></a>使用檔案輸入的翻譯

下列程式碼片段示範如何從英文的語音輸入翻譯為德文和法文。
它會使用檔案作為輸入。

[!code-csharp[Translation Using File Input](~/samples-cognitive-services-speech-sdk/samples/csharp/sharedcontent/console/translation_samples.cs#TranslationWithFileAsync)]

[!code-java [Translation Using File Input](~/samples-cognitive-services-speech-sdk/samples/java/jre/console/src/com/microsoft/cognitiveservices/speech/samples/console/TranslationSamples.java#TranslationWithFileAsync)]

[!include[Download the sample](../../../includes/cognitive-services-speech-service-speech-sdk-sample-download-h2.md)]

## <a name="next-steps"></a>後續步驟

- [語音辨識](./speech-to-text-sample.md)

- [意圖辨識](./intent.md)
