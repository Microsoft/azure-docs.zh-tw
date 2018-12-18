---
title: 升級至最新的彈性資料庫用戶端程式庫 | Microsoft Docs
description: 使用 Nuget 升級彈性資料庫用戶端程式庫。
services: sql-database
ms.service: sql-database
ms.subservice: elastic-scale
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: 9fb29b18397be83f5dc56464b3366d91c47f43b3
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47160786"
---
# <a name="upgrade-an-app-to-use-the-latest-elastic-database-client-library"></a>將應用程式升級以使用最新的彈性資料庫用戶端程式庫
新版本的 [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md) 是透過 Visual Studio 中的 NuGet 和 NuGetPackage Manager 介面提供。 升級包含用戶端程式庫的錯誤修正以及對新功能的支援。

**如需最新版本** ：請瀏覽 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。

使用新的程式庫重新建置您的應用程式，以及變更您 Azure SQL Database 中儲存的現有分區對應管理員中繼資料以支援新功能。

依照順序執行步驟可確保在更新中繼資料物件時，舊版用戶端程式庫不會再於您的環境中出現，這表示升級之後將不會再建立舊版中繼資料物件。   

## <a name="upgrade-steps"></a>升級步驟
**1.升級您的應用程式。** 在 Visual Studio 中，將最新的用戶端程式庫版本下載到您所有使用程式庫的開發專案中，並加以參照；然後重新建置及部署。 

* 在您的 Visual Studio 解決方案中，依序選取 [工具] --> [NuGet 套件管理員] -->  [管理解決方案的 NuGet 套件]。 
* (Visual Studio 2013) 在左邊面板中，選取 [更新]，然後在視窗中顯示的 [Azure SQL Database Elastic Scale 用戶端程式庫] 套件上選取 [更新] 按鈕。
* (Visual Studio 2015) 設定篩選方塊為 [可升級] 。 選取要更新的套件，然後按一下 [更新]  按鈕。
* (Visual Studio 2017) 在對話方塊頂端，選取 [更新]。 選取要更新的套件，然後按一下 [更新] 按鈕。
* 建置並部署。 

**2.升級您的指令碼。** 如果您是使用 **PowerShell** 指令碼來管理分區，請[下載新的程式庫版本](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)並將它複製到您從中執行指令碼的目錄。 

**3.升級您的分割合併服務。** 如果您使用彈性資料庫分割合併工具來重新安排分區資料，請[下載並部署最新版本的工具](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Service.SplitMerge/)。 服務的詳細升級步驟可以在[這裡](sql-database-elastic-scale-overview-split-and-merge.md)找到。 

**4.升級您的分區對應管理員資料庫**。 升級支援您 Azure SQL Database 中之分區對應的中繼資料。  您可以使用 PowerShell 或 C# 來完成此作業。 下面會說明這兩個選項。

***選項 1：使用 PowerShell 升級中繼資料***

1. 請從 [這裡](http://nuget.org/nuget.exe) 下載 NuGet 的最新命令列公用程式並儲存至資料夾。 
2. 開啟命令提示字元，瀏覽至相同的資料夾，然後下達命令： `nuget install Microsoft.Azure.SqlDatabase.ElasticScale.Client`
3. 瀏覽至包含您剛剛下載的新用戶端 DLL 版本的子資料夾，例如： `cd .\Microsoft.Azure.SqlDatabase.ElasticScale.Client.1.0.0\lib\net45`
4. 從 [指令碼中心](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-Database-Elastic-6442e6a9)下載彈性資料庫用戶端升級程式碼片段，然後將它儲存到包含 DLL 的相同資料夾。
5. 從該資料夾中，從命令提示字元執行 “PowerShell .\upgrade.ps1”，然後依照提示完成作業。

***選項 2：使用 C# 升級中繼資料***

或者，建立一個可以開啟您的 ShardMapManager 的 Visual Studio 應用程式，複製到所有分區，然後透過呼叫 [UpgradeLocalStore](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradelocalstore.aspx) 和 [UpgradeGlobalStore](https://msdn.microsoft.com/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.upgradeglobalstore.aspx) 方法來執行中繼資料升級，如此範例中所示： 

    ShardMapManager smm =
       ShardMapManagerFactory.GetSqlShardMapManager
       (connStr, ShardMapManagerLoadPolicy.Lazy); 
    smm.UpgradeGlobalStore(); 

    foreach (ShardLocation loc in
     smm.GetDistinctShardLocations()) 
    {   
       smm.UpgradeLocalStore(loc); 
    } 

這些中繼資料升級技巧可以套用多次且不會造成損害。 例如，如果較舊的用戶端版本在您已經更新之後意外建立分區，您可以在所有分區上再次執行升級，以確保您的整個基礎結構都擁有最新的中繼資料版本。 

**注意：** 至今發佈的新版本用戶端程式庫可以繼續和 Azure SQL DB 上的舊版分區對應管理員中繼資料搭配使用，反之亦然。   但是，若要使用最新用戶端中的部分新功能，則必須升級中繼資料。   請注意，中繼資料升級將不會影響任何使用者資料或應用程式特定資料，只會影響分區對應管理員所建立及使用的物件。  而且應用程式會繼續透過上述的升級順序運作。 

## <a name="elastic-database-client-version-history"></a>彈性資料庫用戶端版本歷程記錄
如需版本歷程記錄，請瀏覽 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]:./media/sql-database-elastic-scale-upgrade-client-library/nuget-upgrade.png

