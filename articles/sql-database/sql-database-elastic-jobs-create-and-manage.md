---
title: 管理 Azure SQL Database 的群組 | Microsoft Docs
description: 逐步解說如何建立和管理彈性工作。
services: sql-database
ms.service: sql-database
subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
manager: craigg
ms.date: 07/16/2018
ms.openlocfilehash: dca4ae2c8aa75b7af40ed4f8430968b8477c6802
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47165665"
---
# <a name="create-and-manage-scaled-out-azure-sql-databases-using-elastic-jobs-preview"></a>使用彈性工作建立和管理相應放大的 Azure SQL 資料庫 (預覽)


[!INCLUDE [elastic-database-jobs-deprecation](../../includes/sql-database-elastic-jobs-deprecate.md)]


**彈性資料庫工作** 可以簡化資料庫群組的管理，方法是執行系統管理作業 (例如結構描述變更、認證管理、參考資料更新、效能資料收集，或租用戶 (客戶) 遙測收集)。 彈性資料庫工作目前可透過 Azure 入口網站和 PowerShell Cmdlet 使用。 不過，Azure 入口網站呈現精簡功能會限制為跨[彈性集區](sql-database-elastic-pool.md) 中的所有資料庫執行。 若要存取其他功能以及跨資料庫群組執行指令碼，包括自訂定義集合或分區集 (使用[彈性資料庫用戶端程式庫](sql-database-elastic-scale-introduction.md)建立)，請參閱[使用 PowerShell 建立和管理作業](sql-database-elastic-jobs-powershell.md)。 如需工作的詳細資訊，請參閱 [彈性資料庫工作概觀](sql-database-elastic-jobs-overview.md)。 

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶。 如需免費試用版，請參閱 [免費試用版](https://azure.microsoft.com/pricing/free-trial/)。
* 彈性集區。 請參閱[關於彈性集區](sql-database-elastic-pool.md)。
* 安裝彈性資料庫工作服務元件。 請參閱 [安裝彈性資料庫工作服務](sql-database-elastic-jobs-service-installation.md)。

## <a name="creating-jobs"></a>建立工作 (Job)
1. 使用 [Azure 入口網站](https://portal.azure.com)，從現有的彈性資料庫工作集區，按一下 [建立作業]。
2. 輸入工作控制資料庫 (工作的中繼資料儲存體) 之資料庫系統管理員 (在安裝工作時建立) 的使用者名稱與密碼。
   
    ![為工作命名，輸入或貼上程式碼，然後按一下 [執行]][1]
3. 在 [建立工作]  刀鋒視窗中，輸入工作的名稱。
4. 輸入使用者名稱與密碼來連線至目標資料庫，以取得足夠權限來成功執行指令碼。
5. 貼上或輸入 T-SQL 指令碼。
6. 按一下 [儲存]，然後按一下 [執行]。
   
    ![建立工作並執行][5]

## <a name="run-idempotent-jobs"></a>執行等冪工作
當您對一組資料庫執行指令碼時，您必須確定指令碼是等冪。 換句話說，指令碼必須能夠重複執行，即使之前在未完成狀態失敗亦然。 例如，當指令碼失敗時，系統會自動重試工作，直到成功為止 (在限制內，因為重試邏輯最終將會停止重試)。 其作法是使用 "IF EXISTS" 子句，並刪除任何找到的執行個體，再建立新的物件。 範例如下所示：

    IF EXISTS (SELECT name FROM sys.indexes
            WHERE name = N'IX_ProductVendor_VendorID')
    DROP INDEX IX_ProductVendor_VendorID ON Purchasing.ProductVendor;
    GO
    CREATE INDEX IX_ProductVendor_VendorID
    ON Purchasing.ProductVendor (VendorID);

或者，使用 "IF NOT EXISTS" 子句，再建立新的執行個體：

    IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
    BEGIN
     CREATE TABLE TestTable(
      TestTableId INT PRIMARY KEY IDENTITY,
      InsertionTime DATETIME2
     );
    END
    GO

    INSERT INTO TestTable(InsertionTime) VALUES (sysutcdatetime());
    GO

這個指令碼會接著更新之前建立的資料表。

    IF NOT EXISTS (SELECT columns.name FROM sys.columns INNER JOIN sys.tables on columns.object_id = tables.object_id WHERE tables.name = 'TestTable' AND columns.name = 'AdditionalInformation')
    BEGIN

    ALTER TABLE TestTable

    ADD AdditionalInformation NVARCHAR(400);
    END
    GO

    INSERT INTO TestTable(InsertionTime, AdditionalInformation) VALUES (sysutcdatetime(), 'test');
    GO


## <a name="checking-job-status"></a>檢查工作狀態
工作開始之後，您可以檢查它的進度。

1. 從 [彈性集區] 頁面，按一下 [管理工作]。
   
    ![按一下 [管理工作]][2]
2. 按一下工作的名稱 (a)。 [狀態]  可以是 [已完成] 或 [失敗]。 工作的詳細資料隨即出現 (b)，並提供建立和執行的日期和時間。 其下方的清單 (c) 顯示對集區中每個資料庫執行指令碼的進度，並提供其日期和時間詳細資料。
   
    ![檢查完成的工作][3]

## <a name="checking-failed-jobs"></a>檢查失敗的工作
如果工作失敗，您可以找到其執行記錄檔。 按一下失敗工作的名稱，以查看其詳細資料。

![查看失敗的工作][4]

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-create-and-manage/screen-1.png
[2]: ./media/sql-database-elastic-jobs-create-and-manage/click-manage-jobs.png
[3]: ./media/sql-database-elastic-jobs-create-and-manage/running-jobs.png
[4]: ./media/sql-database-elastic-jobs-create-and-manage/failed.png
[5]: ./media/sql-database-elastic-jobs-create-and-manage/screen-2.png


