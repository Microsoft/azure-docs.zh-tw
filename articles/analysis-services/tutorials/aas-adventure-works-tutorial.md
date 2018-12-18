---
title: Azure Analysis Services Adventure Works 教學課程 | Microsoft Docs
description: Azure Analysis Services 的 Adventure Works 教學課程簡介
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: a53bcbe90f39999a838c84807ca834a1e6cd1813
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42139862"
---
# <a name="azure-analysis-services---adventure-works-tutorial"></a>Azure Analysis Services - Adventure Works 教學課程

本教學課程示範如何使用 Visual Studio [Analysis Services 專案](https://marketplace.visualstudio.com/items?itemName=ProBITools.MicrosoftAnalysisServicesModelingProjects)或 [SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt) 來建立及部署 1400 相容性等級的表格式模型。  
如果您熟悉 Analysis Services 和表格式模型化，則本教學課程可讓您以最快的方式，了解如何使用 Visual Studio 來建立及部署基本表格式模型。 備齊必要條件後，完成本教學課程需要二至三小時的時間。  
  
## <a name="what-you-learn"></a>您學到什麼   
  
-   如何在 Visual Studio 中建立 **1400 相容性等級**的新表格式模型專案。
  
-   如何將關聯式資料庫的資料匯入表格式模型專案工作區資料庫中。  
  
-   如何在模型中建立及管理資料表之間的關聯性。  
  
-   如何建立計算結果欄、量值和關鍵效能指標，幫助使用者分析重要商務計量。  
  
-   如何建立及管理檢視方塊和階層，藉由提供商務和應用程式專屬視點，以協助使用者更輕鬆地瀏覽模型資料。  
  
-   如何建立分割區將資料表資料分割成較小的邏輯部分，以獨立於其他分割區來處理。  
  
-   如何建立具有使用者成員的角色以保護模型物件和資料。  
  
-   如何使用 Visual Studio 將表格式模型部署到 **Azure Analysis Services** 伺服器或 **SQL Server 2017 Analysis Services** 伺服器。  
  
## <a name="prerequisites"></a>必要條件  
若要完成本教學課程，您需要：  
  
-   Azure Analysis Services 伺服器。 註冊免費的 [Azure Analysis Services 試用版](https://azure.microsoft.com/services/analysis-services/)和[建立伺服器](../analysis-services-create-server.md)。 

-   隨附**範例 AdventureWorksDW 資料庫**的 [Azure SQL 資料倉儲](../../sql-data-warehouse/create-data-warehouse-portal.md)，或隨附 [Adventure Works 範例資料庫](https://github.com/Microsoft/sql-server-samples/releases/tag/adventureworks)的 SQL Server 資料倉儲。

    **重要事項︰** 如果您將範例資料庫安裝到內部部署 SQL Server 資料倉儲，且將模型部署到 Azure Analysis Services 伺服器，就需要[內部部署資料閘道](../analysis-services-gateway.md)。

-   適用於 Visual Studio 的最新版本 [SQL Server Data Tools (SSDT)](https://msdn.microsoft.com/library/mt204009.aspx)。

-   最新版本的 [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)。    

-   用戶端應用程式，例如 [Power BI Desktop](https://powerbi.microsoft.com/desktop/) 或 Excel。 

## <a name="scenario"></a>案例  
本教學課程以虛構公司 Adventure Works Cycles 做為基礎。 Adventure Works 是一家大型的跨國製造公司，主要生產自行車、零件和配件，產品銷售到北美洲、歐洲和亞洲的商業市場。 該公司雇用 500 名員工。 此外，Adventure Works 僱用數個區域銷售團隊，遍布其市場規模。 您的專案是建立一個表格式模型，讓銷售和行銷使用者分析 AdventureWorksDW 資料庫中的網際網路銷售資料。  
  
若要完成本教學課程，您必須完成各個課程。 每個課程有一些工作。 必須依序完成每個工作，才能完成本課程。 特定的課程中可能有幾項工作會達成類似的結果，但您完成每項工作的方式會稍有不同。 此方法顯示完成工作通常有一種以上的方法，並激發您運用先前的課程和工作中所學到的技術。  
  
這些課程的目的是引導您使用 SSDT 內含的許多功能來製作基本表格式模型。 因為每個課程都是根據上一課，您應該依序完成課程。
  
本教學課程提供的課題不包括：在 Azure 入口網站中管理伺服器、使用 SSMS 管理伺服器或資料庫，或使用用戶端應用程式來瀏覽模型資料。 


## <a name="lessons"></a>課程  
本教學課程包括下列課程︰  
  
|課程|預估完成時間|  
|----------|------------------------------|  
|[1 - 建立新的表格式模型專案](../tutorials/aas-lesson-1-create-a-new-tabular-model-project.md)|10 分鐘|  
|[2 - 取得資料](../tutorials/aas-lesson-2-get-data.md)|10 分鐘|  
|[3 - 標記為日期資料表](../tutorials/aas-lesson-3-mark-as-date-table.md)|3 分鐘|  
|[4 - 建立關聯性](../tutorials/aas-lesson-4-create-relationships.md)|10 分鐘|  
|[5 - 建立計算結果欄](../tutorials/aas-lesson-5-create-calculated-columns.md)|15 分鐘|
|[6 - 建立量值](../tutorials/aas-lesson-6-create-measures.md)|30 分鐘|  
|[7 - 建立關鍵效能指標 (KPI)](../tutorials/aas-lesson-7-create-key-performance-indicators.md)|15 分鐘|  
|[8 - 建立檢視方塊](../tutorials/aas-lesson-8-create-perspectives.md)|5 分鐘|  
|[9 - 建立階層](../tutorials/aas-lesson-9-create-hierarchies.md)|20 分鐘|  
|[10 - 建立分割區](../tutorials/aas-lesson-10-create-partitions.md)|15 分鐘|  
|[11 - 建立角色](../tutorials/aas-lesson-11-create-roles.md)|15 分鐘|  
|[12 - 在 Excel 中分析](../tutorials/aas-lesson-12-analyze-in-excel.md)|5 分鐘| 
|[13 - 部署](../tutorials/aas-lesson-13-deploy.md)|5 分鐘|  
  
## <a name="supplemental-lessons"></a>補充課程  
完成這個教學課程並不需要這些課程，但是能幫助您更了解表格式模型的進階製作功能。  
  
|課程|預估完成時間|  
|----------|------------------------------|  
|[詳細資料列](../tutorials/aas-supplemental-lesson-detail-rows.md)|10 分鐘|
|[動態安全性](../tutorials/aas-supplemental-lesson-dynamic-security.md)|30 分鐘|
|[不完全階層](../tutorials/aas-supplemental-lesson-ragged-hierarchies.md)|20 分鐘| 

  
## <a name="next-steps"></a>後續步驟  
若要開始，請參閱[第 1 課：建立新的表格式模型專案](../tutorials/aas-lesson-1-create-a-new-tabular-model-project.md)。  
  
  
  

