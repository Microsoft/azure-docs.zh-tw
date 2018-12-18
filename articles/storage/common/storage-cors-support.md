---
title: 跨原始資源共用 (CORS) 支援 | Microsoft Docs
description: 了解如何啟用 Microsoft Azure 儲存體服務的 CORS 支援。
services: storage
author: cbrooksmsft
ms.service: storage
ms.devlang: dotnet
ms.topic: article
ms.date: 2/22/2017
ms.author: cbrooks
ms.component: common
ms.openlocfilehash: fd5df50128885f6a96e68c8ad46204bc21d80264
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39529707"
---
# <a name="cross-origin-resource-sharing-cors-support-for-the-azure-storage-services"></a>Azure 儲存體服務的跨原始資源共用 (CORS) 支援
從 2013-08-15 版本開始，Azure 儲存體服務就針對 Blob、資料表、佇列及檔案服務提供跨原始資源共用 (CORS) 的支援。 CORS 是一項 HTTP 功能，可讓 Web 應用程式在某個網域下執行，以存取其他網域中的資源。 網頁瀏覽器會實作稱為 [相同原始原則](http://www.w3.org/Security/wiki/Same_Origin_Policy) 的安全性限制，它可防止網頁呼叫不同網域中的 API；CORS 則提供了一個安全的方式，可讓一個網域 (原始網域) 能夠呼叫其他網域中的 API。 如需 CORS 的詳細資訊，請參閱 [CORS 規格](http://www.w3.org/TR/cors/) 。

您可以透過呼叫[設定 Blob 服務屬性](https://msdn.microsoft.com/library/hh452235.aspx)、[設定佇列服務屬性](https://msdn.microsoft.com/library/hh452232.aspx)和[設定表格服務屬性](https://msdn.microsoft.com/library/hh452240.aspx)，個別為每個儲存體服務設定 CORS 規則。 一旦設定服務的 CORS 規則之後，將評估從不同網域對服務所提出的適當授權要求，以判斷是否可根據您指定的規則來允許它。

> [!NOTE]
> 請注意，CORS 不是一種驗證機制。 啟用 CORS 時，對儲存體資源所提出的任何要求都必須具有適當的驗證簽章，或者必須對公用資源提出。
> 
> 

## <a name="understanding-cors-requests"></a>了解 CORS 要求
來自原始網域的 CORS 要求可能包含兩個不同要求：

* 預檢要求，其會查詢服務所加諸的 CORS 限制。 除非要求方法是 [簡單的方法](http://www.w3.org/TR/cors/)(意即 GET、HEAD 或 POST)，否則必須執行預檢要求。
* 對所需資源提出的實際要求。

### <a name="preflight-request"></a>預檢要求
預檢要求會查詢由帳戶擁有者為儲存體服務所建立的 CORS 限制。 Web 瀏覽器 (或其他使用者代理程式) 會傳送包含要求標頭、方法和原始網域的 OPTIONS 要求。 儲存體服務會根據一組預先設定的 CORS 規則來評估預定作業，這些規則會指定需在對儲存體資源提出的實際要求上指定哪些原始網域、要求方法和要求標頭。

如果已啟用服務的 CORS 且 CORS 規則符合預檢要求，服務會以狀態碼 200 (確定) 回應，並在回應中包含必要的存取控制標頭。

如果未啟用服務的 CORS，或是 CORS 規則不符合預檢要求，服務會回應狀態碼 403 (禁止)。

如果 OPTIONS 要求未包含必要的 CORS 標頭 (Origin 和 Access-Control-Request-Method 標頭)，服務會回應狀態碼 400 (不正確的要求)。

請注意，預檢要求是針對服務 (Blob、佇列和資料表) 進行評估，而不是針對要求的資源。 帳戶擁有者必須已啟用 CORS 做為帳戶服務屬性一部分，才能讓要求成功。

### <a name="actual-request"></a>實際要求
一旦接受預檢要求並傳回回應之後，瀏覽器便會對儲存體資源發送實際要求。 如果預檢要求遭到拒絕，瀏覽器將立即拒絕實際要求。

實際要求會被視為對儲存體服務提出的一般要求。 Origin 標頭的目前狀態表示要求是 CORS 要求，而服務將檢查 CORS 比對規則。 如果找到相符項目，就會將 Access-Control 標頭加入回應並傳送回用戶端。 如果找不到相符項目，就不會傳回 CORS Access-Control 標頭。

## <a name="enabling-cors-for-the-azure-storage-services"></a>針對 Azure 儲存體服務啟用 CORS
CORS 規則是設定於服務層級，因此您需要針對每個服務 (Blob、佇列和資料表) 個別啟用或停用 CORS。 根據預設，每個服務都會停用 CORS。 若要啟用 CORS，您必須使用 2013-08-15 版本或更新版本來設定適當的服務屬性，並將 CORS 規則加入服務屬性。 如需如何針對服務啟用或停用 CORS，以及如何設定 CORS 規則的詳細資訊，請參閱[設定 Blob 服務屬性](https://msdn.microsoft.com/library/hh452235.aspx)、[設定佇列服務屬性](https://msdn.microsoft.com/library/hh452232.aspx)和[設定表格服務屬性](https://msdn.microsoft.com/library/hh452240.aspx)。

以下範例是透過設定服務屬性作業指定的單一 CORS 規則：

```xml
<Cors>    
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com, http://www.fabrikam.com</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <AllowedHeaders>x-ms-meta-data*,x-ms-meta-target*,x-ms-meta-abc</AllowedHeaders>
        <ExposedHeaders>x-ms-meta-*</ExposedHeaders>
        <MaxAgeInSeconds>200</MaxAgeInSeconds>
    </CorsRule>
<Cors>
```

CORS 規則中包含的每個項目敘述如下：

* **AllowedOrigins**：允許透過 CORS 對儲存體服務發出要求的原始網域。 原始網域是要求的來源網域。 請注意，原始網域的大小寫必須與使用者代理程式傳送至服務的原始網域完全相符。 您也可以使用萬用字元 '*'，允許所有原始網域透過 CORS 提出要求。 在上述範例中，[http://www.contoso.com](http://www.contoso.com) 和 [http://www.fabrikam.com](http://www.fabrikam.com) 網域可以使用 CORS 對服務發出要求。
* **AllowedMethods**：原始網域可能針對 CORS 要求使用的方法 (HTTP 要求動詞命令)。 在上述範例中，只允許 PUT 和 GET 要求。
* **AllowedHeaders**：原始網域可在 CORS 要求上指定的要求標頭。 在上述範例中，允許以 x-ms-meta-data、x-ms-meta-target 及 x-ms-meta-abc 開始的所有中繼資料標頭。 請注意，萬用字元 '*' 表示允許任何以指定前置詞開頭的標頭。
* **ExposedHeaders**：可能包含在對 CORS 要求的回應中傳送，而且由瀏覽器對要求簽發者公開的回應標頭。 在上述範例中，會指示瀏覽器公開任何以 x-ms-meta 開頭的標頭。
* **MaxAgeInSeconds**：瀏覽器應快取預檢 OPTIONS 要求的時間量上限。

Azure 儲存體服務支援為 **AllowedHeaders** 和 **ExposedHeaders** 元素指定有前置詞的標頭。 若要允許標頭的類別，您可以指定該類別的一般前置詞。 例如，指定 *x-ms-meta** 做為有前置詞的標頭會建立一項規則，可比對開頭為 x-ms-meta 的所有標頭。

下列限制適用於 CORS 規則：

* 您最多可以為每個儲存體服務 (Blob、資料表和佇列) 指定五個 CORS 規則。
* 在要求上設定的所有 CORS 規則的大小上限 (不包括 XML 標記) 不應超過 2 KB。
* 允許的標頭、公開的標頭或允許的原始網域的長度不可超過 256 個字元。
* 允許的標頭和公開的標頭可以是：
  * 常值標頭，此標頭是確切的標頭名稱，例如 **x-ms-meta-processed**。 最多可在要求上指定 64 個常值標頭。
  * 有前置詞的標頭，當中包含標頭的前置詞，例如 **x-ms-meta-data**\*。 以這種方式指定前置詞，可允許或公開以指定前置詞開頭的任何標頭。 最多可在要求上指定兩個有前置詞的標頭。
* 在 **AllowedMethods** 元素中指定的方法 (或 HTTP 動詞命令) 必須符合 Azure 儲存體服務 API 所支援的方法。 支援的方法為 DELETE、GET、HEAD、MERGE、POST、OPTIONS 及 PUT。

## <a name="understanding-cors-rule-evaluation-logic"></a>了解 CORS 規則評估邏輯
當儲存體服務收到預檢或實際要求時，它會透過適當的設定服務屬性作業，根據您已為服務建立的 CORS 規則來評估該要求。 CORS 規則會以設定服務屬性作業的要求主體中所設定的順序進行評估。

CORS 規則的評估，如下所示：

1. 首先，會針對為 **AllowedOrigins** 項目列出的網域檢查要求的原始網域。 如果清單中包含原始網域，或是使用了萬用字元 '*' 來允許所有網域，則會繼續評估規則。 如果未包含原始網域，則要求會失敗。
2. 接下來，會針對 **AllowedMethods** 元素中列出的方法檢查要求的方法 (或 HTTP 動詞命令)。 如果方法包含於清單中，就會繼續評估規則；如果沒有，則要求會失敗。
3. 如果要求符合其原始網域及其方法中的規則，就會選取該規則來處理要求，且不會評估任何其他規則。 不過，需要對 **AllowedHeaders** 元素中列出的標頭檢查要求上指定的所有標頭，該要求才會成功。 如果傳送的標頭與允許的標頭不符，則要求會失敗。

因為這些規則會依照它們在要求主體中出現的順序來處理，所以，最佳做法建議您先在清單中指定與原始網域有關的最嚴格規則，以便先評估這些規則。 將較不嚴格的規則指定於清單結尾 - 例如，允許所有原始網域的規則。

### <a name="example--cors-rules-evaluation"></a>範例 - CORS 規則評估
下列範例示範作業的部分要求主體，以設定儲存體服務的 CORS 規則。 如需建構要求的詳細資訊，請參閱[設定 Blob 服務屬性](https://msdn.microsoft.com/library/hh452235.aspx)、[設定佇列服務屬性](https://msdn.microsoft.com/library/hh452232.aspx)和[設定表格服務屬性](https://msdn.microsoft.com/library/hh452240.aspx)。

```xml
<Cors>
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com</AllowedOrigins>
        <AllowedMethods>PUT,HEAD</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-blob-content-type, x-ms-blob-content-disposition</AllowedHeaders>
    </CorsRule>
    <CorsRule>
        <AllowedOrigins>*</AllowedOrigins>
        <AllowedMethods>PUT,GET</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-blob-content-type, x-ms-blob-content-disposition</AllowedHeaders>
    </CorsRule>
    <CorsRule>
        <AllowedOrigins>http://www.contoso.com</AllowedOrigins>
        <AllowedMethods>GET</AllowedMethods>
        <MaxAgeInSeconds>5</MaxAgeInSeconds>
        <ExposedHeaders>x-ms-*</ExposedHeaders>
        <AllowedHeaders>x-ms-client-request-id</AllowedHeaders>
    </CorsRule>
</Cors>
```

接下來，請考慮下列 CORS 要求：

| 要求 |  |  | Response |  |
| --- | --- | --- | --- | --- |
| **方法** |**原始** |**要求標頭** |**規則相符** |**結果** |
| **PUT** |http://www.contoso.com |x-ms-blob-content-type |第一個規則 |成功 |
| **GET** |http://www.contoso.com |x-ms-blob-content-type |第二個規則 |成功 |
| **GET** |http://www.contoso.com |x-ms-client-request-id |第二個規則 |失敗 |

第一個要求符合第一個規則 (原始網域符合允許的原始網域、方法符合允許的方法，而且標頭符合允許的標頭)，因而成功。

第二個要求不符合第一個規則，因為此方法不符合允許的方法。 不過，它與第二個規則相符，因而成功。

第三個要求符合其原始網域和方法中的第二個規則，所以不會進一步評估任何規則。 但是，第二個規則不允許 *x-ms-client-request-id header* ，因此，儘管第三個規則的語意已允許它成功，要求仍會失敗。

> [!NOTE]
> 雖然相較於之前較嚴格的規則，此範例示範的規則已較不嚴格，但一般最佳做法是先列出最嚴格的規則。
> 
> 

## <a name="understanding-how-the-vary-header-is-set"></a>了解如何設定 Vary 標頭
*Vary* 標頭是標準的 HTTP/1.1 標頭，其中包含一組要求標頭欄位，可通知瀏覽器或使用者代理程式有關由伺服器選取用來處理要求的準則。 *Vary* 標頭主要是在 Proxy、瀏覽器及 CDN 進行快取時所使用，它們會用此標頭來判斷快取回應的方式。 如需詳細資訊，請參閱 [Vary 標頭](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)的規格。

當瀏覽器或其他使用者代理程式快取來自 CORS 要求的回應時，會將原始網域快取為允許的原始網域。 當第二個網域在快取作用中時發出相同的儲存體資源要求，使用者代理程式便會擷取快取的原始網域。 第二個網域不符合快取的網域，因此要求失敗 (它本來可以成功)。 在某些情況下，Azure 儲存體會將 Vary 標頭設定為 **Origin** ，在要求網域與快取的原始網域不同時，指示使用者代理程式將後續的 CORS 要求傳送至服務。

在下列情況下，Azure 儲存體會將實際 GET/HEAD 要求的 *Vary* 標頭設為 **Origin** ：

* 當要求的原始網域完全符合 CORS 規則所定義的允許來源。 若要完全符合，CORS 規則不能包含萬用字元 ' * '。
* 沒有比對要求原始網域的規則，但會啟用儲存體服務的 CORS。

在 GET/HEAD 要求符合允許所有原始網域的 CORS 規則的情況下，回應會指出允許所有原始網域，而且當快取作用中時，使用者代理程式快取將會允許來自任何原始網域的後續要求。

請注意，針對使用非 GET/HEAD 方法的要求，儲存體服務將不會設定 Vary 標頭，因為使用者代理程式不會快取這些方法的回應。

下表指出 Azure 儲存體如何根據先前所述的案例來回應 GET/HEAD 要求：

| 要求 | 帳戶設定和規則評估的結果 |  |  | Response |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **要求上存在的 Origin 標頭** |**針對此服務指定的 CORS 規則** |**有允許所有原始網域的比對規則存在 (*)** |**有完全符合原始網域的比對規則存在** |**回應包含已設為 Origin 的 Vary 標頭** |**回應包含 Access-Control-Allowed-Origin："*"** |**回應包含 Access-Control-Exposed-Headers** |
| 否 |否 |否 |否 |否 |否 |否 |
| 否 |是 |否 |否 |是 |否 |否 |
| 否 |yes |yes |否 |否 |yes |yes |
| yes |否 |否 |否 |否 |否 |否 |
| yes |yes |否 |yes |yes |否 |yes |
| yes |yes |否 |否 |是 |否 |否 |
| yes |yes |yes |否 |否 |yes |是 |

## <a name="billing-for-cors-requests"></a>CORS 要求的計費方式
如果您已針對帳戶的所有儲存體服務啟用 CORS (透過呼叫[設定 Blob 服務屬性](https://msdn.microsoft.com/library/hh452235.aspx)、[設定佇列服務屬性](https://msdn.microsoft.com/library/hh452232.aspx)或[設定表格服務屬性](https://msdn.microsoft.com/library/hh452240.aspx))，則成功的預檢要求就會列入計費。 為了將費用降至最低，請考慮將 CORS 規則中的 **MaxAgeInSeconds** 元素設為較大的值，讓使用者代理程式能夠快取要求。

未成功的預檢要求將不會列入計費。

## <a name="next-steps"></a>後續步驟
[設定 Blob 服務屬性](https://msdn.microsoft.com/library/hh452235.aspx)

[設定佇列服務屬性](https://msdn.microsoft.com/library/hh452232.aspx)

[設定資料表服務屬性](https://msdn.microsoft.com/library/hh452240.aspx)

[W3C 跨原始資源共用規格](http://www.w3.org/TR/cors/)

