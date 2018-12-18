---
title: 快速入門：Bing 拼字檢查 API，Ruby
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
ms.openlocfilehash: 3fe8729a9e2524cc2ccda168a857d58664a98b10
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48801075"
---
# <a name="quickstart-for-bing-spell-check-api-with-ruby"></a>搭配使用 Bing 拼字檢查 API 與 Ruby 的快速入門 

本文示範如何使用以 Ruby 撰寫的 [Bing 拼字檢查 API](https://azure.microsoft.com/services/cognitive-services/spell-check/)。 拼字檢查 API 會傳回未能辨識的字詞清單，附帶建議的替代項目。 一般而言，您會將文字提交給此 API，然後在文字中進行建議的替代，或者向應用程式使用者顯示建議的替代項目，讓他們決定是否要進行替代。 本文顯示如何傳送其中包含文字 "Hollo, wrld!" 的要求 建議的替代項目是 "Hello" 和 "world"。

## <a name="prerequisites"></a>必要條件

您需要 [Ruby 2.4](https://www.ruby-lang.org/en/downloads/) 或更新版本以執行此程式碼。

您必須有具備 **Bing 拼字檢查 API v7** 的[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)。 [免費試用版](https://azure.microsoft.com/try/cognitive-services/#lang)即足以供本快速入門使用。 您必須要有啟用免費試用版時所提供的存取金鑰，或者您可以從 Azure 儀表板使用付費訂用帳戶金鑰。

## <a name="get-spell-check-results"></a>取得拼字檢查結果

1. 在您最愛的 IDE 中建立新的 Ruby 專案。
2. 新增下方提供的程式碼。
3. 以訂用帳戶有效的存取金鑰來取代 `subscriptionKey` 值。
4. 執行程式。

```ruby
require 'net/http'
require 'uri'
require 'json'

uri = 'https://api.cognitive.microsoft.com'
path = '/bing/v7.0/spellcheck?'
params = 'mkt=en-us&mode=proof'

uri = URI(uri + path + params)
uri.query = URI.encode_www_form({
    # Request parameters
    'text' => 'Hollo, wrld!'
})

# NOTE: Replace this example key with a valid subscription key.
key = 'ENTER KEY HERE'

# The headers in the following example 
# are optional but should be considered as required:
#
# X-MSEdge-ClientIP: 999.999.999.999  
# X-Search-Location: lat: +90.0000000000000;long: 00.0000000000000;re:100.000000000000
# X-MSEdge-ClientID: <Client ID from Previous Response Goes Here>
#
# See below for more information.

request = Net::HTTP::Post.new(uri)
request['Content-Type'] = "application/x-www-form-urlencoded"

request['Ocp-Apim-Subscription-Key'] = key

response = Net::HTTP.start(uri.host, uri.port, :use_ssl => uri.scheme == 'https') do |http|
    http.request(request)
end

result = JSON.pretty_generate(JSON.parse(response.body))
puts result
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
