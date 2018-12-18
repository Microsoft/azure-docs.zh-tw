---
title: 教學課程：使用 SSMS 設計您的第一個 Azure SQL 資料庫 | Microsoft Docs
description: 了解如何利用 SQL Server Management Studio 設計您的第一個 Azure SQL 資料庫。
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: ''
ms.devlang: ''
ms.topic: tutorial
author: CarlRabeler
ms.author: carlrab
ms.reviewer: ''
manager: craigg
ms.date: 07/16/2018
ms.openlocfilehash: 2eebc083c4595a9190c60a60b8375b3a46296584
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162758"
---
# <a name="tutorial-design-your-first-azure-sql-database-using-ssms"></a>教學課程：使用 SSMS 設計您的第一個 Azure SQL 資料庫

Azure SQL Database 是 Microsoft Cloud (Azure) 中的關聯式資料庫即服務 (DBaaS)。 在本教學課程中，您會了解如何使用 Azure 入口網站和 [SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx) (SSMS)： 

> [!div class="checklist"]
> * 在 Azure 入口網站中建立資料庫*
> * 在 Azure 入口網站中設定伺服器層級的防火牆規則
> * 使用 SSMS 連接到資料庫
> * 使用 SSMS 建立資料表
> * 使用 BCP 大量載入資料
> * 使用 SSMS 查詢該資料

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

   >[!NOTE]
   > 為了進行此教學課程，我們會使用[以 DTU 為基礎的購買模型](sql-database-service-tiers-dtu.md)，但您也可以選擇[以虛擬核心為基礎的購買模型](sql-database-service-tiers-vcore.md)。 

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，請確定您已安裝︰
- 最新版的 [SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx) (SSMS)。
- 最新版的 [BCP 和 SQLCMD](https://www.microsoft.com/download/details.aspx?id=36433)。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-blank-sql-database"></a>建立空白 SQL Database

Azure SQL Database 會使用一組定義的[計算和儲存體資源](sql-database-service-tiers-dtu.md)建立。 此資料庫建立於 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)和 [Azure SQL Database 邏輯伺服器](sql-database-features.md)內。 

遵循以下步驟來建立空白 SQL 資料庫。 

1. 按一下 Azure 入口網站左上角的 [建立資源]。

2. 在 [新增] 頁面上，選取 [Azure Marketplace] 區段中的 [資料庫]，然後按一下 [精選] 區段中的 [SQL Database]。

   ![建立空白資料庫](./media/sql-database-design-first-database/create-empty-database.png)

