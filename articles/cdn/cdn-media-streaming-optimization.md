---
title: 使用 Azure CDN 的媒體串流最佳化
description: 將串流媒體檔案最佳化以便傳遞順暢
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
ms.date: 05/01/2018
ms.author: magattus
ms.openlocfilehash: 9802296170f07bb8599058e230798f647e900d4d
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49093692"
---
# <a name="media-streaming-optimization-with-azure-cdn"></a>使用 Azure CDN 的媒體串流最佳化 
 
網際網路使用高畫質影片的日益頻繁，造成有效率傳遞大型檔案的困難。 客戶期待在世界各地的各種網路和用戶端上順暢播放點播視訊或即時影片資產。 快速且有效的媒體串流檔案傳遞機制，對於確保順暢且愉悅的取用者體驗極為重要。  

即時串流處理媒體特別難傳遞，因為有大量的並行檢視者取用大型檔案。 長時間延遲造成使用者出走。 因為即時串流無法預先進行快取，而且檢視者無法接受長時間延遲，因此必須及時傳遞影片片段。 

串流的要求模式也帶來一些新挑戰。 當熱門的即時串流，或點播視訊發佈新的片段時，同一時間可能有數千到數百萬個檢視者要求資料流。 在此情況下，智慧要求彙總極為重要，因為在尚未快取資產時，它不會讓原始伺服器超過負荷。
 

## <a name="media-streaming-optimizations-for-azure-cdn-from-microsoft"></a>Azure CDN from Microsoft 的媒體串流最佳化

**Azure CDN Standard from Microsoft** 端點會直接使用一般 Web 傳遞最佳化類型來傳遞串流媒體資產。 

**Azure CDN Standard from Microsoft** 媒體串流處理最佳化，適合使用媒體片段傳遞處理即時或點播視訊串流媒體。 此程序不同於透過漸進式下載或使用位元組範圍要求的單一大型資產傳輸。 如需該樣式的媒體傳遞相關資訊，請參閱[使用 Azure CDN 的大型檔案下載最佳化](cdn-large-file-optimization.md)。

一般媒體傳遞或點播視訊媒體傳遞最佳化類型，會使用 Azure 內容傳遞網路 (CDN) 與後端最佳化以更快傳遞媒體資產。 它們也會使用媒體資產組態，此組態是以經過一段時間學習的最佳作法為基礎。

### <a name="partial-cache-sharing"></a>部分快取共用
部分快取共用，可讓 CDN 向新要求提供部分的快取內容。 例如，如果對 CDN 的第一個要求導致快取遺漏，該要求即會傳送到原始伺服器。 雖然此不完整的內容會載入到 CDN 快取中，但是對 CDN 的其他要求可以開始取得此資料。 


## <a name="media-streaming-optimizations-for-azure-cdn-from-verizon"></a>Azure CDN from Verizon 的媒體串流最佳化

**Azure CDN Standard from Verizon** 和 **Azure CDN Premium from Verizon** 端點會直接使用一般 Web 傳遞最佳化類型來傳遞串流媒體資產。 CDN 有一些功能預設可直接協助傳遞媒體資產。

### <a name="partial-cache-sharing"></a>部分快取共用

部分快取共用，可讓 CDN 向新要求提供部分的快取內容。 例如，如果對 CDN 的第一個要求導致快取遺漏，該要求即會傳送到原始伺服器。 雖然此不完整的內容會載入到 CDN 快取中，但是對 CDN 的其他要求可以開始取得此資料。 

### <a name="cache-fill-wait-time"></a>快取填滿等候時間

 快取填滿等候時間功能會強迫邊緣伺服器保存相同資源的所有後續要求，直到原始伺服器的 HTTP 回應標頭送達為止。 如果原始伺服器的 HTTP 回應標頭在計時器終止前送達，則會在成長中快取之外為所有暫停的要求提供服務。 同時，快取會填入原始伺服器的資料。 根據預設，快取填滿等候時間會設定為 3,000 毫秒。 

 
## <a name="media-streaming-optimizations-for-azure-cdn-from-akamai"></a>Azure CDN from Akamai 的媒體串流最佳化
 
**Azure CDN Standard from Akamai** 提供一項功能，可將串流媒體資產有效傳遞給全球的使用者。 此功能會減少延遲，因為它可以降低來源伺服器上的負載。 使用標準 Akamai 定價層可取得這項功能。 

