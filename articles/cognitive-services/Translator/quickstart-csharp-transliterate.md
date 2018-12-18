---
title: 快速入門：轉換文字字集 - 翻譯工具文字、C#
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用翻譯工具文字 API 搭配 C#，將文字轉換成另一種語言的字集。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: quickstart
ms.date: 06/15/2018
ms.author: nolachar
ms.openlocfilehash: 7fa8c5dc76e59ec236d6b9662bd2ba41db0add0f
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320402"
---
# <a name="quickstart-transliterate-text-with-c35"></a>快速入門：使用 C# 來音譯文字

在本快速入門中，您可以使用翻譯工具文字 API，將文字轉換成另一種語言的腳本。

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

若要使用翻譯工具文字 API，您也需要有訂用帳戶金鑰；請參閱[如何註冊翻譯工具文字 API](translator-text-how-to-signup.md)。

## <a name="transliterate-request"></a>Transliterate 要求

> [!TIP]
> 從 [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-C-Sharp) 取得最新程式碼。

下列程式碼會使用 [Transliterate](./reference/v3-0-transliterate.md) 方法，將文字轉換成另一種語言的腳本。

1. 在您最愛的 IDE 中建立新的 C# 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `key` 值。
4. 執行程式。

```csharp
using System;
using System.Net.Http;
using System.Text;
// NOTE: Install the Newtonsoft.Json NuGet package.
using Newtonsoft.Json;

namespace TranslatorTextQuickStart
{
    class Program
    {
        static string host = "https://api.cognitive.microsofttranslator.com";
        static string path = "/transliterate?api-version=3.0";
        // Transliterate text in Japanese from Japanese script (i.e. Hiragana/Katakana/Kanji) to Latin script.
        static string params_ = "&language=ja&fromScript=jpan&toScript=latn";

        static string uri = host + path + params_;

        // NOTE: Replace this example key with a valid subscription key.
        static string key = "ENTER KEY HERE";

        // Transliterate "good afternoon".
        static string text = "こんにちは";

        async static void Transliterate()
        {
            System.Object[] body = new System.Object[] { new { Text = text } };
            var requestBody = JsonConvert.SerializeObject(body);

            using (var client = new HttpClient())
            using (var request = new HttpRequestMessage())
            {
                request.Method = HttpMethod.Post;
                request.RequestUri = new Uri(uri);
                request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
                request.Headers.Add("Ocp-Apim-Subscription-Key", key);

                var response = await client.SendAsync(request);
                var responseBody = await response.Content.ReadAsStringAsync();
                var result = JsonConvert.SerializeObject(JsonConvert.DeserializeObject(responseBody), Formatting.Indented);

                Console.OutputEncoding = UnicodeEncoding.UTF8;
                Console.WriteLine(result);
            }
        }

        static void Main(string[] args)
        {
            Transliterate();
            Console.ReadLine();
        }
    }
}
```

## <a name="transliterate-response"></a>Transliterate 回應

成功的回應會以 JSON 格式傳回，如下列範例所示：

```json
[
  {
    "text": "konnnichiha",
    "script": "latn"
  }
]
```

## <a name="next-steps"></a>後續步驟

瀏覽本快速入門及其他文件的範例程式碼，包括翻譯和語言識別，以及 GitHub 上的其他「翻譯工具文字」專案範例。

> [!div class="nextstepaction"]
> [瀏覽 GitHub 上的 C# 範例](https://aka.ms/TranslatorGitHub?type=&language=c%23)
