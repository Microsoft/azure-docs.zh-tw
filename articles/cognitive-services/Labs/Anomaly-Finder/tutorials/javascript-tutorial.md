---
title: 教學課程：使用 Javascript 進行異常偵測
titlesuffix: Azure Cognitive Services
description: 探索會使用異常偵測 API 的 Javascript Web 應用程式。 將原始資料點傳送到 API，並取得預期的值和異常點。
services: cognitive-services
author: wenya
manager: bix
ms.service: cognitive-services
ms.component: anomaly-detection
ms.topic: tutorial
ms.date: 05/01/2018
ms.author: wenya
ms.openlocfilehash: cd65a5275fac651968197c9000b2d4cd3962693b
ms.sourcegitcommit: 55952b90dc3935a8ea8baeaae9692dbb9bedb47f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2018
ms.locfileid: "48883698"
---
# <a name="tutorial-anomaly-detection-with-javascript-application"></a>教學課程：使用 Javascript 應用程式進行異常偵測

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

探索使用異常偵測 REST API 來偵測輸入異常的 Web 應用程式。 此範例會使用您的訂用帳戶金鑰來將時間序列資料提交到異常偵測 API，然後從該 API 針對每個資料點取得所有異常點和預期的值。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

本教學課程是使用簡單的文字編輯器開發的。

### <a name="subscribe-to-anomaly-detection-and-get-a-subscription-key"></a>訂閱異常偵測並取得訂用帳戶金鑰 

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="get-and-use-the-example"></a>取得和使用範例

本教學課程提供兩個時間序列資料異常偵測案例。 現在就開始吧。

<a name="Step1"></a> 
### <a name="download-the-tutorial-project"></a>下載教學課程專案

複製[認知服務 JavaScript 異常偵測教學課程](https://github.com/MicrosoftAnomalyDetection/javascript-sample)，或下載 .zip 檔案，然後將它解壓縮至空白目錄。

<a name="Step2"></a>
### <a name="run-the-example"></a>執行範例

您可以在兩種案例下嘗試執行範例。
1. 將您的**訂用帳戶金鑰**放入 anomalydetection.html 上位於偵測函式的 [訂用帳戶金鑰] 欄位中。
2. 放置異常偵測 API 端點，然後確認您在 [訂用帳戶區域] 中使用的區域正確無誤。
3. 在網頁瀏覽器中開啟 **anomalydetection.html** 檔案。

**案例 1：偵測每週時間序列資料**
1. 在 [期間] 欄位中，輸入期間 **7**。 
2. 在 [點] 欄位中將範例資料取代為您的每週時間序列資料點 (Json)，或直接使用範例資料。
3. 按一下 [異常偵測] 按鈕，並確認右側 [回應] 文字方塊中的結果。

**案例 2：偵測不具期間的時間序列資料**
1. 將 [期間] 欄位中的期間保留為空白，假設您不知道時間序列的期間。
2. 使用相同的時間序列資料作為案例 1。
3. 按一下 [異常偵測] 按鈕，並確認右側 [回應] 文字方塊中的 [期間] 欄位。

<a name="Step3"></a>
### <a name="read-the-result"></a>讀取結果

[!INCLUDE [diagrams](../includes/diagrams.md)]

<a name="Review"></a>
### <a name="review-and-learn"></a>檢閱和學習

此時您會取得執行中的應用程式。 我們將觀察此範例應用程式如何與認知服務技術整合。 此步驟將可讓您使用 Microsoft 異常偵測，更輕鬆地繼續建置此應用程式或開發您自己的應用程式。
此範例應用程式會使用異常偵測 Restful API 端點。
讓我們看一下來自 anomalydetection.html 的程式碼片段，以觀察如何在範例應用程式中使用 Restful API。
```JavaScript
function anomalyDetection(url, subscriptionKey, points, period) {
    var obj = new Object();
    obj.Points = JSON.parse(points); // this points are read from text box.
    obj.Period = parseInt(period);//period=7 weekly period
    var tsData = JSON.stringify(obj);
    // Perform the REST API call.
    $.ajax({
        url: url, //Anomaly Detection API endpoint
        // Request headers.
        beforeSend: function (xhrObj) {
            xhrObj.setRequestHeader("Content-Type", "application/json");
            xhrObj.setRequestHeader("Ocp-Apim-Subscription-Key", subscriptionKey); // Replace your subscription key
        },
        type: "POST",
        // Request body.
        data: tsData, // json format
        })
        .done(function (data) {
            // Show formatted JSON on webpage.
            $("#responseTextArea").val(JSON.stringify(data, null, 2));
        })
        .fail(function (jqXHR, textStatus, errorThrown) {
            // Display error message.
            var errorString = (errorThrown === "") ? "Error. " : errorThrown + " (" + jqXHR.status + "): ";
            errorString += (jqXHR.responseText === "") ? "" : jQuery.parseJSON(jqXHR.responseText).message;
            $("#responseTextArea").val(errorString);           
        });
}

```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
