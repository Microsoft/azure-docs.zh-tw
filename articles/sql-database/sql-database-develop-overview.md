---
title: SQL Database 應用程式開發概觀 | Microsoft Docs
description: 深入了解連接至 SQL Database 的應用程式的可用連線庫和最佳作法。
services: sql-database
author: stevestein
manager: craigg
ms.reviewer: genemi
ms.service: sql-database
ms.custom: develop apps
ms.topic: conceptual
ms.date: 06/20/2018
ms.author: sstein
ms.openlocfilehash: 1986c19e6625df7c281f7de9b322124da6a12c45
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46973828"
---
# <a name="sql-database-application-development-overview"></a>SQL Database 應用程式開發概觀
本文將逐步解說開發人員在撰寫程式碼以連接到 Azure SQL Database 時應注意的基本考量事項。

> [!TIP]
> 如需示範如何建立伺服器、建立伺服器型防火牆、檢視伺服器屬性、使用 SQL Server Management Studio 進行連接、查詢 Master 資料庫、建立範例資料庫和空白資料庫、查詢資料庫屬性、使用 SQL Server Management Studio 進行連接，以及查詢範例資料庫的教學課程，請參閱[開始使用教學課程](sql-database-get-started-portal.md)。
>

## <a name="language-and-platform"></a>語言和平台
有一些程式碼範例可供各種程式設計語言和平台使用。 您可以在下列位置找到程式碼範例的連結： 

* 詳細資訊：[SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md)。

## <a name="tools"></a>工具 
您可以利用 [cheetah](https://github.com/wunderlist/cheetah)、[sql-cli](https://www.npmjs.com/package/sql-cli)、[VS Code](https://code.visualstudio.com/) 等開放原始碼工具。 此外，Azure SQL Database 使用 [Visual Studio](https://www.visualstudio.com/downloads/) 和 [SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx) 等 Microsoft 工具。  您也可以使用 Azure 管理入口網站、PowerShell 和 REST API 協助您獲得額外生產力。

## <a name="resource-limitations"></a>資源限制
Azure SQL Database 使用兩種不同機制來管理資料庫可使用的資源：資源管理和限制強制執行。 如需詳細資訊，請參閱

- [以 DTU 為基礎的資源模型限制 - 單一資料庫](sql-database-dtu-resource-limits-single-databases.md)
- [以 DTU 為基礎的資源模型限制 - 彈性集區](sql-database-dtu-resource-limits-elastic-pools.md)
- [以虛擬核心為基礎的資源限制 - 單一資料庫](sql-database-vcore-resource-limits-single-databases.md)
- [以虛擬核心為基礎的資源限制 - 彈性集區](sql-database-vcore-resource-limits-elastic-pools.md)

## <a name="security"></a>安全性
Azure SQL Database 提供資源以在 SQL Database 上限制存取、保護資料，以及監視活動。

* 詳細資訊：[保護您的 SQL Database](sql-database-security-overview.md)。

## <a name="authentication"></a>驗證
* Azure SQL Database 支援 SQL Server 驗證使用者和登入，以及 [Azure Active Directory 驗證](sql-database-aad-authentication.md) 使用者和登入。
* 您必須指定特定的資料庫，而非預設為 master  資料庫。
* 您無法在 SQL Database 上使用 **USE myDatabaseName;** 陳述式來切換到其他資料庫。
* 詳細資訊：[SQL Database 安全性：管理資料庫存取與登入安全性](sql-database-manage-logins.md)。

## <a name="resiliency"></a>災害復原
當連接到 SQL Database 發生暫時性錯誤時，您的程式碼應該重試呼叫。  我們建議重試邏輯使用輪詢邏輯，因此它不會同時重試多個用戶端而讓 SQL Database 超過負荷。

* 程式碼範例︰如需示範重試邏輯的程式碼範例，請在以下位置參閱您選擇語言的範例︰[SQL Database 和 SQL Server 的連線庫](sql-database-libraries.md)。
* 詳細資訊：[SQL Database 用戶端程式的錯誤訊息](sql-database-develop-error-messages.md)。

## <a name="managing-connections"></a>管理連線
* 在您的用戶端連線邏輯中，將預設的逾時覆寫為 30 秒。  預設 15 秒對於依賴網際網路的連線而言太短。
* 如果您使用的是 [連接集區](http://msdn.microsoft.com/library/8xx3tyca.aspx)，請確定在程式未主動使用時即時關閉連接，而不是準備重複使用連接。

## <a name="network-considerations"></a>網路考量事項
* 在託管您的用戶端程式的電腦上，請確定防火牆允許連接埠 1433 上的傳出 TCP 通訊。  詳細資訊：[設定 Azure SQL Database 防火牆](sql-database-configure-firewall-settings.md)。
* 如果您的用戶端程式是在 Azure 虛擬機器 (VM) 上執行，而用戶端程式會連線至 SQL Database，您就必須開啟該 VM 上特定的連接埠範圍。 詳細資訊：[針對 ADO.NET 4.5 和 SQL Database 之 1433 以外的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)。
* 與 Azure SQL Database 的用戶端連線有時會略過 proxy 並直接與資料庫互動。 1433 以外的連接埠變得重要。 如需詳細資訊，請參閱 [Azure SQL Database 連線架構](sql-database-connectivity-architecture.md)和[針對 ADO.NET 4.5 及 SQL Database 的 1433 以外的連接埠](sql-database-develop-direct-route-ports-adonet-v12.md)一節。

## <a name="data-sharding-with-elastic-scale"></a>使用 Elastic Scale 的資料分區化
彈性擴縮會簡化相應放大 (和相應縮小) 的程序。 

* [多租用戶 SaaS 應用程式與 Azure SQL Database 的設計模式](sql-database-design-patterns-multi-tenancy-saas-applications.md)。
* [資料相依路由](sql-database-elastic-scale-data-dependent-routing.md)。
* [開始使用 Azure SQL Database 彈性擴縮預覽](sql-database-elastic-scale-get-started.md)。

## <a name="next-steps"></a>後續步驟
瀏覽 [SQL Database 的所有功能](sql-database-technical-overview.md)。
