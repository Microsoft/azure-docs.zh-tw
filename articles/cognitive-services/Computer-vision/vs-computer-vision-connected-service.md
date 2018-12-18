---
title: 教學課程：電腦視覺 C#
titleSuffix: Azure Cognitive Services
description: 從 ASP.NET Core Web 應用程式連線至電腦視覺 API。
services: cognitive-services
author: ghogen
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: Tutorial
ms.date: 03/01/2018
ms.author: ghogen
ms.openlocfilehash: ebfcabdea1e83a83af5eea8025ba5a411c3f9880
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077945"
---
# <a name="use-connected-services-in-visual-studio-to-connect-to-the-computer-vision-api"></a>使用 Visual Studio 中已連線的服務來連線到電腦視覺 API

透過使用認知服務電腦視覺 API，您可以擷取豐富資訊，進而分類及處理視覺化資料，並對影像進行機器輔助審核，以協助籌備您的服務。

這篇文章和其附屬文件提供使用適用於認知服務電腦視覺 API 的 Visual Studio 連線服務功能詳細資料。 此功能適用於已安裝認知服務擴充功能的 Visual Studio 2017 15.7 或更新版本。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**。 如果您沒有訂用帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
- **Visual Studio 2017 版本 15.7**，並且已安裝**網頁程式開發**工作負載。 [立即下載](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs)。

[!INCLUDE [vs-install-cognitive-services-vsix](../../../includes/vs-install-cognitive-services-vsix.md)]

## <a name="add-support-to-your-project-for-cognitive-services-computer-vision-api"></a>對您的專案新增認知服務電腦視覺 API 的支援

1. 建立新的 ASP.NET Core Web 專案。 使用空白專案範本。 

1. 在 [方案總管] 中，選擇 [新增] > [連線服務]。
   [連線服務] 頁面隨即出現，並顯示您可新增至專案的服務。

   ![[新增連線服務] 功能表項目](../media/vs-common/Connected-Service-Menu.PNG)

1. 在可用服務的功能表中，選擇 [認知服務電腦視覺 API]。

   ![選擇要連線的服務](./media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-0.PNG)

   如果您已登入 Visual Studio，並具有與您帳戶相關聯的 Azure 訂用帳戶，則會出現一個頁面，其中顯示含有您訂用帳戶的下拉式清單。

   ![選取您的訂用帳戶](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-1.PNG)

1. 選取您想要使用的訂用帳戶，然後選擇電腦視覺 API 的名稱，或選擇 [編輯] 連結來修改自動產生的名稱，選擇資源群組及定價層。

   ![編輯連線服務詳細資料](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-2.PNG)

   連入此連結，以取得定價層的詳細資料。

1. 選擇 [新增] 以新增連線服務的支援。
   Visual Studio 會修改專案，以新增 NuGet 套件、設定檔項目及其他變更，以支援電腦視覺 API 連線。 [輸出] 視窗會顯示您的專案發生什麼情形的記錄。 您應該會看到如下的內容：

   ```output
   [4/26/2018 5:15:31.664 PM] Adding Computer Vision API to the project.
   [4/26/2018 5:15:32.084 PM] Creating new ComputerVision...
   [4/26/2018 5:15:32.153 PM] Creating new Resource Group...
   [4/26/2018 5:15:40.286 PM] Installing NuGet package 'Microsoft.Azure.CognitiveServices.Vision.ComputerVision' version 2.1.0.
   [4/26/2018 5:15:44.117 PM] Retrieving keys...
   [4/26/2018 5:15:45.602 PM] Changing appsettings.json setting: ComputerVisionAPI_ServiceKey=<service key>
   [4/26/2018 5:15:45.606 PM] Changing appsettings.json setting: ComputerVisionAPI_ServiceEndPoint=https://australiaeast.api.cognitive.microsoft.com/vision/v2.0
   [4/26/2018 5:15:45.609 PM] Changing appsettings.json setting: ComputerVisionAPI_Name=WebApplication-Core-ComputerVision_ComputerVisionAPI
   [4/26/2018 5:15:46.747 PM] Successfully added Computer Vision API to the project.
   ```
 
## <a name="use-the-computer-vision-api-to-detect-attributes-of-an-image"></a>使用電腦視覺 API 來偵測影像的屬性

1. 在 Startup.cs 中，新增下列 using 陳述式。
 
   ```csharp
   using System.IO;
   using System.Text;
   using Microsoft.Extensions.Configuration;
   using System.Net.Http;
   using System.Net.Http.Headers;
   ```
 
