---
title: SQL Database 的 XEvent 事件檔案程式碼 | Microsoft Docs
description: 提供 PowerShell 和 Transact-SQL 的兩階段程式碼範例，示範 Azure SQL Database 上擴充事件中的事件檔案目標。 此案例必須要有 Azure 儲存體。
services: sql-database
author: MightyPen
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: genemi
ms.openlocfilehash: b905c921ae967d7f755f084bd6b9b30de34f76b4
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34649630"
---
# <a name="event-file-target-code-for-extended-events-in-sql-database"></a>SQL Database 中擴充事件的事件檔案目標程式碼

[!INCLUDE [sql-database-xevents-selectors-1-include](../../includes/sql-database-xevents-selectors-1-include.md)]

您想要完整的程式碼範例以穩健方式擷取和報告擴充事件的資訊。

在 Microsoft SQL Server 中， [事件檔案目標](http://msdn.microsoft.com/library/ff878115.aspx) 是用來將事件輸出儲存到本機硬碟機檔案中。 但是這類檔案並不適用於 Azure SQL Database。 我們改為使用 Azure 儲存體服務來支援事件檔案目標。

本主題示範一個兩階段的程式碼範例：

* 使用 PowerShell 在雲端中建立 Azure 儲存體容器
* Transact-SQL：
  
  * 將 Azure 儲存體容器指定為事件檔案目標。
  * 建立和啟動事件工作階段等等。

## <a name="prerequisites"></a>先決條件

* Azure 帳戶和訂用帳戶。 您可以註冊 [免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 您可以在當中建立資料表的任何資料庫。
  
  * 您可以選擇性快速[建立 **AdventureWorksLT** 示範資料庫](sql-database-get-started.md)。
* SQL Server Management Studio (ssms.exe)，最好是最新的每月更新版本。 
  您可以從下列位置下載最新的 ssms.exe：
  
  * 名稱為 [下載 SQL Server Management Studio](http://msdn.microsoft.com/library/mt238290.aspx)的主題。
  * [下載的直接連結。](http://go.microsoft.com/fwlink/?linkid=616025)
* 您必須安裝 [Azure PowerShell 模組](http://go.microsoft.com/?linkid=9811175) 。
  
  * 模組提供 **New-AzureStorageAccount**這類的命令。

## <a name="phase-1-powershell-code-for-azure-storage-container"></a>第 1 階段：Azure 儲存體容器的 PowerShell 程式碼

這個 PowerShell 是兩階段程式碼範例的第 1 階段。

此指令碼是以可清除先前可能之執行結果的命令為開頭，並且可重複執行。

1. 將 PowerShell 指令碼貼到如 Notepad.exe 的簡單文字編輯器，並將指令碼儲存為有 **.ps1**副檔名的檔案。
2. 以系統管理員身分啟動 PowerShell ISE。
3. 在提示中輸入<br/>`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`<br/>然後按 Enter 鍵。
4. 在 PowerShell ISE 中開啟您的 **.ps1** 檔案。 執行指令碼。
5. 指令碼會先啟動新的視窗讓您登入 Azure。
   
   * 如果您重複執行指令碼而不中斷您的工作階段，可以很方便地選擇將 **Add-AzureAccount** 命令標記為註解。

![準備好 PowerShell ISE 和安裝的 Azure 模組，以便執行指令碼。][30_powershell_ise]

### <a name="powershell-code"></a>PowerShell 程式碼

這個 PowerShell 指令碼假設您已經執行 AzureRm 模組的 Cmdlet Import-Module。 如需參考文件，請參閱 [PowerShell 模組瀏覽器](https://docs.microsoft.com/powershell/module/)。

```powershell
## TODO: Before running, find all 'TODO' and make each edit!!

cls;

#--------------- 1 -----------------------

'Script assumes you have already logged your PowerShell session into Azure.
But if not, run  Connect-AzureRmAccount (or  Connect-AzureRmAccount), just one time.';
#Connect-AzureRmAccount;   # Same as  Connect-AzureRmAccount.

#-------------- 2 ------------------------

'
TODO: Edit the values assigned to these variables, especially the first few!
';

# Ensure the current date is between
# the Expiry and Start time values that you edit here.

$subscriptionName    = 'YOUR_SUBSCRIPTION_NAME';
$resourceGroupName   = 'YOUR_RESOURCE-GROUP-NAME';

$policySasExpiryTime = '2018-08-28T23:44:56Z';
$policySasStartTime  = '2017-10-01';

$storageAccountLocation = 'West US';
$storageAccountName     = 'YOUR_STORAGE_ACCOUNT_NAME';
$contextName            = 'YOUR_CONTEXT_NAME';
$containerName          = 'YOUR_CONTAINER_NAME';
$policySasToken         = ' ? ';

$policySasPermission = 'rwl';  # Leave this value alone, as 'rwl'.

#--------------- 3 -----------------------

# The ending display lists your Azure subscriptions.
# One should match the $subscriptionName value you assigned
#   earlier in this PowerShell script. 

'Choose an existing subscription for the current PowerShell environment.';

Select-AzureRmSubscription -Subscription $subscriptionName;

#-------------- 4 ------------------------

'
Clean up the old Azure Storage Account after any previous run, 
before continuing this new run.';

If ($storageAccountName)
{
    Remove-AzureRmStorageAccount `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName;
}

#--------------- 5 -----------------------

[System.DateTime]::Now.ToString();

'
Create a storage account. 
This might take several minutes, will beep when ready.
  ...PLEASE WAIT...';

New-AzureRmStorageAccount `
    -Name              $storageAccountName `
    -Location          $storageAccountLocation `
    -ResourceGroupName $resourceGroupName `
    -SkuName           'Standard_LRS';

[System.DateTime]::Now.ToString();
[System.Media.SystemSounds]::Beep.Play();

'
Get the access key for your storage account.
';

$accessKey_ForStorageAccount = `
    (Get-AzureRmStorageAccountKey `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName
        ).Value[0];

"`$accessKey_ForStorageAccount = $accessKey_ForStorageAccount";

'Azure Storage Account cmdlet completed.
Remainder of PowerShell .ps1 script continues.
';

#--------------- 6 -----------------------

# The context will be needed to create a container within the storage account.

'Create a context object from the storage account and its primary access key.
';

$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey  $accessKey_ForStorageAccount;

'Create a container within the storage account.
';

$containerObjectInStorageAccount = New-AzureStorageContainer `
    -Name    $containerName `
    -Context $context;

'Create a security policy to be applied to the SAS token.
';

New-AzureStorageContainerStoredAccessPolicy `
    -Container  $containerName `
    -Context    $context `
    -Policy     $policySasToken `
    -Permission $policySasPermission `
    -ExpiryTime $policySasExpiryTime `
    -StartTime  $policySasStartTime;

'
Generate a SAS token for the container.
';
Try
{
    $sasTokenWithPolicy = New-AzureStorageContainerSASToken `
        -Name    $containerName `
        -Context $context `
        -Policy  $policySasToken;
}
Catch 
{
    $Error[0].Exception.ToString();
}

#-------------- 7 ------------------------

'Display the values that YOU must edit into the Transact-SQL script next!:
';

"storageAccountName: $storageAccountName";
"containerName:      $containerName";
"sasTokenWithPolicy: $sasTokenWithPolicy";

'
REMINDER: sasTokenWithPolicy here might start with "?" character, which you must exclude from Transact-SQL.
';

'
(Later, return here to delete your Azure Storage account. See the preceding  Remove-AzureRmStorageAccount -Name $storageAccountName)';

'
Now shift to the Transact-SQL portion of the two-part code sample!';

# EOFile
```


記下 PowerShell 指令碼結束時列印出的幾個具名值。 您必須將這些值寫入第 2 階段的 Transact-SQL 指令碼。

## <a name="phase-2-transact-sql-code-that-uses-azure-storage-container"></a>第 2 階段：使用 Azure 儲存體容器的 Transact-SQL 程式碼

* 在此程式碼範例的第 1 階段中，您執行了 PowerShell 指令碼來建立「Azure 儲存體」容器。
* 接下來在第 2 階段中，下列 Transact-SQL 指令碼必須使用該容器。

此指令碼是以可清除先前可能之執行結果的命令為開頭，並且可重複執行。

PowerShell 指令碼在結束時列印出幾個具名的值。 您必須編輯 Transact-SQL 指令碼以使用這些值。 在 Transact-SQL 指令碼中尋找 **TODO** 找出編輯點。

1. 開啟 SQL Server Management Studio (ssms.exe)。
2. 連接到您的 Azure SQL Database 資料庫。
3. 按一下以開啟新的查詢窗格。
4. 將下列 Transact-SQL 指令碼貼入查詢窗格。
5. 在指令碼中尋找每個 **TODO** 並進行適當的編輯。
6. 儲存並執行指令碼。


> [!WARNING]
> 上述 PowerShell 指令碼所產生的 SAS 金鑰值可能會以 '?' (問號) 開頭。 當您在下列 T-SQL 指令碼中使用 SAS 金鑰時，您必須「移除前置 '?'」 。 否則您的動作可能會遭到安全性封鎖。


### <a name="transact-sql-code"></a>Transact-SQL 程式碼

```sql
---- TODO: First, run the earlier PowerShell portion of this two-part code sample.
---- TODO: Second, find every 'TODO' in this Transact-SQL file, and edit each.

---- Transact-SQL code for Event File target on Azure SQL Database.


SET NOCOUNT ON;
GO


----  Step 1.  Establish one little table, and  ---------
----  insert one row of data.


IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'gmTabEmployee')
BEGIN
    DROP TABLE gmTabEmployee;
END
GO


CREATE TABLE gmTabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO gmTabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO


------  Step 2.  Create key, and  ------------
------  Create credential (your Azure Storage container must already exist).


IF NOT EXISTS
    (SELECT * FROM sys.symmetric_keys
        WHERE symmetric_key_id = 101)
BEGIN
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '0C34C960-6621-4682-A123-C7EA08E3FC46' -- Or any newid().
END
GO


IF EXISTS
    (SELECT * FROM sys.database_scoped_credentials
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        WHERE name = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent')
BEGIN
    DROP DATABASE SCOPED CREDENTIAL
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent] ;
END
GO


CREATE
    DATABASE SCOPED
    CREDENTIAL
        -- use '.blob.',   and not '.queue.' or '.table.' etc.
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    WITH
        IDENTITY = 'SHARED ACCESS SIGNATURE',  -- "SAS" token.
        -- TODO: Paste in the long SasToken string here for Secret, but exclude any leading '?'.
        SECRET = 'sv=2014-02-14&sr=c&si=gmpolicysastoken&sig=EjAqjo6Nu5xMLEZEkMkLbeF7TD9v1J8DNB2t8gOKTts%3D'
    ;
GO


------  Step 3.  Create (define) an event session.  --------
------  The event session has an event with an action,
------  and a has a target.

IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'gmeventsessionname240b')
BEGIN
    DROP
        EVENT SESSION
            gmeventsessionname240b
        ON DATABASE;
END
GO


CREATE
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE

    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE 'UPDATE gmTabEmployee%'
            )
    ADD TARGET
        package0.event_file
            (
            -- TODO: Assign AzureStorageAccount name, and the associated Container name.
            -- Also, tweak the .xel file name at end, if you like.
            SET filename =
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
            )
    WITH
        (MAX_MEMORY = 10 MB,
        MAX_DISPATCH_LATENCY = 3 SECONDS)
    ;
GO


------  Step 4.  Start the event session.  ----------------
------  Issue the SQL Update statements that will be traced.
------  Then stop the session.

------  Note: If the target fails to attach,
------  the session must be stopped and restarted.

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = START;
GO


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
GO


ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = STOP;
GO


-------------- Step 5.  Select the results. ----------

SELECT
        *, 'CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS!' as [CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS],
        CAST(event_data AS XML) AS [event_data_XML]  -- TODO: In ssms.exe results grid, double-click this cell!
    FROM
        sys.fn_xe_file_target_read_file
            (
                -- TODO: Fill in Storage Account name, and the associated Container name.
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b',
                null, null, null
            );
GO


-------------- Step 6.  Clean up. ----------

DROP
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE;
GO

DROP DATABASE SCOPED CREDENTIAL
    -- TODO: Assign AzureStorageAccount name, and the associated Container name.
    [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    ;
GO

DROP TABLE gmTabEmployee;
GO

PRINT 'Use PowerShell Remove-AzureStorageAccount to delete your Azure Storage account!';
GO
```


如果目標在執行時無法附加，您就必須停止事件工作階段並重新啟動：

```sql
ALTER EVENT SESSION ... STATE = STOP;
GO
ALTER EVENT SESSION ... STATE = START;
GO
```


## <a name="output"></a>輸出

Transact-SQL 指令碼完成時，按一下 **event_data_XML** 資料欄標題下的儲存格。 此時會顯示一個 **<event>** 元素，此元素會顯示一個 UPDATE 陳述式。

以下是測試期間所產生的一個 **<event>** 元素：


```xml
<event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T19:18:45.420Z">
  <data name="state">
    <value>0</value>
    <text>Normal</text>
  </data>
  <data name="line_number">
    <value>5</value>
  </data>
  <data name="offset">
    <value>148</value>
  </data>
  <data name="offset_end">
    <value>368</value>
  </data>
  <data name="statement">
    <value>UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe'</value>
  </data>
  <action name="sql_text" package="sqlserver">
    <value>

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
</value>
  </action>
</event>
```


上述 TRANSACT-SQL 指令碼使用下列系統函數讀取 event_file：

* [sys.fn_xe_file_target_read_file (Transact-SQL)](http://msdn.microsoft.com/library/cc280743.aspx)

您可以在下列文章中取得進階選項的說明，這些選項可用來檢視擴充事件的資料：

* [進一步檢視擴充事件的目標資料](http://msdn.microsoft.com/library/mt752502.aspx)


## <a name="converting-the-code-sample-to-run-on-sql-server"></a>轉換程式碼範例在 SQL Server 上執行

假設您想要在 Microsoft SQL Server 上執行上述的 Transact-SQL 範例。

* 為了簡單起見，您可以用簡單的檔案 (例如 **C:\myeventdata.xel**) 來取代「Azure 儲存體」容器的使用。 檔案會寫入裝載 SQL Server 之電腦的本機硬碟。
* 針對 **CREATE MASTER KEY** 和**CREATE CREDENTIAL**，您不需要任何類型的 Transact-SQL 陳述式。
* 在 **CREATE EVENT SESSION** 陳述式的 **ADD TARGET** 子句中，您要將對 **filename=** 指派的 Http 值取代為完整路徑字串 (例如 **C:\myfile.xel**)。
  
  * 不需要牽涉到任何 Azure 儲存體帳戶。

## <a name="more-information"></a>詳細資訊

如需 Azure 儲存體服務中帳戶和容器的詳細資訊，請參閱：

* [如何使用 .NET 的 Blob 儲存體](../storage/blobs/storage-dotnet-how-to-use-blobs.md)
* [命名和參考容器、Blob 及中繼資料](http://msdn.microsoft.com/library/azure/dd135715.aspx)
* [使用根容器](http://msdn.microsoft.com/library/azure/ee395424.aspx)
* [第 1 課：在 Azure 容器上建立預存的存取原則和共用存取簽章](http://msdn.microsoft.com/library/dn466430.aspx)
  * [第 2 課：使用共用存取簽章建立 SQL Server 認證](http://msdn.microsoft.com/library/dn466435.aspx)
* [Microsoft SQL Server 的擴充事件](https://docs.microsoft.com/sql/relational-databases/extended-events/extended-events)

<!--
Image references.
-->

[30_powershell_ise]: ./media/sql-database-xevent-code-event-file/event-file-powershell-ise-b30.png

