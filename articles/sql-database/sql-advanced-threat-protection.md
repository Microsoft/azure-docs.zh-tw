---
title: 進階威脅防護 - Azure SQL 資料庫 | Microsoft Docs
description: 了解探索及分類敏感性資料、管理資料庫弱點，以及偵測可能表示對 Azure SQL 資料庫有威脅之異常活動的功能。
services: sql-database
author: ronitr
manager: craigg
ms.service: sql-database
ms.topic: conceptual
ms.date: 05/17/2018
ms.author: ronitr
ms.reviewer: vanto
ms.openlocfilehash: 350f43dcb15062b4d44fc537e664b257f77d8e88
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/20/2018
ms.locfileid: "46497819"
---
# <a name="advanced-threat-protection-for-azure-sql-database"></a>Azure SQL 資料庫的進階威脅防護

「SQL 進階威脅防護」是進階 SQL 安全性功能的整合套件。 其中包含的功能可用於探索和分類敏感性資料、找出潛在資料庫弱點並減低其風險，及偵測可能指出資料庫遇到威脅的異常活動。 此套件可讓您從單一的進入點位置啟用及管理前述功能。 

## <a name="overview"></a>概觀

SQL 進階威脅防護 (ATP) 提供一組進階的 SQL 安全性功能，包括資料探索與分類、弱點評估及威脅偵測。 

- [資料探索與分類](sql-database-data-discovery-and-classification.md) (目前處於預覽階段) 提供內建於 Azure SQL Database 的功能，可用來探索、分類、標記和保護資料庫中的敏感性資料。 它可用來讓您檢視資料庫分類狀態、追蹤對資料庫內敏感性資料的存取，並具有其他多方面的用途。
- [弱點評估](sql-vulnerability-assessment.md)是容易設定的服務，可探索、追蹤及協助您修復潛在的資料庫弱點。 它可讓您檢視安全性狀態，且包含解決安全性問題和增強資料庫防護性的可行步驟。
- [威脅偵測](sql-database-threat-detection.md)可偵測異常活動，指出有不尋常及可能有害的活動試圖存取或惡意探索您的資料庫。 它會持續監視您的資料庫是否有可疑的活動，並在發現潛在弱點、SQL 插入式攻擊和異常資料庫存取模式時提供即時安全性警示。 威脅偵測警示會提供可疑活動的詳細資料，以及如何調查與降低威脅的建議。

只要啟用 SQL ATP，其中包含的所有功能即會一併啟用。 只要按一下按鍵，即可對您的整個資料庫伺服器啟用 ATP，套用至伺服器上所有的資料庫。 

ATP 的定價與 Azure 資訊安全中心 標準層相同 (每一節點每月 15 美元)；每一個受保護的 SQL Database 伺服器計為一個節點。 啟用後的 60 天內視為免費試用期，不需付費。 如需詳細資訊，請參閱 [Azure 資訊安全中心定價頁面](https://azure.microsoft.com/pricing/details/security-center/)。


## <a name="getting-started-with-atp"></a>開始使用 ATP 
下列步驟可讓您開始使用 ATP。 

## <a name="1-enable-atp"></a>1.啟用 ATP

在您的 [Azure SQL 資料庫] 窗格中，瀏覽至 [安全性] 標題下的 [進階威脅防護]，以啟用 ATP。 若要為伺服器上的所有資料庫啟用 ATP，請按一下 [在伺服器上啟用進階威脅防護]。

![啟用 ATP](./media/sql-advanced-protection/enable_atp.png) 

> [!NOTE]
> ATP 的成本是每一節點每月 15 美元；節點是指整個 SQL 邏輯伺服器。 因此，您只需付費一次，即可使用 ATP 保護伺服器上的所有資料庫。 前 60 天視為免費試用期。

## <a name="2-configure-vulnerability-assessment"></a>2.設定弱點評估

若要開始使用弱點評估，您必須設定用來儲存掃描結果的儲存體帳戶。 若要執行此作業，請按一下 [弱點評估] 卡。

![設定 VA](./media/sql-advanced-protection/configure_va.png) 

選取或建立用來儲存掃描結果的儲存體帳戶。 您也可以開啟定期的週期性掃描，將弱點評估設定為每週執行一次自動掃描。 掃描結果摘要會傳送到您所提供的電子郵件地址。

![VA 設定](./media/sql-advanced-protection/va_settings.png) 

## <a name="3-start-classifying-data-tracking-vulnerabilities-and-investigating-threat-alerts"></a>3.開始分類資料、追蹤弱點，並調查威脅警示

按一下 [資料探索與分類] 卡可查看建議要分類的敏感性資料行，以及使用持續敏感性標籤為資料分類。 按一下 [弱點評估] 卡可檢視和管理弱點掃描和報告，以及追蹤您的安全性水準。 如果收到安全性警示，您可以按一下 [威脅偵測] 卡以檢視警示的詳細資料，並可透過 Azure 資訊安全中心安全性警示頁面，查看您 Azure 訂用帳戶中所有警示的整合報告。

## <a name="4-manage-atp-settings-on-your-sql-server"></a>4.管理 SQL 伺服器上的 ATP 設定

若要檢視及管理 [進階威脅防護] 設定，請在您的 [SQL 伺服器] 窗格中，瀏覽至 [安全性] 標題下的 [進階威脅防護]。 在此頁面上，您可以啟用或停用 ATP，以及修改整個 SQL 伺服器的威脅偵測設定。

![伺服器設定](./media/sql-advanced-protection/server_settings.png) 

## <a name="5-manage-atp-settings-for-a-sql-database"></a>5.管理 SQL 資料庫的 ATP 設定

若要覆寫特定資料庫的 ATP 威脅偵測設定，請勾選 [啟用資料庫層級的進階威脅防護] 核取方塊。 只有在您明確需要就個別資料庫接收個別的威脅偵測警示，用以取代或補強對伺服器上的所有資料庫接收的警示時，才須使用此選項。 

在選取此核取方塊後，請按一下 [此資料庫的威脅偵測設定]，然後設定此資料庫的相關設定。

![資料庫和威脅偵測設定](./media/sql-advanced-protection/database_threat_detection_settings.png) 

伺服器的進階威脅防護設定也可以從 ATP 資料庫窗格來存取。 請按一下主要 ATP 窗格中的 [設定]，然後按一下 [檢視進階威脅防護伺服器設定]。 

![資料庫設定](./media/sql-advanced-protection/database_settings.png) 

## <a name="next-steps"></a>後續步驟 

- 深入了解[資料探索與分類](sql-database-data-discovery-and-classification.md) 
- 深入了解[弱點評估](sql-vulnerability-assessment.md) 
- 深入了解[威脅偵測](sql-database-threat-detection.md)
- 深入了解 [Azure 資訊安全中心](https://docs.microsoft.com/azure/security-center/security-center-intro)
