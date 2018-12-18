---
title: 使用 Azure CDN 的大型檔案下載最佳化
description: 本文說明如何將大型檔案下載最佳化。
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
ms.openlocfilehash: 9793348b47763e6de10992b9a8a4606fc532cc4d
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094015"
---
# <a name="large-file-download-optimization-with-azure-cdn"></a>使用 Azure CDN 的大型檔案下載最佳化

透過網際網路傳遞的內容檔案大小，由於功能增強、圖形改善和媒體內容更豐富，而持續成長。 這種成長是由許多因素所驅動，包括：寬頻滲透、大型廉價存放裝置、高畫質影片普遍增加，以及連線至網際網路的裝置 (IoT) 等。 快速且有效的大型檔案傳遞機制，對於確保順暢且愉悅的取用者體驗極為重要。

傳送大型檔案有幾項困難。 首先，大型檔案的平均下載時間很可觀，因為許多應用程式可能無法循序下載所有資料。 在某些情況下，應用程式可能先下載檔案的最後一部分，然後再下載第一部分。 當只要求少量的檔案，或使用者暫停下載時，下載會失敗。 下載也可能會延遲到內容傳遞網路 (CDN) 從來源伺服器擷取整個檔案之後。 

其次，使用者電腦和檔案之間的延遲，決定使用者檢視內容的速度。 此外，網路壅塞和容量問題也會影響輸送量。 伺服器與使用者之間遼遠的距離，為封包遺失額外提供許多機會，降低品質。 因輸送量有限與封包遺失增多造成的品質降低，可能表現於完成檔案下載的等待時間大幅增加。 

第三，許多大型檔案不會完整傳遞。 使用者可能會中途取消下載，或只觀賞冗長 MP4 影片的前幾分鐘。 因此，許多軟體和媒體傳遞公司希望只傳遞所要求檔案的一部分。 有效率散佈要求的部分會降低原始伺服器的輸出流量。 有效率的散佈也會減少原始伺服器上的記憶體與 I/O 壓力。 


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-from-microsoft"></a>透過 Azure CDN from Microsoft 將大型檔案的傳遞最佳化

**Azure CDN Standard from Microsoft** 端點可傳遞大型檔案，且檔案大小沒有限制。 預設會開啟額外的功能，以更快傳遞大型檔案。

### <a name="object-chunking"></a>物件區塊化 

**Azure CDN Standard from Microsoft** 使用稱為物件區塊化的技術。 要求大型檔案時，CDN 會從原始伺服器擷取較小的檔案片段。 CDN POP 伺服器收到完整或位元組範圍的檔案要求後，CDN 邊緣伺服器會從原始伺服器以 8 MB 的區塊大小要求檔案。 

區塊抵達 CDN Edge 之後，會被快取並立即提供給使用者。 然後 CDN 會以平行方式預先提取下一個區塊。 此預先擷取可確保內容領先使用者一個區塊，以降低延遲。 此程序一直持續到整個檔案下載完畢 (如有要求)、取得所有位元組範圍 (如有要求)，或用戶端終止連線。 

