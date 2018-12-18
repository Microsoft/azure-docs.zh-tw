---
title: 在 HDInsight 上使用 Hadoop 分析航班延誤資料 - Azure
description: 了解如何使用一個 Windows PowerShell 指令碼建立 HDInsight 叢集、執行 Hive 工作、執行 Sqool 工作和刪除叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 1d39fd1cf40834781375b2c7bd6cb801ccd3cb97
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43091893"
---
# <a name="analyze-flight-delay-data-by-using-hive-in-hdinsight"></a>在 HDInsight 上使用 Hadoop 分析航班延誤資料
Hive 可透過一種類似 SQL 的指令碼語言 (稱為 *[HiveQL][hadoop-hiveql]*) 來執行 Hadoop MapReduce 作業，可用來彙總、查詢和分析大量資料。

> [!IMPORTANT]
> 此文件中的步驟需要 Windows 型 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。 如需與 Linux 叢集搭配使用的步驟，請參閱 [在 HDInsight (Linux) 中使用 Hive 分析航班延誤資料](hdinsight-analyze-flight-delay-data-linux.md)。

Azure HDInsight 的其中一個主要優點就是區隔資料儲存和運算。 HDInsight 使用 Azure Blob 儲存體來儲存資料。 典型的工作包含三個部分：

1. **將資料儲存在 Azure Blob 儲存體中。**  例如，將天氣資料、感應器資料、Web 記錄，以及此案例中的航班延誤資料儲存到 Azure Blob 儲存體中。
2. **執行工作。** 在處理該資料時，您就要執行 Windows PowerShell 指令碼 (或用戶端應用程式) 來建立 HDInsight 叢集、執行工具，然後刪除叢集。 工作會將輸出資料儲存至 Azure Blob 儲存體。 即使在刪除叢集之後，輸出資料仍會保留。 因此，您只需要對已耗用的部分付費。
3. **從 Azure Blob 儲存體擷取輸出**，或在此教學課程中將資料匯出至 Azure SQL Database。

下圖說明本教學課程的案例和結構：

![HDI.FlightDelays.flow][img-hdi-flightdelays-flow]

請注意，圖表中的號碼對應至章節標題。 **M** 代表主要程序。 **A** 代表附錄中的內容。

教學課程的主要部分將顯示如何使用某個 Windows PowerShell 指令碼執行下列工作：

* 建立 HDInsight 叢集。
* 在叢集上執行 Hive 工作，以計算機場的平均延遲。 航班延誤資料會儲存在 Azure Blob 儲存體帳戶
* 執行 Sqoop 工作來匯出 Hive 工作輸出至 Azure SQL Database。
* 刪除 HDInsight 叢集。

在附錄中，您可以找到上傳航班延誤資料、建立/上傳 Hive 查詢字串和針對 Sqoop 工作準備 Azure SQL Database 的指示。

> [!NOTE]
> 此文件中的步驟是針對以 Windows 為基礎的 HDInsight 叢集。 如需與 Linux 叢集搭配使用的步驟，請參閱[在 HDInsight (Linux) 中使用 Hive 分析航班延誤資料](hdinsight-analyze-flight-delay-data-linux.md)

