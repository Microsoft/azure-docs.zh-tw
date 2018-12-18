---
title: 快速入門：使用 C# 呼叫端點 - Bing 自訂搜尋
titlesuffix: Azure Cognitive Services
description: 本快速入門說明如何使用 C# 呼叫 Bing 自訂搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。
services: cognitive-services
author: brapel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-custom-search
ms.topic: quickstart
ms.date: 05/07/2018
ms.author: v-brapel
ms.openlocfilehash: 1c3b1031c2d08b1f346216b54d351c99f01db933
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167302"
---
# <a name="quickstart-call-bing-custom-search-endpoint-c"></a>快速入門：呼叫 Bing 自訂搜尋端點 (C#)

此快速入門說明如何使用 C# 呼叫 Bing 自訂搜尋端點，以從您的自訂搜尋執行個體要求搜尋結果。 

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，您需要：

- 已經可以使用的自訂搜尋執行個體。 請參閱[建立您的第一個 Bing 自訂搜尋執行個體](quick-start.md)。
- 已安裝 [.Net Core](https://www.microsoft.com/net/download/core)。
- 訂用帳戶金鑰。 您可以在啟用[免費試用版](https://azure.microsoft.com/try/cognitive-services/?api=bing-custom-search)時取得訂用帳戶金鑰，或者可以從 Azure 儀表板使用付費訂用帳戶金鑰 (請參閱[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account))。    


## <a name="run-the-code"></a>執行程式碼

若要執行此範例，請遵循下列步驟：

1. 建立程式碼的資料夾。  
  
2. 從命令提示字元或終端機，巡覽至您剛剛建立的資料夾。  
  
3. 執行下列命令：
    ```
    dotnet new console -o BingCustomSearch
    cd BingCustomSearch
    dotnet add package Newtonsoft.Json
    dotnet restore
    ```
  
4. 將下列程式碼複製到 Program.cs。 將 **YOUR-SUBSCRIPTION-KEY** 與 **YOUR-CUSTOM-CONFIG-ID** 取代為您的訂用帳戶金鑰與設定識別碼。

    ```csharp
    using System;
    using System.Net.Http;
    using System.Web;
    using Newtonsoft.Json;
    
    namespace bing_custom_search_example_dotnet
    {
        class Program
        {
            static void Main(string[] args)
            {
                var subscriptionKey = "YOUR-SUBSCRIPTION-KEY";
                var customConfigId = "YOUR-CUSTOM-CONFIG-ID";
                var searchTerm = args.Length > 0 ? args[0]: "microsoft";            
    
                var url = "https://api.cognitive.microsoft.com/bingcustomsearch/v7.0/search?" +
                    "q=" + searchTerm +
                    "&customconfig=" + customConfigId;
    
                var client = new HttpClient();
                client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", subscriptionKey);
                var httpResponseMessage = client.GetAsync(url).Result;
                var responseContent = httpResponseMessage.Content.ReadAsStringAsync().Result;
                BingCustomSearchResponse response = JsonConvert.DeserializeObject<BingCustomSearchResponse>(responseContent);
                
                for(int i = 0; i < response.webPages.value.Length; i++)
                {                
                    var webPage = response.webPages.value[i];
                    
                    Console.WriteLine("name: " + webPage.name);
                    Console.WriteLine("url: " + webPage.url);                
                    Console.WriteLine("displayUrl: " + webPage.displayUrl);
                    Console.WriteLine("snippet: " + webPage.snippet);
                    Console.WriteLine("dateLastCrawled: " + webPage.dateLastCrawled);
                    Console.WriteLine();
                }            
            }
        }
    
        public class BingCustomSearchResponse
        {        
            public string _type{ get; set; }            
            public WebPages webPages { get; set; }
        }
    
        public class WebPages
        {
            public string webSearchUrl { get; set; }
            public int totalEstimatedMatches { get; set; }
            public WebPage[] value { get; set; }        
        }
    
        public class WebPage
        {
            public string name { get; set; }
            public string url { get; set; }
            public string displayUrl { get; set; }
            public string snippet { get; set; }
            public DateTime dateLastCrawled { get; set; }
            public string cachedPageUrl { get; set; }
            public OpenGraphImage openGraphImage { get; set; }        
        }
        
        public class OpenGraphImage
        {
            public string contentUrl { get; set; }
            public int width { get; set; }
            public int height { get; set; }
        }
    }
    ```
6. 使用下列命令建置應用程式。 記下命令輸出所參考的 DLL 路徑。

    <pre>
    dotnet build 
    </pre>
    
7. 使用下列命令，將 **PATH TO OUTPUT** 取代為步驟 6 中參考的 DLL 路徑，執行應用程式。

    <pre>    
    dotnet **PATH TO OUTPUT**
    </pre>

## <a name="next-steps"></a>後續步驟
- [設定託管的 UI 體驗](./hosted-ui.md)
- [使用裝飾標記醒目提示文字](./hit-highlighting.md)
- [頁面網頁](./page-webpages.md)
