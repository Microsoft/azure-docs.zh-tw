---
title: 使用 Azure PowerShell 管理 Azure Data Lake Analytics
description: 本文說明如何使用 Azure PowerShell 來管理 Data Lake Analytics 帳戶、資料來源、使用者和作業。
services: data-lake-analytics
ms.service: data-lake-analytics
author: matt1883
ms.author: mahi
ms.reviewer: jasonwhowell
ms.assetid: ad14d53c-fed4-478d-ab4b-6d2e14ff2097
ms.topic: conceptual
ms.date: 06/29/2018
ms.openlocfilehash: b6bed1d42f67c2b5d2ef34a547215c280acb50b8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963759"
---
# <a name="manage-azure-data-lake-analytics-using-azure-powershell"></a>使用 Azure PowerShell 管理 Azure Data Lake Analytics
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用 Azure PowerShell 來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。

## <a name="prerequisites"></a>必要條件

若要搭配使用 PowerShell 與 Data Lake Analytics，請收集下列幾項資訊： 

* **訂用帳戶識別碼**：Data Lake Analytics 帳戶所在 Azure 訂用帳戶的識別碼。
* **資源群組**：包含您 Data Lake Analytics 帳戶的 Azure 資源群組名稱。
* **Data Lake Analytics 帳戶名稱**：Data Lake Analytics 帳戶的名稱。
* **預設 Data Lake Store 帳戶名稱**：每個 Data Lake Analytics 帳戶都有一個預設的 Data Lake Store 帳戶。
* **位置**：您 Data Lake Analytics 帳戶的位置，例如「美國東部 2」或其他支援的位置。

本教學課程中的 PowerShell 程式碼片段會使用這些變數來儲存此資訊

```powershell
$subId = "<SubscriptionId>"
$rg = "<ResourceGroupName>"
$adla = "<DataLakeAnalyticsAccountName>"
$adls = "<DataLakeStoreAccountName>"
$location = "<Location>"
```

## <a name="log-in-to-azure"></a>登入 Azure

### <a name="log-in-using-interactive-user-authentication"></a>使用互動式使用者驗證來登入

使用訂用帳戶識別碼或以訂用帳戶名稱來登入

```powershell
# Using subscription id
Connect-AzureRmAccount -SubscriptionId $subId

# Using subscription name
Connect-AzureRmAccount -SubscriptionName $subname 
```

## <a name="saving-authentication-context"></a>儲存驗證內容

`Connect-AzureRmAccount` Cmdlet 一律會提示輸入認證。 您可以使用下列 Cmdlet 來避免出現提示：

```powershell
# Save login session information
Save-AzureRmProfile -Path D:\profile.json  

# Load login session information
Select-AzureRmProfile -Path D:\profile.json 
```

### <a name="log-in-using-a-service-principal-identity-spi"></a>使用服務主體身分識別 (SPI) 來登入

```powershell
$tenantid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"  
$spi_appname = "appname" 
$spi_appid = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX" 
$spi_secret = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" 

$pscredential = New-Object System.Management.Automation.PSCredential ($spi_appid, (ConvertTo-SecureString $spi_secret -AsPlainText -Force))
Login-AzureRmAccount -ServicePrincipal -TenantId $tenantid -Credential $pscredential -Subscription $subid
```

## <a name="manage-accounts"></a>管理帳戶


### <a name="list-accounts"></a>列出帳戶

```powershell
# List Data Lake Analytics accounts within the current subscription.
Get-AdlAnalyticsAccount

# List Data Lake Analytics accounts within a specific resource group.
Get-AdlAnalyticsAccount -ResourceGroupName $rg
```

### <a name="create-an-account"></a>建立帳戶

每個 Data Lake Analytics 帳戶都需要預設 Data Lake Store 帳戶，用於儲存記錄。 您可以重複使用現有的帳戶，或建立一個帳戶。 

```powershell
# Create a data lake store if needed, or you can re-use an existing one
New-AdlStore -ResourceGroupName $rg -Name $adls -Location $location
New-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla -Location $location -DefaultDataLake $adls
```

### <a name="get-account-information"></a>取得帳戶資訊

取得帳戶的相關詳細資料。

```powershell
Get-AdlAnalyticsAccount -Name $adla
```

### <a name="check-if-an-account-exists"></a>檢查帳戶是否存在

```powershell
Test-AdlAnalyticsAccount -Name $adla
```

## <a name="manage-data-sources"></a>管理資料來源
Azure Data Lake Analytics 目前支援下列資料來源：

* [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md)
* [Azure 儲存體](../storage/common/storage-introduction.md)

每個 Data Lake Analytics 帳戶都有預設 Data Lake Store 帳戶。 預設的 Data Lake Store 帳戶是用來儲存工作中繼資料與工作稽核記錄。 

