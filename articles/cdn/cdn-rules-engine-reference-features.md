---
title: Azure CDN 規則引擎功能 | Microsoft Docs
description: Azure CDN 規則引擎功能的參考文件。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.assetid: 669ef140-a6dd-4b62-9b9d-3f375a14215e
ms.service: cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/09/2018
ms.author: magattus
ms.openlocfilehash: d5be292c66a07f43b40d12e10e4939d9d91559e1
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49395236"
---
# <a name="azure-cdn-rules-engine-features"></a>Azure CDN 規則引擎功能
本本會針對 Azure 內容傳遞網路 (CDN) [規則引擎](cdn-rules-engine.md)列出可用功能的詳細說明。

規則的第三個部分是功能。 功能會定義動作類型，以套用到透過一組比對條件來識別的要求類型。

## <a name="access-features"></a>存取功能

這些功能是設計來控制內容的存取權。

名稱 | 目的
-----|--------
[拒絕存取 (403)](#deny-access-403) | 判斷所有要求是否已遭拒絕且含有 [403 禁止] 回應。
[權杖驗證](#token-auth) | 判斷是否將權杖型驗證套用到要求。
[權杖驗證拒絕代碼](#token-auth-denial-code) | 判斷在要求因權杖型驗證而遭到拒絕時傳回給使用者的回應類型。
[權杖驗證會忽略 URL 的大小寫](#token-auth-ignore-url-case) | 判斷透過權杖型驗證所做的 URL 比較是否區分大小寫。
[權杖驗證參數](#token-auth-parameter) | 判斷是否應將權杖型驗證查詢字串參數重新命名。


## <a name="caching-features"></a>快取功能

這些功能是設計來自訂快取內容的時機和方法。

名稱 | 目的
-----|--------
[頻寬參數](#bandwidth-parameters) | 判斷是否使用頻寬節流設定參數 (例如 ec_rate 和 ec_prebuf)。
[頻寬節流設定](#bandwidth-throttling) | 針對存在點 (POP) 所提供的回應進行頻寬節流。
[略過快取](#bypass-cache) | 判斷要求是否應略過快取。
[Cache-Control 標頭處理](#cache-control-header-treatment) | 當 [外部最大壽命] 功能為作用中時，透過 POP 來控制 `Cache-Control` 標頭的產生。
[快取索引鍵查詢字串](#cache-key-query-string) | 判斷快取索引鍵會包含或排除與要求相關聯的佇列字串參數。
[快取索引鍵重寫](#cache-key-rewrite) | 重寫與要求相關聯的快取索引鍵。
[完成快取填滿](#complete-cache-fill) | 決定當要求導致在 POP 上發生部分快取遺失時要執行的動作。
[壓縮檔案類型](#compress-file-types) | 定義將在伺服器上壓縮的檔案之檔案格式。
[預設的內部最大壽命](#default-internal-max-age) | 決定 POP 向原始伺服器重新驗證快取之前的預設最大壽命間隔。
[Expires 標頭處理](#expires-header-treatment) | 當 [外部最大壽命] 功能為作用中時，透過 POP 來控制 `Expires` 標頭的產生。
[外部最大壽命](#external-max-age) | 決定瀏覽器向 POP 重新驗證快取之前的最大壽命間隔。
[強制執行內部最大壽命](#force-internal-max-age) | 決定 POP 向原始伺服器重新驗證快取之前的最大壽命間隔。
[H.264 支援 (HTTP 漸進式下載)](#h264-support-http-progressive-download) | 判斷可能用於串流處理內容的 H.264 檔案格式類型。
[接受 No-Cache 要求](#honor-no-cache-request) | 判斷是否要將 HTTP 用戶端的 no-cache 要求轉送到原始伺服器。
[忽略原始的 No-Cache](#ignore-origin-no-cache) | 判斷 CDN 是否忽略原始伺服器所提供的特定指示詞。
[忽略無法滿足的範圍](#ignore-unsatisfiable-ranges) | 判斷在要求產生「416 無法滿足的要求範圍」狀態代碼時傳回給用戶端的要求。
[內部最大過時](#internal-max-stale) | 控制當 POP 無法向原始伺服器重新驗證所快取的資產時，在過了標準到期時間多久之後仍可從 POP 提供所快取的資產。
[部分快取共用](#partial-cache-sharing) | 判斷要求是否可以產生部分快取的內容。
[預先驗證快取的內容](#prevalidate-cached-content) | 在快取內容的 TTL 到期之前，判斷其是否適合進行早期重新驗證。
[重新整理零位元組的快取檔案](#refresh-zero-byte-cache-files) | 決定 POP 如何處理 HTTP 用戶端的 0 位元組快取資產要求。
[設定可快取的狀態碼](#set-cacheable-status-codes) | 定義一組可產生快取內容的狀態碼。
[發生錯誤時傳遞過時的內容](#stale-content-delivery-on-error) | 判斷在快取重新驗證期間發生錯誤時，或者在接收到來自客戶原始伺服器的要求內容時，是否要傳遞到期的快取內容。
[在重新驗證時過期](#stale-while-revalidate) | 藉由允許 POP 在進行重新驗證時提供過時的用戶端給要求者來改善效能。

## <a name="comment-feature"></a>註解功能

這項功能設計用來提供規則內的其他資訊。

名稱 | 目的
-----|--------
[Comment](#comment) | 能夠在規則中新增附註。
 
## <a name="header-features"></a>標題功能

這些功能是設計來新增、修改或刪除要求或回應的標頭。

名稱 | 目的
-----|--------
[Age 回應標頭](#age-response-header) | 判斷 Age 回應標頭是否包含於傳送給要求者的回應中。
[偵錯快取回應標頭](#debug-cache-response-headers) | 判斷回應是否會包含於 X-EC-Debug 回應標頭中，其會在快取原則上提供要求資產的相關資訊。
[修改用戶端要求標頭](#modify-client-request-header) | 覆寫、附加或刪除要求的標頭。
[修改用戶端回應標頭](#modify-client-response-header) | 覆寫、附加或刪除回應的標頭。
[設定用戶端 IP 自訂標頭](#set-client-ip-custom-header) | 允許將要新增到要求的要求用戶端 IP 位址做為自訂要求標頭。


## <a name="logging-features"></a>記錄功能

這些功能是設計來自訂儲存於原始記錄檔中的資料。

名稱 | 目的
-----|--------
[自訂記錄欄位 1](#custom-log-field-1) | 判斷要指派給原始記錄檔中自訂記錄欄位的格式和內容。
[記錄查詢字串](#log-query-string) | 判斷查詢字串以及 URL 是否會一起儲存於存取記錄中。


<!---
## Optimize

These features determine whether a request will undergo the optimizations provided by Edge Optimizer.

Name | Purpose
-----|--------
Edge Optimizer | Determines whether Edge Optimizer can be applied to a request.
Edge Optimizer – Instantiate Configuration | Instantiates or activates the Edge Optimizer configuration associated with a site.

### Edge Optimizer
**Purpose:** Determines whether Edge Optimizer can be applied to a request.

If this feature has been enabled, then the following criteria must also be met before the request will be processed by Edge Optimizer:

- The requested content must use an edge CNAME URL.
- The edge CNAME referenced in the URL must correspond to a site whose configuration has been activated in a rule.

This feature requires the ADN platform and the Edge Optimizer feature.

Value|Result
-|-
Enabled|Indicates that the request is eligible for Edge Optimizer processing.
Disabled|Restores the default behavior. The default behavior is to deliver content over the ADN platform without any additional processing.

**Default Behavior:** Disabled
 

### Edge Optimizer - Instantiate Configuration
**Purpose:** Instantiates or activates the Edge Optimizer configuration associated with a site.

This feature requires the ADN platform and the Edge Optimizer feature.

Key information:

- Instantiation of a site configuration is required before requests to the corresponding edge CNAME can be processed by Edge Optimizer.
- This instantiation only needs to be performed a single time per site configuration. A site configuration that has been instantiated will remain in that state until the Edge Optimizer – Instantiate Configuration feature that references it is removed from the rule.
- The instantiation of a site configuration does not mean that all requests to the corresponding edge CNAME will automatically be processed by Edge Optimizer. The Edge Optimizer feature determines whether an individual request will be processed.

If the desired site does not appear in the list, then you should edit its configuration and verify that the Active option has been marked.

**Default Behavior:** Site configurations are inactive by default.
--->

## <a name="origin-features"></a>原始功能

這些功能是設計來控制 CDN 與原始伺服器通訊的方式。

名稱 | 目的
-----|--------
[最大 Keep-Alive 要求數目](#maximum-keep-alive-requests) | 判斷在關閉 Keep-Alive 連線之前，適用於該連線的最大要求數目。
[Proxy 特殊標頭](#proxy-special-headers) | 定義從 POP 轉送到原始伺服器的一組 CDN 特定要求標頭。


## <a name="specialty-features"></a>特性功能

這些功能可為進階使用者提供進階功能。

名稱 | 目的
-----|--------
[可快取的 HTTP 方法](#cacheable-http-methods) | 判斷可在網路上快取的其他 HTTP 方法組。
[可快取的要求主體大小](#cacheable-request-body-size) | 定義用以判斷是否可快取 POST 回應的臨界值。
[User 變數](#user-variable) | 僅供內部使用。

 
## <a name="url-features"></a>URL 功能

這些功能可讓要求重新導向至不同的 URL 或重寫為不同的 URL。

名稱 | 目的
-----|--------
[遵循重新導向](#follow-redirects) | 判斷要求是否可以重新導向至定義於客戶原始伺服器所傳回之位置標頭中的主機名稱。
[URL 重新導向](#url-redirect) | 透過位置標頭將要求重新導向。
[URL 重寫](#url-rewrite)  | 重寫要求 URL。



## <a name="azure-cdn-rules-engine-features-reference"></a>Azure CDN 規則引擎功能參考

---
### <a name="age-response-header"></a>Age 回應標頭
**目的**：判斷 Age 回應標頭是否包含於傳送給要求者的回應中。
值|結果
--|--
已啟用 | Age 回應標頭會包含於傳送給要求者的回應中。
已停用 | Age 回應標頭會從傳送給要求者的回應中排除。

**預設行為**：已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="bandwidth-parameters"></a>頻寬參數
**目的：** 判斷是否使用頻寬節流參數 (例如 ec_rate 和 ec_prebuf)。

頻寬節流參數會判斷是否要將用戶端要求的資料傳輸速率限制為自訂的速率。

值|結果
--|--
已啟用|允許 POP 接受頻寬節流設定要求。
已停用|導致 POP 忽略頻寬節流設定參數。 通常會提供要求的內容 (亦即，不需頻寬節流)。

**預設行為：** 已啟用。
 
[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="bandwidth-throttling"></a>頻寬節流設定
**目的：** 針對 POP 所提供的回應進行頻寬節流。

您必須定義下列這兩個選項，才能正確設定頻寬節流設定。

選項|說明
--|--
每秒 KB 數|將此選項設定為可能用來傳遞回應的最大頻寬 (每秒 KB 數)。
Prebuf 秒|將此選項設定為 POP 在頻寬進行節流前所等候的秒數。 這段不受頻寬限制之期間的用意是防止媒體播放器因為頻寬節流設定而遇到間斷或緩衝處理問題。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="bypass-cache"></a>略過快取
**目的：** 判斷要求是否應略過快取。

值|結果
--|--
已啟用|導致將所有要求都直接導向至原始伺服器，即使先前已將內容快取在 POP 上也一樣。
已停用|導致 POP 根據其回應標頭中所定義的快取原則來快取資產。

**預設行為：**

- **HTTP 大型︰** 已停用

<!---
- **ADN:** Enabled

--->

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="cacheable-http-methods"></a>可快取的 HTTP 方法
**目的：** 判斷一組可在網路上快取的其他 HTTP 方法。

重要資訊：

- 此功能假設應一律快取 GET 回應。 如此一來，在設定此功能時，就不應包含 GET HTTP 方法。
- 此功能僅支援 POST HTTP 方法。 將此功能設定為 `POST`，以啟用 POST 回應快取。
- 根據預設，只會快取主體小於 14 Kb 的要求。 使用 [可快取的要求主體大小] 功能來設定要求主體大小上限。

**預設行為︰** 只會快取 GET 回應。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="cacheable-request-body-size"></a>可快取的要求主體大小
**目的：** 定義用以判斷是否可快取 POST 回應的臨界值。

此臨界值是藉由指定要求主體大小上限來決定。 系統不會快取包含較大要求主體的要求。

重要資訊：

- 只有在 POST 回應適合用來快取時，才適用此功能。 使用 [可快取的 HTTP 方法] 功能來啟用 POST 要求快取。
- 已針對下列項目將要求主體納入考量：
    - x-www-form-urlencoded 值
    - 確保唯一的快取索引鍵
- 定義大型的要求主體大小上限可能會影響資料傳遞效能。
    - **建議值：** 14 Kb
    - **最小值︰** 1 Kb

**預設行為︰** 14 Kb

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="cache-control-header-treatment"></a>Cache-Control 標頭處理
**目的：** 當 [外部最大壽命] 功能為作用中時，透過 POP 來控制 `Cache-Control` 標頭的產生。

完成這類型組態的最簡單方式是將 [外部最大壽命] 和 [Cache-Control 標頭處理] 功能放在同一個陳述式中。

值|結果
--|--
覆寫|確保會進行下列動作：<br/> - 覆寫原始伺服器所產生的 `Cache-Control` 標頭。 <br/>- 將 [外部最大壽命] 功能所產生的 `Cache-Control` 標頭新增至回應。
傳遞|確保永遠不會將 [外部最大壽命] 功能所產生的 `Cache-Control` 標頭新增至回應。 <br/> 如果原始伺服器產生 `Cache-Control` 標頭，它就會傳遞給使用者。 <br/> 如果原始伺服器未產生 `Cache-Control` 標頭，則此選項可能導致 `Cache-Control` 標頭不會包含回應標頭。
如果遺失則加入|如果 `Cache-Control` 標頭不是接收自原始伺服器，則此選項會新增 [外部最大壽命] 功能所產生的 `Cache-Control` 標頭。 若要確保為所有資產指派 `Cache-Control` 標頭，此選項非常有用。
移除| 此選項可確保標頭回應中不會包含 `Cache-Control` 標頭。 如果已經指派 `Cache-Control` 標頭，則會從標頭回應中移除它。

**預設行為：** 覆寫。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="cache-key-query-string"></a>快取索引鍵查詢字串
**目的：** 判斷快取索引鍵會包含或排除與要求相關聯的佇列字串參數。

重要資訊：

- 指定一或多個查詢字串參數名稱，然後以單一空格分隔每個參數名稱。
- 此功能會判斷要在快取索引鍵中包含查詢字串參數或從中排除。 下表提供每個選項的其他資訊。

類型|說明
--|--
 包含|  指出應該在快取索引鍵中包含每個指定的參數。 系統會針對每個要求產生唯一的快取索引鍵，其中包含此功能中所定義之查詢字串參數的唯一值。 
 全部包含  |指出會針對每個包含唯一查詢字串的資產要求建立唯一的快取索引鍵。 通常不建議使用此類型的設定，因為它可能會導致一小部分的快取命中數。 快取命中數過低會使原始伺服器的負載增加，因為它必須提供更多的要求。 這個組態會複製 [查詢字串快取] 頁面上名為「unique-cache」的快取行為。 
 排除 | 指出只會從快取索引鍵中排除指定的參數。 所有的其他查詢字串參數都會包含於快取索引鍵中。 
 全部排除  |指出會從快取索引鍵中排除所有查詢字串參數。 此組態會複製 [查詢字串快取] 頁面上的 "standard-cache" 預設快取行為。  

規則引擎可讓您自訂實作查詢字串快取的方式。 例如，您可以指定只在特定位置或檔案類型上執行查詢字串快取。

若要複製 [查詢字串快取] 頁面上的 "no-cache" 查詢字串快取行為，請建立規則，其中包含 [URL 查詢萬用字元] 比對條件和 [略過快取] 功能。 將 [URL 查詢萬用字元] 比對條件設為星號 (*)。

>[!IMPORTANT] 
> 如果已針對此帳戶上的任何路徑啟用權杖授權，則標準快取模式就會是唯一可用於查詢字串快取的模式。 如需詳細資訊，請參閱[使用查詢字串控制 Azure CDN 快取行為](cdn-query-string-premium.md)。

#### <a name="sample-scenarios"></a>範例案例

下列範例是這項功能的使用方式，會提供範例要求和預設快取金鑰：

- **要求範例︰** http://wpc.0001.&lt;網域&gt;/800001/Origin/folder/asset.htm?sessionid=1234&language=EN&userid=01
- **預設的快取索引鍵︰**/800001/Origin/folder/asset.htm

##### <a name="include"></a>包含

範例組態：

- **類型︰** 包括
- **參數︰** 語言

這類型的組態會產生下列查詢字串參數快取索引鍵：

    /800001/Origin/folder/asset.htm?language=EN

##### <a name="include-all"></a>全部包含

範例組態：

- **類型︰** 全部包括

這類型的組態會產生下列查詢字串參數快取索引鍵：

    /800001/Origin/folder/asset.htm?sessionid=1234&language=EN&userid=01

##### <a name="exclude"></a>排除

範例組態：

- **類型︰** 排除
- **參數︰** 工作階段識別碼的使用者識別碼

這類型的組態會產生下列查詢字串參數快取索引鍵：

    /800001/Origin/folder/asset.htm?language=EN

##### <a name="exclude-all"></a>全部排除

範例組態：

- **類型︰** 全部排除

這類型的組態會產生下列查詢字串參數快取索引鍵：

    /800001/Origin/folder/asset.htm

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="cache-key-rewrite"></a>快取索引鍵重寫
**目的：** 重寫與要求相關聯的快取索引鍵。

快取索引鍵是基於快取目的識別資產的相對路徑。 換句話說，伺服器會根據資產的路徑 (如其快取索引鍵所定義) 來檢查它的快取版本。

藉由定義兩個下列選項來設定此功能：

選項|說明
--|--
原始路徑| 定義將重寫快取索引鍵之要求類型的相對路徑。 相對路徑可藉由選取基底原始路徑，然後定義規則運算式模式來定義。
新路徑|定義新快取索引鍵的相對路徑。 相對路徑可藉由選取基底原始路徑，然後定義規則運算式模式來定義。 此相對路徑可使用 [HTTP 變數](cdn-http-variables.md)來動態建構。
**預設行為︰** 要求的快取索引鍵是依要求 URI 來判斷。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="comment"></a>註解
**目的：** 能夠在規則中加入附註。

此功能的用途之一是提供一般用途的規則，或是為何要將特定比對條件或功能加入至規則的其他相關資訊。

重要資訊：

- 您最多可以指定 150 個字元。
- 僅使用英數字元。
- 此功能不會影響規則的行為。 這只表示會提供一個區域，讓您可在其中提供資訊以供未來參考之用，或是有助於疑難排解規則的資訊。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="complete-cache-fill"></a>完成快取填滿
**目的：** 決定當要求導致在 POP 上發生部分快取遺失時要執行的動作。

部分快取遺失描述的是未完全下載至 POP 之資產的快取狀態。 如果資產只有部分快取在 POP 上，則會將該資產的下一個要求再次轉送給原始伺服器。
<!---
This feature is not available for the ADN platform. The typical traffic on this platform consists of relatively small assets. The size of the assets served through these platforms helps mitigate the effects of partial cache misses, since the next request will typically result in the asset being cached on that POP.

--->
部分快取遺失通常發生於使用者中止下載之後，或發生於只要求使用 HTTP 範圍要求的資產。 對於通常不會完整下載的大型資產 (例如，影片) 而言，此功能非常實用。 因此，HTTP 大型平台上預設會啟用此功能。 所有其他平台上則會加以停用。

針對 HTTP 大型平台保留預設組態，因為它可減少客戶原始伺服器上的負載，並提高客戶下載內容的速度。

值|結果
--|--
已啟用|還原預設行為。 預設行為是強制 POP 在背景從原始伺服器起始資產擷取。 之後，資產就會在 POP 的本機快取中。
已停用|防止 POP 在背景執行資產擷取。 結果是從該區域對該資產發出下一個要求時，會導致 POP 向客戶原始伺服器要求該資產。

**預設行為：** 已啟用。

#### <a name="compatibility"></a>相容性
由於追蹤快取設定的行為，因而導致此功能無法與下列比對條件產生關聯： 
- AS 號碼
- 用戶端 IP 位址
- Cookie 參數
- Cookie 參數 Regex
- 國家 (地區)
- 裝置
- 邊緣 Cname
- 轉介網域
- 要求標頭常值
- 要求標頭 Regex
- 要求標頭萬用字元
- 要求方法
- 要求配置
- URL 查詢常值
- URL 查詢 Regex
- URL 查詢萬用字元
- URL 查詢參數

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="compress-file-types"></a>壓縮檔案類型
**目的：** 定義將在伺服器上壓縮的檔案之檔案格式。

您可以使用檔案格式的網際網路媒體類型 (例如，內容類型) 來指定檔案格式。 網際網路媒體類型是與平台無關的中繼資料，讓伺服器能夠識別特定資產的檔案格式。 以下提供常用的網際網路媒體類型清單。

網際網路媒體類型|說明
--|--
text/plain|純文字檔案
text/html| HTML 檔案
text/css|階層式樣式表 (CSS)
application/x-javascript|Javascript
application/javascript|Javascript
重要資訊：

- 使用單一空格來分隔每個網際網路媒體類型，藉以指定多個類型。 
- 此功能只會壓縮大小小於 1 MB 的資產。 伺服器不會壓縮較大型的資產。
- 諸如影片、視訊和音訊媒體資產 (例如，JPG、MP3、MP4 等) 的特定類型內容均已壓縮。 因為這些資產類型的額外壓縮不會大幅降低檔案大小，建議您不要在這些類型的資產上啟用壓縮。
- 不支援萬用字元 (例如星號)。
- 將此功能新增至規則之前，請確定會針對套用此規則的平台，在 [壓縮] 頁面上設定 [壓縮已停用] 選項。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="custom-log-field-1"></a>自訂記錄欄位 1
**目的：** 判斷將指派給原始記錄檔中自訂記錄欄位的格式和內容。

此自訂欄位可讓您判斷哪些要求和回應標頭值會儲存於記錄檔中。

根據預設，自訂記錄欄位稱為「x-ec_custom-1」。 您可以從 [未經處理記錄設定] 頁面自訂此欄位的名稱。

用於指定要求和回應標頭的格式定義如下：

標頭類型|格式|範例
-|-|-
要求標頭|`%{[RequestHeader]()}[i]()` | %{Accept-Encoding}i <br/> {Referer}i <br/> %{Authorization}i
回應標頭|`%{[ResponseHeader]()}[o]()`| %{Age}o <br/> %{Content-Type}o <br/> %{Cookie}o

重要資訊：

- 自訂記錄欄位可包含標頭欄位和純文字的任意組合。
- 此欄位的有效字元如下：英數字元 (0-9、a-z 和 A-Z)、連字號、冒號、分號、單引號、逗號、句號、底線、等號、括弧、中括弧和空格。 只有在用於指定標頭欄位時，才能使用百分比符號和大括弧。
- 每個指定標頭欄位的拼字都必須符合所需的要求/回應標頭名稱。
- 如果您需要指定多個標頭，請使用分隔符號來表示每個標頭。 例如，您可以針對每個標頭使用縮寫：
    - AE: %{Accept-Encoding}i A: %{Authorization}i CT: %{Content-Type}o 

**預設值：** -

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="debug-cache-response-headers"></a>偵錯快取回應標頭
**目的：** 判斷回應是否包含 [X-EC-Debug 回應標頭](cdn-http-debug-headers.md)，其可在快取原則上提供所要求資產的相關資訊。

符合下列兩種情況時，將會在回應中包含偵錯快取回應標頭：

- 已在指定的要求上啟用 [偵錯快取回應標頭] 功能。
- 指定的要求會定義一組將包含於回應中的偵錯快取回應標頭。

偵錯快取回應標頭可能是藉由在要求中包含下列標頭和指定的指示詞來要求：

`X-EC-Debug: _&lt;Directive1&gt;_,_&lt;Directive2&gt;_,_&lt;DirectiveN&gt;_`

**範例：**

X-EC-Debug: x-ec-cache,x-ec-check-cacheable,x-ec-cache-key,x-ec-cache-state

值|結果
-|-
已啟用|偵錯快取回應標頭的要求將傳回包含 X-EC-Debug 標頭的回應。
已停用|X-EC-Debug 回應標頭將會從回應中排除。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="default-internal-max-age"></a>預設的內部最大壽命
**目的：** 決定 POP 向原始伺服器重新驗證快取之前的預設最大壽命間隔。 換句話說，就是 POP 檢查所快取資產是否符合原始伺服器上所儲存資產之前將經歷的時間長度。

重要資訊：

- 只有原始伺服器未在 `Cache-Control` 或 `Expires` 標頭中指派最大壽命指示時，此動作才會針對來自該原始伺服器的回應加以執行。
- 此動作將不會針對未被視為可快取的資產加以執行。
- 此動作不會影響瀏覽器向 POP 重新驗證快取。 這些類型的重新驗證取決於傳送到瀏覽器的 `Cache-Control` 或 `Expires` 標頭，而其可使用 [外部最大壽命] 功能來自訂。
- 此動作的結果對於回應標頭及針對您內容從 POP 傳回的內容並無顯著影響，但可能影響從 POP 傳送至原始伺服器的重新驗證流量多寡。
- 以下列方式設定此功能：
    - 選取可套用預設內部最大壽命的狀態碼。
    - 指定整數值，然後選取所需的時間單位 (例如，秒、分鐘、小時等)。 此值會定義預設的內部最大壽命間隔。

- 設定為「關閉」的時間單位，將針對未在其 `Cache-Control` 或 `Expires` 標頭中指派最大壽命指示的要求，指派 7 天的預設內部最大壽命間隔。

**預設值：** 7 天

#### <a name="compatibility"></a>相容性
由於追蹤快取設定的行為，因而導致此功能無法與下列比對條件產生關聯： 
- AS 號碼
- 用戶端 IP 位址
- Cookie 參數
- Cookie 參數 Regex
- 國家 (地區)
- 裝置
- 邊緣 Cname
- 轉介網域
- 要求標頭常值
- 要求標頭 Regex
- 要求標頭萬用字元
- 要求方法
- 要求配置
- URL 查詢常值
- URL 查詢 Regex
- URL 查詢萬用字元
- URL 查詢參數

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="deny-access-403"></a>拒絕存取 (403)
**目的**：判斷所有要求是否已遭拒且含有 [403 禁止] 回應。

值 | 結果
------|-------
已啟用| 導致所有滿足比對準則的要求遭拒，且具有 [403 禁止] 回應。
已停用| 還原預設行為。 預設行為是讓原始伺服器能夠判斷將傳回的回應類型。

**預設行為**：已停用

> [!TIP]
   > 此功能的可能用法之一是將它關聯至 [要求標頭] 比對條件，以封鎖存取使用內嵌連結連至您內容的 HTTP 推薦者。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="expires-header-treatment"></a>Expires 標頭處理
**目的：** 當 [外部最大壽命] 功能為作用中時，透過 POP 來控制 `Expires` 標頭的產生。

完成這類型組態的最簡單方式是將 [外部最大壽命] 和 [Expires 標頭處理] 功能放在同一個陳述式中。

值|結果
--|--
覆寫|確保將會進行下列動作：<br/>- 覆寫原始伺服器所產生的 `Expires` 標頭。<br/>- 將 [外部最大壽命] 功能所產生的 `Expires` 標頭新增至回應。
傳遞|確保永遠不會將 [外部最大壽命] 功能所產生的 `Expires` 標頭新增至回應。 <br/> 如果原始伺服器會產生 `Expires` 標頭，則它將會傳遞給使用者。 <br/>如果原始伺服器未產生 `Expires` 標頭，則此選項可能導致 `Expires` 標頭不會包含回應標頭。
如果遺失則加入| 如果 `Expires` 標頭不是接收自原始伺服器，則此選項會新增 [外部最大壽命] 功能所產生的 `Expires` 標頭。 若要確保會為所有資產指派 `Expires` 標頭，此選項非常有用。
移除| 確保標頭回應中不會包含 `Expires` 標頭。 如果已經指派 `Expires` 標頭，則會從標頭回應中移除它。

**預設行為：** 覆寫

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="external-max-age"></a>外部最大壽命
**目的：** 決定瀏覽器向 POP 重新驗證快取之前的最大壽命間隔。 換句話說，就是瀏覽器可向 POP 檢查是否有新版資產之前將經歷的時間長度。

啟用此功能會從 POP 產生 `Cache-Control: max-age` 和 `Expires` 標頭，並將它們傳送給 HTTP 用戶端。 根據預設，這些標頭將會覆寫原始伺服器所建立的標頭。 不過，可能會使用 [Cache-Control 標頭處理] 和 [Expires 標頭處理] 功能來改變此行為。

重要資訊：

- 此動作不會影響 POP 向原始伺服器重新驗證快取。 這些類型的重新驗證取決於接收自原始伺服器的 `Cache-Control` 和 `Expires` 標頭，並可使用 [預設的內部最大壽命] 和 [強制執行內部最大壽命] 功能來自訂。
- 指定整數值，然後選取所需的時間單位 (例如，秒、分鐘、小時等)，從而設定此功能。
- 將此功能設定為負數值時，會導致 POP 將過去利用每個回應設定的 `Cache-Control: no-cache` 和 `Expires` 時間傳送給瀏覽器。 儘管 HTTP 用戶端將不會快取回應，但此設定並不影響 POP 從原始伺服器快取回應的能力。
- 將時間單位設為「關閉」，即會停用此功能。 使用原始伺服器回應快取的 `Cache-Control` 和 `Expires` 標頭將會傳遞到瀏覽器。

**預設行為：** 關閉

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="follow-redirects"></a>遵循重新導向
**目的：** 判斷是否可將要求重新導向至定義於客戶原始伺服器所傳回之位置標頭中的主機名稱。

重要資訊：

- 您只能將要求重新導向至對應到相同平台的 Edge CNAME。

值|結果
-|-
已啟用|要求可重新導向。
已停用|要求將不會重新導向。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="force-internal-max-age"></a>強制執行內部最大壽命
**目的：** 決定 POP 向原始伺服器重新驗證快取之前的最大壽命間隔。 換句話說，就是 POP 可檢查所快取資產是否符合原始伺服器上所儲存資產之前將經歷的時間長度。

重要資訊：

- 此功能將會覆寫從原始伺服器產生之 `Cache-Control` 或 `Expires` 標頭中所定義的最大壽命間隔。
- 此功能不會影響瀏覽器向 POP 重新驗證快取。 這些類型的重新驗證取決於傳送至瀏覽器的 `Cache-Control` 或 `Expires` 標頭。
- 此功能對於由 POP 傳遞給要求者的回應並沒有顯著的影響。 不過，可能會影響從 POP 傳送給原始伺服器的重新驗證流量多寡。
- 以下列方式設定此功能：
    - 選取將套用內部最大壽命的狀態碼。
    - 指定整數值並選取所需的時間單位 (例如，秒、分鐘、小時等)。 此值會定義要求的最大壽命間隔。

- 將時間單位設為「關閉」，即會停用此功能。 系統不會將內部最大壽命間隔指派給要求的資產。 如果原始標頭未包含快取指示，則將根據 [預設內部最大壽命] 功能的作用中設定來快取資產。

**預設行為：** 關閉

#### <a name="compatibility"></a>相容性
由於追蹤快取設定的行為，因而導致此功能無法與下列比對條件產生關聯： 
- AS 號碼
- 用戶端 IP 位址
- Cookie 參數
- Cookie 參數 Regex
- 國家 (地區)
- 裝置
- 邊緣 Cname
- 轉介網域
- 要求標頭常值
- 要求標頭 Regex
- 要求標頭萬用字元
- 要求方法
- 要求配置
- URL 查詢常值
- URL 查詢 Regex
- URL 查詢萬用字元
- URL 查詢參數

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="h264-support-http-progressive-download"></a>H.264 支援 (HTTP 漸進式下載)
**目的：** 判斷可能用於串流處理內容的 H.264 檔案格式類型。

重要資訊：

- 在 [副檔名] 選項中，定義一組允許的 H.264 副檔名 (以空格分隔)。 [副檔名] 選項將會覆寫預設行為。 在設定此選項時包括這些副檔名，藉以保有 MP4 和 F4V 支援。 
- 指定每個副檔名 (例如 _.mp4_、_.f4v_) 時，請包含一個句點。

**預設行為︰** HTTP 漸進式下載預設支援 MP4 和 F4V 媒體。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="honor-no-cache-request"></a>接受 No-Cache 要求
**目的：** 判斷是否要將 HTTP 用戶端的 no-cache 要求轉送到原始伺服器。

當 HTTP 用戶端在 HTTP 要求中傳送 `Cache-Control: no-cache` 和/或 `Pragma: no-cache` 標題時，即會發生 no-cache 要求。

值|結果
--|--
已啟用|允許將 HTTP 用戶端的 no-cache 要求轉送給原始伺服器，而原始伺服器則會透過 POP 將回應標頭和主體傳回給 HTTP 用戶端。
已停用|還原預設行為。 預設行為是防止將 no-cache 要求轉送到原始伺服器。

針對所有生產環境的流量，強烈建議將此功能保留為其預設的停用狀態。 否則，將無法為原始伺服器提供防護，來防範可能在重新整理網頁時不小心觸發許多 no-cache 要求的使用者，或防範許多已編碼成要隨著每個視訊要求傳送 no-cache 標題的熱門媒體播放器。 不過，此功能非常適合用來套用至特定的非生產環境暫存或測試目錄，以便隨時從原始伺服器提取隨最新內容。

針對因此功能而得以轉送至原始伺服器的要求，報告的快取狀態為 `TCP_Client_Refresh_Miss`。 快取狀態報表 (可在 [核心報告] 模組中取得) 會依快取狀態提供統計資訊。 這份報告讓您能夠追蹤因此功能而轉送至原始伺服器的要求數目和百分比。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="ignore-origin-no-cache"></a>忽略原始的 No-Cache
**目的：** 判斷 CDN 是否將忽略原始伺服器所提供的特定指示詞：

- `Cache-Control: private`
- `Cache-Control: no-store`
- `Cache-Control: no-cache`
- `Pragma: no-cache`

重要資訊：

- 定義將忽略上述指示詞的狀態碼清單 (以空格分隔)，藉以設定此功能。
- 以下是此功能的有效狀態碼集合︰200、203、300、301、302、305、307、400、401、402、403、404、405、406、407、408、409、410、411、412、413、414、415、416、417、500、501、502、503、504 和 505。
- 藉由將其設為空白值來停用此功能。

**預設行為︰** 預設行為是採用上述指示詞。

#### <a name="compatibility"></a>相容性
由於追蹤快取設定的行為，因而導致此功能無法與下列比對條件產生關聯： 
- AS 號碼
- 用戶端 IP 位址
- Cookie 參數
- Cookie 參數 Regex
- 國家 (地區)
- 裝置
- 邊緣 Cname
- 轉介網域
- 要求標頭常值
- 要求標頭 Regex
- 要求標頭萬用字元
- 要求方法
- 要求配置
- URL 查詢常值
- URL 查詢 Regex
- URL 查詢萬用字元
- URL 查詢參數

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="ignore-unsatisfiable-ranges"></a>忽略無法滿足的範圍 
**目的：** 判斷在要求產生 [416 無法滿足的要求範圍] 狀態代碼時將傳回用戶端的要求。

根據預設，當 POP 無法滿足指定的位元組範圍要求，且未指定 If-Range 要求標頭欄位時，即會傳回此狀態碼。

值|結果
-|-
已啟用|防止 POP 使用「416 無法滿足的要求範圍」狀態碼來回應無效的位元組範圍要求。 伺服器將改為傳遞要求的資產，並將 [200 確定] 傳回用戶端。
已停用|還原預設行為。 預設行為是接受 [416 無法滿足的要求範圍] 狀態碼。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="internal-max-stale"></a>內部最大過時
**目的：** 控制當 POP 無法向原始伺服器重新驗證所快取的資產時，在過了標準到期時間多久之後仍可從 POP 提供所快取的資產。

一般來說，當資產的最大壽命時間到期時，POP 會向原始伺服器傳送重新驗證要求。 接著，原始伺服器會使用「304 未修改」來回應，為 POP 提供所快取資產的全新租用，或使用「200 確定」來為 POP 提供所快取資產的已更新版本。

如果 POP 在嘗試進行這類重新驗證時，無法與原始伺服器重新建立連線，則這個 [內部最大過時] 功能就會控制 POP 是否可繼續提供現已過時的資產，以及可以持續提供多久。

請注意，這個時間間隔是從資產的最大壽命到期時開始，而不是從發生失敗的重新驗證時開始。 因此，在未成功重新驗證的情況下可為資產提供服務的最長期間，是由 max-age 加上 max-stale 的組合所指定的時間長度。 例如，如果資產是在 9:00 快取，而 max-age 為 30 分鐘且 max-stale 為 15 分鐘，則在 9:44 進行的失敗重新驗證嘗試會導致使用者接收到過時的快取資產，而在 9:46 進行的失敗重新驗證嘗試則會導致使用者接收到 [504 閘道逾時]。

任何針對此功能設定的值都會由接收自原始伺服器的 `Cache-Control: must-revalidate` 或 `Cache-Control: proxy-revalidate` 標頭所取代。 如果在一開始快取資產時從原始伺服器收到這其中一個標頭，POP 就不會提供過時的已快取資產。 在這種情況下，如果在資產的最大壽命間隔到期時，POP 無法向原始伺服器重新驗證，POP 就會傳回「504 閘道逾時」錯誤。

重要資訊：

- 以下列方式設定此功能：
    - 選取將套用 max-stale 的狀態碼。
    - 指定整數值，然後選取所需的時間單位 (例如，秒、分鐘、小時等)。 此值可定義將套用的內部 max-stale。

- 將時間單位設為「關閉」，即會停用此功能。 系統將不會在快取的資產超過一般到期時間之後為其提供服務。

**預設行為：** 兩分鐘

#### <a name="compatibility"></a>相容性
由於追蹤快取設定的行為，因而導致此功能無法與下列比對條件產生關聯： 
- AS 號碼
- 用戶端 IP 位址
- Cookie 參數
- Cookie 參數 Regex
- 國家 (地區)
- 裝置
- 邊緣 Cname
- 轉介網域
- 要求標頭常值
- 要求標頭 Regex
- 要求標頭萬用字元
- 要求方法
- 要求配置
- URL 查詢常值
- URL 查詢 Regex
- URL 查詢萬用字元
- URL 查詢參數

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="log-query-string"></a>記錄查詢字串
**目的：** 判斷查詢字串是否將與 URL 一起儲存於存取記錄中。

值|結果
-|-
已啟用|在存取記錄中記錄 URL 時，允許儲存查詢字串。 如果 URL 未包含查詢字串，則此選項將不會有任何作用。
已停用|還原預設行為。 預設行為是在存取記錄中記錄 URL 時忽略查詢字串。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="maximum-keep-alive-requests"></a>Keep-Alive 要求的最大值
**目的：** 判斷在關閉 Keep-Alive 連線之前，適用於該連線的最大要求數目。

建議您不要將要求數目上限設為較低的值，這樣可能會導致效能變差。

重要資訊：

- 將此值指定為整數。
- 請勿在指定的值中包含逗號或句號。

**預設值︰** 10,000 個要求

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="modify-client-request-header"></a>修改用戶端要求標頭
**目的︰** 每個要求都包含一組說明其本身的要求標頭。 此功能可以：

- 附加或覆寫指派給要求標頭的值。 如果指定的要求標頭不存在，則此功能會將其加入至要求。
- 刪除要求的要求標頭。

要轉送到原始伺服器的要求將會反映此功能所做的變更。

您可以在要求標頭上執行下列其中一個動作：

選項|說明|範例
-|-|-
Append|指定的值會新增至現有要求標頭值的結尾。|**要求標頭值 (用戶端)：**<br/>Value1<br/>**要求標頭值 (規則引擎)：**<br/>Value2 <br/>**新的要求標頭值：** <br/>Value1Value2
覆寫|要求標頭值將會設定為指定的值。|**要求標頭值 (用戶端)：**<br/>Value1<br/>**要求標頭值 (規則引擎)：**<br/>Value2<br/>**新的要求標頭值：**<br/> Value2 <br/>
刪除|刪除指定的要求標頭。|**要求標頭值 (用戶端)：**<br/>Value1<br/>**修改用戶端要求標頭設定：**<br/>刪除指定的要求標頭。<br/>**結果︰**<br/>指定的要求標頭將不會轉送到原始伺服器。

重要資訊：

- 確定 [名稱] 選項中指定的值與所需的要求標頭完全相符。
- 基於識別標頭的目的，不會將大小寫納入考量。 例如，可以使用 `Cache-Control` 標頭名稱的下列任一個變化來識別它：
    - cache-control
    - CACHE-CONTROL
    - cachE-Control
- 在指定標頭名稱時，僅使用英數字元、連字號或底線。
- 刪除標頭將可防止 POP 將它轉送給原始伺服器。
- 以下為保留的標頭，且此功能無法加以修改：
    - forwarded
    - host
    - via
    - 警告
    - x-forwarded-for
    - 以「x-ec」開頭的所有標頭名稱都會保留。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="modify-client-response-header"></a>修改用戶端回應標頭
每個回應都包含一組說明其本身的回應標頭。 此功能可以：

- 附加或覆寫指派給回應標頭的值。 如果指定的回應標頭不存在，則此功能會將其新增至回應。
- 刪除回應的回應標頭。

預設會由原始伺服器和 POP 定義回應標頭值。

您可以在回應標頭上執行下列其中一個動作：

選項|說明|範例
-|-|-
Append|指定的值會新增至現有回應標頭值的結尾。|**回應標頭值 (用戶端)：**<br />Value1<br/>**回應標頭值 (規則引擎)：**<br/>Value2<br/>**新的回應標頭值：**<br/>Value1Value2
覆寫|回應標頭值將會設定為指定的值。|**回應標頭值 (用戶端)：**<br/>Value1<br/>**回應標頭值 (規則引擎)：**<br/>Value2 <br/>**新的回應標頭值：**<br/>Value2 <br/>
刪除|刪除指定的回應標頭。|**回應標頭值 (用戶端)：**<br/>Value1<br/>**修改用戶端回應標頭設定：**<br/>刪除指定的回應標頭。<br/>**結果︰**<br/>指定的回應標頭將不會轉送給要求者。

重要資訊：

- 確定 [名稱] 選項中指定的值與所需的回應標頭完全相符。 
- 基於識別標頭的目的，不會將大小寫納入考量。 例如，可以使用 `Cache-Control` 標頭名稱的下列任一個變化來識別它：
    - cache-control
    - CACHE-CONTROL
    - cachE-Control
- 刪除標頭可防止它被轉送給要求者。
- 以下為保留的標頭，且此功能無法加以修改：
    - accept-encoding
    - age
    - connection
    - content-encoding
    - content-length
    - content-range
    - 日期
    - 伺服器
    - trailer
    - transfer-encoding
    - 升級
    - vary
    - via
    - 警告
    - 以「x-ec」開頭的所有標頭名稱都會保留。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="partial-cache-sharing"></a>部分快取共用
**目的：** 判斷要求是否可以產生部分快取的內容。

接著可能會使用這個部分快取來滿足該內容的新要求，直到完全快取要求的內容為止。

值|結果
-|-
已啟用|要求可以產生部分快取的內容。
已停用|要求只能產生所要求內容的完整快取版本。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="prevalidate-cached-content"></a>預先驗證快取的內容
**目的：** 在快取內容的 TTL 到期之前，判斷其是否適用進行早期重新驗證。

定義在要求內容的 TTL 到期之前的時間長度，而要求的內容在這段期間將適用進行早期驗證。

重要資訊：

- 選取「關閉」做為時間單位，需要在快取內容的 TTL 到期之後進行重新驗證。 不應指定時間，且將會予以忽略。

**預設行為：** 關閉。 重新驗證可能只會在快取內容的 TTL 到期之後進行。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="proxy-special-headers"></a>Proxy 特定的標頭
**目的：** 定義一組將從 POP 轉送給原始伺服器的 [Verizon 特定 HTTP 要求標頭](cdn-verizon-http-headers.md)。

重要資訊：

- 此功能中定義的每個 CDN 特定要求標頭都會轉送到原始伺服器。 不會轉送已排除的標頭。
- 若要防止轉送 CDN 特定要求標頭，請從標頭清單欄位中以空格分隔的清單中將它移除。

下列 HTTP 標頭會包含在預設清單中：
- Via
- X-Forwarded-For
- X-Forwarded-Proto
- X-Host
- X-Midgress
- X-Gateway-List
- X-EC-Name
- Host

**預設行為︰** 所有 CDN 特定的要求標頭都將轉送到原始伺服器。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="refresh-zero-byte-cache-files"></a>重新整理零位元組的快取檔案
**目的：** 決定 POP 如何處理 HTTP 用戶端的 0 位元組快取資產要求。

有效值為：

值|結果
--|--
已啟用|導致 POP 從原始伺服器重新擷取資產。
已停用|還原預設行為。 預設行為是根據要求提供有效的快取資產。
不需要此功能即可進行正確的快取和內容傳遞，但此功能可能有助於解決這個問題。 例如，原始伺服器上的動態內容產生器可能會不小心導致將 0 位元組回應傳送給 POP。 POP 通常會快取這些類型的回應。 如果您知道對於這類內容而言，0 位元組的回應絕對不是有效的回應， 

則此功能可防止將這些類型的資產提供給用戶端。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="set-cacheable-status-codes"></a>設定可快取的狀態碼
**目的：** 定義一組可產生快取內容的狀態碼。

根據預設，只會針對 [200 確定] 回應啟用快取。

定義一組所需的狀態碼 (以空格分隔)。

重要資訊：

- 啟用 [忽略原始的 No-Cache] 功能。 如果未啟用此功能，可能不會快取非 [200 確定] 的回應。
- 以下是此功能的有效狀態碼集合︰203、300、301、302、305、307、400、401、402、403、404、405、406、407、408、409、410、411、412、413、414、415、416、417、500、501、502、503、504 和 505。
- 此功能無法用來針對產生 [200 確定] 狀態碼的回應停用快取。

**預設行為︰** 只會針對產生 [200 確定] 狀態碼的回應啟用快取。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="set-client-ip-custom-header"></a>設定用戶端 IP 自訂標頭
**目的︰** 依 IP 位址識別要求用戶端的自訂標頭加入至要求。

[標頭名稱] 選項會定義儲存用戶端 IP 位址的自訂要求標頭名稱。

此功能讓客戶原始伺服器能夠透過自訂要求標頭找出用戶端 IP 位址。 如果是從快取提供要求，則不會將用戶端的 IP 位址告知原始伺服器。 因此，建議將此功能使用於不會快取的資產。

請確定指定的標頭名稱不符合下列任一個名稱：

- 標準要求標頭名稱。 您可以在 [RFC 2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) 中找到標準標頭名稱清單。
- 保留的標頭名稱：
    - forwarded-for
    - host
    - vary
    - via
    - 警告
    - x-forwarded-for
    - 以「x-ec」開頭的所有標頭名稱都會保留。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="stale-content-delivery-on-error"></a>發生錯誤時傳遞過時的內容
**目的：** 判斷在快取重新驗證期間發生錯誤時，或者在接收到來自客戶原始伺服器的要求內容時，是否要傳遞到期的快取內容。

值|結果
-|-
已啟用|與原始伺服器連接期間發生錯誤時，即會將過時的內容提供給要求者。
已停用|系統會將原始伺服器的錯誤轉送給要求者。

**預設行為：** 已停用

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="stale-while-revalidate"></a>在重新驗證時過期
**目的：** 藉由允許 POP 在進行重新驗證時提供過時的內容給要求者來改善效能。

重要資訊：

- 此功能的行為會根據選取的時間單位而不同。
    - **時間單位︰** 指定的時間長度，然後選取允許過時內容傳遞的時間單位 (例如，秒、分鐘、小時等)。 這類型的設定可讓 CDN 根據下列公式，延長在要求驗證之前可傳遞內容的時間長度：**TTL** + **重新驗證時過時時間** 
    - **關閉：** 選取「關閉」，以便在可能提供過時內容的要求之前，要求重新驗證。
        - 請勿指定時間長度，因為它不適用且將會遭到忽略。

**預設行為：** 關閉。 重新驗證必須在可提供要求的內容之前進行。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="token-auth"></a>權杖驗證
**目的**：判斷是否要將權杖型驗證套用到要求。

如果啟用權杖型驗證，則只會接受提供加密權杖且符合該權杖所指定需求的要求。

用於加密和解密權杖值的加密金鑰，是根據 [權杖驗證] 頁面上的 [主要金鑰] 和 [備份金鑰] 選項來決定。 請記住，加密金鑰是特定平台的。

**預設行為：** 已停用。

這項功能的優先順序高於大部分功能 (URL 重寫功能除外)。

值 | 結果
------|---------
已啟用 | 使用權杖型驗證保護要求的內容。 只接受來自用戶端且可提供有效權杖並符合其需求的要求。 FTP 交易會從權杖型驗證中排除。
已停用| 還原預設行為。 預設行為是讓您的權杖型驗證組態能夠判斷要求是否將會受到保護。

#### <a name="compatibility"></a>相容性
請勿使用權杖驗證搭配一律比對條件。 

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="token-auth-denial-code"></a>權杖驗證拒絕代碼
**目的：** 判斷在要求因權杖型驗證而遭到拒絕時，將傳回給使用者的回應類型。

下表中列出可用的回應碼。

回應碼|回應名稱|說明
-------------|-------------|--------
301|已永久移動|此狀態碼會將未經授權的使用者重新導向至位置標頭中指定的 URL。
302|已找到|此狀態碼會將未經授權的使用者重新導向至位置標頭中指定的 URL。 此狀態碼是執行重新導向的業界標準方法。
307|暫時重新導向|此狀態碼會將未經授權的使用者重新導向至位置標頭中指定的 URL。
401|未經授權|將此狀態碼與 WWW 驗證回應標頭相結合，讓您能夠提示使用者進行驗證。
403|禁止|此訊息是標準的 [403 禁止] 狀態訊息，未經授權的使用者將會在嘗試存取受保護的內容時看見此訊息。
404|找不到檔案|此狀態碼表示 HTTP 用戶端能夠與伺服器通訊，但找不到要求的內容。

#### <a name="compatibility"></a>相容性
請勿使用權杖驗證拒絕程式碼搭配一律比對條件。 請改用 [管理] 入口網站中 [權杖驗證] 頁面的 [自訂拒絕處理] 區段。 如需詳細資訊，請參閱[使用權杖驗證來保護 Azure CDN 資產](cdn-token-auth.md)。

#### <a name="url-redirection"></a>URL 重新導向

將此功能設為傳回 3xx 狀態碼時，此功能支援將 URL 重新導向至使用者定義的 URL。 執行下列步驟，即可指定此使用者定義的 URL：

1. 針對 [權杖驗證拒絕代碼] 功能選取 3xx 回應碼。
2. 從 [選用標頭名稱] 選項中選取 [位置]。
3. 將 [選用標頭值] 選項設為所需的 URL。

如果未定義適用於 3xx 狀態碼的 URL，則會將適用於 3xx 狀態碼的標準回應頁面傳回給使用者。

URL 重新導向只適用於 3xx 回應碼。

[選用標頭值] 選項支援英數字元、引號和空格。

#### <a name="authentication"></a>驗證

此功能支援的功能可在針對權杖型驗證所保護的內容回應未經授權的要求時包含 WWW-Authenticate 標頭。 如果您的設定中已將 WWW-Authenticate 標頭設為「basic」，則會提示未經授權的使用者提供帳戶認證。

您可以藉由執行下列步驟來完成上述組態：

1. 針對 [權杖驗證拒絕代碼] 功能選取 [401] 做為回應碼。
2. 從 [選用標頭名稱] 選項中選取 [WWW-Authenticate]。
3. 將 [選用標頭值] 選項設為 [basic]。

WWW-Authenticate 標頭只適用於 401 回應碼。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="token-auth-ignore-url-case"></a>權杖驗證會忽略 URL 的大小寫
**目的：** 判斷透過權杖型驗證所做的 URL 比較是否區分大小寫。

受此功能影響的參數如下：

- ec_url_allow
- ec_ref_allow
- ec_ref_deny

有效值為：

值|結果
---|----
已啟用|導致 POP 在比較「權杖型驗證」參數的 URL 時忽略大小寫。
已停用|還原預設行為。 預設行為是在進行權杖型驗證的 URL 比較時會區分大小寫。

**預設行為：** 已停用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="token-auth-parameter"></a>權杖驗證參數
**目的：** 判斷是否應將權杖型驗證查詢字串參數重新命名。

重要資訊：

- [值] 選項會定義可能要透過其指定權杖的查詢字串參數名稱。
- [值] 選項不能設定為 [ec_token]。
- 確認 [值] 選項中定義的名稱僅包含有效的 URL 字元。

值|結果
----|----
已啟用|[值] 選項會定義應透過其定義權杖的查詢字串參數名稱。
已停用|您可以指定權杖做為要求 URL 中未定義的查詢字串參數。

**預設行為：** 已停用。 您可以指定權杖做為要求 URL 中未定義的查詢字串參數。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="url-redirect"></a>[URL 重新導向]
**目的：** 透過位置標頭來將要求重新導向。

此功能的組態必須設定下列選項：

選項|說明
-|-
代碼|選取將傳回給要求者的回應碼。
來源與模式| 這些設定會定義要求 URI 模式，此模式會識別可能要重新導向的要求類型。 只會重新導向 URL 符合下列這兩個準則的要求： <br/> <br/> **來源 (或內容存取點)：** 選取識別原始伺服器的相對路徑。 此路徑是 /XXXX/ 區段和您的端點名稱。 <br/><br/> **來源 (模式)︰** 必須定義依相對路徑識別要求的模式。 這個規則運算式模式必須定義一個路徑，該路徑會在先前選取的內容存取點 (請參閱上述內容) 之後直接啟動。 <br/> - 確定先前定義的要求 URI 準則 (亦即，[來源與模式]) 不會與針對此功能所定義的任何比對條件相衝突。 <br/> - 指定模式；如果您是使用空白值作為模式，所有字串都會相符。
目的地| 定義要將上述要求重新導向至其中的 URL。 <br/><br/> 使用下列方式來動態建構此 URL： <br/> - 規則運算式模式 <br/>- [HTTP 變數](cdn-http-variables.md) <br/><br/> 使用 $_n_，將擷取自來源模式的值替代至目的地模式，其中_n_ 可依擷取的順序來識別值。 例如，$1 表示擷取自來源模式的第一個值，而 $2 代表第二個值。 <br/> 
強烈建議使用絕對 URL。 使用相對 URL 可能會將 CDN URL 重新導向至不正確的路徑。

**範例案例**

此範例示範如何將解析為下列基底 CDN URL 的 Edge CNAME URL 重新導向：http:\//marketing.azureedge.net/brochures

符合資格的要求將會被重新導向至此基底邊緣 CNAME URL：http:\//cdn.mydomain.com/resources

此 URL 重新導向可透過下列設定來達成：![URL 重新導向](./media/cdn-rules-engine-reference/cdn-rules-engine-redirect.png)

**重點︰**

- [URL 重新導向] 功能會定義將重新導向的要求 URL。 如此一來，就不需要額外的比對條件。 雖然將比對條件定義為 [一律]，但只會重新導向指向 [marketing] 客戶原始伺服器上 [brochures] 資料夾的要求。 
- 所有相符的要求都將重新導向到 [目的地] 選項中所定義的 Edge CNAME URL。 
    - 範例案例 1： 
        - 範例要求 (CDN URL)：http:\//marketing.azureedge.net/brochures/widgets.pdf 
        - 要求 URL (重新導向之後)：http:\//cdn.mydomain.com/resources/widgets.pdf  
    - 範例案例 2： 
        - 範例要求 (邊緣 CNAME URL)：http:\//marketing.mydomain.com/brochures/widgets.pdf 
        - 要求 URL (重新導向之後)：http:\//cdn.mydomain.com/resources/widgets.pdf
    - 範例案例 3： 
        - 範例要求 (邊緣 CNAME URL)：http:\//brochures.mydomain.com/campaignA/final/productC.ppt 
        - 要求 URL (重新導向之後)：http:\//cdn.mydomain.com/resources/campaignA/final/productC.ppt  
- 要求配置 (%{scheme}) 變數會運用在 [目的地] 選項，以確保重新導向之後要求的配置維持不變。
- 擷取自要求的 URL 區段會透過「$1」附加到新的 URL。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="url-rewrite"></a>URL 重寫
**目的：** 重寫要求 URL。

重要資訊：

- 此功能的組態必須設定下列選項：

選項|說明
-|-
 來源與模式 | 這些設定會定義要求 URI 模式，此模式會識別可能要重寫的要求類型。 只會重寫 URL 符合下列這兩個準則的要求： <br/><br/>  - **來源 (或內容存取點)：** 選取識別原始伺服器的相對路徑。 此路徑是 /XXXX/ 區段和您的端點名稱。 <br/><br/> - **來源 (模式)︰** 必須定義依相對路徑識別要求的模式。 這個規則運算式模式必須定義一個路徑，該路徑會在先前選取的內容存取點 (請參閱上述內容) 之後直接啟動。 <br/> - 確認先前定義的要求 URI 準則 (亦即，[來源與模式]) 不會與針對此功能所定義的任何比對條件相衝突。 指定模式；如果您是使用空白值作為模式，所有字串都會相符。 
 目的地  |使用下列方式來定義要將上述要求重寫至其中的相對 URL： <br/>    1.選取可識別原始伺服器的內容存取點。 <br/>    2.使用下列方式來定義相對路徑： <br/>        - 規則運算式模式 <br/>        - [HTTP 變數](cdn-http-variables.md) <br/> <br/> 使用 $_n_，將擷取自來源模式的值替代至目的地模式，其中_n_ 可依擷取的順序來識別值。 例如，$1 表示擷取自來源模式的第一個值，而 $2 代表第二個值。 
 此功能可讓 POP 不需執行傳統的重新導向就能重寫 URL。 也就是，如果收到重寫 URL 的要求，要求者會收到相同的回應碼。

**範例案例 1**

此範例示範如何重新導向解析為下列基底 CDN URL 的邊緣 CNAME URL：http:\//marketing.azureedge.net/brochures/

符合資格的要求將會重新導向至此基底邊緣 CNAME URL：http:\//MyOrigin.azureedge.net/resources/

此 URL 重新導向可透過下列設定來達成：![URL 重新導向](./media/cdn-rules-engine-reference/cdn-rules-engine-rewrite.png)

**範例案例 2**

此範例示範如何使用規則運算式，將邊緣 CNAME URL 從「大寫」重新導向到小寫。

此 URL 重新導向可透過下列設定來達成：![URL 重新導向](./media/cdn-rules-engine-reference/cdn-rules-engine-to-lowercase.png)


**重點︰**

- [URL 重寫] 功能會定義將要重寫的要求 URL。 如此一來，就不需要額外的比對條件。 雖然將比對條件定義為 [一律]，但只會重寫指向 [marketing] 客戶原始伺服器上 [brochures] 資料夾的要求。

- 擷取自要求的 URL 區段會透過「$1」附加到新的 URL。

#### <a name="compatibility"></a>相容性
此功能包括必須符合才能套用到要求的比對準則。 為了防止設定衝突的比對準則，此功能與下列比對條件不相容：

- AS 號碼
- CDN 原點
- 用戶端 IP 位址
- 客戶原點
- 要求配置
- URL 路徑目錄
- URL 路徑的副檔名
- URL 路徑的檔案名稱
- URL 路徑常值
- URL 路徑 Regex
- URL 路徑萬用字元
- URL 查詢常值
- URL 查詢參數
- URL 查詢 Regex
- URL 查詢萬用字元

[回到頁首](#azure-cdn-rules-engine-features)

</br>

---
### <a name="user-variable"></a>User 變數
**目的：** 僅供內部使用。

[回到頁首](#azure-cdn-rules-engine-features)

</br>

## <a name="next-steps"></a>後續步驟
* [規則引擎參考](cdn-rules-engine-reference.md)
* [規則引擎條件運算式](cdn-rules-engine-reference-conditional-expressions.md)
* [規則引擎比對條件](cdn-rules-engine-reference-match-conditions.md)
* [使用規則引擎覆寫 HTTP 行為](cdn-rules-engine.md)
* [Azure CDN 概觀](cdn-overview.md)
