---
title: 快速入門：專案答案搜尋 C#
titlesuffix: Azure Cognitive Services
description: 開始使用專案答案搜尋與 C# 的程式碼範例。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-answer-search
ms.topic: quickstart
ms.date: 04/13/2018
ms.author: rosh
ms.openlocfilehash: bea98ac4c4f7da42d0b41a279508253b987ad9a6
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48867216"
---
# <a name="quickstart-project-answer-search-query-in-c"></a>快速入門：C# 中的專案答案搜尋查詢

下列 C# 範例會建立並傳送微積分第三規則相關資訊的查詢。

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

取得免費試用版[認知服務實驗室](https://aka.ms/answersearchsubscription)的存取金鑰

## <a name="code-scenario"></a>程式碼案例

下列 C# 程式碼會建立並傳送查詢。 

實際進行步驟如下：
1. 宣告變數以指定用於預覽的端點和查詢 URL。  
2. 建立要求。
3. 新增 *Ocp-Apim-Subscription-Key* 標題。 
4. 以非同步方式執行 Web 要求。 
5. 讀取回應。
6. 將標頭和 JSON 結果列印到主控台。

**原始程式碼** \(英文\)

```
using System;
using System.Collections.Generic;
using System.Text;
using System.Net;
using System.IO;

namespace Answers_csharp
{
    class Program
    {
        // Replace the accessKey string value with your valid access key.
        const string accessKey = "YOUR-SUBSCRIPTION-KEY";

        const string uriBase = "https://api.labs.cognitive.microsoft.com/answerSearch/v7.0/search"; 

        const string searchTerm = "third law of calculus"; 

        // Used to return news search results including relevant headers
        struct SearchResult
        {
            public String jsonResult;
            public Dictionary<String, String> relevantHeaders;
        }

        static void Main()
        {
            Console.OutputEncoding = System.Text.Encoding.UTF8;
            Console.WriteLine("Searching locally for: " + searchTerm);

            SearchResult result = BingLocalSearch(searchTerm);

            Console.WriteLine("\nRelevant HTTP Headers:\n");
            foreach (var header in result.relevantHeaders)
                Console.WriteLine(header.Key + ": " + header.Value);

            Console.WriteLine("\nJSON Response:\n");
            Console.WriteLine(JsonPrettyPrint(result.jsonResult));

            Console.Write("\nPress Enter to exit ");
            Console.ReadLine();
        }

        /// <summary>
        /// Does Bing knowledge search and returns the results as a SearchResult.
        /// </summary>
        static SearchResult BingLocalSearch(string searchQuery)
        {
            // Construct the URI of the search request
            var uriQuery = uriBase + "?q=" + Uri.EscapeDataString(searchQuery) + "&mkt=en-us";

            // Send the Web request and get the response.
            WebRequest request = HttpWebRequest.Create(uriQuery);
            request.Headers["Ocp-Apim-Subscription-Key"] = accessKey; 

            HttpWebResponse response = (HttpWebResponse)request.GetResponseAsync().Result;
            string json = new StreamReader(response.GetResponseStream()).ReadToEnd();

            // Create result object for return
            var searchResult = new SearchResult();
            searchResult.jsonResult = json;
            searchResult.relevantHeaders = new Dictionary<String, String>();

            // Extract Bing HTTP headers
            foreach (String header in response.Headers)
            {
                if (header.StartsWith("BingAPIs-") || header.StartsWith("X-MSEdge-"))
                    searchResult.relevantHeaders[header] = response.Headers[header];
            }

            return searchResult;
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

            StringBuilder sb = new StringBuilder();
            bool quote = false;
            bool ignore = false;
            int offset = 0;
            int indentLength = 3;

            foreach (char ch in json)
            {
                switch (ch)
                {
                    case '"':
                        if (!ignore) quote = !quote;
                        break;
                    case '\'':
                        if (quote) ignore = !ignore;
                        break;
                }

                if (quote)
                    sb.Append(ch);
                else
                {
                    switch (ch)
                    {
                        case '{':
                        case '[':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', ++offset * indentLength));
                            break;
                        case '}':
                        case ']':
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', --offset * indentLength));
                            sb.Append(ch);
                            break;
                        case ',':
                            sb.Append(ch);
                            sb.Append(Environment.NewLine);
                            sb.Append(new string(' ', offset * indentLength));
                            break;
                        case ':':
                            sb.Append(ch);
                            sb.Append(' ');
                            break;
                        default:
                            if (ch != ' ') sb.Append(ch);
                            break;
                    }
                }
            }

            return sb.ToString().Trim();
        }

    }
}

```
## <a name="running-the-application"></a>執行應用程式

若要執行應用程式：

1. 在 Visual Studio 中建立新的主控台解決方案。
2. 將 `Program.cs` 取代為提供的程式碼。
3. 將 `YOUR-ACCESS-KEY` 值取代為您訂用帳戶的有效存取金鑰。
4. 執行程式。

## <a name="next-steps"></a>後續步驟
[JAVA 快速入門](java-quickstart.md)
