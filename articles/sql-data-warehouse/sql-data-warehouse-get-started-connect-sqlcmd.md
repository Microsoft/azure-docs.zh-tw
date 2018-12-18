---
title: 連線到 Azure SQL 資料倉儲 sqlcmd | Microsoft Docs
description: 使用 [sqlcmd][sqlcmd] 命令列公用程式來連接及查詢 Azure SQL 資料倉儲。
services: sql-data-warehouse
author: kavithaj
manager: craigg
ms.service: sql-data-warehouse
ms.topic: conceptual
ms.component: consume
ms.date: 04/17/2018
ms.author: kavithaj
ms.reviewer: igorstan
ms.openlocfilehash: 94f3955f9ce94fa52e89180fa649c4e412b80109
ms.sourcegitcommit: 2b2129fa6413230cf35ac18ff386d40d1e8d0677
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43247708"
---
# <a name="connect-to-sql-data-warehouse-with-sqlcmd"></a>使用 sqlcmd 連接到 SQL 資料倉儲
> [!div class="op_single_selector"]
> * [Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md)
> * [Azure Machine Learning](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md)
> * [Visual Studio](sql-data-warehouse-query-visual-studio.md)
> * [sqlcmd](sql-data-warehouse-get-started-connect-sqlcmd.md) 
> * [SSMS](sql-data-warehouse-query-ssms.md)
> 
> 

使用 [sqlcmd][sqlcmd] 命令列公用程式來連線及查詢 Azure SQL 資料倉儲。  

## <a name="1-connect"></a>1.連線
若要開始使用 [sqlcmd][sqlcmd]，請開啟命令提示字元，然後輸入 **sqlcmd** 並在後面加上 SQL 資料倉儲資料庫的連接字串。 連接字串需要下列參數：

* **伺服器 (-S)：** 採用 `<`Server Name`>`.database.windows.net 格式的伺服器
* **資料庫 (-d)：** 資料庫名稱。
* **啟用引號識別碼 (-I)：** 必須啟用引號識別碼，才能連接到 SQL 資料倉儲執行個體。

若要使用 SQL Server 驗證，您需要新增使用者名稱/密碼參數︰

* **使用者 (-U)：**`<`使用者`>`表單中的伺服器使用者
* **密碼 (-P)：** 與使用者相關聯的密碼。

例如，您的連接字串看起來可能如下所示︰

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
```

若要使用 Azure Active Directory 整合式驗證，您需要新增 Azure Active Directory 參數︰

* **Azure Active Directory 驗證 (-G)：** 使用 Azure Active Directory 進行驗證

例如，您的連接字串看起來可能如下所示︰

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -G -I
```

> [!NOTE]
> 您需要使用 Active Directory [啟用 Azure Active Directory 驗證](sql-data-warehouse-authentication.md) 以進行驗證。
> 
> 

## <a name="2-query"></a>2.查詢
連線之後，您可以對執行個體發出任何支援的 Transact-SQL 陳述式。  此範例會在互動模式中提交查詢。

```sql
C:\>sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I
1> SELECT name FROM sys.tables;
2> GO
3> QUIT
```

後續的這些範例會說明如何使用 -Q 選項或以管線將 SQL 傳送到 sqlcmd，來執行批次模式查詢。

```sql
sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I -Q "SELECT name FROM sys.tables;"
```

```sql
"SELECT name FROM sys.tables;" | sqlcmd -S MySqlDw.database.windows.net -d Adventure_Works -U myuser -P myP@ssword -I > .\tables.out
```

## <a name="next-steps"></a>後續步驟
如需 sqlcmd 中可用選項的詳細資訊，請參閱 [sqlcmd 文件][sqlcmd]。

<!--Image references-->

<!--Article references-->

<!--MSDN references--> 
[sqlcmd]: https://msdn.microsoft.com/library/ms162773.aspx
[Azure portal]: https://portal.azure.com

<!--Other Web references-->
