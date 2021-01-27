---
title: 開始使用彈性資料庫工具
description: Azure SQL Database 彈性資料庫工具功能的基本解說，包括易於執行的範例應用程式。
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: anumjs
ms.author: anjangsh
ms.reviewer: sstein
ms.date: 01/25/2019
ms.openlocfilehash: 74343b2f05bb4a59e475449c87524ff66cdd605d
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98919539"
---
# <a name="get-started-with-elastic-database-tools"></a>開始使用彈性資料庫工具
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

本文將協助您執行範例應用程式，以介紹[彈性資料庫用戶端程式庫](elastic-database-client-library.md)的開發人員體驗。 範例應用程式會建立簡易的分區化應用程式，並探討 Azure SQL Database 彈性資料庫工具功能的主要功能。 它著重在[分區對應管理](elastic-scale-shard-map-management.md)、[資料相依路由](elastic-scale-data-dependent-routing.md)和[多分區查詢](elastic-scale-multishard-querying.md)的使用案例。 用戶端程式庫可供 .NET 和 Java 取得。

## <a name="elastic-database-tools-for-java"></a>適用於 Java 的彈性資料庫工具

### <a name="prerequisites"></a>先決條件

* Java Developer Kit (JDK) 1.8 版或更新版本
* [Maven](https://maven.apache.org/download.cgi)
* SQL Database 或本機 SQL Server 實例

### <a name="download-and-run-the-sample-app"></a>下載及執行範例應用程式

若要建置 JAR 檔案並開始使用範例專案，請執行下列步驟：

1. 複製包含用戶端程式庫和範例應用程式的 [GitHub 存放庫](https://github.com/Microsoft/elastic-db-tools-for-java)。

2. 編輯 _./sample/src/main/resources/resource.properties_ 檔案，以設定下列項目：
    * TEST_CONN_USER
    * TEST_CONN_PASSWORD
    * TEST_CONN_SERVER_NAME

3. 若要建置範例專案，請在 _./sample_ 目錄中執行下列命令：

    ```
    mvn install
    ```

4. 若要啟動範例專案，請在 _./sample_ 目錄中執行下列命令：

    ```
    mvn -q exec:java "-Dexec.mainClass=com.microsoft.azure.elasticdb.samples.elasticscalestarterkit.Program"
    ```

5. 若要深入了解用戶端程式庫功能，請利用各種選項進行實驗。 請隨意探索程式碼，以了解範例應用程式的實作。

    ![進度 - Java][5]

恭喜！ 您已使用彈性資料庫工具，在 Azure SQL Database 上成功建置並執行您的第一個分區化應用程式。 使用 Visual Studio 或 SQL Server Management Studio 連接到您的資料庫，並快速查看範例所建立的分區。 您會看見範例所建立的新範例分區資料庫和分區對應管理員資料庫。

若要將用戶端程式庫加入您自己的 Maven 專案中，請在您的 POM 檔案中加入下列相依性：

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>elastic-db-tools</artifactId>
    <version>1.0.0</version>
</dependency>
```

## <a name="elastic-database-tools-for-net"></a>適用於 .NET 的彈性資料庫工具

### <a name="prerequisites"></a>先決條件

* 含 C# 的 Visual Studio 2012 或更新版本。 請在 [Visual Studio 下載](https://www.visualstudio.com/downloads/download-visual-studio-vs.aspx)上下載免費版本。
* NuGet 2.7 或更新版本。 若要取得最新版本，請參閱[安裝 NuGet](https://docs.nuget.org/docs/start-here/installing-nuget)。

### <a name="download-and-run-the-sample-app"></a>下載及執行範例應用程式

若要安裝程式庫，請移至 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。 程式庫是使用範例應用程式所安裝，如下一節所述。

若要下載及執行範例，請遵循下列步驟：

1. 下載 [適用于 AZURE SQL 的彈性 DB 工具-開始使用範例](https://github.com/Azure/elastic-db-tools)。 將範例解壓縮至您選擇的位置。

2. 若要建立專案，請從 *彈性-* -----master 目錄開啟 *ElasticDatabaseTools .sln* 方案。 

3. 將 *>elasticscalestarterkit.sln* 專案設定為啟始專案。

4. 在 *>elasticscalestarterkit.sln* 專案中，開啟 *App.config* 檔案。 然後依照檔案中的指示，將您的伺服器名稱和登入資訊 (使用者名稱和密碼) 。

5. 建置並執行應用程式。 出現提示時，允許 Visual Studio 還原解決方案的 NuGet 套件。 此動作會從 NuGet 下載最新版的彈性資料庫用戶端程式庫。

6. 若要深入了解用戶端程式庫功能，請利用各種選項進行實驗。 請記下應用程式在主控台輸出中採用的步驟，並盡情探索其後的程式碼。

   ![進度][4]

恭喜！ 您已使用彈性資料庫工具，在 SQL Database 上成功建置並執行您的第一個分區化應用程式。 使用 Visual Studio 或 SQL Server Management Studio 連接到您的資料庫，並快速查看範例所建立的分區。 您會看見範例所建立的新範例分區資料庫和分區對應管理員資料庫。

> [!IMPORTANT]
> 建議您一律使用最新版的 Management Studio，如此就能與 Azure 及 SQL Database 更新保持同步。 [更新 SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)。

## <a name="key-pieces-of-the-code-sample"></a>程式碼範例的主要部分

* **管理分區和分區對應**：此程式碼會解說如何使用 *ShardManagementUtils.cs* 檔案中的分區、範圍和對應。 如需詳細資訊，請參閱[使用分區對應管理員相應放大資料庫](https://go.microsoft.com/?linkid=9862595)。  

* **資料相依路由**：*DataDependentRoutingSample.cs* 檔案中示範如何將交易路由傳送至正確的分區。 如需詳細資訊，請參閱[資料相依路由](https://go.microsoft.com/?linkid=9862596)。

* **查詢多個分區**：*MultiShardQuerySample.cs* 檔案中說明如何在各個分區間進行查詢。 如需詳細資訊，請參閱[多分區查詢](https://go.microsoft.com/?linkid=9862597)。

* **新增空的分區**：反覆新增空分區的作業，是由 *CreateShardSample.cs* 檔案中的程式碼所執行。 如需詳細資訊，請參閱[使用分區對應管理員相應放大資料庫](https://go.microsoft.com/?linkid=9862595)。

## <a name="other-elastic-scale-operations"></a>其他 Elastic Scale 作業

* **分割現有的分區**：分割分區的功能是透過分割合併工具來提供。 如需詳細資訊，請參閱[在向外延展的雲端資料庫之間移動資料](elastic-scale-overview-split-and-merge.md)。

* **合併現有的分區**：分區合併也可使用分割合併工具來執行。 如需詳細資訊，請參閱[在向外延展的雲端資料庫之間移動資料](elastic-scale-overview-split-and-merge.md)。

## <a name="cost"></a>成本

彈性資料庫工具程式庫是免費的。 當您使用彈性資料庫工具時，不會衍生除了您的 Azure 使用成本以外的任何額外費用。

例如，範例應用程式會建立新資料庫。 此容量的費用取決於您選擇的 SQL Database 版本，以及您應用程式的 Azure 使用量。

如需定價資訊，請參閱 [SQL Database 定價詳細資料](https://azure.microsoft.com/pricing/details/sql-database/)。

## <a name="next-steps"></a>後續步驟

如需有關彈性資料庫工具的詳細資訊，請參閱下列文章：

* 程式碼範例：
  * 彈性資料庫工具 ([.NET](https://github.com/Azure/elastic-db-tools)、[Java](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-elasticdb-tools%22))
  * [適用於 Azure SQL 的彈性資料庫工具 - Entity Framework 整合](https://code.msdn.microsoft.com/Elastic-Scale-with-Azure-bae904ba?SRC=VSIDE)
  * [指令碼中心的分區彈性](https://gallery.technet.microsoft.com/scriptcenter/Elastic-Scale-Shard-c9530cbe)
* 部落格：[Elastic Scale 公告 (英文)](https://azure.microsoft.com/blog/20../../introducing-elastic-scale-preview-for-azure-sql-database/)
* 第 9 頻道：[Elastic Scale 概觀影片 (英文)](https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Database-Elastic-Scale)
* 討論論壇： [Microsoft 問&Azure SQL Database 的問題頁面](/answers/topics/azure-sql-database.html)
* 若要測量效能︰ [分區對應管理員的效能計數器](elastic-database-client-library.md)

<!--Anchors-->
[The Elastic Scale Sample Application]: #The-Elastic-Scale-Sample-Application
[Download and Run the Sample App]: #Download-and-Run-the-Sample-App
[Cost]: #Cost
[Next steps]: #next-steps

<!--Image references-->
[1]: ./media/elastic-scale-get-started/newProject.png
[2]: ./media/elastic-scale-get-started/click-online.png
[3]: ./media/elastic-scale-get-started/click-CSharp.png
[4]: ./media/elastic-scale-get-started/output2.png
[5]: ./media/elastic-scale-get-started/java-client-library.PNG