**Azure CDN Standard from Akamai** 媒體串流處理最佳化，適合使用媒體片段傳遞處理即時或點播視訊串流媒體。 此程序不同於透過漸進式下載或使用位元組範圍要求的單一大型資產傳輸。 如需該樣式的媒體傳遞相關資訊，請查看[大型檔案最佳化](cdn-large-file-optimization.md)。

一般媒體傳遞或點播視訊媒體傳遞最佳化類型，會使用 CDN 與後端最佳化以更快傳遞媒體資產。 它們也會使用媒體資產組態，此組態是以經過一段時間學習的最佳作法為基礎。

### <a name="configure-an-akamai-cdn-endpoint-to-optimize-media-streaming"></a>設定 Akamai CDN 端點以最佳化媒體串流處理
 
您可以設定您的內容傳遞網路 (CDN) 端點，以最佳化透過 Azure 入口網站傳遞大型檔案。 使用 REST API 或任何用戶端 SDK 也都能達到相同目的。 下列步驟示範 **Azure CDN Standard from Akamai** 設定檔透過 Azure 入口網站執行的程序：

1. 若要新增端點，請在 Akamai 的 [CDN 設定檔] 頁面上選取 [端點]。
  
    ![新增端點](./media/cdn-media-streaming-optimization/cdn-new-akamai-endpoint.png)

2. 在 [最佳化對象] 下拉式清單中，為點播視訊資產選取 [點播視訊媒體串流]。 如果您結合即時和點播視訊串流處理，請選取 [General media streaming] \(一般媒體串流處理)。

    ![選取的串流](./media/cdn-media-streaming-optimization/02_Creating.png) 
 
建立端點後，最佳化就會套用到符合特定準則的所有檔案。 下一節會說明此程序。 

### <a name="caching"></a>快取

如果 **Azure CDN Standard from Akamai** 偵測到資產是串流資訊清單或片段，它會使用與一般 Web 傳遞不同的快取到期時間。 (請參閱下表中的完整清單。)如往常一樣接受從來源傳送的 cache-control 或 Expires 標頭。 如果資產不是媒體資產，就會使用一般 Web 傳遞的逾期時間快取。

當許多使用者要求還不存在的片段時，短的負快取時間對來源卸載就很有用。 例如該秒無法從原始伺服器取得封包的即時串流。 較長的快取間隔也有助於卸載原始伺服器的要求，因為通常不會修改影片內容。
 

|   | 一般 Web 傳遞 | 一般媒體串流處理 | 點播視訊媒體串流處理  
--- | --- | --- | ---
快取：正向 <br> HTTP 200、203、300、 <br> 301、302 和 410 | 7 天 |365 天 | 365 天   
快取：負向 <br> HTTP 204、305、404 <br> 和 405 | None | 1 秒 | 1 秒
 
### <a name="deal-with-origin-failure"></a>處理原始伺服器失敗  

一般媒體傳遞和點播視訊媒體傳遞也都有以典型要求模式的最佳做法為基礎的原始伺服器逾時和重試記錄。 例如，因為一般媒體傳遞適合即時和點播視訊媒體傳遞，所以會因為即時串流講求時效性而使用比較短的連線逾時。

當連線逾時，CDN 會先重試幾次，再向用戶端傳送「504 - 閘道逾時」錯誤。 

當檔案符合檔案類型和大小條件清單，CDN 會使用媒體串流處理的行為。 否則，會使用一般 Web 傳遞。
   
### <a name="conditions-for-media-streaming-optimization"></a>媒體串流最佳化的條件 

下表列出媒體串流處理最佳化需要滿足的準則集合： 
 
支援的串流類型 | 副檔名  
--- | ---  
Apple HLS | m3u8、m3u、m3ub、key、ts、aac
Adobe HDS | f4m、f4x、drmmeta、bootstrap、f4f、<br>Seg-Frag URL 結構 <br> (matching regex: ^(/.*)Seq(\d+)-Frag(\d+)
DASH | mpd、dash、divx、ismv、m4s、m4v、mp4、mp4v、 <br> sidx、webm、mp4a、m4a、isma
Smooth Streaming | /manifest/, /QualityLevels/Fragments/
  
 
