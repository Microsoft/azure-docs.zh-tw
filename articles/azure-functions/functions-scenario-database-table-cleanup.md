---
title: 使用 Azure Functions 執行資料庫清除工作 | Microsoft Docs
description: 使用 Azure Functions 排程可連接到 Azure SQL Database 以定期清除資料列的工作。
services: functions
documentationcenter: na
author: ggailey777
manager: jeconnoc
ms.assetid: 076f5f95-f8d2-42c7-b7fd-6798856ba0bb
ms.service: azure-functions
ms.devlang: multiple
ms.topic: conceptual
ms.date: 05/22/2017
ms.author: glenga
ms.openlocfilehash: a257948c97437d6045f705acb02054928d22ff89
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44092864"
---
# <a name="use-azure-functions-to-connect-to-an-azure-sql-database"></a>使用 Azure Functions 連接到 Azure SQL Database
本主題示範如何使用 Azure Functions 建立可清除 Azure SQL Database 資料表中資料列的排程作業。 新的 C# 指令碼函數是根據 Azure 入口網站中預先定義的計時器觸發程序範本所建立。 若要支援此案例，您也必須在函式應用程式中設定資料庫連接字串以作為設定。 此案例會對資料庫使用大量作業。 

若要讓您的函式程序在 Mobile Apps 資料表中個別建立、讀取、更新及刪除 (CRUD) 作業，您應該改用 [Mobile Apps 繫結](functions-bindings-mobile-apps.md)。

## <a name="prerequisites"></a>必要條件

+ 本主題使用計時器觸發的函數。 完成主題[在 Azure 中建立由計時器觸發的函數](functions-create-scheduled-function.md)中的步驟，以建立此函數的 C# 版本。   

+ 本主題將示範在 AdventureWorksLT 範例資料庫的 **SalesOrderHeader** 資料表中執行大量清除作業的 Transact-SQL 命令。 若要建立 AdventureWorksLT 範例資料庫，請完成本主題[在 Azure 入口網站中建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)中的步驟。 

## <a name="get-connection-information"></a>取得連線資訊

完成[在 Azure 入口網站中建立 Azure SQL 資料庫](../sql-database/sql-database-get-started-portal.md)時，您必須取得所建立之資料庫的連接字串。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
 
3. 從左側功能表中選取 [SQL Database]，然後選取 [SQL 資料庫] 頁面上的資料庫。

4. 選取 [顯示資料庫連接字串]，然後複製完整的 **ADO.NET** 連接字串。 

    ![複製 ADO.NET 連接字串。](./media/functions-scenario-database-table-cleanup/adonet-connection-string.png)

## <a name="set-the-connection-string"></a>設定連接字串 

函式應用程式可在 Azure 中主控函式的執行。 在函數應用程式設定中儲存連接字串和其他機密資料是最佳做法。 使用應用程式設定可避免意外洩露連接字串與您的程式碼。 

1. 瀏覽至您建立的函數應用程式。[在 Azure 中建立由計時器觸發的函數](functions-create-scheduled-function.md)。

2. 選取 [平台功能] > [應用程式設定]。
   
    ![函數應用程式的應用程式設定。](./media/functions-scenario-database-table-cleanup/functions-app-service-settings.png)

2. 向下捲動至 [連接字串]，然後使用資料表中指定的設定來新增連接字串。
   
    ![將連接字串新增至函數應用程式設定。](./media/functions-scenario-database-table-cleanup/functions-app-service-settings-connection-strings.png)

    | 設定       | 建議的值 | 說明             | 
    | ------------ | ------------------ | --------------------- | 
    | **名稱**  |  sqldb_connection  | 用於存取函數程式碼的預存連接字串。    |
    | **值** | 複製的字串  | 貼上您在上一節中複製的連接字串，並將 `{your_username}` 和 `{your_password}` 預留位置取代為實際的值。 |
    | **類型** | SQL Database | 使用預設的 SQL Database 連接。 |   

3. 按一下 [檔案] 。

現在，您可以加入 C# 函數程式碼來連接到 SQL Database。

## <a name="update-your-function-code"></a>更新函數程式碼

1. 在入口網站的函數應用程式中，選取計時器觸發的函數。
 
3. 在現有 C# 指令碼函數程式碼頂端新增下列組件參考：

    ```cs
    #r "System.Configuration"
    #r "System.Data"
    ```
    >[!NOTE]
    >這些範例中的程式碼是來自入口網站的 C# 指令碼。 於本機開發先行編譯的 C# 函數時，必須改為將參考新增至本機專案中的這些組件。  

3. 將下列 `using` 陳述式加入至函數：
    ```cs
    using System.Configuration;
    using System.Data.SqlClient;
    using System.Threading.Tasks;
    ```

4. 使用下列程式碼來取代現有的 `Run` 函式：
    ```cs
    public static async Task Run(TimerInfo myTimer, TraceWriter log)
    {
        var str = ConfigurationManager.ConnectionStrings["sqldb_connection"].ConnectionString;
        using (SqlConnection conn = new SqlConnection(str))
        {
            conn.Open();
            var text = "UPDATE SalesLT.SalesOrderHeader " + 
                    "SET [Status] = 5  WHERE ShipDate < GetDate();";

            using (SqlCommand cmd = new SqlCommand(text, conn))
            {
                // Execute the command and log the # rows affected.
                var rows = await cmd.ExecuteNonQueryAsync();
                log.Info($"{rows} rows were updated");
            }
        }
    }
    ```

    此範例命令會以出貨日期作為基礎來更新 `Status` 資料行。 其應該會更新 32 個資料列。

5. 按一下 [儲存]、針對下一個函數執行監看 [記錄] 視窗，然後記下 **SalesOrderHeader** 資料表中更新的資料列數目。

    ![檢視函數記錄。](./media/functions-scenario-database-table-cleanup/functions-logs.png)

## <a name="next-steps"></a>後續步驟

接下來，了解如何將 Functions 與 Logic Apps 搭配使用以與其他服務整合。

> [!div class="nextstepaction"] 
> [建立與 Logic Apps 整合的函數](functions-twitter-email.md)

如需有關 Functions 的詳細資訊，請參閱下列主題：

* [Azure Functions 開發人員參考](functions-reference.md)  
  可供程式設計人員撰寫函數程式碼及定義觸發程序和繫結時參考。
* [測試 Azure Functions](functions-test-a-function.md)  
  說明可用於測試函式的各種工具和技巧。  
