---
title: 快速入門：翻譯工具語音 API C#
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用翻譯工具語音 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-speech
ms.topic: quickstart
ms.date: 3/5/2018
ms.author: v-jaswel
ms.openlocfilehash: 224a0ab83720bb9605f2dad9c2612630e90fea2a
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49341730"
---
# <a name="quickstart-translator-speech-api-with-c"></a>快速入門：翻譯工具語音 API (使用 C#) 
<a name="HOLTop"></a>

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-translator-speech-deprecation-note.md)]

此文章說明如何使用翻譯工具語音 API 來翻譯 .wav 檔案中所說的話語。

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

您將需要將一個名為 "speak.wav" 的 .wav 檔案，放在與您從下方程式碼編譯之可執行檔相同的資料夾中。 這個 .wav 檔案應該採用標準 PCM 16 位元 16 kHz 單聲道格式。

您必須擁有具備 **Microsoft Translator Speech API** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 您將需要一個來自 [Azure 儀表板](https://portal.azure.com/#create/Microsoft.CognitiveServices)的付費訂用帳戶金鑰。

## <a name="translate-speech"></a>翻譯語音

以下程式碼會將語音從一種語言翻譯成另一種語言。

1. 在您最愛的 IDE 中建立新的 C# 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```csharp
using System;
using System.IO;
using System.Net.WebSockets;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace TranslateSpeechQuickStart
{
    class Program
    {
        static string host = "wss://dev.microsofttranslator.com";
        static string path = "/speech/translate";

        // NOTE: Replace this example key with a valid subscription key.
        static string key = "ENTER KEY HERE";

        async static Task Send (ClientWebSocket client, string input_path)
        {
            var audio = File.ReadAllBytes(input_path);
            var audio_out_buffer = new ArraySegment<byte>(audio);
            Console.WriteLine("Sending audio.");
            await client.SendAsync(audio_out_buffer, WebSocketMessageType.Binary, true, CancellationToken.None);

            /* Make sure the audio file is followed by silence.
             * This lets the service know that the audio input is finished. */
            var silence = new byte[32000];
            var silence_buffer = new ArraySegment<byte>(silence);
            await client.SendAsync(silence_buffer, WebSocketMessageType.Binary, true, CancellationToken.None);

            Console.WriteLine("Done sending.");
            System.Threading.Thread.Sleep(3000);
            await client.CloseAsync(WebSocketCloseStatus.NormalClosure, "", CancellationToken.None);
        }

        async static Task Receive(ClientWebSocket client, string output_path)
        {
            var inbuf = new byte[102400];
            var segment = new ArraySegment<byte>(inbuf);
            var stream = new FileStream(output_path, FileMode.Create);

            Console.WriteLine("Awaiting response.");
            while (client.State == WebSocketState.Open)
            {
                var result = await client.ReceiveAsync(segment, CancellationToken.None);
                switch (result.MessageType)
                {
                    case WebSocketMessageType.Close:
                        Console.WriteLine("Received close message. Status: " + result.CloseStatus + ". Description: " + result.CloseStatusDescription);
                        await client.CloseAsync(WebSocketCloseStatus.NormalClosure, string.Empty, CancellationToken.None);
                        break;
                    case WebSocketMessageType.Text:
                        Console.WriteLine("Received text.");
                        Console.WriteLine(Encoding.UTF8.GetString(inbuf).TrimEnd('\0'));
                        break;
                    case WebSocketMessageType.Binary:
                        Console.WriteLine("Received binary data: " + result.Count + " bytes.");
                        stream.Write(inbuf, 0, result.Count);
                        break;
                }
            }

            stream.Close();
            stream.Dispose();
        }

        async static void TranslateSpeech()
        {
            var client = new ClientWebSocket();
            client.Options.SetRequestHeader ("Ocp-Apim-Subscription-Key", key);

            string from = "en-US";
            string to = "it-IT";
            string features = "texttospeech";
            string voice = "it-IT-Elsa";
            string api = "1.0";

            string input_path = "speak.wav";
            string output_path = "speak2.wav";

            string uri = host + path +
                "?from=" + from +
                "&to=" + to +
                "&api-version=" + api +
                "&features=" + features +
                "&voice=" + voice;

            Console.WriteLine("uri: " + uri);
            Console.WriteLine("Opening connection.");
            await client.ConnectAsync(new Uri(uri), CancellationToken.None);
            Console.WriteLine("Connection open.");
            Task.WhenAll(Send(client, input_path), Receive(client, output_path)).Wait();
        }

        static void Main()
        {
            TranslateSpeech();
            Console.ReadLine();
        }

    }
}
```

**翻譯語音回應**

成功的結果是會建立名為 "speak2.wav" 的檔案。 此檔案包含 "speak.wav" 中所說話語的翻譯。

[回到頁首](#HOLTop)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Translator Speech 教學課程](../tutorial-translator-speech-csharp.md)

## <a name="see-also"></a>另請參閱 

[Translator Speech 概觀](../overview.md)
[API 參考](https://docs.microsoft.com/azure/cognitive-services/translator-speech/reference)
