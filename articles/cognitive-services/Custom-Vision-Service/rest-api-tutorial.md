---
title: 教學課程：使用自訂視覺服務 REST API
titlesuffix: Azure Cognitive Services
description: 使用 REST API 來建立、定型、測試及匯出自訂視覺模型。
services: cognitive-services
author: blackmist
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 08/07/2018
ms.author: larryfr
ms.openlocfilehash: a38f737b5281903328a53d6552b1666ca4f58d80
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364953"
---
# <a name="tutorial-use-the-custom-vision-rest-api"></a>教學課程：使用自訂視覺 REST API

了解如何使用「自訂視覺 REST API」來建立、定型、測試及匯出模型。

此文件中的資訊會示範如何使用 REST 用戶端與 REST API 搭配運作，來進行「自訂視覺」服務定型。 範例會從 Bash 環境使用 `curl` 公用程式及從 Windows PowerShell 使用 `Invoke-WebRequest`，來說明如何使用 API。

> [!div class="checklist"]
> * 取得金鑰
> * 建立專案
> * 建立標記
> * 新增影像
> * 進行模型定型及測試
> * 匯出模型

## <a name="prerequisites"></a>必要條件

* 對「具象狀態傳輸」(REST) 有基本的了解。 此文件不詳細探討 HTTP 指令動詞、JSON 或其他常與 REST 搭配使用的項目。

* Bash (Bourne Again Shell) 搭配 [curl](https://curl.haxx.se) 公用程式，或是 Windows PowerShell 3.0 (或更新版本)。

* 一個「自訂視覺」帳戶。 如需詳細資訊，請參閱[建置分類器](getting-started-build-a-classifier.md)文件。

## <a name="get-keys"></a>取得金鑰

使用 REST API 時，您必須使用金鑰來進行驗證。 執行管理或定型作業時，您需使用__定型金鑰__。 使用模型來進行預測時，您需使用__預測金鑰__。

發出要求時，會將金鑰當作要求標頭來傳送。

若要取得您帳戶的金鑰，請瀏覽[自訂視覺網頁](https://customvision.ai) \(英文\)，然後選取右上角的__齒輪圖示__。 在 [帳戶] 區段中，複製 [定型金鑰] 和 [預測金鑰] 欄位的值。

![金鑰 UI 的影像](./media/rest-api-tutorial/training-prediction-keys.png)

> [!IMPORTANT]
> 由於金鑰會用來驗證每個要求，因此此文件中的範例會假設金鑰值包含在環境變數中。 請先使用下列命令將金鑰儲存至環境變數，再使用此文件中的任何其他程式碼片段：
>
> ```bash
> read -p "Enter the training key: " TRAININGKEY
> read -p "Enter the prediction key: " PREDICTIONKEY
> ```
>
> ```powershell
> $trainingKey = Read-Host 'Enter the training key'
> $predictionKey = Read-Host 'Enter the prediction key'
> ```

## <a name="create-a-new-project"></a>建立新專案

下列範例會在您的「自訂視覺」服務執行個體中，建立一個名為 `myproject` 的新專案。 此服務會預設為 `General` 網域：

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects?name=myproject" -H "Training-Key: $TRAININGKEY" --data-ascii ""
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects?name=myproject" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey" }
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
{
  "id":"45d1b19b-69b6-4a22-8e7e-d1ca37504686",
  "name":"myproject",
  "description":"",
  "settings":{
    "domainId":"ee85a72c-405e-4adc-bb47-ffa8ca0c9f31",
    "useNegativeSet":true,
    "classificationType":"Multilabel",
    "detectionParameters":null
  },
  "created":"2018-08-10T17:39:02.5633333",
  "lastModified":"2018-08-10T17:39:02.5633333",
  "thumbnailUri":null
}
```

> [!TIP]
> 回應中的 `id` 項目是新專案的識別碼。 在此文件稍後的其他範例中會用到此識別碼。

如需有關此要求的詳細資訊，請參閱 [CreateProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a8290)。

### <a name="specific-domains"></a>特定網域

若要為特定網域建立專案，您可以提供__網域識別碼__作為選擇性參數。 下列範例說明如何擷取可用網域的清單：

```bash
curl -X GET "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/domains" -H "Training-Key: $TRAININGKEY" --data-ascii ""
```

```powershell
$resp = Invoke-WebRequest -Method 'GET' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/domains" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey" }
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
[
    {
        "id": "ee85a74c-405e-4adc-bb47-ffa8ca0c9f31",
        "name": "General",
        "type": "Classification",
        "exportable": false,
        "enabled": true
    },
    {
        "id": "c151d5b5-dd07-472a-acc8-15d29dea8518",
        "name": "Food",
        "type": "Classification",
        "exportable": false,
        "enabled": true
    },
    {
        "id": "ca455789-012d-4b50-9fec-5bb63841c793",
        "name": "Landmarks",
        "type": "Classification",
        "exportable": false,
        "enabled": true
    },
    ...
]
```

如需有關此要求的詳細資訊，請參閱 [GetDomains](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a827d)。

下列範例示範如何建立使用 __Landmarks__ 網域的新專案：

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects?name=myproject&domainId=ca455789-012d-4b50-9fec-5bb63841c793" -H "Training-Key: $TRAININGKEY" --data-ascii ""
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects?name=myproject&domainId=ca455789-012d-4b50-9fec-5bb63841c793" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey" }
$resp.Content
```

