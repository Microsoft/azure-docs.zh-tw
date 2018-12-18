---
title: Azure 搜尋服務的效能與最佳化考量 | Microsoft Docs
description: 調整 Azure 搜尋服務效能並設定最佳規模
author: LiamCavanagh
manager: jlembicz
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 05/01/2017
ms.author: liamca
ms.openlocfilehash: 89c0352723f1ed00784250b566902028af853d10
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31797757"
---
# <a name="azure-search-performance-and-optimization-considerations"></a>Azure 搜尋服務的效能與最佳化考量
絕佳搜尋體驗是許多行動和 Web 應用程式成功的關鍵。 從房地產到二手車市場，再到線上目錄，快速搜尋和相關結果都將影響客戶體驗。 本文旨在協助您探索如何充分利用 Azure 搜尋服務的最佳做法，特別適用於對延展性、多語言支援或自訂排名有複雜需求的進階案例。  此外，本文件也會概述本質，並涵蓋可在真實世界客戶應用程式中有效率地工作的處理方法。

## <a name="performance-and-scale-tuning-for-search-services"></a>適用於搜尋服務的效能和規模調整
我們全都適用於搜尋引擎 (例如 Bing 和 Google) 以及它們提供的高效能。  因此，當客戶使用您已具備搜尋功能的 Web 或行動應用程式時，他們將期望類似的效能特性。  針對搜尋效能進行最佳化時，其中一個最佳處理方法是將重點放在延遲，也就是查詢完成並傳回結果所花費的時間。  針對搜尋延遲進行最佳化時，請務必：

1. 挑選完成一般搜尋要求所應花費的目標延遲 (或最大時間量)。
2. 針對您的搜尋服務，使用實際資料集來建立和測試真正的工作負載，以衡量這些延遲率。
3. 從較低的每秒查詢數目 (QPS) 開始，持續增加在測試中執行的數目，直到查詢延遲低於定義的目標延遲為止。  這是很重要的效能評定，可協助您規劃應用程式將在使用量方面成長的規模。
4. 如果可能，請重複使用 HTTP 連接。  如果您使用 Azure 搜尋服務 .NET SDK，這表示您應該重複使用執行個體或 [SearchIndexClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient) 執行個體，而且如果您使用 REST API，就應該重複使用單一的 HttpClient。

建立這些測試工作負載時，有一些要牢記的 Azure 搜尋服務特性：

1. 它能夠推送，因此，同一時間可以有許多搜尋查詢，而您 Azure 搜尋服務中的可用資源將會過度負荷。  發生這種情況時，您將會看到 HTTP 503 回應碼。  基於這個理由，最好從各種不同範圍的搜尋要求開始，以查看當您新增更多搜尋要求時延遲率中的差異。
2. 將內容上傳至 Azure 搜尋服務，會影響 Azure 搜尋服務的整體效能和延遲。  如果您預期會在使用者執行搜尋時傳送資料，請務必考慮將這個工作負載納入測試中。
3. 並非所有搜尋查詢都將在同一個效能層級執行。  例如，比起具有大量 Facet 和篩選的查詢，文件查閱或搜尋建議的執行速度通常會更快。  建置測試時，最好將您預期要看見的各種查詢納入考量。  
4. 搜尋要求的變化是非常重要，因為如果您持續執行相同的搜尋要求，比起資料可能包含差異性更大的查詢集，快取資料會開始讓效能看起來變得更好。

