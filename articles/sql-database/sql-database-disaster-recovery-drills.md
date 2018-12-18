---
title: SQL Database 災害復原演練 | Microsoft Docs
description: 使用 Azure SQL Database 來執行災害復原鑽研的學習指引和最佳做法。
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: business continuity
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: sashan
ms.reviewer: carlrab
ms.openlocfilehash: 52973758404faa4158afe81a92079c1acdb4cfd7
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34645458"
---
# <a name="performing-disaster-recovery-drill"></a>執行災害復原演練
建議您定期驗證應用程式的復原工作流程整備。 最佳設計作法是，驗證容錯移轉所涉及之資料遺失和 (或) 中斷的應用程式行為和影響。 這也是大多數業界標準在商務持續性認證中的規定。

災害復原演練內容包括：

* 模擬資料層中斷情況
* 復原
* 驗證復原後的應用程式完整性

執行演練的工作流程會因您 [設計商務持續性之應用程式](sql-database-business-continuity.md)的方式而異。 本文說明在 Azure SQL Database 的內容中進行災害復原演練的最佳做法。

## <a name="geo-restore"></a>異地還原
為了避免在進行災害復原演練時可能遺失資料，請使用測試環境來執行演練，方法是建立生產環境的複本，然後使用這個複本來驗證應用程式的容錯移轉工作流程。

#### <a name="outage-simulation"></a>中斷模擬
若要模擬中斷，您可以重新命名來源資料庫。 這會導致應用程式連線失敗。

#### <a name="recovery"></a>復原
* 如[此處](sql-database-disaster-recovery.md)所述，將資料庫異地還原至其他伺服器。
* 將應用程式組態變更為連接到復原資料庫，並遵循[在復原後設定資料庫](sql-database-disaster-recovery.md)指南以完成復原。

#### <a name="validation"></a>驗證
* 驗證復原後的應用程式完整性 (包括連接字串、登入、基本功能測試或標準應用程式登出程序的其他驗證部分)，完成演練。

## <a name="failover-groups"></a>容錯移轉群組
針對使用容錯移轉群組所保護的資料庫，本演練內容涵蓋規劃容錯移轉至次要伺服器。 計劃性容錯移轉可確保在切換角色時，容錯移轉群組中的主要與次要資料庫會保持同步。 與未計畫的容錯移轉不同的是，這項作業不會導致資料遺失，所以可以在生產環境中執行這項演練。

#### <a name="outage-simulation"></a>中斷模擬
若要模擬中斷，您可以停用連接到資料庫的 Web 應用程式或虛擬機器。 這會導致 Web 用戶端的連線失敗。

#### <a name="recovery"></a>復原
* 請確定 DR 區域中的應用程式組態指向先前的次要資料庫，該資料庫會變成可完全存取的新主要資料庫。
* 從次要伺服器起始容錯移轉群組的[計劃性容錯移轉](scripts/sql-database-setup-geodr-and-failover-database-powershell.md)。
* 請遵循 [在復原後設定資料庫](sql-database-disaster-recovery.md) 指南以完成復原。

#### <a name="validation"></a>驗證
驗證復原後的應用程式完整性 (包括連線能力、基本功能測試或演練登出所需的其他驗證) 來完成演練。

## <a name="next-steps"></a>後續步驟
* 若要了解商務持續性案例，請參閱[持續性案例](sql-database-business-continuity.md)。
* 若要了解 Azure SQL Database 自動備份，請參閱 [SQL Database 自動備份](sql-database-automated-backups.md)
* 若要了解如何使用自動備份進行復原，請參閱 [從服務起始的備份還原資料庫](sql-database-recovery-using-backups.md)。
* 若要了解更快速的復原選項，請參閱[作用中異地複寫和容錯移轉群組](sql-database-geo-replication-overview.md)。  