## <a name="create-tags"></a>建立標記

若要標記影像，您必須使用標記識別碼。 下列範例示範如何建立名為 `cat` 的新標記，然後取得標記 ID。 使用您專案的識別碼來取代 `{projectId}`。 使用 `name=` 參數來指定標記的名稱：

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/tags?name=cat" -H "Training-Key: $TRAININGKEY" --data-ascii ""
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/tags?name=cat" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey" }
$resp.Content
```

對要求的回應會與下列 JSON 文件類似： 

```json
{"id":"ed6f7ab6-5132-47ad-8649-3ec42ee62d43","name":"cat","description":null,"imageCount":0}
```

儲存 `id` 值，因為標記影像時會用到它。

如需有關此要求的詳細資訊，請參閱 [CreateTag](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a829d)。

## <a name="add-images"></a>新增影像

下列範例示範如何從 URL 新增檔案。 使用您專案的識別碼來取代 `{projectId}`。 使用影像的標記識別碼來取代 `{tagId}`：

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/images/urls" -H "Training-Key: $TRAININGKEY" -H "Content-Type: application/json" --data-ascii '{"images": [{"url": "http://myimages/cat.jpg","tagIds": ["{tagId}"],"regions": [{"tagId": "{tagId}","left": 119.0,"top": 94.0,"width": 240.0,"height": 140.0}]}], "tagIds": ["{tagId}"]}'
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/images/urls" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey"; "Content-Type"="application/json" } `
    -Body '{"images": [{"url": "http://myimages/cat.jpg","tagIds": ["{tagId}"],"regions": [{"tagId": "{tagId}","left": 119.0,"top": 94.0,"width": 240.0,"height": 140.0}]}], "tagIds": ["{tagId}"]}'
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
{
    "isBatchSuccessful": true,
    "images": [
        {
            "sourceUrl": "http://myimages/cat.jpg",
            "status": "OK",
            "image": {
                "id": "081adaee-a76b-4d94-a70e-e4fd0935a28f",
                "created": "2018-08-13T13:24:22.0815638",
                "width": 640,
                "height": 480,
                "imageUri": "https://linktoimage",
                "thumbnailUri": "https://linktothumbnail",
                "tags": [
                    {
                        "tagId": "ed6f7ab6-5132-47ad-8649-3ec42ee62d43",
                        "tagName": null,
                        "created": "2018-08-13T13:24:22.104936"
                    }
                ],
                "regions": [
                    {
                        "regionId": "40f206a1-3f8a-4de7-a6c3-c7b4643117df",
                        "tagName": null,
                        "created": "2018-08-13T13:24:22.104936",
                        "tagId": "ed6f7ab6-5132-47ad-8649-3ec42ee62d43",
                        "left": 119,
                        "top": 94,
                        "width": 240,
                        "height": 140
                    }
                ]
            }
        }
    ]
}
```

如需有關此要求的詳細資訊，請參閱 [CreateImagesFromUrls](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a8287)。

## <a name="train-the-model"></a>訓練模型

下列範例示範如何進行模型定型。 使用您專案的識別碼來取代 `{projectId}`：

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/train" -H "Training-Key: $TRAININGKEY" -H "Content-Type: application/json" --data-ascii ""
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/train" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey"; "Content-Type"="application/json" } 
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
{
    "id": "23de09d6-42a1-413e-839e-8db6ee6d3496",
    "name": "Iteration 1",
    "isDefault": false,
    "status": "Training",
    "created": "2018-08-10T17:39:02.5766667",
    "lastModified": "2018-08-16T17:15:07.5250661",
    "projectId": "45d1b19b-69b8-4b22-8e7e-d1ca37504686",
    "exportable": false,
    "domainId": null
}
```

