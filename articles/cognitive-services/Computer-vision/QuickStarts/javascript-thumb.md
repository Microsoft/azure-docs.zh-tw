---
title: 快速入門：產生縮圖 - REST、JavaScript - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在此快速入門中，您可以使用電腦視覺 API 搭配 JavaScript，從影像產生縮圖。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: e221e2f65182092123ba3a61cf8d8844d9b6ee96
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343073"
---
# <a name="quickstart-generate-a-thumbnail-using-the-rest-api-and-javascript-in-computer-vision"></a>快速入門：使用電腦視覺中的 REST API 與 JavaScript 產生縮圖

在此快速入門中，您會使用電腦視覺的 REST API 從影像產生縮圖。 您可以使用[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法產生影像的縮圖。 您可指定高度和寬度，可以與輸入影像的外觀比例不同。 「電腦視覺」會使用智慧型裁剪，以智慧方式識別感興趣的區域，並且根據該區域產生裁剪的座標。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

您必須具有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample"></a>建立並執行範例

若要建立並執行範例，請執行下列步驟：

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscriptionKey` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將 `uriBase` 的值取代為[取得縮圖](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb)方法的端點 URL。
    1. (選擇性) 將 `inputImage` 控制項的 `value` 屬性值取代為您要分析之不同影像的 URL。
1. 將程式碼儲存為副檔名為 `.html` 的檔案。 例如： `get-thumbnail.html`。
1. 開啟瀏覽器視窗。
1. 在瀏覽器中，將檔案拖放到瀏覽器視窗中。
1. 當瀏覽器中顯示該網頁時，選擇 [產生縮圖] 按鈕。

```html
<!DOCTYPE html>
<html>
<head>
    <title>Thumbnail Sample</title>
</head>
<body>

<script type="text/javascript">
    function processImage() {
        // **********************************************
        // *** Update or verify the following values. ***
        // **********************************************

        // Replace <Subscription Key> with your valid subscription key.
        var subscriptionKey = "<Subscription Key>";

        // You must use the same Azure region in your REST API method as you used to
        // get your subscription keys. For example, if you got your subscription keys
        // from the West US region, replace "westcentralus" in the URL
        // below with "westus".
        //
        // Free trial subscription keys are generated in the West Central US region.
        // If you use a free trial subscription key, you shouldn't need to change
        // this region.
        var uriBase =
            "https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/generateThumbnail";

        // Request parameters.
        var params = "?width=100&height=150&smartCropping=true";

        // Display the source image.
        var sourceImageUrl = document.getElementById("inputImage").value;
        document.querySelector("#sourceImage").src = sourceImageUrl;

        // Prepare the REST API call:

        // Create the HTTP Request object.
        var xhr = new XMLHttpRequest();

        // Identify the request as a POST, with the URL and parameters.
        xhr.open("POST", uriBase + params);

        // Add the request headers.
        xhr.setRequestHeader("Content-Type","application/json");
        xhr.setRequestHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

        // Set the response type to "blob" for the thumbnail image data.
        xhr.responseType = "blob";

        // Process the result of the REST API call.
        xhr.onreadystatechange = function(e) {
            if(xhr.readyState === XMLHttpRequest.DONE) {

                // Thumbnail successfully created.
                if (xhr.status === 200) {
                    // Show response headers.
                    var s = JSON.stringify(xhr.getAllResponseHeaders(), null, 2);
                    document.getElementById("responseTextArea").value =
                        JSON.stringify(xhr.getAllResponseHeaders(), null, 2);

                    // Show thumbnail image.
                    var urlCreator = window.URL || window.webkitURL;
                    var imageUrl = urlCreator.createObjectURL(this.response);
                    document.querySelector("#thumbnailImage").src = imageUrl;
                } else {
                    // Display the error message. The error message is the response
                    // body as a JSON string. The code in this code block extracts
                    // the JSON string from the blob response.
                    var reader = new FileReader();

                    // This event fires after the blob has been read.
                    reader.addEventListener('loadend', (e) => {
                        document.getElementById("responseTextArea").value =
                            JSON.stringify(JSON.parse(e.srcElement.result), null, 2);
                    });

                    // Start reading the blob as text.
                    reader.readAsText(xhr.response);
                }
            }
        }

        // Make the REST API call.
        xhr.send('{"url": ' + '"' + sourceImageUrl + '"}');
    };
</script>

<h1>Generate thumbnail image:</h1>
Enter the URL to an image to use in creating a thumbnail image,
then click the <strong>Generate thumbnail</strong> button.
<br><br>
Image for thumbnail:
<input type="text" name="inputImage" id="inputImage"
    value="https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Shorkie_Poo_Puppy.jpg/1280px-Shorkie_Poo_Puppy.jpg" />
<button onclick="processImage()">Generate thumbnail</button>
<br><br>
<div id="wrapper" style="width:1160px; display:table;">
    <div id="jsonOutput" style="width:600px; display:table-cell;">
        Response:
        <br><br>
        <textarea id="responseTextArea" class="UIInput"
                  style="width:580px; height:400px;"></textarea>
    </div>
    <div id="imageDiv" style="width:420px; display:table-cell;">
        Source image:
        <br><br>
        <img id="sourceImage" width="400" />
    </div>
    <div id="thumbnailDiv" style="width:140px; display:table-cell;">
        Thumbnail:
        <br><br>
        <img id="thumbnailImage" />
    </div>
</div>
</body>
</html>
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以二進位資料的形式傳回，代表縮圖的影像資料。 如果要求成功，就會從回應中的二進位資料產生縮圖並顯示在瀏覽器視窗中。 如果要求失敗，則回應會顯示在主控台視窗中。 如果要求失敗，回應會包含錯誤碼和訊息以協助判斷問題所在。

## <a name="clean-up-resources"></a>清除資源

不再需要檔案時，請將它刪除。

## <a name="next-steps"></a>後續步驟

探索使用「電腦視覺」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記和描述視覺特徵 (包括臉部) 的 JavaScript 應用程式。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [電腦視覺 API JavaScript 教學課程](../Tutorials/javascript-tutorial.md)
