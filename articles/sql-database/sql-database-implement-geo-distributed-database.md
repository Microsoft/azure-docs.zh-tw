---
title: 實作異地分散的 Azure SQL Database 解決方案| Microsoft Docs
description: 了解如何設定您的 Azure SQL Database 和應用程式來容錯移轉至複寫資料庫，並測試容錯移轉。
services: sql-database
ms.service: sql-database
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: anosov1960
ms.author: sashan
ms.reviewer: carlrab
manager: craigg
ms.date: 09/07/2018
ms.openlocfilehash: 65cf954f5d91176715181620671f620264069bdc
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47166243"
---
# <a name="implement-a-geo-distributed-database"></a>實作異地分散資料庫

在本教學課程中，您要設定 Azure SQL Database 和應用程式來容錯移轉到遠端區域，然後測試您的容錯移轉計畫。 您會了解如何： 

> [!div class="checklist"]
> * 建立資料庫使用者並授與其權限
> * 設定資料庫層級防火牆規則
> * 建立[異地複寫容錯移轉群組](sql-database-geo-replication-overview.md)
> * 建立及編譯 Java 應用程式以查詢 Azure SQL Database
> * 執行災害復原演練

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。


## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定已完成下列必要條件：

- 已安裝最新的 [Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。 
- 已安裝 Azure SQL Database。 本教學課程使用下列其中一個快速入門中，名稱為 **mySampleDatabase** 的 AdventureWorksLT 範例資料庫︰

   - [建立 DB - 入口網站](sql-database-get-started-portal.md)
   - [建立 DB - CLI](sql-database-cli-samples.md)
   - [建立 DB - PowerShell](sql-database-powershell-samples.md)

- 已找出對您資料庫執行 SQL 指令碼的方法，您可以使用下列其中一個查詢工具：
   - [Azure 入口網站](https://portal.azure.com)中的查詢編輯器。 如需在 Azure 入口網站中使用查詢編輯器的詳細資訊，請參閱[使用查詢編輯器連線和查詢](sql-database-get-started-portal.md#query-the-sql-database)。
   - 最新版 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) 是整合式環境，可用來管理任何 SQL 基礎結構，範圍從 Microsoft Windows 的 SQL Server 到 SQL Database。
   - 最新版 [Visual Studio Code](https://code.visualstudio.com/docs) 是一個圖形化程式碼編輯器，適用於支援延伸模組的 Linux、macOS 和 Windows，包括可查詢 Microsoft SQL Server、Azure SQL Database 和 SQL 資料倉儲的 [mssql 延伸模組](https://aka.ms/mssql-marketplace)。 如需搭配使用此工具與 Azure SQL Database 的詳細資訊，請參閱[使用 VS Code 連線及查詢](sql-database-connect-query-vscode.md)。 

## <a name="create-database-users-and-grant-permissions"></a>建立資料庫使用者並授與權限

連線到您的資料庫，並使用下列其中一個查詢工具來建立使用者帳戶：

- Azure 入口網站中的查詢編輯器
- SQL Server Management Studio
- Visual Studio Code

這些使用者帳戶會自動複寫到次要伺服器 (並保持同步)。 若要使用 SQL Server Management Studio 或 Visual Studio Code，如果您從 IP 位址尚未設定防火牆的用戶端連線，則可能需要設定防火牆規則。 如需詳細步驟，請參閱[建立伺服器層級防火牆規則](sql-database-get-started-portal-firewall.md)。

- 在查詢視窗中，執行下列查詢以在資料庫中建立兩個使用者帳戶。 此指令碼會將 **db_owner** 權限授與 **app_admin** 帳戶，並將 **SELECT** 和 **UPDATE** 權限授與 **app_user** 帳戶。 

   ```sql
   CREATE USER app_admin WITH PASSWORD = 'ChangeYourPassword1';
   --Add SQL user to db_owner role
   ALTER ROLE db_owner ADD MEMBER app_admin; 
   --Create additional SQL user
   CREATE USER app_user WITH PASSWORD = 'ChangeYourPassword1';
   --grant permission to SalesLT schema
   GRANT SELECT, INSERT, DELETE, UPDATE ON SalesLT.Product TO app_user;
   ```

## <a name="create-database-level-firewall"></a>建立資料庫層級防火牆

為 SQL Database 建立[資料庫層級防火牆規則](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-set-database-firewall-rule-azure-sql-database)。 此資料庫層級防火牆規則會自動複寫到您在本教學課程中建立的次要伺服器中。 為求簡化 (在本教學課程中)，請使用您在本教學課程中執行步驟之電腦的公用 IP 位址。 若要判斷用於目前電腦之伺服器層級防火牆規則的 IP 位址，請參閱[建立伺服器層級防火牆](sql-database-get-started-portal-firewall.md)。  

- 在開啟的查詢視窗中，將先前的查詢取代為下列查詢，將 IP 位址取代為環境的適當 IP 位址。  

   ```sql
   -- Create database-level firewall setting for your public IP address
   EXECUTE sp_set_database_firewall_rule @name = N'myGeoReplicationFirewallRule',@start_ip_address = '0.0.0.0', @end_ip_address = '0.0.0.0';
   ```

## <a name="create-an-active-geo-replication-auto-failover-group"></a>建立作用中異地複寫自動容錯移轉群組 

使用 Azure PowerShell，在現有 Azure SQL Server 與 Azure 區域的新空白 Azure SQL Server 之間建立[作用中異地複寫自動容錯移轉群組](sql-database-geo-replication-overview.md)，然後將範例資料庫新增到容錯移轉群組。

> [!IMPORTANT]
> 這些 Cmdlet 需要 Azure PowerShell 4.0。 [!INCLUDE [sample-powershell-install](../../includes/sample-powershell-install-no-ssh.md)]
>

1. 使用現有伺服器和範例資料庫的值來填入 PowerShell 指令碼的變數，並提供容錯移轉群組名稱的全域唯一值。

   ```powershell
   $adminlogin = "ServerAdmin"
   $password = "ChangeYourAdminPassword1"
   $myresourcegroupname = "<your resource group name>"
   $mylocation = "<your resource group location>"
   $myservername = "<your existing server name>"
   $mydatabasename = "mySampleDatabase"
   $mydrlocation = "<your disaster recovery location>"
   $mydrservername = "<your disaster recovery server name>"
   $myfailovergroupname = "<your unique failover group name>"
   ```

2. 在容錯移轉區域中建立空的備份伺服器。

   ```powershell
   $mydrserver = New-AzureRmSqlServer -ResourceGroupName $myresourcegroupname `
      -ServerName $mydrservername `
      -Location $mydrlocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $mydrserver   
   ```

3. 在兩部伺服器之間建立容錯移轉群組。

   ```powershell
   $myfailovergroup = New-AzureRMSqlDatabaseFailoverGroup `
      –ResourceGroupName $myresourcegroupname `
      -ServerName $myservername `
      -PartnerServerName $mydrservername  `
      –FailoverGroupName $myfailovergroupname `
      –FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   $myfailovergroup   
   ```

4. 在容錯移轉群組中新增資料庫。

   ```powershell
   $myfailovergroup = Get-AzureRmSqlDatabase `
      -ResourceGroupName $myresourcegroupname `
      -ServerName $myservername `
      -DatabaseName $mydatabasename | `
    Add-AzureRmSqlDatabaseToFailoverGroup `
      -ResourceGroupName $myresourcegroupname ` `
      -ServerName $myservername `
      -FailoverGroupName $myfailovergroupname
   $myfailovergroup   
   ```

## <a name="install-java-software"></a>安裝 Java 軟體

本節中的步驟假設您已熟悉使用 Java 進行開發且不熟悉 Azure SQL Database。 

### <a name="mac-os"></a>**Mac OS**
開啟您的終端機，然後瀏覽至您打算在其中建立 Java 專案的目錄。 輸入下列命令來安裝 **brew** 和 **Maven**： 

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
brew install maven
```

如需安裝及設定 Java 和 Maven 環境的詳細指引，請前往[使用 SQL Server 建置應用程式](https://www.microsoft.com/sql-server/developer-get-started/)，並依序選取 [Java] 和 [MacOS]，然後遵循步驟 1.2 和 1.3 中的詳細指示來設定 Java 和 Maven。

### <a name="linux-ubuntu"></a>**Linux (Ubuntu)**
開啟您的終端機，然後瀏覽至您打算在其中建立 Java 專案的目錄。 輸入下列命令來安裝 **Maven**：

```bash
sudo apt-get install maven
```

如需安裝及設定 Java 和 Maven 環境的詳細指引，請前往[使用 SQL Server 建置應用程式](https://www.microsoft.com/sql-server/developer-get-started/)，並依序選取 [Java] 和 [Ubuntu]，然後遵循步驟 1.2、1.3 和 1.4 中的詳細指示來設定 Java 和 Maven。

### <a name="windows"></a>**Windows**
使用官方安裝程式來安裝 [Maven](https://maven.apache.org/download.cgi)。 請使用 Maven 來協助管理相依性，並建置、測試和執行您的 Java 專案。 如需安裝及設定 Java 和 Maven 環境的詳細指引，請前往[使用 SQL Server 建置應用程式](https://www.microsoft.com/sql-server/developer-get-started/)，並依序選取 [Java] 和 [Windows]，然後遵循步驟 1.2 和 1.3 中的詳細指示來設定 Java 和 Maven。

## <a name="create-sqldbsample-project"></a>建立 SqlDbSample 專案

1. 在命令主控台 (例如 Bash) 中，建立 Maven 專案。 
   ```bash
   mvn archetype:generate "-DgroupId=com.sqldbsamples" "-DartifactId=SqlDbSample" "-DarchetypeArtifactId=maven-archetype-quickstart" "-Dversion=1.0.0"
   ```
2. 鍵入 **Y**，然後按一下 **Enter**。
3. 將目錄變更為新建立的專案。

   ```bash
   cd SqlDbSamples
   ```

4. 使用您慣用的編輯器，開啟專案資料夾中的 pom.xml 檔案。 

5. 開啟慣用文字編輯器，然後將下列數行複製並貼入 pom.xml 檔案，以將 Microsoft JDBC Driver for SQL Server 相依性新增至 Maven 專案。 請不要覆寫已預先填入檔案中的現有值。 JDBC 相依性必須貼入較大的 “dependencies” 區段 ( ) 內。   

   ```xml
   <dependency>
     <groupId>com.microsoft.sqlserver</groupId>
     <artifactId>mssql-jdbc</artifactId>
    <version>6.1.0.jre8</version>
   </dependency>
   ```

6. 指定要對其編譯專案的 Java 版本，方法是將下列 “properties” 區段新增至 pom.xml 檔案的 "dependencies" 區段後面。 

   ```xml
   <properties>
     <maven.compiler.source>1.8</maven.compiler.source>
     <maven.compiler.target>1.8</maven.compiler.target>
   </properties>
   ```
7. 將下列 "build" 區段新增至 pom.xml 檔案的 "properties" 區段後面，以支援 jars 中的資訊清單檔。       

   ```xml
   <build>
     <plugins>
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-jar-plugin</artifactId>
         <version>3.0.0</version>
         <configuration>
           <archive>
             <manifest>
               <mainClass>com.sqldbsamples.App</mainClass>
             </manifest>
           </archive>
        </configuration>
       </plugin>
     </plugins>
   </build>
   ```
8. 儲存並關閉 pom.xml 檔案。
9. 開啟 App.java 檔案 (C:\apache-maven-3.5.0\SqlDbSample\src\main\java\com\sqldbsamples\App.java)，並將內容取代為下列內容。 將容錯移轉群組名稱取代為您容錯移轉群組的名稱。 如果您已變更資料庫名稱、使用者或密碼的值，也請變更這些值。

   ```java
   package com.sqldbsamples;

   import java.sql.Connection;
   import java.sql.Statement;
   import java.sql.PreparedStatement;
   import java.sql.ResultSet;
   import java.sql.Timestamp;
   import java.sql.DriverManager;
   import java.util.Date;
   import java.util.concurrent.TimeUnit;

   public class App {

      private static final String FAILOVER_GROUP_NAME = "myfailovergroupname";
  
      private static final String DB_NAME = "mySampleDatabase";
      private static final String USER = "app_user";
      private static final String PASSWORD = "ChangeYourPassword1";

      private static final String READ_WRITE_URL = String.format("jdbc:sqlserver://%s.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);
      private static final String READ_ONLY_URL = String.format("jdbc:sqlserver://%s.secondary.database.windows.net:1433;database=%s;user=%s;password=%s;encrypt=true;hostNameInCertificate=*.database.windows.net;loginTimeout=30;", FAILOVER_GROUP_NAME, DB_NAME, USER, PASSWORD);

      public static void main(String[] args) {
         System.out.println("#######################################");
         System.out.println("## GEO DISTRIBUTED DATABASE TUTORIAL ##");
         System.out.println("#######################################");
         System.out.println(""); 

         int highWaterMark = getHighWaterMarkId();

         try {
            for(int i = 1; i < 1000; i++) {
                //  loop will run for about 1 hour
                System.out.print(i + ": insert on primary " + (insertData((highWaterMark + i))?"successful":"failed"));
                TimeUnit.SECONDS.sleep(1);
                System.out.print(", read from secondary " + (selectData((highWaterMark + i))?"successful":"failed") + "\n");
                TimeUnit.SECONDS.sleep(3);
            }
         } catch(Exception e) {
            e.printStackTrace();
      }
   }

   private static boolean insertData(int id) {
      // Insert data into the product table with a unique product name that we can use to find the product again later
      String sql = "INSERT INTO SalesLT.Product (Name, ProductNumber, Color, StandardCost, ListPrice, SellStartDate) VALUES (?,?,?,?,?,?);";

      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL); 
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         pstmt.setInt(2, 200989 + id + 10000);
         pstmt.setString(3, "Blue");
         pstmt.setDouble(4, 75.00);
         pstmt.setDouble(5, 89.99);
         pstmt.setTimestamp(6, new Timestamp(new Date().getTime()));
         return (1 == pstmt.executeUpdate());
      } catch (Exception e) {
         return false;
      }
   }

   private static boolean selectData(int id) {
      // Query the data that was previously inserted into the primary database from the geo replicated database
      String sql = "SELECT Name, Color, ListPrice FROM SalesLT.Product WHERE Name = ?";

      try (Connection connection = DriverManager.getConnection(READ_ONLY_URL); 
              PreparedStatement pstmt = connection.prepareStatement(sql)) {
         pstmt.setString(1, "BrandNewProduct" + id);
         try (ResultSet resultSet = pstmt.executeQuery()) {
            return resultSet.next();
         }
      } catch (Exception e) {
         return false;
      }
   }

   private static int getHighWaterMarkId() {
      // Query the high water mark id that is stored in the table to be able to make unique inserts 
      String sql = "SELECT MAX(ProductId) FROM SalesLT.Product";
      int result = 1;
        
      try (Connection connection = DriverManager.getConnection(READ_WRITE_URL); 
              Statement stmt = connection.createStatement();
              ResultSet resultSet = stmt.executeQuery(sql)) {
         if (resultSet.next()) {
             result =  resultSet.getInt(1);
            }
         } catch (Exception e) {
          e.printStackTrace();
         }
         return result;
      }
   }
   ```
6. 儲存並關閉 App.java 檔案。

## <a name="compile-and-run-the-sqldbsample-project"></a>編譯和執行 SqlDbSample 專案

1. 在命令主控台中，執行下列命令。

   ```bash
   mvn package
   ```
2. 完成後，請執行下列命令以執行應用程式 (除非您手動停止，否則應用程式大約會執行 1 小時)：

   ```bash
   mvn -q -e exec:java "-Dexec.mainClass=com.sqldbsamples.App"
   
   #######################################
   ## GEO DISTRIBUTED DATABASE TUTORIAL ##
   #######################################

   1. insert on primary successful, read from secondary successful
   2. insert on primary successful, read from secondary successful
   3. insert on primary successful, read from secondary successful
   ```

## <a name="perform-disaster-recovery-drill"></a>執行災害復原鑽研

1. 呼叫容錯移轉群組的手動容錯移轉。 

   ```powershell
   Switch-AzureRMSqlDatabaseFailoverGroup `
   -ResourceGroupName $myresourcegroupname  `
   -ServerName $mydrservername `
   -FailoverGroupName $myfailovergroupname
   ```

2. 在容錯移轉期間，觀察應用程式結果。 DNS 快取重新整理時，某些插入失敗。     

3. 找出災害復原伺服器正在執行的角色。

   ```powershell
   $mydrserver.ReplicationRole
   ```

4. 容錯回復。

   ```powershell
   Switch-AzureRMSqlDatabaseFailoverGroup `
   -ResourceGroupName $myresourcegroupname  `
   -ServerName $myservername `
   -FailoverGroupName $myfailovergroupname
   ```

5. 在容錯回復期間，觀察應用程式結果。 DNS 快取重新整理時，某些插入失敗。     

6. 找出災害復原伺服器正在執行的角色。

   ```powershell
   $fileovergroup = Get-AzureRMSqlDatabaseFailoverGroup `
      -FailoverGroupName $myfailovergroupname `
      -ResourceGroupName $myresourcegroupname `
      -ServerName $mydrservername
   $fileovergroup.ReplicationRole
   ```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何設定 Azure SQL 資料庫和應用程式來容錯移轉到遠端區域，然後測試您的容錯移轉計畫。  您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料庫使用者並授與其權限
> * 設定資料庫層級防火牆規則
> * 建立異地複寫容錯移轉群組
> * 建立及編譯 Java 應用程式以查詢 Azure SQL Database
> * 執行災害復原演練

前進到下一個教學課程，以使用 DMS 將 SQL Server 遷移至 Azure SQL Database 受控執行個體。

> [!div class="nextstepaction"]
>[使用 DMS 將 SQL Server 移轉至 Azure SQL Database 受控執行個體](../dms/tutorial-sql-server-to-managed-instance.md)

