---
title: Azure 媒體服務中的串流端點 | Microsoft Docs
description: 本文解釋串流端點是什麼，以及 Azure 媒體服務用它們來做什麼。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 5ba80df26731f9687ec4518d7f12f3c2e3cb0665
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49379173"
---
# <a name="streaming-endpoints"></a>串流端點

在「Microsoft Azure 媒體服務」(AMS) 中，[串流端點](https://docs.microsoft.com/rest/api/media/streamingendpoints)實體代表可以直接將內容傳遞給用戶端播放程式應用程式，或傳遞給「內容傳遞網路」(CDN) 以進行進一步散發的串流服務。 來自串流端點服務的輸出串流可以是即時資料流，或媒體服務帳戶中的隨選視訊資產。 當您建立媒體服務帳戶時，系統會為您建立**預設**串流端點 (處於已停止狀態)。 您無法刪除預設串流端點。 您可以在帳戶下建立額外的串流端點。 若要開始串流處理視訊，您需要啟動串流端點。 

## <a name="streamingendpoint-types"></a>StreamingEndpoint 類型  

**StreamingEndpoint** 有兩種類型：「標準」和「進階」。 類型會根據您為串流端點配置的縮放單位數 (`scaleUnits`) 來定義。 

類型描述如下表所示：  

|類型|縮放單位|說明|
|--------|--------|--------|  
|**標準串流端點** (建議)|0|**標準**類型是幾乎所有串流案例和觀眾規模的建議選項。 **標準**類型會自動調整輸出頻寬。 <br/>針對需求極高的客戶，媒體服務提供**進階**串流端點，可用來相應增加容量，以因應極大量的網際網路觀眾。 如果您預期有大量觀眾和同時觀看人數，請透過 amsstreaming@microsoft.com 與我們連絡，以取得您否需要移至**進階**類型的指引。 |
|**進階串流端點**|>0|**進階**串流端點適合進階工作負載，提供專用並能靈活調整的頻寬容量。 您可以藉由調整 `scaleUnits` 來移至**進階**類型。 `scaleUnits` 提供您專用的輸出容量，您可以透過每次增量 200 Mbps 的方式來購買。 使用**進階**類型時，每個啟用的單位會提供額外頻寬容量給應用程式。 |

## <a name="working-with-cdn"></a>使用 CDN

您應該在大部分情況下啟用 CDN。 不過，如果您預期同時觀看人數最多不會超過 500 人，則建議您停用 CDN，因為 CDN 的功能是依據同時觀看人數進行最佳調整。

### <a name="detailed-explanation-of-how-caching-works"></a>詳細說明快取的運作方式

新增 CDN 時並沒有特定的頻寬值，因為在已啟用 CDN 的串流端點上，所需的頻寬量會有所不同。 這大部分取決於內容類型、熱門程度、位元速率及通訊協定。 CDN 只會快取要求的內容。 這表示，熱門內容會直接從 CDN 提供 (前提是已快取視訊片段)。 快取實況內容的機率很高，因為通常會有許多人觀看一模一樣的內容。 隨選內容可能比較難處理，因為可能會有一些內容很熱門，有一些內容較不受歡迎。 如果您有數百萬個不是很熱門的視訊資產 (每週只有 1 個或 2 個觀看者)，但觀看這些不同視訊的人有數千位，則 CDN 會變得較不具效率。 因為此快取遺漏，您會增加串流端點上的負載。
 
您也需要考量彈性資料流的運作方式。 每個個別的視訊片段都會快取為自有的實體。 例如，如果某個影片第一次有人觀看，而此觀看者跳著觀看幾個幾秒鐘的片段，則只有與此觀看者所觀看內容相關聯的視訊片段會快取到 CDN。 因為有彈性資料流，您通常會有 5 到 7 個位元速率不同的視訊。 如果一個人觀看某個位元速率，而另一個人觀看不同的位元速率，則這些位元速率會分別在 CDN 中快取。 即使兩個人觀看相同的位元速率，視訊也可能透過不同的通訊協定來串流處理。 每個通訊協定 (HLS、MPEG DASH、Smooth Streaming) 會分別進行快取。 因此，每一個位元速率和通訊協定都會分別進行快取，而且只會快取已要求的那些視訊片段。
 
## <a name="streamingendpoint-properties"></a>StreamingEndpoint 屬性 

本節將詳細說明一些 StreamingEndpoint 的屬性。 如需如何建立新串流端點的範例及所有屬性的說明，請參閱[串流端點](https://docs.microsoft.com/rest/api/media/streamingendpoints/create)。 

|屬性|說明|  
|--------------|----------|
|`accessControl`|用來設定此串流端點的下列安全性設定：Akamai 簽章標頭驗證金鑰和可以連線至此端點的 IP 位址。<br />此屬性可以在 `cdnEnabled`"` 設為 false 時進行設定。|  
|`cdnEnabled`|指出此串流端點的 Azure CDN 整合是否已啟用 (預設為停用)。<br /><br /> 如果您將 `cdnEnabled` 設為 True，下列設定就會停用：`customHostNames` 和 `accessControl`。<br /><br />並非所有資料中心都支援 Azure CDN 整合。 若要檢查您的資料中心是否有 Azure CDN 整合，可執行下列動作：<br /><br /> - 試著將 `cdnEnabled` 設為 True。<br /><br /> - 查看 `HTTP Error Code 412` (PreconditionFailed) 的傳回結果，其中的訊息為「無法將串流端點的 CdnEnabled 屬性設定為 True，因為 CDN 功能不適用於目前的區域」。<br /><br /> 如果您收到此錯誤，表示資料中心不支援此功能。 您應該嘗試使用另一個資料中心。|  
|`cdnProfile`|當 `cdnEnabled` 設為 True 時，您也可以傳遞 `cdnProfile` 值。 `cdnProfile` 是 CDN 設定檔的名稱，而該設定檔就是要建立 CDN 端點的位置。 您可以提供現有的 cdnProfile，或使用新的。 如果值為 NULL，而且 `cdnEnabled` 是 True，則會使用預設值 "AzureMediaStreamingPlatformCdnProfile"。 如果提供的 `cdnProfile` 已經存在，則會在其下方建立端點。 如果設定檔不存在，則會自動建立新的設定檔。|
|`cdnProvider`|啟用 CDN 時，您也可以傳遞 `cdnProvider` 值。 `cdnProvider` 會控制將使用哪一個提供者。 目前支援三個值："StandardVerizon"、"PremiumVerizon" 和 "StandardAkamai"。 如果未提供任何值，而且 `cdnEnabled` 是 True，則會使用 "StandardVerizon" (也就是預設值)。|
|`crossSiteAccessPolicies`|用來指定不同用戶端的跨網站存取原則。 如需詳細資訊，請參閱[跨網域原則檔案規格](http://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)和[使服務可跨網域界限使用](http://msdn.microsoft.com/library/cc197955\(v=vs.95\).aspx)。|  
|`customHostNames`|用來將串流端點設定為可接受導向至自訂主機名稱的流量。 這可讓您透過全域流量管理員 (GTM) 進行較簡單的流量管理設定，以及使用品牌化的網域名稱作為串流端點名稱。<br /><br /> 網域名稱的擁有權必須通過 Azure 媒體服務的確認。 Azure 媒體服務會透過要求 `CName` 記錄包含 Azure 媒體服務帳戶識別碼，以作為要新增至使用中網域的元件，來驗證網域名稱的擁有權。 例如，若要將 "sports.contoso.com" 作為串流端點的自訂主機名稱，`<accountId>.contoso.com` 的記錄就必須設定為指向其中一個媒體服務驗證名稱。 驗證主機名稱會由 verifydns.\<mediaservices-dns-zone> 組成。 下表針對不同 Azure 區域，列出預期要在驗證記錄中使用的 DNS 區域。<br /><br /> 北美洲、歐洲、新加坡、香港特別行政區、日本：<br /><br /> - mediaservices.windows.net<br /><br /> - verifydns.mediaservices.windows.net<br /><br /> 中國：<br /><br /> - mediaservices.chinacloudapi.cn<br /><br /> - verifydns.mediaservices.chinacloudapi.cn<br /><br /> 例如，將 "945a4c4e-28ea-45cd-8ccb-a519f6b700ad.contoso.com" 對應到 "verifydns.mediaservices.windows.net" 的 `CName` 記錄，表示 Azure 媒體服務識別碼 945a4c4e-28ea-45cd-8ccb-a519f6b700ad 有 contoso.com 網域的擁有權，因此可讓 contoso.com 底下的任何名稱作為該帳戶底下串流端點的自訂主機名稱。<br /><br /> 若要尋找媒體服務識別碼的值，請前往 [Azure 入口網站](https://portal.azure.com/)，然後選取您的媒體服務帳戶。 媒體服務識別碼會出現在 [儀表板] 頁面的右側。<br /><br /> **警告**：如果有人嘗試設定自訂主機名稱，但沒有適當的 `CName` 記錄驗證，則 DNS 回應會失敗，然後快取會執行一段時間。 當適當的記錄就緒後，可能需要等待一段時間，才會重新驗證快取的回應。 視自訂網域的 DNS 提供者而定，重新驗證記錄可能會花費幾分鐘到一小時的時間。<br /><br /> 除了將 `<accountId>.<parent domain>` 對應到 `verifydns.<mediaservices-dns-zone>` 的 `CName`，您必須建立另一個 `CName`，將自訂主機名稱 (例如，`sports.contoso.com`) 對應到您的媒體服務 StreamingEndpont 主機名稱 (例如，`amstest.streaming.mediaservices.windows.net`)。<br /><br /> **請注意**：如果串流端點位於相同的資料中心內，則不能共用相同的自訂主機名稱。<br /> 此屬性僅適用於標準和進階串流端點，而且可以在 "cdnEnabled" 設為 False 時進行設定<br/><br/> 目前 AMS 不支援使用 SSL 搭配自訂網域。  |  
|`maxCacheAge`|覆寫媒體片段和隨選資訊清單上預設的 max-age HTTP 快取控制標頭 (由串流端點設定)。 此值的設定會以秒為單位。|
|`resourceState`|屬性的值包括：<br /><br /> - 已停止。 串流端點建立之後的初始狀態。<br /><br /> - 啟動中。 串流端點正在轉換為執行中狀態。<br /><br /> - 執行中。 串流端點可將內容串流至用戶端。<br /><br /> - 正在調整大小。 縮放單位正在增加或減少。<br /><br /> - 停止中。 串流端點正在轉換為已停止狀態。<br/><br/> - 刪除中。 正在刪除串流端點。|
|`scaleUnits `|scaleUnits 提供您專用的輸出容量，您可以透過每次增量 200 Mbps 的方式來購買。 如果您需要移至**進階**類型，請調整 `scaleUnits`。 |

## <a name="next-steps"></a>後續步驟

[此存放庫中](https://github.com/Azure-Samples/media-services-v3-dotnet-quickstarts/blob/master/AMSV3Quickstarts/EncodeAndStreamFiles/Program.cs)的範例會示範如何使用 .NET 啟動預設的串流端點。

