---
title: Microsoft Translator Text API V3.0 參考 | Microsoft Docs
description: Microsoft Translator Text API V3.0 參考文件。
services: cognitive-services
author: Jann-Skotdal
manager: chriswendt1
ms.service: cognitive-services
ms.technology: microsoft translator
ms.topic: article
ms.date: 03/29/2018
ms.author: v-jansko
ms.openlocfilehash: cfaa9584e833b137b417d9074fbfcf606eb21388
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35368722"
---
#<a name="translator-text-api-v30"></a>Microsoft Translator Text API v3.0

## <a name="whats-new"></a>新功能

Microsoft Translator Text API 第 3 版提供最新 JSON 型 Web API。 其藉由將現有功能合併成較少的作業來提升可用性和效能，並提供新功能。

 * 轉換功能，可將某種語言的文字從某個字集轉換成另一個字集。
 * 透過一個要求即可翻譯成多種語言。
 * 透過一個要求即可進行語言偵測、翻譯和轉換。
 * 可查閱字詞替代翻譯、找到反向翻譯及顯示字詞實際應用範例的字典。
 * 包含更多資訊的語言偵測結果。

## <a name="base-urls"></a>基底 URL

Text API v3.0 可在下列雲端中使用：

| 說明 | 區域 | 基底 URL                                        |
|-------------|--------|-------------------------------------------------|
| Azure       | 全域 | api.cognitive.microsofttranslator.com           |


## <a name="authentication"></a>驗證

訂閱 Microsoft 認知服務中的 Translator Text API，並使用您的訂用帳戶金鑰 (可在 Azure 入口網站中取得) 來進行驗證。 

最簡單方式是使用要求標頭 `Ocp-Apim-Subscription-Key`，將您的 Azure 祕密金鑰傳遞至翻譯工具服務。

替代方法是使用祕密金鑰從權杖服務中取得授權權杖。 然後，使用 `Authorization` 要求標頭將授權權杖傳遞至翻譯工具服務。 若要取得授權權杖，請對下列 URL 提出 `POST` 要求：

| 環境     | 驗證服務 URL                                |
|-----------------|-----------------------------------------------------------|
| Azure           | `https://api.cognitive.microsoft.com/sts/v1.0/issueToken` |

以下要求範例示範如何取得具有祕密金鑰的權杖：

```
// Pass secret key using header
curl --header 'Ocp-Apim-Subscription-Key: <your-key>' --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken'
// Pass secret key using query string parameter
curl --data "" 'https://api.cognitive.microsoft.com/sts/v1.0/issueToken?Subscription-Key=<your-key>'
```

成功的要求會以純文字形式在回應本文中傳回已編碼的存取權杖。 有效的權杖會傳遞至翻譯工具服務，以作為授權的持有人權杖。

```
Authorization: Bearer <Base64-access_token>
```

驗證權杖的有效時間為 10 分鐘。 對翻譯工具 API 執行多次呼叫時，應重複使用權杖。 不過，如果您的程式會在超過一段時間後才對翻譯工具 API 提出要求，則您的程式必須以固定間隔 (例如每 8 分鐘) 要求新存取權杖。

總而言之，對翻譯工具 API 提出的用戶端要求，將包含一個從下表取用的授權標頭：

<table width="100%">
  <th width="30%">headers</th>
  <th>說明</th>
  <tr>
    <td>Ocp-Apim-Subscription-Key</td>
    <td>如果您要傳遞祕密金鑰，請使用認知服務訂用帳戶。<br/>此值是您 Translator Text API 訂用帳戶的 Azure 祕密金鑰。</td>
  </tr>
  <tr>
    <td>Authorization</td>
    <td>如果您要傳遞驗證權杖，請使用認知服務訂用帳戶。<br/>此值是持有人權杖：`Bearer <token>`。</td>
  </tr>
</table> 

## <a name="errors"></a>Errors

標準錯誤回應是名稱/值組為 `error` 的 JSON 物件。 此值也可以是具有下列屬性的 JSON 物件：

  * `code`：伺服器定義的錯誤碼。

  * `message`：以人類可閱讀形式表示錯誤的字串。

例如，持有免費試用訂用帳戶的客戶會在免費配額用完時，收到下列錯誤：

```
{
  "error": {
    "code":403000,
    "message":"The subscription has exceeded its free quota."
    }
}
```
