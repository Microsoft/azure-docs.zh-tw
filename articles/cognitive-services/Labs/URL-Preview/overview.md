---
title: 什麼是專案 URL 預覽？
titlesuffix: Azure Cognitive Services
description: 專案 URL 預覽簡介。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: project-url-preview
ms.topic: overview
ms.date: 03/16/2018
ms.author: rosh
ms.openlocfilehash: 5d0b8260bf1c58af915c1be18c32cec678f4f09c
ms.sourcegitcommit: 0bb8db9fe3369ee90f4a5973a69c26bff43eae00
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48869426"
---
# <a name="what-is-project-url-preview"></a>什麼是專案 URL 預覽？
預覽 URL 端點使用 URL 查詢參數，並傳回 JSON 回應，其中包含目標資源的名稱、簡短描述，以及在預覽中顯示影像的連結。 回應也包含 [isFamilyFriendly](url-preview-reference.md#query-parameters) 旗標，指出 URL 是否包含成人、盜版或其他不合法的內容。 

若要取得 URL 預覽結果，請提交 GET 要求，並包含 *Ocp-Apim-Subscription-Key* 標題和有效的安全性權杖：  
```
https://api.labs.cognitive.microsoft.com/urlpreview/v7.0/search?q=https://swiftkey.com

```
回應： 
````
HTTP Headers:
BingAPIs-TraceId: 3CC74C94769440C0851D9DF0869FCE7F
BingAPIs-SessionId: 52219085A6364692958C9C83983A0DBA
X-MSEdge-ClientID: 13D44DC2DE946B4B0F25460CDF036AD6
BingAPIs-Market: en-US
X-MSEdge-Ref: Ref A: 3CC74C94769440C0851D9DF0869FCE7F Ref B: CO1EDGE0315 Ref C: 2018-04-11T18:47:40Z
{
  "_type": "WebPage",
  "name": "SwiftKey - Smart prediction technology for easier mobile typing",
  "url": "https:\/\/swiftkey.com\/en",
  "description": "Discover the best Android and iPhone and iPad apps for faster, easier typing with emoji, colorful themes and more - download SwiftKey Keyboard free today.",
  "isFamilyFriendly": true,
  "primaryImageOfPage": {
    "contentUrl": "https:\/\/swiftkey.com\/images\/og\/default.jpg"
  }
}

````
## <a name="scenarios"></a>案例 

URL 預覽 API 支援 Web 資源的簡短說明。 開發人員會使用它來建立豐富的預覽經驗。  使用者可以共用網頁、新聞、部落格、論壇等等，也可以將這些設為書籤。此 API 也可以用於內容仲裁。    

應用程式使用 URL 預覽，藉由對於預覽的 URL 指派的查詢將 Web 要求傳送至端點。  JSON 回應包含預覽資訊：名稱、資源描述、*familyFriendly* 旗標，以及可存取代表性映像和完整線上資源的連結。 

## <a name="terms-of-use"></a>使用規定
僅於使用者在社群媒體、聊天機器人或類似產品上進行 URL 分享時，使用來自專案 URL 預覽的資料以顯示超連結至其來源網站的預覽文字與縮圖映像。 請勿複製、儲存或快取從專案 URL 預覽收到的任何資料。 接受任何要求以停用可能會從網站或內容擁有者收到的預覽。

您或代表您的第三方不得使用、保留、儲存、快取、共用或散發 URL 預覽 API 的任何資料進行測試、開發、訓練、散發或提供給任何非 Microsoft 服務或功能。 

## <a name="throttling-requests"></a>節流要求

[!INCLUDE [cognitive-services-bing-throttling-requests](../../../../includes/cognitive-services-bing-throttling-requests.md)]

## <a name="next-steps"></a>後續步驟
- [C# 快速入門](csharp.md)
- [JAVA 快速入門](java-quickstart.md)
- [JavaScript 快速入門](javascript.md)
- [Node 快速入門](node-quickstart.md)
- [Python 快速入門](python-quickstart.md)
