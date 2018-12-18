---
title: 關於 Bing 拼字檢查 API 的常見問題集 - Azure 認知服務 | Microsoft Docs
description: 在 Azure 上取得 Bing 拼字檢查 API 相關常見問題的解答。
services: cognitive-services
author: HeidiSteen
manager: jhubbard
ms.service: cognitive-services
ms.component: bing-spell-check
ms.topic: conceptual
ms.date: 07/26/2017
ms.author: heidist
ms.openlocfilehash: 87b1f3ed3e0aaa9f3c3c804dc9eac3ee60b4a565
ms.sourcegitcommit: 95d9a6acf29405a533db943b1688612980374272
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/23/2018
ms.locfileid: "35370219"
---
# <a name="frequently-asked-questions-about-the-bing-spell-check-api"></a>Bing 拼字檢查 API 的常見問題集

 尋找 Azure 上 Microsoft 認知服務的 Bing 拼字檢查 API 相關概念、程式碼和案例常見問題的解答。

## <a name="how-do-i-get-the-optional-client-headers-when-calling-the-bing-spell-check-api-from-javascript"></a>從 JavaScript 呼叫 Bing 拼字檢查 API 時，如何取得選擇性的用戶端標頭？

下列標頭為選擇性，但建議您將其視為必要。 這些標頭可協助 Bing 拼字檢查 API 傳回更精確的結果。

- X-Search-Location
- X-MSEdge-ClientID
- X-MSEdge-ClientIP

不過，當您從 JavaScript 呼叫 Bing 拼字檢查 API 時，瀏覽器的內建安全性功能可能會讓您無法存取這些標頭的值。

若要解決此問題，您可以透過 CORS Proxy 提出 Bing 拼字檢查 API 要求。 來自這類 Proxy 的回應包含 `Access-Control-Expose-Headers` 標頭，可將回應標頭列入白名單並提供給 JavaScript 使用。

您可以輕鬆安裝 CORS Proxy，讓[教學課程應用程式](tutorials/spellcheck.md)存取選擇性用戶端標頭。 首先，請[安裝 Node.js](https://nodejs.org/en/download/) (若尚未安裝)。 在命令提示字元中，輸入下列命令。

    npm install -g cors-proxy-server

接下來，將 HTML 檔案中的 Bing 拼字檢查 API 端點變更為：

    http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/spellcheck/

最後，使用下列命令啟動 CORS Proxy：

    cors-proxy-server

當您使用教學課程應用程式時，請保持開啟命令視窗；關閉視窗會停止 Proxy。 在可展開之 [HTTP 標頭] 區段的搜尋結果下，您現在可以看到 `X-MSEdge-ClientID` 標頭 (及其他標頭)，並確認每個要求的此標頭都相同。

## <a name="next-steps"></a>後續步驟

您的問題是否與缺少特性或功能相關？ 請考慮在 [UserVoice 網站](https://cognitive.uservoice.com/)上要求或對它投票。

## <a name="see-also"></a>另請參閱

 [StackOverflow：認知服務](http://stackoverflow.com/questions/tagged/microsoft-cognitive)
