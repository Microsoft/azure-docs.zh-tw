---
title: 如何搭配 Java 使用異常搜尋工具 API - Microsoft 認知服務 | Microsoft Docs
description: 取得資訊和程式碼範例，以協助您快速開始使用 Java 和認知服務中的異常偵測。
services: cognitive-services
author: chliang
manager: bix
ms.service: cognitive-services
ms.technology: anomaly-detection
ms.topic: article
ms.date: 05/01/2018
ms.author: kefre
ms.openlocfilehash: 8152c23e6c5332d243d851be56bab1e4085dbe5a
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369151"
---
# <a name="use-the-anomaly-finder-api-with-java"></a>搭配 Java 使用異常搜尋工具 API

本文提供資訊和程式碼範例，協助您快速開始搭配 Java 使用異常偵測 API，以便完成對於時間序列資料取得異常偵測結果的工作。

## <a name="prerequisites"></a>先決條件

[!INCLUDE [GetSubscriptionKey](../includes/get-subscription-key.md)]

## <a name="getting-anomaly-points-with-the-anomaly-detection-api-using-java"></a>搭配 Java 使用異常偵測 API 取得異常點

[!INCLUDE [DataContract](../includes/datacontract.md)]

### <a name="example-of-time-series-data"></a>時間序列資料範例

時間序列資料點的範例如下所示。

[!INCLUDE [Request](../includes/request.md)]

### <a name="analyze-data-and-get-anomaly-points-java-example"></a>分析資料，並取得異常點的 Java 範例

若要執行範例，請執行下列步驟：
1. 建立新的命令列應用程式。
2. 以下列程式碼取代主要類別 (保留任何 `package` 陳述式)。
3. 以有效的訂用帳戶金鑰取代 `[YOUR_SUBSCRIPTION_KEY]` 值。
4. 使用範例或您自己的資料點取代 `[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]`。
5. 將 Maven 存放庫中的這些通用程式庫下載到您專案中的 `lib` 目錄：
   * `org.apache.httpcomponents:httpclient:4.5.2`
6. 執行 'Main'。

```java

import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.util.EntityUtils;

public class Main {
    // **********************************************
    // *** Update or verify the following values. ***
    // **********************************************

    // Replace the subscriptionKey string value with your valid subscription key.
    public static final String subscriptionKey = "[YOUR_SUBSCRIPTION_KEY]";

    public static final String uriBase = "https://api.labs.cognitive.microsoft.com/anomalyfinder/v1.0/anomalydetection";

    public static void main(String[] args) {
        final String content = "[REPLACE_WITH_THE_EXAMPLE_OR_YOUR_OWN_DATA_POINTS]";

        CloseableHttpClient client = HttpClients.createDefault();
        HttpPost request = new HttpPost(uriBase);

        // Request headers.
        request.setHeader("Content-Type", "application/json");
        request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKey);

        try {
            StringEntity params = new StringEntity(content);
            request.setEntity(params);

            CloseableHttpResponse response = client.execute(request);
            try {
                HttpEntity respEntity = response.getEntity();
                if (respEntity != null) {
                    System.out.println("----------");
                    System.out.println(response.getStatusLine());
                    System.out.println("Response content is :\n");
                    System.out.println(EntityUtils.toString(respEntity, "utf-8"));
                    System.out.println("----------");
                }
            } catch (Exception respEx) {
                respEx.printStackTrace();
            } finally {
                response.close();
            }

        } catch (Exception ex) {
            System.err.println("Exception on Anomaly Detection: " + ex.getMessage());
            ex.printStackTrace();
        } finally {
            try {
                client.close();
            } catch (Exception e) {
                System.err.println("Exception on closing HttpClient: " + e.getMessage());
                e.printStackTrace();
            }
        }
    }
}

```

### <a name="example-response"></a>範例回應

在 JSON 中，會回傳成功的回應。 範例回應如下所示。
[!INCLUDE [Response](../includes/response.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Java 應用程式](../tutorials/java-tutorial.md)
