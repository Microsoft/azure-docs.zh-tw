---
title: 快速入門：專案 URL 預覽 C#
titlesuffix: Azure Cognitive Services
description: 開始使用專案 URL 預覽與 C#。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-url-preview
ms.topic: quickstart
ms.date: 03/16/2018
ms.author: rosh
ms.openlocfilehash: 8d31d3a83f9873ce550b9c78626eea0d96ac39bb
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48867437"
---
# <a name="quickstart-url-preview-query-in-c"></a>快速入門：C# 中的 URL 預覽查詢

下列 C# 範例會建立 SwiftKey 網站的 URL 預覽： https://swiftkey.com/en。

## <a name="prerequisites"></a>必要條件

您將需要 [Visual Studio 2017](https://www.visualstudio.com/downloads/) 以在 Windows 上執行此程式碼。 (可使用免費的 Community Edition)。

取得免費試用版[認知服務實驗室](https://aka.ms/answersearchsubscription)的存取金鑰

## <a name="code-scenario"></a>程式碼案例

下列 C# 程式碼會建立 SwiftKey 網站的 URL 預覽： https://swiftkey.com/en。 

其實作的步驟如下：
1. 宣告變數以指定用於預覽的端點和查詢 URL。  
2. 建立要求。
3. 新增 *Ocp-Apim-Subscription-Key* 標題。 
4. 以非同步方式執行 Web 要求。 
5. 讀取回應。
6. 將標頭和 JSON 結果列印到主控台。

**原始程式碼** \(英文\)

```
using System;
using System.IO;
using System.Net;
using System.Text;

namespace UrlPrevCshp
{
    class Program
    {
        static void Main(string[] args)
        {
            String uriBase = "https://api.labs.cognitive.microsoft.com/urlpreview/v7.0/search";
            String searchQuery = "https://swiftkey.com/en"; 
            var uriQuery = uriBase + "?q=" + Uri.EscapeDataString(searchQuery);

            // Do the Web request and get response
            WebRequest request = HttpWebRequest.Create(uriQuery);
            request.Headers["Ocp-Apim-Subscription-Key"] = "YOUR-SUBSCRIPTION-KEY";
            HttpWebResponse response = (HttpWebResponse)request.GetResponseAsync().Result;
            string json = new StreamReader(response.GetResponseStream()).ReadToEnd();

            Console.WriteLine("\nHTTP Headers:\n");
            foreach (String header in response.Headers)
            {
                if (header.StartsWith("BingAPIs-") || header.StartsWith("X-MSEdge-"))
                    Console.WriteLine(header + ": " + response.Headers[header]);
            }

            Console.WriteLine(JsonPrettyPrint(json));
            

            Console.ReadKey();
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
            char last = ' ';
            int offset = 0;
            int indentLength = 2;

            foreach (char ch in json)
            {
                switch (ch)
                {
                    case '"':
                        if (!ignore) quote = !quote;
                        break;
                    case '\\':
                        if (quote && last != '\\') ignore = true;
                        break;
                }

                if (quote)
                {
                    sb.Append(ch);
                    if (last == '\\' && ignore) ignore = false;
                }
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
                            if (quote || ch != ' ') sb.Append(ch);
                            break;
                    }
                }
                last = ch;
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
- [JAVA 快速入門](java-quickstart.md)
- [JavaScript 快速入門](javascript.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)