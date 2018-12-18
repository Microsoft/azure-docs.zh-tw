---
title: Go 語言快速入門 - 變更模型及訓練 LUIS 應用程式
description: 在此 Go 語言快速入門中，會將範例語句新增至「家庭自動化」應用程式，並且訓練應用程式。 範例語句是對應到意圖的交談使用者文字。 您可以藉由提供與意圖相關的範例語句，教導 LUIS 哪些種類的使用者提供文字屬於哪種意圖。
titleSuffix: Microsoft Cognitive Services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: quickstart
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 2cd8c1aa2deb2af97d4f720efa0e40146b15b888
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031861"
---
# <a name="quickstart-change-model-using-go"></a>快速入門：使用 Go 來變更模型

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-endpoint-intro-para.md)]

## <a name="prerequisites"></a>必要條件

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-change-model-prereq.md)]
* 安裝 [Go](https://golang.org/) (英文) 程式設計語言。
* [VSCode](https://code.visualstudio.com) \(英文\) 

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-change-model-luis-repo-note.md)]

## <a name="example-utterances-json-file"></a>範例語句 JSON 檔案

[!INCLUDE [Quickstart introduction for endpoint](../../../includes/cognitive-services-luis-qs-change-model-json-ex-utt.md)]

## <a name="create-quickstart-code"></a>建立快速入門程式碼 

1. 使用 VSCode 建立 `add-utterances.go`。 

2. 新增相依性。 

   [!code-go[Add dependencies.](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=2-10 "Add dependencies.")]

3. 新增泛型 HTTP 要求函式，在標頭中包含傳遞製作金鑰。 

   [!code-go[Add HTTP request function which includes passing authoring key in header. ](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=12-36 "Add HTTP request function, which includes passing authoring key in header. ")]

4. 從 JSON 檔案新增範例語句。

   [!code-go[Add example utterances from JSON file.](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=62-76 "Add example utterances from JSON file.")]

5. 要求定型。 使用 helper 函式，將相同路由的 VERB 設為定型狀態。 

   [!code-go[Request training. ](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=77-86 "Request training. ")]

6. 要求定型狀態。 使用 helper 函式，將相同路由的 VERB 設為要求定型。 

   [!code-go[Request training status. ](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=87-90 "Request training status. ")]

7. 新增 main 函式以處理命令列剖析。

   [!code-go[Add main function to handle command line parsing. ](~/samples-luis/documentation-samples/quickstarts/change-model/go/add-utterances.go?range=38-60 "Add main function to handle command-line parsing.")]

## <a name="add-an-utterance-from-the-command-line-train-and-get-status"></a>從命令列新增語句、定型及取得狀態

1. 從您建立 Go 檔案所在目錄中的命令提示字元，輸入 `go build add-utterances.go` 以編譯 Go 檔案。 命令提示字元不會傳回成功組建的任何資訊。

2. 藉由在命令提示字元中輸入下列文字，從命令列執行 Go 應用程式： 

    ```CMD
    add-utterances -appID <your-app-id> -authoringKey <add-your-authoring-key> -version <your-version-id> -region westus -utteranceFile utterances.json

    ```

    使用您的製作金鑰值 (也稱為入門金鑰) 取代 `<add-your-authoring-key>`。 使用您的應用程式識別碼值取代 `<your-app-id>`。 使用您的版本值取代 `<your-version-id>`。 預設版本為 `0.1`。

    這個命令提示字元會顯示結果：

    ```CMD
    add example utterances requested
    [
        {
            "text": "go lang 1",
            "intentName": "None",
            "entityLabels": []
        }
    ,
        {
            "text": "go lang 2",
            "intentName": "None",
            "entityLabels": []
        }
    ]
    201
    [
        {
            "value": {
                "ExampleId": 77783998,
                "UtteranceText": "go lang 1"
            },
            "hasError": false
        },
        {
            "value": {
                "ExampleId": 77783999,
                "UtteranceText": "go lang 2"
            },
            "hasError": false
        }
    ]
    training selected
    202
    {"statusId":9,"status":"Queued"}
    training status selected
    200
    [
        {
            "modelId": "c52d6509-9261-459e-90bc-b3c872ee4a4b",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "5119cbe8-97a1-4c1f-85e6-6449f3a38d77",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "01e6b6bc-9872-47f9-8a52-da510cddfafe",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "33b409b2-32b0-4b0c-9e91-31c6cfaf93fb",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "1fb210be-2a19-496d-bb72-e0c2dd35cbc1",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "3d098beb-a1aa-423f-a0ae-ce08ced216d6",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "cce854f8-8f8f-4ed9-a7df-44dfea562f62",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        },
        {
            "modelId": "4d97bf0d-5213-4502-9712-2d6e77c96045",
            "details": {
                "statusId": 3,
                "status": "InProgress",
                "exampleCount": 24
            }
        }
    ]
    ```

    此回應包含三個 HTTP 呼叫的 HTTP 狀態碼，以及在回應主體中傳回的任何 JSON 回應。 

## <a name="clean-up-resources"></a>清除資源
當您完成快速入門時，請移除在本快速入門中建立的所有檔案。 

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [建置具有自訂網域的應用程式](luis-quickstart-intents-only.md) 