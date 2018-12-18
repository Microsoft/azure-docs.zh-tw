---
title: 快速入門：從文字識別語言 - 翻譯工具文字、Go
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用翻譯工具文字 API 搭配 Go，識別來源文字的語言。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: quickstart
ms.date: 06/29/2018
ms.author: nolachar
ms.openlocfilehash: 25834fae2ce60d93ebeb3a25298b97294748bf9b
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46129704"
---
# <a name="quickstart-identify-language-from-text-with-go"></a>快速入門：使用 Go 從文字識別語言

在本快速入門中，您可以使用翻譯工具文字 API，識別來源文字的語言。

## <a name="prerequisites"></a>必要條件

您必須安裝 [Go 發佈](https://golang.org/doc/install) (英文) 才能執行此程式碼。 範例程式碼只會使用**核心**程式庫，因此沒有外部相依性。

若要使用翻譯工具文字 API，您也需要有訂用帳戶金鑰；請參閱[如何註冊翻譯工具文字 API](translator-text-how-to-signup.md)。

## <a name="detect-request"></a>Detect 要求

下列程式碼會使用 [Detect](./reference/v3-0-detect.md) 方法，以識別來源文字的語言。

1. 在您慣用的程式碼編輯器中，建立新的 Go 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 以 '.go' 副檔名儲存檔案。
5. 在已安裝 Go 的電腦上開啟命令提示字元。
6. 建置檔案，例如：'go build quickstart-detect.go'。
7. 執行檔案，例如：'quickstart-detect'。

```golang
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "strconv"
    "strings"
    "time"
)

func main() {
    // Replace the subscriptionKey string value with your valid subscription key
    const subscriptionKey = "<Subscription Key>"

    const uriBase = "https://api.cognitive.microsofttranslator.com"
    const uriPath = "/detect?api-version=3.0"

    const uri = uriBase + uriPath

    const text = "Salve, mondo!"

    r := strings.NewReader("[{\"Text\" : \"" + text + "\"}]")

    client := &http.Client{
        Timeout: time.Second * 2,
    }

    req, err := http.NewRequest("POST", uri, r)
    if err != nil {
        fmt.Printf("Error creating request: %v\n", err)
        return
    }

    req.Header.Add("Content-Type", "application/json")
    req.Header.Add("Content-Length", strconv.FormatInt(req.ContentLength, 10))
    req.Header.Add("Ocp-Apim-Subscription-Key", subscriptionKey)

    resp, err := client.Do(req)
    if err != nil {
        fmt.Printf("Error on request: %v\n", err)
        return
    }
    defer resp.Body.Close()

    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        fmt.Printf("Error reading response body: %v\n", err)
        return
    }

    var f interface{}
    json.Unmarshal(body, &f)

    jsonFormatted, err := json.MarshalIndent(f, "", "  ")
    if err != nil {
        fmt.Printf("Error producing JSON: %v\n", err)
        return
    }
    fmt.Println(string(jsonFormatted))
}
```

## <a name="detect-response"></a>Detect 回應

成功的回應會以 JSON 格式傳回，如下列範例所示：

```json
[
  {
    "alternatives": [
      {
        "isTranslationSupported": true,
        "isTransliterationSupported": false,
        "language": "pt",
        "score": 1
      },
      {
        "isTranslationSupported": true,
        "isTransliterationSupported": false,
        "language": "en",
        "score": 1
      }
    ],
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "language": "it",
    "score": 1
  }
]
```

## <a name="next-steps"></a>後續步驟

從 GitHub 上的 [Azure SDK for Go](https://github.com/Azure/azure-sdk-for-go) (英文)，探索認知服務 API 的 Go 套件。

> [!div class="nextstepaction"]
> [探索 GitHub 上的 Go 套件](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices) (英文)
