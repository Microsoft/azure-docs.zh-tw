---
title: 範例：使用預測端點以程式設計方式利用分類器測試影像 - 自訂視覺
titlesuffix: Azure Cognitive Services
description: 了解如何使用 API，以程式設計方式利用您的自訂視覺服務分類器來測試影像。
services: cognitive-services
author: anrothMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: sample
ms.date: 05/03/2018
ms.author: anroth
ms.openlocfilehash: 3a81f3cef6aaeb5c98022d9fc93f4d84f3f58a6e
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46363644"
---
# <a name="use-the-prediction-endpoint-to-test-images-programmatically-with-a-custom-vision-service-classifier"></a>使用預測端點以程式設計方式利用自訂視覺服務分類器來測試影像

為模型定型之後，您可以藉由送出影像到預測 API，以程式設計方式測試它們。 

> [!NOTE]
> 此文件示範如何使用 C#，送出影像到預測 API。 如需使用 API 的詳細資訊和範例，請參閱[預測 API 參考](https://go.microsoft.com/fwlink/?linkid=865445) \(英文\)。

## <a name="get-the-url-and-prediction-key"></a>取得 URL 和預測金鑰

從[自訂視覺網頁](https://customvision.ai) \(英文\) 選取您的專案，然後選取 [效能] 索引標籤。若要顯示使用預測 API 的相關資訊 (包括 __Prediction-key__)，請選取 [預測 URL]。 對於連結至 Azure 資源的專案，在 [Azure 入口網站](https://portal.azure.com)頁面中 (相關聯 Azure 資源的 [金鑰] 下) 也可以找到您的 __Prediction-key__。 複製下列資訊以便在應用程式中使用：

* 使用__影像檔__的 __URL__。
* __預測金鑰__值。

> [!TIP]
> 如果您有多個反覆項目，可藉由將其中一個反覆項目設為預設值來控制要使用哪一個。 從 [反覆項目] 區段選取反覆項目，然後選取頁面頂端的 [設成預設值]。

![[效能] 索引標籤會顯示以紅色矩形框起來的預測 URL。](./media/use-prediction-api/prediction-url.png)

## <a name="create-the-application"></a>建立應用程式

1. 從 Visual Studio 中，建立新的 C# 主控台應用程式。

2. 使用下列程式碼作為 __Program.cs__ 檔案的主體。

    > [!IMPORTANT]
    > 變更下列資訊：
    >
    > * 將__命名空間__設為專案的名稱。
    > * 在以 `client.DefaultRequestHeaders.Add("Prediction-Key",` 開頭的行中，設定您稍早收到的__預測金鑰__值。
    > * 在以 `string url =` 開頭的行中，設定您稍早收到的 __URL__ 值。

    ```csharp
    using System;
    using System.IO;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Threading.Tasks;

    namespace CSPredictionSample
    {
        static class Program
        {
            static void Main()
            {
                Console.Write("Enter image file path: ");
                string imageFilePath = Console.ReadLine();

                MakePredictionRequest(imageFilePath).Wait();

                Console.WriteLine("\n\n\nHit ENTER to exit...");
                Console.ReadLine();
            }

            static byte[] GetImageAsByteArray(string imageFilePath)
            {
                FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
                BinaryReader binaryReader = new BinaryReader(fileStream);
                return binaryReader.ReadBytes((int)fileStream.Length);
            }

            static async Task MakePredictionRequest(string imageFilePath)
            {
                var client = new HttpClient();

                // Request headers - replace this example key with your valid subscription key.
                client.DefaultRequestHeaders.Add("Prediction-Key", "13hc77781f7e4b19b5fcdd72a8df7156");

                // Prediction URL - replace this example URL with your valid prediction URL.
                string url = "http://southcentralus.api.cognitive.microsoft.com/customvision/v1.0/prediction/d16e136c-5b0b-4b84-9341-6a3fff8fa7fe/image?iterationId=f4e573f6-9843-46db-8018-b01d034fd0f2";

                HttpResponseMessage response;

                // Request body. Try this sample with a locally stored image.
                byte[] byteData = GetImageAsByteArray(imageFilePath);

                using (var content = new ByteArrayContent(byteData))
                {
                    content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
                    response = await client.PostAsync(url, content);
                    Console.WriteLine(await response.Content.ReadAsStringAsync());
                }
            }
        }
    }
    ```

## <a name="use-the-application"></a>使用應用程式

執行應用程式時，輸入影像檔的路徑。 送出影像到 API，並以 JSON 文件傳回結果。 下列 JSON 是回應的範例

```json
{
    "Id":"3f76364c-b8ae-4818-a2b2-2794cfbe377a",
    "Project":"2277aca4-7aff-4742-8afb-3682e251c913",
    "Iteration":"84105bfe-73b5-4fcc-addb-756c0de17df2",
    "Created":"2018-05-03T14:15:22.5659829Z",
    "Predictions":[
        {"TagId":"35ac2ad0-e3ef-4e60-b81f-052a1057a1ca","Tag":"dog","Probability":0.102716163},
        {"TagId":"28e1a872-3776-434c-8cf0-b612dd1a953c","Tag":"cat","Probability":0.02037274}
    ]
}
```

## <a name="next-steps"></a>後續步驟

[匯出模型以供行動裝置使用](export-your-model.md)
