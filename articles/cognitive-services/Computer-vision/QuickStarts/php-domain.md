---
title: 快速入門：使用領域模型 - REST、PHP - 電腦視覺
titleSuffix: Azure Cognitive Services
description: 在本快速入門中，您將搭配使用電腦視覺 API 與 PHP，使用領域模型來識別影像中的地標。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: quickstart
ms.date: 08/28/2018
ms.author: pafarley
ms.openlocfilehash: 176e11615fe2f2c1b58b7608dfd5cbd349f9e547
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340064"
---
# <a name="quickstart-use-a-domain-model-using-the-rest-api-and-php-in-computer-vision"></a>快速入門：在電腦視覺中利用 REST API 和 PHP 來使用領域模型

在本快速入門中，您將利用電腦視覺的 REST API，使用領域模型來識別遠端儲存之影像中的地標或名人 (選擇性)。 使用[辨識特定領域內容](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) \(英文\) 方法，您可以套用特定領域的模型來辨識影像中的內容。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/ai/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=cognitive-services) 。

## <a name="prerequisites"></a>必要條件

- 您必須安裝 [PHP](https://secure.php.net/downloads.php)。
- 您必須安裝 [Pear](https://pear.php.net)。
- 您必須具有電腦視覺的訂用帳戶金鑰。 若要取得訂用帳戶金鑰，請參閱[取得訂用帳戶金鑰](../Vision-API-How-to-Topics/HowToSubscribe.md)。

## <a name="create-and-run-the-sample"></a>建立並執行範例

若要建立並執行範例，請執行下列步驟：

1. 安裝 PHP5 [`HTTP_Request2`](http://pear.php.net/package/HTTP_Request2) 套件。
   1. 以系統管理員身分開啟 [命令提示字元] 視窗。
   1. 執行以下命令：

      ```console
      pear install HTTP_Request2
      ```

   1. 成功安裝套件之後，關閉命令提示字元視窗。

1. 將下列程式碼複製到文字編輯器。
1. 視需要在程式碼中進行下列變更：
    1. 將 `subscriptionKey` 的值取代為您的訂用帳戶金鑰。
    1. 如有需要，請從您取得訂用帳戶金鑰的 Azure 區域，將 `uriBase` 的值取代為[辨識特定領域內容](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e200) \(英文\) 方法的端點 URL。
    1. (選擇性) 將 `imageUrl` 的值取代為您要分析之不同影像的 URL。
    1. (選擇性) 如果您想要使用 `celebrities` 領域模型，而不是 `landmarks` 領域模型，請將 `domain` 要求參數的值取代為 `celebrites`。
1. 將程式碼儲存為副檔名為 `.php` 的檔案。 例如： `use-domain-model.php`。
1. 開啟具有 PHP 支援的瀏覽器視窗。
1. 將檔案拖放到瀏覽器視窗中。

```php
<html>
<head>
    <title>Analyze Domain Model Sample</title>
</head>
<body>
<?php
// Replace <Subscription Key> with a valid subscription key.
$ocpApimSubscriptionKey = '<Subscription Key>';

// You must use the same location in your REST call as you used to obtain
// your subscription keys. For example, if you obtained your subscription keys
// from westus, replace "westcentralus" in the URL below with "westus".
$uriBase = 'https://westcentralus.api.cognitive.microsoft.com/vision/v2.0/';

// Change 'landmarks' to 'celebrities' to use the Celebrities model.
$domain = 'landmarks';

$imageUrl =
    'https://upload.wikimedia.org/wikipedia/commons/2/23/Space_Needle_2011-07-04.jpg';

require_once 'HTTP/Request2.php';

$request = new Http_Request2($uriBase . 'models/' . $domain . '/analyze');
$url = $request->getUrl();

$headers = array(
    // Request headers
    'Content-Type' => 'application/json',
    'Ocp-Apim-Subscription-Key' => $ocpApimSubscriptionKey
);
$request->setHeader($headers);

$parameters = array(
    // Request parameters
    'model' => $domain
);
$url->setQueryVariables($parameters);

$request->setMethod(HTTP_Request2::METHOD_POST);

// Request body parameters
$body = json_encode(array('url' => $imageUrl));

// Request body
$request->setBody($body);

try
{
    $response = $request->send();
    echo "<pre>" .
        json_encode(json_decode($response->getBody()), JSON_PRETTY_PRINT) . "</pre>";
}
catch (HttpException $ex)
{
    echo "<pre>" . $ex . "</pre>";
}
?>
</body>
</html>
```

## <a name="examine-the-response"></a>檢查回應

成功的回應會以 JSON 的形式傳回。 範例網站會在瀏覽器視窗中剖析並顯示成功的回應，如下列範例所示：

```json
{
    "result": {
        "landmarks": [
            {
                "name": "Space Needle",
                "confidence": 0.9998177886009216
            }
        ]
    },
    "requestId": "4d26587b-b2b9-408d-a70c-1f8121d84b0d",
    "metadata": {
        "height": 4132,
        "width": 2096,
        "format": "Jpeg"
    }
}
```

## <a name="clean-up-resources"></a>清除資源

不再需要檔案時請將它刪除，然後再解除安裝 PHP5 `HTTP_Request2` 套件。 若要解除安裝套件，請執行下列步驟：

1. 以系統管理員身分開啟 [命令提示字元] 視窗。
2. 執行以下命令：

   ```console
   pear uninstall HTTP_Request2
   ```

3. 將套件成功解除安裝之後，關閉命令提示字元視窗。

## <a name="next-steps"></a>後續步驟

探索用來分析影像、偵測名人與地標、建立縮圖，以及擷取印刷與手寫文字的電腦視覺 API。 若要快速地試驗電腦視覺 API，請嘗試 [Open API 測試主控台](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console)。

> [!div class="nextstepaction"]
> [探索電腦視覺 API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
