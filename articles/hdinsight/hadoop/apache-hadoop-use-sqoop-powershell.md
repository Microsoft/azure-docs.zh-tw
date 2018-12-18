---
title: 使用 PowerShell 和 Azure HDInsight 執行 Sqoop 作業
description: 了解如何從工作站使用 Azure PowerShell，在 Hadoop 叢集與 Azure SQL Database 之間執行 Sqoop 匯入和匯出。
ms.reviewer: jasonh
services: hdinsight
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 71edb2fb9c3b02ac62670bbfd54adef096cb36e6
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041197"
---
# <a name="run-sqoop-jobs-by-using-azure-powershell-for-hadoop-in-hdinsight"></a>在 HDInsight 中使用 Azure PowerShell for Hadoop 執行 Sqoop 工作
[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

了解如何在 Azure HDInsight 上使用 Azure PowerShell 執行 Sqoop 工作，以進行 HDInsight 叢集與 Azure SQL Database 或 SQL Server Database 之間的匯入和匯出作業。

> [!NOTE]
> 雖然本文中的程序可與以 Windows 為基礎或以 Linux 為基礎的 HDInsight 叢集搭配使用，不過，這些程序只能從 Windows 用戶端運作。 若要選擇其他方法，您可使用本文頂端的索引標籤選取器。 
> 
> 

### <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列項目：

* 具有 Azure PowerShell 的工作站。
* HDInsight 中的 Hadoop 叢集。 如需詳細資訊，請參閱[建立叢集和 SQL 資料庫](hdinsight-use-sqoop.md#create-cluster-and-sql-database)。

## <a name="run-sqoop-by-using-powershell"></a>使用 PowerShell 執行 Sqoop
下列 PowerShell 指令碼會前置處理來源檔案，然後將它匯出至 Azure SQL Database：

    $resourceGroupName = "<AzureResourceGroupName>"
    $hdinsightClusterName = "<HDInsightClusterName>"

    $httpUserName = "admin"
    $httpPassword = "<Password>"

    $defaultStorageAccountName = $hdinsightClusterName + "store"
    $defaultBlobContainerName = $hdinsightClusterName


    $sqlDatabaseServerName = $hdinsightClusterName + "dbserver"
    $sqlDatabaseName = $hdinsightClusterName + "db"
    $sqlDatabaseLogin = "sqluser"
    $sqlDatabasePassword = "<Password>"

    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{Connect-AzureRmAccount}
    #endregion

    #region - pre-process the source file

    Write-Host "`nPreprocessing the source file ..." -ForegroundColor Green

    # This procedure creates a new file with $destBlobName
    $sourceBlobName = "example/data/sample.log"
    $destBlobName = "tutorials/usesqoop/data/sample.log"

    # Define the connection string
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $storageConnectionString = "DefaultEndpointsProtocol=https;AccountName=$defaultStorageAccountName;AccountKey=$defaultStorageAccountKey"

    # Create block blob objects referencing the source and destination blob.
    $storageAccount = Get-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName -Name $defaultStorageAccountName
    $storageContainer = ($storageAccount |Get-AzureStorageContainer -Name $defaultBlobContainerName).CloudBlobContainer
    $sourceBlob = $storageContainer.GetBlockBlobReference($sourceBlobName)
    $destBlob = $storageContainer.GetBlockBlobReference($destBlobName)

    # Define a MemoryStream and a StreamReader for reading from the source file
    $stream = New-Object System.IO.MemoryStream
    $stream = $sourceBlob.OpenRead()
    $sReader = New-Object System.IO.StreamReader($stream)

    # Define a MemoryStream and a StreamWriter for writing into the destination file
    $memStream = New-Object System.IO.MemoryStream
    $writeStream = New-Object System.IO.StreamWriter $memStream

    # Pre-process the source blob
    $exString = "java.lang.Exception:"
    while(-Not $sReader.EndOfStream){
        $line = $sReader.ReadLine()
        $split = $line.Split(" ")

        # remove the "java.lang.Exception" from the first element of the array
        # for example: java.lang.Exception: 2012-02-03 19:11:02 SampleClass8 [WARN] problem finding id 153454612
        if ($split[0] -eq $exString){
            #create a new ArrayList to remove $split[0]
            $newArray = [System.Collections.ArrayList] $split
            $newArray.Remove($exString)

            # update $split and $line
            $split = $newArray
            $line = $newArray -join(" ")
        }

        # remove the lines that has less than 7 elements
        if ($split.count -ge 7){
            write-host $line
            $writeStream.WriteLine($line)
        }
    }

    # Write to the destination blob
    $writeStream.Flush()
    $memStream.Seek(0, "Begin")
    $destBlob.UploadFromStream($memStream)

    #endregion

    #region - export the log file from the cluster to the SQL database

    Write-Host "Exporting the log file ..." -ForegroundColor Green

    $pw = ConvertTo-SecureString -String $httpPassword -AsPlainText -Force
    $httpCredential = New-Object System.Management.Automation.PSCredential($httpUserName,$pw)

    # Connection string for Azure SQL Database.
    # Comment if using SQL Server
    $connectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.windows.net;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"
    # Connection string for SQL Server.
    # Uncomment if using SQL Server.
    #$connectionString = "jdbc:sqlserver://$sqlDatabaseServerName;user=$sqlDatabaseLogin;password=$sqlDatabasePassword;database=$sqlDatabaseName"

    $tableName_log4j = "log4jlogs"
    $exportDir_log4j = "/tutorials/usesqoop/data"
    $sqljdbcdriver = "/user/oozie/share/lib/sqoop/sqljdbc41.jar"

    # Submit a Sqoop job
    $sqoopDef = New-AzureRmHDInsightSqoopJobDefinition `
        -Command "export --connect $connectionString --table $tableName_log4j --export-dir $exportDir_log4j --input-fields-terminated-by \0x20 -m 1" `
        -Files $sqljdbcdriver

    $sqoopJob = Start-AzureRmHDInsightJob `
                    -ClusterName $hdinsightClusterName `
                    -HttpCredential $httpCredential `
                    -JobDefinition $sqoopDef #-Debug -Verbose

    Wait-AzureRmHDInsightJob `
        -ResourceGroupName $resourceGroupName `
        -ClusterName $hdinsightClusterName `
        -HttpCredential $httpCredential `
        -JobId $sqoopJob.JobId

    Write-Host "Standard Error" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardError
    Write-Host "Standard Output" -BackgroundColor Green
    Get-AzureRmHDInsightJobOutput -ResourceGroupName $resourceGroupName -ClusterName $hdinsightClusterName -DefaultStorageAccountName $defaultStorageAccountName -DefaultStorageAccountKey $defaultStorageAccountKey -DefaultContainer $defaultBlobContainerName -HttpCredential $httpCredential -JobId $sqoopJob.JobId -DisplayOutputType StandardOutput
    #endregion

## <a name="limitations"></a>限制
以 Linux 為基礎的 HDInsight 存在下列限制：

* 大量匯出：用來將資料匯出至 Microsoft SQL Server 或 Azure SQL Database 的 Sqoop 連接器目前不支援大量插入。

* 批次處理：執行插入時若使用 `-batch` 參數，Sqoop 將會執行多個插入，而不是將插入作業批次處理。 

## <a name="next-steps"></a>後續步驟
現在，您已了解如何使用 Sqoop。 若要深入了解，請參閱：

* [搭配 HDInsight 使用 Oozie](../hdinsight-use-oozie.md)：在 Oozie 工作流程中使用 Sqoop 動作。
* [使用 HDInsight 分析航班延誤資料](../hdinsight-analyze-flight-delay-data.md)：使用 Hive 分析航班誤點資料，然後使用 Sqoop 將資料匯出至 Azure SQL Database。
* [將資料上傳至 HDInsight](../hdinsight-upload-data.md)：尋找其他方法將資料上傳至 HDInsight 或 Azure Blob 儲存體。

[sqoop-user-guide-1.4.4]: https://sqoop.apache.org/docs/1.4.4/SqoopUserGuide.html
