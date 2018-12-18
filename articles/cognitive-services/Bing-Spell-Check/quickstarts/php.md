---
title: 快速入門：Bing 拼字檢查 API，PHP
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用 Bing 拼字檢查 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-spell-check
ms.topic: quickstart
ms.date: 09/14/2017
ms.author: v-jaswel
ms.openlocfilehash: b35545cbf8e814cd1a175b722fd46367ca7558e4
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48801115"
---
# <a name="quickstart-for-bing-spell-check-api-with-php"></a>搭配使用 Bing 拼字檢查 API 與 PHP 的快速入門 

本文示範如何使用以 PHP 撰寫的 [Bing 拼字檢查 API](https://azure.microsoft.com/services/cognitive-services/spell-check/)。 拼字檢查 API 會傳回未能辨識的字詞清單，附帶建議的替代項目。 一般而言，您會將文字提交給此 API，然後在文字中進行建議的替代，或者向應用程式使用者顯示建議的替代項目，讓他們決定是否要進行替代。 本文顯示如何傳送其中包含文字 "Hollo, wrld!" 的要求 建議的替代項目是 "Hello" 和 "world"。

## <a name="prerequisites"></a>必要條件

您需要 [PHP 5.6.x](http://php.net/downloads.php) 以執行此程式碼。

您必須有具備 **Bing 拼字檢查 API v7** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/#lang)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="get-spell-check-results"></a>取得拼字檢查結果

1. 在您最愛的 IDE 中建立新的 PHP 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 執行程式。

```php
<?php

// NOTE: Be sure to uncomment the following line in your php.ini file.
// ;extension=php_openssl.dll

// These properties are used for optional headers (see below).
// define("CLIENT_ID", "<Client ID from Previous Response Goes Here>");
// define("CLIENT_IP", "999.999.999.999");
// define("CLIENT_LOCATION", "+90.0000000000000;long: 00.0000000000000;re:100.000000000000");

$host = 'https://api.cognitive.microsoft.com';
$path = '/bing/v7.0/spellcheck?';
$params = 'mkt=en-us&mode=proof';

$input = "Hollo, wrld!";

$data = array (
    'text' => urlencode ($input)
);

// NOTE: Replace this example key with a valid subscription key.
$key = 'ENTER KEY HERE';

// The following headers are optional, but it is recommended
// that they are treated as required. These headers will assist the service
// with returning more accurate results.
//'X-Search-Location' => CLIENT_LOCATION
//'X-MSEdge-ClientID' => CLIENT_ID
//'X-MSEdge-ClientIP' => CLIENT_IP

$headers = "Content-type: application/x-www-form-urlencoded\r\n" .
    "Ocp-Apim-Subscription-Key: $key\r\n";

// NOTE: Use the key 'http' even if you are making an HTTPS request. See:
// http://php.net/manual/en/function.stream-context-create.php
$options = array (
    'http' => array (
        'header' => $headers,
        'method' => 'POST',
        'content' => http_build_query ($data)
    )
);
$context  = stream_context_create ($options);
$result = file_get_contents ($host . $path . $params, false, $context);

if ($result === FALSE) {
    /* Handle error */
}

$json = json_encode(json_decode($result), JSON_UNESCAPED_UNICODE | JSON_PRETTY_PRINT);
echo $json;
?>
```

**回應**

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
   "_type": "SpellCheck",
   "flaggedTokens": [
      {
         "offset": 0,
         "token": "Hollo",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "Hello",
               "score": 0.9115257530801
            },
            {
               "suggestion": "Hollow",
               "score": 0.858039839213461
            },
            {
               "suggestion": "Hallo",
               "score": 0.597385084464481
            }
         ]
      },
      {
         "offset": 7,
         "token": "wrld",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "world",
               "score": 0.9115257530801
            }
         ]
      }
   ]
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 拼字檢查教學課程](../tutorials/spellcheck.md)

## <a name="see-also"></a>另請參閱

- [Bing 拼字檢查概觀](../proof-text.md)
- [Bing 拼字檢查 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-spell-check-api-v7-reference)
