---
title: 透過 Azure CDN 進行動態網站加速
description: Azure CDN 針對具有動態內容的檔案，支援動態網站加速 (DSA) 最佳化。
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
ms.date: 03/01/2018
ms.author: magattus
ms.openlocfilehash: 4fa681e800197ea241ba1c6cf2180ba04b6e565b
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49092573"
---
# <a name="dynamic-site-acceleration-via-azure-cdn"></a>透過 Azure CDN 進行動態網站加速

隨著社交媒體、電子商務和超高度個人化 web 的爆炸性成長，呈現給使用者的內容百分比正快速成長，從而須以即時方式產生。 使用者期待擁有快速、可靠且個人化的 Web 體驗，無關乎其瀏覽器、位置、裝置或網路。 不過，與這些體驗高度相關的創新，卻也使得頁面下載速度緩慢，並且讓消費者體驗的品質面臨風險。 

標準內容傳遞網路 (CDN) 功能包含快取接近使用者的檔案，從而加速靜態檔案傳遞。 不過，使用動態 Web 應用程式，就無法在邊緣位置中快取該內容，因為伺服器所產生的內容是用以回應使用者的行為。 相較於傳統的邊緣快取，要加速傳遞這類內容更為複雜，並且需要端對端解決方案，在從開始到傳遞的整個資料路徑上微調每個元素。 使用 Azure CDN 動態網站加速 (DSA) 最佳化後，能顯著提升具有動態內容的網頁效能。

**Akamai 中的 Azure CDN** 和 **Verizon 中的 Azure CDN** 都會在端點建立期間透過 [最佳化] 功能表來提供 DSA 最佳化。

> [!Important]
> 針對**來自 Akamai 的 Azure CDN**設定檔，您可以在 CDN 端點建立之後，變更其最佳化。
>   
> 針對**來自 Verizon 的 Azure CDN**設定檔，您無法在 CDN 端點建立之後，變更其最佳化。

## <a name="cdn-endpoint-configuration-to-accelerate-delivery-of-dynamic-files"></a>設定 CDN 端點以加速動態檔案傳遞

若要將 CDN 端點設定成要最佳化動態檔案的傳遞，您可以使用 Azure 入口網站、REST API 或任何用戶端 SDK 以程式設計方式執行相同的動作。 

**若要將 CDN 端點設定成使用 Azure 入口網站進行 DSA 最佳化：**

1. 在 [CDN 設定檔] 頁面中選取 [端點]。

   ![新增 CDN 端點](./media/cdn-dynamic-site-acceleration/cdn-endpoint-profile.png) 

   此時會顯示 [新增端點] 窗格。

2. 在 [最佳化] 之下，選取 [動態網站加速]。

    ![使用 DSA 建立新的 CDN 端點](./media/cdn-dynamic-site-acceleration/cdn-endpoint-dsa.png)

3. 針對 [探查路徑]，輸入檔案的有效路徑。

    探查路徑是 DSA 特定的功能，在建立時需要有效的路徑。 DSA 會使用放在原始伺服器的小型「探查路徑」檔案，將 CDN 的網路路由組態最佳化。 如需探查路徑檔案，您可以下載我們的範例檔案並上傳至您的網站，或使用原始伺服器上大約 10 KB 的現有資產。

