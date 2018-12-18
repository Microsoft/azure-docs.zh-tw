---
title: 輪流應用程式升級 - Azure SQL Database | Microsoft Docs
description: 了解如何使用 Azure SQL Database 異地複寫以支援雲端應用程式的線上升級。
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: business continuity
ms.topic: conceptual
ms.date: 08/23/2018
ms.author: sashan
ms.openlocfilehash: 37960995c89c2b30d90ac45dcd8cc44d80088398
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818611"
---
# <a name="managing-rolling-upgrades-of-cloud-applications-using-sql-database-active-geo-replication"></a>使用 SQL Database 主動式異地複寫管理雲端應用程式的輪流升級
> [!NOTE]
> [作用中異地複寫](sql-database-geo-replication-overview.md)現在可供所有層中的所有資料庫使用。
> 

了解如何使用 SQL Database 中的[異地複寫](sql-database-geo-replication-overview.md)來啟用雲端應用程式的輪流升級。 因為升級是干擾性作業，它應該是您商務持續性規劃與設計的一部分。 在本文中，我們將探討兩種不同的升級程序協調方法，並討論每個選項的優點和取捨。 基於本文的目的，我們將使用簡單的應用程式，它是由連線到單一資料庫作為其資料層的網站所組成。 我們的目標是將應用程式的版本 1 升級到版本 2，而不對使用者體驗造成顯著的影響。 

評估升級選項時，您應該考慮下列因素：

* 升級期間對應用程式可用性的影響。 應用程式功能可能受到限制或降級多久時間。
* 防止萬一升級失敗的復原功能。
* 升級期間發生不相關的重大失敗時，應用程式的弱點。
* 總金額成本。  這包括額外備援以及升級程序所使用之暫時性元件的累加成本。 

## <a name="upgrading-applications-that-rely-on-database-backups-for-disaster-recovery"></a>升級依賴資料庫備份以進行災害復原的應用程式
如果您的應用程式依賴自動資料庫備份，並針對災害復原使用異地復原，則它通常是部署到單一 Azure 區域。 在此案例中，升級程序涉及建立升級中相關的所有應用程式元件的備份部署。 若要對使用者的干擾降到最低，您要利用 Azure 流量管理員 (ATM) 與容錯移轉設定檔。  下圖說明升級程序之前的作業環境。 端點 <i>contoso-1.azurewebsites.net</i> 代表需要升級之應用程式的生產位置。 若要啟用復原升級的功能，您必須建立含有應用程式完整同步之複本的預備位置。 準備升級應用程式需要執行下列步驟：

1. 為升級建立預備位置。 若要這樣做，請建立次要資料庫 (1) 並在相同的 Azure 區域中部署相同的網站。 監視次要資料庫，查看植入程序是否完成。
2. 在 ATM 中建立容錯移轉設定檔，<i>contoso-1.azurewebsites.net</i> 作為線上端點，<i>contoso-2.azurewebsites.net</i> 作為離線端點。 

> [!NOTE]
> 請注意，準備步驟不會影響生產位置中的應用程式，且它可以在完整存取模式下運作。
>  

![SQL Database「異地複寫」組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option1-1.png)

一旦準備步驟完成後，應用程式就已準備好實際升級。 下圖說明升級程序相關的步驟。 

1. 將生產位置中的主要資料庫設定為唯讀模式 (3)。 這樣會保證應用程式的生產執行個體 (V1) 在升級期間會保持唯讀，藉此防止 V1 與 V2 資料庫執行個體間的資料分歧。  
2. 使用規劃的終止模式 (4) 中斷次要資料庫的連線。 它會建立完整同步的主要資料庫獨立複本。 此資料庫將會升級。
3. 在預備位置 (5) 中將主要資料庫切換為讀寫模式，然後執行升級指令碼。     

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option1-2.png)

如果升級順利完成，您現在已準備好將使用者切換至應用程式的預備複本。 它現在將會成為應用程式的生產位置。  這涉及一些步驟，如下圖所示。

1. 將 ATM 設定檔中的線上端點切換為 <i>contoso-2.azurewebsites.net</i>，它會指向 V2 版本的網站 (6)。 它現在已成為含有 V2 應用程式的生產位置，而且使用者流量會導向至它。  
2. 如果您不再需要 V1 應用程式元件，您可以安全地移除它們 (7)。   

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option1-3.png)

如果升級程序失敗 (例如升級指令碼中發生錯誤)，可能要將預備位置視為遭到危害。 若要將應用程式復原到升級前狀態，您只需將生產位置中的應用程式還原至完整存取。 下圖顯示相關的步驟。    

1. 將資料庫複本設定為讀寫模式 (8)。 這會還原生產位置中完整的 V1 功能。
2. 執行根本原因分析，然後移除預備位置 (9) 中遭到危害的元件。 

此時應用程式已經可以完全運作，並且可以重複升級步驟。

> [!NOTE]
> 復原不需要變更 ATM 設定檔，因為它已經指向 <i>contoso-1.azurewebsites.net</i> 作為作用中端點。
> 
> 

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option1-4.png)

這個選項的重要**優點**是您可以使用一組簡單步驟升級單一區域中的應用程式。 升級的金額成本相對較低。 主要的 **取捨** 是如果在升級期間發生嚴重失敗，復原到升級前狀態會需要將不同區域的應用程式重新部署，並使用異地復原從備份還原資料庫。 此程序將產生顯著的停機時間。   

