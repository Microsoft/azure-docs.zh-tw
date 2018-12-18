---
title: 快速入門：Bing 自動建議 API (PHP)
titlesuffix: Azure Cognitive Services
description: 取得資訊和程式碼範例，以協助您快速開始使用 Bing 自動建議 API。
services: cognitive-services
author: v-jaswel
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-autosuggest
ms.topic: quickstart
ms.date: 09/14/2017
ms.author: v-jaswel
ms.openlocfilehash: 837fdfef0df0c8589a7395c49c3b25753039acd8
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831512"
---
# <a name="quickstart-for-bing-autosuggest-api-with-php"></a>以 PHP 撰寫之 Bing 快速建議 API 的快速入門

本文示範如何使用以 PHP 撰寫之 [Bing 自動建議 API](https://azure.microsoft.com/services/cognitive-services/autosuggest/)。 Bing 自動建議 API 會根據使用者在搜尋方塊中輸入的部份字串，傳回建議的查詢清單。 通常，每次使用者在搜尋方塊中鍵入新字元時，都會呼叫此 API，然後在搜尋方塊的下拉式清單中顯示建議。 本文示範如何傳送要求，以針對 *sail* 傳回建議的查詢字串。

## <a name="prerequisites"></a>必要條件

您需要 [PHP 5.6.x](http://php.net/downloads.php) 以執行此程式碼。

您必須有具備 **Bing 自動建議 API v7** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/#search)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="get-autosuggest-results"></a>取得自動建議的結果

1. 在您愛用的 IDE 中建立新的 PHP 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 執行程式。

```php
<?php

// NOTE: Be sure to uncomment the following line in your php.ini file.
// ;extension=php_openssl.dll

// **********************************************
// *** Update or verify the following values. ***
// **********************************************

// Replace the subscriptionKey string value with your valid subscription key.
$subscriptionKey = 'enter key here';

$host = "https://api.cognitive.microsoft.com";
$path = "/bing/v7.0/Suggestions";

$mkt = "en-US";
$query = "sail";

function get_suggestions ($host, $path, $key, $mkt, $query) {

  $params = '?mkt=' . $mkt . '&q=' . $query;

  $headers = "Content-type: text/json\r\n" .
    "Ocp-Apim-Subscription-Key: $key\r\n";

  // NOTE: Use the key 'http' even if you are making an HTTPS request. See:
  // http://php.net/manual/en/function.stream-context-create.php
  $options = array (
    'http' => array (
      'header' => $headers,
      'method' => 'GET'
    )
  );
  $context  = stream_context_create ($options);
  $result = file_get_contents ($host . $path . $params, false, $context);
  return $result;
}

$result = get_suggestions ($host, $path, $subscriptionKey, $mkt, $query);

echo json_encode (json_decode ($result), JSON_PRETTY_PRINT);
?>
```

### <a name="response"></a>Response

如以下範例所示，成功的回應會以 JSON 格式來傳回： 

```json
{
  "_type": "Suggestions",
  "queryContext": {
    "originalQuery": "sail"
  },
  "suggestionGroups": [
    {
      "name": "Web",
      "searchSuggestions": [
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dgvtP9TS9NwhajSapY2Se6y1eCbP2fq_GiP2n-cxi6OY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailrite%26FORM%3dUSBAPI\u0026p\u003dDevEx,5003.1",
          "displayText": "sailrite",
          "query": "sailrite",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dBTS0G6AakxntIl9rmbDXtk1n6rQpsZZ99aQ7ClE7dTY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsail%2bsand%2bpoint%26FORM%3dUSBAPI\u0026p\u003dDevEx,5004.1",
          "displayText": "sail sand point",
          "query": "sail sand point",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dc0QOA_j6swCZJy9FxqOwke2KslJE7ZRmMooGClAuCpY\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboats%2bfor%2bsale%26FORM%3dUSBAPI\u0026p\u003dDevEx,5005.1",
          "displayText": "sailboats for sale",
          "query": "sailboats for sale",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dmnMdREUH20SepmHQH1zlh9Hy_w7jpOlZFm3KG2R_BoA\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailing%2banarchy%26FORM%3dUSBAPI\u0026p\u003dDevEx,5006.1",
          "displayText": "sailing anarchy",
          "query": "sailing anarchy",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dWLFO-B1GG5qtBGnoU1Bizz02YKkg5fgAQtHwhXn4z8I\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailpoint%26FORM%3dUSBAPI\u0026p\u003dDevEx,5007.1",
          "displayText": "sailpoint",
          "query": "sailpoint",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003dquBMwmKlGwqC5wAU0K7n416plhWcR8zQCi7r-Fw9Y0w\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailflow%26FORM%3dUSBAPI\u0026p\u003dDevEx,5008.1",
          "displayText": "sailflow",
          "query": "sailflow",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003d0udadFl0gCTKCp0QmzQTXS3_y08iO8FpwsoKPHPS6kw\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboatdata%26FORM%3dUSBAPI\u0026p\u003dDevEx,5009.1",
          "displayText": "sailboatdata",
          "query": "sailboatdata",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG\u003d2ACC4FE8B02F4AACB9182A6502B0E556\u0026CID\u003d1D546424A4CB64AF2D386F26A5CD6583\u0026rd\u003d1\u0026h\u003deSSt0MRSbl2V0RFPSuVd-gC7fGOT4717pz55EBUgPec\u0026v\u003d1\u0026r\u003dhttps%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailor%2b2025%26FORM%3dUSBAPI\u0026p\u003dDevEx,5010.1",
          "displayText": "sailor 2025",
          "query": "sailor 2025",
          "searchKind": "WebSearch"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [Bing 自動建議教學課程l](../tutorials/autosuggest.md)

## <a name="see-also"></a>另請參閱

- [什麼是 Bing 自動建議？](../get-suggested-search-terms.md)
- [Bing 自動建議 API v7 參考](https://docs.microsoft.com/rest/api/cognitiveservices/bing-autosuggest-api-v7-reference)