---
title: Azure Cosmos DB 如何啟用周全的全域散發 | Microsoft Docs
description: 了解如何從 Azure Cosmos DB (全域散發的多模型資料庫服務)，使用全域資料庫進行全球規模的異地複寫、多重主機、容錯移轉及資料復原。
services: cosmos-db
author: markjbrown
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: mjbrown
ms.openlocfilehash: 1aa0fa3d4d9e1821a6980d9334456a78eba7bfbc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956909"
---
# <a name="how-azure-cosmos-db-enables-turnkey-global-distribution"></a>Azure Cosmos DB 如何啟用周全的全域散發
Azure Cosmos DB 提供下列功能，讓您輕鬆地撰寫全域散發的應用程式。 透過 Azure Cosmos DB 的資源提供者型 [REST API](https://docs.microsoft.com/rest/api/cosmos-db-resource-provider/) 和 Azure 入口網站，即可使用這些功能。

## <a id="GlobalDistribution"></a>全域散發

### <a id="RegionalPresence"></a>遍及各區，無所不在 
Azure 透過不斷連上[新的地理區域](https://azure.microsoft.com/regions/)，而日益普及。 Azure Cosmos DB 在 Azure 中是歸類為*基本服務*，預設會在所有新的 Azure 區域中提供。 一旦 Azure 為企業開啟新的區域，這就能讓您將地理區域關聯至 Azure Cosmos DB 資料庫帳戶。

### <a id="MultiMasterSupport"></a>多重主機支援
Azure Cosmos DB 會提供對多重主機區域的原生伺服器端支援，這些區域同樣會參與隨處寫入模型。 此支援會透過[無法達成即提供折讓的 SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/) 提供 <10 毫秒的寫入延遲和 99.999% 的寫入可用性。 多重主機適用於所有的 API (包括[SQL](sql-api-introduction.md)、[MongoDB](mongodb-introduction.md)、[Cassandra](cassandra-introduction.md)、[圖形](graph-introduction.md)與[資料表](table-introduction.md)) 並來自 Cosmos DB 的所有 SDK 語言。 Azure Cosmos DB 針對具有多重主機功能的帳戶支援 4 個不同的一致性層級 (限定過期、工作階段、開頭一致與最終)。

### <a id="UnlimitedRegionsPerAccount"></a>將數目不拘的區域與您的 Azure Cosmos DB 資料庫帳戶產生關聯
Azure Cosmos DB 可讓您將任何數目的 Azure 區域與您的 Azure Cosmos DB 資料庫帳戶產生關聯。 除了異地隔離限制 (例如中國、德國) 之外，可和 Azure Cosmos DB 資料庫帳戶產生關聯的區域數沒有限制。 下圖顯示設定為橫跨 25 個 Azure 區域的資料庫帳戶：

![橫跨 25 個 Azure 區域的 Azure Cosmos DB 資料庫帳戶](./media/distribute-data-globally-turnkey/spanning-regions.png)


### <a id="PolicyBasedGeoFencing"></a>原則式異地隔離限制
Azure Cosmos DB 是專為支援原則式異地隔離所設計的。 異地隔離這個重要元件可確保資料管理和法規遵循限制，並防止特定地區與您的帳戶產生關聯。 異地隔離範例包括 (但不限於) 在主權雲端 (例如中國和德國) 或政府稅務界限 (例如澳洲) 內界定各區域的全域散發範疇。 使用 Azure 訂用帳戶的中繼資料控制原則。

### <a id="DynamicallyAddRegions"></a>動態新增和移除區域
Azure Cosmos DB 允許您隨時從資料庫帳戶新增 (建立關聯) 或移除 (中斷關聯) 區域 (請參閱[上圖](#UnlimitedRegionsPerAccount))。 由於 Azure Cosmos DB 會跨分割區平行複寫資料，新區域在加入後的 30 分鐘內即可在世界各地使用 (假設您的資料在 100 TB 以內)。 

### <a id="FailoverPriorities"></a>容錯移轉優先順序
若未使用多重主機，客戶可以在服務中斷時，精確地控制區域性容錯移轉的順序，Azure Cosmos DB 允許您為各個與資料庫帳戶相關聯的區域排定「優先順序」(請參閱下圖)。 Azure Cosmos DB 確保會以您所指定的優先順序自動進行容錯移轉。 如需有關區域性容錯移轉的詳細資訊，請參閱 [Azure Cosmos DB 中商務持續性的自動區域容錯移轉](regional-failover.md)。 下圖顯示 Azure Cosmos DB 中的租用戶如何為與資料庫帳戶相關聯的區域設定容錯移轉優先順序 (右窗格)：

![使用 Azure Cosmos DB 設定容錯移轉優先順序](./media/distribute-data-globally-turnkey/failover-priorities.png)

### <a id="ConsistencyLevels"></a>多個定義完善且適用於全域散發資料庫的一致性模型
Azure Cosmos DB 支援以 SLA 為後援的[多個定義完善、直覺且實用的一致性模型](consistency-levels.md)。 您可以根據工作負載/案例選擇特定的一致性模型 (從可用的選項清單中)。

### <a id="TunableConsistency"></a>可微調全球複寫資料庫不一致之處
Azure Cosmos DB 可讓您在執行階段以程式設計方式覆寫和放寬個別要求的預設一致性選擇。

### <a id="DynamicallyConfigurableReadWriteRegions"></a>可動態設定讀取和寫入區域
Azure Cosmos DB 可讓您將區域 (和資料庫相關聯) 設定為「讀取」、「寫入」或「讀取/寫入」區域。

### <a id="ElasticallyScaleThroughput"></a>在 Azure 區域，來彈性調整的輸送量
您能以程式設計方式佈建輸送量，來彈性地調整 Azure Cosmos DB 容器。 該輸送量會套用至散發 Azure Cosmos DB 容器的所有區域。

### <a id="GeoLocalReadsAndWrites"></a>異地本機讀取和寫入
資料庫分散在世界各地的主要優點，是世界各地資料都能提供低延遲存取。 Azure Cosmos DB 幾乎可在世界各地提供最低延遲 (<10 毫秒) 的讀取及寫入。 它可確保所有讀取服務都會從最接近的 (本地) 區域提供。 為了提供讀取要求的服務，系統會使用發出讀取所在區域的仲裁。 這同樣適用於寫入。 只有大多數複本在本機永久認可寫入，而不限在遠端複本上認可寫入之後，寫入才會受到認可。 換句話說，Azure Cosmos DB 的複寫通訊協定的運作方式會假設讀取和寫入仲裁永遠是發出要求所在區域的仲裁。

### <a id="ManualFailover"></a>手動容錯移轉
Azure Cosmos DB 可讓您觸發資料庫帳戶的容錯移轉，以驗證整個應用程式 (超越資料庫) 的*端對端*可用性屬性。 由於能保證失敗偵測和選出領導者的安全性與作用中屬性，因此 Azure Cosmos DB 針對由租用戶起始的手動容錯移轉作業，能夠做出*零資料遺失*的保證。

### <a id="AutomaticFailover"></a>自動容錯移轉
Azure Cosmos DB 支援在一或多個區域中斷期間自動容錯移轉。 進行區域容錯移轉時，Azure Cosmos DB 會維持其讀取延遲、執行時間可用性、一致性及輸送量 SLA。 Azure Cosmos DB 針對自動容錯移轉作業的持續時間設有上限， 因為區域服務中斷時很可能會遺失資料。

### <a id="GranularFailover"></a>專為不同的容錯移轉資料粒度設計
目前是在資料庫帳戶的資料粒度公開自動和手動容錯移轉功能。 請注意，Azure Cosmos DB 在內部的設計是以更精細的資料庫、容器或甚至是 (擁有一系列索引鍵的容器) 分割區的資料粒度來提供*自動*容錯移轉的。 

### <a id="MultiHomingAPIs"></a>Azure Cosmos DB 中的多路連接

在具有一個寫入區域和多個唯讀複本的單一主機案例中，Azure Cosmos DB 可讓您使用邏輯 (無從驗證區域) 或實體 (區域特定) 端點來與資料庫互動。 使用邏輯端點，可確保應用程式在容錯移轉時能夠透明地進行多路連接。 實體端點則可微調控制應用程式，將讀取和寫入將重新導向特定區域。 如果有多個已啟用寫入的區域，使用者應該指定區域特定的端點，其中包含依使用量和重新導向的優先順序新增的區域。

在這些文章中，可以找到如何設定 [SQL API](../cosmos-db/tutorial-global-distribution-sql-api.md)、[資料表 API](../cosmos-db/tutorial-global-distribution-table.md) 和 [MongoDB API](../cosmos-db/tutorial-global-distribution-mongodb.md) 讀取喜好設定的資訊。

### <a id="TransparentSchemaMigration"></a>透明且一致的資料庫結構描述與索引移轉 
Azure Cosmos DB 為完全[無從驗證結構描述](http://www.vldb.org/pvldb/vol8/p1668-shukla.pdf) \(英文\)。 資料庫引擎的獨特設計能讓 Azure Cosmos DB 自動且同步對擷取時的所有資料編製索引，不需要使用者提供任何結構描述或次要索引。 這可讓您快速地反覆查看分散在世界各地的應用程式，而不必擔心資料庫結構描述和索引移轉，或協調多階段應用程式推出結構描述變更。 Azure Cosmos DB 可確保任何明確由您對編製索引原則所做的變更，都不會導致效能或可用性降低。  

### <a id="ComprehensiveSLAs"></a>全方位 SLA (不只有高可用性)
Azure Cosmos DB 是全域散發的資料庫服務，可為全域規模的資料庫提供在**可用性**、**延遲**、**輸送量**和**一致性**方面定義完善且全方位的 SLA，無論與資料庫相關聯的區域數目為何。  

## <a id="LatencyGuarantees"></a>延遲保證
像是 Azure Cosmos DB 之全域散發資料庫服務的主要優點是，為您在世界各地的資料提供低延遲存取。 Azure Cosmos DB 可為各種資料庫作業提供 99% 的情況下低延遲保證。 Azure Cosmos DB 採用的複寫通訊協定可確保一律會在用戶端所在區域執行資料庫作業 (讀取和寫入)。 Azure Cosmos DB 的延遲 SLA，可為各種要求和回應大小的讀取、(同步) 編製索引的寫入和查詢提供 99% 的情況下最低延遲保證。 寫入延遲保證，包含在本地區域內的持久多數仲裁認可。

### <a id="LatencyAndConsistency"></a>延遲和一致性的關聯性 
為使全域散發的服務在全域散發的安裝程式中提供強式一致性，它需要同步複寫寫入資料，或同步執行跨區域讀取。 光速和廣域網路可靠性，使得強式一致性將會導致資料庫作業有較高的延遲和降低的可用性。 因此，為使一致性寬鬆的所有單一區域帳戶和多重區域帳戶擁有最低延遲和 99.99% 的可用性保證，並使所有多重區域資料庫帳戶擁有 99.999% 的可用性保證，服務需採用非同步複寫。 這轉而要求服務同時必須提供[定義完善、寬鬆的一致性模型](consistency-levels.md)，比強式稍差 (以保證提供低延遲與高可用性)，且最好比「最終」一致性 (直覺式程式設計模型) 更強。

Azure Cosmos DB 能確保不需要讀取作業來連絡跨多個區域的複本，以提供特定一致性層級保證。 同樣地，它可確保跨所有區域複寫資料時，不會封鎖寫入作業 (也就是，寫入會以非同步的方式跨區域複寫)。 對於多重區域資料庫帳戶，有提供強式與多個寬鬆的一致性層級。 

### <a id="LatencyAndAvailability"></a>延遲和可用性的關聯性 
延遲和可用性就像錢幣的一體兩面。 我們談論的是穩定狀態下作業的延遲，以及在存在失敗或網路分割之情況下的可用性。 從應用程式的觀點而言，執行緩慢的資料庫作業和無法使用的資料庫，並無法辨別。 

Azure Cosmos DB 會就各種資料庫作業的延遲提供絕對的時間上限，來區分高延遲和無法使用的情況。 如果完成資料庫作業所花費的時間超過上限，Azure Cosmos DB 就會傳回逾時錯誤。 Azure Cosmos DB 可用性 SLA 可確保針對可用性 SLA 計算逾時。 

### <a id="LatencyAndThroughput"></a>延遲和輸送量的關聯性
Azure Cosmos DB 不會要您在延遲和輸送量之間做出選擇。 Azure Cosmos DB 遵循 99% 的情況下延遲的 SLA，並傳遞您已佈建的輸送量。 

## <a id="ConsistencyGuarantees"></a>一致性保證
雖然[強式一致性模型](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf) \(英文\) 是資料可程式性的黃金標準，但它需要付出高延遲 (穩定狀態下) 和降低可用性 (失敗時) 的高昂代價。 

Azure Cosmos DB 提供定義完善的程式設計模型，讓您可以預期複寫資料的一致性。 為了讓您輕鬆建置具有多路連接功能的全域散發應用程式，Azure Cosmos DB 所公開的一致性模型是設計來達成區域無從驗證，且不依存於提供讀取和寫入服務的區域。 

Azure Cosmos DB 的一致性 SLA 保證 100% 的讀取要求都將符合您所指定一致性模型 (無論是在資料庫帳戶或要求層級) 的一致性保證。 如果滿足和一致性層級相關聯的所有一致性保證，就會將讀取要求視為已符合一致性 SLA。 下表會擷取一致性模型與一致性保證。 此一致性模型中的每一個均可保證 100% 的 SLA。 

|一致性模型  | 特性  |
|---------|---------|
|強式 |  線性  |
|限定過期  |  單純讀取 (區域內)、開頭一致、過期限定 &lt; K、T   |
|工作階段  |  讀取自己的寫入、單純讀取、開頭一致    |
|一致的前置詞  | 一致的前置詞  |

### <a id="ConsistencyAndAvailability"></a>一致性和可用性的關聯性
[CAP 定理](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf) \(英文\) 的[不可達的結果](http://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf) \(英文\) 證明在發生失敗時，系統確實不可能在維持可用的同時，提供線性的一致性。 資料庫服務必須選擇 CP 或 AP，其中 CP 系統會為了線性一致性而放棄可用性，而 AP 系統則會為了可用性而放棄[線性一致性](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf) \(英文\)。 由於 Azure Cosmos DB 永遠不會違反要求的一致性模型，這使其正式成為 CP 系統。 然而實際上，一致性並不是非有及無的極端主張；在線性和最終一致性之間的一致性範圍內，還有數個定義完善的一致性模型。 在 Azure Cosmos DB 中，找出數個適用於真實世界案例且能夠直覺使用的寬鬆一致性模型。 Azure Cosmos DB 會權衡一致性和可用性之間的取捨，並針對所有單一資料庫帳戶提供[多個寬鬆但定義完善的一致性模型](consistency-levels.md)和 99.99% 的可用性，並針對所有多重區域資料庫帳戶提供 99.999% 的讀取和寫入可用性。 

### <a id="ConsistencyAndAvailability"></a>一致性和延遲的關聯性
更全面的 CAP 定理變化稱為 [PACELC](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf)，它也能解釋在穩定狀態下對延遲和一致性的權衡取捨。 它指出在穩定狀態下，資料庫系統必須在一致性和延遲之間做出選擇。 使用多個寬鬆的一致性模型 (受非同步複寫和本機讀取、寫入仲裁所支援)，Azure Cosmos DB 能確保所有讀取和寫入都會分別在讀取和寫入區域本地進行。 這可讓 Azure Cosmos DB 在指定的一致性模型的區域內提供低延遲保證。  

### <a id="ConsistencyAndThroughput"></a>一致性和輸送量的關聯性
由於實作特定一致性模型，需取決於選擇的[仲裁類型](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf) \(英文\)，輸送量也會根據一致性模型的選擇而有所不同。 例如，在 Azure Cosmos DB 中，強式一致讀取的 RU 費用大約是最終一致讀取的*兩倍*。 在此情況下，您必須佈建兩倍的 RU 才能達到相同的輸送量。 下圖顯示 Azure Cosmos DB 中特定一致性模型的讀取容量關聯性：

![一致性和輸送量之間的關聯性](./media/distribute-data-globally-turnkey/consistency-and-throughput.png)

## <a id="ThroughputGuarantees"></a>輸送量保證 
Azure Cosmos DB 可讓您視需要或需求彈性地跨任意數目的區域調整輸送量 (以及儲存體)。 下圖顯示的單一 Azure Cosmos DB 容器會進行水平分割 (跨一個區域內的三個資源分割區)，然後再跨三個 Azure 區域進行全域散發：

![Azure Cosmos DB 已散發並分割的容器](../cosmos-db/media/introduction/azure-cosmos-db-global-distribution.png)

Azure Cosmos DB 容器會在兩個維度中散發：(i) 在區域內與 (ii) 跨區域。 方式如下： 

* **本地散發**：就*資源分割區*而言，在單一區域內，Azure Cosmos DB 容器會水平向外延展。 每個資源分割區會管理一組索引鍵，而且具有強式一致性和高度可用性，因為它實際上會由四個複本 (也稱為*複本集*) 和那些複本中的狀態機器複寫所代表。 Azure Cosmos DB 是全面資源管理的系統，其中的資源分割區會依照系統所配置資源的預算來傳遞其輸送量。 Azure Cosmos DB 容器的調整對使用者而言公開透明。 Azure Cosmos DB 會管理資源分割區，並隨儲存體和輸送量需求的變更，視需要分割及合併它們。 
* **全域散發**：如果是多重區域資料庫，每個資源分割區接著會在那些區域之間散發。 擁有同一組跨區域索引鍵的資源分割區，會組成*分割集* (請參閱[上圖](#ThroughputGuarantees))。  分割集內的資源分割區，會在與該資料庫相關聯的多個區域上使用狀態機器複寫來進行協調。 根據設定的一致性層級，分割集內的資源資料分割設定為動態使用不同的拓撲 (例如，星形、菊輪鍊、樹狀目錄等)。 

透過回應相當靈敏的分割區管理、負載平衡及嚴格的資源管理，Azure Cosmos DB 可讓您彈性地調整與 Azure Cosmos DB 容器或資料庫相關聯的多個 Azure 區域輸送量。 變更佈建輸送量是 Azure Cosmos DB 中的執行階段作業。 與其他資料庫作業類似，Azure Cosmos DB 可針對變更已佈建輸送量的要求，保證其絕對延遲上限。 例如，下圖顯示的客戶容器會根據需求彈性地調整佈建輸送量 (在兩個區域之間能提供每秒 1M-10M 個要求的輸送量)。

![Azure Cosmos DB 可彈性佈建輸送量](./media/distribute-data-globally-turnkey/elastic-throughput.png)

### <a id="ThroughputAndConsistency"></a>輸送量和一致性的關聯性 
這與在[一致性和輸送量的關聯性](#ConsistencyAndThroughput)中所描述的內容相同。

### <a id="ThroughputAndAvailability"></a>輸送量和可用性的關聯性
Azure Cosmos DB 在對佈建的輸送量做出變更的情況下，仍能維持高可用性。 Azure Cosmos DB 會明確管理資源分割區 (並執行分割、合併和複製作業)，並在應用程式彈性增加或減少輸送量時，確保這些作業不會降低效能或可用性。 

## <a id="AvailabilityGuarantees"></a>可用性保證
Azure Cosmos DB 對具有寬鬆一致性的所有單一區域資料庫帳戶和所有多重區域帳戶提供 99.99% 的可用性 SLA，並對所有多重區域資料庫帳戶提供 99.999% 的可用性。 如先前所述，Azure Cosmos DB 的可用性保證包含針對每個資料面和控制面作業的延遲絕對上限。 可用性保證不會隨區域數目或區域之間的地理距離變化而有所改變。 可用性保證適用於手動及自動容錯移轉。 Azure Cosmos DB 提供透明的多路連接 API，可以確保您的應用程式能針對邏輯端點進行運作，並能在容錯移轉的情況下透明地將要求路由傳送至新的區域，或者，您可以依使用量的優先順序來指定區域性端點。 您的應用程式並不需要在進行區域性容錯移轉的情況下重新部署，並能持續維持可用性 SLA。

### <a id="AvailabilityAndConsistency"></a>可用性和一致性、延遲及輸送量的關聯性
可用性和一致性、延遲及輸送量之間的關聯性，已於[一致性和可用性的關聯性](#ConsistencyAndAvailability)、[延遲和可用性的關聯性](#LatencyAndAvailability)及[輸送量和可用性的關聯性](#ThroughputAndAvailability)等小節中描述。 

## <a id="CustomerFacingSLAMetrics"></a>客戶面向 SLA 計量
Azure Cosmos DB 會直接公開輸送量、延遲、一致性和可用性計量。 這些計量可以程式設計方式或透過 Azure 入口網站存取 (請參閱下圖)。 您也可以設定各種使用 Azure Application Insights 的臨界值警示。 下圖顯示每個租用戶都能直接取得的一致性、延遲、輸送量和可用性計量：
 
![Azure Cosmos DB 客戶可見的 SLA 計量](./media/distribute-data-globally-turnkey/customer-slas.png)

## <a id="Next Steps"></a>後續步驟

* [Azure Cosmos DB 全域散發主要優點](distribute-data-globally-benefits.md)

* [如何設定 Azure Cosmos DB 全域資料庫複寫](tutorial-global-distribution-sql-api.md)

* [如何為 Azure Cosmos DB 帳戶啟用多重主機功能](enable-multi-master.md)

* [自動化與手動容錯移轉在Azure Cosmos DB 中的運作方式](regional-failover.md)

* [了解 Azure Cosmos DB 中的衝突解決方法](multi-master-conflict-resolution.md)

* [搭配開放原始碼 NoSQL 資料庫使用多重主機](multi-master-oss-nosql.md)


## <a id="References"></a>參考
1. Eric Brewer。 [面對健全的分散式系統 (英文)](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf)
2. Eric Brewer。 [CAP 12 年後 - 規則如何變化(英文)](http://informatik.unibas.ch/fileadmin/Lectures/HS2012/CS341/workshops/reportsAndSlides/PresentationKevinUrban.pdf)
3. Gilbert、Lynch。 - [一致、可用、資料分割容錯 Web 服務的布魯爾推測和可行性](http://www.glassbeam.com/sites/all/themes/glassbeam/images/blog/10.1.1.67.6951.pdf) (英文)
4. Daniel Abadi。 [現代分散式資料庫系統設計的一致性取捨 (英文)](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-pacelc.pdf)
5. Martin Kleppmann。 [不要再以 CP 或 AP 來稱呼資料庫 (英文)](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html)
6. Peter Bailis et al。[實際部分仲裁的隨機限定過期 (PBS) (英文)](http://vldb.org/pvldb/vol5/p776_peterbailis_vldb2012.pdf)
7. Naor 和 Wool。 [仲裁系統的負載、容量及可用性 (英文)](http://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)
8. Herlihy 和 Wing。 [線性：並行物件的正確性條件 (英文)](http://cs.brown.edu/~mph/HerlihyW90/p463-herlihy.pdf)
9. [Azure Cosmos DB SLA](https://azure.microsoft.com/support/legal/sla/cosmos-db/)