4. 輸入其他必要的端點選項 (如需詳細資訊，請參閱[建立新的 CDN 端點](cdn-create-new-endpoint.md#create-a-new-cdn-endpoint))，然後選取 [新增]。

   建立 CDN 端點之後，就會對所有符合特定準則的檔案套用 DSA 最佳化。 


**若要針對 DSA 設定現有端點 (僅限 Akamai 中的 Azure CDN 設定檔)：**

1. 在 [CDN 設定檔] 頁面中，選取您要修改的端點。

2. 從左窗格中，選取 [最佳化]。 

   [最佳化] 頁面隨即出現。

3. 在 [最佳化] 之下，選取 [動態網站加速]，然後選取 [儲存]。

> [!Note]
> DSA 會產生額外費用。 如需詳細資訊，請參閱[內容傳遞網路定價](https://azure.microsoft.com/pricing/details/cdn/)。

## <a name="dsa-optimization-using-azure-cdn"></a>使用 Azure CDN 將 DSA 最佳化

Azure CDN 上之動態網站加速會使用下列技術，從 Akamai 加速傳遞動態資產：

-   [路由最佳化](#route-optimization)
-   [TCP 最佳化](#tcp-optimizations)
-   [物件預先擷取 (僅限 Akamai 中的 Azure CDN)](#object-prefetch-azure-cdn-from-akamai-only)
-   [調整映像壓縮 (僅限 Akamai 中的 Azure CDN)](#adaptive-image-compression-azure-cdn-from-akamai-only)

### <a name="route-optimization"></a>路由最佳化

路由最佳化很重要，因為網際網路是變化不斷的場所，其中的流量和暫時中斷經常會變更網路拓撲。 Border Gateway Protocol (BGP) 是網際網路的路由通訊協定，但透過中繼的存在點 (PoP) 伺服器將會有更快速的路由。 

路線最佳化會選擇與來源之間的最佳路徑，使站台能持續存取，且可透過最快速且最可靠的路由，將動態內容傳遞給使用者。 

Akamai 網路會使用一些技術，透過 Akamai 伺服器中不同的節點來收集即時資料並比較各種路徑，以及透過跨開放式網際網路的預設 BGP 路由，判斷來源與 CDN 邊緣之間的最快路由。 這些技術可避開網際網路壅塞點和長途路由。 

同樣地，Verizon 網路會組合使用「任一傳播」DNS、高容量支援的 PoP 和健康情況檢查，來判斷能以最有效率的方式將資料從用戶端路由到來源的最佳閘道。
 
如此一來，就能更快速且更可靠地將完全動態和交易式內容傳遞給使用者，即使它不可快取也一樣。 

### <a name="tcp-optimizations"></a>TCP 最佳化

傳輸控制通訊協定 (TCP) 是網際網路通訊協定套件的標準，可用來傳遞 IP 網路上應用程式之間的資訊。  根據預設，必須有數個來回要求才能設定 TCP 連線，且必須有數個限制才能避免會導致大規模效率不足的網路壅塞。 **Akamai 中的 Azure CDN** 會處理這個問題，方法是將下列三個方面最佳化： 

 - [排除 TCP 慢速啟動](#eliminating-tcp-slow-start)
 - [利用持續連線](#leveraging-persistent-connections)
 - [調整 TCP 封包參數](#tuning-tcp-packet-parameters)

#### <a name="eliminating-tcp-slow-start"></a>排除 TCP 慢速啟動

TCP「慢速啟動」是 TCP 通訊協定的演算法，可藉由限制透過網路傳送的資料量來防止網路壅塞。 它會從傳送者與接收者之間的小型壅塞視窗大小開始，直到觸達最大值或偵測到封包遺失為止。

 **來自 Akamai 的 Azure CDN** 和**來自 Verizon 的 Azure CDN** 設定檔都可透過下列三個步驟消除 TCP 啟動緩慢的情況：

1. 健康狀態和頻寬監視用來測量 PoP Edge Server 之間的連線頻寬。
    
2. PoP Edge Server 之間會共用這些計量數據，因此每一部伺服器都會知道其他周遭 PoP 的網路狀況和伺服器健康情況。  
    
3. CDN Edge Server 會提出某些傳輸參數的假設，例如與其附近的其他 CDN Edge Server 通訊時的最佳視窗大小。 這個步驟表示，如果 CDN Edge Server 之間的連線健康情況能夠負荷較高的封包資料傳輸，就可增加初始壅塞視窗的大小。  

#### <a name="leveraging-persistent-connections"></a>利用持續連線

相較於使用者直接連線到您的來源，使用 CDN 時，直接連線到您原始伺服器的唯一機器較少。 Azure CDN 也會將使用者要求一起加入集區，能與來源建立較少的連線。

如先前所述，需要幾個交握要求才能建立 TCP 連線。 持續連線 (由 `Keep-Alive` HTTP 標頭所實作) 會對多個 HTTP 要求重複使用現有的 TCP 連線，以省下往返時間並加速傳遞。 

**Verizon 中的 Azure CDN** 也會透過 TCP 連線定期傳送保持連線封包，以防止開啟的連線變成關閉狀態。

#### <a name="tuning-tcp-packet-parameters"></a>調整 TCP 封包參數

**Akamai 中的 Azure CDN** 會調整管理伺服器對伺服器連線的參數，並減少擷取網站中內嵌之內容所需的長途往返量，方法是使用下列技術：

- 增加一開始的壅塞範圍，讓更多封包無須等候確認即可傳送出去。
- 減少初始重新傳輸逾時，以便偵測到遺失，並更快速地重新傳輸。
- 減少最小和最大的重新傳輸逾，從而降低假設封包在傳輸中遺失之前的等待時間。

### <a name="object-prefetch-azure-cdn-from-akamai-only"></a>物件預先擷取 (僅限 Akamai 中的 Azure CDN)

大多數網站是由參考諸如映像和指令碼等各種其他資源的 HTML 網頁所組成。 一般而言，當用戶端要求網頁時，瀏覽器會先下載及剖析 HTML 物件，然後再進行完全載入網頁所需的其他連結資產之要求。 

預先擷取是 HTML 提供給瀏覽器時，並甚至在瀏覽器提出這些物件要求之前，擷取內嵌在 HTML 網頁中之映像和指令碼的技術。 

若在 CDN 提供以 HTML 為基礎的網頁給用戶端瀏覽器時開啟 [預先擷取] 選項，CDN 就會剖析 HTML 檔案，並針對任何連結的資源提出其他要求，並將它儲存在其快取中。 當用戶端提出連結資產的要求時，CDN Edge Server 就已擁有要求的物件，並可以立即提供而不需要在來源之間往返。 此最佳化有助於可快取和不可快取的內容。

### <a name="adaptive-image-compression-azure-cdn-from-akamai-only"></a>調整映像壓縮 (僅限 Akamai 中的 Azure CDN)

某些裝置 (特別是行動裝置) 時常會遇到較緩慢的網路速度。 在這些情節中，對使用者較有利的方式，是在他們的網頁上更快速地接收較小的映像，而非長時間等待完整解析度的映像。

這項功能會在網路速度較慢時自動監視網路品質，並採用標準 JPEG 壓縮方法，以改善傳遞時間。

調整映像壓縮 | 副檔名  
--- | ---  
JPEG 壓縮 | .jpg、.jpeg、.jpe、.jig、.jgig、.jgi

## <a name="caching"></a>快取

若使用 DSA，CDN 預設會關閉快取功能，即使來源在回應中納入 `Cache-Control` 或 `Expires` 標頭也是如此。 DSA 通常用於不得快取的動態資產，因為這類資產對每個用戶端都是唯一的。 快取可能會破壞此行為。

如果您的網站混雜了靜態和動態資產，您最好採用混合式方法以獲得最佳效能。 

針對**來自 Verizon 的 Azure CDN 標準**和**來自 Akamai 的 Azure CDN 標準**設定檔，您可以使用[快取規則](cdn-caching-rules.md)來開啟特定 DSA 端點的快取。

存取快取規則：

1. 從 [CDN 設定檔] 頁面的 [設定] 底下，選取 [快取規則]。  
    
    ![CDN [快取規則] 按鈕](./media/cdn-dynamic-site-acceleration/cdn-caching-rules-btn.png)

    [快取規則] 頁面隨即開啟。

2. 建立全域或自訂快取規則以開啟 DSA 端點的快取。 

針對**來自 Verizon 的 Azure CDN 進階**設定檔，您可以使用[規則引擎](cdn-rules-engine.md)來開啟特定 DSA 端點的快取。 所建立的任何規則只會影響您設定檔中已針對 DSA 最佳化的這些端點。 

存取規則引擎：
    
1. 從 [CDN 設定檔] 頁面選取 [管理]。  
    
    ![CDN 設定檔管理按鈕](./media/cdn-dynamic-site-acceleration/cdn-manage-btn.png)

    隨即開啟 CDN 管理入口網站。

2. 從 CDN 管理入口網站，選取 [ADN]，然後選取 [規則引擎]。 

    ![DSA 的規則引擎](./media/cdn-dynamic-site-acceleration/cdn-dsa-rules-engine.png)



此外，您可以使用兩個 CDN 端點：一個使用 DSA 最佳化的端點用來傳遞動態資產，另一個使用靜態最佳化類型 (例如一般 Web 傳遞) 最佳化的端點用來傳遞可快取的資產。 將網頁 URL 修改為直接連結至您打算使用之 CDN 端點上的資產。 

例如：`mydynamic.azureedge.net/index.html` 是從 DSA 端點載入的動態網頁。  HTML 網頁會參考多個靜態資產 (例如 JavaScript 程式庫)，或參考從靜態 CDN 端點載入的映像 (例如 `mystatic.azureedge.net/banner.jpg` 和 `mystatic.azureedge.net/scripts.js`)。 



