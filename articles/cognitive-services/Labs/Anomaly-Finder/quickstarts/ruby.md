---
title: 如何搭配 Ruby 使用異常搜尋工具 API - Microsoft 認知服務 | Microsoft Docs
description: 取得資訊和程式碼範例，以協助您快速開始使用 Ruby 和認知服務中的異常搜尋工具 API。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 05/01/2018
ms.author: chliang
ms.openlocfilehash: ef85b82adf3b177678cc8175a7fb2d0eb44323fa
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48246854"
---
# <a name="use-the-anomaly-finder-api-with-ruby"></a>搭配 Ruby 使用異常搜尋工具 API

[!INCLUDE [PrivatePreviewNote](../../../../../includes/cognitive-services-anomaly-finder-private-preview-note.md)]

本文提供資訊和程式碼範例，協助您快速開始搭配 Ruby 使用異常搜尋工具 API，以便完成對於時間序列資料取得異常偵測結果的工作。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="getting-anomaly-points-with-anomaly-finder-api-using-ruby"></a>搭配 Ruby 使用異常搜尋工具 API 取得異常點 
[!INCLUDE [DataContract](../includes/datacontract.md)]

### <a name="example-of-time-series-data"></a>時間序列資料範例
時間序列資料點的範例如下所示。

[!INCLUDE [Request](../includes/request.md)]

### <a name="analyze-data-and-get-anomaly-points-ruby-example"></a>分析資料並取得異常點的 Ruby 範例

使用此範例的步驟如下所示。

1. 執行 'gem install rest-client' 來安裝 [rest-client](https://github.com/rest-client/rest-client)。
2. 將下列程式碼儲存為 .rb 檔案。
3. 以您的有效訂用帳戶金鑰取代 `[YOUR_SUBSCRIPTION_KEY]` 值。
4. 以範例或您自己的資料點取代 `[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]`。
5. 執行並檢查回應。

```ruby
# https://github.com/rest-client/rest-client
require 'rest_client'

# **********************************************
# *** Update or verify the following values. ***
# **********************************************

# Replace the subscriptionKey string value with your valid subscription key.
subscription_key = '[YOUR_SUBSCRIPTION_KEY]';

endpoint = "https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection";

# Replace the request data with your real data.
requestData = '[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]';

header = {
    :content_type => 'application/json',
    :'Ocp-Apim-Subscription-Key' => subscription_key
}

response = RestClient::Request.execute(
    :url => endpoint,
    :method => :post,
    :verify_ssl => true,
    :payload => requestData,
    :header => header)

# You will see the response with anomaly results
puts response.body
```

### <a name="example-response"></a>範例回應

成功的回應會以 JSON 的形式傳回。 範例回應如下所示。
[!INCLUDE [Response](../includes/response.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [REST API 參考資料](https://dev.labs.cognitive.microsoft.com/docs/services/anomaly-detection/operations/post-anomalydetection)