儲存 `id` 值，因為會使用它來測試和匯出模型。

如需詳細資訊，請參閱 [TrainProject](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a8294)。

## <a name="test-the-model"></a>測試模型

下列範例示範如何執行模型測試。 使用您專案的識別碼來取代 `{projectId}`。 使用從模型定型傳回的識別碼來取代 `{iterationId}`。 使用測試影像的路徑來取代 `https://linktotestimage`。

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/quicktest/url?iterationId={iterationId}" -H "Training-Key: $TRAININGKEY" -H "Content-Type: application/json" --data-ascii '{"url":"https://linktotestimage"}'
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/quicktest/url?iterationId={iterationId}" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey"; "Content-Type"="application/json" } `
    -Body '{"url":"https://linktotestimage"}'
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
{
    "id": "369b010b-2a92-4f48-a918-4c1a0af91888",
    "project": "45d1b19b-69b8-4b22-8e7e-d1ca37504686",
    "iteration": "23de09d6-42a1-413e-839e-8db6ee6d3496",
    "created": "2018-08-16T17:39:20.7944508Z",
    "predictions": [
        {
            "probability": 0.8390652,
            "tagId": "ed6f7ab6-5132-47ad-8649-3ec42ee62d43",
            "tagName": "cat"
        }
    ]
}
```

如需詳細資訊，請參閱 [QuickTestImageUrl](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a828d)。

## <a name="export-the-model"></a>匯出模型

匯出模型是一個兩步驟的流程。 首先，您必須指定模型格式，然後要求所匯出模型的 URL。

### <a name="request-a-model-export"></a>要求模型匯出

下列範例示範如何匯出 `coreml` 模型。 使用您專案的識別碼來取代 `{projectId}`。 使用從模型定型傳回的識別碼來取代 `{iterationId}`。

```bash
curl -X POST "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/iterations/{iterationId}/export?platform=coreml" -H "Training-Key: $TRAININGKEY" -H "Content-Type: application/json" --data-ascii ''
```

```powershell
$resp = Invoke-WebRequest -Method 'POST' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/iterations/{iterationId}/export?platform=coreml" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey"; "Content-Type"="application/json" }
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
{
    "platform": "CoreML",
    "status": "Exporting",
    "downloadUri": null,
    "flavor": null
}
```

如需詳細資訊，請參閱 [ExportIteration](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a829b)。

### <a name="download-the-exported-model"></a>下載所匯出的模型

下列範例示範如何擷取所匯出模型的 URL。 使用您專案的識別碼來取代 `{projectId}`。 使用從模型定型傳回的識別碼來取代 `{iterationId}`。

```bash
curl -X GET "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/iterations/{iterationId}/export" -H "Training-Key: $TRAININGKEY" -H "Content-Type: application/json" --data-ascii ''
```

```powershell
$resp = Invoke-WebRequest -Method 'GET' `
    -Uri "https://southcentralus.api.cognitive.microsoft.com/customvision/v2.0/Training/projects/{projectId}/iterations/{iterationId}/export" `
    -UseBasicParsing `
    -Headers @{ "Training-Key"="$trainingKey"; "Content-Type"="application/json" }
$resp.Content
```

對要求的回應會與下列 JSON 文件類似：

```json
[
    {
        "platform": "CoreML",
        "status": "Done",
        "downloadUri": "https://linktoexportedmodel",
        "flavor": null
    }
]
```

如需詳細資訊，請參閱 [GetExports](https://southcentralus.dev.cognitive.microsoft.com/docs/services/d0e77c63c39c4259a298830c15188310/operations/5a59953940d86a0f3c7a829a)。