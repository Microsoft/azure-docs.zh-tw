---
title: 使用 PowerShell 建立和管理彈性作業 | Microsoft Docs
description: 用來管理 Azure SQL Database 集區的 PowerShell
services: sql-database
manager: craigg
author: stevestein
ms.service: sql-database
ms.custom: scale out apps
ms.topic: conceptual
ms.date: 06/14/2018
ms.author: sstein
ms.openlocfilehash: 3efee2f9a4b205b87a65eb473271b0ab5e58046d
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498414"
---
# <a name="create-and-manage-sql-database-elastic-jobs-using-powershell-preview"></a>使用 PowerShell 建立和管理 SQL Database 彈性作業 (預覽)


[!INCLUDE [elastic-database-jobs-deprecation](../../includes/sql-database-elastic-jobs-deprecate.md)]


適用於 **彈性資料庫工作** (預覽版) 的 PowerShell API 可讓您定義一組資料庫，然後針對這組資料庫執行指令碼。 本文將說明如何使用 Powershell Cmdlet 建立和管理 **彈性資料庫工作** 。 請參閱 [彈性工作概觀](sql-database-elastic-jobs-overview.md)。 

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶。 如需免費試用，請參閱 [免費試用一個月](https://azure.microsoft.com/pricing/free-trial/)。
* 一組使用彈性資料庫工具所建立的資料庫。 請參閱 [開始使用彈性資料庫工具](sql-database-elastic-scale-get-started.md)。
* Azure PowerShell。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)。
* **彈性資料庫工作** PowerShell 封裝：請參閱 [安裝彈性資料庫工作](sql-database-elastic-jobs-service-installation.md)

### <a name="select-your-azure-subscription"></a>選取您的 Azure 訂用帳戶
若要選取所需的訂用帳戶，您必須提供訂用帳戶 ID (**-SubscriptionId**) 或訂用帳戶名稱 (**-SubscriptionName**)。 如果您有多個訂用帳戶，則可以執行 **Get-AzureRmSubscription** Cmdlet，然後從結果集複製所需的訂用帳戶資訊。 一旦您具有訂用帳戶資訊，請執行下列 commandlet 將此訂用帳戶設定為預設值，也就是建立和管理工作的目標：

    Select-AzureRmSubscription -SubscriptionId {SubscriptionID}

