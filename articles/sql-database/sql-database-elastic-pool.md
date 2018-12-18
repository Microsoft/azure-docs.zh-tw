---
title: 使用彈性集區管理多個 SQL Database - Azure | Microsoft Docs
description: 管理及調整多個 SQL Database - 成百上千 - 使用彈性集區。 可視需要散發的資源只有一個價格。
services: sql-database
ms.service: sql-database
subservice: elastic-pool
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: oslake
ms.author: moslake
ms.reviewer: ninarn, carlrab
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: 71269b4888d1b5c9724248ac91f0818d7f8f5bf5
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162350"
---
# <a name="elastic-pools-help-you-manage-and-scale-multiple-azure-sql-databases"></a>彈性集區可協助您管理及調整多個 Azure SQL Database

SQL Database 彈性集區是簡單、符合成本效益的解決方案，可用來管理及調整使用需求變化不定且無法預測的多個資料庫。 彈性集區中的資料庫位於單一 Azure SQL Database 伺服器上，並以固定價格共用固定數量的資源。 Azure SQL Database 中的彈性集區可讓 SaaS 開發人員將一組資料庫的價格效能最佳化在規定的預算內，同時為每個資料庫提供效能彈性。

## <a name="what-are-sql-elastic-pools"></a>SQL 彈性集區是什麼？

SaaS 開發人員會在由多個資料庫組成的大規模資料層上建置應用程式。 常見的應用程式模式是為每個客戶佈建單一資料庫。 但是不同的客戶經常會有不同且無法預測的使用模式，而且很難預測每個個別資料庫使用者的資源需求。 您通常有兩個選項：

- 根據尖峰使用量額外佈建資源並額外付款，或是
- 少量佈建來節省成本，但會降低尖峰期間的效能和客戶滿意度。

彈性集區可藉由確保資料庫會在需要時取得它們所需的效能資源，來解決此問題。 它們會在可預測的預算內提供簡單的資源配置機制。 若要深入了解使用彈性集區的 SaaS 應用程式的設計模式，請參閱 [採用 Azure SQL Database 的多租用戶 SaaS 應用程式的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Elastic-databases-helps-SaaS-developers-tame-explosive-growth/player]
>

彈性集區可讓開發人員為由多個資料庫共用的集區購買資源，以容納個別資料庫無法預期的使用量期間。 您可以根據 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)或[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)，為集區設定資源。 集區的資源需求取決於其資料庫的彙總使用量。 集區可用的資源數量是由開發人員預算控制。 開發人員只需將資料庫新增至集區、設定資料庫的資源下限和上限 (不是 DTU 下限和上限就是虛擬核心下限或上限，這取決於您的資源模型選擇)，然後根據其預算設定集區的資源。 開發人員可以使用集區順暢地擴大其服務，以漸增的規模從精簡的新創公司到成熟的企業。

在集區內，會給予個別資料庫彈性以在設定的參數內自動調整。 負載量大時，資料庫可以取用更多的資源以滿足需求。 負載較輕的資料庫取用較少的資源，而完全無負載的資料庫不會取用任何資源。 針對整個集區佈建資源，而不是針對單一資料庫佈建資源，可簡化管理工作。 此外，您還可以有可預測的集區預算。 資料庫不必停機就可以在現有集區中新增額外的資源，不過可能需要移動資料庫來為新的資源保留項目提供額外的計算資源。 同樣地，如果不再需要額外資源，則隨時可以從現有集區中移除。 您也可以在集區加入或減少資料庫。 如果可以預測資料庫使用少量資源，則將它移出。

> [!NOTE]
> 將資料庫移入或移出彈性集區時，除了資料庫連線中斷時作業結尾的一小段時間之外 (大約數秒)，不會有停機時間。

## <a name="when-should-you-consider-a-sql-database-elastic-pool"></a>何時該考慮使用 SQL Database 彈性集區？

集區很適合具備特定使用模式的大量資料庫。 針對指定的資料庫，此模式的特徵是低平均使用量與相對不頻繁的使用量高峰。

您可以加入集區的資料庫愈多，可獲得的節約就愈高。 根據您的應用程式使用量模式，可能會發現與使用兩個 S3 資料庫一樣少的節約。

下列各節會協助您了解如何評估您特定的資料庫集合是否可以因為位於集區而受益。 範例會使用標準集區，但是相同的原則也適用於基本和進階的集區。

### <a name="assessing-database-utilization-patterns"></a>評估資料庫使用量模式

