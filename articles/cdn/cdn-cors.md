---
title: 搭配 CORS 使用 Azure CDN | Microsoft Docs
description: 了解如何使用 Azure 內容傳遞網路 (CDN) 搭配跨來源資源共用 (CORS)。
services: cdn
documentationcenter: ''
author: zhangmanling
manager: erikre
editor: ''
ms.assetid: 86740a96-4269-4060-aba3-a69f00e6f14e
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: f9429e88525e27c0b6bad29d1927d53d05dfbcc8
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33765359"
---
# <a name="using-azure-cdn-with-cors"></a>搭配 CORS 使用 Azure CDN
## <a name="what-is-cors"></a>CORS 是什麼？
CORS (Cross Origin Resource Sharing；跨來源資源共用) 是一項 HTTP 功能，可讓在某個網域下執行的 Web 應用程式存取其他網域中的資源。 為了減少跨網站指令碼攻擊的可能性，所有現代網頁瀏覽器都實作稱為 [Same-Origin Policy (同源原則)](http://www.w3.org/Security/wiki/Same_Origin_Policy)的安全性限制。  這樣可以防止網頁呼叫其他網域中的 API。  CORS 提供安全的方式來允許從一個來源 (來源網域) 呼叫其他來源中的 API。

## <a name="how-it-works"></a>運作方式
CORS 要求有兩種類型，*簡單要求*和*複雜要求*。

### <a name="for-simple-requests"></a>對於簡單要求︰

1. 瀏覽器會傳送有額外**來源** HTTP 要求標頭的 CORS 要求。 此標頭的值是提供父頁面的來源，是以*通訊協定*、*網域*和*連接埠*的組合來定義的。  當來自 https://www.contoso.com 的頁面嘗試存取位於 fabrikam.com 來源的使用者資料時，會將以下要求標頭傳送到 fabrikam.com：

   `Origin: https://www.contoso.com`

2. 該伺服器的回應可能是下列其中一項：

   * 回應中的 **Access-Control-Allow-Origin** 標頭指出所允許的來源網站。 例如︰

     `Access-Control-Allow-Origin: https://www.contoso.com`

   * HTTP 錯誤碼 (例如 403)；如果伺服器在檢查 Origin 標頭之後，不允許跨來源要求，則會出現此錯誤碼

   * **Access-Control-Allow-Origin** 標頭包含允許所有來源的萬用字元：

     `Access-Control-Allow-Origin: *`

### <a name="for-complex-requests"></a>對於複雜要求︰

複雜要求指的是瀏覽器在傳送實際 CORS 要求之前，必須先傳送*預檢要求* (也就是預備探查) 的 CORS 要求。 如果原始 CORS 可以處理，而且是對相同 URL 的 `OPTIONS` 要求，那麼預檢要求會要求伺服器權限。

> [!TIP]
> 如需 CORS 流程和常見陷阱的詳細資訊，請參閱 [REST API 的 CORS 指南 (英文)](https://www.moesif.com/blog/technical/cors/Authoritative-Guide-to-CORS-Cross-Origin-Resource-Sharing-for-REST-APIs/)。
>
>

## <a name="wildcard-or-single-origin-scenarios"></a>萬用字元或單一來源的狀況
當 **Access-Control-Allow-Origin** 標頭設為萬用字元 (\*) 或單一來源時，Azure CDN 上的 CORS 不需要額外設定就會自動生效。  CDN 會快取第一個回應，且後續的要求將使用相同的標頭。

如果在您的來源上設定 CORS 之前已經向 CDN 發出要求，您必須清除您端點內容上的內容，以重新載入包含 **Access-Control-Allow-Origin** 標頭的內容。

## <a name="multiple-origin-scenarios"></a>多重來源的狀況
如果您要針對 CORS 允許使用特定清單中的來源，則會稍微複雜一點。 問題會在 CDN 快取第一個 CORS 來源的 **Access-Control-Allow-Origin** 標頭時發生。  當其他 CORS 來源發出後續要求時，CDN 會提供已快取的 **Access-Control-Allow-Origin** 標頭，而此標頭會不相符。  有幾個方法可以修正此問題。

### <a name="azure-cdn-premium-from-verizon"></a>來自 Verizon 的 Azure CDN 進階
完成此作業的最佳方法是使用「來自 Verizon 的 Azure CDN 進階」 ，它會公開一些進階功能。 

您必須 [建立規則](cdn-rules-engine.md) 來檢查要求的 **Origin** 標頭。  如果是有效的來源，您的規則將使用要求中提供的來源設定 **Access-Control-Allow-Origin** 標頭。  如果 **Origin** 中指定的來源是不允許的，您的規則應會忽略 **Access-Control-Allow-Origin** 標頭，這將導致瀏覽器拒絕要求。 

使用規則引擎來這樣做的方法有兩種。 在這兩種情況下，會忽略來自檔案原始伺服器的 **Access-Control-Allow-Origin** 標頭，而允許的 CORS 來源會完全由 CDN 的規則引擎來管理。

#### <a name="one-regular-expression-with-all-valid-origins"></a>包含所有有效來源的規則運算式
在此情況下，您會建立包含您要允許使用之所有來源的規則運算式。 

    https?:\/\/(www\.contoso\.com|contoso\.com|www\.microsoft\.com|microsoft.com\.com)$

> [!TIP]
> **來自 Verizon 的進階 Azure CDN** 會使用 [Perl 相容的規則運算式](http://pcre.org/)作為其規則運算式的引擎。  您可以使用像 [Regular Expressions 101](https://regex101.com/) 這樣的工具來驗證您的規則運算式。  請注意，規則運算式中的「/」字元是有效的，不需要逸出，不過將該字元逸出被視為是最佳做法，且某些規則運算式驗證程式也預期將它逸出。
> 
> 

如果規則運算式相符，您的規則會將來自來源的 **Access-Control-Allow-Origin** 標頭 (如果有) 取代為傳送該要求的來源。  您也可以加入額外的 CORS 標頭，例如 **Access-Control-Allow-Methods**。

![包含規則運算式的規則範例](./media/cdn-cors/cdn-cors-regex.png)

#### <a name="request-header-rule-for-each-origin"></a>針對每個來源要求標頭規則
除了規則運算式之外，您也可以使用**要求標頭萬用字元**[比對條件](https://msdn.microsoft.com/library/mt757336.aspx#Anchor_1)，針對每個要允許的來源建立個別規則。 如同規則運算式方法一樣，也是僅由規則引擎設定 CORS 標頭。 

![不含規則運算式的規則範例](./media/cdn-cors/cdn-cors-no-regex.png)

> [!TIP]
> 在上述範例中，萬用字元 (*) 的作用是告訴規則引擎 HTTP 和 HTTPS 兩者都要比對。
> 
> 

### <a name="azure-cdn-standard-profiles"></a>標準 Azure CDN 設定檔
在標準 Azure CDN 設定檔 (**來自 Microsoft 的標準 Azure CDN**、**來自 Akamai 的標準 Azure CDN** 及**來自 Verizon 的標準 Azure CDN**) 上，在不使用萬用字元來源情況下允許多重來源的唯一機制是使用[查詢字串快取](cdn-query-string.md)。 啟用 CDN 端點的查詢字串設定，然後針對來自每個允許之網域的查詢使用唯一的查詢字串。 這麼做將使 CDN 針對每個唯一的查詢字串快取個別物件。 但是這不是最理想的方法，因為它會導致在 CDN 上快取同一個檔案的多個複本。  

