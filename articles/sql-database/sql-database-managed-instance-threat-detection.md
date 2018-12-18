---
title: 威脅偵測 - Azure SQL Database 受控執行個體 | Microsoft Docs
description: 威脅偵測會偵測異常資料庫活動，指出受控執行個體中的資料庫有潛在的安全性威脅。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: rmatchoro
ms.author: ronmat
ms.reviewer: vanto
manager: craigg
ms.date: 09/19/2018
ms.openlocfilehash: 7f05bd6c96a8e22b1e039c3edcec491b7c079d54
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162404"
---
# <a name="azure-sql-database-managed-instance-threat-detection"></a>Azure SQL Database 受控執行個體威脅偵測

SQL 威脅偵測會偵測意圖存取或惡意探索 Azure SQL Database 受控執行個體中的資料庫，並可能會造成損害的異常活動。

## <a name="overview"></a>概觀

威脅偵測會偵測異常資料庫活動，指出受控執行個體有潛在的安全性威脅。 受控執行個體的威脅偵測功能目前是預覽狀態。

威脅偵測提供新的一層安全性，在發生異常資料庫活動時會提供安全性警示，讓客戶偵測並回應潛在威脅。 您不必是安全性專家，也不需要管理進階的安全性監視系統，威脅偵測讓您輕鬆解決受控執行個體的潛在威脅。 如需完整的調查體驗，建議您啟用 Azure 受控執行個體稽核，這會將資料庫事件寫入您 Azure 儲存體帳戶中的稽核記錄。 

SQL 威脅偵測整合了警示與 [Azure 資訊安全中心](https://azure.microsoft.com/services/security-center/)。每個受保護受控執行個體的收費與 Azure 資訊安全中心標準層相同，全部是每月每個節點 $15，其中每個受保護的受控執行個體各會計為一個節點。  

## <a name="set-up-threat-detection-for-your-managed-instance-in-the-azure-portal"></a>使用 Azure 入口網站為受控執行個體設定威脅偵測
1. 在 [https://portal.azure.com](https://portal.azure.com) 上啟動 Azure 入口網站。
2. 瀏覽至您要保護的受控執行個體組態頁面。 在 [設定] 頁面中，選取 [威脅偵測]。 
3. 在威脅偵測組態頁面中 
   - [開啟]  威脅偵測。
   - 設定在偵測到異常資料庫活動時收到安全性警示的**電子郵件清單**。
   - 選取儲存異常威脅稽核記錄的 [Azure 儲存體帳戶]。 
4.  按一下 [儲存] ，以儲存新的或更新的威脅偵測原則。

   ![威脅偵測](./media/sql-database-managed-instance-threat-detection/threat-detection.png)

## <a name="explore-anomalous-managed-instance-activities-upon-detection-of-a-suspicious-event"></a>偵測到可疑事件時探索異常受控執行個體活動

1. 偵測到異常資料庫活動時，您會收到電子郵件通知。 

   電子郵件會提供可疑安全性事件的相關資訊，包括異常活動的性質、資料庫名稱、伺服器名稱和事件時間。 此外，該電子郵件還會提供可能原因和建議動作的相關資訊，以協助您調查和減輕受控執行個體的潛在威脅。

   ![威脅偵測電子郵件](./media/sql-database-managed-instance-threat-detection/threat-detection-email.png)

2. 按一下電子郵件中的 [檢視最近的 SQL 警示] 連結來啟動 Azure 入口網站，並顯示 Azure 資訊安全中心警示頁面，其中會概述在受控執行個體資料庫上偵測到的作用中 SQL 威脅。

   ![作用中的威脅](./media/sql-database-managed-instance-threat-detection/active-threats.png)

3. 按一下特定警示可取得其他詳細資料和調查此威脅的建議，並對未來的威脅採取補救措施。

   例如，SQL 插入式攻擊是網際網路上常見的 Web 應用程式安全性問題之一。 SQL 插入式攻擊用於攻擊資料導向的應用程式。 攻擊者利用應用程式弱點將惡意的 SQL 陳述式插入應用程式輸入欄位，破壞或修改資料庫中的資料。 針對 SQL 插入式攻擊，警示的詳細資料會包括已遭利用且有弱點的 SQL 陳述式。

   ![SQL 插入](./media/sql-database-managed-instance-threat-detection/sql-injection.png)

## <a name="managed-instance-threat-detection-alerts"></a>受控執行個體威脅偵測警示 

受控執行個體的威脅偵測會偵測意圖存取或攻擊資料庫，而可能會造成損害的異常活動，並觸發下列警示：
- **SQL 插入的弱點**：應用程式在資料庫中產生錯誤的 SQL 陳述式時，會觸發此警示。 此警示表示 SQL 插入式攻擊的可能弱點。 錯誤的陳述式之所以產生，有兩項可能的原因：
 - 應用程式的程式碼中有缺失，而建構了錯誤的 SQL 陳述式
 - 應用程式的程式碼或預存程序在建構錯誤的 SQL 陳述式時未處理使用者輸入，這可能會遭到 SQL 插入式攻擊的侵害
- **可能的 SQL 插入**：在 SQL 插入的已知應用程式弱點遭到作用中的攻擊時，會觸發此警示。 這表示有攻擊者嘗試使用有弱點的應用程式程式碼或預存程序插入惡意 SQL 陳述式。
- **從不尋常的位置存取**：有人從不尋常的地理位置登入受控執行個體，而使受控執行個體的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (新的應用程式或開發人員維護作業)。 在其他情況下，警示則是偵測惡意動作 (離職員工、外部攻擊者等)。
- **從不尋常的 Azure 資料中心存取**：有人從 Azure 資料中心 (在近期未發現有存取此受控執行個體) 登入受控執行個體 ，而使受控執行個體的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (您在 Azure、Power BI、Azure SQL 查詢編輯器等之中使用新的應用程式)。 在其他情況下，警示則是偵測來自 Azure 資源/服務的惡意動作 (離職員工、外部攻擊者)。
- **從不熟悉的主體存取**：有人使用不尋常的主體 (SQL 使用者) 登入受控執行個體，而使受控執行個體的存取模式有所變更時，會觸發此警示。 在某些情況下，警示會偵測到合法的動作 (新應用程式的開發人員維護作業)。 在其他情況下，警示則是偵測惡意動作 (離職員工、外部攻擊者)。
- **從可能有害的應用程式存取**：使用可能有害的應用程式用存取資料庫時，會觸發此警示。 在某些情況下，警示會偵測到執行中的滲透測試。 在其他情況下，警示則是偵測使用常見攻擊工具的攻擊。
- **暴力 SQL 認證**：有使用不同認證的異常大量登入失敗時，會觸發此警示。 在某些情況下，警示會偵測到執行中的滲透測試。 在其他情況下，警示則是偵測暴力攻擊。

## <a name="next-steps"></a>後續步驟

- 若要了解受控執行個體，請參閱[受控執行個體是什麼](sql-database-managed-instance.md)
- 深入了解[受控執行個體稽核](https://go.microsoft.com/fwlink/?linkid=869430) 
- 深入了解 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)
