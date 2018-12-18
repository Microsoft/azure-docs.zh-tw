---
title: Azure SQL Database 連線和查詢快速入門 | Microsoft Docs
description: Azure SQL Database 快速入門說明如何連線及查詢 Azure SQL 資料庫。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 04/24/2018
ms.openlocfilehash: 576e0adca21f5cd3313893035ff66bd83a0a0164
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47063518"
---
# <a name="azure-sql-database-connect-and-query-quickstarts"></a>Azure SQL Database 連線和查詢快速入門

以下文件包含的連結可將您引導至 Azure 範例，以了解如何連線及查詢 Azure SQL 資料庫。 它也會提供傳輸層安全性的一些建議。

## <a name="quickstarts"></a>快速入門

| |  |
|---|---|
|[SQL Server Management Studio](sql-database-connect-query-ssms.md)|此快速入門示範如何使用 SSMS 來連線至 Azure SQL Database，然後使用 Transact-SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。|
|[Azure Data Studio](https://docs.microsoft.com/sql/azure-data-studio/quickstart-sql-database?toc=/azure/sql-database/toc.json)|此快速入門示範如何使用 Azure Data Studio 連線至 Azure SQL 資料庫，然後使用 Transact-SQL (T-SQL) 陳述式建立 Azure Data Studio 教學課程中使用的 TutorialDB。|
|[Azure 入口網站](sql-database-connect-query-portal.md)|此快速入門會示範如何使用查詢編輯器來連線至 SQL Database，然後使用 Transact-SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。|
|[Visual Studio Code](sql-database-connect-query-vscode.md)|此快速入門示範如何使用 Visual Studio Code 來連線至 Azure SQL 資料庫，然後使用 Transact-SQL 陳述式來查詢、插入、更新和刪除資料庫中的資料。|
|[.NET 搭配 Visual Studio](sql-database-connect-query-dotnet-visual-studio.md)|此快速入門示範如何搭配使用 .NET Framework 與 Visual Studio 建立 C# 程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|[.NET Core](sql-database-connect-query-dotnet-core.md)|此快速入門示範如何在 Windows/Linux/macOS 上使用 .NET Core 建立 C# 程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|[Go](sql-database-connect-query-go.md)|此快速入門示範如何使用 Go 連線至 Azure SQL 資料庫。 也會示範用以查詢及修改資料的 TRANSACT-SQL 陳述式。|
|[Java](sql-database-connect-query-java.md)|此快速入門示範如何使用 JAVA 來連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|[Node.js](sql-database-connect-query-nodejs.md)|此快速入門示範如何使用 Node.js 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|[PHP](sql-database-connect-query-php.md)|此快速入門示範如何使用 PHP 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|[Python](sql-database-connect-query-python.md)|此快速入門示範如何使用 Python 來連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。 |
|[Ruby](sql-database-connect-query-ruby.md)|此快速入門示範如何使用 Ruby 建立程式以連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。|
|||

## <a name="tls-considerations-for-sql-database-connectivity"></a>SQL Database 連線的 TLS 考量
Microsoft 針對連線至 Azure SQL Database 提供或支援的所有驅動程式都會使用傳輸層安全性 (TLS)。 不需要特別設定。 對於 SQL Server 或 Azure SQL Database 的所有連線，我們建議所有應用程式設定下列組態或其對等項目：

 - **Encrypt = 開啟**
 - **TrustServerCertificate = 關閉**

某些系統會對這些組態關鍵字使用不同但對等的關鍵字。 這些組態可確保用戶端驅動程式會驗證從伺服器接收的 TLS 憑證的身分識別。

如果您需要符合支付卡產業 - 資料安全性標準 (PCI-DSS)，我們也建議在用戶端上停用 TLS 1.1 和 1.0。

根據預設，非 Microsoft 驅動程式可能不會使用 TLS。 這可能是連線至 Azure SQL Database 時的一個因素。 具有內嵌驅動程式的應用程式可能無法讓您控制這些連線設定。 我們建議您在與敏感性資料互動的系統上使用這類驅動程式和應用程式之前，先檢查它們的安全性。

## <a name="next-steps"></a>後續步驟

如需連線架構資訊，請參閱 [Azure SQL Database 連線架構](sql-database-connectivity-architecture.md)。