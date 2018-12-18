---
title: Azure 資訊安全中心和 Azure SQL Database 服務 | Microsoft Docs
description: 本文說明資訊安全中心如何協助您保護 Azure SQL Database 中的資料庫。
services: sql-database
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: f109adfd-daed-4257-9692-2042a1399480
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: b507a62db9a80866005cb63d2008fb14612b516f
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31415940"
---
# <a name="azure-security-center-and-azure-sql-database-service"></a>Azure 資訊安全中心和 Azure SQL Database 服務
[Azure 資訊安全中心](https://azure.microsoft.com/documentation/services/security-center/)可協助您保護、偵測威脅並採取相應的措施。 它提供您 Azure 訂用帳戶之間的整合式安全性監視和原則管理，協助您偵測可能會忽略的威脅，且適用於廣泛的安全性解決方案生態系統。

本文說明資訊安全中心如何協助您保護 Azure SQL Database 中的資料庫。

## <a name="why-use-security-center"></a>為何使用資訊安全中心？
資訊安全中心藉由提供您所有伺服器和資料庫的安全性可見性，協助您保護 SQL Database 中的資料。 使用資訊安全中心，您可以︰

* 定義 SQL Database 加密和稽核原則。
* 監視所有訂用帳戶的 SQL Database 資源安全性。
* 快速找出並修復安全性問題。
* 整合 [Azure SQL Database 威脅偵測](../sql-database/sql-database-threat-detection.md)所提供的警示。

除了協助您保護 SQL Database 資源，資訊安全中心也提供 Azure 虛擬機器、雲端服務、應用程式服務、虛擬網路等的安全性監視和管理功能。 [在此](security-center-intro.md)深入了解資訊安全中心。

## <a name="prerequisites"></a>先決條件
若要開始使用資訊安全中心，您必須有 Microsoft Azure 訂用帳戶。 您的訂用帳戶已經啟用資訊安全中心的免費層。 如需資訊安全中心的免費和標準層的詳細資訊，請參閱[安全性中心價格](https://azure.microsoft.com/pricing/details/security-center/)。

資訊安全中心支援角色型存取。 若要深入了解 Azure 中的角色型存取控制 (RBAC)，請參閱 [Azure Active Directory 角色型存取控制](../role-based-access-control/role-assignments-portal.md)。 資訊安全中心常見問題集提供[在資訊安全中心處理權限的方式](security-center-faq.md#permissions)。

## <a name="access-security-center"></a>存取資訊安全中心
您可以從 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)存取資訊安全中心。 [登入入口網站](https://portal.azure.com/)，然後選取 [資訊安全中心] 選項。

![資訊安全中心選項][1]

[資訊安全中心] 刀鋒視窗隨即開啟。
![資訊安全中心刀鋒視窗][2]

## <a name="set-security-policy"></a>設定安全性原則
安全性原則定義了針對指定之訂用帳戶或資源群組內的資源所建議的一組控制項。 在資訊安全中心，您可以根據公司安全性需求，以及每個訂用帳戶中應用程式的類型或資料的敏感性，為您的訂用帳戶或資源群組定義原則。

您可以設定原則，以顯示 SQL 稽核和 SQL 透明資料加密 (TDE) 的建議。

* 當您開啟 [SQL 稽核和威脅偵測] 時，資訊安全中心會建議針對法規遵循、進階偵測及調查用途，啟用 Azure 資料庫的存取稽核。
* 當您開啟 [SQL 透明資料加密] 時，資訊安全中心會建議為您的 Azure SQL Database、關聯的備份及交易記錄檔啟用待用期加密。

若要設定安全性原則，請選取 [資訊安全中心] 刀鋒視窗上的 [原則] 圖格。 在 [安全性原則] 刀鋒視窗上，選取您想要啟用安全性原則的訂用帳戶。 選取 [預防原則] 並 [開啟] 您想要在此訂用帳戶使用的安全性建議。
![安全性原則][3]

若要深入了解，請參閱[設定安全性原則](security-center-policies.md)。

## <a name="manage-security-recommendation"></a>管理安全性建議
資訊安全中心會定期分析 Azure 資源的安全性狀態。 當資訊安全中心識別潛在的安全性弱點時，它會建立建議。 這些建議會引導您完成設定所需控制項的程序。

設定安全性原則之後，資訊安全中心會分析您資源的安全性狀態，以識別潛在的弱點。 系統會以表格格式顯示建議，其中每一行代表一個特定的建議。 使用下表做為參考，協助您了解 Azure SQL Database 的可用建議，以及如果套用建議，每一個建議將產生的作用。 選取某項建議，就會出現說明如何在資訊安全中心實作建議的文章。

| 建議 | 說明 |
| --- | --- |
| [在 SQL Server 上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-servers.md) |建議您針對 SQL Database 伺服器開啟稽核和威脅偵測功能。 (僅限 SQL Database 服務。 不包含在虛擬機器上執行的 Microsoft SQL Server。) |
| [在 SQL Database 上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-databases.md) |針對 SQL Database 資料庫開啟稽核和威脅偵測的建議。 (僅限 SQL Database 服務。 不包含在虛擬機器上執行的 Microsoft SQL Server。) |
| [啟用透明資料加密](security-center-enable-transparent-data-encryption.md) |建議您針對 SQL Database 啟用加密功能。 (僅限 SQL Database 服務。) |

若要查看 Azure 資源的相關建議，請選取 [資訊安全中心] 刀鋒視窗上的 [建議] 圖格。 在 [建議] 刀鋒視窗上，選取某項建議以查看詳細資料。 在此範例中，我們選取 [在 SQL Server 上啟用稽核與威脅偵測]。

![建議][4]

如下所示，資訊安全中心會顯示未啟用稽核與威脅偵測的 SQL Server。 開啟稽核之後，您即可設定威脅偵測設定和電子郵件設定來接收安全性警示。 威脅偵測會在偵測到異常資料庫活動時警示您，指出資料庫有潛在的安全性威脅。 警示會顯示在 [資訊安全中心] 儀表板中。
![稽核與威脅偵測][5]

遵循 [Azure 入口網站中的 SQL Database 威脅偵測](../sql-database/sql-database-threat-detection-portal.md)的步驟，來開啟並設定威脅偵測，以及設定將在偵測到異常活動時接收到安全性警示的電子郵件清單。

若要深入了解相關建議，請參閱[管理安全性建議](security-center-recommendations.md)。

## <a name="monitor-security-health"></a>監視安全性健康狀態
在您為訂用帳戶的資源啟用 [安全性原則](security-center-policies.md) 之後，資訊安全中心會分析您資源的安全性狀態，以找出潛在的弱點。  您可以在 [資源安全性健康狀態] 圖格中，檢視資源的安全性狀態。 當您按一下 [資源安全性健康狀態] 圖格中的 [資料] 時，將會開啟 [資料資源] 刀鋒視窗，內含一些問題 (例如未啟用稽核和透明資料加密) 的 SQL 建議。 它也具有資料庫的一般健全狀況狀態建議。
![資源安全性健康狀態][6]

若要深入了解，請參閱[安全性健康狀態監視](security-center-monitoring.md)。

## <a name="manage-and-respond-to-security-alerts"></a>管理和回應安全性警示
資訊安全中心會自動收集、分析及整合來自 [Azure SQL 威脅偵測](../sql-database/sql-database-threat-detection.md)和其他 Azure 資源的記錄檔資料，以偵測真正的威脅並降低誤判。 「資訊安全中心」會顯示優先安全性警示清單，以及需要您快速調查問題的資訊，和如何修復攻擊行為的建議。

若要設定警示，請選取 [資訊安全中心] 刀鋒視窗上的 [安全性警示] 圖格。 在 [安全性警示] 刀鋒視窗上，選取一個警示以深入了解觸發警示的事件；如果發現項目，您需要進行一些步驟來阻止攻擊。 在此範例中，我們選取 [潛在的 SQL 插入式攻擊] 。
![安全性警示][7]

如下所示，資訊安全中心會提供額外的詳細資料以便深入了解什麼會觸發警示、目標資源、來源 IP 位址 (若適用)，以及有關如何補救的建議。
![潛在的 SQL 插入式攻擊][8]

若要深入了解，請參閱[管理和回應安全性警示](security-center-managing-and-responding-alerts.md)。

## <a name="next-steps"></a>後續步驟
* [資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [資訊安全中心規劃和操作指南](security-center-planning-and-operations-guide.md) - 遵循一組步驟和工作，以根據您組織的安全性需求和雲端管理模型，將您的資訊安全中心使用最佳化。
* [資訊安全中心資料安全性](security-center-data-security.md) – 了解資訊安全中心如何收集和處理 Azure 資源的相關資料，包括組態資訊、中繼資料、事件記錄檔、損毀傾印檔等等。
* [處理安全性事件](security-center-incident.md) - 了解如何使用資訊安全中心的安全性警示功能，協助您處理安全性事件。

<!--Image references-->
[1]: ./media/security-center-sql-database/security-center.png
[2]: ./media/security-center-sql-database/security-center-blade.png
[3]: ./media/security-center-sql-database/security-policy.png
[4]: ./media/security-center-sql-database/recommendation.png
[5]: ./media/security-center-sql-database/turn-on-auditing.png
[6]: ./media/security-center-sql-database/monitor-health.png
[7]: ./media/security-center-sql-database/alert.png
[8]: ./media/security-center-sql-database/sql-injection.png
