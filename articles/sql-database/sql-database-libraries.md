---
title: SQL Database 的連接庫 | Microsoft Docs
description: 提供模組下載連結，這些模組可讓開發人員從各種用戶端程式設計語言連線到 SQL Server 與 SQL Database。
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: genemi
ms.openlocfilehash: cb9bc6dc21d043bfa2d5b278eec4d91494263515
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34647522"
---
# <a name="connectivity-libraries-and-frameworks-for-sql-server"></a>適用於 SQL Server 的連線程式庫和架構

請查看我們的[快速入門教學課程](http://aka.ms/sqldev)來快速地開始使用程式設計語言，例如 C#、Java、Node.js、PHP 和 Python。 然後，在 Linux 或 Windows 上使用 SQL Server，或者在 macOS 上使用 Docker 建置應用程式。

下表列出連線程式庫或驅動程式，用戶端應用程式可以用來從各種不同的語言連線到內部部署或雲端上執行之 SQL Server 並使用。 您可以在 Linux、Windows 或 Docker 上使用它們，並使用它們來連線到 Azure SQL Database 和 Azure SQL 資料倉儲。 

| 語言 | 平台 | 其他資源 | 下載 | 開始使用 |
| :-- | :-- | :-- | :-- | :-- |
| C# | Windows、Linux、macOS | [Microsoft ADO.NET for SQL Server](https://docs.microsoft.com/sql/connect/ado-net/microsoft-ado-net-for-sql-server) | [下載](https://www.microsoft.com/net/download/) | [開始使用](https://www.microsoft.com/sql-server/developer-get-started/csharp/ubuntu)
| Java | Windows、Linux、macOS | [Microsoft JDBC Driver for SQL Server](http://msdn.microsoft.com/library/mt484311.aspx) | [下載](https://go.microsoft.com/fwlink/?linkid=852460) |  [開始使用](https://www.microsoft.com/sql-server/developer-get-started/java/ubuntu)
| PHP | Windows、Linux、macOS| [PHP SQL Driver for SQL Server](http://msdn.microsoft.com/library/dn865013.aspx) | 作業系統： <br/> \* [Windows](https://www.microsoft.com/download/details.aspx?id=55642) <br/> \* [Linux](https://github.com/Microsoft/msphpsql/tree/dev#install-unix) <br/> \* [macOS](https://github.com/Microsoft/msphpsql/tree/dev#install-unix) |  [開始使用](https://www.microsoft.com/sql-server/developer-get-started/php/ubuntu)
| Node.js | Windows、Linux、macOS | [Node.js Driver for SQL Server](http://msdn.microsoft.com/library/mt652093.aspx) | [安裝](https://msdn.microsoft.com/library/mt652094.aspx) |  [開始使用](https://www.microsoft.com/sql-server/developer-get-started/node/ubuntu)
| Python | Windows、Linux、macOS | [Python SQL 驅動程式](http://msdn.microsoft.com/library/mt652092.aspx) | 安裝選項： <br/> \* [pymssql](https://msdn.microsoft.com/library/mt694094.aspx) <br/> \* [pyodbc](http://msdn.microsoft.com/library/mt763257.aspx) |  [開始使用](https://www.microsoft.com/sql-server/developer-get-started/python/ubuntu)
| Ruby | Windows、Linux、macOS | [Ruby Driver for SQL Server](http://msdn.microsoft.com/library/mt691981.aspx) | [安裝](https://msdn.microsoft.com/library/mt711041.aspx) | [開始使用](https://www.microsoft.com/sql-server/developer-get-started/ruby/ubuntu)
| C++ | Windows、Linux、macOS | [Microsoft ODBC Driver for SQL Server](https://msdn.microsoft.com/library/mt654048(v=sql.1).aspx) | [下載](https://msdn.microsoft.com/library/mt654048(v=sql.1).aspx) |  

下表列出物件關聯式對應 (ORM) 架構和 Web 架構的範例，用戶端應用程式可搭配內部部署或雲端上執行之 SQL Server 使用。 您可以在 Linux、Windows 或 Docker 上使用架構，並使用它們來連線到 SQL Database 和 SQL 資料倉儲。 

| 語言 | 平台 | ORM |
| :-- | :-- | :-- |
| C# | Windows、Linux、macOS | [Entity Framework](https://docs.microsoft.com/ef)<br>[Entity Framework Core](https://docs.microsoft.com/ef/core/index) |
| Java | Windows、Linux、macOS |[Hibernate ORM](http://hibernate.org/orm)|
| PHP | Windows、Linux | [Laravel (Eloquent)](https://laravel.com/docs/5.0/eloquent) |
| Node.js | Windows、Linux、macOS | [Sequelize ORM](http://docs.sequelizejs.com) |
| Python | Windows、Linux、macOS |[Django](https://www.djangoproject.com/) |
| Ruby | Windows、Linux、macOS | [Ruby on Rails](http://rubyonrails.org/) |
||||

## <a name="related-links"></a>相關連結
- [SQL Server 驅動程式](http://msdn.microsoft.com/library/mt654049.aspx)，適用於從用戶端應用程式連線
- 連線到 SQL Database：
    - [使用 .NET (C#) 連接到 SQL Database](sql-database-connect-query-dotnet.md)
    - [使用 PHP 連接到 SQL Database](sql-database-connect-query-php.md)
    - [使用 Node.js 連接到 SQL Database](sql-database-connect-query-nodejs.md)
    - [使用 Java 連接到 SQL Database](sql-database-connect-query-java.md)
    - [使用 Python 連接到 SQL Database](sql-database-connect-query-python.md)
    - [使用 Ruby 連接到 SQL Database](sql-database-connect-query-ruby.md)
- 重試邏輯程式碼範例：
    - [使用 ADO.NET 彈性連接到 SQL][step-4-connect-resiliently-to-sql-with-ado-net-a78n]
    - [使用 PHP 復原連線 SQL][step-4-connect-resiliently-to-sql-with-php-p42h]


<!-- Link references. -->

[step-4-connect-resiliently-to-sql-with-ado-net-a78n]: https://docs.microsoft.com/sql/connect/ado-net/step-4-connect-resiliently-to-sql-with-ado-net

[step-4-connect-resiliently-to-sql-with-php-p42h]: https://docs.microsoft.com/sql/connect/php/step-4-connect-resiliently-to-sql-with-php

