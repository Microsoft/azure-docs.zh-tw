---
title: 快速入門：擷取文字 - SDK、C# - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您會使用電腦視覺 Windows C# 用戶端程式庫，擷取影像中的文字。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 09/27/2018
ms.author: pafarley
ms.openlocfilehash: d819aee7c42759e2427e27e0f640d79536069ccb
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343600"
---
# <a name="quickstart-extract-text-using-the-computer-vision-sdk-and-c"></a>快速入門：使用電腦視覺 SDK 和 C# 擷取文字

在本快速入門中，您會使用電腦視覺 Windows 用戶端程式庫，擷取影像中的手寫或列印文字。

## <a name="prerequisites"></a>必要條件

* 若要使用「電腦視覺」，您需要訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。
* 任何 [Visual Studio 2015 或 2017](https://www.visualstudio.com/downloads/) 版本。
* [Microsoft.Azure.CognitiveServices.Vision.ComputerVision](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision) (英文) 用戶端程式庫 NuGet 套件。 您不必下載此套件。 下面會提供安裝指示。

## <a name="recognizetextasync-method"></a>RecognizeTextAsync 方法

> [!TIP]
> 從 [Github](https://github.com/Azure-Samples/cognitive-services-vision-csharp-sdk-quickstarts/tree/master/ComputerVision) 取得最新程式碼作為 Visual Studio 解決方案。

`RecognizeTextAsync` 和 `RecognizeTextInStreamAsync` 方法可分別針對遠端和本機影像包裝[辨識文字 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2c6a154055056008f200) (英文)。 `GetTextOperationResultAsync` 方法會包裝[取得辨識文字作業結果 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2cf1154055056008f201) (英文)。  您可以使用這些方法來偵測影像中的文字，然後將辨識出的字元擷取到電腦可使用的字元資料流中。

若要執行範例，請執行下列步驟：

1. 在 Visual Studio 中建立 Visual C# 主控台應用程式。
1. 安裝「電腦視覺」用戶端程式庫 NuGet 套件。
    1. 在功能表中，按一下 [工具]，選取 [NuGet 套件管理員]，然後選取 [管理解決方案的 NuGet 套件]。
    1. 按一下 [瀏覽] 索引標籤，然後在 [搜尋] 方塊中，鍵入 "Microsoft.Azure.CognitiveServices.Vision.ComputerVision"。
    1. 顯示時選取 [Microsoft.Azure.CognitiveServices.Vision.ComputerVision]，按一下專案名稱旁邊的核取方塊，然後按一下 [安裝]。
1. 使用以下程式碼來取代 `Program.cs`。
1. 將 `<Subscription Key>` 換成您的有效訂用帳戶金鑰。
1. 必要時，請將 `computerVision.Endpoint` 變更為與您的訂用帳戶金鑰相關聯的 Azure 區域。
1. 選擇性地將 `textRecognitionMode` 設定為 `TextRecognitionMode.Printed`。
1. 使用本機影像的路徑和檔案名稱來取代 `<LocalImage>`。
1. (選擇性) 將 `remoteImageUrl` 設為不同的影像。
1. 執行程式。

```csharp
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

using System;
using System.IO;
using System.Threading.Tasks;

namespace ExtractText
{
    class Program
    {
        // subscriptionKey = "0123456789abcdef0123456789ABCDEF"
        private const string subscriptionKey = "<SubscriptionKey>";

        // For printed text, change to TextRecognitionMode.Printed
        private const TextRecognitionMode textRecognitionMode =
            TextRecognitionMode.Handwritten;

        // localImagePath = @"C:\Documents\LocalImage.jpg"
        private const string localImagePath = @"<LocalImage>";

        private const string remoteImageUrl =
            "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/" +
            "Cursive_Writing_on_Notebook_paper.jpg/" +
            "800px-Cursive_Writing_on_Notebook_paper.jpg";

        private const int numberOfCharsInOperationId = 36;

        static void Main(string[] args)
        {
            ComputerVisionClient computerVision = new ComputerVisionClient(
                new ApiKeyServiceClientCredentials(subscriptionKey),
                new System.Net.Http.DelegatingHandler[] { });

            // You must use the same region as you used to get your subscription
            // keys. For example, if you got your subscription keys from westus,
            // replace "westcentralus" with "westus".
            //
            // Free trial subscription keys are generated in the westcentralus
            // region. If you use a free trial subscription key, you shouldn't
            // need to change the region.

            // Specify the Azure region
            computerVision.Endpoint = "https://westcentralus.api.cognitive.microsoft.com";

            Console.WriteLine("Images being analyzed ...");
            var t1 = ExtractRemoteTextAsync(computerVision, remoteImageUrl);
            var t2 = ExtractLocalTextAsync(computerVision, localImagePath);

            Task.WhenAll(t1, t2).Wait(5000);
            Console.WriteLine("Press ENTER to exit");
            Console.ReadLine();
        }

        // Recognize text from a remote image
        private static async Task ExtractRemoteTextAsync(
            ComputerVisionClient computerVision, string imageUrl)
        {
            if (!Uri.IsWellFormedUriString(imageUrl, UriKind.Absolute))
            {
                Console.WriteLine(
                    "\nInvalid remoteImageUrl:\n{0} \n", imageUrl);
                return;
            }

            // Start the async process to recognize the text
            RecognizeTextHeaders textHeaders =
                await computerVision.RecognizeTextAsync(
                    imageUrl, textRecognitionMode);

            await GetTextAsync(computerVision, textHeaders.OperationLocation);
        }

        // Recognize text from a local image
        private static async Task ExtractLocalTextAsync(
            ComputerVisionClient computerVision, string imagePath)
        {
            if (!File.Exists(imagePath))
            {
                Console.WriteLine(
                    "\nUnable to open or read localImagePath:\n{0} \n", imagePath);
                return;
            }

            using (Stream imageStream = File.OpenRead(imagePath))
            {
                // Start the async process to recognize the text
                RecognizeTextInStreamHeaders textHeaders =
                    await computerVision.RecognizeTextInStreamAsync(
                        imageStream, textRecognitionMode);

                await GetTextAsync(computerVision, textHeaders.OperationLocation);
            }
        }

        // Retrieve the recognized text
        private static async Task GetTextAsync(
            ComputerVisionClient computerVision, string operationLocation)
        {
            // Retrieve the URI where the recognized text will be
            // stored from the Operation-Location header
            string operationId = operationLocation.Substring(
                operationLocation.Length - numberOfCharsInOperationId);

            Console.WriteLine("\nCalling GetHandwritingRecognitionOperationResultAsync()");
            TextOperationResult result =
                await computerVision.GetTextOperationResultAsync(operationId);

            // Wait for the operation to complete
            int i = 0;
            int maxRetries = 10;
            while ((result.Status == TextOperationStatusCodes.Running ||
                    result.Status == TextOperationStatusCodes.NotStarted) && i++ < maxRetries)
            {
                Console.WriteLine(
                    "Server status: {0}, waiting {1} seconds...", result.Status, i);
                await Task.Delay(1000);

                result = await computerVision.GetTextOperationResultAsync(operationId);
            }

            // Display the results
            Console.WriteLine();
            var lines = result.RecognitionResult.Lines;
            foreach (Line line in lines)
            {
                Console.WriteLine(line.Text);
            }
            Console.WriteLine();
        }
    }
}
```

## <a name="recognizetextasync-response"></a>RecognizeTextAsync 回應

成功的回應會顯示每個影像的可辨識文字行數。

請參閱[快速入門：擷取手寫文字 - REST、C#](../QuickStarts/CSharp-hand-text.md#examine-the-response)，以取得原始 JSON 輸出的範例。

```cmd
Calling GetHandwritingRecognitionOperationResultAsync()

Calling GetHandwritingRecognitionOperationResultAsync()
Server status: Running, waiting 1 seconds...
Server status: Running, waiting 1 seconds...

dog
The quick brown fox jumps over the lazy
Pack my box with five dozen liquor jugs
```

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人和地標、建立縮圖及擷取印刷和手寫文字的「電腦視覺 API」。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44) (英文)
