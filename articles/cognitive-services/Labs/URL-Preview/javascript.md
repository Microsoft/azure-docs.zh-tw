---
title: 快速入門：專案 URL 預覽 JavaScript
titlesuffix: Azure Cognitive Services
description: 快速開始使用 Bing URL 預覽 API 與 JavaScript 的指令碼範例。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-url-preview
ms.topic: quickstart
ms.date: 03/16/2018
ms.author: rosh
ms.openlocfilehash: b10e33a0947057c35bb6227cc43f92c4f0a56ceb
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48869460"
---
# <a name="quickstart-url-preview-in-javascript"></a>快速入門：JavaScript 中的 URL 預覽 

下列單頁應用程式使用 JavaScript 建立 SwiftKey 網站的 URL 預覽： https://swiftkey.com/en。 

## <a name="prerequisites"></a>必要條件

取得免費試用版[認知服務實驗室](https://labs.cognitive.microsoft.com/en-us/project-url-preview)的存取金鑰

## <a name="code-scenario"></a>程式碼案例
下列 javascript 範例包含使用者輸入預覽 URL 的文字方塊輸入物件。  使用者按一下 [預覽] 按鈕時，onclick 方法會傳送到 `getPreview`，其中的程式碼會產生 **UrlPreview** 端點的 Web 要求。

程式碼會建立 *XMLHttpRequest*，並新增 *Ocp-Apim-Subscription-Key* 標頭和金鑰，然後傳送要求。  這會加入非同步事件處理常式來處理回應。

如果回應成功傳回，此處理常式會將回應的 JSON 文字指派到頁面上的 `demo` 段落。 其他回應元素會設定為下列段落進行顯示。

**未經處理的 JSON 回應**

````
{
  "_type": "WebPage",
  "name": "SwiftKey - Smart prediction technology for easier mobile typing",
  "url": "https://swiftkey.com/en",
  "description": "Discover the best Android and iPhone and iPad apps for faster, easier typing with emoji, colorful themes and more - download SwiftKey Keyboard free today.",
  "isFamilyFriendly": true,
  "primaryImageOfPage": {
    "contentUrl": "https://swiftkey.com/images/og/default.jpg"
  }
}

````

**執行中示範**

![JavaScript Url 預覽範例](./media/java-script-demo.png)

## <a name="running-the-application"></a>執行應用程式

若要執行應用程式：

1. 以您訂用帳戶有效的存取金鑰取代 `YOUR-SUBSCRIPTION-KEY` 值。
2. 將 HTML 和指令碼儲存至 .html 副檔名的檔案中。
3. 在瀏覽器中執行網頁。
4. 使用現有的 URL，或在文字方塊中輸入其他 URL。
5. 按一下 [預覽] 按鈕。

**原始程式碼**：

```
<!DOCTYPE html>

<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
    <title>urlPreview Demo</title>
</head>
<body>
    <h3>URL Preview Demo</h3>

    Page to preview: <input type="url" id="myURL" value="https://swiftkey.com/en">
    <button onclick="getPreview()">Preview</button>

    <p id="demo"></p>
    <br />
    <p id="jsonDesc"></p>
    <p><a id="familyFriendly"></a></p>
    <a id="contentUrl"></a>
    <p />
    <img id="jsonImage" />

    <script>

        var BING_ENDPOINT = "https://api.labs.cognitive.microsoft.com/urlpreview/v7.0/search"; 
        var key = "YOUR-SUBSCRIPTION-KEY";
        var xhr;

        function getPreview() {
            xhr = new XMLHttpRequest();

            var queryUrl = BING_ENDPOINT + "?q=" +
                encodeURIComponent(document.getElementById("myURL").value);
            xhr.open('GET', queryUrl, true);
            xhr.setRequestHeader("Ocp-Apim-Subscription-Key", key);

            xhr.send();

            xhr.addEventListener("readystatechange", processRequest, false);
        }

        function processRequest(e) {

            if (xhr.readyState == 4 && xhr.status == 200) {
                document.getElementById("demo").innerHTML = xhr.responseText;
                var obj = JSON.parse(xhr.responseText);
                document.getElementById("jsonDesc").innerHTML = obj.description;
                document.getElementById("familyFriendly").innerHTML = "Family Friendly: " + obj.isFamilyFriendly;
                document.getElementById("contentUrl").innerHTML = obj.url;
                document.getElementById("contentUrl").href = obj.url;
                document.getElementById("jsonImage").width = 350;
                document.getElementById("jsonImage").src = obj.primaryImageOfPage.contentUrl;

            }

        }

    </script>

</body>
</html>

```

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](csharp.md)
- [Java 快速入門](java-quickstart.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)
