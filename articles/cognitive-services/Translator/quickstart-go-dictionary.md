---
title: 快速入門：尋找替代的翻譯 - 翻譯工具文字、Go
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用翻譯工具文字 API 搭配 Go，為內容中的字詞尋找替代的翻譯與範例。
services: cognitive-services
author: noellelacharite
manager: cgronlun
ms.service: cognitive-services
ms.component: translator-text
ms.topic: quickstart
ms.date: 06/29/2018
ms.author: nolachar
ms.openlocfilehash: 4ea21673c993c6dd9478f42bfee790b62317946f
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46128871"
---
# <a name="quickstart-find-alternate-translations-and-usage-with-go"></a>快速入門：使用 Go 尋找替代的翻譯和使用方式

在本快速入門中，您可以使用翻譯工具文字 API，尋找字詞可能的替代翻譯的詳細資料，以及這些替代翻譯的使用範例。

## <a name="prerequisites"></a>必要條件

您必須安裝 [Go 發佈](https://golang.org/doc/install) (英文) 才能執行此程式碼。 範例程式碼只會使用**核心**程式庫，因此沒有外部相依性。

若要使用翻譯工具文字 API，您也需要有訂用帳戶金鑰；請參閱[如何註冊翻譯工具文字 API](translator-text-how-to-signup.md)。

## <a name="dictionary-lookup-request"></a>Dictionary Lookup 要求

下列程式碼會使用 [Dictionary Lookup](./reference/v3-0-dictionary-lookup.md) 方法，以取得字組的替代翻譯。

1. 在您慣用的程式碼編輯器中，建立新的 Go 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 以 '.go' 副檔名儲存檔案。
5. 在已安裝 Go 的電腦上開啟命令提示字元。
6. 建置檔案，例如：'go build quickstart-lookup.go'。
7. 執行檔案，例如：'quickstart-lookup'。

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
    const uriPath = "/dictionary/lookup?api-version=3.0"

    // From English to French
    const params = "&from=en&to=fr"

    const uri = uriBase + uriPath + params

    const text = "great"

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

## <a name="dictionary-lookup-response"></a>Dictionary Lookup 回應

成功的回應會以 JSON 格式傳回，如下列範例所示：

```json
[
  {
    "normalizedSource": "great",
    "displaySource": "great",
    "translations": [
      {
        "normalizedTarget": "grand",
        "displayTarget": "grand",
        "posTag": "ADJ",
        "confidence": 0.2783,
        "prefixWord": "",
        "backTranslations": [
          {
            "normalizedText": "great",
            "displayText": "great",
            "numExamples": 15,
            "frequencyCount": 34358
          },
          {
            "normalizedText": "big",
            "displayText": "big",
            "numExamples": 15,
            "frequencyCount": 21770
          },
...
        ]
      },
      {
        "normalizedTarget": "super",
        "displayTarget": "super",
        "posTag": "ADJ",
        "confidence": 0.1514,
        "prefixWord": "",
        "backTranslations": [
          {
            "normalizedText": "super",
            "displayText": "super",
            "numExamples": 15,
            "frequencyCount": 12023
          },
          {
            "normalizedText": "great",
            "displayText": "great",
            "numExamples": 15,
            "frequencyCount": 10931
          },
...
        ]
      },
...
    ]
  }
]
```

## <a name="dictionary-examples-request"></a>Dictionary Examples 要求

下列程式碼會使用 [Dictionary Examples](./reference/v3-0-dictionary-examples.md) 方法，以取得如何在字典中使用字詞的內容相關範例。

1. 在您慣用的程式碼編輯器中，建立新的 Go 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 以 '.go' 副檔名儲存檔案。
5. 在已安裝 Go 的電腦上開啟命令提示字元。
6. 建置檔案，例如：'go build quickstart-examples.go'。
7. 執行檔案，例如：'quickstart-examples'。

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
    const uriPath = "/dictionary/examples?api-version=3.0"

    // From English to French
    const params = "&from=en&to=fr"

    const uri = uriBase + uriPath + params

    const text = "great"
    const translation = "formidable"

    r := strings.NewReader("[{\"Text\" : \"" + text + "\", \"Translation\" : \"" + translation + "\"}]")

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

## <a name="dictionary-examples-response"></a>Dictionary Examples 回應

成功的回應會以 JSON 格式傳回，如下列範例所示：

```json
[
  {
    "normalizedSource": "great",
    "normalizedTarget": "formidable",
    "examples": [
      {
        "sourcePrefix": "You have a ",
        "sourceTerm": "great",
        "sourceSuffix": " expression there.",
        "targetPrefix": "Vous avez une expression ",
        "targetTerm": "formidable",
        "targetSuffix": "."
      },
      {
        "sourcePrefix": "You played a ",
        "sourceTerm": "great",
        "sourceSuffix": " game today.",
        "targetPrefix": "Vous avez été ",
        "targetTerm": "formidable",
        "targetSuffix": "."
      },
...
    ]
  }
]
```

## <a name="next-steps"></a>後續步驟

從 GitHub 上的 [Azure SDK for Go](https://github.com/Azure/azure-sdk-for-go) (英文)，探索認知服務 API 的 Go 套件。

> [!div class="nextstepaction"]
> [探索 GitHub 上的 Go 套件](https://github.com/Azure/azure-sdk-for-go/tree/master/services/cognitiveservices) (英文)
