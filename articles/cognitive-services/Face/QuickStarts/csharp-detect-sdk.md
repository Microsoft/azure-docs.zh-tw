---
title: 快速入門：偵測影像中的人臉 - SDK、C#
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用認知服務的臉部 Windows C# 用戶端程式庫，偵測影像中的臉部。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: quickstart
ms.date: 09/14/2018
ms.author: pafarley
ms.openlocfilehash: b5fd10aaab28303079d3eef9c4c5c25207bb672e
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344977"
---
# <a name="quickstart-detect-faces-in-an-image-using-c35---face"></a>快速入門：使用 C&#35; 偵測影像中的臉部 - 臉部

在此快速入門中，您會使用臉部 Windows 用戶端程式庫來偵測影像中的人臉。

## <a name="prerequisites"></a>必要條件

* 您需要有訂用帳戶金鑰才能執行範例。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)取得免費的試用訂用帳戶金鑰。
* 任何一版的 [Visual Studio 2017](https://www.visualstudio.com/downloads/)。
* [Microsoft.Azure.CognitiveServices.Vision.Face 2.2.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview) 用戶端程式庫 NuGet 套件。 您不必下載此套件。 下面會提供安裝指示。

## <a name="detectwithurlasync-method"></a>DetectWithUrlAsync 方法

> [!TIP]
> 從 [Github](https://github.com/Azure-Samples/cognitive-services-vision-csharp-sdk-quickstarts/tree/master/Face) 取得最新程式碼作為 Visual Studio 解決方案。

`DetectWithUrlAsync` 與 `DetectWithStreamAsync` 方法可分別針對遠端與本機影像包裝[臉部 - 偵測 API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) \(英文\)。 您可以使用這些方法，偵測影像中的人臉並傳回臉部屬性，包括：

* Face ID：數個臉部 API 案例中使用的唯一識別碼。
* 臉部矩形：左側、頂端、寬度和高度，表示影像中臉部的位置。
* 特徵點：27 點臉部特徵點陣列，指向臉部元件的重要位置。
* 臉部屬性，包括年齡、性別、笑容程度、頭部姿勢以及臉部毛髮。

若要執行範例，請執行下列步驟：

1. 在 Visual Studio 中建立 Visual C# 主控台應用程式。
1. 安裝臉部用戶端程式庫 NuGet 套件。
    1. 在頂端功能表中，按一下 [工具]，選取 [NuGet 套件管理員]，然後選取 [管理解決方案的 NuGet 套件]。
    1. 按一下 [瀏覽] 索引標籤，然後選取 [包括搶鮮版]。
    1. 在 [搜尋] 方塊中，輸入 "Microsoft.Azure.CognitiveServices.Vision.Face"。
    1. 顯示時選取 **Microsoft.Azure.CognitiveServices.Vision.Face**，按一下專案名稱旁邊的核取方塊，然後按一下 [安裝]。
1. 以下列程式碼取代 *Program.cs*。
1. 將 `<Subscription Key>` 換成您的有效訂用帳戶金鑰。
1. 必要時，請將 `faceEndpoint` 變更為與您的訂用帳戶金鑰相關聯的 Azure 區域。
1. (選擇性) 使用本機影像的路徑與檔案名稱來取代 <`LocalImage>` (若未設定則會忽略)。
1. (選擇性) 將 `remoteImageUrl` 設為不同的影像。
1. 執行程式。

```csharp
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;

using System;
using System.Collections.Generic;
using System.IO;
using System.Threading.Tasks;

namespace DetectFace
{
    class Program
    {
        // subscriptionKey = "0123456789abcdef0123456789ABCDEF"
        private const string subscriptionKey = "<SubscriptionKey>";

        // You must use the same region as you used to get your subscription
        // keys. For example, if you got your subscription keys from westus,
        // replace "westcentralus" with "westus".
        //
        // Free trial subscription keys are generated in the westcentralus
        // region. If you use a free trial subscription key, you shouldn't
        // need to change the region.
        // Specify the Azure region
        private const string faceEndpoint =
            "https://westcentralus.api.cognitive.microsoft.com";

        // localImagePath = @"C:\Documents\LocalImage.jpg"
        private const string localImagePath = @"<LocalImage>";

        private const string remoteImageUrl =
            "https://upload.wikimedia.org/wikipedia/commons/3/37/Dagestani_man_and_woman.jpg";

        private static readonly FaceAttributeType[] faceAttributes =
            { FaceAttributeType.Age, FaceAttributeType.Gender };

        static void Main(string[] args)
        {
            FaceClient faceClient = new FaceClient(
                new ApiKeyServiceClientCredentials(subscriptionKey),
                new System.Net.Http.DelegatingHandler[] { });
            faceClient.Endpoint = faceEndpoint;

            Console.WriteLine("Faces being detected ...");
            var t1 = DetectRemoteAsync(faceClient, remoteImageUrl);
            var t2 = DetectLocalAsync(faceClient, localImagePath);

            Task.WhenAll(t1, t2).Wait(5000);
            Console.WriteLine("Press any key to exit");
            Console.ReadLine();
        }

        // Detect faces in a remote image
        private static async Task DetectRemoteAsync(
            FaceClient faceClient, string imageUrl)
        {
            if (!Uri.IsWellFormedUriString(imageUrl, UriKind.Absolute))
            {
                Console.WriteLine("\nInvalid remoteImageUrl:\n{0} \n", imageUrl);
                return;
            }

            try
            {
                IList<DetectedFace> faceList =
                    await faceClient.Face.DetectWithUrlAsync(
                        imageUrl, true, false, faceAttributes);

                DisplayAttributes(GetFaceAttributes(faceList, imageUrl), imageUrl);
            }
            catch (APIErrorException e)
            {
                Console.WriteLine(imageUrl + ": " + e.Message);
            }
        }

        // Detect faces in a local image
        private static async Task DetectLocalAsync(FaceClient faceClient, string imagePath)
        {
            if (!File.Exists(imagePath))
            {
                Console.WriteLine(
                    "\nUnable to open or read localImagePath:\n{0} \n", imagePath);
                return;
            }

            try
            {
                using (Stream imageStream = File.OpenRead(imagePath))
                {
                    IList<DetectedFace> faceList =
                            await faceClient.Face.DetectWithStreamAsync(
                                imageStream, true, false, faceAttributes);
                    DisplayAttributes(
                        GetFaceAttributes(faceList, imagePath), imagePath);
                }
            }
            catch (APIErrorException e)
            {
                Console.WriteLine(imagePath + ": " + e.Message);
            }
        }

        private static string GetFaceAttributes(
            IList<DetectedFace> faceList, string imagePath)
        {
            string attributes = string.Empty;

            foreach (DetectedFace face in faceList)
            {
                double? age = face.FaceAttributes.Age;
                string gender = face.FaceAttributes.Gender.ToString();
                attributes += gender + " " + age + "   ";
            }

            return attributes;
        }

        // Display the face attributes
        private static void DisplayAttributes(string attributes, string imageUri)
        {
            Console.WriteLine(imageUri);
            Console.WriteLine(attributes + "\n");
        }
    }
}
```

### <a name="detectwithurlasync-response"></a>DetectWithUrlAsync 回應

成功的回應會顯示影像中每張臉的性別與年齡。

請參閱 [API 快速入門：使用 C# 偵測影像中的人臉](CSharp.md)，以取得未經處理 JSON 輸出的範例。

```
https://upload.wikimedia.org/wikipedia/commons/3/37/Dagestani_man_and_woman.jpg
Male 37   Female 56
```

## <a name="next-steps"></a>後續步驟

了解如何建立 WPF Windows 應用程式，使用臉部服務來偵測影像中的人臉。 此應用程式會在每張臉的周圍繪出邊框，然後在狀態列上顯示該張臉的描述。

> [!div class="nextstepaction"]
> [教學課程：建立 WPF 應用程式來偵測並框出影像中的臉部](../Tutorials/FaceAPIinCSharpTutorial.md)
