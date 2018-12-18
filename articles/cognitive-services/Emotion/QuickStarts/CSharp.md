---
title: 快速入門：辨識影像中的臉部表情 - 表情 API，C#
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您搭配 C# 快速開始使用表情 API。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: emotion-api
ms.topic: quickstart
ms.date: 11/02/2017
ms.author: anroth
ROBOTS: NOINDEX
ms.openlocfilehash: 530d05887e585884b184635e01031c1332fad3fb
ms.sourcegitcommit: 1981c65544e642958917a5ffa2b09d6b7345475d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48239365"
---
# <a name="quickstart-build-an-app-to-recognize-emotions-on-faces-in-an-image"></a>快速入門：建置應用程式以辨識影像中的臉部表情。

> [!IMPORTANT]
> 表情 API 將於 2019 年 2 月 15 日淘汰。 表情辨識功能現已公開推出，是[臉部 API](https://docs.microsoft.com/azure/cognitive-services/face/) 的一部分。

本文提供資訊和程式碼範例，以協助您搭配 C# 快速開始使用[表情 API 辨識方法](https://westus.dev.cognitive.microsoft.com/docs/services/5639d931ca73072154c1ce89/operations/563b31ea778daf121cc3a5fa)。 您可以使用它來辨識影像中一或多位人員所表現出來的表情。

## <a name="prerequisites"></a>必要條件
* 取得認知服務的[表情 API Windows SDK](https://www.nuget.org/packages/Microsoft.ProjectOxford.Emotion/)。
* 取得免費的[訂用帳戶金鑰](https://azure.microsoft.com/try/cognitive-services/)。

## <a name="emotion-recognition-c-example-request"></a>表情辨識 C# 範例要求

在 Visual Studio 中建立新的主控台方案，然後以下列程式碼取代 Program.cs。 將 `string uri` 變更為使用您取得訂用帳戶金鑰的區域。 以有效的訂用帳戶金鑰取代 **Ocp-Apim-Subscription-Key** 值。 若要尋找訂用帳戶金鑰，請移至 Azure 入口網站。 在左側導覽窗格的 [金鑰] 區段底下，瀏覽至表情 API 資源。 同樣地，您可以在 [端點] 底下所列資源的 [概觀] 面板中，取得適當的連線 URI。

![您的 API 資源金鑰](../../media/emotion-api/keys.png)

若要處理要求的回應，請使用 `Newtonsoft.Json` 之類的資源庫。 如此一來，您就可以處理 JSON 字串，作為一系列稱為「權杖」的可管理物件。 若要將這個資源庫新增至套件，請在 [方案總管] 中以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]。 然後搜尋 **Newtonsoft**。 第一個結果應該是 **Newtonsoft.Json**。 選取 [安裝]。 您現在可以在應用程式中參考此資源庫。

![安裝 Newtonsoft.Json](../../media/emotion-api/newtonsoft-nuget.png)

```csharp
using System;
using System.IO;
using System.Net.Http.Headers;
using System.Net.Http;
using Newtonsoft.Json.Linq;

namespace CSHttpClientSample
{
    static class Program
    {
        static void Main()
        {
            Console.Write("Enter the path to a JPEG image file:");
            string imageFilePath = Console.ReadLine();

            MakeRequest(imageFilePath);

            Console.WriteLine("\n\n\nWait for the result below, then hit ENTER to exit...\n\n\n");
            Console.ReadLine(); // wait for ENTER to exit program
        }

        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        static async void MakeRequest(string imageFilePath)
        {
            var client = new HttpClient();

            // Request headers - replace this example key with your valid key.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "<your-subscription-key>"); //

            // NOTE: You must use the same region in your REST call as you used to obtain your subscription keys.
            //   For example, if you obtained your subscription keys from westcentralus, replace "westus" in the
            //   URI below with "westcentralus".
            string uri = "https://westus.api.cognitive.microsoft.com/emotion/v1.0/recognize?";
            HttpResponseMessage response;
            string responseContent;

            // Request body. Try this sample with a locally stored JPEG image.
            byte[] byteData = GetImageAsByteArray(imageFilePath);

            using (var content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                response = await client.PostAsync(uri, content);
                responseContent = response.Content.ReadAsStringAsync().Result;
            }

            // A peek at the raw JSON response.
            Console.WriteLine(responseContent);

            // Processing the JSON into manageable objects.
            JToken rootToken = JArray.Parse(responseContent).First;

            // First token is always the faceRectangle identified by the API.
            JToken faceRectangleToken = rootToken.First;

            // Second token is all emotion scores.
            JToken scoresToken = rootToken.Last;

            // Show all face rectangle dimensions
            JEnumerable<JToken> faceRectangleSizeList = faceRectangleToken.First.Children();
            foreach (var size in faceRectangleSizeList) {
                Console.WriteLine(size);
            }

            // Show all scores
            JEnumerable<JToken> scoreList = scoresToken.First.Children();
            foreach (var score in scoreList) {
                Console.WriteLine(score);
            }
        }
    }
}
```

## <a name="recognize-emotions-sample-response"></a>辨識表情範例回應
成功呼叫將會傳回臉部項目的陣列，以及其相關聯的表情分數。 它們會依臉部矩形大小以遞減方式排序。 空的回應表示沒有偵測到任何臉部。 表情項目包含下列欄位：

* faceRectangle：臉部於影像中的矩形位置
* scores：影像中每個臉部的表情分數

```json
application/json
[
  {
    "faceRectangle": {
      "left": 68,
      "top": 97,
      "width": 64,
      "height": 97
    },
    "scores": {
      "anger": 0.00300731952,
      "contempt": 5.14648448E-08,
      "disgust": 9.180124E-06,
      "fear": 0.0001912825,
      "happiness": 0.9875571,
      "neutral": 0.0009861537,
      "sadness": 1.889955E-05,
      "surprise": 0.008229999
    }
  }
]
