---
title: Azure SQL Database 常見問題集 | Microsoft Docs
description: 客戶詢問有關雲端資料庫與 Azure SQL Database、Microsoft 的關聯式資料庫管理系統 (RDBMS) 以及資料庫作為雲端中服務的一般問題的解答。
services: sql-database
ms.service: sql-database
ms.subservice: ''
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 8aaf89426cf392faaf6339f49300bf7d2b7dcfcd
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166617"
---
# <a name="sql-database-faq"></a>SQL Database 常見問題集

## <a name="what-is-the-current-version-of-sql-database"></a>最新的 SQL Database 版本為何？
最新的 SQL Database 版本是 V12。 版本 V11 已被淘汰。

## <a name="what-is-the-sla-for-sql-database"></a>SQL Database 的 SLA 是什麼？
我們保證，無論您屬於哪個服務層，您的 Microsoft Azure SQL Database 可與我們的網際網路閘道正常連線的時間至少須達 99.99%。 0.01% 保留供修補、升級和容錯移轉使用。 如需詳細資訊，請參閱 [SLA](http://azure.microsoft.com/support/legal/sla/)。 如需 Azure SQL Database 可用性架構的資訊，請參閱[高可用性和 Azure SQL Database](sql-database-high-availability.md)。 

## <a name="can-i-control-when-patching-downtime-occurs"></a>我能否控制修補停機時間何時發生
否。 如果您在應用程式中[採用重試邏輯](sql-database-develop-overview.md#resiliency)，修補的影響通常不明顯。
## <a name="what-is-the-new-vcore-based-purchasing-model-for-azure-sql-database"></a>適用於 Azure SQL Database 的虛擬核心形式新購買模型為何？

新的購買模型是在以 DTU 為基礎的現有模型以外新增的模型。 以虛擬核心為基礎的模型可為客戶提供彈性、可控制、透明且直接的方法，讓他們將內部部署工作負載需求移轉到雲端。 此模型可讓客戶根據工作負載需求來調整計算和儲存資源。 使用 vCore 模型的單一資料庫和彈性集區選項可透過[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md) 來節省成本，最多可省下 30%。 如需詳細資訊，請參閱 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)和[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)。 

## <a name="what-is-a-vcore"></a>什麼是 vCore？ 
虛擬核心代表可以選擇使用的邏輯 CPU，可在各硬體世代間進行選擇。 第 4 代邏輯 CPU 以 E5-2673 v3 (Haswell) 2.4 GHz 處理器為基礎，而第 5 代邏輯 CPU 則以 Intel E5-2673 v4 (Broadwell) 2.3 GHz 處理器為基礎。

## <a name="is-moving-to-the-vcore-based-model-required"></a>一定要改用以 vCore 為基礎的模型嗎？
不一定。在彈性集區和單一資料庫部署選項中加入以 vCore 為基礎的模型，反映了我們對客戶選擇和彈性的承諾。 如果客戶想要繼續使用 DTU 架構的模型，他們可以忽略此公告，其使用和計費方式都會和原來一樣。 

在許多情況下，應用程式都可基於預先設定的資源組合所具備的簡易性而獲益。 因此，我們會繼續為客戶提供並支援這些以 DTU 為基礎的選項。 如果您正在使用此類模型，而它也符合您的業務需求，您就應該繼續使用。

以 DTU 和 vCore 為基礎的模型將繼續並存。 我們推出以虛擬核心為基礎的模型是為了回應客戶的要求，使其資料庫資源更透明，並且能夠分開調整其計算和儲存資源。 以 vCore 為基礎的模型也能讓擁有有效軟體保證的客戶透過適用於 SQL Server 的 Azure Hybrid Benefit 節省更多成本。

## <a name="how-should-i-choose-between-the-dtu-based-purchasing-model-vs-the-vcore-based-purchasing-model"></a>如何在以 DTU 為基礎的購買模型和以 vCore 為基礎的購買模型之間做出選擇？ 
資料庫交易單位 (DTU) 是以 CPU、記憶體、讀取和寫入的混合量值為基礎。 DTU 型計算大小代表已預先設定的資源組合，以期達成不同等級的應用程式效能。 如果客戶不想煩惱基礎資源的事，偏好簡單的預先設定資源組合，並且每個月支付固定金額，以 DTU 為基礎的模型可能較符合其需求。 然而，如果客戶需要深入了解基礎資源的情況，或是需要個別調整資源以達到最佳效能，以 vCore 為基礎的模型將是最佳選擇。  此外，如果客戶的 SQL Server 具有有效的軟體保證 (SA)，則運用其現有的投資，透過[適用於 SQL Server 的 Azure Hybrid Ese Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md) 節省成本，最多可達 30%。  這兩種購買模型內的選項皆提供完全受控服務的權益，例如自動備份、軟體更新和修補程式。 

## <a name="what-is-the-azure-hybrid-benefit-for-sql-server"></a>什麼是適用於 SQL Server 的 Azure Hybrid Benefit？ 
[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md) 有助您充分發揮目前授權投資的價值，並加快移轉到雲端的速度。 適用於 SQL Server 的 Azure Hybrid Benefit 是以 Azure 為基礎的權益，可讓您搭配使用 SQL Server 授權與軟體保證，以較低的費率 (「基本費率」) 支付 SQL Database。 適用於 SQL Server 的 Azure Hybrid Benefit 適用於 SQL Database 單一資料庫和彈性集區以 vCore 為基礎的購買模型的公開預覽版。 即使是使用中的 SKU，您仍可以套用此權益，但是請注意，一旦您在 Azure 入口網站中加以選取，便會開始向您收取基本費率。 點數的核發不溯及既往。

## <a name="are-there-dual-use-rights-with-azure-hybrid-benefit-for-sql-server"></a>適用於 SQL Server 的 Azure Hybrid Benefit 是否有雙重使用權？
對於此授權，您具有 180 天的雙重使用權，以確保移轉能順暢地執行。 180 天的期間結束後，SQL Server 授權就只能用於雲端的 SQL Database 中，且在內部部署和雲端中都沒有雙重使用權。

## <a name="how-does-azure-hybrid-benefit-for-sql-server-differ-from-license-mobility"></a>適用於 SQL Server 的 Azure Hybrid Benefit 與授權行動性有何不同？
我們現在為 SQL Server 客戶提供的軟體保證授權行動性權益，允許客戶將其授權重新指派給第三方共用伺服器。 此權益可用於 Azure IaaS 和 AWS EC2。
適用於 SQL Server 的 Azure Hybrid Benefit 主要在以下兩方面與授權行動性有所差異：
- 它提供將高度虛擬化工作負載移轉至 Azure 的經濟效益。 在一般目的 SKU 中，SQL EE 客戶在內部部署所擁有的每個核心，將可使其在 Azure 中獲得 4 個核心，以用於高度虛擬化應用程式。 授權行動性並不允許將虛擬化工作負載移轉至雲端的任何特殊成本優勢。
- 它是針對 Azure 上與內部部署 SQL Server 高度相容的 PaaS 目的地 (SQL Database 受控執行個體) 而提供的。

## <a name="what-are-the-specific-rights-of-the-azure-hybrid-benefit-for-sql-server"></a>適用於 SQL Server 的 Azure Hybrid Benefit 有哪些明確的權利？
SQL Database 客戶將會有下列與適用於 SQL Server 的 Azure Hybrid Benefit 相關聯的權利：

|授權情形|適用於 SQL Server 的 Azure Hybrid Benefit 對您而言有何功能？|
|---|---|
|具有 SA 的 SQL Server Enterprise Edition 核心客戶|<li>使用一般目的或業務關鍵 SKU 時可支付基本費率</li><br><li>1 個內部部署核心 = 一般目的 SKU 中的 4 個核心</li><br><li>1 個內部部署核心 = 業務關鍵 SKU 中的 1 個核心</li>|
|具有 SA 的 SQL Server Standard Edition 核心客戶|<li>只有使用一般目的 SKU 時才可支付基本費率</li><br><li>1 個內部部署核心 = 一般目的 SKU 中的 1 個核心</li>|
|||

## <a name="is-the-vcore-based-model-available-to-sql-database-managed-instance"></a>以 vCore 為基礎的模型是否適用於 SQL Database 受控執行個體？
[受控執行個體](sql-database-managed-instance.md)只能搭配使用以 vCore 為基礎的模型。 如需詳細資訊，另請參閱 [SQL Database 價格頁面](https://azure.microsoft.com/pricing/details/sql-database/managed/)。 

## <a name="does-the-cost-of-compute-and-storage-depend-on-the-service-tier-that-i-choose"></a>計算和儲存體的成本是否取決於我所選擇的服務層？
計算成本會反映已為應用程式佈建的總計算容量。 在業務關鍵服務層中，我們會自動配置至少 3 個 Always On 複本。 為了反映這項額外的計算資源配置，「業務關鍵」中的 vCore 價格大約會高出 2.7 倍。 基於相同原因，業務關鍵層中較高的每一 GB 儲存體價格，則是反映 SSD 儲存體的高 IO 和低延遲性。 同時，備份儲存體的成本並無不同，因為在這兩種情況下，我們都使用相同類別的標準儲存體。

## <a name="how-am-i-charged-for-storage---based-on-what-i-configure-upfront-or-on-what-the-database-uses"></a>我的儲存體是如何收費的 - 根據預先的設定還是資料庫的用量？
不同類型的儲存體會以不同方式計費。 對於資料儲存體，您必須根據您所選取的資料庫或集區大小上限，為佈建的儲存體支付費用。 除非您減少或增加此最大值，否則成本不會變更。 備份儲存體與您執行個體的自動備份相關聯，而且是動態配置的。 增加您的備份保留期限，即會增加執行個體所取用的備份儲存體。 備份儲存體在未達已佈建的伺服器儲存體總計的 100% 之前，無須額外付費。 備份儲存體的額外使用量，會按月以 GB 為單位收費。 例如，若您的資料庫儲存體大小為 100 GB，則在 100 GB 的備份範圍內，無需額外的費用。 但若備份達到 110 GB，則需支付額外 10 GB 的費用。 

對於單一資料庫的備份儲存體，您必須根據配置給資料庫備份的儲存體減去資料庫大小所得出的差額，按比例支付其費用。 對於彈性集區的備份儲存體，您必須根據為集區中所有資料庫的資料庫備份配置的儲存體減去彈性集區的資料大小上限所得出的差額，按比例支付其費用。 只要增加資料庫大小或彈性集區，或是增加交易速率，就需要更多儲存體，而使您的備份儲存體費用連帶提高。  當您提高資料大小上限時，這個新數量將會從計費的備份儲存體大小中扣除。

## <a name="how-do-i-select-the-right-sku-when-converting-an-existing-database-to-the-new-service-tiers"></a>將現有的資料庫轉換為新的服務層時，要如何選取正確的 SKU？ 
如果是現有的 SQL Database 應用程式，且使用以 DTU 為基礎的模型，一般目的服務層會相當於標準層。 業務關鍵服務層會相當於進階層。 在這兩種情況下，對於您的應用程式在以 DTU 為基礎的模型中使用的每 100 個 DTU，您均應配置至少 1 個 vCore。

## <a name="do-the-new-vcore-based-service-tiers-offer-the-compute-sizes-compatible-with-all-existing-compute-sizes"></a>新的 vCore 型服務層是否提供與所有現有計算大小都相容的計算大小？
vCore 型的新服務層對於所有使用 100 個或更多 DTU 的彈性集區或資料庫，都會提供相當的效能選項。  我們將持續新增更多計算大小，以支援低於 100 DTU 的工作負載。

## <a name="are-there-any-database-feature-differences-between-the-existing-dtu-based-and-new-vcore-based-service-tiers"></a>DTU 型的現有服務層和以 vCore 為基礎的新服務層之間，是否有任何資料庫功能方面的差異？ 
新的服務層支援目前以 DTU 為基礎的供應項目所提供的功能超集。 額外的功能包括一組附加的動態管理檢視 (DMV) 和附加的資源組態選項。 

## <a name="if-my-database-is-cpu-bound-and-does-not-use-much-storage-can-i-increase-the-compute-without-paying-for-extra-storage"></a>如果資料庫的效能依存於 CPU，且不會使用太多儲存體，我是否可增加計算容量而不需支付額外的儲存體費用？
是，您可以單獨選取應用程式所需的計算等級，並維持既有的儲存體等級。 儲存體最低可設為 32GB。 

## <a name="will-the-new-vcore-based-tiers-support-point-in-time-restore-pitr-for-35-days-as-today"></a>以 vCore 為基礎的新服務層是否可支援即日起 35 天內的時間點還原 (PITR)？ 
您可以為備份保留設定 7 到 35 天的 PITR。 如果實際儲存體用量超過等同於資料大小上限的儲存體數量時，將會根據該實際用量收取備份儲存體的費用。 在預覽期間，預設的 PITR 保留期限設為 7 天。 在多數情況下，資料大小上限皆足以儲存 7 天的備份。

## <a name="why-do-you-allow-selection-of-the-hardware-generation-for-compute"></a>用於計算的硬體世代為何可供選取？
我們的目標是要提供最大的彈性，讓您能夠選擇最符合應用程式需求的效能組態。 在 Gen4 硬體中，每個虛擬核心的記憶體會高出許多。 不過，Gen5 硬體可讓您大幅相應增加計算資源。 如需詳細資訊，請參閱[虛擬核心購買模型](sql-database-service-tiers-vcore.md)

## <a name="do-i-need-to-take-my-application-offline-to-convert-from-a-dtu-based-database-to-a-vcore-based-service-tier"></a>是否需要先讓應用程式離線，才能從以 DTU 為基礎的資料庫轉換為以 vCore 為基礎的服務層？ 
新的服務層提供簡單的線上轉換方法，類似現在將資料庫從標準版升級為進階版服務層的程序，反之亦然。 這項轉換可使用 Azure 入口網站、PowerShell、Azure CLI、T-SQL 或 REST API 來起始。 請參閱[管理單一資料庫](sql-database-single-database-scale.md)和[管理彈性集區](sql-database-elastic-pool.md)。

## <a name="can-i-convert-a-database-from-a-vcore-based-service-tier-to-a-dtu-based-one"></a>是否可將資料庫從以 vCore 為基礎的服務層轉換為以 DTU 為基礎的服務層？ 
是，您可以使用 Azure 入口網站、PowerShell、Azure CLI、T-SQL 或 REST API，輕鬆地將資料庫轉換為任何支援的效能目標。 請參閱[管理單一資料庫](sql-database-single-database-scale.md)和[管理彈性集區](sql-database-elastic-pool.md)。

## <a name="can-i-upgrade-or-downgrade-between-the-general-purpose-and-business-critical-service-tiers"></a>我是否可在一般目的與業務關鍵服務層之間升級或降級？ 
是，但有某些限制。 您的目的地 SKU 必須符合您為現有部署設定的資料庫或彈性集區大小上限。 如果您使用[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md)，則業務關鍵 SKU 僅可供具有 Enterprise Edition 授權的客戶使用。 只有使用適用於 SQL Server 的 Azure Hybrid Benefit 與 Enterprise Edition 授權從內部部署移轉至「一般目的」的客戶，才可升級為「業務關鍵」。 如需詳細資訊，請參閱[適用於 SQL Server 的 Azure Hybrid Use Benefit 有哪些明確的權利](../virtual-machines/windows/hybrid-use-benefit-licensing.md)？

這項轉換不會導致停止運作，並可使用 Azure 入口網站、PowerShell、Azure CLI、T-SQL 或 REST API 來起始。 請參閱[管理單一資料庫](sql-database-single-database-scale.md)和[管理彈性集區](sql-database-elastic-pool.md)。

## <a name="i-am-using-a-premium-rs-database-that-will-not-be-generally-available---can-i-upgrade-it-to-a-new-tier-and-achieve-a-similar-priceperformance-benefit"></a>我所使用的進階 RS 資料庫並不會正式推出 - 是否可將其升級至新服務層而達到類似的價格/效能優勢？
由於 vCore 模型允許您個別控制佈建的計算和儲存體數量，因此您可以更有效地管理產生的成本，這正是您使用進階 RS 資料庫的誘因。 此外，在使用以 vCore 為基礎的模型時，[適用於 SQL Server 的 Azure Hybrid Use Benefit](../virtual-machines/windows/hybrid-use-benefit-licensing.md) 會提供大幅折扣。 

## <a name="how-often-can-i-adjust-the-resources-per-pool"></a>每一集區的資源多久可調整一次？
視您所需，不限次數。 請參閱[管理彈性集區](sql-database-elastic-pool.md)。

## <a name="how-long-does-it-take-to-change-the-service-tier-or-compute-size-of-a-single-database-or-move-a-database-in-and-out-of-an-elastic-pool"></a>變更單一資料庫的服務層或計算大小，或將資料庫移入和移出彈性集區需要多久的時間？
變更資料庫的服務層和移入和移出集區需要在平台上複製資料庫做為背景作業。 視資料庫的大小而定，變更服務層可能需要數分鐘到數小時的時間不等。 在這兩種情況下，在移動期間，資料庫將維持完全連線且可供使用的狀態。 如需變更單一資料庫的詳細資訊，請參閱 [變更資料庫的服務層](sql-database-service-tiers-dtu.md)。 

## <a name="when-should-i-use-a-single-database-vs-elastic-databases"></a>何時應該使用單一資料庫與彈性資料庫？
一般而言，彈性集區是專為一般 [軟體即服務 (SaaS) 應用程式模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)而設計，其中每一客戶或租用戶會有一個資料庫。 購買個別資料庫並為了符合每個資料庫的變動尖峰需求而進行超規空間管理，通常不具成本效益。 利用集區，您可管理集區的集體效能，而資料庫會自動相應增加和相應減少。 Azure 的智慧型引擎如果發現必要的使用模式，就會對資料庫建議集區。 如需詳細資訊，請參閱[彈性集區指引](sql-database-elastic-pool.md)。

## <a name="how-does-the-usage-of-sql-database-using-the-dtu-based-purchasing-model-show-up-on-my-bill"></a>使用以 DTU 為基礎的購買模型時，我的帳單上會如何顯示 SQL Database 的使用量？
SQL Database 會根據[購買模型](sql-database-service-tiers-dtu.md)以可預期的每小時費率來計費。 實際使用量會每小時依比例分配的方式計算，因此有可能會出現不滿一小時的帳單。 例如，如果資料庫在一個月內的存在時間是 12 個小時，則您的帳單會顯示 0.5 天的使用量。 

## <a name="what-if-a-single-database-is-active-for-less-than-an-hour-or-uses-a-higher-service-tier-for-less-than-an-hour"></a>如果單一資料庫作用中的時間少於一小時，或使用更高服務層的時間少於一小時，會發生什麼情況？
您需要支付使用最高服務層資料庫存在的時數 + 在該小時適用的計算大小，不論使用狀況或資料庫是否在作用中少於一小時。 例如，假設您建立了單一資料庫並在五分鐘後刪除，您的帳單就會反映一個資料庫時數的費用。 

範例：

* 如果您建立 Basic 資料庫，並立即將它升級到 Standard S1，則我們會以 Standard S1 的費率向您收取第一個小時的費用。
* 如果您在下午 10:00 將資料庫從「基本」升級為「進階」， 在隔天上午 1:35 完成升級， 則會從上午 1:00 開始「進階」費率計費。 
* 如果您在上午 11:00 將資料庫從「進階」降級為「基本」， 且該作業在下午 2:15 完成，則會針對資料庫以「進階」費率收費到下午 3:00，之後便以「基本」費率收費。

## <a name="how-does-elastic-pool-usage-using-the-dtu-based-purchasing-model-show-up-on-my-bill"></a>使用以 DTU 為基礎的購買模型時，我的帳單上會如何顯示彈性集區使用量？
在您的帳單上，彈性集區收費會顯示為彈性 DTU (eDTU) 或 vCore 和儲存體，並採用[價格頁面](https://azure.microsoft.com/pricing/details/sql-database/)上顯示的增量。 彈性集區沒有每一資料庫的費用。 對於集區存在的每個小時，您均需要支付最高的 eDTU 或 vCore，不論使用量多高，或使用集區的時間是否少於一小時。 

以 DTU 為基礎的購買模型範例：

* 如果您在上午 11:18 以 200 個 eDTU 建立「標準」彈性集區，將五個資料庫加入至集區，則會向您收取該小時 200 個 eDTU 的費用，從上午 11:00 開始。 到當天剩餘的時間。
* 在第 2 天上午 5:05，資料庫 1 開始取用 50 eDTU 並穩定持有一天。 資料庫 2-5 會在 0 和 80 eDTU 之間波動。 在當天，您加入全天取用不同 eDTU 的其他五個資料庫。 第 2 天是全天，以 200 eDTU 計費。 
* 在第 3 天上午 5:00， 您加入另外 15 個資料庫。 資料庫使用量全天增加，到下午 8:05 您決定將集區的 eDTU 從 200 增加為 400。 以 200 個 eDTU 等級收費的有效期間是到下午 8:00，並針對其餘的四小時增加至 400 個 eDTU。 

## <a name="how-are-elastic-pool-billed-for-the-dtu-based-purchasing-model"></a>使用以 DTU 為基礎的購買模型時，彈性集區會如何計費？
彈性集區會依據下列特性計費：

* 彈性集區在建立時就會開始計費，即使集區中沒有資料庫也一樣。
* 彈性集區會以每小時計費。 此計量頻率和單一資料庫計算大小的相同。
* 如果彈性集區的大小經過調整，則在調整作業完成之前，將不會根據新的資源數量來計算集區費用。 這樣可允許和變更單一資料庫計算大小之模式相同的模式。
* 彈性集區的價格是以集區資源作為計算基礎。 彈性集區的價格與其中之彈性資料庫的數目和使用量無關。

如需詳細資訊，請參閱 [SQL Database 價格](https://azure.microsoft.com/pricing/details/sql-database/)、[DTU 形式的購買模型](sql-database-service-tiers-dtu.md)和[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)。

## <a name="how-does-the-vcore-based-usage-show-up-in-my-bill"></a>我的帳單中會如何顯示以 vCore 為基礎的使用量？ 
在以 vCore 為基礎的模型中，服務的計費會根據服務層、vCore 中佈建的計算、佈建的儲存體 (GB/月)、取用的備份儲存體，以可預測的每小時費率計費。 如果備份的儲存體超過資料庫大小總計 (也就是資料庫大小的 100%)，就會產生額外費用。 vCore 時數、設定的資料庫儲存體、取用的 IO 和備份儲存體都會在帳單中清楚列出來，讓您更容易詳盡查看所使用的資源。 內含最高可達資料庫大小上限 100% 的備份儲存體，超過此容量時，將以一個月使用的量 (GB/月) 計費。

例如︰
- 如果一個月裡 SQL 資料庫存在達 12 小時，則帳單會顯示 12 個小時的 vCore 使用量。 如果 SQL 資料庫額外佈建了 100 GB 的儲存體，則帳單會依時數比例顯示儲存體使用量 (以 GB/月為單位) 以及一個月內的 IO 使用量。
- 如果 SQL 資料庫使用時間不到一小時，則會針對資料庫存在的每個小時，使用所選取的最高服務層及該小時內所套用的佈建儲存體和 IO 來計費，不論使用量或資料庫使用時間是否不到一小時。
- 若在建立受控執行個體的 5 分鐘之後才將其刪除，則需支付一個小時的資料庫時間。
- 若您在一般目的層內建立有 8 個 vCore 的受控執行個體，然後立即將其升級為 16 個 vCore，則第一小時需支付 16 個 vCore 的費率。

> [!NOTE]
> 限時提供免費的備份與 IO。

## <a name="how-does-the-use-of-active-geo-replication-in-an-elastic-pool-show-up-on-my-bill"></a>在彈性集區中使用作用中異地複寫會在我的帳單上如何顯示？
與單一資料庫不同，搭配彈性資料庫使用[作用中異地複寫](sql-database-geo-replication-overview.md)對計費並沒有直接的影響。  您只需要為對每個集區 (主要集區和次要集區) 佈建的資源付費

## <a name="how-does-the-use-of-the-auditing-feature-impact-my-bill"></a>使用稽核功能會如何影響帳單？
稽核內建在 SQL Database 服務中供免費使用，並且可在所有服務層上使用。 但是，為了儲存稽核記錄檔，稽核功能會使用 Azure 儲存體帳戶，而 Azure 儲存體的費率資料表和佇列 會根據稽核記錄檔的大小套用。

## <a name="how-do-i-reset-the-password-for-the-server-admin"></a>如何重設伺服器系統管理員的密碼？
在 [Azure 入口網站](https://portal.azure.com)中，按一下 [SQL Server]、從清單中選取伺服器，然後按一下 [重設密碼]。

## <a name="how-do-i-manage-databases-and-logins"></a>如何管理資料庫與登入？
請參閱[管理資料庫與登入](sql-database-manage-logins.md)。 

> [!NOTE]
> 當伺服器系統管理員帳戶建立之後，即無法變更。

## <a name="how-do-i-make-sure-only-authorized-ip-addresses-are-allowed-to-access-a-server"></a>如何確保只有授權的 IP 位址可以存取伺服器？
請參閱 [如何：在 SQL Database 上進行防火牆設定](sql-database-configure-firewall-settings.md)。

## <a name="what-is-an-expected-replication-lag-when-geo-replicating-a-database-between-two-regions-within-the-same-azure-geography"></a>在兩個 Azure 地理位置相同的區域之間，針對資料庫進行異地複寫時，預期的複寫延遲是多久？
我們目前支援五秒的 RPO，而且當地區次要資料庫裝載於 Azure 建議的配對區域中且位於同一服務層，複寫延遲便會小於五秒。

## <a name="what-is-an-expected-replication-lag-when-geo-secondary-is-created-in-the-same-region-as-the-primary-database"></a>若地區次要資料庫建立於主要資料庫相同的區域中，預期的複寫延遲是多少？
根據實證資料，若使用 Azure 建議配對區域，區域內部和區域之間的複寫延遲不會有太大差異。 

## <a name="if-there-is-a-network-failure-between-two-regions-how-does-the-retry-logic-work-when-geo-replication-is-set-up"></a>如果兩個區域之間網路故障，在已設定異地複寫的情況下，重試邏輯如何運作？
如果中斷連線，我們會每隔 10 秒重試，以便重新建立連線。

## <a name="what-can-i-do-to-guarantee-that-a-critical-change-on-the-primary-database-is-replicated"></a>我該如何保證主要資料庫上的重大變更能夠確實複寫？
地區次要資料庫為非同步複本，我們不會嘗試將該資料庫與主要資料庫維持完整同步。 但是，我們會提供方法來強制進行同步處理，以確保重大變更 (例如，密碼更新) 的複寫。 強制進行同步處理會對效能產生影響，因為它會封鎖呼叫執行緒，直到所有認可的交易都完成複寫為止。 如需詳細資訊，請參閱 [sp_wait_for_database_copy_sync](https://msdn.microsoft.com/library/dn467644.aspx)。 

## <a name="what-tools-are-available-to-monitor-the-replication-lag-between-the-primary-database-and-geo-secondary"></a>若要監視主要資料庫和地區資料庫之間的複寫延遲，有哪些工具可以使用？
我們會透過 DMV 將主要資料庫和地區次要資料庫之間的即時複寫延遲公開。 如需詳細資訊，請參閱 [sys.dm_geo_replication_link_status](https://msdn.microsoft.com/library/mt575504.aspx)。

## <a name="to-move-a-database-to-a-different-server-in-the-same-subscription"></a>將資料庫移到相同訂用帳戶中的不同伺服器
在 [Azure 入口網站](https://portal.azure.com)中，按一下 [SQL Database]、從清單中選取資料庫，然後按一下 [複製]。 如需詳細資訊，請參閱 [複製 Azure SQL Database](sql-database-copy.md) 。

## <a name="to-move-a-database-between-subscriptions"></a>在訂用帳戶之間移動資料庫
在 [Azure 入口網站](https://portal.azure.com)中，按一下 [SQL Server]，然後從清單中選取裝載您資料庫的伺服器。 按一下 [移動] ，然後挑選要移動的資源以及要移入的訂用帳戶。

