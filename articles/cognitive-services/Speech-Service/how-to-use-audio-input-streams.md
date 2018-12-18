---
title: 語音 SDK 音訊輸入資料流概念
description: 語音 SDK 音訊輸入資料流 API 功能概觀。
titleSuffix: Microsoft Cognitive Services
services: cognitive-services
author: fmegen
ms.service: cognitive-services
ms.component: speech-service
ms.topic: article
ms.date: 09/24/2018
ms.author: fmegen
ms.openlocfilehash: 6c2d7c5787305f60b73ab83ea17367b04e03ac12
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985175"
---
# <a name="about-the-speech-sdk-audio-input-stream-api"></a>關於語音 SDK 音訊輸入資料流 API

語音 SDK 的**音訊輸入資料流** API 提供將音訊資料流串流到辨識器中的方法，而不需使用麥克風或輸入檔 API。

使用音訊輸入資料流時，必須執行下列步驟：

- 識別音訊資料流的格式。 語音 SDK 和語音服務必須支援此格式。 目前僅支援下列設定：

  PCM 格式、一個通道、每秒 16000 個樣本、每秒 32000 個位元組、兩個區塊對齊 (一個樣本 16 位元，包含填補)、每個樣本 16 位元 的音訊樣本。

  SDK 中用於建立音訊格式相對應程式碼看起來像這樣：

  ```
  byte channels = 1;
  byte bitsPerSample = 16;
  int samplesPerSecond = 16000;
  var audioFormat = AudioStreamFormat.GetWaveFormatPCM(samplesPerSecond, bitsPerSample, channels);
  ```

- 請確定您的程式碼可以根據這些規格提供未經處理的音訊資料。 如果您的音訊來源資料不符合支援的格式，則必須將音訊轉碼成所需的格式。

- 建立自己衍生自 `PullAudioInputStreamCallback` 的音訊輸入資料流類別。 實作`Read()` 和 `Close()` 成員。 確切的函式簽章視語言而定，但程式碼看起來類似此程式碼範例：

  ```
   public class ContosoAudioStream : PullAudioInputStreamCallback {
      ContosoConfig config;

      public ContosoAudioStream(const ContosoConfig& config) {
          this.config = config;
      }

      public size_t Read(byte *buffer, size_t size) {
          // returns audio data to the caller.
          // e.g. return read(config.YYY, buffer, size);
      }

      public void Close() {
          // close and cleanup resources.
      }
   };
  ```

- 根據您的音訊格式與輸入資料流建立音訊設定。 在建立自己的辨識器時，請傳入一般語音設定和音訊輸入設定。 例如︰

  ```
  var audioConfig = AudioConfig.FromStreamInput(new ContosoAudioStream(config), audioFormat);

  var speechConfig = SpeechConfig.FromSubscription(...);
  var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

  // run stream through recognizer
  var result = await recognizer.RecognizeOnceAsync();

  var text = result.GetText();
  ```

## <a name="next-steps"></a>後續步驟

* [試用認知服務](https://azure.microsoft.com/try/cognitive-services/)
* [了解如何以 C# 辨識語音](quickstart-csharp-dotnet-windows.md) (英文)
