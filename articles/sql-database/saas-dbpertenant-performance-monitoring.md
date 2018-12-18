---
title: 監視多租用戶 SaaS 應用程式中許多 Azure SQL Database 的效能 | Microsoft Docs
description: 監視及管理多租用戶 SaaS 應用程式中 Azure SQL Database 和集區的效能
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: e774394eeb95fbc8d80e181a614a7e30258a100e
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056765"
---
# <a name="monitor-and-manage-performance-of-azure-sql-databases-and-pools-in-a-multi-tenant-saas-app"></a>監視及管理多租用戶 SaaS 應用程式中 Azure SQL Database 和集區的效能

本教學課程會探索 SaaS 應用程式中使用的數個關鍵效能管理案例。 使用負載產生器來跨所有租用戶資料庫模擬活動，而且示範 SQL Database 和彈性集區的內建監視和警示功能。

Wingtip Tickets SaaS Database Per Tenant 應用程式使用單一租用戶資料模型，其中每個場地 (租用戶) 都有各自的資料庫。 與許多 SaaS 應用程式類似，預期的租用戶工作負載模式無法預測且偶爾發生。 換句話說，票證銷售可能會在任何時間發生。 若要利用這種一般資料庫使用模式，租用戶資料庫要部署到彈性資料庫集區中。 彈性集區可透過跨多個資料庫共用資源，來最佳化解決方案的成本。 使用這種類型的模式，請務必監視資料庫和集區資源使用量，以確保合理地跨集區平衡負載。 您也必須確定個別的資料庫擁有足夠的資源，且集區未達到其 [eDTU](sql-database-service-tiers.md#what-are-database-transaction-units-dtus) 限制。 本教學課程將探討監視及管理資料庫與集區的方式，以及如何採取矯正措施以回應工作負載的變化。

在本教學課程中，您將了解如何：

> [!div class="checklist"]

> * 透過執行提供的負載產生器，來模擬租用戶資料庫上的使用量
> * 在租用戶資料庫回應負載增加時監視它們
> * 相應增加彈性集區，以回應增加的資料庫負載
> * 佈建第二個彈性集區以針對資料庫活動進行負載平衡


若要完成本教學課程，請確定已完成下列必要條件：

* 已部署 Wingtip Tickets SaaS Database Per Tenant 應用程式。 若要在五分鐘內完成部署，請參閱[部署及探索 Wingtip Tickets SaaS Database Per Tenant 應用程式](saas-dbpertenant-get-started-deploy.md)
* 已安裝 Azure PowerShell。 如需詳細資料，請參閱[開始使用 Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps)

## <a name="introduction-to-saas-performance-management-patterns"></a>SaaS 效能管理模式簡介

管理資料庫效能包含了編譯及分析效能資料，然後透過調整參數回應此資料，以維持您應用程式的可接受回應時間。 裝載多個租用戶時，彈性資料庫集區是一種符合成本效益的方式，可針對包含無法預測之工作負載的資料庫群組提供及管理資源。 使用特定工作負載模式，最少為兩個 S3 資料庫可以受惠於在集區中管理。

![應用程式圖表](./media/saas-dbpertenant-performance-monitoring/app-diagram.png)

您仍然需要監視集區和集區中的資料庫，以確保它們的效能在可接受範圍內。 調整集區設定以符合所有資料庫之彙總工作負載的需求，確保集區 eDTU 適用於整體工作負載。 針對您的特定應用程式需求，將每個資料庫最小和最大 eDTU 值調整為適當的值。

### <a name="performance-management-strategies"></a>效能管理策略

* 若要避免手動監視效能，最有效的方式是**設定在資料庫或集區偏離正常範圍時觸發的警示**。
* 若要回應集區彙總計算大小中的短期波動，**可以相應增加或減少集區 eDTU 層級**。 如果這樣的波動是以定期或可預測的基礎發生，則**可排定自動調整集區**。 例如，當您知道夜間或者週末期間工作負載較輕時，就可以相應減少。
* 若要回應長期波動或多個資料庫中的變更，**個別資料庫可以移至其他集區**。
* 若要回應「個別」資料庫負載中的短期增加，**可由集區中取出個別資料庫並指派個別計算大小**。 一旦負載降低後，就可以讓資料庫返回到集區。 事先知道時，則能預先移動資料庫以確保資料庫一律擁有它所需的資源，和避免影響集區中的其他資料庫。 如果可預測此需求，例如場地因熱門活動而發生票券銷售熱潮，則此管理行為可以與應用程式整合。

[Azure 入口網站](https://portal.azure.com)提供大部分資源的內建監視與警示功能。 針對 SQL Database，資料庫與集區提供監視與警示功能。 這個內建的監視與警示功能是資源特定，因此針對少數資源使用很方便，但是搭配許多資源使用時則不是很方便。

在您處理許多資源的大量案例中，可以使用 [Log Analytics](saas-dbpertenant-log-analytics.md)。 這是個別的 Azure 服務，可針對發出的診斷記錄和記錄分析工作區中收集的遙測提供分析。 Log Analytics 可以收集來自許多服務的遙測並用來查詢及設定警示。

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>取得 Wingtip Tickets SaaS Database Per Tenant 應用程式指令碼

可在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫中使用 Wingtip Tickets SaaS 多租用戶資料庫指令碼和應用程式來源程式碼。 關於下載和解除封鎖 Wingtip Tickets SaaS 指令碼的步驟，請參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)。

## <a name="provision-additional-tenants"></a>佈建其他租用戶

因為集區在只有兩個 S3 資料庫的時候符合成本效益，所以集區中有越多資料庫則平均效果越符合成本效益。 為了深入了解效能監視與管理大規模運作的方式，本教學課程要求您至少部署 20 個資料庫。

如果您已在先前的教學課程中佈建批次的租用戶，請跳至[模擬所有租用戶資料庫上的使用量](#simulate-usage-on-all-tenant-databases)一節。

1. 在 **PowerShell ISE** 中開啟 …\\Learning Modules\\Performance Monitoring and Management\\Demo-PerformanceMonitoringAndManagement.ps1。 保持此指令碼開啟，因為您將在本教學課程期間執行數個案例。
1. 設定 **$DemoScenario** = **1**，**佈建批次的租用戶**
1. 按 **F5** 以執行指令碼。

指令碼將在五分鐘內部署 17 個租用戶。

*New-TenantBatch* 指令碼會使用建立租用戶批次之 [Resource Manager](../azure-resource-manager/index.md) 範本的巢狀或連結集合，根據預設會複製類別目錄伺服器上的 **basetenantdb** 資料庫，以建立新的租用戶資料庫，然後在類別目錄中註冊這些資料庫，最後則使用租用戶名稱和場地類型將它們初始化。 這與應用程式佈建新租用戶的方式一致。 針對 *basetenantdb* 所做的任何變更會套用至之後所佈建的任何新租用戶。 請參閱[結構描述管理教學課程](saas-tenancy-schema-management.md)，以了解如何針對「現有」租用戶資料庫 (包括 basetenantdb 資料庫) 進行結構描述變更。

## <a name="simulate-usage-on-all-tenant-databases"></a>模擬所有租用戶資料庫的使用情形

提供的 Demo-PerformanceMonitoringAndManagement.ps1 指令碼會模擬對所有租用戶資料庫執行的工作負載。 使用其中一個可用的負載案例產生負載：

| 示範 | 案例 |
|:--|:--|
| 2 | 產生一般強度負載 (大約 40 個 DTU) |
| 3 | 每個資料庫產生時間更長、更頻繁高載的負載|
| 4 | 每個資料庫產生具有更高 DTU 高載的負載 (大約 80 個 DTU)|
| 5 | 在單一租用戶上產生一般負載加上高負載 (大約 95 個 DTU)|
| 6 | 產生跨多個集區的不對稱負載|

負載產生器會將僅限「綜合」 CPU 的負載套用到每一個租用戶資料庫。 產生器會針對每個租用戶資料庫啟動作業，這會定期呼叫產生負載的預存程序。 負載層級 (單位為 eDTU)、持續時間和間隔會跨所有的資料庫而變動，以模擬無法預測的租用戶活動。

1. 在 **PowerShell ISE** 中開啟 …\\Learning Modules\\Performance Monitoring and Management\\Demo-PerformanceMonitoringAndManagement.ps1。 保持此指令碼開啟，因為您將在本教學課程期間執行數個案例。
1. 設定 **$DemoScenario** = **2**，*產生一般強度負載*。
1. 按下 **F5** 將負載套用到您的所有租用戶資料庫。

Wingtip Tickets SaaS Database Per Tenant 是 SaaS 應用程式，而實際 SaaS 應用程式上的負載通常是偶爾發生且無法預測的。 為了模擬此情況，負載產生器會產生跨所有租用戶散發的隨機負載。 需要數分鐘才會出現負載模式，所以在嘗試監視以下各節中的負載時，請執行負載產生器 3-5 分鐘。

> [!IMPORTANT]
> 負載產生器正於您的本機 PowerShell 工作階段中以一系列作業的方式執行。 保持 [Demo-PerformanceMonitoringAndManagement.ps1] 索引標籤開啟！ 如果您關閉該索引標籤或暫停您的電腦，負載產生器將會停止。 負載產生器會維持 job-invoking 狀態，它會在產生器啟動後佈建的任何新租用戶上產生負載。 使用 *CTRL-C* 來停止叫用新作業並結束指令碼。 負載產生器將會繼續執行，但只在現有租用戶上執行。

## <a name="monitor-resource-usage-using-the-azure-portal"></a>使用 Azure 入口網站監視資源使用量

若要監視從已套用的負載所產生的資源使用量，請開啟入口網站並移至包含租用戶資料庫的集區：

1. 開啟 [Azure 入口網站](https://portal.azure.com)並瀏覽至 tenants1-dpt-&lt;USER&gt; 伺服器。
1. 向下捲動並找出彈性集區，然後按一下 [Pool1]。 此集區包含到目前為止所建立的所有租用戶資料庫。

觀察 [彈性集區監視] 和 [彈性資料庫監視] 圖表。

集區的資源使用量是集區中所有資料庫的資料庫使用量彙總。 資料庫圖表會顯示最熱門的 5 個資料庫︰

![資料庫圖表](./media/saas-dbpertenant-performance-monitoring/pool1.png)

因為集區中前 5 名資料庫以外還有其他資料庫，集區使用量會顯示未反映在前五個資料庫圖表中的活動。 如需其他詳細資料，請按一下 [資料庫資源使用量]：

![資料庫資源使用率](./media/saas-dbpertenant-performance-monitoring/database-utilization.png)


## <a name="set-performance-alerts-on-the-pool"></a>設定集區上的效能警示

在集區上設定 \>75% 使用率時觸發的警示，如下所示：

1. 在 [Azure 入口網站](https://portal.azure.com)中開啟 *Pool1* (在 tenants1-dpt-\<user\> 伺服器上)。
1. 按一下 [警示規則]，然後按一下 [+ 加入警示]：

   ![加入警示](media/saas-dbpertenant-performance-monitoring/add-alert.png)

1. 提供名稱，例如「高 DTU」，
1. 設定下列值：
   * **度量 = eDTU 百分比**
   * **條件 = 大於**
   * **閾值 = 75**
   * **期間 = 過去 30 分鐘內**
1. 將電子郵件地址新增至 [其他系統管理員電子郵件] 方塊，然後按一下 [確定]。

   ![設定警示](media/saas-dbpertenant-performance-monitoring/alert-rule.png)


## <a name="scale-up-a-busy-pool"></a>相應增加忙碌的集區

如果集區上的彙總負載層級增加到超出集區最大值且達到 100% eDTU 使用率，則個別資料庫效能會受到影響，可能拖慢集區中所有資料庫的查詢回應時間。

就**短期**來說，請考慮相應增加集區以提供其他資源，或從集區移除資料庫 (將資料庫移動至其他集區，或從集區移出至獨立的服務層)。

就**長期**來說，請考慮最佳化查詢或使用索引來改善資料庫效能。 根據應用程式對效能問題的敏感度，最佳做法是在集區達到 100% eDTU 使用量之前相應增加它。 使用警示來事先警告您。

您可以藉由增加產生器所產生的負載，模擬忙碌的集區。 讓資料庫高載時間更頻繁且更長，增加集區的彙總負載而不變更個別資料庫的需求。 在入口網站或從 PowerShell 可以輕鬆地相應增加集區。 此練習使用入口網站。

1. 設定 *$DemoScenario* = **3**，_每個資料庫產生時間更長、更頻繁高載的負載_，以增加集區的彙總負載強度，而不變更每個資料庫所需的尖峰負載。
1. 按下 **F5** 將負載套用到您的所有租用戶資料庫。

1. 在 Azure 入口網站中，移至 [Pool1]。

監視上方圖表上增加的集區 eDTU 使用量。 需要花數分鐘讓新的高負載開始作用，但您應該很快會看到集區開始達到最大使用率，而隨著負載穩定進入新的模式，它會快速地使集區多載。

1. 若要相應增加集區，請按一下 [Pool1] 頁面頂端的 [設定集區]。
1. 將 [集區 eDTU] 設定調整至 **100**。 變更集區 eDTU 不會變更每個資料庫設定 (仍然是每個資料庫最高 50 eDTU)。 您可以在 [設定集區] 頁面右側看到每個資料庫的設定。
1. 按一下 [儲存] 以提交調整集區的要求。

回到 [Pool1] > [概觀]，以檢視監視圖表。 監視為集區提供更多資源的影響 (雖然有一些資料庫和隨機負載，但是在您執行一段時間前，也不容易明確地看出)。 當您查看圖表時，請記住上方圖表上的 100% 現在代表 100 個 eDTU，而下方圖表上的 100% 仍是 50 個 eDTU，因為每個資料庫最大值仍是 50 個 eDTU。

整個過程中，資料庫會維持連線且完全可供使用。 最後每個資料庫都準備好以新的集區 eDTU 啟用時，任何使用中的連線都會中斷。 應用程式程式碼應該一律撰寫為重試中斷的連線，且因此重新連線至相應增加集區中的資料庫。

## <a name="load-balance-between-pools"></a>平衡集區之間的負載

做為相應增加集區的替代方案，您可以建立第二個集區並將資料庫移到其中，以平衡兩個集區之間的負載。 若要這樣做，必須在與第一個集區相同的伺服器上建立新的集區。

1. 在 [Azure 入口網站](https://portal.azure.com) 中，開啟 **tenants1-dpt-&lt;USER&gt;** 伺服器。
1. 按一下 [+ 新增集區] 以在目前的伺服器上建立集區。
1. 在 [彈性資料庫集區] 範本上︰

    1. 將 [名稱] 設定為 *Pool2*。
    1. 保留定價層為**標準集區**。
    1. 按一下 [設定集區]，
    1. 將 [集區 eDTU] 設定為 [50 eDTU]。
    1. 按一下 [新增資料庫]，以查看伺服器上可以新增至 *Pool2* 的資料庫清單。
    1. 任選 10 個資料庫以將這些資料庫移至新的集區，然後按一下 [選取]。 如果您已在執行負載產生器，該服務已經知道您的效能設定檔需要大於預設大小 50 eDTU 的較大集區，並建議從 100 eDTU 設定著手。

    ![建議](media/saas-dbpertenant-performance-monitoring/configure-pool.png)

    1. 在此教學課程中，保留預設值 50 eDTU，然後再次按一下 [選取]。
    1. 選取 [確定] 以建立新集區並將所選的資料庫移入其中。

建立集區並移動資料庫需要數分鐘的時間。 移動的資料庫會維持連線並完全可供存取直到最後一刻，屆時會關閉任何開啟的連線。 只要您有一些重試邏輯，則用戶端會連線至新集區中的資料庫。

瀏覽至 **Pool2** (在 tenants1-dpt-\<user\> 伺服器上)，以開啟集區並監視其效能。 如果您沒有看到它，請等候新的集區佈建完成。

您現在可看到 Pool1 上的資源使用量已下降，而 Pool2 目前維持同樣的負載。

## <a name="manage-performance-of-a-single-database"></a>管理單一資料庫的效能

如果集區中的單一資料庫發生持續的高負載，根據集區設定，它可能傾向於支配該集區中的資源，並影響其他資料庫。 如果活動可能會持續一陣子，則可以暫時將資料庫移出集區。 這可讓該資料庫擁有超出其所需的額外資源，並將它與其他資料庫隔離。

本練習會模擬在銷售熱門音樂會的票券時，Contoso 演藝廳發生高負載的效果。

1. 在 **PowerShell ISE** 中，開啟 …\\*Demo-PerformanceMonitoringAndManagement.ps1* 指令碼。
1. 設定 **$DemoScenario = 5，在單一租用戶上產生一般負載加上高負載 (大約 95 個 DTU)**。
1. 設定 **$SingleTenantDatabaseName = contosoconcerthall**
1. 使用 **F5** 執行指令碼。


1. 在 [Azure 入口網站](https://portal.azure.com)中，瀏覽至 *tenants1-dpt-\<user\>* 伺服器上的資料庫清單。 
1. 按一下 **contosoconcerthall** 資料庫。
1. 按一下 **contosoconcerthall** 所在的集區。 在 [彈性資料庫集區] 區段中找出集區。

1. 檢查 [彈性集區監視] 圖表，並尋找增加的集區 DTU 使用量。 在一或兩分鐘後，應該會開始產生較高的負載，且您應該會快速看到集區達到 100% 使用率。
2. 檢查 [彈性資料庫監視] 顯示，它會顯示過去一小時內最熱門的資料庫。 contosoconcerthall 資料庫應該很快會顯示為 5 個最熱門資料庫的其中之一。
3. **按一下彈性資料庫監視****圖表**，可以開啟 [資料庫資源使用量] 頁面，您可以在其中監視任何資料庫。 這可讓您找出 contosoconcerthall 資料庫的顯示。
4. 從資料庫清單中，按一下 contosoconcerthall。
5. 按一下 [定價層 (調整 DTU)] 以開啟 [設定效能] 頁面，您可以在其中設定資料庫的獨立計算大小。
6. 按一下 [標準] 索引標籤，以開啟標準層中的調整選項。
7. 將 [DTU] 滑桿向右滑動，以選取 **100** 個 DTU。 請注意，這符合服務目標 **S3**。
8. 按一下 [套用] 將資料庫移出集區，並讓它成為「標準 S3」資料庫。
9. 調整完成後，在彈性集區和資料庫刀鋒視窗上監視對 contosoconcerthall 資料庫和 Pool1 的效果。

一旦 contosoconcerthall 資料庫上的高負載下降，您應該立即讓它返回到集區，以降低其成本。 如果不確定何時會發生，您應該在資料庫上設定一個當其 DTU 使用量下降到低於集區上每個資料庫最大值時觸發的警示。 練習 5 中說明如何將資料庫移入集區。

## <a name="other-performance-management-patterns"></a>其他效能管理模式

**預先調整** 在您探索如何調整隔離資料庫的上述練習中，您知道要尋找的資料庫。 如果 Contoso 演藝廳管理已通知 Wingtips 即將發生的票證銷售，則資料庫可能已預先移出集區。 否則，它可能會需要集區或資料庫上的警示來了解發生的事件。 您不會想要從集區中其他抱怨效能降低的租用戶得知這件事。 而且如果租用戶可以預測需要其他資源多久的時間，您可以設定 Azure 自動化 Runbook 將資料庫移出集區，然後依定義的排程將它重新移回到集區中。

**租用戶自助調整** 因為調整是可以透過管理 API 輕鬆呼叫的工作，您可以輕易地在您的租用戶面向應用程式中建立調整租用戶資料庫的能力，並以 SaaS 服務的功能形式提供。 例如，讓租用戶自行管理相應增加和減少，或許會直接影響其計費！

**依排程相應增加和減少集區，以符合使用模式**

當彙總租用戶使用量遵循可預測的使用模式時，您可以使用「Azure 自動化」依排程相應增加和減少集區。 例如，當您知道資源需求會下降時，在工作日下午 6 點後相應減少集區，而在上午 6 點之前重新相應增加。



## <a name="next-steps"></a>後續步驟

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 透過執行提供的負載產生器，來模擬租用戶資料庫上的使用量
> * 在租用戶資料庫回應負載增加時監視它們
> * 相應增加彈性集區，以回應增加的資料庫負載
> * 佈建第二個彈性集區以針對資料庫活動進行負載平衡

[還原單一租用戶教學課程](saas-dbpertenant-restore-single-tenant.md)


## <a name="additional-resources"></a>其他資源

* [在 Tickets SaaS Database Per Tenant 應用程式部署上建置的其他教學課程](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [SQL 彈性集區](sql-database-elastic-pool.md)
* [Azure 自動化](../automation/automation-intro.md)
* [Log Analytics](saas-dbpertenant-log-analytics.md) - 設定及使用 Log Analytics 教學課程
