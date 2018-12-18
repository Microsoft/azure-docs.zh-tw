---
title: 快速入門：產生縮圖 - REST、C# - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 C#，從影像產生縮圖。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: b3822fee5d729064375e5088cb207aebfa74f8ac
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342021"
---
# <a name="quickstart-generate-a-thumbnail-using-the-rest-api-and-c35-in-computer-vision"></a>快速入門：使用電腦視覺中的 REST API 和 C&#35; 產生縮圖

在本快速入門中，您會使用電腦視覺的 REST API 從影像產生縮圖。 您可以使用[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法產生影像的縮圖。 您可指定高度和寬度，可以與輸入影像的外觀比例不同。 「電腦視覺」會使用智慧型裁剪，以智慧方式識別感興趣的區域，並且根據該區域產生裁剪的座標。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

- 您必須有 [Visual Studio 2015](https://visualstudio.microsoft.com/downloads/) 或更新版本。
- 您必須有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample-application"></a>建立並執行範例應用程式

若要在 Visual Studio 中建立範例，請執行下列步驟：

1. 在 Visual Studio 中，使用 Visual C# 主控台應用程式範本建立新的 Visual Studio 解決方案。
1. 安裝 Newtonsoft.Json NuGet 套件。
    1. 在功能表中，按一下 [工具]，選取 [NuGet 套件管理員]，然後選取 [管理解決方案的 NuGet 套件]。
    1. 按一下 [瀏覽] 索引標籤，然後在 [搜尋] 方塊中鍵入 "Newtonsoft.Json"。
    1. 顯示時選取 [Newtonsoft.Json]，按一下專案名稱旁邊的核取方塊，然後按一下 [安裝]。
1. 將 `Program.cs` 中的程式碼取代為下列程式碼，然後視需要在程式碼中進行下列變更：
    1. 將 `subscriptionKey` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將 `uriBase` 的值取代為[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法的端點 URL。
1. 執行程式。
1. 在系統提示時，輸入本機影像的路徑。

```csharp
using Newtonsoft.Json.Linq;
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;

namespace CSHttpClientSample
{
    static class Program
    {
        // Replace <Subscription Key> with your valid subscription key.
        const string subscriptionKey = "<Subscription Key>";

        // You must use the same Azure region in your REST API method as you used to
        // get your subscription keys. For example, if you got your subscription keys
        // from the West US region, replace "westcentralus" in the URL
        // below with "westus".
        //
        // Free trial subscription keys are generated in the West Central US region.
        // If you use a free trial subscription key, you shouldn't need to change
        // this region.
        const string uriBase =
            "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail";

        static void Main()
        {
            // Get the path and filename to process from the user.
            Console.WriteLine("Thumbnail:");
            Console.Write(
                "Enter the path to the image you wish to use to create a thumbnail image: ");
            string imageFilePath = Console.ReadLine();

            if (File.Exists(imageFilePath))
            {
                // Call the REST API method.
                Console.WriteLine("\nWait a moment for the results to appear.\n");
                MakeThumbNailRequest(imageFilePath).Wait();
            }
            else
            {
                Console.WriteLine("\nInvalid file path");
            }
            Console.WriteLine("\nPress Enter to exit...");
            Console.ReadLine();
        }

        /// <summary>
        /// Gets a thumbnail image from the specified image file by using
        /// the Computer Vision REST API.
        /// </summary>
        /// <param name="imageFilePath">The image file to use to create the thumbnail image.</param>
        static async Task MakeThumbNailRequest(string imageFilePath)
        {
            try
            {
                HttpClient client = new HttpClient();

                // Request headers.
                client.DefaultRequestHeaders.Add(
                    "Ocp-Apim-Subscription-Key", subscriptionKey);

                // Request parameters.
                // The width and height parameters specify a thumbnail that's 
                // 200 pixels wide and 150 pixels high.
                // The smartCropping parameter is set to true, to enable smart cropping.
                string requestParameters = "width=200&height=150&smartCropping=true";

                // Assemble the URI for the REST API method.
                string uri = uriBase + "?" + requestParameters;

                HttpResponseMessage response;

                // Read the contents of the specified local image
                // into a byte array.
                byte[] byteData = GetImageAsByteArray(imageFilePath);

                // Add the byte array as an octet stream to the request body.
                using (ByteArrayContent content = new ByteArrayContent(byteData))
                {
                    // This example uses the "application/octet-stream" content type.
                    // The other content types you can use are "application/json"
                    // and "multipart/form-data".
                    content.Headers.ContentType =
                        new MediaTypeHeaderValue("application/octet-stream");

                    // Asynchronously call the REST API method.
                    response = await client.PostAsync(uri, content);
                }

                // Check the HTTP status code of the response. If successful, display
                // display the response and save the thumbnail.
                if (response.IsSuccessStatusCode)
                {
                    // Display the response data.
                    Console.WriteLine("\nResponse:\n{0}", response);

                    // Get the image data for the thumbnail from the response.
                    byte[] thumbnailImageData =
                        await response.Content.ReadAsByteArrayAsync();

                    // Save the thumbnail to the same folder as the original image,
                    // using the original name with the suffix "_thumb".
                    // Note: This will overwrite an existing file of the same name.
                    string thumbnailFilePath =
                        imageFilePath.Insert(imageFilePath.Length - 4, "_thumb");
                    File.WriteAllBytes(thumbnailFilePath, thumbnailImageData);
                    Console.WriteLine("\nThumbnail written to: {0}", thumbnailFilePath);
                }
                else
                {
                    // Display the JSON error data.
                    string errorString = await response.Content.ReadAsStringAsync();
                    Console.WriteLine("\n\nResponse:\n{0}\n",
                        JToken.Parse(errorString).ToString());
                }
            }
            catch (Exception e)
            {
                Console.WriteLine("\n" + e.Message);
            }
        }

        /// <summary>
        /// Returns the contents of the specified file as a byte array.
        /// </summary>
        /// <param name="imageFilePath">The image file to read.</param>
        /// <returns>The byte array of the image data.</returns>
        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            // Open a read-only file stream for the specified file.
            using (FileStream fileStream =
                new FileStream(imageFilePath, FileMode.Open, FileAccess.Read))
            {
                // Read the file's contents into a byte array.
                BinaryReader binaryReader = new BinaryReader(fileStream);
                return binaryReader.ReadBytes((int)fileStream.Length);
            }
        }
    }
}
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以二進位資料的形式傳回，代表縮圖的影像資料。 如果要求成功，系統會使用原始名稱加上後置詞 "_thumb"，將縮圖儲存至與本機影像相同的資料夾中。 如果要求失敗，回應會包含錯誤碼和訊息以協助判斷問題所在。

範例應用程式會在主控台視窗中顯示成功的回應，如下列範例所示：

```text
Response:

StatusCode: 200, ReasonPhrase: 'OK', Version: 1.1, Content: System.Net.Http.StreamContent, Headers:
{
  Pragma: no-cache
  apim-request-id: 131eb5b4-5807-466d-9656-4c1ef0a64c9b
  Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
  x-content-type-options: nosniff
  Cache-Control: no-cache
  Date: Tue, 06 Jun 2017 20:54:07 GMT
  X-AspNet-Version: 4.0.30319
  X-Powered-By: ASP.NET
  Content-Length: 5800
  Content-Type: image/jpeg
  Expires: -1
}
```

## <a name="clean-up-resources"></a>清除資源

不再需要 Visual Studio 解決方案時，請將它刪除。 若要這樣做，請開啟檔案總管、瀏覽到您在其中建立 Visual Studio 解決方案的資料夾，然後刪除該資料夾。

## <a name="next-steps"></a>後續步驟

探索使用「電腦視覺」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的基本 Windows 應用程式。 若要快速地試驗「電腦視覺 API」，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console) (英文)。

> [!div class="nextstepaction"]
> [電腦視覺 API C&#35; 教學課程](../Tutorials/CSharpTutorial.md)