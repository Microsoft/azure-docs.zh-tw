---
title: Azure Front Door Service - 快取 | Microsoft Docs
description: 此文章將協助您了解 Azure Front Door Service 如何監視後端的健康情況
services: frontdoor
documentationcenter: ''
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: sharadag
ms.openlocfilehash: 6c62e2e559749ae8dc29e86d9c2414c28b487995
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965614"
---
# <a name="caching-with-azure-front-door-service"></a>使用 Azure Front Door Service 進行快取
下列文件會利用已啟用快取的路由規則來指定 Front Door 的行為。

## <a name="delivery-of-large-files"></a>傳遞大型檔案
Azure Front Door Service 可傳遞大型檔案且檔案大小沒有上限。 Front Door 會使用稱為物件區塊化的技術。 要求大型檔案時，Front Door 會從後端擷取較小的檔案片段。 接收到完整或位元組範圍檔案要求之後，Front Door 環境就會向後端要求檔案 (以 8 MB 的區塊為單位)。

</br>當區塊抵達 Front Door 環境之後，就會被快取並立即提供給使用者。 然後 Front Door 會以平行方式預先提取下一個區塊。 此預先擷取可確保內容會領先使用者一個區塊，以降低延遲。 此程序一直持續到整個檔案下載完畢 (如有要求)、取得所有位元組範圍 (如有要求)，或用戶端終止連線。