### <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列項目：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **具有 Azure PowerShell 的工作站**。

    > [!IMPORTANT]
    > 使用 Azure Service Manager 管理 HDInsight 資源的 Azure PowerShell 支援已**被取代**，並已在 2017 年 1 月 1 日移除。 本文件中的步驟會使用可與 Azure Resource Manager 搭配使用的新 HDInsight Cmdlet。
    >
    > 請遵循 [安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步驟來安裝最新版的 Azure PowerShell。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱 [移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md) ，以取得詳細資訊。

**本教學課程中使用的檔案**

本教學課程使用的飛行航班準點率資料取自[創新技術研究管理部運輸統計處或 RITA][rita-website]。
資料複本已上傳至具有「公用」Blob 存取權限的 Azure Blob 儲存體容器。
PowerShell 指令碼其中一部分會將資料從公用 Blob 容器複製到叢集的預設 Blob 容器。 HiveQL 指令碼也會複製到相同的 Blob 容器。
如需了解如何將資料放入/上傳至您自己的儲存體帳戶，以及如何建立/上傳 HiveQL 指令碼檔案，請參閱[附錄 A](#appendix-a) 和[附錄 B](#appendix-b)。

下表列出本教學課程中使用的檔案：

<table border="1">
<tr><th>檔案</th><th>說明</th></tr>
<tr><td>wasb://flightdelay@hditutorialdata.blob.core.windows.net/flightdelays.hql</td><td>Hive 工作所用的 HiveQL 指令碼檔案。 此指令碼已上傳至具有公用存取的 Azure Blob 儲存體帳戶。 <a href="#appendix-b">附錄 B</a> 具有準備和上傳此檔案至您自己的 Azure Blob 儲存體帳戶的相關指示。</td></tr>
<tr><td>wasb://flightdelay@hditutorialdata.blob.core.windows.net/2013Data</td><td>Hive 工作的輸入資料。 此資料已上傳至具有公用存取的 Azure Blob 儲存體帳戶。 <a href="#appendix-a">附錄 A</a> 具有取得資料和上傳資料至您自己的 Azure Blob 儲存體帳戶的相關指示。</td></tr>
<tr><td>\tutorials\flightdelays\output</td><td>Hive 工作的輸出路徑。 預設容器用來儲存輸出資料。</td></tr>
<tr><td>\tutorials\flightdelays\jobstatus</td><td>預設容器上的 Hive 工作狀態資料夾。</td></tr>
</table>

## <a name="create-cluster-and-run-hivesqoop-jobs"></a>建立叢集和執行 Hive/Sqoop 工作
Hadoop MapReduce 是批次處理。 執行 Hive 工作時，最具成本效益的方法是建立工作的叢集，並於工作完成之後刪除工作。 下列指令碼涵蓋整個程序。
如需有關建立 HDInsight 叢集和執行 Hive 工作的詳細資訊，請參閱[在 HDInsight 中建立 Hadoop 叢集][hdinsight-provision]和[搭配 HDInsight 使用 Hive][hdinsight-use-hive]。

**使用 Azure PowerShell 執行 Hive 查詢**

1. 使用 [附錄 C](#appendix-c)中的指示，為 Sqoop 工作輸出建立 Azure SQL Database 和資料表。
2. 開啟 Windows PowerShell ISE 並執行下列指令碼：

    ```powershell
    $subscriptionID = "<Azure Subscription ID>"
    $nameToken = "<Enter an Alias>"

    ###########################################
    # You must configure the follwing variables
    # for an existing Azure SQL Database
    ###########################################
    $existingSqlDatabaseServerName = "<Azure SQL Database Server>"
    $existingSqlDatabaseLogin = "<Azure SQL Database Server Login>"
    $existingSqlDatabasePassword = "<Azure SQL Database Server login password>"
    $existingSqlDatabaseName = "<Azure SQL Database name>"

    $localFolder = "E:\Tutorials\Downloads\" # A temp location for copying files.
    $azcopyPath = "C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy" # depends on the version, the folder can be different

    ###########################################
    # (Optional) configure the following variables
    ###########################################

    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $resourceGroupName = $namePrefix + "rg"
    $location = "EAST US 2"

    $HDInsightClusterName = $namePrefix + "hdi"
    $httpUserName = "admin"
    $httpPassword = "<Enter the Password>"

    $defaultStorageAccountName = $namePrefix + "store"
    $defaultBlobContainerName = $HDInsightClusterName # use the cluster name

    $existingSqlDatabaseTableName = "AvgDelays"
    $sqlDatabaseConnectionString = "jdbc:sqlserver://$existingSqlDatabaseServerName.database.windows.net;user=$existingSqlDatabaseLogin@$existingSqlDatabaseServerName;password=$existingSqlDatabaseLogin;database=$existingSqlDatabaseName"

    $hqlScriptFile = "/tutorials/flightdelays/flightdelays.hql"

    $jobStatusFolder = "/tutorials/flightdelays/jobstatus"

    ###########################################
    # Login
    ###########################################
    try{
        $acct = Get-AzureRmSubscription
    }
    catch{
        Connect-AzureRmAccount
    }
    Select-AzureRmSubscription -SubscriptionID $subscriptionID

    ###########################################
    # Create a new HDInsight cluster
    ###########################################

    # Create ARM group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

    # Create the default storage account
    New-AzureRmStorageAccount -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName -Location $location -Type Standard_LRS

    # Create the default Blob container
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
    New-AzureStorageContainer -Name $defaultBlobContainerName -Context $defaultStorageAccountContext

    # Create the HDInsight cluster
    $pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

    New-AzureRmHDInsightCluster `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $HDInsightClusterName `
        -Location $location `
        -ClusterType Hadoop `
        -OSType Windows `
        -ClusterSizeInNodes 2 `
        -HttpCredential $httpCredential `
        -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -DefaultStorageContainer $existingDefaultBlobContainerName

    ###########################################
    # Prepare the HiveQL script and source data
    ###########################################

    # Create the temp location
    New-Item -Path $localFolder -ItemType Directory -Force

    # Download the sample file from Azure Blob storage
    $context = New-AzureStorageContext -StorageAccountName "hditutorialdata" -Anonymous
    $blobs = Get-AzureStorageBlob -Container "flightdelay" -Context $context
    #$blobs | Get-AzureStorageBlobContent -Context $context -Destination $localFolder

    # Upload data to default container

    $azcopycmd = "cmd.exe /C '$azcopyPath\azcopy.exe' /S /Source:'$localFolder' /Dest:'https://$defaultStorageAccountName.blob.core.windows.net/$defaultBlobContainerName/tutorials/flightdelays' /DestKey:$defaultStorageAccountKey"

    Invoke-Expression -Command:$azcopycmd

    ###########################################
    # Submit the Hive job
    ###########################################
    Use-AzureRmHDInsightCluster -ClusterName $HDInsightClusterName -HttpCredential $httpCredential
    $response = Invoke-AzureRmHDInsightHiveJob `
                    -Files $hqlScriptFile `
                    -DefaultContainer $defaultBlobContainerName `
                    -DefaultStorageAccountName $defaultStorageAccountName `
                    -DefaultStorageAccountKey $defaultStorageAccountKey `
                    -StatusFolder $jobStatusFolder

    write-Host $response

    ###########################################
    # Submit the Sqoop job
    ###########################################
    $exportDir = "wasb://$defaultBlobContainerName@$defaultStorageAccountName.blob.core.windows.net/tutorials/flightdelays/output"

    $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
                    -Command "export --connect $sqlDatabaseConnectionString --table $sqlDatabaseTableName --export-dir $exportDir --fields-terminated-by \001 "
    $sqoopJob = Start-AzureRmHDInsightJob `
                    -ResourceGroupName $resourceGroupName `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef #-Debug -Verbose

    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $HDInsightClusterName `
        -HttpCredential $httpCredential `
        -WaitTimeoutInSeconds 3600 `
        -Job $sqoopJob.JobId

    Get-AzureRmHDInsightJobOutput `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -DefaultContainer $existingDefaultBlobContainerName `
        -DefaultStorageAccountName $defaultStorageAccountName `
        -DefaultStorageAccountKey $defaultStorageAccountKey `
        -JobId $sqoopJob.JobId `
        -DisplayOutputType StandardError

    ###########################################
    # Delete the cluster
    ###########################################
    Remove-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName
    ```
3. 連接到您的 SQL Database，然後在 AvgDelays 資料表中依城市檢視航班誤點平均值：

    ![HDI.FlightDelays.AvgDelays.Dataset][image-hdi-flightdelays-avgdelays-dataset]

- - -

## <a id="appendix-a"></a>附錄 A - 將航班誤點資料上傳至 Azure Blob 儲存體
上傳資料檔案和 HiveQL 指令碼檔案之前 (請參閱 [附錄 B](#appendix-b)) 需要一些規劃。 作法是在建立 HDInsight 叢集之前儲存資料檔案和 HiveQL 檔案，並執行 Hive 工作。 您有兩個選擇：

* **使用 HDInsight 將使用的相同 Azure 儲存體帳戶，作為預設檔案系統。** 由於 HDInsight 叢集將具有儲存體帳戶存取金鑰，您將不需進行任何額外的變更。
* **使用與 HDInsight 叢集預設檔案系統不同的 Azure 儲存體帳戶。** 如果是這樣，您必須修改 Windows PowerShell 指令碼的建立部分 (可在 [建立 HDInsight 叢集和執行 Hive/Sqoop 工作](#runjob) 中找到)，以將儲存體帳戶納入為額外的儲存體帳戶。 如需指示，請參閱[在 HDInsight 中建立 Hadoop 叢集][hdinsight-provision]。 HDInsight 叢集便會知道儲存體帳戶的存取金鑰。

> [!NOTE]
> 資料檔案的 Blob 儲存體路徑會在 HiveQL 指令碼檔案中硬式編碼。 您必須據以更新。

**下載航班資料**

1. 瀏覽至[創新技術研究管理部運輸統計處][rita-website]。
2. 在此頁面上選取下列值：

    <table border="1">
    <tr><th>名稱</th><th>值</th></tr>
    <tr><td>篩選年份</td><td>2013 </td></tr>
    <tr><td>篩選期間</td><td>一月</td></tr>
    <tr><td>欄位</td><td>*Year*、*FlightDate*、*UniqueCarrier*、*Carrier*、*FlightNum*、*OriginAirportID*、*Origin*、*OriginCityName*、*OriginState*、*DestAirportID*、*Dest*、*DestCityName*、*DestState*、*DepDelayMinutes*、*ArrDelay*、*ArrDelayMinutes*、*CarrierDelay*、*WeatherDelay*、*NASDelay*、*SecurityDelay*、*LateAircraftDelay* (請清除其餘所有欄位)</td></tr>
    </table>

3. 按一下 [下載] 。
4. 將檔案解壓縮至 **C:\Tutorials\FlightDelay\2013Data** 資料夾。 每個檔案皆為 CSV 檔案，大小約為 60 GB。
5. 將檔案重新命名為檔案資料所屬月份的名稱。 例如，包含一月份資料的檔案，應命名為 *January.csv*。
6. 重複步驟 2 到 5，以下載 2013 年 12 個月份的檔案。 至少要有一個檔案，才能執行此教學課程。

**將航班誤點資料上傳至 Azure Blob 儲存體**

1. 準備參數：

    <table border="1">
    <tr><th>變數名稱</th><th>注意</th></tr>
    <tr><td>$storageAccountName</td><td>您要上傳資料的 Azure 儲存體帳戶。</td></tr>
    <tr><td>$blobContainerName</td><td>您要上傳資料的 Blob 容器。</td></tr>
    </table>
    
2. 開啟 Azure PowerShell ISE。
3. 將下列指令碼貼到指令碼窗格中：

    ```powershell
    [CmdletBinding()]
    Param(

        [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure storage account name for creating a new HDInsight cluster. If the account doesn't exist, the script will create one.")]
        [String]$storageAccountName,

        [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure blob container name for creating a new HDInsight cluster. If not specified, the HDInsight cluster name will be used.")]
        [String]$blobContainerName
    )

    #Region - Variables
    $localFolder = "C:\Tutorials\FlightDelay\2013Data"  # The source folder
    $destFolder = "tutorials/flightdelay/2013data"     #The blob name prefix for the files to be uploaded
    #EndRegion

    #Region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Connect-AzureRmAccount}
    #EndRegion

    #Region - Validate user input
    Write-Host "`nValidating the Azure Storage account and the Blob container..." -ForegroundColor Green
    # Validate the Storage account
    if (-not (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}))
    {
        Write-Host "The storage account, $storageAccountName, doesn't exist." -ForegroundColor Red
        exit
    }
    else{
        $resourceGroupName = (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}).ResourceGroupName
    }

    # Validate the container
    $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

    if (-not (Get-AzureStorageContainer -Context $storageContext |Where-Object{$_.Name -eq $blobContainerName}))
    {
        Write-Host "The Blob container, $blobContainerName, doesn't exist" -ForegroundColor Red
        Exit
    }
    #EngRegion

    #Region - Copy the file from local workstation to Azure Blob storage
    if (test-path -Path $localFolder)
    {
        foreach ($item in Get-ChildItem -Path $localFolder){
            $fileName = "$localFolder\$item"
            $blobName = "$destFolder/$item"

            Write-Host "Copying $fileName to $blobName" -ForegroundColor Green

            Set-AzureStorageBlobContent -File $fileName -Container $blobContainerName -Blob $blobName -Context $storageContext
        }
    }
    else
    {
        Write-Host "The source folder on the workstation doesn't exist" -ForegroundColor Red
    }

    # List the uploaded files on HDInsight
    Get-AzureStorageBlob -Container $blobContainerName  -Context $storageContext -Prefix $destFolder
    #EndRegion
    ```
4. 按 **F5** 以執行指令碼。

如果您選擇使用不同的方法來上傳檔案，請確定檔案路徑為 tutorials/flightdelay/data。 存取檔案的語法為：

    wasb://<ContainerName>@<StorageAccountName>.blob.core.windows.net/tutorials/flightdelay/data

tutorials/flightdelay/data 路徑是您在上傳檔案時所建立的虛擬資料夾。 請確認共有 12 個檔案，每個月份各一個。

> [!NOTE]
> 您必須更新 Hive 查詢，以從新位置讀取。
>
> 您必須設定容器存取權限，使其成為公用，或將儲存體帳戶繫結至 HDInsight 叢集。 否則，Hive 查詢字串將無法存取資料檔案。

- - -

## <a id="appendix-b"></a>附錄 B - 建立及上傳 HiveQL 指令碼
使用 Azure PowerShell 可讓您逐一執行多個 HiveQL 陳述式，或將 HiveQL 陳述式封裝到指令碼檔案中。 本節說明如何建立 HiveQL 指令碼，以及使用 Azure PowerShell 將指令碼上傳至 Azure Blob 儲存體。 Hive 要求 HiveQL 指令碼必須儲存在 Azure Blob 儲存體中。

HiveQL 指令碼將執行下列作業：

1. **捨棄 delays_raw 資料表** (若此資料表已存在)。
2. **建立 delays_raw 外部 Hive 資料表** (指向含有航班誤點檔案的 Blob 儲存體位置)。 此查詢會指定欄位將以 "," 分隔，且每一行都會以 "\n" 結尾。 如此，當欄位值含有逗號時，就會產生問題，因為 Hive 無法區分作為欄位分隔符號的逗號，與屬於欄位值的逗號 (在 ORIGIN\_CITY\_NAME 和 DEST\_CITY\_NAME 的欄位值中，就會出現此狀況)。 為解決此問題，查詢會建立 TEMP 資料行來放置不當分割為資料行的資料。
3. **捨棄 delays 資料表**(若此資料表已存在)。
4. **建立 delays 資料表**。 此資料表有助於您在進一步處理之前先清除資料。 此查詢會從 delays_raw 資料表建立新資料表 *delays*。 請注意，TEMP 資料行 (如前所述) 並不會複製，並且會使用 **substring** 函數來移除資料中的引號。
5. **計算天候誤點平均值，並依城市名稱將結果分組。** 此作業也會將結果輸出至 Blob 儲存體。 請注意，查詢將會移除資料中的上標號並將排除 **weather_delay** 值為 null 的資料列。 這是必要動作，因為 Sqoop (稍後使用於本教學課程) 預設不會正常處理這些值。

如需 HiveQL 命令的完整清單，請參閱 [Hive 資料定義語言][hadoop-hiveql]。 每個 HiveQL 命令都必須以分號結尾。

**建立 HiveQL 指令碼檔案**

1. 準備參數：

    <table border="1">
    <tr><th>變數名稱</th><th>注意</th></tr>
    <tr><td>$storageAccountName</td><td>您要上傳 HiveQL 指令碼的 Azure 儲存體帳戶。</td></tr>
    <tr><td>$blobContainerName</td><td>您要上傳 HiveQL 指令碼的 Blob 容器。</td></tr>
    </table>
    
2. 開啟 Azure PowerShell ISE。  

3. 將下列指令碼複製並貼到指令碼窗格中：  

    ```powershell
    [CmdletBinding()]
    Param(

        # Azure Blob storage variables
        [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure storage account name for creating a new HDInsight cluster. If the account doesn't exist, the script will create one.")]
        [String]$storageAccountName,

        [Parameter(Mandatory=$True,
                    HelpMessage="Enter the Azure blob container name for creating a new HDInsight cluster. If not specified, the HDInsight cluster name will be used.")]
        [String]$blobContainerName
    )

    #region - Define variables
    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    # The HiveQL script file is exported as this file before it's uploaded to Blob storage
    $hqlLocalFileName = "e:\tutorials\flightdelay\flightdelays.hql"

    # The HiveQL script file will be uploaded to Blob storage as this blob name
    $hqlBlobName = "tutorials/flightdelay/flightdelays.hql"

    # These two constants are used by the HiveQL script file
    #$srcDataFolder = "tutorials/flightdelay/data"
    $dstDataFolder = "/tutorials/flightdelay/output"
    #endregion

    #Region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Connect-AzureRmAccount}
    #EndRegion

    #Region - Validate user input
    Write-Host "`nValidating the Azure Storage account and the Blob container..." -ForegroundColor Green
    # Validate the Storage account
    if (-not (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}))
    {
        Write-Host "The storage account, $storageAccountName, doesn't exist." -ForegroundColor Red
        exit
    }
    else{
        $resourceGroupName = (Get-AzureRmStorageAccount|Where-Object{$_.StorageAccountName -eq $storageAccountName}).ResourceGroupName
    }

    # Validate the container
    $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

    if (-not (Get-AzureStorageContainer -Context $storageContext |Where-Object{$_.Name -eq $blobContainerName}))
    {
        Write-Host "The Blob container, $blobContainerName, doesn't exist" -ForegroundColor Red
        Exit
    }
    #EngRegion

    #region - Validate the file and file path

    # Check if a file with the same file name already exists on the workstation
    Write-Host "`nvalidating the folder structure on the workstation for saving the HQL script file ..."  -ForegroundColor Green
    if (test-path $hqlLocalFileName){

        $isDelete = Read-Host 'The file, ' $hqlLocalFileName ', exists.  Do you want to overwirte it? (Y/N)'

        if ($isDelete.ToLower() -ne "y")
        {
            Exit
        }
    }

    # Create the folder if it doesn't exist
    $folder = split-path $hqlLocalFileName
    if (-not (test-path $folder))
    {
        Write-Host "`nCreating folder, $folder ..." -ForegroundColor Green

        new-item $folder -ItemType directory
    }
    #end region

    #region - Write the Hive script into a local file
    Write-Host "`nWriting the Hive script into a file on your workstation ..." `
                -ForegroundColor Green

    $hqlDropDelaysRaw = "DROP TABLE delays_raw;"

    $hqlCreateDelaysRaw = "CREATE EXTERNAL TABLE delays_raw (" +
            "YEAR string, " +
            "FL_DATE string, " +
            "UNIQUE_CARRIER string, " +
            "CARRIER string, " +
            "FL_NUM string, " +
            "ORIGIN_AIRPORT_ID string, " +
            "ORIGIN string, " +
            "ORIGIN_CITY_NAME string, " +
            "ORIGIN_CITY_NAME_TEMP string, " +
            "ORIGIN_STATE_ABR string, " +
            "DEST_AIRPORT_ID string, " +
            "DEST string, " +
            "DEST_CITY_NAME string, " +
            "DEST_CITY_NAME_TEMP string, " +
            "DEST_STATE_ABR string, " +
            "DEP_DELAY_NEW float, " +
            "ARR_DELAY_NEW float, " +
            "CARRIER_DELAY float, " +
            "WEATHER_DELAY float, " +
            "NAS_DELAY float, " +
            "SECURITY_DELAY float, " +
            "LATE_AIRCRAFT_DELAY float) " +
        "ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' " +
        "LINES TERMINATED BY '\n' " +
        "STORED AS TEXTFILE " +
        "LOCATION 'wasb://flightdelay@hditutorialdata.blob.core.windows.net/2013Data';"

    $hqlDropDelays = "DROP TABLE delays;"

    $hqlCreateDelays = "CREATE TABLE delays AS " +
        "SELECT YEAR AS year, " +
            "FL_DATE AS flight_date, " +
            "substring(UNIQUE_CARRIER, 2, length(UNIQUE_CARRIER) -1) AS unique_carrier, " +
            "substring(CARRIER, 2, length(CARRIER) -1) AS carrier, " +
            "substring(FL_NUM, 2, length(FL_NUM) -1) AS flight_num, " +
            "ORIGIN_AIRPORT_ID AS origin_airport_id, " +
            "substring(ORIGIN, 2, length(ORIGIN) -1) AS origin_airport_code, " +
            "substring(ORIGIN_CITY_NAME, 2) AS origin_city_name, " +
            "substring(ORIGIN_STATE_ABR, 2, length(ORIGIN_STATE_ABR) -1)  AS origin_state_abr, " +
            "DEST_AIRPORT_ID AS dest_airport_id, " +
            "substring(DEST, 2, length(DEST) -1) AS dest_airport_code, " +
            "substring(DEST_CITY_NAME,2) AS dest_city_name, " +
            "substring(DEST_STATE_ABR, 2, length(DEST_STATE_ABR) -1) AS dest_state_abr, " +
            "DEP_DELAY_NEW AS dep_delay_new, " +
            "ARR_DELAY_NEW AS arr_delay_new, " +
            "CARRIER_DELAY AS carrier_delay, " +
            "WEATHER_DELAY AS weather_delay, " +
            "NAS_DELAY AS nas_delay, " +
            "SECURITY_DELAY AS security_delay, " +
            "LATE_AIRCRAFT_DELAY AS late_aircraft_delay " +
        "FROM delays_raw;"

    $hqlInsertLocal = "INSERT OVERWRITE DIRECTORY '$dstDataFolder' " +
        "SELECT regexp_replace(origin_city_name, '''', ''), " +
            "avg(weather_delay) " +
        "FROM delays " +
        "WHERE weather_delay IS NOT NULL " +
        "GROUP BY origin_city_name;"

    $hqlScript = $hqlDropDelaysRaw + $hqlCreateDelaysRaw + $hqlDropDelays + $hqlCreateDelays + $hqlInsertLocal

    $hqlScript | Out-File $hqlLocalFileName -Encoding ascii -Force
    #endregion

    #region - Upload the Hive script to the default Blob container
    Write-Host "`nUploading the Hive script to the default Blob container ..." -ForegroundColor Green

    # Create a storage context object
    $storageAccountKey = (Get-AzureRmStorageAccountKey -StorageAccountName $storageAccountName -ResourceGroupName $resourceGroupName)[0].Value
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

    # Upload the file from local workstation to Blob storage
    Set-AzureStorageBlobContent -File $hqlLocalFileName -Container $blobContainerName -Blob $hqlBlobName -Context $destContext
    #endregion

    Write-host "`nEnd of the PowerShell script" -ForegroundColor Green
    ```

    以下是指令碼中使用的三個變數：

   * **$hqlLocalFileName** - 指令碼會先將 HiveQL 指令碼檔案儲存在本機，然後才上傳至 Blob 儲存體。 這是檔名。 預設值為 <u>C:\tutorials\flightdelay\flightdelays.hql</u>。
   * **$hqlBlobName** - 這是 Azure Blob 儲存體中使用的 HiveQL 指令碼檔案 Blob 名稱。 預設值為 tutorials/flightdelay/flightdelays.hql。 因為檔案會直接寫入 Azure Blob 儲存體，所以 Blob 名稱開頭「沒有」"/"。 如果您要從 Blob 儲存體存取檔案，則必須在檔名開頭加上 "/"。
   * **$srcDataFolder** 和 **$dstDataFolder** - = "tutorials/flightdelay/data" = "tutorials/flightdelay/output"

- - -
## <a id="appendix-c"></a>附錄 C - 針對 Sqoop 工作輸出準備 Azure SQL Database
**準備 SQL 資料庫 (將這部分與 Sqoop 指令碼合併)**

1. 準備參數：

    <table border="1">
    <tr><th>變數名稱</th><th>注意</th></tr>
    <tr><td>$sqlDatabaseServerName</td><td>Azure SQL Database 伺服器的名稱。 不輸入則會建立新的伺服器。</td></tr>
    <tr><td>$sqlDatabaseUsername</td><td>Azure SQL Database 的登入名稱。 如果 $sqlDatabaseServerName 是現有的伺服器，登入和登入密碼會用來向伺服器驗證。 否則會建立新的伺服器。</td></tr>
    <tr><td>$sqlDatabasePassword</td><td>Azure SQL Database 的登入密碼。</td></tr>
    <tr><td>$sqlDatabaseLocation</td><td>只有在建立新的 Azure 資料庫伺服器時才會使用此值。</td></tr>
    <tr><td>$sqlDatabaseName</td><td>用來建立 Sqoop 工作的 AvgDelays 資料表的 SQL Database。 保留空白會建立名為 HDISqoop 的資料庫。 Sqooop 工作輸出的資料表名稱為 AvgDelays。 </td></tr>
    </table>
    
2. 開啟 Azure PowerShell ISE。

3. 將下列指令碼複製並貼到指令碼窗格中：  

    ```powershell
    [CmdletBinding()]
    Param(

        # Azure Resource group variables
        [Parameter(Mandatory=$True,
                HelpMessage="Enter the Azure resource group name. It will be created if it doesn't exist.")]
        [String]$resourceGroupName,

        # SQL database server variables
        [Parameter(Mandatory=$True,
                HelpMessage="Enter the Azure SQL Database Server Name. It will be created if it doesn't exist.")]
        [String]$sqlDatabaseServer,

        [Parameter(Mandatory=$True,
                HelpMessage="Enter the Azure SQL Database admin user.")]
        [String]$sqlDatabaseLogin,

        [Parameter(Mandatory=$True,
                HelpMessage="Enter the Azure SQL Database admin user password.")]
        [String]$sqlDatabasePassword,

        [Parameter(Mandatory=$True,
                HelpMessage="Enter the region to create the Database in.")]
        [String]$sqlDatabaseLocation,   #For example, West US.

        # SQL database variables
        [Parameter(Mandatory=$True,
                HelpMessage="Enter the database name. It will be created if it doesn't exist.")]
        [String]$sqlDatabaseName # specify the database name if you have one created. Otherwise use "" to have the script create one for you.
    )

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    #region - Constants and variables

    # IP address REST service used for retrieving external IP address and creating firewall rules
    [String]$ipAddressRestService = "http://bot.whatismyipaddress.com"
    [String]$fireWallRuleName = "FlightDelay"

    # SQL database variables
    [String]$sqlDatabaseMaxSizeGB = 10

    #SQL query string for creating AvgDelays table
    [String]$sqlDatabaseTableName = "AvgDelays"
    [String]$sqlCreateAvgDelaysTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
                [origin_city_name] [nvarchar](50) NOT NULL,
                [weather_delay] float,
            CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED
            (
                [origin_city_name] ASC
            )
            )"
    #endregion

    #Region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Connect-AzureRmAccount}
    #EndRegion

    #region - Create and validate Azure resouce group
    try{
        Get-AzureRmResourceGroup -Name $resourceGroupName
    }
    catch{
        New-AzureRmResourceGroup -Name $resourceGroupName -Location $sqlDatabaseLocation
    }

    #EndRegion

    #region - Create and validate Azure SQL database server
    try{
        Get-AzureRmSqlServer -ServerName $sqlDatabaseServer -ResourceGroupName $resourceGroupName}
    catch{
        Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green

        $sqlDatabasePW = ConvertTo-SecureString -String $sqlDatabasePassword -AsPlainText -Force
        $credential = New-Object System.Management.Automation.PSCredential($sqlDatabaseLogin,$sqlDatabasePW)

        $sqlDatabaseServer = (New-AzureRmSqlServer -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -SqlAdministratorCredentials $credential -Location $sqlDatabaseLocation).ServerName
        Write-Host "`tThe new SQL database server name is $sqlDatabaseServer." -ForegroundColor Cyan

        Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
        $workstationIPAddress = Invoke-RestMethod $ipAddressRestService
        New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -FirewallRuleName "$fireWallRuleName-workstation" -StartIpAddress $workstationIPAddress -EndIpAddress $workstationIPAddress

        #To allow other Azure services to access the server add a firewall rule and set both the StartIpAddress and EndIpAddress to 0.0.0.0. Note that this allows Azure traffic from any Azure subscription to access the server.
        New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -FirewallRuleName "$fireWallRuleName-Azureservices" -StartIpAddress "0.0.0.0" -EndIpAddress "0.0.0.0"
    }

    #endregion

    #region - Create and validate Azure SQL database

    try {
        Get-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -DatabaseName $sqlDatabaseName
    }
    catch {
        Write-Host "`nCreating SQL Database, $sqlDatabaseName ..."  -ForegroundColor Green
        New-AzureRMSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlDatabaseServer -DatabaseName $sqlDatabaseName -Edition "Standard" -RequestedServiceObjectiveName "S1"
    }

    #endregion

    #region -  Execute an SQL command to create the AvgDelays table

    Write-Host "`nCreating SQL Database table ..."  -ForegroundColor Green
    $conn = New-Object System.Data.SqlClient.SqlConnection
    $conn.ConnectionString = "Data Source=$sqlDatabaseServer.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
    $conn.open()
    $cmd = New-Object System.Data.SqlClient.SqlCommand
    $cmd.connection = $conn
    $cmd.commandtext = $sqlCreateAvgDelaysTable
    $cmd.executenonquery()

    $conn.close()

    Write-host "`nEnd of the PowerShell script" -ForegroundColor Green
    ```

   > [!NOTE]
   > 指令碼使用具象狀態傳輸 (REST) 服務 (http://bot.whatismyipaddress.com) 來擷取外部 IP 位址。 IP 位址用來建立 SQL Database 伺服器的防火牆規則。

    以下是指令碼中使用的一些常數：

   * **$ipAddressRestService** - 預設值是 http://bot.whatismyipaddress.com。 這是用來取得外部 IP 位址的公用 IP 位址 REST 服務。 想要的話，您可以使用其他服務。 透過此服務所擷取的外部 IP 位址將用來建立 Azure SQL Database 伺服器的防火牆規則，讓您能夠從工作站存取資料庫 (使用 Windows PowerShell 指令碼)。
   * **$fireWallRuleName** - 這是 Azure SQL Database 伺服器的防火牆規則名稱。 預設名稱為 <u>FlightDelay</u>。 想要的話，您可以將它重新命名。
   * **$sqlDatabaseMaxSizeGB** - 只有在建立新的 Azure SQL Database 伺服器時才會使用此值。 預設值為 10GB。 10GB 足夠供本教學課程使用。
   * **$sqlDatabaseName** - 只有在建立新的 Azure SQL Database 時才會使用此值。 預設值為 HDISqoop。 如果將它重新命名，則必須相應地更新 Sqoop Windows PowerShell 指令碼。
4. 按 **F5** 以執行指令碼。
5. 驗證指令碼輸出。 請確定指令碼已成功執行。

## <a id="nextsteps"></a> 後續步驟
現在您已了解如何將檔案上傳至 Azure Blob 儲存體、如何使用 Azure Blob 儲存體中的資料填入 Hive 資料表、如何執行 Hive 查詢，以及如何使用 Sqoop 將資料從 HDFS 匯出至 Azure SQL Database。 若要深入了解，請參閱下列文章：

* [開始使用 HDInsight][hdinsight-get-started]
* [搭配 HDInsight 使用 Hivet][hdinsight-use-hive]
* [搭配 HDInsight 使用 Oozie][hdinsight-use-oozie]
* [搭配 HDInsight 使用 Sqoop][hdinsight-use-sqoop]
* [搭配 HDInsight 使用 Pig][hdinsight-use-pig]
* [開發 HDInsight 的 Java MapReduce 程式][hdinsight-develop-mapreduce]

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[rita-website]: http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236&DB_Short_Name=On-Time
[powershell-install-configure]: /powershell/azureps-cmdlets-docs

[hdinsight-use-oozie]: hdinsight-use-oozie.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-use-sqoop]:hadoop/hdinsight-use-sqoop.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[hadoop-hiveql]: https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL
[hadoop-shell-commands]: http://hadoop.apache.org/docs/r0.18.3/hdfs_shell.html

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx

[image-hdi-flightdelays-avgdelays-dataset]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.AvgDelays.DataSet.png
[img-hdi-flightdelays-run-hive-job-output]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.RunHiveJob.Output.png
[img-hdi-flightdelays-flow]: ./media/hdinsight-analyze-flight-delay-data/HDI.FlightDelays.Flow.png
