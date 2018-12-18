---
title: 使用 Visual Studio 和 .NET 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 Visual Studio 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: dotnet
ms.topic: quickstart
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 04/01/2018
ms.openlocfilehash: e6eaaa67914579edd5c3b53362835aa1ad168eca
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47064028"
---
# <a name="use-net-c-with-visual-studio-to-connect-and-query-an-azure-sql-database"></a>使用 .NET (C#) 搭配 Visual Studio 連線及查詢 Azure SQL database

此快速入門示範如何搭配使用 [.NET Framework](https://www.microsoft.com/net/) 與 Visual Studio 建立 C# 程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。

## <a name="prerequisites"></a>必要條件

若要完成本快速入門，請確定您具備下列項目︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。

- 安裝 [Visual Studio Community 2017、Visual Studio Professional 2017 或 Visual Studio Enterprise 2017](https://www.visualstudio.com/downloads/)。

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

#### <a name="for-adonet"></a>針對 ADO.NET

1. 按一下 [顯示資料庫連接字串] 以繼續。

2. 檢閱完整的 **ADO.NET** 連接字串。

    ![ADO.NET 連接字串](./media/sql-database-connect-query-dotnet/adonet-connection-string.png)

> [!IMPORTANT]
> 在您執行本教學課程的電腦上，公用 IP 位址必須有防火牆規則。 如果您在不同電腦上或有不同的公用 IP 位址，請建立[使用 Azure 入口網站的伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。 
>
  
## <a name="create-a-new-visual-studio-project"></a>建立新的 Visual Studio 專案

1. 在 Visual Studio 中，選擇 [檔案]、[新增]、[專案]。 
2. 在 [新增專案] 對話方塊中，展開 [Visual C#]。
3. 選取 [主控台應用程式] 並輸入 sqltest 作為專案名稱。
4. 按一下 [確定] 以在 Visual Studio 中建立並開啟新專案
4. 在 [方案總管] 中，以滑鼠右鍵按一下 [sqltest]，然後按一下 [管理 NuGet 套件]。 
5. 在 [瀏覽] 索引標籤上搜尋 ```System.Data.SqlClient```，並在找到後加以選取。
6. 在 **System.Data.SqlClient** 頁面上，按一下 [安裝]。
7. 當安裝完成時，檢閱所做的變更，然後按一下 [確定] 來關閉 [預覽] 視窗。 
8. 如果 [授權接受] 視窗出現時，按一下 [我接受]。

## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫
1. 切換到 (或視需要開啟) **Program.cs**

2. 使用下列程式碼取代 **Program.cs**的內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

```csharp
using System;
using System.Data.SqlClient;
using System.Text;

namespace sqltest
{
    class Program
    {
        static void Main(string[] args)
        {
            try 
            { 
                SqlConnectionStringBuilder builder = new SqlConnectionStringBuilder();
                builder.DataSource = "your_server.database.windows.net"; 
                builder.UserID = "your_user";            
                builder.Password = "your_password";     
                builder.InitialCatalog = "your_database";

                using (SqlConnection connection = new SqlConnection(builder.ConnectionString))
                {
                    Console.WriteLine("\nQuery data example:");
                    Console.WriteLine("=========================================\n");
                    
                    connection.Open();       
                    StringBuilder sb = new StringBuilder();
                    sb.Append("SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName ");
                    sb.Append("FROM [SalesLT].[ProductCategory] pc ");
                    sb.Append("JOIN [SalesLT].[Product] p ");
                    sb.Append("ON pc.productcategoryid = p.productcategoryid;");
                    String sql = sb.ToString();

                    using (SqlCommand command = new SqlCommand(sql, connection))
                    {
                        using (SqlDataReader reader = command.ExecuteReader())
                        {
                            while (reader.Read())
                            {
                                Console.WriteLine("{0} {1}", reader.GetString(0), reader.GetString(1));
                            }
                        }
                    }                    
                }
            }
            catch (SqlException e)
            {
                Console.WriteLine(e.ToString());
            }
            Console.ReadLine();
        }
    }
}
```

## <a name="run-the-code"></a>執行程式碼

1. 按 **F5** 鍵執行應用程式。
2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。

## <a name="next-steps"></a>後續步驟

- 了解如何在 Windows/Linux/macOS 中[使用 .NET Core 來連線及查詢 Azure SQL 資料庫](sql-database-connect-query-dotnet-core.md)。  
- 了解 [使用命令列以開始使用在 Windows/Linux/macOS 上的 .NET Core](/dotnet/core/tutorials/using-with-xplat-cli)。
- 深入了解如何[使用 SSMS 設計您的第一個 Azure SQL 資料庫](sql-database-design-first-database.md)或[使用 .NET 設計您的第一個 Azure SQL 資料庫](sql-database-design-first-database-csharp.md)。
- 如需 .NET 的詳細資訊，請參閱 [.NET 文件](https://docs.microsoft.com/dotnet/)。
- [重試邏輯範例：使用 ADO.NET 彈性連接到 SQL][step-4-connect-resiliently-to-sql-with-ado-net-a78n]


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-to-sql-with-ado-net