## <a name="upgrading-applications-that-rely-on-database-geo-replication-for-disaster-recovery"></a>升級依賴資料庫異地複寫來進行災害復原的應用程式
如果您的應用程式利用異地複寫來提供商務持續性，則它至少會部署到兩個不同區域，其中作用中部署會在「主要」區域，而待命部署則會在「備份」區域。 除了先前提到的因素之外，升級程序也必須保證：

* 升級程序期間，應用程式隨時受到保護以防嚴重失敗。
* 應用程式的異地備援元件會與作用中元件以平行方式升級

若要達成這些目標，您要利用 Azure 流量管理員 (ATM)，使用含有一個作用中端點以及三個備份端點的容錯移轉設定檔。  下圖說明升級程序之前的作業環境。 網站 <i>contoso-1.azurewebsites.net</i> 和 <i>contoso-dr.azurewebsites.net</i> 代表具有完整異地備援之應用程式的生產位置。 若要啟用復原升級的功能，您必須建立含有應用程式完整同步之複本的預備位置。 因為您必須確保應用程式可以快速復原，以防在升級程序期間發生嚴重失敗，因此預備位置也需要異地備援。 準備升級應用程式需要執行下列步驟：

1. 為升級建立預備位置。 若要這樣做，請建立次要資料庫 (1) 並在相同的 Azure 區域中部署相同的網站複本。 監視次要資料庫，查看植入程序是否完成。
2. 將次要資料庫異地複寫至備份區域 (這裡稱為「鏈結異地複寫」)，在預備位置建立異地備援次要資料庫。 監視備份次要資料庫，查看植入程序是否完成 (3)。
3. 在備份區域中建立網站待命複本，然後將它連結至異地備援次要資料庫 (4)。  
4. 將額外的端點 <i>contoso-2.azurewebsites.net</i> 和 <i>contoso-3.azurewebsites.net</i> 新增到 ATM 中的容錯移轉設定檔作為離線端點 (5)。 

> [!NOTE]
> 請注意，準備步驟不會影響生產位置中的應用程式，且它可以在完整存取模式下運作。
> 
> 

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option2-1.png)

一旦準備步驟完成後，預備位置就準備好升級。 下圖說明升級步驟。

1. 將生產位置中的主要資料庫設定為唯讀模式 (6)。 這樣會保證應用程式的生產執行個體 (V1) 在升級期間會保持唯讀，藉此防止 V1 與 V2 資料庫執行個體間的資料分歧。  
2. 使用規劃的終止模式 (7) 中斷相同區域內次要資料庫的連線。 它會建立主要資料庫完整同步的獨立複本，它會在終止後自動成為主要資料庫。 此資料庫將會升級。
3. 將預備位置中的主要資料庫切換為讀寫模式，然後執行升級指令碼 (8)。    

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option2-2.png)

如果升級順利完成，您現在已準備好將使用者切換至應用程式的 V2 版本。 下圖說明相關步驟。

1. 將 ATM 設定檔中的作用中端點切換為 <i>contoso-2.azurewebsites.net</i>，它現在會指向 V2 版本的網站 (9)。 它現在已成為 V2 應用程式的生產位置，而且使用者流量會導向至它。 
2. 如果您不再需要 V1 應用程式，您可以安全地移除它 (10 和 11)。  

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option2-3.png)

如果升級程序失敗 (例如升級指令碼中發生錯誤)，可能要將預備位置視為遭到危害。 若要將應用程式復原到升級前狀態，您只需以完整存取使用生產位置中的應用程式即可復原。 下圖顯示相關的步驟。    

1. 將生產位置中的主要資料庫複本設定為讀寫模式 (12)。 這會還原生產位置中完整的 V1 功能。
2. 執行根本原因分析，然後移除預備位置 (13 和 14) 中遭到危害的元件。 

此時應用程式已經可以完全運作，並且可以重複升級步驟。

> [!NOTE]
> 復原不需要變更 ATM 設定檔，因為它已經指向 <i>contoso-1.azurewebsites.net</i> 作為作用中端點。
> 
> 

![SQL Database 異地複寫組態。 雲端災害復原。](media/sql-database-manage-application-rolling-upgrade/Option2-4.png)

這個選項的重要 **優點** 是您可以以平行方式升級應用程式及其異地備援複本，而不會在升級期間危及商務持續性。 主要的 **取捨** 是它需要每個應用程式元件的雙重備援，因此會產生較高的金額成本。 它也涉及更複雜的工作流程。 

## <a name="summary"></a>總結
本文中所描述的兩種升級方法有不同的複雜度與金額成本，但它們都著重於將使用者受限為唯讀作業的時間降到最低。 該時間由升級指令碼的期間直接定義。 時間不會取決於資料庫大小、您選擇的服務層，以及網站設定和其他您無法輕鬆控制的因素。 這是因為所有的準備步驟與升級步驟分離，並且不需影響實際執行應用程式即可完成。 升級指令的效率是決定升級期間使用者的體驗的重要因素。 因此您改進效率的最佳方式，是把工作焦點放在盡可能提高升級指令碼的效率。  

## <a name="next-steps"></a>後續步驟
* 如需商務持續性概觀和案例，請參閱 [商務持續性概觀](sql-database-business-continuity.md)。
* 若要了解 Azure SQL Database 自動備份，請參閱 [SQL Database 自動備份](sql-database-automated-backups.md)。
* 若要了解如何使用自動備份進行復原，請參閱[從自動備份還原資料庫](sql-database-recovery-using-backups.md)。
* 若要了解更快速的復原選項，請參閱[主動式異地複寫](sql-database-geo-replication-overview.md)。