### <a name="find-the-default-data-lake-store-account"></a>尋找預設的 Data Lake Store 帳戶

```powershell
$adla_acct = Get-AdlAnalyticsAccount -Name $adla
$dataLakeStoreName = $adla_acct.DefaultDataLakeAccount
```

您可以用 `IsDefault` 屬性篩選資料來源清單，藉此方式尋找預設的 Data Lake Store 帳戶：

```powershell
Get-AdlAnalyticsDataSource -Account $adla  | ? { $_.IsDefault } 
```

### <a name="add-a-data-source"></a>建立資料來源

```powershell

# Add an additional Storage (Blob) account.
$AzureStorageAccountName = "<AzureStorageAccountName>"
$AzureStorageAccountKey = "<AzureStorageAccountKey>"
Add-AdlAnalyticsDataSource -Account $adla -Blob $AzureStorageAccountName -AccessKey $AzureStorageAccountKey

# Add an additional Data Lake Store account.
$AzureDataLakeStoreName = "<AzureDataLakeStoreAccountName"
Add-AdlAnalyticsDataSource -Account $adla -DataLakeStore $AzureDataLakeStoreName 
```

### <a name="list-data-sources"></a>列出資料來源

```powershell
# List all the data sources
Get-AdlAnalyticsDataSource -Account $adla

# List attached Data Lake Store accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "DataLakeStore"

# List attached Storage accounts
Get-AdlAnalyticsDataSource -Account $adla | where -Property Type -EQ "Blob"
```

## <a name="submit-u-sql-jobs"></a>提交 U-SQL 作業

### <a name="submit-a-string-as-a-u-sql-job"></a>以 U-SQL 作業的形式提交字串

```powershell
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"@

$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath 

Submit-AdlJob -AccountName $adla -Script $script -Name "Demo"
```

### <a name="submit-a-file-as-a-u-sql-job"></a>以 U-SQL 作業的形式提交檔案

```powershell
$scriptpath = "d:\test.usql"
$script | Out-File $scriptpath 
Submit-AdlJob -AccountName $adla –ScriptPath $scriptpath -Name "Demo"
```

### <a name="list-jobs"></a>列出工作

輸出包含目前執行中作業和最近完成的作業。

```powershell
Get-AdlJob -Account $adla
```

### <a name="list-the-top-n-jobs"></a>列出前 N 個作業

預設會在提交時排序作業清單。 因此，最新提交的作業會顯示在最前面。 根據預設，ADLA 帳戶會記住 180 天內的作業，但 Ge-AdlJob Cmdlet 預設只會傳回前 500 個作業。 請使用 -Top 參數來列出特定數目的作業。

```powershell
$jobs = Get-AdlJob -Account $adla -Top 10
```

### <a name="list-jobs-by-job-state"></a>依作業狀態列出作業

使用 `-State` 參數。 您可以結合以下這些值：

* `Accepted`
* `Compiling`
* `Ended`
* `New`
* `Paused`
* `Queued`
* `Running`
* `Scheduling`
* `Start`

```powershell
# List the running jobs
Get-AdlJob -Account $adla -State Running

# List the jobs that have completed
Get-AdlJob -Account $adla -State Ended

# List the jobs that have not started yet
Get-AdlJob -Account $adla -State Accepted,Compiling,New,Paused,Scheduling,Start
```

### <a name="list-jobs-by-job-result"></a>依作業結果列出作業

使用 `-Result` 參數來偵測已結束的工作是否順利完成。 它有下列值：

* 已取消
* Failed
* None
* Succeeded

``` powershell
# List Successful jobs.
Get-AdlJob -Account $adla -State Ended -Result Succeeded

# List Failed jobs.
Get-AdlJob -Account $adla -State Ended -Result Failed
```

### <a name="list-jobs-by-job-submitter"></a>依作業提交者列出作業

`-Submitter` 參數可協助您識別由誰提交工作。

```powershell
Get-AdlJob -Account $adla -Submitter "joe@contoso.com"
```

### <a name="list-jobs-by-submission-time"></a>依提交時間列出作業

`-SubmittedAfter` 用於篩選時間範圍。


```powershell
# List  jobs submitted in the last day.
$d = [DateTime]::Now.AddDays(-1)
Get-AdlJob -Account $adla -SubmittedAfter $d

# List  jobs submitted in the last seven day.
$d = [DateTime]::Now.AddDays(-7)
Get-AdlJob -Account $adla -SubmittedAfter $d
```

### <a name="get-job-status"></a>取得作業狀態

取得特定作業的狀態。

```powershell
Get-AdlJob -AccountName $adla -JobId $job.JobId
```


### <a name="cancel-a-job"></a>取消工作

```powershell
Stop-AdlJob -Account $adla -JobID $jobID
```