1. 新增設定欄位，然後在 `Startup` 類別中，新增可初始化設定欄位的建構函式，以啟用程式中的設定。

   ```csharp
      private IConfiguration configuration;

      public Startup(IConfiguration configuration)
      {
          this.configuration = configuration;
      }
   ```

1. 在專案的 wwwroot 資料夾中，新增影像資料夾，然後將影像檔案新增至 wwwroot 資料夾。 例如，您可以使用這個[電腦視覺 API 頁面](https://azure.microsoft.com/services/cognitive-services/computer-vision/)上的其中一個影像。 在其中一個影像上按一下滑鼠右鍵，儲存到本機硬碟機，然後在 [方案總管] 中的影像資料夾上按一下滑鼠右鍵，然後選擇 [新增] > [現有項目]，將它新增至您的專案。 您的專案在 [方案總管] 中看起來應該如下所示： 
  
   ![含有影像檔案的影像資料夾](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-3.PNG) 

1. 在影像檔案上按一下滑鼠右鍵，選擇 [屬性]，然後選擇 [有更新時才複製]。 

   ![有更新時才複製](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-5.PNG) 
 
1. 以下列程式碼取代 Configure 方法，以存取電腦視覺 API 和測試影像。

   ```csharp
        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            // TODO: Change this to your image's path on your site. 
            string imagePath = @"images/subway.png";

            // Enable static files such as image files. 
            app.UseStaticFiles();

            string visionApiKey = this.configuration["ComputerVisionAPI_ServiceKey"];
            string visionApiEndPoint = this.configuration["ComputerVisionAPI_ServiceEndPoint"];

            HttpClient client = new HttpClient();

            // Request headers.
            client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", visionApiKey);

            // Request parameters. A third optional parameter is "details".
            string requestParameters = "visualFeatures=Categories,Description,Color&language=en";

            // Assemble the URI for the REST API Call.
            string uri = visionApiEndPoint + "/analyze" + "?" + requestParameters;

            HttpResponseMessage response;

            // Request body. Posts an image you've added to your site's images folder. 
            var fileInfo = env.WebRootFileProvider.GetFileInfo(imagePath);
            byte[] byteData = GetImageAsByteArray(fileInfo.PhysicalPath);

            string contentString = string.Empty;
            using (ByteArrayContent content = new ByteArrayContent(byteData))
            {
                // This example uses content type "application/octet-stream".
                // The other content types you can use are "application/json" and "multipart/form-data".
                content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");

                // Execute the REST API call.
                response = client.PostAsync(uri, content).Result;

                // Get the JSON response.
                contentString = response.Content.ReadAsStringAsync().Result;
            }

            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.Run(async (context) =>
            {
                await context.Response.WriteAsync("<h1>Cognitive Services Demo</h1>");
                await context.Response.WriteAsync($"<p><b>Test Image:</b></p>");
                await context.Response.WriteAsync($"<div><img src=\"" + imagePath + "\" /></div>");
                await context.Response.WriteAsync($"<p><b>Computer Vision API results:</b></p>");
                await context.Response.WriteAsync("<p>");
                await context.Response.WriteAsync(JsonPrettyPrint(contentString));
                await context.Response.WriteAsync("<p>");
            });
        }

   ```
    此處程式碼建構的 HTTP 要求包含 URI 和影像作為二進位內容，用於電腦視覺 REST API 的呼叫。

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

1. 執行 Web 應用程式，並查看電腦視覺 API 在影像中找到什麼。

   ![電腦視覺 API 影像與格式化的結果](media/vs-computer-vision-connected-service/Cog-Vision-Connected-Service-4.PNG)  

## <a name="clean-up-resources"></a>清除資源

不再需要資源群組時，請加以刪除。 這會刪除認知服務與相關資源。 若要透過入口網站刪除資源群組：

1. 在入口網站頂端的 [搜尋] 方塊中，輸入資源群組的名稱。 當您在搜尋結果中看到本快速入門中使用的資源群組時，請加以選取。
2. 選取 [刪除資源群組]。
3. 在 [輸入資源群組名稱:] 方塊中輸入資源群組的名稱，然後選取 [刪除]。

## <a name="next-steps"></a>後續步驟

藉由閱讀[電腦視覺 API 文件](Home.md)，來深入了解電腦視覺 API。
