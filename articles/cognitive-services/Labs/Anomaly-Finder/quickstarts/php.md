---
title: 如何搭配 PHP 使用異常搜尋工具 API - Microsoft 認知服務 | Microsoft Docs
description: 取得資訊和程式碼範例，以協助您搭配 PHP 快速開始使用認知服務中的異常搜尋工具。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: f5f055be584aa9c6afb66ff651fd3c01851afd76
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48246390"
---
# <a name="use-the-anomaly-finder-api-with-php"></a>搭配 PHP 使用異常搜尋工具 API

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

本文提供資訊和程式碼範例，協助您快速開始搭配 PHP 使用異常搜尋工具 API，以便完成對於時間序列資料取得異常結果的工作。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="getting-anomaly-points-with-anomaly-finder-api-using-php"></a>搭配 PHP 使用異常搜尋工具 API 取得異常點
[!INCLUDE [DataContract](../includes/datacontract.md)]

### <a name="example-of-time-series-data"></a>時間序列資料範例
時間序列資料的範例如下所示。
[!INCLUDE [Request](../includes/request.md)]

### <a name="analyze-data-and-get-anomaly-points-php-example"></a>分析資料，並取得異常點 PHP 範例
1. 以有效的訂用帳戶金鑰取代 `[YOUR_SUBSCRIPTION_KEY]` 值。
2. 以範例或您自己的資料點取代 `[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]`。
3. 執行並檢查回應。

```PHP
<?php
# This sample uses the Apache HTTP client from HTTP components (http://hc.apache.org/httpcomponents-client-ga/)
require_once 'HTTP/Request2.php';

$request = new HTTP_Request2('https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection');
$url = $request->getUrl();

$requestData = '[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]';

$headers = array(
    # Request headers
    'Content-Type' => 'application/json',
    # NOTE: Replace the "Ocp-Apim-Subscription-Key" value with a valid subscription key.
    'Ocp-Apim-Subscription-Key' => '[YOUR_SUBSCRIPTION_KEY]',
);

$request->setHeader($headers);

$request->setMethod(HTTP_Request2::METHOD_POST);

# Request body
$request->setBody($requestData);

try
{
    $response = $request->send();
    echo $response->getBody();
}
catch (HttpException $ex)
{
    echo $ex;
}
?>
```

### <a name="example-response"></a>範例回應

成功的回應會以 JSON 的形式傳回。 範例回應如下所示。
[!INCLUDE [Response](../includes/response.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