> [!NOTE]
> [Visual Studio 負載測試](https://www.visualstudio.com/docs/test/performance-testing/run-performance-tests-app-before-release)是真正適合用來執行您效能評定測試的方式，因為，當您需要針對 Azure 搜尋服務執行查詢並啟用要求的平行處理時，它允許您執行 HTTP 要求。
> 
> 

## <a name="scaling-azure-search-for-high-query-rates-and-throttled-requests"></a>調整 Azure 搜尋服務以提供高速查詢和節流要求
當您收到太多節流要求或增加的查詢負載已超過目標延遲率時，您可以使用下列其中一種方式來查看以降低延遲率：

1. **增加複本︰** 複本就像是您的資料複本，允許 Azure 搜尋服務針對多個複本進行要求的負載平衡。  在複本之間針對資料進行的所有負載平衡和複寫都是由 Azure 搜尋服務所管理，而您可以隨時變更配置給服務的複本數目。  您最多可在標準搜尋服務中配置 12 個複本，並在基本搜尋服務中配置 3 個複本。 您可以從 [Azure 入口網站](search-create-service-portal.md)或 [PowerShell](search-manage-powershell.md) 來調整複本。
2. **增加搜尋服務層︰** Azure 搜尋服務來自[層數](https://azure.microsoft.com/pricing/details/search/)，而這其中的每一層都會提供不同等級的效能。  在某些情況下，您可能會有這麼多查詢，即使當複本超量時，您所在的層仍不足以提供低延遲率。在此情況下，您可能要考慮使用較高的搜尋層，例如 Azure 搜尋服務 S3 層，這非常適合具有大量文件且有極高查詢工作負載的案例。

## <a name="scaling-azure-search-for-slow-individual-queries"></a>針對速度較慢的個別查詢調整 Azure 搜尋服務
延遲率為什麼很慢的另一個原因是，因為單一查詢花費太長的時間才能完成。  在此情況下，新增複本將無法改善延遲率。  在此情況下，有兩個可用選項：

1. **增加分割區** 分割區是一種機制，可分割資料以分散到額外的資源上。  基於這個理由，當您新增第二個分割區時，您的資料就會一分為二。  第三個分割區會將您的索引分成三等份，依此類推。這也會產生效果，在某些情況下，速度較慢的查詢會因為平行處理計算而執行得更快。  我們可以在一些範例中看見這個平行處理搭配具有低度選擇性查詢的查詢時運作得非常好。  這其中包括符合許多文件的查詢，或者當設定 Facet 需要提供超過大量文件的計數時。  由於有許多需要調整文件相關性或計算文件數目所需的計算，因此，新增額外的分割區有助於提供其他計算。  
   
   在標準搜尋服務中最多可有 12 個分割區，在基本搜尋服務中則為 1 個分割區。  您可以從 [Azure 入口網站](search-create-service-portal.md)或 [PowerShell](search-manage-powershell.md) 來調整分割區。
2. **限制高基數欄位︰** 高基數欄位包含具有大量唯一值的可 Facet 或可篩選欄位，因此，會取得許多要在其上計算結果的資源。   例如，將 [產品識別碼] 或 [說明] 欄位設定為可 Facet 或可篩選的會導致高基數，因為文件彼此間大多數值都是唯一的。 如果可能，請限制高基數欄位數目。
3. **增加搜尋服務層︰** 另一種方式是往上移動到更高的 Azure 搜尋服務層，可為速度較慢的查詢改善效能。  每一個較高的層也會提供更快速的 CPU 和更多記憶體，可以對查詢效能產生正面的影響。

## <a name="scaling-for-availability"></a>調整可用性
複本不僅有助於減少查詢延遲，也可以允許高可用性。  利用單一複本，您應該可以預期定期的停機時間，因為伺服器會在軟體更新之後重新開機，或針對其他將發生的維護事件重新開機。  因此，請務必考慮應用程式是否需要搜尋 (查詢) 及寫入 (編製事件的索引) 的高可用性。  Azure 搜尋服務在具有下列屬性的所有付費搜尋供應項目上提供 SLA 選項：

* 針對唯讀工作負載 (查詢)，需有 2 個複本才能達到高可用性
* 針對讀取/寫入工作負載 (查詢和索引) 的高可用性為 3 或更多個複本

如需此內容的詳細資訊，請瀏覽 [Azure 搜尋服務等級協定](https://azure.microsoft.com/support/legal/sla/search/v1_0/)。

由於複本是資料的複本，因此，擁有多個複本可讓 Azure 搜尋服務同時針對一個複本進行電腦重新開機和維護，同時能夠繼續針對其他複本執行查詢。  基於這個理由，如果查詢現在必須根據少一個複本的資料加以執行，您也必須考慮這種停機情形可能會對該查詢產生何種影響。

## <a name="scaling-geo-distributed-workloads-and-provide-geo-redundancy"></a>調整異地分散的工作負載並提供異地備援
針對異地分散的工作負載，您將會發現遠離資料中心 (裝載您 Azure 搜尋服務所在) 的使用者延遲率比較高。  基於這個理由，在更接近這類使用者的區域中，具有多個搜尋服務通常很重要。  Azure 搜尋服務目前不提供自動化方法來跨區域進行 Azure 搜尋服務索引的異地複寫，但是有一些技術可用來讓這個程序更容易實作與管理。 我們將在後續小節中加以概述。

一組異地分散的搜尋服務目標是，在兩個以上的區域中，可以使用兩個以上的索引，而使用者將在這些區域中路由傳送到 Azure 搜尋服務以提供最低延遲，如此範例中所示：

   ![依區域交叉分析服務][1]

### <a name="keeping-data-in-sync-across-multiple-azure-search-services"></a>跨多個 Azure 搜尋服務，讓資料維持同步
有兩個選項可讓分散的搜尋服務維持同步，包括使用 [Azure 搜尋服務索引子](search-indexer-overview.md) 或推送 API (也稱為 [Azure 搜尋服務 REST API](https://docs.microsoft.com/rest/api/searchservice/))。  

### <a name="azure-search-indexers"></a>Azure 搜尋服務索引子
如果您使用 Azure 搜尋服務索引子，您就已經從中央資料存放區 (例如 Azure SQL DB 或 Azure Cosmos DB) 匯入資料變更。 當您建立新的搜尋服務時，針對指向這個相同資料存放區的服務，也只需要建立新的 Azure 搜尋服務索引子。 這樣一來，每當新的變更出現在資料存放區時，接著將會透過各種索引子為它們編製索引。  

以下範例為該架構所呈現的樣子。

   ![具有分散式索引子和服務組合的單一資料來源][2]

### <a name="push-api"></a>推送 API
如果您正在使用 Azure 搜尋推送 API 來 [更新 Azure 搜尋服務索引的內容](https://docs.microsoft.com/rest/api/searchservice/update-index)，您可以在需要更新時，將變更推送到所有搜尋服務，藉以讓各種搜尋服務維持同步。  執行這項操作時，務必確定會處理某一個搜尋服務更新失敗且有一或多個更新成功時的情況。

## <a name="leveraging-azure-traffic-manager"></a>運用 Azure 流量管理員
[Azure 流量管理員](../traffic-manager/traffic-manager-overview.md) 可讓您將要求路由傳送至位於多個地理位置的網站，然後透過多個 Azure 搜尋服務進行備份。  流量管理員的一個優點是，它可以探查 Azure 搜尋服務，以確保它可供使用，並在發生停機時將使用者路由傳送至替代的搜尋服務。  此外，如果您透過 Azure 網站路由傳送搜尋要求，Azure 流量管理員可讓您在網站已啟動，但 Azure 搜尋服務沒有的情況下進行負載平衡。  以下範例為運用流量管理員的架構。

   ![利用中央流量管理員依區域交叉分析服務][3]

## <a name="monitoring-performance"></a>監視效能
Azure 搜尋服務透過 [搜尋流量分析 (STA)](search-traffic-analytics.md)提供功能來分析和監視您的服務。 透過 STA，您可以選擇性地將個別搜尋作業以及彙總的計量記錄到 Azure 儲存體帳戶，然後加以處理來進行分析或在 Power BI 中視覺化。  使用 STA 計量，您可以檢閱效能統計資料，例如查詢或查詢回應時間的平均數目。  此外，作業記錄可讓您向下切入到特定搜尋作業的詳細資料。

STA 是可從 Azure 搜尋服務觀點了解延遲率的寶貴工具。  由於記錄的查詢效能計量是以查詢在 Azure 搜尋服務中進行完整處理所花費的時間為基礎 (從要求它的時間到送出時間)，因此，您能夠使用這個工具來判斷延遲問題是來自 Azure 搜尋服務端或服務外部，例如，來自網路延遲。  

## <a name="next-steps"></a>後續步驟
若要深入了解定價層及每一層的服務限制，請參閱 [Azure 搜尋中的服務限制](search-limits-quotas-capacity.md)。

請參閱 [容量規劃](search-capacity-planning.md) ，以便深入了解分割區和複本組合。

如需更多效能的詳細說明，以及查看一些如何實作本文中討論的最佳化示範，請觀賞下列影片：

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 
> 

<!--Image references-->
[1]: ./media/search-performance-optimization/geo-redundancy.png
[2]: ./media/search-performance-optimization/scale-indexers.png
[3]: ./media/search-performance-optimization/geo-search-traffic-mgr.png
