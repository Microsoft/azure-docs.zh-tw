---
title: 威脅偵測 - Azure SQL Database | Microsoft Docs
description: 威脅偵測會偵測異常資料庫活動，指出單一資料庫或彈性集區中的資料庫有潛在的安全性威脅。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto, carlrab
manager: craigg
ms.date: 09/19/2018
ms.openlocfilehash: ed7a117d4422b3b3feddd7d2fafa5eabea43f9e5
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47164932"
---
# <a name="azure-sql-database-threat-detection"></a>Azure SQL Database 威脅偵測

Azure SQL Database 威脅偵測會偵測意圖存取或攻擊資料庫，並可能會造成損害的異常活動。

威脅偵測是 [SQL 進階威脅防護](sql-advanced-threat-protection.md) (ATP) 供應項目的一部分，該供應項目是進階 SQL 安全性功能的整合套件。 威脅偵測可以透過中央 SQL ATP 入口網站存取及管理。

## <a name="what-is-threat-detection"></a>什麼是威脅偵測？

SQL 威脅偵測提供新的一層安全性，在發生異常活動時會提供安全性警示，讓客戶偵測並回應潛在威脅。 一旦有可疑活動、潛在弱點、SQL 插入式攻擊以及異常的資料庫存取和查詢模式發生時，使用者就會收到警示。 SQL 威脅偵測會整合警示與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)，其包括可疑活動的詳細資料，以及如何調查與降低威脅的建議。 您不必是安全性專家，也不需要管理進階的安全性監視系統，SQL 威脅偵測可讓您輕鬆解決資料庫的潛在威脅。 

如需完整的調查體驗，建議您啟用 [SQL Database 稽核](sql-database-auditing.md)，這會將資料庫事件寫入您 Azure 儲存體帳戶中的稽核記錄。  

## <a name="set-up-threat-detection-for-your-database-in-the-azure-portal"></a>使用 Azure 入口網站為資料庫設定威脅偵測
1. 在 [https://portal.azure.com](https://portal.azure.com) 上啟動 Azure 入口網站。
2. 瀏覽至所要保護的 Azure SQL Database 伺服器的組態頁面。 在 [安全性] 設定中，選取 [進階威脅防護]。
3. 在 [進階威脅防護] 組態頁面上：

   - 在伺服器上啟用進階威脅防護。
   - 在 [威脅偵測設定] 的 [傳送警示給] 文字方塊中，提供要在偵測到異常資料庫活動時收到安全性警示的電子郵件清單。
  
   ![設定威脅偵測](./media/sql-database-threat-detection/set_up_threat_detection.png)

## <a name="set-up-threat-detection-using-powershell"></a>使用 PowerShell 設定威脅偵測

如需指令碼範例，請參閱[使用 PowerShell 設定稽核與威脅偵測](scripts/sql-database-auditing-and-threat-detection-powershell.md)。

## <a name="explore-anomalous-database-activities-upon-detection-of-a-suspicious-event"></a>偵測到可疑事件時探索異常資料庫活動

偵測到異常資料庫活動時，您會收到電子郵件通知。 電子郵件會提供可疑安全性事件的相關資訊，包括異常活動的性質、資料庫名稱、伺服器名稱、應用程式名稱和事件時間。 此外，該電子郵件還會提供可能原因和建議動作的相關資訊，以協助您調查和減輕資料庫的潛在威脅。

![異常活動報告](./media/sql-database-threat-detection/anomalous_activity_report.png)
     
1. 按一下電子郵件中的 [檢視最近的 SQL 警示] 連結來啟動 Azure 入口網站，並顯示 Azure 資訊安全中心警示頁面，其中會概述在 SQL 資料庫上偵測到的作用中威脅。

   ![活動威脅](./media/sql-database-threat-detection/active_threats.png)

2. 按一下特定警示可取得其他詳細資料和調查此威脅的建議，並對未來的威脅採取補救措施。

   例如，SQL 插入式攻擊是網際網路上最常見的 Web 應用程式安全性問題之一，用於攻擊資料導向應用程式。 攻擊者利用應用程式弱點將惡意的 SQL 陳述式插入應用程式輸入欄位，破壞或修改資料庫中的資料。 針對 SQL 插入式攻擊，警示的詳細資料會包括已遭利用且有弱點的 SQL 陳述式。

   ![特定警示](./media/sql-database-threat-detection/specific_alert.png)

## <a name="explore-threat-detection-alerts-for-your-database-in-the-azure-portal"></a>在 Azure 入口網站中探索資料庫的威脅偵測警示

SQL Database 威脅偵測將自有的警示與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)整合。 在 Azure 入口網站中，資料庫和 SQL ATP 刀鋒視窗內的 SQL 動態威脅偵測圖格會追蹤作用中威脅的狀態。