下圖顯示資料庫的範例，該資料庫花費太多時間閒置，但也定期因活動達到尖峰。 這是適合集區的使用量模式：

   ![適合某個集區的單一資料庫](./media/sql-database-elastic-pool/one-database.png)

針對上述的五分鐘期間，DB1 尖峰最高達 90 個 DTU，但其整體平均使用量小於 5 個 DTU。 需要 S3 計算大小，才能在單一資料庫中執行此工作負載，但這會在活動較少的期間保留大多數的資源未使用。

集區可讓這些未使用的 DTU 跨多個資料庫共用，並因此減少需要的 DTU 和整體成本。

以上一個範例為建置基礎，假設有其他資料庫具有與 DB1 類似的使用量模式。 在接下來的兩個圖形中，4 個資料庫和 20 個資料庫的使用量分層放在相同的圖形，說明使用以 DTU 為基礎的購買模型經過一段時間其使用量非重疊的本質：

   ![使用量模式適合某個集區的 4 個資料庫](./media/sql-database-elastic-pool/four-databases.png)

   ![使用量模式適合某個集區的 20 個資料庫](./media/sql-database-elastic-pool/twenty-databases.png)

在上圖中，黑色線條說明跨所有 20 個資料庫的彙總 DTU 使用量。 這顯示彙總的 DTU 使用量永遠不會超過 100 個 DTU，並指出 20 個資料庫可以在這段期間共用 100 個 eDTU。 相較於將每個資料庫放在單一資料庫的 S3 計算大小，這會導致 DTU 減少 20 倍且價格降低 13 倍。

由於以下原因，此範例很理想：

* 每一資料庫之間的尖峰使用量和平均使用量有相當大的差異。
* 每個資料庫的尖峰使用量會在不同時間點發生。
* eDTU 會在許多資料庫之間共用。

集區的價格是集區 eDTU 的函式。 雖然集區的 eDTU 單價較單一資料庫的 DTU 單價高 1.5 倍，但是**集區 eDTU 可由多個資料庫共用，而需要的 eDTU 總數會比較少**。 價格方面和 eDTU 共用的這些差異是集區可以提供價格節約潛力的基礎。

下列資料庫計數和資料庫使用量相關規則的經驗法則，可協助確保集區可提供相較於使用單一資料庫的計算大小降低的成本。

### <a name="minimum-number-of-databases"></a>資料庫的最小數目

如果單一資料庫的資源彙總數量大於集區所需資源的 1.5 倍，則彈性集區會更符合成本效益。

***以 DTU 為基礎的購買模型範例***<br>
100 個 eDTU 集區需要至少 2 個 S3 資料庫或至少 15 個 S0 資料庫，才能較使用單一資料庫的計算大小更具成本效益。

### <a name="maximum-number-of-concurrently-peaking-databases"></a>並行尖峰資料庫的最大數目

藉由共用資源，並非集區中的所有資料庫都能同時使用資源達到單一資料庫的最大限制。 同時尖峰的資料庫愈少，可以設定的集區資源愈低，集區就能更符合成本效益。 一般而言，集區中應該不能有超過 2/3 (或 67%) 的資料庫同時達到其資源限制的尖峰。

***以 DTU 為基礎的購買模型範例***<br>
為了降低 200 個 eDTU 集區中 3 個 S3 資料庫的成本，最多可以有 2 個資料庫同時到達其使用量尖峰。 否則，如果 4 個 S3 資料庫中超過 2 個同時尖峰，則必須將集區調整為超過 200 個 eDTU。 如果將集區調整大小為超過 200 個 eDTU，則需要加入更多的 S3 資料庫至集區，以使成本保持低於單一資料庫的計算大小。

請注意，此範例不考慮集區中其他資料庫的使用量。 如果所有資料庫在任何指定的時間點都有一些使用量，則可同時到達尖峰的資料庫會少於 2/3 (或 67%)。

### <a name="resource-utilization-per-database"></a>每個資料庫的資源使用量
資料庫的尖峰和平均使用量之間的差異為，長時間的低使用量和短時間的高使用量。 這個使用量模式非常適合在資料庫之間共用資源。 若資料庫的尖峰使用量為平均使用量的 1.5 倍大，就應該將該資料庫視為集區。

***以 DTU 為基礎的購買模型範例***<br>
尖峰為 100 個 DTU 且平均使用 67 個 DTU 或更少的 S3 資料庫是在集區中共用 eDTU 的良好候選項目。 或者，尖峰為 20 個 DTU 且平均使用 13 個 DTU 或更少的 S1 資料庫是集區的良好候選項目。

