---
title: 快速入門：分析影像 - SDK、C# - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您會使用電腦視覺 Windows C# 用戶端程式庫，分析影像。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 09/14/2018
ms.author: pafarley
ms.openlocfilehash: 81a7b32ef2970efc7f53ec8d25350efb217d7b36
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343636"
---
# <a name="quickstart-analyze-an-image-using-the-computer-vision-sdk-and-c"></a>本快速入門：使用電腦視覺 SDK 和 C# 來分析影像

在本快速入門中，您使用電腦視覺 Windows 用戶端程式庫，來分析要擷取視覺功能的本機與遠端影像。

## <a name="prerequisites"></a>必要條件

* 若要使用「電腦視覺」，您需要訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。
* 任何 [Visual Studio 2015 或 2017](https://www.visualstudio.com/downloads/) 版本。
* [Microsoft.Azure.CognitiveServices.Vision.ComputerVision](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.ComputerVision) (英文) 用戶端程式庫 NuGet 套件。 您不必下載此套件。 下面會提供安裝指示。

## <a name="analyzeimageasync-method"></a>AnalyzeImageAsync 方法

> [!TIP]
> 從 [Github](https://github.com/Azure-Samples/cognitive-services-vision-csharp-sdk-quickstarts/tree/master/ComputerVision) 取得最新程式碼作為 Visual Studio 解決方案。

`AnalyzeImageAsync` 和 `AnalyzeImageInStreamAsync` 方法可分別針對遠端和本機影像包裝[分析影像 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) (英文)。 您可以使用這些方法，根據影像內容來擷取視覺功能，並且選擇要傳回哪些功能，包括：

* 與影像內容相關的標記詳細清單。
* 對影像內容的完整句子描述。
* 影像中所包含任何臉部的座標、性別和年齡。
* ImageType (美工圖案或線條繪圖)。
* 主要色彩、輔色，或影像是否為黑白。
* 在此[分類法](../Category-Taxonomy.md)中定義的分類。
* 影像是否包含成人或性暗示內容？

若要執行範例，請執行下列步驟：

1. 在 Visual Studio 中建立 Visual C# 主控台應用程式。
1. 安裝「電腦視覺」用戶端程式庫 NuGet 套件。
    1. 在功能表中，按一下 [工具]，選取 [NuGet 套件管理員]，然後選取 [管理解決方案的 NuGet 套件]。
    1. 按一下 [瀏覽] 索引標籤，然後在 [搜尋] 方塊中，鍵入 "Microsoft.Azure.CognitiveServices.Vision.ComputerVision"。
    1. 顯示時選取 [Microsoft.Azure.CognitiveServices.Vision.ComputerVision]，按一下專案名稱旁邊的核取方塊，然後按一下 [安裝]。
1. 使用以下程式碼來取代 `Program.cs`。
1. 將 `<Subscription Key>` 換成您的有效訂用帳戶金鑰。
1. 必要時，請將 `computerVision.Endpoint` 變更為與您的訂用帳戶金鑰相關聯的 Azure 區域。
1. 使用本機影像的路徑和檔案名稱來取代 `<LocalImage>`。
1. (選擇性) 將 `remoteImageUrl` 設為不同的影像。
1. 執行程式。

```csharp
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;

using System;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;

namespace ImageAnalyze
{
    class Program
    {
        // subscriptionKey = "0123456789abcdef0123456789ABCDEF"
        private const string subscriptionKey = "<SubscriptionKey>";

        // localImagePath = @"C:\Documents\LocalImage.jpg"
        private const string localImagePath = @"<LocalImage>";

        private const string remoteImageUrl =
            "http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg";

        // Specify the features to return
        private static readonly List<VisualFeatureTypes> features =
            new List<VisualFeatureTypes>()
        {
            VisualFeatureTypes.Categories, VisualFeatureTypes.Description,
            VisualFeatureTypes.Faces, VisualFeatureTypes.ImageType,
            VisualFeatureTypes.Tags
        };

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
            var t1 = AnalyzeRemoteAsync(computerVision, remoteImageUrl);
            var t2 = AnalyzeLocalAsync(computerVision, localImagePath);

            Task.WhenAll(t1, t2).Wait(5000);
            Console.WriteLine("Press ENTER to exit");
            Console.ReadLine();
        }

        // Analyze a remote image
        private static async Task AnalyzeRemoteAsync(
            ComputerVisionClient computerVision, string imageUrl)
        {
            if (!Uri.IsWellFormedUriString(imageUrl, UriKind.Absolute))
            {
                Console.WriteLine(
                    "\nInvalid remoteImageUrl:\n{0} \n", imageUrl);
                return;
            }

            ImageAnalysis analysis =
                await computerVision.AnalyzeImageAsync(imageUrl, features);
            DisplayResults(analysis, imageUrl);
        }

        // Analyze a local image
        private static async Task AnalyzeLocalAsync(
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
                ImageAnalysis analysis = await computerVision.AnalyzeImageInStreamAsync(
                    imageStream, features);
                DisplayResults(analysis, imagePath);
            }
        }

        // Display the most relevant caption for the image
        private static void DisplayResults(ImageAnalysis analysis, string imageUri)
        {
            Console.WriteLine(imageUri);
            Console.WriteLine(analysis.Description.Captions[0].Text + "\n");
        }
    }
}
```

## <a name="analyzeimageasync-response"></a>AnalyzeImageAsync 回應

成功的回應會顯示每個影像的最相關標題。

請參閱 [API 快速入門：使用 C# 分析本機影像](../QuickStarts/CSharp-analyze.md#examine-the-response)，以取得未經處理 JSON 輸出的範例。

```
http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg
a large waterfall over a rocky cliff
```

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人和地標、建立縮圖及擷取印刷和手寫文字的「電腦視覺 API」。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44) (英文)
