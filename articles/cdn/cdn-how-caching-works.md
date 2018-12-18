---
title: 快取的運作方式 | Microsoft Docs
description: 快取是在本機儲存資料的程序，以便未來可以更快速地存取對該資料的要求。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/30/2018
ms.author: magattus
ms.openlocfilehash: 563c073e781e2a2bee88b4ecdcdc82541c21ec4f
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092381"
---
# <a name="how-caching-works"></a>快取的運作方式

本文提供一般快取概念，以及 [Azure 內容傳遞網路 (CDN)](cdn-overview.md) 如何使用快取來改善效能的總覽。 如果您需要了解如何在 CDN 端點上自訂快取行為，請參閱[使用快取規則控制 Azure CDN 快取行為](cdn-caching-rules.md)和[使用查詢字串控制 Azure CDN 快取行為](cdn-query-string.md)。

## <a name="introduction-to-caching"></a>快取簡介

快取是在本機儲存資料的程序，以便未來可以更快速地存取對該資料的要求。 在網頁瀏覽器快取這個最常見的快取類型中，網頁瀏覽器會在本機硬碟上本機儲存靜態資料的複本。 網頁瀏覽器可以使用快取來避免多次往返伺服器，並改為在本機存取相同的資料，因此能節省時間和資源。 快取非常適用於本機管理小型的靜態資料，例如靜態映像、CSS 檔案和 JavaScript 檔案。

同樣地，內容傳遞網路會在接近使用者的邊緣伺服器上使用快取，來避免要求移回原點及減少使用者延遲。 不同於僅用於單一使用者的網頁瀏覽器快取，CDN 具有共用快取。 在 CDN 共用的快取中，一位使用者所要求的檔案稍後可由其他使用者存取，從而大幅降低對原始伺服器的要求數目。

無法快取經常變更或個別使用者特有的動態資源。 不過，這些類型的資源，可以利用 Azure 內容傳遞網路上的動態站台加速 (DSA) 最佳化來改進效能。

原始伺服器與使用者之間的多個層級可能會發生快取：

- 網頁伺服器：使用共用快取 (適用於多個使用者)。
- 內容傳遞網路：使用共用快取 (適用於多個使用者)。
- 網際網路服務提供者 (ISP)：使用共用快取 (適用於多個使用者)。
- 網頁瀏覽器：使用私人快取 (適用於一位使用者)。

