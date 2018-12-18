---
title: 使用 Azure SQL Database 相應放大 | Microsoft Docs
description: 軟體即服務 (SaaS) 開發人員可以輕鬆地使用這些工具在雲端中建立彈性的可擴充資料庫
services: sql-database
ms.service: sql-database
ms.subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 09/14/2018
ms.openlocfilehash: ab787e8f659d2f57f23bb87397608736e7f6848a
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47164908"
---
# <a name="scaling-out-with-azure-sql-database"></a>使用 Azure SQL Database 相應放大
您可以使用 **彈性資料庫** 工具，輕鬆地向外擴充 Azure SQL 資料庫。 這些工具和功能可讓您使用 **Azure SQL Database** 中的資料庫資源，建立交易式工作負載的解決方案，尤其是軟體即服務 (SaaS) 應用程式。 彈性資料庫功能的組成如下：

* [E彈性資料庫用戶端資料庫](sql-database-elastic-database-client-library.md)：用戶端資料庫這項功能可允許您建立和維護分化資料庫。  請參閱 [開始使用彈性資料庫工具](sql-database-elastic-scale-get-started.md)。
* [彈性資料庫分割合併工具](sql-database-elastic-scale-overview-split-and-merge.md)︰在分區化資料庫之間移動資料。 在將資料從多租用戶資料庫移到單一租用戶資料庫 (反之亦然) 時，此工具十分實用。 請參閱[彈性資料庫分割合併工具教學課程](sql-database-elastic-scale-configure-deploy-split-and-merge.md)。
* [彈性資料庫工作](sql-database-elastic-jobs-overview.md) (預覽)：使用工作來管理大量的 Azure SQL 資料庫。 輕鬆執行系統管理作業，例如結構描述變更、認證管理、參考資料更新、效能資料收集，或使用作業的租用戶 (客戶) 遙測收集。
* [彈性資料庫查詢](sql-database-elastic-query-overview.md) (預覽)：可讓您跨多個 Azure SQL 資料庫執行 Transact-SQL 查詢。 這可讓您連線至報告工具，例如 Excel、Power BI、Tableau 等等。
* [彈性交易](sql-database-elastic-transactions-overview.md)︰此功能可讓您在 Azure SQL Database 中跨多個資料庫執行交易。 彈性資料庫交易適用於使用 ADO .NET 的 .NET 應用程式，而且與以往熟悉使用 [System.Transaction 類別](https://msdn.microsoft.com/library/system.transactions.aspx)的程式設計經驗整合。

下圖顯示的架構包含與資料庫集合有關的**彈性資料庫功能**。

此圖中，資料庫色彩代表結構描述。 相同色彩的資料庫共用相同的結構描述。

1. 一組 **Azure SQL 資料庫** 使用分區化架構裝載於 Azure 中。
2. **彈性資料庫用戶端程式庫** 用來管理分區集。
3. 一個資料庫子集被放入**彈性集區**中。 (請參閱[何謂集區？](sql-database-elastic-pool.md))。
4. **彈性資料庫作業**會針對所有資料庫執行排定的或 ad-hoc T-SQL 指令碼。
5. **分割合併工具**是用來將資料從某個分區移到另一個分區。
6. **彈性資料庫查詢**可讓您撰寫一個跨分區集所有資料庫的查詢。
7. **彈性交易**可讓您多個資料庫執行交易。 

![彈性資料庫工具][1]

## <a name="why-use-the-tools"></a>為何要使用這些工具？
已簡單達成雲端應用程式在 VM和 Blob 儲存體方面的彈性和縮放 - 增加或減少單位，或增加電源即可。 但對於關聯式資料庫中可設定狀態的資料處理，仍然是個挑戰。 在這些情況下出現的挑戰：

* 針對工作負載的關聯式資料庫部分放大和縮小容量。
* 管理可能會影響特定資料子集的作用區 - 例如忙碌的終端客戶 (租用戶)。

傳統上，要解決這類情況，都是經由投資更大型的資料庫伺服器來支援應用程式。 不過，此選項在雲端有其限制，因為所有處理都發生在預先定義的商用硬體上。 相反地，將資料和處理分散至許多相同結構的資料庫 (一種稱為「分區化」的相應放大模式)，不論在成本或彈性上，都是超越傳統相應增加方法的另一種選擇。

## <a name="horizontal-and-vertical-scaling"></a>水平和垂直縮放
下圖顯示縮放的水平和垂直面向，也是彈性資料庫的基本縮放方法。

![水平與垂直相應放大][2]

水平縮放指的是加入或移除資料庫來調整容量或整體效能，也稱為「相應放大」。 分區化是常用的水平縮放實作方法，主要是將資料分割到結構相同的一組資料庫上。  

垂直縮放指的是增加或減少個別資料庫的計算大小，也稱為「相應增加」。

大部分雲端級別的資料庫應用程式都採用這些兩種策略的組合。 例如，軟體即服務應用程式可能使用水平擴充來供應終端客戶，使用垂直縮放來允許每個終端客戶的資料庫隨工作負載所需而擴大或縮減資源。

* 水平縮放是透過[彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)來管理的。
* 垂直縮放是透過 Azure PowerShell Cmdlet 變更服務層，或將資料庫放入彈性集區中來達成。

## <a name="sharding"></a>分區化
「分區化」是一種將大量相同結構的資料分散在許多獨立資料庫的技術。 為一般客戶或企業建立軟體即服務 (SAAS) 供應項目的開發人員尤其愛用。 這些一般客戶通常稱為「租用戶」。 需要使用分區化可能有各種原因：  

* 資料總量太大而超出單一資料庫的條件約束
* 整體工作負載的交易輸送量超過單一資料庫的能力
* 租用戶可能需要彼此實際隔離，因此每個租用戶需要個別的資料庫
* 基於規範、效能或地理政治的理由，資料庫的不同區段可能需要位於不同的地理位置。

在其他情況下，例如從分散式裝置擷取資料，分區化可用於填滿一組暫時組織的資料庫。 例如，一個不同的資料庫可專供每日或每週使用。 在此情況下，分區化索引鍵可以是表示日期的整數 (出現在分區化資料表的所有資料列)，而擷取日期範圍資訊的查詢，必須由應用程式遞送至涵蓋相關範圍的資料庫子集。

當應用程式中的每一筆交易可以限制為單一分區化索引鍵的單一值時，分區化表現最佳。 這可確保所有交易都在特定資料庫的範圍內發生。

## <a name="multi-tenant-and-single-tenant"></a>多租用戶和單一租用戶
有些應用程式採用最簡單的方法，為每個租用戶建立個別的資料庫。 這個方法就是**單一租用戶分區化模式**，它會在租用戶的細微層級上提供隔離、備份/還原能力和資源規模調整。 使用單一租用戶分區化時，每個資料庫與特定的租用戶識別碼值 (或客戶索引鍵值) 相關聯，但該索引鍵不一定出現在資料本身。 應用程式必須負責將每個要求遞送至適當的資料庫 - 用戶端程式庫可以簡化此工作。

![單一租用戶與多租用戶][4]

其他案例將多個租用戶一起放入資料庫中，而不是將它們隔離至個別的資料庫。 這個模式就是典型的**多租用戶分區化模式**，應用程式管理大量的小型租用戶會驅使此情況。 在多租用戶分區化中，資料庫資料表中的資料列都設計成具有索引鍵 (識別租用戶識別碼) 或分區化索引鍵。 同樣地，應用程式層負責將租用戶的要求遞送至適當的資料庫，而彈性資料庫用戶端程式庫支援此功能。 此外，資料列層級安全性可用來篩選每個租用戶可以存取的資料列 - 如需詳細資訊，請參閱[使用彈性資料庫工具和資料列層級安全性的多租用戶應用程式](sql-database-elastic-tools-multi-tenant-row-level-security.md)。 使用多租用戶分區化模式時，可能需要在資料庫之間重新分配資料，而彈性資料庫分割合併工具可協助達成此工作。 若要深入了解使用彈性集區的 SaaS 應用程式的設計模式，請參閱[採用 Azure SQL Database 的多租用戶 SaaS 應用程式的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。

### <a name="move-data-from-multiple-to-single-tenancy-databases"></a>將資料從多租用戶資料庫移到單一租用戶資料庫
當建立 SaaS 應用程式時，通常會提供試用版軟體給潛在客戶。 在此情況下，使用多租用戶資料庫來處理資料較符合成本效益。 不過，當潛在客戶變成真正客戶時，單一租用戶資料庫就比較好，因為提供更好的效能。 如果客戶已在試用期間建立資料，請使用[分割合併工具](sql-database-elastic-scale-overview-split-and-merge.md) ，將資料從多租用戶資料庫移到新的單一租用戶資料庫。

## <a name="next-steps"></a>後續步驟
如需示範用戶端程式庫的範例應用程式，請參閱[開始使用彈性資料庫工具](sql-database-elastic-scale-get-started.md)。

若要將現有的資料庫轉換為使用該工具，請參閱[移轉現有的資料庫以相應放大](sql-database-elastic-convert-to-use-elastic-tools.md)。

若要查看彈性集區的細節，請參閱[彈性集區的價格和效能考量](sql-database-elastic-pool.md)，或使用[彈性集區](sql-database-elastic-pool-manage-portal.md)來建立新的集區。  

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->
[1]:./media/sql-database-elastic-scale-introduction/tools.png
[2]:./media/sql-database-elastic-scale-introduction/h_versus_vert.png
[3]:./media/sql-database-elastic-scale-introduction/overview.png
[4]:./media/sql-database-elastic-scale-introduction/single_v_multi_tenant.png

