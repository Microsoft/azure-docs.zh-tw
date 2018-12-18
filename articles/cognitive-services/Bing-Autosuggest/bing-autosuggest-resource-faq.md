---
title: 關於 Azure 自動建議 API 的常見問題集 (FAQ) | Microsoft Docs
description: 取得 Azure 上的 Azure 認知服務自動建議 API 常見問題的解答。
services: cognitive-services
author: HeidiSteen
manager: jhubbard
ms.service: cognitive-services
ms.component: bing-autosuggest
ms.topic: article
ms.date: 07/26/2017
ms.author: heidist
ms.openlocfilehash: 00b91728bcfec52ff30697f080d5c2619bab79a8
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35369903"
---
# <a name="frequently-asked-questions-faq-about-autosuggest-api-cognitive-services"></a>關於自動建議 API (認知服務) 的常見問題集 (FAQ)
 
 尋找 Azure 認知服務自動建議 API 的相關概念、程式碼和案例的常見問題解答。

### <a name="how-do-i-get-the-optional-client-headers-when-calling-the-bing-autosuggest-api-from-javascript"></a>從 JavaScript 呼叫 Bing 自動建議 API 時，如何取得選擇性的用戶端標頭？

下列標頭是選擇性的，但建議您將其視為必要的。 這些標頭可協助 Bing 自動建議 API 傳回更精確的結果。

- X-Search-Location
- X-MSEdge-ClientID
- X-MSEdge-ClientIP

不過，當您從 JavaScript 呼叫 Bing 自動建議 API 時，瀏覽器的內建安全性功能可能會讓您無法存取這些標頭的值。

若要解決此問題，您可以透過 CORS Proxy 提出 Bing 自動建議 API 要求。 來自這類 Proxy 的回應包含 `Access-Control-Expose-Headers` 標頭，可將回應標頭列入白名單並提供給 JavaScript 使用。

您可以輕鬆地安裝 CORS Proxy，讓我們的[教學課程應用程式](tutorials/autosuggest.md)存取選擇性用戶端標頭。 首先，請[安裝 Node.js](https://nodejs.org/en/download/) (如果尚未安裝)。 在命令提示字元中，輸入下列命令。

    npm install -g cors-proxy-server

接下來，將 HTML 檔案中的 Bing 自動建議 API 端點變更為：

    http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/Suggestions

最後，使用下列命令啟動 CORS Proxy：

    cors-proxy-server

當您使用教學課程應用程式時，請保留開啟命令視窗；關閉視窗會停止 Proxy。 在可展開的 [HTTP 標頭] 區段搜尋結果下，您現在可以看到 `X-MSEdge-ClientID` 標頭 (及其他標頭)，並確認每個要求的此標頭都相同。

## <a name="next-steps"></a>後續步驟

您的問題是否與缺少特性或功能相關？ 請考慮在我們的 [User Voice 網站](https://cognitive.uservoice.com/)上要求或對它投票。

## <a name="see-also"></a>另請參閱

- [Stack Overflow：認知服務](http://stackoverflow.com/questions/tagged/microsoft-cognitive)