建議使用 [PowerShell ISE](https://technet.microsoft.com/library/dd315244.aspx) 以針對彈性資料庫工作開發和執行 PowerShell 指令碼。

## <a name="elastic-database-jobs-objects"></a>彈性資料庫工作物件
下表列出 **彈性資料庫工作** 的所有物件類型，以及其描述和相關 PowerShell API。

<table style="width:100%">
  <tr>
    <th>物件類型</th>
    <th>說明</th>
    <th>相關 PowerShell API</th>
  </tr>
  <tr>
    <td>認證</td>
    <td>連接到資料庫以執行指令碼或 DACPAC 的應用程式時使用的使用者名稱和密碼。 <p>密碼在傳送並儲存在彈性資料庫工作資料庫之前會先行加密。  密碼會由彈性資料庫工作服務透過安裝指令碼建立及上傳的認證解密。</td>
    <td><p>Get-AzureSqlJobCredential</p>
    <p>New-AzureSqlJobCredential</p><p>Set-AzureSqlJobCredential</p></td></td>
  </tr>

  <tr>
    <td>指令碼</td>
    <td>用於跨資料庫執行的 Transact-SQL 指令碼。  指令碼應該撰寫為等冪，因為服務將會在失敗時重試執行指令碼。
    </td>
    <td>
    <p>Get-AzureSqlJobContent</p>
    <p>Get-AzureSqlJobContentDefinition</p>
    <p>New-AzureSqlJobContent</p>
    <p>Set-AzureSqlJobContentDefinition</p>
    </td>
  </tr>

  <tr>
    <td>DACPAC</td>
    <td>要跨資料庫套用的<a href="https://msdn.microsoft.com/library/ee210546.aspx">資料層應用程式</a>套件。
    </td>
    <td>
    <p>Get-AzureSqlJobContent</p>
    <p>New-AzureSqlJobContent</p>
    <p>Set-AzureSqlJobContentDefinition</p>
    </td>
  </tr>
  <tr>
    <td>資料庫目標</td>
    <td>指向 Azure SQL Database 的資料庫和伺服器名稱。
    </td>
    <td>
    <p>Get-AzureSqlJobTarget</p>
    <p>New-AzureSqlJobTarget</p>
    </td>
  </tr>
  <tr>
    <td>分區對應目標</td>
    <td>資料庫目標和認證的組合，用來判斷彈性資料庫分區對應內儲存的資訊。
    </td>
    <td>
    <p>Get-AzureSqlJobTarget</p>
    <p>New-AzureSqlJobTarget</p>
    <p>Set-AzureSqlJobTarget</p>
    </td>
  </tr>
<tr>
    <td>自訂集合目標</td>
    <td>共同用於執行的已定義資料庫群組。</td>
    <td>
    <p>Get-AzureSqlJobTarget</p>
    <p>New-AzureSqlJobTarget</p>
    </td>
  </tr>
<tr>
    <td>自訂集合子目標</td>
    <td>從自訂集合參考的資料庫目標。</td>
    <td>
    <p>Add-AzureSqlJobChildTarget</p>
    <p>Remove-AzureSqlJobChildTarget</p>
    </td>
  </tr>

<tr>
    <td>工作 (Job)</td>
    <td>
    <p>工作的參數的定義，可用來觸發執行或完成排程。</p>
    </td>
    <td>
    <p>Get-AzureSqlJob</p>
    <p>New-AzureSqlJob</p>
    <p>Set-AzureSqlJob</p>
    </td>
  </tr>

<tr>
    <td>工作執行</td>
    <td>
    <p>必要的作業容器，以使用資料庫連線的認證執行指令碼或將 DACPAC 套用到目標，具有根據執行原則處理的失敗。</p>
    </td>
    <td>
    <p>Get-AzureSqlJobExecution</p>
    <p>Start-AzureSqlJobExecution</p>
    <p>Stop-AzureSqlJobExecution</p>
    <p>Wait-AzureSqlJobExecution</p>
  </tr>

<tr>
    <td>工作作業執行</td>
    <td>
    <p>完成作業的單一工作單位。</p>
    <p>如果工作作業不能成功執行，將會記錄產生的例外狀況訊息，並且建立新的比對工作作業及根據指定的執行原則執行。</p></p>
    </td>
    <td>
    <p>Get-AzureSqlJobExecution</p>
    <p>Start-AzureSqlJobExecution</p>
    <p>Stop-AzureSqlJobExecution</p>
    <p>Wait-AzureSqlJobExecution</p>
  </tr>

<tr>
    <td>工作執行原則</td>
    <td>
    <p>控制重試之間的工作執行逾時、重試限制和間隔。</p>
    <p>彈性資料庫工作包括預設工作執行原則，基本上會導致無限的工作作業失敗重試，每次重試之間具有間隔的指數輪詢。</p>
    </td>
    <td>
    <p>Get-AzureSqlJobExecutionPolicy</p>
    <p>New-AzureSqlJobExecutionPolicy</p>
    <p>Set-AzureSqlJobExecutionPolicy</p>
    </td>
  </tr>

<tr>
    <td>排程</td>
    <td>
    <p>以時間為基礎的執行指定會在重複間隔發生或單一次數發生。</p>
    </td>
    <td>
    <p>Get-AzureSqlJobSchedule</p>
    <p>New-AzureSqlJobSchedule</p>
    <p>Set-AzureSqlJobSchedule</p>
    </td>
  </tr>

<tr>
    <td>工作觸發程序</td>
    <td>
    <p>工作與排程之間的對應，以根據排程觸發工作執行。</p>
    </td>
    <td>
    <p>New-AzureSqlJobTrigger</p>
    <p>Remove-AzureSqlJobTrigger</p>
    </td>
  </tr>
</table>

## <a name="supported-elastic-database-jobs-group-types"></a>支援的彈性資料庫工作群組類型
工作可以跨資料庫群組執行 Transact-SQL (T-SQL) 指令碼或 DACPAC 的應用程式。 將提交工作是跨資料庫群組執行時，工作會「展開」子工作，由每個子工作針對群組中的單一資料庫執行要求的動作。 

您可以建立兩種群組： 

* [分區對應](sql-database-elastic-scale-shard-map-management.md) 群組：當提交工作是以分區對應為目標時，工作會先查詢分區對應來判斷其目前的分區集，然後為分區對應中的每個分區建立子工作。
* 自訂集合群組：一組自訂定義的資料庫。 當工作以自訂集合為目標時，它會為目前在自訂集合中的每個資料庫建立子工作。

## <a name="to-set-the-elastic-database-jobs-connection"></a>設定彈性資料庫工作連接
使用工作 API 之前，必須設定工作「控制資料庫」  的連接。 執行此 Cmdlet 會顯示認證視窗，要求提供安裝彈性資料庫工作時所建立的使用者名稱和密碼。 本主題中提供的所有範例都假設已經執行第一個步驟。

開啟彈性資料庫工作的連線：

    Use-AzureSqlJobConnection -CurrentAzureSubscription 

## <a name="encrypted-credentials-within-the-elastic-database-jobs"></a>彈性資料庫工作內的已加密認證
資料庫認證可以插入至工作的「控制資料庫」，而且密碼會加密。 必須儲存認證，稍後才能執行工作 (使用工作排程)。

加密是透過建立為安裝指令碼一部分的憑證來運作。 安裝指令碼會建立憑證並將其上傳至 Azure 雲端服務，以解密已儲存的加密密碼。 Azure 雲端服務稍後會在工作的「控制資料庫」  內儲存公開金鑰，讓 PowerShell API 或 Azure 入口網站介面加密提供的密碼，而不需要在本機安裝憑證。

認證密碼會加密，以防範對彈性資料庫工作物件只具有唯讀存取權的使用者。 但對於彈性資料庫工作物件具有讀寫存取權的惡意使用者，有可能會擷取密碼。 認證是設計為跨工作執行重複使用。 當建立連線時，認證會傳遞至目標資料庫。 用於每個認證的目標資料庫目前沒有限制，惡意使用者可以為他掌控之下的資料庫加入資料庫目標。 該使用者接著就可以針對此資料庫啟動工作，取得認證的密碼。

彈性資料庫工作的安全性最佳作法包括：

* 將 API 的使用限制為受信任的個人。
* 認證應該具有執行工作作業所需的最低權限。  如需詳細資訊，請參閱 [SQL Server 中的授權和權限](https://msdn.microsoft.com/library/bb669084.aspx) 這篇 MSDN 文章。

### <a name="to-create-an-encrypted-credential-for-job-execution-across-databases"></a>建立跨資料庫執行工作的加密認證
若要建立新的加密認證，[**Get-Credential Cmdlet**](/powershell/module/microsoft.powershell.security/get-credential) 會提示您輸入可以傳遞至 [**New-AzureSqlJobCredential Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljobcredential) 的使用者名稱和密碼。

    $credentialName = "{Credential Name}"
    $databaseCredential = Get-Credential
    $credential = New-AzureSqlJobCredential -Credential $databaseCredential -CredentialName $credentialName
    Write-Output $credential

### <a name="to-update-credentials"></a>更新認證
當密碼變更時，請使用 [**Set-AzureSqlJobCredential Cmdlet**](/powershell/module/elasticdatabasejobs/set-azuresqljobcredential) 並設定 **CredentialName** 參數。

    $credentialName = "{Credential Name}"
    Set-AzureSqlJobCredential -CredentialName $credentialName -Credential $credential 

## <a name="to-define-an-elastic-database-shard-map-target"></a>定義彈性資料庫分區對應目標
若要針對分區集的所有資料庫執行作業 (使用 [彈性資料庫用戶端程式庫](sql-database-elastic-database-client-library.md)建立)，請使用分區對應做為資料庫目標。 此範例需要一個使用彈性資料庫用戶端程式庫建立的分區應用程式。 請參閱 [開始使用彈性資料庫工具範例](sql-database-elastic-scale-get-started.md)。

您必須將分區對應管理員資料庫設為資料庫目標，然後將特定分區對應指定為目標。

    $shardMapCredentialName = "{Credential Name}"
    $shardMapDatabaseName = "{ShardMapDatabaseName}" #example: ElasticScaleStarterKit_ShardMapManagerDb
    $shardMapDatabaseServerName = "{ShardMapServerName}"
    $shardMapName = "{MyShardMap}" #example: CustomerIDShardMap
    $shardMapDatabaseTarget = New-AzureSqlJobTarget -DatabaseName $shardMapDatabaseName -ServerName $shardMapDatabaseServerName
    $shardMapTarget = New-AzureSqlJobTarget -ShardMapManagerCredentialName $shardMapCredentialName -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapDatabaseServerName -ShardMapName $shardMapName
    Write-Output $shardMapTarget

## <a name="create-a-t-sql-script-for-execution-across-databases"></a>針對跨資料庫執行建立 T-SQL 指令碼
建立要執行的 T-SQL 指令碼時，強烈建議將其建置為 [等冪性質](https://en.wikipedia.org/wiki/Idempotence) 且失敗時迅速恢復。 每當執行發生失敗時，不論失敗的分類，彈性資料庫工作將重試執行指令碼。

使用 [**New-AzureSqlJobContent Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljobcontent) 建立並儲存要執行的指令碼，並且設定 **-ContentName** 和 **-CommandText** 參數。

    $scriptName = "Create a TestTable"

    $scriptCommandText = "
    IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
    BEGIN
        CREATE TABLE TestTable(
            TestTableId INT PRIMARY KEY IDENTITY,
            InsertionTime DATETIME2
        );
    END
    GO
    INSERT INTO TestTable(InsertionTime) VALUES (sysutcdatetime());
    GO"

    $script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
    Write-Output $script

### <a name="create-a-new-script-from-a-file"></a>從檔案建立新的指令碼
如果 T-SQL 指令碼定義在檔案內，請利用此選項匯入指令碼：

    $scriptName = "My Script Imported from a File"
    $scriptPath = "{Path to SQL File}"
    $scriptCommandText = Get-Content -Path $scriptPath
    $script = New-AzureSqlJobContent -ContentName $scriptName -CommandText $scriptCommandText
    Write-Output $script

### <a name="to-update-a-t-sql-script-for-execution-across-databases"></a>針對跨資料庫執行更新 T-SQL 指令碼
此 PowerShell 指令碼會更新現有指令碼的 T-SQL 命令文字。

設定下列變數以反映要設定的所需指令碼定義：

    $scriptName = "Create a TestTable"
    $scriptUpdateComment = "Adding AdditionalInformation column to TestTable"
    $scriptCommandText = "
    IF NOT EXISTS (SELECT name FROM sys.tables WHERE name = 'TestTable')
    BEGIN
    CREATE TABLE TestTable(
        TestTableId INT PRIMARY KEY IDENTITY,
        InsertionTime DATETIME2
    );
    END
    GO

    IF NOT EXISTS (SELECT columns.name FROM sys.columns INNER JOIN sys.tables on columns.object_id = tables.object_id WHERE tables.name = 'TestTable' AND columns.name = 'AdditionalInformation')
    BEGIN
    ALTER TABLE TestTable
    ADD AdditionalInformation NVARCHAR(400);
    END
    GO

    INSERT INTO TestTable(InsertionTime, AdditionalInformation) VALUES (sysutcdatetime(), 'test');
    GO"

### <a name="to-update-the-definition-to-an-existing-script"></a>更新現有指令碼的定義
    Set-AzureSqlJobContentDefinition -ContentName $scriptName -CommandText $scriptCommandText -Comment $scriptUpdateComment 

## <a name="to-create-a-job-to-execute-a-script-across-a-shard-map"></a>建立工作以跨分區對應執行指令碼
此 PowerShell 指令碼會啟動工作，以跨 Elastic Scale 分區對應中每個分區執行指令碼。

設定下列變數以反映所需的指令碼和目標：

    $jobName = "{Job Name}"
    $scriptName = "{Script Name}"
    $shardMapServerName = "{Shard Map Server Name}"
    $shardMapDatabaseName = "{Shard Map Database Name}"
    $shardMapName = "{Shard Map Name}"
    $credentialName = "{Credential Name}"
    $shardMapTarget = Get-AzureSqlJobTarget -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapServerName -ShardMapName $shardMapName 
    $job = New-AzureSqlJob -ContentName $scriptName -CredentialName $credentialName -JobName $jobName -TargetId $shardMapTarget.TargetId
    Write-Output $job

## <a name="to-execute-a-job"></a>執行工作
此 PowerShell 指令碼會執行現有工作：

更新下列變數以反映要執行的所需工作名稱：

    $jobName = "{Job Name}"
    $jobExecution = Start-AzureSqlJobExecution -JobName $jobName 
    Write-Output $jobExecution

## <a name="to-retrieve-the-state-of-a-single-job-execution"></a>擷取單一工作執行的狀態
使用 [**Get-AzureSqlJobExecution Cmdlet**](/powershell/module/elasticdatabasejobs/get-azuresqljobexecution) 並且設定 **JobExecutionId** 參數，以檢視工作執行的狀態。

    $jobExecutionId = "{Job Execution Id}"
    $jobExecution = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId
    Write-Output $jobExecution

使用相同 **Get-AzureSqlJobExecution** Cmdlet 搭配 **IncludeChildren** 參數，以檢視子工作執行的狀態，也就是工作在每個目標資料庫上的每個工作執行的特定狀態。

    $jobExecutionId = "{Job Execution Id}"
    $jobExecutions = Get-AzureSqlJobExecution -JobExecutionId $jobExecutionId -IncludeChildren
    Write-Output $jobExecutions 

## <a name="to-view-the-state-across-multiple-job-executions"></a>檢視跨多個工作執行的狀態
[**Get-AzureSqlJobExecution Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljob) 具有多個選用參數，可用來顯示多個工作執行、透過提供的參數篩選。 以下示範一些使用 Get-AzureSqlJobExecution 的可能方式：

擷取所有作用中最上層工作執行：

    Get-AzureSqlJobExecution

擷取所有最上層工作執行，包括非使用中工作執行：

    Get-AzureSqlJobExecution -IncludeInactive

擷取已提供工作執行 ID 的所有子工作執行，包括非使用中工作執行：

    $parentJobExecutionId = "{Job Execution Id}"
    Get-AzureSqlJobExecution -AzureSqlJobExecution -JobExecutionId $parentJobExecutionId -IncludeInactive -IncludeChildren

擷取使用排程/工作組合建立的所有工作執行，包括非使用中工作：

    $jobName = "{Job Name}"
    $scheduleName = "{Schedule Name}"
    Get-AzureSqlJobExecution -JobName $jobName -ScheduleName $scheduleName -IncludeInactive

擷取以指定的分區對應為目標的所有工作，包括非使用中工作：

    $shardMapServerName = "{Shard Map Server Name}"
    $shardMapDatabaseName = "{Shard Map Database Name}"
    $shardMapName = "{Shard Map Name}"
    $target = Get-AzureSqlJobTarget -ShardMapManagerDatabaseName $shardMapDatabaseName -ShardMapManagerServerName $shardMapServerName -ShardMapName $shardMapName
    Get-AzureSqlJobExecution -TargetId $target.TargetId -IncludeInactive

擷取以指定的自訂集合為目標的所有工作，包括非使用中工作：

    $customCollectionName = "{Custom Collection Name}"
    $target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
    Get-AzureSqlJobExecution -TargetId $target.TargetId -IncludeInactive

擷取特定工作執行內的工作作業執行的清單：

    $jobExecutionId = "{Job Execution Id}"
    $jobTaskExecutions = Get-AzureSqlJobTaskExecution -JobExecutionId $jobExecutionId
    Write-Output $jobTaskExecutions 

擷取工作作業執行詳細資料：

下列 PowerShell 指令碼可用來檢視工作作業執行的詳細資料，在偵錯執行失敗時特別有用。

    $jobTaskExecutionId = "{Job Task Execution Id}"
    $jobTaskExecution = Get-AzureSqlJobTaskExecution -JobTaskExecutionId $jobTaskExecutionId
    Write-Output $jobTaskExecution

## <a name="to-retrieve-failures-within-job-task-executions"></a>擷取工作作業執行內的失敗
**JobTaskExecution 物件** 包括作業生命週期的屬性和訊息屬性。 如果工作作業執行失敗，生命週期屬性將設為「失敗」  ，且訊息屬性將設為產生的例外狀況訊息和其堆疊。 如果工作不成功，務必檢視指定作業不成功的工作作業的詳細資料。

    $jobExecutionId = "{Job Execution Id}"
    $jobTaskExecutions = Get-AzureSqlJobTaskExecution -JobExecutionId $jobExecutionId
    Foreach($jobTaskExecution in $jobTaskExecutions) 
        {
        if($jobTaskExecution.Lifecycle -ne 'Succeeded')
            {
            Write-Output $jobTaskExecution
            }
        }

## <a name="to-wait-for-a-job-execution-to-complete"></a>等候工作執行完成
下列 PowerShell 指令碼可以用來等候工作作業完成：

    $jobExecutionId = "{Job Execution Id}"
    Wait-AzureSqlJobExecution -JobExecutionId $jobExecutionId 

## <a name="create-a-custom-execution-policy"></a>建立自訂執行原則
彈性資料庫工作支援建立自訂執行原則，可以在啟動作業時套用。

執行原則目前允許定義：

* 名稱：執行原則的識別碼。
* 工作逾時：彈性資料庫工作取消工作之前的總時間。
* 初始重試間隔：第一次重試之前等候的間隔。
* 最大重試間隔：要使用的重試間隔端點。
* 重試間隔輪詢係數：用來計算重試之間下一個間隔的係數。  使用下列公式：(初始重試間隔) * Math.pow ((間隔輪詢係數), (重試次數) -2)。 
* 嘗試上限：工作內執行的重試嘗試數目上限。

預設的執行原則會使用下列值：

* 名稱：預設執行原則
* 工作逾時：1 週
* 初始重試間隔：100 毫秒
* 最大重試間隔：30 分鐘
* 重試間隔係數：2
* 嘗試上限：2,147,483,647

建立想要的執行原則：

    $executionPolicyName = "{Execution Policy Name}"
    $initialRetryInterval = New-TimeSpan -Seconds 10
    $jobTimeout = New-TimeSpan -Minutes 30
    $maximumAttempts = 999999
    $maximumRetryInterval = New-TimeSpan -Minutes 1
    $retryIntervalBackoffCoefficient = 1.5
    $executionPolicy = New-AzureSqlJobExecutionPolicy -ExecutionPolicyName $executionPolicyName -InitialRetryInterval $initialRetryInterval -JobTimeout $jobTimeout -MaximumAttempts $maximumAttempts -MaximumRetryInterval $maximumRetryInterval 
    -RetryIntervalBackoffCoefficient $retryIntervalBackoffCoefficient
    Write-Output $executionPolicy

### <a name="update-a-custom-execution-policy"></a>更新自訂執行原則
更新要更新之想要的執行原則：

    $executionPolicyName = "{Execution Policy Name}"
    $initialRetryInterval = New-TimeSpan -Seconds 15
    $jobTimeout = New-TimeSpan -Minutes 30
    $maximumAttempts = 999999
    $maximumRetryInterval = New-TimeSpan -Minutes 1
    $retryIntervalBackoffCoefficient = 1.5
    $updatedExecutionPolicy = Set-AzureSqlJobExecutionPolicy -ExecutionPolicyName $executionPolicyName -InitialRetryInterval $initialRetryInterval -JobTimeout $jobTimeout -MaximumAttempts $maximumAttempts -MaximumRetryInterval $maximumRetryInterval -RetryIntervalBackoffCoefficient $retryIntervalBackoffCoefficient
    Write-Output $updatedExecutionPolicy

## <a name="cancel-a-job"></a>取消工作
彈性資料庫工作支援取消工作要求。  如果彈性資料庫工作偵測到目前正在執行工作的取消要求，它會嘗試停止工作。

彈性資料庫工作有兩種不同的方式可以執行取消作業：

1. 取消目前正在執行的作業：如果作業正在執行時偵測到取消，將會在目前正在執行的作業層面內嘗試取消。  例如：當嘗試取消時，如果有長時間執行查詢目前正在執行，將會嘗試取消查詢。
2. 取消作業重試：如果控制執行緒在啟動作業執行之前偵測到取消，控制執行緒會避免啟動作業，並且將要求宣告為已取消。

如果針對父工作要求工作取消，則會對父工作和其所有子工作執行取消要求。

若要提交取消要求，請使用 [**Stop-AzureSqlJobExecution Cmdlet**](/powershell/module/elasticdatabasejobs/stop-azuresqljobexecution) 並設定 **JobExecutionId** 參數。

    $jobExecutionId = "{Job Execution Id}"
    Stop-AzureSqlJobExecution -JobExecutionId $jobExecutionId

## <a name="to-delete-a-job-and-job-history-asynchronously"></a>以非同步方式刪除工作和工作歷程記錄
彈性資料庫工作支援非同步刪除工作。 工作可以標示為刪除，系統將會在工作的工作執行皆已完成之後，刪除工作和其所有工作歷程記錄。 系統不會自動取消作用中的工作執行。  

叫用 [**Stop-AzureSqlJobExecution**](/powershell/module/elasticdatabasejobs/stop-azuresqljobexecution) 以取消作用中的工作執行。

若要觸發工作刪除，請使用 [**Remove-AzureSqlJob Cmdlet**](/powershell/module/elasticdatabasejobs/remove-azuresqljob) 並設定 **JobName** 參數。

    $jobName = "{Job Name}"
    Remove-AzureSqlJob -JobName $jobName

## <a name="to-create-a-custom-database-target"></a>建立自訂資料庫目標
您可以定義自訂資料庫目標以直接執行或包含在自訂資料庫群組內。 例如，由於使用 PowerShell API 還無法直接支援「彈性集區」，因此您可以建立自訂資料庫目標和自訂資料庫集合目標，以包含集區中的所有資料庫。

設定下列變數以反映所需的資料庫資訊：

    $databaseName = "{Database Name}"
    $databaseServerName = "{Server Name}"
    New-AzureSqlJobTarget -DatabaseName $databaseName -ServerName $databaseServerName 

## <a name="to-create-a-custom-database-collection-target"></a>建立自訂資料庫集合目標
使用 [**New-AzureSqlJobTarget Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljobtarget) 定義自訂資料庫集合目標，以跨多個已定義的資料庫目標執行。 建立資料庫群組之後，資料庫可以與自訂集合目標相關聯。

設定下列變數以反映所需的自訂集合目標組態：

    $customCollectionName = "{Custom Database Collection Name}"
    New-AzureSqlJobTarget -CustomCollectionName $customCollectionName 

### <a name="to-add-databases-to-a-custom-database-collection-target"></a>將資料庫新增至自訂資料庫集合目標
若要將資料庫新增至特定的自訂集合，請使用 [**Add-AzureSqlJobChildTarget Cmdlet**](/powershell/module/elasticdatabasejobs/add-azuresqljobchildtarget)。

    $databaseServerName = "{Database Server Name}"
    $databaseName = "{Database Name}"
    $customCollectionName = "{Custom Database Collection Name}"
    Add-AzureSqlJobChildTarget -CustomCollectionName $customCollectionName -DatabaseName $databaseName -ServerName $databaseServerName 

#### <a name="review-the-databases-within-a-custom-database-collection-target"></a>檢閱自訂資料庫集合目標內的資料庫
使用 [**Get-AzureSqlJobTarget Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljobtarget) 以擷取自訂資料庫集合目標內的子資料庫。 

    $customCollectionName = "{Custom Database Collection Name}"
    $target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
    $childTargets = Get-AzureSqlJobTarget -ParentTargetId $target.TargetId
    Write-Output $childTargets

### <a name="create-a-job-to-execute-a-script-across-a-custom-database-collection-target"></a>建立工作以跨自訂資料庫集合目標執行指令碼
使用 [**New-AzureSqlJob Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljob) 以根據自訂資料庫集合目標定義的資料庫群組建立工作。 彈性資料庫工作會將工作展開成多個子工作，每個子工作對應至與自訂資料庫集合目標相關聯的資料庫，並且確保指令碼會針對每個資料庫執行。 再次重申，很重要的是指令碼具有等冪處理重試的彈性。

    $jobName = "{Job Name}"
    $scriptName = "{Script Name}"
    $customCollectionName = "{Custom Collection Name}"
    $credentialName = "{Credential Name}"
    $target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName
    $job = New-AzureSqlJob -JobName $jobName -CredentialName $credentialName -ContentName $scriptName -TargetId $target.TargetId
    Write-Output $job

## <a name="data-collection-across-databases"></a>跨資料庫的資料集合
您可以使用工作來跨一組資料庫執行查詢，並將結果傳送至特定的資料表。 可以在事實之後查詢資料表，以查看每個資料庫的查詢結果。 這麼做即可以非同步方式執行跨許多資料庫的查詢。 嘗試失敗時自動經由重試來處理。

如果指定的目的地資料表尚未存在，則會自動建立。 新的資料表符合傳回的結果集的結構描述。 如果指令碼傳回多個結果集，彈性資料庫工作只會將第一個結果集傳送至目的地資料表。

下列 PowerShell 指令碼會執行指令碼，並將結果收集至指定的資料表。 此指令碼假設已建立一個會輸出單一結果集的 T-SQL 指令碼，也假設已建立自訂資料庫集合目標。

此指令碼使用 [**Get-AzureSqlJobTarget Cmdlet**](/powershell/module/elasticdatabasejobs/new-azuresqljobtarget)。 設定指令碼、認證和執行目標的參數：

    $jobName = "{Job Name}"
    $scriptName = "{Script Name}"
    $executionCredentialName = "{Execution Credential Name}"
    $customCollectionName = "{Custom Collection Name}"
    $destinationCredentialName = "{Destination Credential Name}"
    $destinationServerName = "{Destination Server Name}"
    $destinationDatabaseName = "{Destination Database Name}"
    $destinationSchemaName = "{Destination Schema Name}"
    $destinationTableName = "{Destination Table Name}"
    $target = Get-AzureSqlJobTarget -CustomCollectionName $customCollectionName

### <a name="to-create-and-start-a-job-for-data-collection-scenarios"></a>建立和啟動資料庫集合案例的工作
此指令碼使用 [**Start-AzureSqlJobExecution Cmdlet**](/powershell/module/elasticdatabasejobs/start-azuresqljobexecution)。

    $job = New-AzureSqlJob -JobName $jobName 
    -CredentialName $executionCredentialName 
    -ContentName $scriptName 
    -ResultSetDestinationServerName $destinationServerName 
    -ResultSetDestinationDatabaseName $destinationDatabaseName 
    -ResultSetDestinationSchemaName $destinationSchemaName 
    -ResultSetDestinationTableName $destinationTableName 
    -ResultSetDestinationCredentialName $destinationCredentialName 
    -TargetId $target.TargetId
    Write-Output $job
    $jobExecution = Start-AzureSqlJobExecution -JobName $jobName
    Write-Output $jobExecution

## <a name="to-schedule-a-job-execution-trigger"></a>排程工作執行觸發程序
下列 PowerShell 指令碼可以用來建立週期性排程。 這個指令碼使用分鐘間隔，但是 [**New-AzureSqlJobSchedule**](/powershell/module/elasticdatabasejobs/new-azuresqljobschedule) 也支援 -DayInterval、-HourInterval、-MonthInterval 和 -WeekInterval 參數。 您可以藉由傳遞 -OneTime，建立僅執行一次的排程。

建立新的排程：

    $scheduleName = "Every one minute"
    $minuteInterval = 1
    $startTime = (Get-Date).ToUniversalTime()
    $schedule = New-AzureSqlJobSchedule 
    -MinuteInterval $minuteInterval 
    -ScheduleName $scheduleName 
    -StartTime $startTime 
    Write-Output $schedule

### <a name="to-trigger-a-job-executed-on-a-time-schedule"></a>觸發依時間排程執行的工作
可以定義工作觸發程序，讓工作根據時間排程執行。 下列 PowerShell 指令碼可以用來建立工作觸發程序。

使用 [New-AzureSqlJobTrigger](/powershell/module/elasticdatabasejobs/new-azuresqljobtrigger) ，並設定下列變數以對應至所需的工作和排程：

    $jobName = "{Job Name}"
    $scheduleName = "{Schedule Name}"
    $jobTrigger = New-AzureSqlJobTrigger
    -ScheduleName $scheduleName
    -JobName $jobName
    Write-Output $jobTrigger

### <a name="to-remove-a-scheduled-association-to-stop-job-from-executing-on-schedule"></a>移除排程關聯以停止依排程執行工作
若要透過工作觸發程序中止工作重複執行，可以移除工作觸發程序。 使用 [**Remove-AzureSqlJobTrigger Cmdlet**](/powershell/module/elasticdatabasejobs/remove-azuresqljobtrigger)，移除工作觸發程序以停止工作根據排程執行。

    $jobName = "{Job Name}"
    $scheduleName = "{Schedule Name}"
    Remove-AzureSqlJobTrigger 
    -ScheduleName $scheduleName 
    -JobName $jobName

### <a name="retrieve-job-triggers-bound-to-a-time-schedule"></a>擷取繫結至時間排程的工作觸發程序
下列 PowerShell 指令碼可用來取得並顯示註冊至特定時間排程的工作觸發程序。

    $scheduleName = "{Schedule Name}"
    $jobTriggers = Get-AzureSqlJobTrigger -ScheduleName $scheduleName
    Write-Output $jobTriggers

### <a name="to-retrieve-job-triggers-bound-to-a-job"></a>擷取繫結至工作的工作觸發程序
使用 [Get-AzureSqlJobTrigger](/powershell/module/elasticdatabasejobs/get-azuresqljobtrigger) 以取得和顯示包含已註冊工作的排程。

    $jobName = "{Job Name}"
    $jobTriggers = Get-AzureSqlJobTrigger -JobName $jobName
    Write-Output $jobTriggers

## <a name="to-create-a-data-tier-application-dacpac-for-execution-across-databases"></a>建立資料層應用程式 (DACPAC) 以跨資料庫執行
若要建立 DACPAC，請參閱 [資料層應用程式](https://msdn.microsoft.com/library/ee210546.aspx)。 若要部署 DACPAC、請使用 [New-AzureSqlJobContent Cmdlet](/powershell/module/elasticdatabasejobs/new-azuresqljobcontent)。 DACPAC 必須可供服務存取。 建議將建立的 DACPAC 上傳至 Azure 儲存體，並且為 DACPAC 建立 [共用存取簽章](../storage/common/storage-dotnet-shared-access-signature-part-1.md) 。

    $dacpacUri = "{Uri}"
    $dacpacName = "{Dacpac Name}"
    $dacpac = New-AzureSqlJobContent -DacpacUri $dacpacUri -ContentName $dacpacName 
    Write-Output $dacpac

### <a name="to-update-a-data-tier-application-dacpac-for-execution-across-databases"></a>更新資料層應用程式 (DACPAC) 以跨資料庫執行
彈性資料庫工作內的現有已註冊 DACPAC 可以更新以指向新的 URI。 使用 [**Set-AzureSqlJobContentDefinition Cmdlet**](/powershell/module/elasticdatabasejobs/set-azuresqljobcontentdefinition) 更新現有已註冊 DACPAC 上的 DACPAC URI：

    $dacpacName = "{Dacpac Name}"
    $newDacpacUri = "{Uri}"
    $updatedDacpac = Set-AzureSqlJobDacpacDefinition -ContentName $dacpacName -DacpacUri $newDacpacUri
    Write-Output $updatedDacpac

## <a name="to-create-a-job-to-apply-a-data-tier-application-dacpac-across-databases"></a>建立工作以跨資料庫套用資料層應用程式 (DACPAC)
在彈性資料庫工作內建立 DACPAC 之後，可以建立工作以跨資料庫群組套用 DACPAC。 下列 PowerShell 指令碼可以用來跨自訂資料庫集合建立 DACPAC 工作：

    $jobName = "{Job Name}"
    $dacpacName = "{Dacpac Name}"
    $customCollectionName = "{Custom Collection Name}"
    $credentialName = "{Credential Name}"
    $target = Get-AzureSqlJobTarget 
    -CustomCollectionName $customCollectionName
    $job = New-AzureSqlJob 
    -JobName $jobName 
    -CredentialName $credentialName 
    -ContentName $dacpacName -TargetId $target.TargetId
    Write-Output $job 

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-jobs-powershell/cmd-prompt.png
[2]: ./media/sql-database-elastic-jobs-powershell/portal.png
<!--anchors-->