### <a name="wait-for-a-job-to-finish"></a>等候作業結束

您可以不重複執行 `Get-AdlAnalyticsJob` 直到作業結束，而是使用 `Wait-AdlJob` Cmdlet 來等候作業結束。

```powershell
Wait-AdlJob -Account $adla -JobId $job.JobId
```

## <a name="analyzing-job-history"></a>分析作業記錄

使用 Azure PowerShell 來分析已在 Data Lake 分析中執行之作業的記錄，是一項強大的技術。 您可以使用這項技術來深入了解使用情形和成本。 若要深入了解，請參閱[作業記錄分析存放庫範例](https://github.com/Azure-Samples/data-lake-analytics-powershell-job-history-analysis)  

## <a name="list-job-pipelines-and-recurrences"></a>列出作業管線和週期

使用 `Get-AdlJobPipeline` Cmdlet 來查看先前提交作業的管線資訊。

```powershell
$pipelines = Get-AdlJobPipeline -Account $adla
$pipeline = Get-AdlJobPipeline -Account $adla -PipelineId "<pipeline ID>"
```

使用 `Get-AdlJobRecurrence` Cmdlet 來查看先前提交作業的週期資訊。

```powershell
$recurrences = Get-AdlJobRecurrence -Account $adla

$recurrence = Get-AdlJobRecurrence -Account $adla -RecurrenceId "<recurrence ID>"
```


## <a name="manage-compute-policies"></a>管理計算原則

### <a name="list-existing-compute-policies"></a>列出現有的計算原則

`Get-AdlAnalyticsComputePolicy` Cmdlet 會擷取 Data Lake Analytics 帳戶的計算原則清單。

```powershell
$policies = Get-AdlAnalyticsComputePolicy -Account $adla
```

### <a name="create-a-compute-policy"></a>建立計算原則

`New-AdlAnalyticsComputePolicy` Cmdlet 會為 Data Lake Analytics 帳戶建立新的計算原則。 這個範例會將指定使用者的可用 AU 上限設定為 50，將最低作業優先權設為 250。

```powershell
$userObjectId = (Get-AzureRmAdUser -SearchString "garymcdaniel@contoso.com").Id

New-AdlAnalyticsComputePolicy -Account $adla -Name "GaryMcDaniel" -ObjectId $objectId -ObjectType User -MaxDegreeOfParallelismPerJob 50 -MinPriorityPerJob 250
```
## <a name="manage-files"></a>管理檔案

### <a name="check-for-the-existence-of-a-file"></a>檢查檔案是否存在。

```powershell
Test-AdlStoreItem -Account $adls -Path "/data.csv"
```

### <a name="uploading-and-downloading"></a>上傳和下載

上傳檔案。

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\data.tsv" -Destination "/data_copy.csv" 
```

以遞迴方式上傳整個資料夾。

```powershell
Import-AdlStoreItem -AccountName $adls -Path "c:\myData\" -Destination "/myData/" -Recurse
```

下載檔案。

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/data.csv" -Destination "c:\data.csv"
```

以遞迴方式下載整個資料夾。

```powershell
Export-AdlStoreItem -AccountName $adls -Path "/" -Destination "c:\myData\" -Recurse
```

> [!NOTE]
> 如果上傳或下載程序中斷，您可以搭配 ``-Resume`` 旗標來重新執行該 Cmdlet，以嘗試繼續該程序。

## <a name="manage-the-u-sql-catalog"></a>管理 U-SQL 目錄

U-SQL 目錄是用來建構資料和程式碼，讓 U-SQL 指令碼可以共用它們。 目錄可以讓 Azure Data Lake 中的資料具有可能的最高效能。 如需詳細資訊，請參閱 [使用 U-SQL 目錄](data-lake-analytics-use-u-sql-catalog.md)。

### <a name="list-items-in-the-u-sql-catalog"></a>列出 U-SQL 目錄中的項目

```powershell
# List U-SQL databases
Get-AdlCatalogItem -Account $adla -ItemType Database 

# List tables within a database
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database"

# List tables within a schema.
Get-AdlCatalogItem -Account $adla -ItemType Table -Path "database.schema"
```

### <a name="list-all-the-assemblies-the-u-sql-catalog"></a>列出所有組件 U-SQL 目錄

```powershell
$dbs = Get-AdlCatalogItem -Account $adla -ItemType Database

foreach ($db in $dbs)
{
    $asms = Get-AdlCatalogItem -Account $adla -ItemType Assembly -Path $db.Name

    foreach ($asm in $asms)
    {
        $asmname = "[" + $db.Name + "].[" + $asm.Name + "]"
        Write-Host $asmname
    }
}
```

### <a name="get-details-about-a-catalog-item"></a>取得目錄項目的相關詳細資料

```powershell
# Get details of a table
Get-AdlCatalogItem  -Account $adla -ItemType Table -Path "master.dbo.mytable"

# Test existence of a U-SQL database.
Test-AdlCatalogItem  -Account $adla -ItemType Database -Path "master"
```

### <a name="store-credentials-in-the-catalog"></a>將認證儲存在目錄中

在 U-SQL 資料庫內，為裝載於 Azure 中的資料庫建立認證物件。 目前，U-SQL 認證是您可以透過 PowerShell 建立的唯一目錄項目類型。

```powershell
$dbName = "master"
$credentialName = "ContosoDbCreds"
$dbUri = "https://contoso.database.windows.net:8080"

New-AdlCatalogCredential -AccountName $adla `
          -DatabaseName $db `
          -CredentialName $credentialName `
          -Credential (Get-Credential) `
          -Uri $dbUri
```

## <a name="manage-firewall-rules"></a>管理防火牆規則

### <a name="list-firewall-rules"></a>列出防火牆規則

```powershell
Get-AdlAnalyticsFirewallRule -Account $adla
```

### <a name="add-a-firewall-rule"></a>新增防火牆規則

```powershell
$ruleName = "Allow access from on-prem server"
$startIpAddress = "<start IP address>"
$endIpAddress = "<end IP address>"

Add-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="modify-a-firewall-rule"></a>修改防火牆規則

```powershell
Set-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName -StartIpAddress $startIpAddress -EndIpAddress $endIpAddress
```

### <a name="remove-a-firewall-rule"></a>移除防火牆規則

```powershell
Remove-AdlAnalyticsFirewallRule -Account $adla -Name $ruleName
```

### <a name="allow-azure-ip-addresses"></a>允許 Azure IP 位址

```powershell
Set-AdlAnalyticsAccount -Name $adla -AllowAzureIpState Enabled
```

```powershell
Set-AdlAnalyticsAccount -Name $adla -FirewallState Enabled
Set-AdlAnalyticsAccount -Name $adla -FirewallState Disabled
```


## <a name="working-with-azure"></a>使用 Azure

### <a name="get-details-of-azurerm-errors"></a>取得 AzureRm 錯誤詳細資料

```powershell
Resolve-AzureRmError -Last
```

### <a name="verify-if-you-are-running-as-an-administrator-on-your-windows-machine"></a>確認您在 Windows 機器上是否以系統管理員的身分執行

```powershell
function Test-Administrator  
{  
    $user = [Security.Principal.WindowsIdentity]::GetCurrent();
    $p = New-Object Security.Principal.WindowsPrincipal $user
    $p.IsInRole([Security.Principal.WindowsBuiltinRole]::Administrator)  
}
```

### <a name="find-a-tenantid"></a>尋找 TenantID

從訂用帳戶名稱：

```powershell
function Get-TenantIdFromSubcriptionName( [string] $subname )
{
    $sub = (Get-AzureRmSubscription -SubscriptionName $subname)
    $sub.TenantId
}

Get-TenantIdFromSubcriptionName "ADLTrainingMS"
```

從訂用帳戶識別碼：

```powershell
function Get-TenantIdFromSubcriptionId( [string] $subid )
{
    $sub = (Get-AzureRmSubscription -SubscriptionId $subid)
    $sub.TenantId
}

$subid = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
Get-TenantIdFromSubcriptionId $subid
```

從網域位址 (例如 "contoso.com")

```powershell
function Get-TenantIdFromDomain( $domain )
{
    $url = "https://login.windows.net/" + $domain + "/.well-known/openid-configuration"
    return (Invoke-WebRequest $url|ConvertFrom-Json).token_endpoint.Split('/')[3]
}

$domain = "contoso.com"
Get-TenantIdFromDomain $domain
```

### <a name="list-all-your-subscriptions-and-tenant-ids"></a>列出您的所有訂用帳戶和租用戶識別碼

```powershell
$subs = Get-AzureRmSubscription
foreach ($sub in $subs)
{
    Write-Host $sub.Name "("  $sub.Id ")"
    Write-Host "`tTenant Id" $sub.TenantId
}
```

## <a name="create-a-data-lake-analytics-account-using-a-template"></a>使用範本建立 Data Lake Analytics 帳戶

您也可以使用下列範例來使用 Azure 資源群組範本：[使用範本建立 Data Lake Analytics 帳戶](https://github.com/Azure-Samples/data-lake-analytics-create-account-with-arm-template)

## <a name="next-steps"></a>後續步驟
* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* 透過 [Azure 入口網站](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [Azure CLI](data-lake-analytics-get-started-cli.md) 來開始使用 Data Lake Analytics
* 使用 [Azure 入口網站](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) 來管理 Azure Data Lake Analytics 
