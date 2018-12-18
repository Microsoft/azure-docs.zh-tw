---
title: 使用 JAVA 查詢 Azure SQL Database | Microsoft Docs
description: 本主題說明如何使用 JAVA 來建立連線到 Azure SQL Database 的程式，並使用 Transact-SQL 陳述式查詢。
services: sql-database
author: ajlam
manager: craigg
ms.service: sql-database
ms.custom: mvc,develop apps
ms.devlang: java
ms.topic: quickstart
ms.date: 04/01/2018
ms.author: andrela
ms.openlocfilehash: e19006d177777b18e1665ec5039f9c28f7e54df5
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38704635"
---
# <a name="use-java-to-query-an-azure-sql-database"></a>使用 JAVA 查詢 Azure SQL 資料庫

此快速入門示範如何使用 [JAVA](https://docs.microsoft.com/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server) 來連線至 Azure SQL 資料庫，並使用 Transact-SQL 陳述式來查詢資料。

## <a name="prerequisites"></a>先決條件

若要完成本快速入門，請確定您具備下列必要條件︰

[!INCLUDE [prerequisites-create-db](../../includes/sql-database-connect-query-prerequisites-create-db-includes.md)]

- 在此快速入門中，您所使用電腦的公用 IP 位址[伺服器層級防火牆規則](sql-database-get-started-portal.md#create-a-server-level-firewall-rule)。

- 您已安裝適用於您作業系統的 JAVA 和相關軟體：

    - **MacOS**：安裝 Homebrew 和 JAVA，然後再安裝 Maven。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/java/mac/)。
    - **Ubuntu**：安裝 JAVA Development Kit，並安裝 Maven。 請參閱[步驟 1.2、1.3 和 1.4](https://www.microsoft.com/sql-server/developer-get-started/java/ubuntu/)。
    - **Windows**：安裝 JAVA Development Kit，並安裝 Maven。 請參閱[步驟 1.2 和 1.3](https://www.microsoft.com/sql-server/developer-get-started/java/windows/)。    

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

[!INCLUDE [prerequisites-server-connection-info](../../includes/sql-database-connect-query-prerequisites-server-connection-info-includes.md)]

## <a name="create-maven-project-and-dependencies"></a>**建立 Maven 專案和相依性**
1. 從終端機，建立名為 **sqltest**的新 Maven 專案。 

   ```bash
   mvn archetype:generate "-DgroupId=com.sqldbsamples" "-DartifactId=sqltest" "-DarchetypeArtifactId=maven-archetype-quickstart" "-Dversion=1.0.0"
   ```

2. 在出現提示時按下 Y 鍵 。
3. 將目錄切換到 **sqltest** 並使用您慣用的文字編輯器開啟 pom.xml。  使用下列代碼將 **Microsoft JDBC Driver for SQL Server** 新增到專案的相依性：

   ```xml
   <dependency>
       <groupId>com.microsoft.sqlserver</groupId>
       <artifactId>mssql-jdbc</artifactId>
       <version>6.4.0.jre8</version>
   </dependency>
   ```

4. 此外，在 pom.xml 中，將下列屬性新增至您的專案。  如果您沒有屬性區段，您可以將它新增在相依性後面。

   ```xml
   <properties>
       <maven.compiler.source>1.8</maven.compiler.source>
       <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```

5. 儲存並關閉 pom.xml。

## <a name="insert-code-to-query-sql-database"></a>插入程式碼以查詢 SQL 資料庫

1. 您的 Maven 專案中應該已經有名為 App.java 的檔案，位於：\sqltest\src\main\java\com\sqlsamples\App.java

2. 開啟檔案並使用下列程式碼取代其內容，並為您的伺服器、資料庫、使用者和密碼新增適當的值。

   ```java
   package com.sqldbsamples;

   import java.sql.Connection;
   import java.sql.Statement;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.DriverManager;

   public class App {

    public static void main(String[] args) {
    
        // Connect to database
           String hostName = "your_server.database.windows.net";
           String dbName = "your_database";
           String user = "your_username";
           String password = "your_password";
           String url = String.format("jdbc:sqlserver://%s:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", hostName, dbName, user, password);
           Connection connection = null;

           try {
                   connection = DriverManager.getConnection(url);
                   String schema = connection.getSchema();
                   System.out.println("Successful connection - Schema: " + schema);

                   System.out.println("Query data example:");
                   System.out.println("=========================================");

                   // Create and execute a SELECT SQL statement.
                   String selectSql = "SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName " 
                       + "FROM [SalesLT].[ProductCategory] pc "  
                       + "JOIN [SalesLT].[Product] p ON pc.productcategoryid = p.productcategoryid";
                
                   try (Statement statement = connection.createStatement();
                       ResultSet resultSet = statement.executeQuery(selectSql)) {

                           // Print results from select statement
                           System.out.println("Top 20 categories:");
                           while (resultSet.next())
                           {
                               System.out.println(resultSet.getString(1) + " "
                                   + resultSet.getString(2));
                           }
                    connection.close();
                   }                   
           }
           catch (Exception e) {
                   e.printStackTrace();
           }
       }
   }
   ```

## <a name="run-the-code"></a>執行程式碼

1. 在命令提示字元中，執行下列命令：

   ```bash
   mvn package
   mvn -q exec:java "-Dexec.mainClass=com.sqldbsamples.App"
   ```

2. 請確認前 20 個資料列已傳回，然後關閉應用程式視窗。


## <a name="next-steps"></a>後續步驟
- [設計您的第一個 Azure SQL Database](sql-database-design-first-database.md)
- [Microsoft JDBC Driver for SQL Server](https://github.com/microsoft/mssql-jdbc)
- [回報問題/發問](https://github.com/microsoft/mssql-jdbc/issues)