按一下 [威脅偵測警示] 會啟動 Azure 資訊安全中心的警示頁面，並獲得在資料庫中偵測到的作用中 SQL 威脅概觀。

   ![威脅偵測警示](./media/sql-database-threat-detection/threat_detection_alert.png)
   
   ![威脅偵測警示 2](./media/sql-database-threat-detection/threat_detection_alert_atp.png)

## <a name="azure-sql-database-threat-detection-alerts"></a>Azure SQL Database 威脅偵測警示 
Azure SQL Database 的威脅偵測會偵測意圖存取或攻擊資料庫，而可能會造成損害的異常活動，並觸發下列警示：
- **SQL 插入的弱點**：應用程式在資料庫中產生錯誤的 SQL 陳述式時，會觸發此警示。 此警示表示 SQL 插入式攻擊的可能弱點。 錯誤的陳述式之所以產生，有兩項可能的原因：
   - 應用程式的程式碼中有缺失，而建構了錯誤的 SQL 陳述式
   - 應用程式的程式碼或預存程序在建構錯誤的 SQL 陳述式時未處理使用者輸入，這可能會遭到 SQL 插入式攻擊的侵害
- **可能的 SQL 插入**：在 SQL 插入的已知應用程式弱點遭到作用中的攻擊時，會觸發此警示。 這表示有攻擊者嘗試使用有弱點的應用程式程式碼或預存程序插入惡意 SQL 陳述式。
- **從不尋常的位置存取**：有人從不尋常的地理位置登入 SQL Server，而使 SQL Server 的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (新的應用程式或開發人員維護)。 在其他情況下，警示則是偵測惡意動作 (離職員工、外部攻擊者)。
- **從不尋常的 Azure 資料中心存取**：有人從不尋常的 Azure 資料中心登入 SQL Server (在近期曾在此伺服器上發現此資料中心)，而使 SQL Server 的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (您在 Azure、Power BI、Azure SQL 查詢編輯器中使用新的應用程式)。 在其他情況下，警示則是偵測來自 Azure 資源/服務的惡意動作 (離職員工、外部攻擊者)。
- **從不熟悉的主體存取**：有人使用不尋常的主體 (SQL 使用者) 登入 SQL Server，而使 SQL Server 的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (新的應用程式、開發人員維護)。 在其他情況下，警示則是偵測惡意動作 (離職員工、外部攻擊者)。
- **從可能有害的應用程式存取**：使用可能有害的應用程式用存取資料庫時，會觸發此警示。 在某些情況下，警示會偵測到執行中的滲透測試。 在其他情況下，警示則是偵測使用常見攻擊工具的攻擊。
- **暴力 SQL 認證**：有使用不同認證的異常大量登入失敗時，會觸發此警示。 在某些情況下，警示會偵測到執行中的滲透測試。 在其他情況下，警示則是偵測暴力攻擊。

## <a name="next-steps"></a>後續步驟

* 深入了解 [SQL 進階威脅防護](sql-advanced-threat-protection.md)。 
* 深入了解 [Azure SQL Database 稽核](sql-database-auditing.md)
* 深入了解 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)
* 如需有關價格的詳細資訊，請參閱 [SQL Database 價格頁面](https://azure.microsoft.com/pricing/details/sql-database/)  