3. 使用下列資訊填寫 SQL Database 表單，如上圖所示︰   

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **資料庫名稱** | mySampleDatabase | 如需有效的資料庫名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。 | 
   | **訂用帳戶** | 您的訂用帳戶  | 如需訂用帳戶的詳細資訊，請參閱[訂用帳戶](https://account.windowsazure.com/Subscriptions)。 |
   | **資源群組** | myResourceGroup | 如需有效的資源群組名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 |
   | **選取來源** | 空白資料庫 | 指定應建立空白資料庫。 |

4. 按一下 [伺服器] 為您的新資料庫建立及設定新的伺服器。 在**新伺服器表單**表單中填寫下列資訊︰ 

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **伺服器名稱** | 任何全域唯一名稱 | 如需有效的伺服器名稱，請參閱[命名規則和限制](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)。 | 
   | **伺服器管理員登入** | 任何有效名稱 | 如需有效的登入名稱，請參閱[資料庫識別碼](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)。|
   | **密碼** | 任何有效密碼 | 您的密碼至少要有 8 個字元，而且必須包含下列幾種字元的其中三種︰大寫字元、小寫字元、數字和非英數字元。 |
   | **位置** | 任何有效位置 | 如需區域的相關資訊，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。 |

   ![建立資料庫伺服器](./media/sql-database-design-first-database/create-database-server.png)

5. 按一下 [選取] 。

6. 按一下 [定價層] 可指定服務層、DTU 或 vCore 的數目，以及儲存體數量。 瀏覽 DTU/vCore 數目的選項，以及可供您每個服務層使用的儲存體。 為了進行此教學課程，我們會使用[以 DTU 為基礎的購買模型](sql-database-service-tiers-dtu.md)，但您也可以選擇[以虛擬核心為基礎的購買模型](sql-database-service-tiers-vcore.md)。 

7. 在此教學課程中，選取 [標準] 服務層，然後使用滑桿選取 **100 DTU (S3)** 和 **400** GB 的儲存體。

   ![建立資料庫-s1](./media/sql-database-design-first-database/create-empty-database-pricing-tier.png)

8. 若要使用 [附加元件儲存體] 選項，請接受預覽條款。 

   > [!IMPORTANT]
   > 所有區域目前均可取得進階層中超過 1 TB 的儲存體，下列區域除外：英國北部、美國中西部、英國南部2、中國東部、USDoDCentral、德國中部、USDoDEast、US Gov (西南部)、US Gov (中南部)、德國東北部、中國北部、US Gov (東部)。 在其他區域，進階層中的儲存空間上限為 1 TB。 請參閱 [P11-P15 目前限制]( sql-database-dtu-resource-limits-single-databases.md#single-database-limitations-of-p11-and-p15-when-the-maximum-size-greater-than-1-tb)。

9. 在選取伺服器層、DTU 數目和儲存體數量之後，按一下 [套用]。  

10. 為空白資料庫選取**定序** (此教學課程中使用預設值)。 如需定序的詳細資訊，請參閱[定序](https://docs.microsoft.com/sql/t-sql/statements/collations)。

11. 您現在已完成 SQL Database 表單，請按一下 [建立] 來佈建資料庫。 佈建需要幾分鐘的時間。 

12. 在工具列上，按一下 [通知] 以監視部署程序。
    
     ![通知](./media/sql-database-get-started-portal/notification.png)

## <a name="create-a-server-level-firewall-rule"></a>建立伺服器層級防火牆規則

SQL Database 服務會在伺服器層級建立防火牆，防止外部應用程式和工具連線到伺服器或伺服器上的任何資料庫，除非建立防火牆規則以針對特定的 IP 位址開啟防火牆。 請遵循下列步驟來為您用戶端的 IP 位址建立 [SQL Database 伺服器層級防火牆規則](sql-database-firewall-configure.md)，並讓外部連線僅能夠穿過您 IP 位址的 SQL Database 防火牆。 

> [!NOTE]
> SQL Database 會透過連接埠 1433 通訊。 如果您嘗試從公司網路進行連線，您網路的防火牆可能不允許透過連接埠 1433 的輸出流量。 若情況如此，除非 IT 部門開啟連接埠 1433，否則您無法連線至 Azure SQL Database 伺服器。
>

1. 部署完成之後，按一下左側功能表中的 [SQL Database]，然後按一下 [SQL Database] 頁面上的 [mySampleDatabase]。 資料庫的概觀頁面隨即開啟，其中會顯示完整伺服器名稱 (例如 **mynewserver-20170824.database.windows.net**)，並提供進一步的組態選項。 

2. 在後續的教學課程和快速入門中，請複製此完整伺服器名稱，以用來連線至您的伺服器及其資料庫。 

   ![伺服器名稱](./media/sql-database-get-started-portal/server-name.png) 

3. 在工具列上按一下 [設定伺服器防火牆]。 SQL Database 伺服器的 [防火牆設定] 頁面隨即開啟。 

   ![伺服器防火牆規則](./media/sql-database-get-started-portal/server-firewall-rule.png) 

4. 按一下工具列上的 [新增用戶端 IP]，將目前的 IP 位址新增至新的防火牆規則。 防火牆規則可以針對單一 IP 位址或 IP 位址範圍開啟連接埠 1433。

5. 按一下 [檔案] 。 系統便會為目前的 IP 位址建立伺服器層級防火牆規則，以便在邏輯伺服器上開啟連接埠 1433。

6. 依序按一下 [確定]，然後關閉 [防火牆設定] 頁面。

您現在可以利用 SQL Server Management Studio 或選擇的其他工具，使用先前建立的伺服器管理帳戶從這個 IP 位址連線至 SQL Database 伺服器及其資料庫。

> [!IMPORTANT]
> 根據預設，已對所有 Azure 服務啟用透過 SQL Database 防火牆存取。 按一下此頁面上的 [關閉] 即可對所有 Azure 服務停用。

## <a name="sql-server-connection-information"></a>SQL Server 連線資訊

在 Azure 入口網站中取得 Azure SQL Database 伺服器的完整伺服器名稱。 透過 SQL Server Management Studio，您可使用此完整伺服器名稱連接到您的伺服器。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
2. 從左側功能表中選取 [SQL Database]，按一下 [SQL Database] 頁面上您的資料庫。 
3. 在 Azure 入口網站中您資料庫的 [基本資訊] 窗格中，找到後複製 [伺服器名稱]。

   ![連線資訊](./media/sql-database-get-started-portal/server-name.png)

## <a name="connect-to-the-database-with-ssms"></a>使用 SSMS 連接到資料庫

使用 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/sql-server-management-studio-ssms) 建立對 Azure SQL Database 伺服器的連線。

1. 開啟 SQL Server Management Studio。

2. 在 [連接到伺服器] 對話方塊中，輸入下列資訊：

   | 設定       | 建議的值 | 說明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | 伺服器類型 | 資料庫引擎 | 這是必要值 |
   | 伺服器名稱 | 完整伺服器名稱 | 名稱應該類似這樣︰**mynewserver20170824.database.windows.net**。 |
   | 驗證 | SQL Server 驗證 | 在本教學課程中，我們只設定了 SQL 驗證這個驗證類型。 |
   | 登入 | 伺服器管理帳戶 | 這是您在建立伺服器時所指定的帳戶。 |
   | 密碼 | 伺服器管理帳戶的密碼 | 這是您在建立伺服器時所指定的密碼。 |

   ![連接到伺服器](./media/sql-database-connect-query-ssms/connect.png)

3. 按一下 [連接到伺服器] 對話方塊中的 [選項]。 在 [連線到資料庫] 區段中，輸入 **mySampleDatabase** 以連線到這個資料庫。

   ![連線到伺服器上的 DB](./media/sql-database-connect-query-ssms/options-connect-to-db.png)  

4. 按一下 [ **連接**]。 [物件總管] 視窗隨即在 SSMS 中開啟。 

5. 在 [物件總管] 中，展開 [資料庫]，然後展開 [mySampleDatabase] 以檢視範例資料庫中的物件。

   ![資料庫物件](./media/sql-database-connect-query-ssms/connected.png)  

## <a name="create-tables-in-the-database"></a>在資料庫中建立資料表 

使用四個資料表建立資料庫結構描述，其會使用 [Transact-SQL](https://docs.microsoft.com/sql/t-sql/language-reference) 建立大學的學生管理系統模型：

- Person
- 課程
- 學生
- 建立大學專屬學生管理系統的信用額度

下圖顯示這些資料表是如何彼此相互關聯。 在這當中有部分資料表會參考其他資料表的資料欄。 例如，[Student (學生)] 資料表會參考 [Person (人員)] 資料表的 [PersonId] 資料欄。 研究圖表，以了解在本教學課程中資料表彼此關連的方式。 如需深入了解建立有效資料庫資料表的方式，請參閱[建立有效資料庫資料表 ](https://msdn.microsoft.com/library/cc505842.aspx)。 如需選擇資料類型的相關資訊，請參閱[資料類型 (英文)](https://docs.microsoft.com/sql/t-sql/data-types/data-types-transact-sql)。

> [!NOTE]
> 您亦可使用 [SQL Server Management Studio 中的資料表設計工具](https://docs.microsoft.com/sql/ssms/visual-db-tools/design-database-diagrams-visual-database-tools)，建立和設計您的資料表。 

![資料表關聯性](./media/sql-database-design-first-database/tutorial-database-tables.png)

1. 在 [物件總管] 中，於 **mySampleDatabase** 上按一下滑鼠右鍵，然後按一下 [新增查詢]。 隨即開啟已連線到您資料庫的空白查詢視窗。

2. 在查詢視窗中，執行下列查詢以在資料庫中建立四個資料表︰ 

   ```sql 
   -- Create Person table

   CREATE TABLE Person
   (
   PersonId   INT IDENTITY PRIMARY KEY,
   FirstName   NVARCHAR(128) NOT NULL,
   MiddelInitial NVARCHAR(10),
   LastName   NVARCHAR(128) NOT NULL,
   DateOfBirth   DATE NOT NULL
   )
   
   -- Create Student table
 
   CREATE TABLE Student
   (
   StudentId INT IDENTITY PRIMARY KEY,
   PersonId  INT REFERENCES Person (PersonId),
   Email   NVARCHAR(256)
   )
   
   -- Create Course table
 
   CREATE TABLE Course
   (
   CourseId  INT IDENTITY PRIMARY KEY,
   Name   NVARCHAR(50) NOT NULL,
   Teacher   NVARCHAR(256) NOT NULL
   ) 

   -- Create Credit table
 
   CREATE TABLE Credit
   (
   StudentId   INT REFERENCES Student (StudentId),
   CourseId   INT REFERENCES Course (CourseId),
   Grade   DECIMAL(5,2) CHECK (Grade <= 100.00),
   Attempt   TINYINT,
   CONSTRAINT  [UQ_studentgrades] UNIQUE CLUSTERED
   (
   StudentId, CourseId, Grade, Attempt
   )
   )
   ```

   ![建立資料表](./media/sql-database-design-first-database/create-tables.png)

3. 在 SQL Server Management Studio 物件總管中展開「資料表」節點，以查看您建立的資料表。

   ![建立 ssms 資料表](./media/sql-database-design-first-database/ssms-tables-created.png)

## <a name="load-data-into-the-tables"></a>將資料載入到資料表

1. 在 [下載] 資料夾中建立一個名為 **SampleTableData** 的資料夾，以存放您資料庫的範例資料。 

2. 以滑鼠右鍵按一下下列連結，將其儲存至 **SampleTableData** 資料夾。 

   - [SampleCourseData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCourseData)
   - [SamplePersonData](https://sqldbtutorial.blob.core.windows.net/tutorials/SamplePersonData)
   - [SampleStudentData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleStudentData)
   - [SampleCreditData](https://sqldbtutorial.blob.core.windows.net/tutorials/SampleCreditData)

3. 開啟命令提示字元視窗，並瀏覽至 SampleTableData 資料夾。

4. 執行下列命令以將範例資料列插入至資料表中，其中以適用於您環境的值取代 **ServerName**、**DatabaseName**、**UserName** 及 **Password** 的值。
  
   ```bcp
   bcp Course in SampleCourseData -S <ServerName>.database.windows.net -d <DatabaseName> -U <Username> -P <password> -q -c -t ","
   bcp Person in SamplePersonData -S <ServerName>.database.windows.net -d <DatabaseName> -U <Username> -P <password> -q -c -t ","
   bcp Student in SampleStudentData -S <ServerName>.database.windows.net -d <DatabaseName> -U <Username> -P <password> -q -c -t ","
   bcp Credit in SampleCreditData -S <ServerName>.database.windows.net -d <DatabaseName> -U <Username> -P <password> -q -c -t ","
   ```

您現在已將範例資料載入至先前建立的資料表。

## <a name="query-data"></a>查詢資料

執行下列查詢，以從資料庫資料表中擷取資訊。 如需深入了解關於撰寫 SQL 查詢的資訊，請參閱[撰寫 SQL 查詢 (英文)](https://technet.microsoft.com/library/bb264565.aspx)。 第一個查詢會聯結所有四個資料表，以尋找由「Dominick Pope」授課且成績高於全班 75% 學生的所有學生。 第二個查詢會連結所有四個資料表，以尋找「Noe Coleman」曾註冊的所有課程。

1. 在 [SQL Server Management Studio] 查詢視窗中，執行下列查詢︰

   ```sql 
   -- Find the students taught by Dominick Pope who have a grade higher than 75%

   SELECT  person.FirstName,
   person.LastName,
   course.Name,
   credit.Grade
   FROM  Person AS person
   INNER JOIN Student AS student ON person.PersonId = student.PersonId
   INNER JOIN Credit AS credit ON student.StudentId = credit.StudentId
   INNER JOIN Course AS course ON credit.CourseId = course.courseId
   WHERE course.Teacher = 'Dominick Pope' 
   AND Grade > 75
   ```

2. 在 [SQL Server Management Studio] 查詢視窗中，執行下列查詢︰

   ```sql
   -- Find all the courses in which Noe Coleman has ever enrolled

   SELECT  course.Name,
   course.Teacher,
   credit.Grade
   FROM  Course AS course
   INNER JOIN Credit AS credit ON credit.CourseId = course.CourseId
   INNER JOIN Student AS student ON student.StudentId = credit.StudentId
   INNER JOIN Person AS person ON person.PersonId = student.PersonId
   WHERE person.FirstName = 'Noe'
   AND person.LastName = 'Coleman'
   ```

## <a name="next-steps"></a>後續步驟 
在本教學課程中，您已了解基本的資料庫工作，例如建立資料庫和資料表、載入和查詢資料，以及將資料庫還原至先前的時間點。 您已了解如何︰
> [!div class="checklist"]
> * 建立資料庫
> * 設定防火牆規則
> * 使用 [SQL Server Management Studio](https://msdn.microsoft.com/library/ms174173.aspx) (SSMS) 來連線到資料庫
> * 建立資料表
> * 大量載入資料
> * 查詢該資料

前進到下一個教學課程，了解如何使用 Visual Studio 和 C# 設計資料庫。

> [!div class="nextstepaction"]
>[設計 Azure SQL Database 並連接 C# 和 ADO.NET](sql-database-design-first-database-csharp.md)
