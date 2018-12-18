---
title: Azure SQL Database 多租用戶應用程式範例 - Wingtip SaaS | Microsoft Docs
description: 藉由使用採用 Azure SQL Database (Wingtip SaaS 範例) 的範例多租用戶應用程式來學習
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: dd67e49a80c403e807b24276cf14b9e8c47037b9
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055504"
---
# <a name="introduction-to-a-multitenant-saas-app-that-uses-the-database-per-tenant-pattern-with-sql-database"></a>搭配 SQL Database 使用每一租用戶一個資料庫模式的多租用戶 SaaS 應用程式簡介

Wingtip SaaS 應用程式是一個範例多租用戶應用程式。 此應用程式使用每一租用戶一個資料庫的 SaaS 應用程式模式來服務多個租用戶。 此應用程式藉由使用數個 SaaS 設計和管理模式，展示支援 SaaS 案例的 Azure SQL Database 功能。 為了快速啟動並執行，Wingtip SaaS 應用程式會在五分鐘內完成部署。

在 [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant) GitHub 存放庫可取得應用程式原始程式碼和管理指令碼。 開始之前，請先參閱[一般指引](saas-tenancy-wingtip-app-guidance-tips.md)，了解有關下載 Wingtip Tickets 管理指令碼並將其解除封鎖的步驟。

## <a name="application-architecture"></a>應用程式架構

Wingtip SaaS 應用程式會使用每一租用戶一個資料庫的模型。 它會使用 SQL 彈性集區來發揮最高效率。 若要將租用戶佈建和對應至其資料，則會使用目錄資料庫。 核心 Wingtip SaaS 應用程式會使用具有三個範例租用戶的集區，再加上目錄資料庫。 目錄和租用戶伺服器已透過 DNS 別名進行佈建。 這些別名會用來為 Wingtip 應用程式使用的作用中資源維護參考。 這些別名會更新為指向災害復原教學課程中的復原資源。 在完成許多 Wingtip SaaS 教學課程之後，初始部署就會擁有附加元件。 導入的附加元件包括分析資料戶和跨資料庫結構描述管理等。


![Wingtip SaaS 架構](media/saas-dbpertenant-wingtip-app-overview/app-architecture.png)


在進行教學課程及搭配應用程式運作的過程中，請將焦點放在 SaaS 模式上，因為它們與資料層相關。 換句話說，就是著重於資料層，而不要過度分析應用程式本身。 了解這些 SaaS 模式的實作方式是在應用程式中實作這些模式的關鍵。 此外，也請考慮針對您特定的業務需求進行任何必要的修改。

## <a name="sql-database-wingtip-saas-tutorials"></a>SQL Database Wingtip SaaS 教學課程

部署應用程式之後，請瀏覽下列以初始部署為基礎的教學課程。 這些教學課程會探索常見的 SaaS 模式，這些模式會利用 SQL Database、「Azure SQL 資料倉儲」及其他 Azure 服務的內建功能。 教學課程包括 PowerShell 指令碼及詳細的說明。 這些說明可讓您更容易了解及在您應用程式中實作相同的 SaaS 管理模式。


| 教學課程 | 說明 |
|:--|:--|
| [SQL Database 多租用戶 SaaS 應用程式範例的指引和祕訣](saas-tenancy-wingtip-app-guidance-tips.md) | 下載並執行 PowerShell 指令碼，以準備應用程式的組件。 |
|[部署及探索 Wingtip SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)|  使用您的 Azure 訂用帳戶來部署及探索 Wingtip SaaS 應用程式。 |
|[佈建租用戶並在目錄中註冊](saas-dbpertenant-provision-and-catalog.md)| 了解應用程式如何使用目錄資料庫來連線至租用戶，以及目錄如何將租用戶對應至其資料。 |
|[監視及管理效能](saas-dbpertenant-performance-monitoring.md)| 了解如何使用 SQL Database 的監視功能，以及設定超過效能閾值時的警示。 |
|[透過 Azure Log Analytics 監視](saas-dbpertenant-log-analytics.md) | 了解如何使用 [Log Analytics](../log-analytics/log-analytics-overview.md) 來監視跨多個集區的大量資源。 |
|[還原單一租用戶](saas-dbpertenant-restore-single-tenant.md)| 了解如何將租用戶資料庫還原到先前的時間點。 此外，也了解如何還原到平行資料庫，這會讓現有的租用戶資料庫維持上線狀態。 |
|[管理租用戶資料庫結構描述](saas-tenancy-schema-management.md)| 了解如何更新結構描述，以及更新跨所有租用戶資料庫的參考資料。 |
|[執行跨租用戶分散式查詢](saas-tenancy-cross-tenant-reporting.md) | 建立臨機操作分析資料庫，並跨所有租用戶執行即時分散式查詢。  |
|[根據擷取的租用戶資料執行分析](saas-tenancy-tenant-analytics.md) | 將租用戶資料擷取到分析資料庫或資料倉儲中，以便執行離線分析查詢。 |


## <a name="next-steps"></a>後續步驟

- [部署和使用 Wingtip Tickets SaaS 應用程式範例時的一般指引和秘訣](saas-tenancy-wingtip-app-guidance-tips.md)
- [部署 Wingtip SaaS 應用程式](saas-dbpertenant-get-started-deploy.md)
