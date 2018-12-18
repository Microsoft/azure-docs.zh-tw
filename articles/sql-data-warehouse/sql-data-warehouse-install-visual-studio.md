---
title: 安裝適用於 SQL 資料倉儲的 Visual Studio 和 SSDT | Microsoft Docs
description: 安裝適用於 Azure SQL 資料倉儲的 Visual Studio 和 SQL Server Development Tools (SSDT)
services: sql-data-warehouse
ms.custom: vs-azure
ms.workload: azure-vs
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: ba84b64afb1d5ebcd5ec153787ddc7d0739bd8d8
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43307380"
---
# <a name="install-visual-studio-and-ssdt-for-sql-data-warehouse"></a>安裝適用於 SQL 資料倉儲的 Visual Studio 和 SSDT
若要開發 SQL 資料倉儲的應用程式，建議使用最新版本的 Visual Studio，搭配最新版本的 SQL Server Data Tools (SSDT)。  Visual Studio 2013 Update 5 搭配 SSDT 也支援回溯相容性。  

搭配使用 Visual Studio 和 SSDT 可讓您使用 SQL Server 物件總管，經由視覺化方式瀏覽資料表、檢視、預存程序和 SQL 資料倉儲中的其他許多物件，以及執行查詢。

> [!NOTE]
> SQL 資料倉儲尚未支援 Visual Studio 資料庫專案。 若要定期接收這項功能的更新，請在 [UserVoice] 投票。
> 
> 

## <a name="step-1-install-visual-studio"></a>步驟 1：安裝 Visual Studio
遵循下列連結來下載並安裝 Visual Studio。 如果已安裝 Visual Studio 2013 或更新版本，可以跳至步驟 2 安裝 SSDT。

1. [下載 Visual Studio][]。
2. 依照 MSDN 上的[安裝 Visual Studio][Installing Visual Studio] 指南並選擇預設組態。

## <a name="step-2-install-ssdt"></a>步驟 2：安裝 SSDT
若要安裝適用於 Visual Studio 的 SSDT，請先依照下列步驟，從 Visual Studio 內檢查 SSDT 更新。

1. 在 Visual Studio 中，按一下 [工具] / [擴充功能和更新] / [更新]。
2. 選取 [產品更新]，然後尋找 [適用於資料庫工具的 Microsoft SQL Server 更新]

如果找不到更新，表示您應該已安裝最新版本。  若要確認已安裝 SSDT，請按一下 [說明] / [關於 Microsoft Visual Studio]，然後在清單中尋找 SQL Server Data Tools。 SSDT 的最新版本是 14.0.60525.0。 如果無法在 Visual Studio 中使用安裝選項，或者您也可以瀏覽 [SSDT 下載][SSDT Download]頁面，手動下載和安裝 SSDT。

## <a name="next-steps"></a>後續步驟
既然有了最新版本的 SSDT，您可以開始 [連接][connect]到 SQL 資料倉儲。

<!--Anchors-->

<!--Image references-->

<!--Articles-->
[connect]: ./sql-data-warehouse-query-visual-studio.md

<!--Other-->
[下載 Visual Studio]: https://www.visualstudio.com/downloads/
[Installing Visual Studio]: https://msdn.microsoft.com/library/e2h7fzkw.aspx
[SSDT Download]: https://msdn.microsoft.com/library/mt204009.aspx
[UserVoice]: https://feedback.azure.com/forums/307516-sql-data-warehouse/suggestions/13313247-database-project-from-visual-studio-to-support-azu