</br>如需位元組範圍要求的詳細資訊，請參閱 [RFC 7233](http://www.rfc-base.org/rfc-7233.html)。
Front Door 會在接收到任何區塊之後進行快取，因此不需要在 Front Door 快取上快取整個檔案。 快取會提供後續的檔案或位元組範圍要求。 如果不會快取所有區塊，就會使用預先擷取來向後端要求區塊。 此最佳化依賴後端的功能來支援位元組範圍要求；如果後端不支援位元組範圍要求，則此最佳化無效。

## <a name="file-compression"></a>檔案壓縮
Front Door 可在邊緣動態壓縮內容，因而對用戶端產生較小且更快速的回應。 所有檔案都符合壓縮資格。 但是，檔案必須是符合壓縮清單資格的 MIME 類型。 目前，Front Door 不允許變更此清單。 目前的清單為：</br>
- "application/eot"
- "application/font"
- "application/font-sfnt"
- "application/javascript"
- "application/json"
- "application/opentype"
- "application/otf"
- "application/pkcs7-mime"
- "application/truetype"
- "application/ttf"，
- "application/vnd.ms-fontobject"
- "application/xhtml+xml"
- "application/xml"
- "application/xml+rss"
- "application/x-font-opentype"
- "application/x-font-truetype"
- "application/x-font-ttf"
- "application/x-httpd-cgi"
- "application/x-mpegurl"
- "application/x-opentype"
- "application/x-otf"
- "application/x-perl"
- "application/x-ttf"
- "application/x-javascript"
- "font/eot"
- "font/ttf"
- "font/otf"
- "font/opentype"
- "image/svg+xml"
- "text/css"
- "text/csv"
- "text/html"
- "text/javascript"
- "text/js"、"text/plain"
- "text/richtext"
- "text/tab-separated-values"
- "text/xml"
- "text/x-script"
- "text/x-component"
- "text/x-java-source"

此外，檔案大小也必須介於 1 KB 到 8 MB (含) 之間。

這些設定檔支援下列壓縮編碼：
- [Gzip (GNU zip)](https://en.wikipedia.org/wiki/Gzip)
- [Brotli](https://en.wikipedia.org/wiki/Brotli)

如果要求支援 gzip 和 Brotli 壓縮，Brotli 壓縮的優先順序最高。</br>
當資產的要求指定壓縮且快取中的要求結果遺失時，Front Door 會在 POP 伺服器上直接對資產執行壓縮。 之後會從快取提供壓縮的檔案。 產生的項目會使用 transfer-encoding: chunked 來傳回。

## <a name="query-string-behavior"></a>查詢字串行為
使用 Front Door，可以控制 Web 要求內含查詢字串時的檔案快取方式。 在包含查詢字串的 Web 要求中，查詢字串是要求中問號 (?) 之後的部分。 查詢字串可以包含一或多個索引鍵/值組，其中的欄位名稱與其值是以等號 (=) 分隔。 每個索引鍵/值組是以 & 符號分隔。 例如： http://www.contoso.com/content.mov?field1=value1&field2=value2。 如果要求的查詢字串中有不止一個索引鍵/值組，其順序無關緊要。
- **忽略查詢字串**：預設模式。 在此模式中，Front Door 會將要求者發出的查詢字串，傳遞至第一個要求的後端並快取資產。 所有後續對該資產提出並由 Front Door 環境提供服務的要求都會忽略查詢字串，直到所快取的資產到期為止。

- **快取所有不重複的 URL**：在此模式中，每個要求都有一個唯一的 URL (包含查詢字串)，會被視為具有專屬快取的唯一資產。 例如，系統會將後端對 `www.example.ashx?q=test1` 要求做出的回應快取於 Front Door 環境中，然後針對後續具有相同查詢字串的快取傳回此回應。 系統快取針對 `www.example.ashx?q=test2` 的要求，會將其視為具有專屬存留時間設定的個別資產。

## <a name="cache-purge"></a>快取清除
Front Door 將會快取資產，直到資產的存留時間 (TTL) 到期。 資產的 TTL 到期之後，當用戶端要求資產時，Front Door 環境將會擷取資產新的更新複本以服務用戶端的要求並儲存重新整理快取。
</br>若要確定使用者一律會取得最新的資產複本，最佳作法是為每個更新設定資產版本，然後將它們發佈為新的 URL。 Front Door 將立即為下一個用戶端要求擷取新的資產。 有時您可能想要清除所有邊緣節點的快取內容，並強制它們全部擷取新的更新的資產。 可能是因為您的 Web 應用程式更新，或快速更新包含不正確資訊的資產。

</br>選取您希望從邊緣節點清除的資產。 如果您希望清除所有資產，請按一下 [全部清除] 核取方塊。 或者，在 [路徑] 文字方塊中輸入每個您想要清除之資產的路徑。 路徑支援下列格式。
1. **單一 URL 清除**︰透過指定完整的 URL (含副檔名) 來清除個別資產，例如 /pictures/strasbourg.png;
2. **萬用字元清除**︰星號 (\*) 可作為萬用字元。 清除路徑中有 /\* 之端點下的所有資料夾、子資料夾與檔案，或指定後接 /\* 的資料夾來清除特定資料夾下的所有子資料夾與檔案，例如 /pictures/\*。
3. **根網域清除**︰清除路徑中有 "/" 之端點的根目錄。

Front Door 上的快取清除是不區分大小寫的。 此外，它們是無從驗證查詢字串的，這表示清除 URL 將會清除它的所有查詢字串變數。 

## <a name="cache-expiration"></a>快取到期
下列標頭的順序可用來判斷項目將在我們的快取中儲存的時間：</br>
1. Cache-Control: s-maxage=<seconds>
2. Cache-Control: maxage=<seconds>
3. Expires: <http-date>

系統會接受指出將不會快取回應的 Cache-Control 回應標頭 (例如，Cache-Control: private、Cache-Control: no-cache 與 Cache-Control: no-store)。 不過，如果 POP 中有多個針對相同 URL 的要求，它們可能共用回應。


## <a name="request-headers"></a>要求標頭

使用快取時，不會將下列要求標頭轉送至後端。
- Authorization
- Content-Length
- Transfer-Encoding

## <a name="next-steps"></a>後續步驟

- 了解如何[建立 Front Door](quickstart-create-front-door.md)。
- 了解 [Front Door 的運作方式](front-door-routing-architecture.md)。