## <a name="how-do-i-choose-the-correct-pool-size"></a>如何選擇正確的集區大小？

集區的最佳大小取決於集區中所有資料庫所需的彙總資源。 這牽涉到決定下列各項：

* 集區中所有資料庫利用的資源上限 (DTU 上限或虛擬核心上限，這取決於您的資源模型選擇)。
* 集區中所有資料庫使用的最大儲存體位元組。

如需每個資源模型的可用服務層，請參閱 [DTU 形式的購買模型](sql-database-service-tiers-dtu.md)或[虛擬核心形式的購買模型](sql-database-service-tiers-vcore.md)。

SQL Database 會自動評估現有 SQL Database 伺服器中資料庫過去的資源使用量，並在 Azure 入口網站中建議適當的集區組態。 除了這些建議之外，內建體驗也會預估伺服器上一組自訂資料庫的 eDTU 使用量。 這可讓您以互動方式將資料庫新增至集區並加以移除，藉此進行「假設」分析，以在認可變更前取得資源使用量分析和大小建議。 如需相關作法，請參閱 [監視、管理和估算彈性集區大小](#monitor-an-elastic-pool-and-its-databases)。

在無法使用工具的情況下，下列步驟可協助您預估集區是否比單一資料庫更符合成本效益：

1. 估計集區所需的 eDTU 或虛擬核心，如下所示：

   針對以 DTU 為基礎的購買模型：最大值(<DB 總數 X 每個 DB 的平均 DTU 使用量>，<br>
   <*並行尖峰 DB 的數目* X *每個 DB 的尖峰 DTU 使用量*)

   針對以虛擬核心為基礎的購買模型：最大值(<DB 總數 X 每個 DB 的平均虛擬核心使用量>，<br>
   <*並行尖峰 DB 的數目* X *每個 DB 的尖峰虛擬核心使用量*)

2. 加總集區中所有資料庫所需的位元組數目，以估計集區所需的儲存空間。 然後判斷可提供此儲存體數量的 eDTU 集區大小。
3. 針對以 DTU 為基礎的購買模型，採用步驟 1 和步驟 2 中較大的 eDTU 估計值。 針對以虛擬核心為基礎的購買模型，採用步驟 1 中的虛擬核心估計值。
4. 請參閱 [SQL Database 價格頁面](https://azure.microsoft.com/pricing/details/sql-database/)並尋找大於步驟 3 估計值的最小集區大小。
5. 將步驟 5 的集區價格與單一資料庫適當計算大小的價格相比較。

## <a name="using-other-sql-database-features-with-elastic-pools"></a>搭配彈性集區使用其他的 SQL Database 功能

### <a name="elastic-jobs-and-elastic-pools"></a>彈性作業和彈性集區

使用集區，只要在**[彈性作業](sql-database-elastic-jobs-overview.md)** 中執行指令碼，就能簡化管理工作。 彈性作業會消除與大量資料庫相關聯的冗長工作。 若要開始，請參閱[開始使用彈性工作](sql-database-elastic-jobs-getting-started.md)。

如需可供使用多個資料庫之其他資料庫工具的詳細資訊，請參閱[使用Azure SQL Database 向上調整](sql-database-elastic-scale-introduction.md)。

### <a name="business-continuity-options-for-databases-in-an-elastic-pool"></a>彈性集區之中的資料庫所適用的業務持續性選項
集區資料庫通常會支援單一資料庫可用的相同[商務持續性功能](sql-database-business-continuity.md)。

- **還原時間點**：還原時間點會使用自動資料庫備份，將集區中的資料庫復原到特定的時間點。 請參閱 [還原時間點](sql-database-recovery-using-backups.md#point-in-time-restore)

- **異地還原**：異地還原會在資料庫因裝載區域中的事件而無法使用時，提供預設復原選項。 請參閱 [還原 Azure SQL Database 或容錯移轉到次要資料庫](sql-database-disaster-recovery.md)

- **作用中異地複寫**：針對較異地還原需要更主動復原的應用程式，設定[作用中異地複寫](sql-database-geo-replication-overview.md)。

## <a name="creating-a-new-sql-database-elastic-pool-using-the-azure-portal"></a>使用 Azure 入口網站建立新的 SQL Database 彈性集區

在 Azure 入口網站中建立彈性集區的方式有兩種。
1. 您可以在 [Marketplace] 中搜尋 **SQL 彈性集區**，或按一下 [SQL 彈性集區] 瀏覽刀鋒視窗上的 [+新增]，以建立彈性集區。 您可以透過此集區佈建工作流程來指定新的或現有的伺服器。
2. 或者，您可以瀏覽至現有 SQL 伺服器，然後按一下 [建立集區] 直接將集區建立至該伺服器，以建立彈性集區。 唯一的差別在於您略過在集區佈建工作流程期間指定伺服器的步驟。

> [!NOTE]
> 您可以在伺服器上建立多個集區，但無法將來自不同伺服器的資料庫新增到相同的集區。

集區的服務層決定了集區中彈性資料庫可用的功能，以及每個資料庫可用的資源數目上限。 如需詳細資訊，請參閱 [DTU 模型](sql-database-dtu-resource-limits-elastic-pools.md#elastic-pool-storage-sizes-and-compute-sizes)中彈性集區的資源限制。 如需彈性集區以虛擬核心為基礎的資源限制，請參閱[以虛擬核心為基礎的資源限制 - 彈性集區](sql-database-vcore-resource-limits-elastic-pools.md)。

若要設定集區的資源和定價，請按一下 [設定集區]。 然後選取服務層、將資料庫新增至集區，以及為集區及其資料庫設定資源限制。

當您完成設定集區時，您可以按一下 [套用]，為集區命名，然後按一下 [確定] 以建立集區。

## <a name="monitor-an-elastic-pool-and-its-databases"></a>監視彈性集區及其資料庫

在 Azure 入口網站中，您可以監視彈性集區與集區內資料庫的使用率。 也可以對彈性集區進行一些變更，並且一次提交所有的變更。 這些變更包括新增或移除資料庫、變更您的彈性集區設定，或變更您的資料庫設定。

若要開始監視您的彈性集區，請在入口網站中尋找並開啟彈性集區。 您會先看到一個畫面，提供您彈性集區的狀態概觀。 其中包括：

* 監視圖表會顯示彈性集區的資源使用量
* 彈性集區的最近警示和建議 (如果有的話)

下圖顯示範例彈性集區：

![集區檢視](./media/sql-database-elastic-pool-manage-portal/basic.png)

如果您想要更多有關集區的資訊，您可以在此概觀中按一下任何可用資訊。 按一下 [資源使用量] 圖表，系統將會帶您前往 [Azure 監視] 檢視，您可以在那裡自訂圖表中顯示的計量和時間範圍。 按一下任何可用的通知，系統將會帶您前往一個刀鋒視窗，其中顯示該警示或建議的完整詳細資料。

如果您想要監視集區內的資料庫，您可以按一下左側資源功能表上 [監視] 區段中的 [資料庫資源使用量]。

![資料庫資源使用率頁面](./media/sql-database-elastic-pool-manage-portal/db-utilization.png)

### <a name="to-customize-the-chart-display"></a>自訂圖表顯示

您可以編輯此圖表和 [計量] 頁面，以顯示其他計量，例如所用的 CPU 百分比、資料 IO 百分比和記錄 IO 百分比。

在 [編輯圖表] 表單上，您可以選取固定時間範圍，或按一下 [自訂] 以選取最近兩週內的任何 24 小時範圍，然後選取要監視的資源。

### <a name="to-select-databases-to-monitor"></a>選取要監視的資料庫

根據預設，[資料庫資源使用量] 刀鋒視窗中的圖表會依照 DTU 或 CPU (取決於您的服務層) 顯示前 5 名資料庫。 您可以在此圖表中切換資料庫，方法是透過左側的核取方塊，從圖表下方的清單中選取和取消選取資料庫。

您也可以選取更多計量，以在此資料庫資料表中並列檢視，進而取得更完整的資料庫效能檢視。

如需詳細資訊，請參閱[在 Azure 入口網站中建立 SQL Database 警示](sql-database-insights-alerts-portal.md)。

## <a name="next-steps"></a>後續步驟

- 若要調整彈性集區，請參閱[調整彈性集區](sql-database-elastic-pool.md)和[調整彈性集區 - 範例程式碼](scripts/sql-database-monitor-and-scale-pool-powershell.md)
* 若要觀賞影片，請參閱[有關 Azure SQL Database 彈性功能的 Microsoft Virtual Academy 視訊課程](https://mva.microsoft.com/training-courses/elastic-database-capabilities-with-azure-sql-db-16554)
* 若要深入了解使用彈性集區的 SaaS 應用程式的設計模式，請參閱 [採用 Azure SQL Database 的多租用戶 SaaS 應用程式的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。
* 如需使用彈性集區的 SaaS 教學課程，請參閱 [Wingtip SaaS 應用程式簡介](sql-database-wtp-overview.md)。