如需位元組範圍要求的詳細資訊，請參閱 [RFC 7233](https://tools.ietf.org/html/rfc7233)。

CDN 會在收到任何區塊時即予以快取。 不必快取 CDN 快取上的整個檔案。 CDN 快取會提供後續的檔案或位元組範圍要求 。 如果不是在 CDN 快取所有的區塊，就會使用預先擷取向原始伺服器要求區塊。 此最佳化依賴原始伺服器的功能來支援位元組範圍的要求；如果來源伺服器不支援位元組範圍要求，這個最佳化則無效。 

### <a name="conditions-for-large-file-optimization"></a>大型檔案最佳化的條件
當您使用一般的 Web 傳遞最佳化類型時，預設會開啟 **Azure CDN Standard from Microsoft** 的大型檔案最佳化功能。 檔案大小上限沒有任何限制。


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-from-verizon"></a>透過 Azure CDN from Verizon 將大型檔案的傳遞最佳化

**Azure CDN Standard from Verizon** 和 **Azure CDN Premium from Verizon** 端點可傳遞大型檔案，且檔案大小沒有限制。 預設會開啟額外的功能，以更快傳遞大型檔案。

### <a name="complete-cache-fill"></a>完成快取填滿

預設的完成快取填滿功能，可讓 CDN 在初始要求被放棄或遺失時，將檔案提取至快取。 

完成快取填滿最適合大型資產。 使用者一般不會從頭到尾下載。 他們使用漸進式下載。 預設行為會強制 Edge Server 從原始伺服器起始資產的背景擷取。 之後，資產會位於 Edge Server 的本機快取中。 在完整物件位於快取之後，Edge Server 可針對快取的物件滿足 CDN 的位元組範圍要求。

預設行為可以透過 **Azure CDN Premium from Verizon** 中的規則引擎來停用。

### <a name="peer-cache-fill-hot-filing"></a>對等快取填滿 Hotfiling

預設的對等快取填滿 hot-filing 功能使用複雜的專屬演算法。 它根據頻寬及彙總要求計量，使用其他的快取 Edge Server，滿足用戶端對大型、熱門物件的要求。 這項功能可防止將大量額外要求傳送至使用者原始伺服器的情況。 

### <a name="conditions-for-large-file-optimization"></a>大型檔案最佳化的條件

當您使用一般的 Web 傳遞最佳化類型時，預設會開啟 **Azure CDN Standard from Verizon** 和 **Azure CDN Premium from Verizon** 的大型檔案最佳化功能。 檔案大小上限沒有任何限制。 


## <a name="optimize-for-delivery-of-large-files-with-azure-cdn-standard-from-akamai"></a>透過 Azure CDN Standard from Akamai 將大型檔案的傳遞最佳化

**Azure CDN Standard from Akamai** 設定檔端點提供一項功能，可大規模地將大型檔案有效傳遞給全球各地的使用者。 此功能會減少延遲，因為它可以降低來源伺服器上的負載。

大型檔案最佳化類型功能會開啟網路最佳化和組態，更快速且反應迅速地傳遞大型檔案。 使用 **Azure CDN Standard from Akamai** 端點的一般 Web 傳遞只會快取低於 1.8 GB 的檔案，而且可以建立通道來傳送 (而非快取) 檔案，最大可達 150 GB。 大型檔案最佳化快取檔案最多 150 GB。

大型檔案最佳化在滿足特定條件下會生效。 條件包括原始伺服器的運作方式以及所要求的檔案大小和類型。 

### <a name="configure-an-akamai-cdn-endpoint-to-optimize-delivery-of-large-files"></a>設定 Akamai CDN 端點以最佳化大型檔案的傳遞

您可以設定 **Azure CDN Standard from Akamai** 端點，以透過 Azure 入口網站將大型檔案的傳遞最佳化。 使用 REST API 或任何用戶端 SDK 也都能達到相同目的。 下列步驟示範 **Azure CDN Standard from Akamai** 設定檔透過 Azure 入口網站執行的程序：

1. 若要新增端點，請在 Akamai 的 [CDN 設定檔] 頁面上選取 [端點]。

    ![新增端點](./media/cdn-large-file-optimization/cdn-new-akamai-endpoint.png)    
 
2. 在 [最佳化對象] 下拉式清單中，選取 [下載大型檔案]。

    ![已選取大型檔案最佳化](./media/cdn-large-file-optimization/cdn-large-file-select.png)


建立 CDN 端點後，就會對所有符合特定準則的檔案套用大型檔案最佳化。 下一節會說明此程序。

### <a name="object-chunking"></a>物件區塊化 

使用 **Azure CDN Standard from Akamai** 的大型檔案最佳化會使用稱為物件區塊化的技術。 要求大型檔案時，CDN 會從原始伺服器擷取較小的檔案片段。 在 CDN POP 伺服器收到完整或位元組範圍的檔案要求之後，它會檢查此最佳化是否支援檔案類型。 它也會檢查檔案類型是否符合檔案大小需求。 如果檔案大小大於 10 MB，則 CDN Edge Server 會向原始伺服器要求 2 MB 區塊的檔案。 

區塊抵達 CDN Edge 之後，會被快取並立即提供給使用者。 然後 CDN 會以平行方式預先提取下一個區塊。 此預先擷取可確保內容領先使用者一個區塊，以降低延遲。 此程序一直持續到整個檔案下載完畢 (如有要求)、取得所有位元組範圍 (如有要求)，或用戶端終止連線。 

如需位元組範圍要求的詳細資訊，請參閱 [RFC 7233](https://tools.ietf.org/html/rfc7233)。

CDN 會在收到任何區塊時即予以快取。 不必快取 CDN 快取上的整個檔案。 CDN 快取會提供後續的檔案或位元組範圍要求 。 如果不是在 CDN 快取所有的區塊，就會使用預先擷取向原始伺服器要求區塊。 此最佳化依賴原始伺服器的功能來支援位元組範圍的要求；如果來源伺服器不支援位元組範圍要求，這個最佳化則無效。

### <a name="caching"></a>快取
大型檔案最佳化會使用不同的預設從一般 Web 傳遞快取逾期時間。 它會根據 HTTP 回應碼來區分正向快取與負向快取。 如果原始伺服器透過回應中的 Cache-control 或 Expires 標頭指定到期時間，則 CDN 會接受該值。 當原始伺服器未指定，而且檔案符合此最佳化類型的類型和大小條件時，CDN 會使用預設值進行大型檔案最佳化。 否則，CDN 會使用預設值進行一般 Web 傳遞。


|    | 一般 Web | 大型檔案最佳化 
--- | --- | --- 
快取：正向 <br> HTTP 200、203、300、 <br> 301、302 和 410 | 7 天 |1 天  
快取：負向 <br> HTTP 204、305、404 <br> 和 405 | None | 1 秒 

### <a name="deal-with-origin-failure"></a>處理原始伺服器失敗

原始伺服器讀取逾時長度從兩秒的一般 Web 傳遞，增加到兩分鐘的大型檔案最佳化類型。 這會增加帳戶，讓較大的檔案大小避免提早逾時連接。

當連線逾時，CDN 會先重試幾次，再向用戶端傳送「504 - 閘道逾時」錯誤。 

### <a name="conditions-for-large-file-optimization"></a>大型檔案最佳化的條件

下表列出大型檔案最佳化所需滿足的準則集合：

條件 | 值 
--- | --- 
支援的檔案類型 | 3g2、3gp、asf、avi、bz2、dmg、exe、f4v、flv、 <br> gz、hdp、iso、jxr、m4v、mkv、mov、mp4、 <br> mpeg、mpg、mts、pkg、qt、rm、swf、tar、 <br> tgz、wdp、webm、webp、wma、wmv、zip  
檔案大小下限 | 10 MB 
檔案大小上限 | 150 GB 
原始伺服器特性 | 必須支援位元組範圍要求 

## <a name="additional-considerations"></a>其他考量

請考慮此最佳化類型的下列其他層面：

- 區塊處理程序會對原始伺服器產生其他要求。 不過，原始伺服器傳遞的整個資料量會小很多。 區塊處理會導致 CDN 更好的快取特性。

- 降低原始伺服器的記憶體和 I/O 壓力，因為傳遞較小的檔案片段。

- 在 CDN 快取的區塊，在內容到期或從快取收回之前，對原始伺服器沒有任何其他要求。

- 使用者可以向 CDN 提出範圍要求，系統會將這些要求視為任何一般檔案來處理。 只有當檔案類型有效，且位元組範圍介於 10 MB 到 150 GB 之間時，才適用最佳化。 如果所要求的檔案大小平均值小於 10 MB，請改用一般 Web 傳遞。

