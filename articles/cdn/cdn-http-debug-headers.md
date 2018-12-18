---
title: Azure CDN 規則引擎的 X-EC-Debug HTTP 標頭 | Microsoft Docs
description: X-EC-Debug 快取要求標頭會提供與要求的資產所套用的快取原則有關的其他資訊。 這些標頭是 Verizon 特有的標頭。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2018
ms.author: magattus
ms.openlocfilehash: 4ba42850ee28e2e212d9bc2b7b64be103218757c
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094219"
---
# <a name="x-ec-debug-http-headers-for-azure-cdn-rules-engine"></a>Azure CDN 規則引擎的 X-EC-Debug HTTP 標頭
偵錯快取要求標頭 `X-EC-Debug` 會提供與要求的資產所套用的快取原則有關的其他資訊。 這些標頭是**來自 Verizon 的 Azure CDN 進階**產品特有的標頭。

## <a name="usage"></a>使用量
只有在符合下列條件時，從 POP 伺服器傳送給使用者的回應才會包含 `X-EC-Debug` 標頭：

- 已在指定要求的規則引擎上啟用 [偵錯快取回應標頭](cdn-rules-engine-reference-features.md#debug-cache-response-headers) 功能。
- 指定的要求會定義一組將包含於回應中的偵錯快取回應標頭。

## <a name="requesting-debug-cache-information"></a>要求偵錯快取資訊
在指定的要求中使用下列指示詞，定義將會包含在回應中的偵錯快取資訊：

要求標頭 | 說明 |
---------------|-------------|
X-EC-Debug: x-ec-cache | [快取狀態碼](#cache-status-code-information)
X-EC-Debug: x-ec-cache-remote | [快取狀態碼](#cache-status-code-information)
X-EC-Debug: x-ec-check-cacheable | [可快取](#cacheable-response-header)
X-EC-Debug: x-ec-cache-key | [快取索引鍵](#cache-key-response-header)
X-EC-Debug: x-ec-cache-state | [快取狀態](#cache-state-response-header)

### <a name="syntax"></a>語法

偵錯快取回應標頭可能是藉由在要求中包含下列標頭和指定的指示詞來要求：

`X-EC-Debug: Directive1,Directive2,DirectiveN`

### <a name="sample-x-ec-debug-header"></a>範例 X-EC-Debug 標頭

`X-EC-Debug: x-ec-cache,x-ec-check-cacheable,x-ec-cache-key,x-ec-cache-state`

## <a name="cache-status-code-information"></a>快取狀態碼資訊
X-EC-Debug 回應標頭可透過下列指示詞來識別伺服器及其處理回應的方式：

頁首 | 說明
-------|------------
X-EC-Debug: x-ec-cache | 每當透過 CDN 路由內容時，就會報告此標頭。 此標頭可識別已回應要求的 POP 伺服器。
X-EC-Debug: x-ec-cache-remote | 只有在原始保護盾伺服器或 ADN 閘道伺服器上快取要求的內容時，才會報告此標頭。

### <a name="response-header-format"></a>回應標頭格式

X-EC-Debug 標頭會以下列格式報告快取狀態碼資訊：

- `X-EC-Debug: x-ec-cache: <StatusCode from Platform (POP/ID)>`

- `X-EC-Debug: x-ec-cache-remote: <StatusCode from Platform (POP/ID)>`

前述回應標頭語法中使用的詞彙定義如下：
- StatusCode：指出 CDN 如何處理要求的內容 (透過快取狀態碼來表示)。
    
    未經授權的要求因權杖型驗證而遭到拒絕時，可能會報告 TCP_DENIED 狀態碼，而不是 NONE。 不過，在檢視快取狀態報告或原始記錄資料時，仍會繼續使用 NONE 狀態碼。

- 平台：表示在其上要求內容的平台。 下列程式碼適用於此欄位：

    代碼  | 平台
    ------| --------
    ECAcc | HTTP Large
    ECS   | HTTP Small
    ECD   | 應用程式傳遞網路 (ADN)

- POP：表示處理要求的 [POP](cdn-pop-abbreviations.md)。 

### <a name="sample-response-headers"></a>範例回應標頭

下列範例標頭提供要求的快取狀態碼資訊：

- `X-EC-Debug: x-ec-cache: TCP_HIT from ECD (lga/0FE8)`

- `X-EC-Debug: x-ec-cache-remote: TCP_HIT from ECD (dca/EF00)`

## <a name="cacheable-response-header"></a>可快取回應標頭
`X-EC-Debug: x-ec-check-cacheable` 回應標頭會指出要求的內容是否可能已快取。

此回應標頭不會指出快取是否已執行。 它只會指出要求是否符合快取的資格。

### <a name="response-header-format"></a>回應標頭格式

報告要求是否可能已快取的 `X-EC-Debug` 回應標頭會採用下列格式：

`X-EC-Debug: x-ec-check-cacheable: <cacheable status>`

前述回應標頭語法中使用的詞彙定義如下：

值  | 說明
-------| --------
是    | 指出要求的內容符合快取的資格。
否     | 指出要求的內容不符合快取的資格。 此狀態可能是下列其中一個原因所造成： <br /> - 客戶特定組態：您的帳戶特有的組態可能防止 POP 伺服器快取某資產。 例如，規則引擎可能對符合資格的要求啟用了「略過快取」功能，而防止快取某項資產。<br /> - 快取回應標頭：要求的資產的 Cache-Control 和 Expires 標頭可能防止 POP 伺服器快取該資產。
UNKNOWN | 指出伺服器無法評估要求的資產是否可快取。 此狀態通常在要求因權杖型驗證而遭到拒絕時發生。

### <a name="sample-response-header"></a>範例回應標頭

下列範例回應標頭會指出要求的內容是否可能已快取：

`X-EC-Debug: x-ec-check-cacheable: YES`

## <a name="cache-key-response-header"></a>快取索引鍵回應標頭
`X-EC-Debug: x-ec-cache-key` 回應標頭會指出與要求的內容相關聯的實體快取索引鍵。 實體快取索引鍵包含一個基於快取目的而識別資產的路徑。 換句話說，伺服器將根據資產的路徑 (如其快取索引鍵所定義) 來檢查它的快取版本。

此實體的快取索引鍵的開頭為雙正斜線 (//)，後面接著用來要求內容的通訊協定 (HTTP 或 HTTPS)。 此通訊協定後面接著所要求之資產的相對路徑，其開頭為內容存取點 (例如 _/000001/_)。

根據預設，HTTP 平台會設定為使用*標準快取*，這表示快取機制會忽略查詢字串。 這種設定可防止快取索引鍵包含查詢字串資料。

查詢字串若記錄在快取索引鍵中，則會轉換為其對等的雜湊，然後在要求的資產名稱和其副檔名 (例如，資產&lt;雜湊值&gt;.html) 之間插入。

### <a name="response-header-format"></a>回應標頭格式

`X-EC-Debug` 回應標頭會以下列格式報告實體快取索引鍵資訊：

`X-EC-Debug: x-ec-cache-key: CacheKey`

### <a name="sample-response-header"></a>範例回應標頭

下列範例回應標頭會指出所要求之內容的實體快取索引鍵：

`X-EC-Debug: x-ec-cache-key: //http/800001/origin/images/foo.jpg`

## <a name="cache-state-response-header"></a>快取狀態回應標頭
`X-EC-Debug: x-ec-cache-state` 回應標頭會指出要求的內容在被要求時的快取狀態。

### <a name="response-header-format"></a>回應標頭格式

`X-EC-Debug` 回應標頭會以下列格式報告快取狀態資訊：

`X-EC-Debug: x-ec-cache-state: max-age=MASeconds (MATimePeriod); cache-ts=UnixTime (ddd, dd MMM yyyy HH:mm:ss GMT); cache-age=CASeconds (CATimePeriod); remaining-ttl=RTSeconds (RTTimePeriod); expires-delta=ExpiresSeconds`

前述回應標頭語法中使用的詞彙定義如下：

- MASeconds：指出所要求之內容的 Cache-Control 標頭所定義的最大存留期 (以秒為單位)。

- MATimePeriod：將最大存留期值 (也就是 MASeconds) 轉換為採用較大單位 (例如天) 的近似對等值。 

- UnixTime：指出要求的內容以 Unix 時間 (也稱為 POSIX 時間或 Unix Epoch) 表示的快取時間戳記。 快取時間戳記可指出資產的 TTL 開始進行計算的日期/時間。 

    如果原始伺服器未使用第三方 HTTP 快取伺服器，或該伺服器未傳回 Age 回應標頭，則在擷取或重新驗證資產時，快取時間戳記將一律為日期/時間。 否則，POP 伺服器將使用 [存留期] 欄位來計算資產的 TTL，如下所示：Retrieval/RevalidateDateTime - Age。

- ddd, dd MMM yyyy HH:mm:ss GMT：指出所要求之內容的快取時間戳記。 如需詳細資訊，請參閱前述的 UnixTime 詞彙。

- CASeconds：指出自快取時間戳記起所經過的秒數。

- RTSeconds：指出快取的內容被視為有效的剩餘秒數。 此值的計算方式如下：RTSeconds = 最大存留期 - 快取存留期。

- RTTimePeriod：將剩餘 TTL 值 (也就是 RTSeconds) 轉換為採用較大單位 (例如天) 的近似對等值。

- ExpiresSeconds：指出到達 `Expires` 回應標頭中指定的日期/時間前的剩餘秒數。 如果 `Expires` 回應標頭未包含在回應中，則此詞彙的值為 *none*。

### <a name="sample-response-header"></a>範例回應標頭

下列範例回應標頭會指出要求的內容在被要求時的快取狀態：

```X-EC-Debug: x-ec-cache-state: max-age=604800 (7d); cache-ts=1341802519 (Mon, 09 Jul 2012 02:55:19 GMT); cache-age=0 (0s); remaining-ttl=604800 (7d); expires-delta=none```

