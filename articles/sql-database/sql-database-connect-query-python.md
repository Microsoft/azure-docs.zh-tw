---
title: 使用 Python 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 Python 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
author: CarlRabeler
manager: craigg
ms.service: sql-database
ms.custom: mvc,develop apps
ms.devlang: python
ms.topic: quickstart
ms.date: 07/02/2018
ms.author: carlrab
ms.openlocfilehash: e88c069bed40bcdf1eae9d356403cc772a11ea85
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38704768"
---
# <a name="use-python-to-query-an-azure-sql-database"></a>使用 Python 查詢 Azure SQL 資料庫

 此快速入門示範如何使用 [Python](https://python.org) 來連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。 如需 SDK 的詳細資訊，請查看我們的[參考](https://docs.microsoft.com/python/api/overview/azure/sql)文件、pyodbc [範例](https://github.com/mkleehammer/pyodbc/wiki/Getting-started)及 [pyodbc](https://github.com/mkleehammer/pyodbc/wiki/) GitHub 存放庫。

## <a name="prerequisites"></a>先決條件

若要完成本快速入門，請確定您具備下列項目︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal.md#create-a-server-level-firewall-rule)。

- 您已安裝適用於您作業系統的 Python 和相關軟體：

    - **MacOS**：安裝 Homebrew 和 Python，安裝 ODBC 驅動程式和 SQLCMD，然後再安裝 Python Driver for SQL Server。 請參閱[步驟 1.2、1.3 和 2.1](https://www.microsoft.com/sql-server/developer-get-started/python/mac/)。
    - **Ubuntu**：安裝 Python 和其他必要的套件，然後安裝 Python Driver for SQL Server。 請參閱[步驟 1.2、1.3 和 2.1](https://www.microsoft.com/sql-server/developer-get-started/python/ubuntu/)。
    - **Windows**：安裝最新版的 Python (環境變數現在已為您設定好)，安裝 ODBC 驅動程式和 SQLCMD，然後再安裝 Python Driver for SQL Server。 請參閱[步驟 1.2、1.3 和 2.1](https://www.microsoft.com/sql-server/developer-get-started/python/windows/)。 

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]
    
## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫 

1. 在您慣用的文字編輯器中，建立新的檔案 **sqltest.py**。  

2. 使用下列程式碼取代內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

```Python
import pyodbc
server = 'your_server.database.windows.net'
database = 'your_database'
username = 'your_username'
password = 'your_password'
driver= '{ODBC Driver 13 for SQL Server}'
cnxn = pyodbc.connect('DRIVER='+driver+';SERVER='+server+';PORT=1433;DATABASE='+database+';UID='+username+';PWD='+ password)
cursor = cnxn.cursor()
cursor.execute("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName FROM [SalesLT].[ProductCategory] pc JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid")
row = cursor.fetchone()
while row:
    print (str(row[0]) + " " + str(row[1]))
    row = cursor.fetchone()
```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```Python
   python sqltest.py
   ```

2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。

## <a name="next-steps"></a>後續步驟

- [設計您的第一個 Azure SQL Database](sql-database-design-first-database.md)
- [Microsoft Python Drivers for SQL Server](https://docs.microsoft.com/sql/connect/python/python-driver-for-sql-server/)
- [Python 開發人員中心](https://azure.microsoft.com/develop/python/?v=17.23h)

