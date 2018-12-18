---
title: 教學課程：臉部 API C#
titleSuffix: Azure Cognitive Services
description: 建立使用認知服務臉部 API 的簡單 Windows 應用程式，以偵測影像中的臉部特徵。
services: cognitive-services
author: ghogen
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: tutorial
ms.date: 05/07/2018
ms.author: ghogen
ms.openlocfilehash: 70414674e563cf3703d2cf3ebc57f09afcdf2691
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129517"
---
# <a name="connecting-to-cognitive-services-face-api-by-using-connected-services-in-visual-studio"></a>使用 Visual Studio 中的連線服務連線至認知服務臉部 API

使用認知服務臉部 API，您可以偵測、分析、組織及標記相片中的臉孔。

這篇文章和其附屬文件提供使用適用於認知服務臉部 API 的 Visual Studio 連線服務功能詳細資料。 此功能適用於已安裝認知服務擴充功能的 Visual Studio 2017 15.7 或更新版本。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
- **Visual Studio 2017 版本 15.7**，並且已安裝**網頁程式開發**工作負載。 [立即下載](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

[!INCLUDE [vs-install-cognitive-services-vsix](../../../includes/vs-install-cognitive-services-vsix.md)]

## <a name="create-a-project-and-add-support-for-cognitive-services-face-api"></a>建立專案並新增認知服務臉部 API 的支援

1. 建立新的 ASP.NET Core Web 專案。 使用空白專案範本。 

1. 在 [方案總管] 中，選擇 [新增] > [連線服務]。
   [連線服務] 頁面隨即出現，並顯示您可新增至專案的服務。

   ![[新增連線服務] 功能表項目](./media/vs-face-connected-service/Connected-Service-Menu.PNG)

1. 在可用服務的功能表中，選擇 [認知服務臉部 API]。

   ![選擇要連線的服務](./media/vs-face-connected-service/Cog-Face-Connected-Service-0.PNG)

   如果您已登入 Visual Studio，並具有與您帳戶相關聯的 Azure 訂用帳戶，則會出現一個頁面，其中顯示含有您訂用帳戶的下拉式清單。

   ![選取您的訂用帳戶](media/vs-face-connected-service/Cog-Face-Connected-Service-1.PNG)

1. 選取您想要使用的訂用帳戶，然後選擇臉部 API 的名稱，或選擇 [編輯] 連結來修改自動產生的名稱，選擇資源群組及定價層。

   ![編輯連線服務詳細資料](media/vs-face-connected-service/Cog-Face-Connected-Service-2.PNG)

   連入此連結，以取得定價層的詳細資料。

1. 選擇 [新增] 以新增連線服務的支援。
   Visual Studio 會修改專案，以新增 NuGet 套件、設定檔項目及其他變更，以支援臉部 API 連線。

## <a name="use-the-face-api-to-detect-attributes-of-faces-in-an-image"></a>使用臉部 API 來偵測影像中的臉部屬性

1. 在 Startup.cs 中，新增下列 using 陳述式。
 
   ```csharp
   using System.IO;
   using System.Text;
   using Microsoft.Extensions.Configuration;
   using System.Net.Http;
   using System.Net.Http.Headers;
   ```
 
1. 新增設定欄位，然後在 Startup 類別中，新增初始化設定欄位的建構函式，以啟用程式中的設定。

   ```csharp
      private IConfiguration configuration;

      public Startup(IConfiguration configuration)
      {
          this.configuration = configuration;
      }
   ```

1. 在專案的 wwwroot 資料夾中，新增影像資料夾，然後將影像檔案新增至 wwwroot 資料夾。 例如，您可以使用這個[臉部 API 頁面](https://azure.microsoft.com/services/cognitive-services/face/)上的其中一個影像。 在其中一個影像上按一下滑鼠右鍵，儲存到本機硬碟，然後在 [方案總管] 中的影像資料夾上按一下滑鼠右鍵，然後選擇 [新增] > [現有項目]，將它新增至您的專案。 您的專案在 [方案總管] 中看起來應該如下所示：
 
   ![含有影像檔案的影像資料夾](media/vs-face-connected-service/Cog-Face-Connected-Service-6.PNG)

1. 在影像檔案上按一下滑鼠右鍵，選擇 [屬性]，然後選擇 [有更新時才複製]。

   ![有更新時才複製](media/vs-face-connected-service/Cog-Face-Connected-Service-5.PNG)
 
1. 以下列程式碼取代 Configure 方法，以存取臉部 API 和測試影像。 將 imagePath 字串變更為臉部影像的正確路徑和檔案名稱。

   ```csharp
    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            // TODO: Change this to your image's path on your site.
            string imagePath = @"images/face1.png";

            // Enable static files such as image files.
            app.UseStaticFiles();

            string faceApiKey = this.configuration["FaceAPI_ServiceKey"];
            string faceApiEndPoint = this.configuration["FaceAPI_ServiceEndPoint"];

            HttpClient client = new HttpClient();

            // Request headers.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", faceApiKey);

            // Request parameters. A third optional parameter is "details".
            string requestParameters = "returnFaceId=true&returnFaceLandmarks=false&returnFaceAttributes=age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise";

            // Assemble the URI for the REST API Call.
            string uri = faceApiEndPoint + "/detect?" + requestParameters;

            // Request body. Posts an image you've added to your site's images folder.
            var fileInfo = env.WebRootFileProvider.GetFileInfo(imagePath);
            var byteData = GetImageAsByteArray(fileInfo.PhysicalPath);

            string contentStringFace = string.Empty;
            using (ByteArrayContent content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");

                // Execute the REST API call.
                var response = client.PostAsync(uri, content).Result;

                // Get the JSON response.
                contentStringFace = response.Content.ReadAsStringAsync().Result;
            }

            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.Run(async (context) =>
            {
                await context.Response.WriteAsync($"<p><b>Face Image:</b></p>");
                await context.Response.WriteAsync($"<div><img src=\"" + imagePath + "\" /></div>");
                await context.Response.WriteAsync($"<p><b>Face detection API results:</b></p>");
                await context.Response.WriteAsync("<p>");
                await context.Response.WriteAsync(JsonPrettyPrint(contentStringFace));
                await context.Response.WriteAsync("<p>");
            });
        }
   ```
    此步驟中的程式碼使用您在新增連線服務時所新增的金鑰，建構含有呼叫臉部 REST API 的 HTTP 要求。

1. 加入 GetImageAsByteArray 與 JsonPrettyPrint 協助程式函式。

   ```csharp
        /// <summary>
        /// Returns the contents of the specified file as a byte array.
        /// </summary>
        /// <param name="imageFilePath">The image file to read.</param>
        /// <returns>The byte array of the image data.</returns>
        static byte[] GetImageAsByteArray(string imageFilePath)
        {
            FileStream fileStream = new FileStream(imageFilePath, FileMode.Open, FileAccess.Read);
            BinaryReader binaryReader = new BinaryReader(fileStream);
            return binaryReader.ReadBytes((int)fileStream.Length);
        }

        /// <summary>
        /// Formats the given JSON string by adding line breaks and indents.
        /// </summary>
        /// <param name="json">The raw JSON string to format.</param>
        /// <returns>The formatted JSON string.</returns>
        static string JsonPrettyPrint(string json)
        {
            if (string.IsNullOrEmpty(json))
                return string.Empty;

            json = json.Replace(Environment.NewLine, "").Replace("\t", "");

            string INDENT_STRING = "    ";
            var indent = 0;
            var quoted = false;
            var sb = new StringBuilder();
            for (var i = 0; i < json.Length; i++)
            {
                var ch = json[i];
                switch (ch)
                {
                    case '{':
                    case '[':
                        sb.Append(ch);
                        if (!quoted)
                        {
                            sb.AppendLine();
                        }
                        break;
                    case '}':
                    case ']':
                        if (!quoted)
                        {
                            sb.AppendLine();
                        }
                        sb.Append(ch);
                        break;
                    case '"':
                        sb.Append(ch);
                        bool escaped = false;
                        var index = i;
                        while (index > 0 && json[--index] == '\\')
                            escaped = !escaped;
                        if (!escaped)
                            quoted = !quoted;
                        break;
                    case ',':
                        sb.Append(ch);
                        if (!quoted)
                        {
                            sb.AppendLine();
                        }
                        break;
                    case ':':
                        sb.Append(ch);
                        if (!quoted)
                            sb.Append(" ");
                        break;
                    default:
                        sb.Append(ch);
                        break;
                }
            }
            return sb.ToString();
        }
   ```

1. 執行 Web 應用程式，並查看臉部 API 在影像中找到什麼。
 
   ![臉部 API 影像與格式化的結果](media/vs-face-connected-service/Cog-Face-Connected-Service-4.PNG)

## <a name="clean-up-resources"></a>清除資源

不再需要資源群組時，請將它刪除。 這會刪除認知服務與相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 當您在搜尋結果中看到本快速入門中使用的資源群組時，請選取它。
1. 選取 [刪除資源群組]。
1. 在 [輸入資源群組名稱:] 方塊中輸入資源群組的名稱，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

藉由閱讀[臉部 API 文件](Overview.md)，深入了解臉部 API。
