---
title: 快速入門：產生縮圖 - REST、cURL - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 cURL，從影像產生縮圖。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: 51c6a8e5693602cdc839de80f268891c247c63a9
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344059"
---
# <a name="quickstart-generate-a-thumbnail-using-the-rest-api-and-curl-in-computer-vision"></a>快速入門：使用電腦視覺中的 REST API 和 cURL 產生縮圖

在本快速入門中，您會使用電腦視覺的 REST API 從影像產生縮圖。 您可以使用[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法產生影像的縮圖。 您可指定高度和寬度，可以與輸入影像的外觀比例不同。 「電腦視覺」會使用智慧型裁剪，以智慧方式識別感興趣的區域，並且根據該區域產生裁剪的座標。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

若要使用「電腦視覺」，您需要訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="get-thumbnail-request"></a>Get Thumbnail 要求

您可以使用 [Get Thumbnail 方法](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb) (英文)，產生影像的縮圖。 您可指定高度和寬度，可以與輸入影像的外觀比例不同。 「電腦視覺」會使用智慧型裁剪，以智慧方式識別感興趣的區域，並且根據該區域產生裁剪的座標。

若要執行範例，請執行下列步驟：

1. 請將下列程式碼複製到編輯器中。
1. 將 `<Subscription Key>` 取代為您的有效訂用帳戶金鑰。
1. 以要用來儲存縮圖的路徑和檔案名稱取代 `<File>`。
1. 必要時，請將要求 URL (`https://westcentralus.api.cognitive.microsoft.com/vision/v2.0`) 變更為使用您取得訂用帳戶金鑰的位置。
1. (選擇性) 變更要分析的影像 (`{\"url\":\"...`)。
1. 在已安裝 cURL 的電腦上開啟命令視窗。
1. 將程式碼貼入視窗中，然後執行命令。

>[!NOTE]
>您必須在 REST 呼叫中使用與用來取得訂用帳戶金鑰相同的位置。 例如，如果您已從 westus 取得訂用帳戶金鑰，請將下列 URL 中的 "westcentralus" 取代為 "westus"。

## <a name="prerequisites"></a>必要條件

- 您必須擁有 [cURL](https://curl.haxx.se/windows)。
- 您必須具有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample-command"></a>建立和執行範例命令

若要建立並執行範例，請執行下列步驟：

1. 將下列命令複製到文字編輯器。
1. 視需要在命令中進行下列變更：
    1. 將 `<subscriptionKey>` 的值取代為您的訂用帳戶金鑰。
    1. 將 `<thumbnailFile>` 的值取代為儲存縮圖之檔案的路徑和名稱。
    1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將要求 URL (`https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail`) 取代為 [Get Thumbnail](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb) 方法的端點 URL。
    1. (選擇性) 將要求本文 (`https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Shorkie_Poo_Puppy.jpg/1280px-Shorkie_Poo_Puppy.jpg\`) 中的影像 URL 變更為要產生縮圖之不同影像的 URL。
1. 開啟命令提示字元視窗。
1. 將文字編輯器中的命令貼到命令提示字元視窗中，然後執行該命令。

```console
curl -H "Ocp-Apim-Subscription-Key: <subscriptionKey>" -o <thumbnailFile> -H "Content-Type: application/json" "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail?width=100&height=100&smartCropping=true" -d "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Shorkie_Poo_Puppy.jpg/1280px-Shorkie_Poo_Puppy.jpg\"}"
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會將縮圖影像寫入至 `<thumbnailFile>` 中指定的檔案。 如果要求失敗，回應會包含錯誤碼和訊息以協助判斷問題所在。

## <a name="clean-up-resources"></a>清除資源

不再需要時，請關閉命令提示字元視窗和文字編輯器。

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人與地標、建立縮圖，以及擷取印刷與手寫文字的電腦視覺 API。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