每個快取通常會管理它自己的資源有效期限，並在檔案過期時執行驗證。 在 HTTP 快取規格 [RFC 7234](https://tools.ietf.org/html/rfc7234) 中會定義此行為。

### <a name="resource-freshness"></a>資源有效期限

因為快取的資源可能會過期或是過時 (相較於原始伺服器上的對應資源)，任何快取機制都必須能夠控制內容重新整理的時間。 為了節省時間和頻寬耗用量，快取的資源並不會在每次存取時都比較原始伺服器上的版本。 相反地，只要快取的資源被視為是全新的，就會假設是最新版本，並直接傳送至用戶端。 當快取資源的存在時間小於快取設定所定義的存在時間或期間，就會視為全新的。 例如，當瀏覽器重新載入網頁時，它會驗證您硬碟機上的每個快取資源是最新的，並將其載入。 如果資源不是最新 (過時) 的，就會從伺服器載入最新的複本。

### <a name="validation"></a>驗證

如果資源被視為過時，就會要求原始伺服器進行驗證，也就是，判斷快取中的資料是否仍符合原始伺服器上的資料。 如果已在原始伺服器上修改檔案，快取就會更新其資源的版本。 否則，如果資源是新的，就會直接從快取傳遞資料，而無須事先驗證。

### <a name="cdn-caching"></a>CDN 快取

快取對於 CDN 的運作方式是整數，可加速傳遞並減少靜態資產的來源載入，例如映像、字型及影片。 在 CDN 快取中，靜態資源會選擇性地儲存在策略性放置的伺服器上，對於使用者較接近本機，並提供下列優點：

- 因為大部分的 web 流量都是靜態的 (例如映像、字型和視訊)，CDN 快取會降低網路延遲，方法是將內容移至更接近使用者，從而降低資料傳送的距離。

- 快取可以透過卸載 CDN 的工作來減少網路流量，以及在原始伺服器上的負載。 這樣做可以降低應用程式的成本和資源需求，即使是有大量使用者時也一樣。

類似於快取在網頁瀏覽器中實作的方式，您可以傳送快取指示詞標頭來控制快取在 CDN 中執行的方式。 快取指示詞標頭是 HTTP 標頭，通常是由原始伺服器所新增。 雖然這些標頭大部分都是原本就設計用於處理用戶端瀏覽器中的快取，現在有的中繼快取也會使用它們，例如 CDN。 

可以使用兩個標頭來定義快取有效期限：`Cache-Control` 和 `Expires`。 `Cache-Control` 較新，且如果同時存在，會優先於 `Expires`。 另外還有兩種類型的標頭會用於驗證 (稱為驗證程式)：`ETag` 和 `Last-Modified`。 `ETag` 較新，且如果同時定義，會優先於 `Last-Modified`。  

## <a name="cache-directive-headers"></a>快取指示詞標頭

> [!IMPORTANT]
> 根據預設，已針對 DSA 最佳化的 Azure CDN 端點會忽略快取指示詞標頭並略過快取。 針對「**來自 Verizon 的 Azure CDN 標準**」和「**來自 Akamai 的 Azure CDN 標準**」設定檔，您可以使用 [CDN 快取規則](cdn-caching-rules.md)來啟用快取，以調整 Azure CDN 端點處理這些標頭的方式。 只有針對「**來自 Verizon 的 Azure CDN 進階**」，您需使用[規則引擎](cdn-rules-engine.md)來啟用快取。

Azure CDN 支援下列 HTTP 快取指示詞標頭，這些標頭會定義快取持續時間和快取共用。

**Cache-Control：**
- 在 HTTP 1.1 中導入，讓 web 發行者能更充分掌控其內容，並處理 `Expires` 標頭的限制。
- 如果已同時定義 `Expires` 標頭和 `Cache-Control`，則覆寫前者。
- 使用於來自用戶端對 CDN POP 的 HTTP 要求時，依預設所有 Azure CDN 設定檔都會忽略 `Cache-Control`。
- 使用於來自用戶端對 CDN POP 的 HTTP 回應時：
     - **來自 Verizon 的標準/進階 Azure CDN** 和**來自 Microsoft 的標準 Azure CDN** 支援所有 `Cache-Control` 指示詞。
     - **來自 Akamai 的標準 Azure CDN** 僅支援下列 `Cache-Control` 指示詞；會忽略所有其他指示詞：
         - `max-age`：快取可以儲存所指定秒數的內容。 例如： `Cache-Control: max-age=5`。 這個指示詞會指定內容被視為是全新的最大時間量。
         - `no-cache`：快取內容，但每次從快取傳遞內容之前，都要加以驗證。 相當於 `Cache-Control: max-age=0`。
         - `no-store`：一律不會快取內容。 如果先前已儲存內容，請加以移除。

**Expires：**
- 在 HTTP 1.0 中導入舊版的標頭；支援回溯相容性。
- 使用以日期作為基礎的到期時間，以秒為有效位數。 
- 類似於 `Cache-Control: max-age`。
- 使用時機是當 `Cache-Control` 不存在時。

**Pragma：**
   - 根據預設，Azure CDN 不接受。
   - 在 HTTP 1.0 中導入舊版的標頭；支援回溯相容性。
   - 用來作為用戶端要求標頭，並具有下列指示詞：`no-cache`。 這個指示詞會指示伺服器傳送全新版本的資源。
   - `Pragma: no-cache` 相當於 `Cache-Control: no-cache`。

## <a name="validators"></a>驗證程式

當快取過期時，HTTP 快取驗證程式可用來比較檔案的快取版本與原始伺服器上的版本。 **來自 Verizon 的標準/進階 Azure CDN** 預設支援 `ETag` 和 `Last-Modified` 驗證器，而**來自 Microsoft 的標準 Azure CDN** 和**來自 Akamai 的標準 Azure CDN** 預設僅支援 `Last-Modified`。

**ETag：**
- **來自 Verizon 的標準/進階 Azure CDN** 預設支援`ETag` ，而**來自 Microsoft 的標準 Azure CDN** 和來自 **Akamai 的標準 Azure CDN** 則不提供支援。
- `ETag` 會定義對每個檔案和檔案版本是唯一的字串。 例如： `ETag: "17f0ddd99ed5bbe4edffdd6496d7131f"`。
- 在 HTTP 1.1 中導入，且較 `Last-Modified` 更新。 上次修改的日期難以判斷時會很有用。
- 支援強式驗證和弱式驗證；不過，Azure CDN 僅支援強式驗證。 針對強式驗證，兩個資源表示法必須是位元組對位元組相同。 
- 快取會驗證使用 `ETag` 的檔案，方法是傳送要求中具有一或多個 `ETag` 驗證程式的 `If-None-Match` 標頭。 例如： `If-None-Match: "17f0ddd99ed5bbe4edffdd6496d7131f"`。 如果伺服器的版本符合清單上的 `ETag` 驗證程式，它會在其回應中傳送狀態碼 304 (未修改)。 如果版本不同，伺服器會以狀態碼 200 (確定) 和更新的資源回應。

**Last-Modified：**
- 僅針對**來自 Verizon 的標準/進階 Azure CDN** 而言，如果 HTTP 回應中未包含 `ETag`，就會使用 `Last-Modified`。 
- 指定原始伺服器判斷上次修改資源的日期和時間。 例如： `Last-Modified: Thu, 19 Oct 2017 09:28:00 GMT`。
- 快取會使用 `Last-Modified` 來驗證檔案，方法是傳送要求中具有日期和時間 `If-Modified-Since` 的標頭。 原始伺服器會比較該日期與最新資源的 `Last-Modified` 標頭。 如果資源從指定時間起尚未修改，伺服器就會在其回應中傳回狀態碼 304 (未修改)。 如果資源已修改，伺服器會傳回狀態碼 200 (確定) 和更新的資源。

## <a name="determining-which-files-can-be-cached"></a>判斷哪些檔案可快取

並非所有的資源都可以快取。 下表以 HTTP 回應的類型作為基礎，顯示可以快取哪些資源。 無法快取與不符合所有條件之 HTTP 回應共同傳遞的資源。 僅針對**來自 Verizon 的進階 Azure CDN** 而言，您可以使用規則引擎來自訂這當中的某些條件。

|                   | 來自 Microsoft 的 Azure CDN          | Azure CDN from Verizon | Azure CDN from Akamai        |
|-------------------|-----------------------------------|------------------------|------------------------------|
| HTTP 狀態碼 | 200、203、206、300、301、410、416 | 200                    | 200、203、300、301、302、401 |
| HTTP 方法      | GET、HEAD                         | GET                    | GET                          |
| 檔案大小限制  | 300 GB                            | 300 GB                 | - 一般 Web 傳遞最佳化：1.8 GB<br />- 媒體串流最佳化：1.8 GB<br />- 大型檔案最佳化：150 GB |

若要讓**來自 Microsoft 的標準 Azure CDN**  快取在資源上運作，原始伺服器必須支援任何 HEAD 和 GET HTTP 要求，而且資產的所有 HEAD 和 GET HTTP 回應的內容長度值都必須相同。 在 HEAD 要求中，原始伺服器必須支援 HEAD 要求，而且必須以相同的標頭回應，如同它已接收 GET 要求。

## <a name="default-caching-behavior"></a>預設快取行為

下表描述 Azure CDN 產品及其最佳化的預設快取行為。

|    | Microsoft：一般 Web 傳遞 | Verizon：一般 Web 傳遞 | Verizon：DSA | Akamai：一般 Web 傳遞 | Akamai：DSA | Akamai：大型檔案下載 | Akamai：一般或 VOD 媒體串流處理 |
|------------------------|--------|-------|------|--------|------|-------|--------|
| **接受來源**       | 是    | yes   | 否   | 是    | 否   | yes   | 是    |
| **CDN 快取持續時間** | 2 天 |7 天 | None | 7 天 | None | 1 天 | 1 年 |

**接受來源**：指定如果[支援的快取指示詞標頭](#http-cache-directive-headers)存在於原始伺服器的 HTTP 回應中，是否要加以接受。

**CDN 快取持續時間**：指定資源會在 Azure CDN 快取的時間量。 不過，如果**接受來源**為 [是]，且來自原始伺服器的 HTTP 回應中包含快取指示詞標頭 `Expires` 或 `Cache-Control: max-age`，Azure CDN 就會改為使用標頭所指定的持續時間值。 

## <a name="next-steps"></a>後續步驟

- 若要了解如何透過快取規則自訂及覆寫 CDN 上的預設快取行為，請參閱[使用快取規則控制 Azure CDN 快取行為](cdn-caching-rules.md)。 
- 若要了解如何使用查詢字串來控制快取行為，請參閱[使用查詢字串控制 Azure CDN 快取行為](cdn-query-string.md)。



