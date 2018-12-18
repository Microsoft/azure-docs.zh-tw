---
title: 在 HDInsight 上使用 Hadoop Oozie
description: 在 HDInsight 上使用 Hadoop Oozie：一項巨量資料服務。 了解如何定義 Oozie 工作流程，以及提交 Oozie 工作。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.author: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/25/2017
ROBOTS: NOINDEX
ms.openlocfilehash: f5a6616403d7e3fc7944c26bbbcbfa9fbfb4fff5
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050490"
---
# <a name="use-oozie-with-hadoop-to-define-and-run-a-workflow-in-hdinsight"></a>在 HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程
[!INCLUDE [oozie-selector](../../includes/hdinsight-oozie-selector.md)]

了解如何使用 Apache Oozie 定義工作流程，以及在 HDInsight 上執行工作流程。 若要了解 Oozie 協調器，請參閱[搭配 HDInsight 使用以時間為基礎的 Hadoop Oozie 協調器][hdinsight-oozie-coordinator-time]。 若要了解 Azure Data Factory，請參閱 [搭配 Data Factory 使用 Pig 和 Hive][azure-data-factory-pig-hive]。

Apache Oozie 是可管理 Hadoop 工作的工作流程/協調系統。 它可與 Hadoop 堆疊相整合，並支援 Apache MapReduce、Apache Pig、Apache Hive 和 Apache Sqoop 的 Hadoop 工作。 它也可用來排程系統的特定工作，例如 Java 程式或 Shell 指令碼。

依照本教學課程的指示以實作此工作流程需要兩個動作：

![Workflow diagram][img-workflow-diagram]

1. Hive 動作會執行一個 HiveQL 指令碼，以計算每個記錄層級類型在 log4j 檔案中的出現次數。 每個 log4j 檔案各由一列欄位組成，其中包括顯示類型與嚴重性的 [LOG LEVEL] 欄位，例如：
   
        2012-02-03 18:35:34 SampleClass6 [INFO] everything normal for id 577725851
        2012-02-03 18:35:34 SampleClass4 [FATAL] system problem at id 1991281254
        2012-02-03 18:35:34 SampleClass3 [DEBUG] detail for id 1304807656
        ...
   
    Hive 指令碼輸出如下：
   
        [DEBUG] 434
        [ERROR] 3
        [FATAL] 1
        [INFO]  96
        [TRACE] 816
        [WARN]  4
   
    如需 Hive 的詳細資訊，請參閱[在 HDInsight 上使用 Hive][hdinsight-use-hive]。
2. Sqoop 動作會將 HiveQL 的輸出匯出至 Azure SQL Database 中的資料表。 如需 Sqoop 的詳細資訊，請參閱[搭配 HDInsight 使用 Hadoop Sqoop][hdinsight-use-sqoop]。

> [!NOTE]
> 如需 HDInsight 叢集支援的 Oozie 版本，請參閱 [HDInsight 提供的 Hadoop 叢集版本有哪些新功能？][hdinsight-versions]。
> 
> 

### <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列項目：

* **具有 Azure PowerShell 的工作站**。 
  

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]
  

## <a name="define-oozie-workflow-and-the-related-hiveql-script"></a>定義 Oozie 工作流程和相關的 HiveQL 指令碼
Oozie 工作流程定義會以 hPDL 撰寫 (一種 XML 程序定義語言)。 預設的工作流程檔案名稱為 *workflow.xml*。 以下是您在本教學課程中使用的工作流程檔案。

    <workflow-app name="useooziewf" xmlns="uri:oozie:workflow:0.2">
        <start to = "RunHiveScript"/>

        <action name="RunHiveScript">
            <hive xmlns="uri:oozie:hive-action:0.2">
                <job-tracker>${jobTracker}</job-tracker>
                <name-node>${nameNode}</name-node>
                <configuration>
                    <property>
                        <name>mapred.job.queue.name</name>
                        <value>${queueName}</value>
                    </property>
                </configuration>
                <script>${hiveScript}</script>
                <param>hiveTableName=${hiveTableName}</param>
                <param>hiveDataFolder=${hiveDataFolder}</param>
                <param>hiveOutputFolder=${hiveOutputFolder}</param>
            </hive>
            <ok to="RunSqoopExport"/>
            <error to="fail"/>
        </action>

        <action name="RunSqoopExport">
            <sqoop xmlns="uri:oozie:sqoop-action:0.2">
                <job-tracker>${jobTracker}</job-tracker>
                <name-node>${nameNode}</name-node>
                <configuration>
                    <property>
                        <name>mapred.compress.map.output</name>
                        <value>true</value>
                    </property>
                </configuration>
            <arg>export</arg>
            <arg>--connect</arg>
            <arg>${sqlDatabaseConnectionString}</arg>
            <arg>--table</arg>
            <arg>${sqlDatabaseTableName}</arg>
            <arg>--export-dir</arg>
            <arg>${hiveOutputFolder}</arg>
            <arg>-m</arg>
            <arg>1</arg>
            <arg>--input-fields-terminated-by</arg>
            <arg>"\001"</arg>
            </sqoop>
            <ok to="end"/>
            <error to="fail"/>
        </action>

        <kill name="fail">
            <message>Job failed, error message[${wf:errorMessage(wf:lastErrorNode())}] </message>
        </kill>

        <end name="end"/>
    </workflow-app>

此工作流程中定義了兩個動作。 起始動作為 *RunHiveScript*。 如果此動作順利執行，則下一個動作為 *RunSqoopExport*。

RunHiveScript 有數個變數。 當您使用 Azure PowerShell 從工作站提交 Oozie 工作時，會傳入這些值。

<table border = "1">
<tr><th>工作流程變數</th><th>說明</th></tr>
<tr><td>${jobTracker}</td><td>指定 Hadoop 工作追蹤器的 URL。 在 HDInsight 3.0 和 2.1 版中使用 <strong>jobtrackerhost:9010</strong>。</td></tr>
<tr><td>${nameNode}</td><td>指定 Hadoop 名稱節點的 URL。 使用預設檔案系統位址，例如 <i>wasb://&lt;containerName&gt;@&lt;storageAccountName&gt;.blob.core.windows.net</i>。</td></tr>
<tr><td>${queueName}</td><td>指定要將工作提交至其中的佇列名稱。 使用<strong>預設值</strong>。</td></tr>
</table>

<table border = "1">
<tr><th>Hive 動作變數</th><th>說明</th></tr>
<tr><td>${hiveDataFolder}</td><td>指定 Hive Create Table 命令的來源目錄。</td></tr>
<tr><td>${hiveOutputFolder}</td><td>指定 INSERT OVERWRITE 陳述式的輸出資料夾。</td></tr>
<tr><td>${hiveTableName}</td><td>指定參考 log4j 資料檔案的 Hive 資料表名稱。</td></tr>
</table>

<table border = "1">
<tr><th>Sqoop 動作變數</th><th>說明</th></tr>
<tr><td>${sqlDatabaseConnectionString}</td><td>指定 Azure SQL Database 連接字串。</td></tr>
<tr><td>${sqlDatabaseTableName}</td><td>指定要將資料匯出至其中的 Azure SQL Database 資料表。</td></tr>
<tr><td>${hiveOutputFolder}</td><td>指定 Hive INSERT OVERWRITE 陳述式的輸出資料夾。 這是 Sqoop 匯出 (export-dir) 的相同資料夾。</td></tr>
</table>

如需關於 Oozie 工作流程和使用工作流程動作的詳細資訊，請參閱 [Apache Oozie 4.0 文件][apache-oozie-400] (適用於 HDInsight 3.0 版) 或 [Apache Oozie 3.3.2 文件][apache-oozie-332] (適用於 HDInsight 2.1 版)。

此工作流程中的 Hive 動作會呼叫 HiveQL 指令碼檔案。 此指令碼檔案包含三個 HiveQL 陳述式：

    DROP TABLE ${hiveTableName};
    CREATE EXTERNAL TABLE ${hiveTableName}(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION '${hiveDataFolder}';
    INSERT OVERWRITE DIRECTORY '${hiveOutputFolder}' SELECT t4 AS sev, COUNT(*) AS cnt FROM ${hiveTableName} WHERE t4 LIKE '[%' GROUP BY t4;

1. **DROP TABLE 陳述式** 會刪除 log4j Hive 資料表 (如果存在)。
2. **CREATE TABLE 陳述式** 會建立指向 log4j 記錄檔位置的 log4j Hive 外部資料表。 欄位分隔符號為 ","。 預設的行分隔符號為 "\n"。 Hive 外部資料表可讓您在需要執行 Oozie 工作流程多次時，避免資料檔案從原始位置遭到移除。
3. **INSERT OVERWRITE 陳述式** 可從 log4j Hive 資料表中計算每個記錄層級類型的出現次數，並將輸出儲存至 Azure 儲存體中的 Blob。

指令碼中使用三種變數：

* ${hiveTableName}
* ${hiveDataFolder}
* ${hiveOutputFolder}

工作流程定義檔 (在本教學課程中為 workflow.xml) 會在執行階段將這些值傳遞至此 HiveQL 指令碼。

工作流程檔案和 HiveQL 檔案均會儲存在 Blob 容器中。  您稍後在本教學課程中，會使用 PowerShell 指令碼，將這兩個檔案複製到預設儲存體帳戶中。 

## <a name="submit-oozie-jobs-using-powershell"></a>使用 PowerShell 提交 Oozie 工作
Azure PowerShell 目前並未提供任何用以定義 Oozie 工作的 Cmdlet。 您可以使用 **Invoke-RestMethod** Cmdlet 來叫用 Oozie Web 服務。 Oozie Web 服務 API 是 HTTP REST JSON API。 如需關於 Oozie Web 服務 API 的詳細資訊，請參閱 [Apache Oozie 4.0 文件][apache-oozie-400] (適用於 HDInsight 3.0 版) 或 [Apache Oozie 3.3.2 文件][apache-oozie-332](適用於 HDInsight 2.1 版)。

本節中的 PowerShell 指令碼會執行下列步驟：

1. 連接到 Azure。
2. 建立 Azure 資源群組。 如需詳細資訊，請參閱 [將 Azure PowerShell 與 Azure 資源管理員搭配使用](../powershell-azure-resource-manager.md)。
3. 建立 Azure SQL Database 伺服器、Azure SQL Database 和兩個資料表。 Sqoop 動作會在工作流程中用到它們。
   
    資料表名稱為 *log4jLogCount*。
4. 建立用來執行 Oozie 工作的 HDInsight 叢集。
   
    若要檢查叢集，您可以使用 Azure 入口網站或 Azure PowerShell。
5. 將 Oozie 工作流程檔案和 HiveQL 指令碼檔案複製到預設檔案系統中。
   
    這兩個檔案會儲存在公用 Blob 容器。
   
   * 將 HiveQL 指令碼 (useoozie.hql) 複製到 Azure 儲存體 (wasb:///tutorials/useoozie/useoozie.hql)。
   * 將 workflow.xml 複製到 wasb:///tutorials/useoozie/workflow.xml。
   * 將資料檔案 (/example/data/sample.log) 複製到 wasb:///tutorials/useoozie/data/sample.log。
6. 提交 Oozie 工作。
   
    若要檢查 Oozie 工作的結果，請使用 Visual Studio 或其他工具來連接到 Azure SQL Database。

指令碼如下。  您可以從 Windows PowerShell ISE 執行指令碼。 您只需要設定前 7 個變數。

    #region - provide the following values

    $subscriptionID = "<Enter your Azure subscription ID>"

    # SQL Database server login credentials used for creating and connecting
    $sqlDatabaseLogin = "<Enter SQL Database Login Name>"
    $sqlDatabasePassword = "<Enter SQL Database Login Password>"

    # HDInsight cluster HTTP user credential used for creating and connectin
    $httpUserName = "admin"  # The default name is "admin"
    $httpPassword = "<Enter HDInsight Cluster HTTP User Password>"

    # Used for creating Azure service names
    $nameToken = "<Enter an Alias>"
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")
    #endregion

    #region - variables

    # Resource group variables
    $resourceGroupName = $namePrefix + "rg"
    $location = "East US 2" # used by all Azure services defined in this tutorial

    # SQL database varialbes
    $sqlDatabaseServerName = $namePrefix + "sqldbserver"
    $sqlDatabaseName = $namePrefix + "sqldb"
    $sqlDatabaseConnectionString = "Data Source=$sqlDatabaseServerName.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
    $sqlDatabaseMaxSizeGB = 10

    # Used for retrieving external IP address and creating firewall rules
    $ipAddressRestService = "http://bot.whatismyipaddress.com"
    $fireWallRuleName = "UseSqoop"

    # HDInsight variables
    $hdinsightClusterName = $namePrefix + "hdi"
    $defaultStorageAccountName = $namePrefix + "store"
    $defaultBlobContainerName = $hdinsightClusterName
    #endregion

    # Treat all errors as terminating
    $ErrorActionPreference = "Stop"

    #region - Connect to Azure subscription
    Write-Host "`nConnecting to your Azure subscription ..." -ForegroundColor Green
    try{Get-AzureRmContext}
    catch{
        Connect-AzureRmAccount
        Select-AzureRmSubscription -SubscriptionId $subscriptionID
    }
    #endregion

    #region - Create Azure resouce group
    Write-Host "`nCreating an Azure resource group ..." -ForegroundColor Green
    try{
        Get-AzureRmResourceGroup -Name $resourceGroupName
    }
    catch{
        New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
    }
    #endregion

    #region - Create Azure SQL database server
    Write-Host "`nCreating an Azure SQL Database server ..." -ForegroundColor Green
    try{
        Get-AzureRmSqlServer -ServerName $sqlDatabaseServerName -ResourceGroupName $resourceGroupName}
    catch{
        Write-Host "`nCreating SQL Database server ..."  -ForegroundColor Green

        $sqlDatabasePW = ConvertTo-SecureString -String $sqlDatabasePassword -AsPlainText -Force
        $sqlLoginCredentials = New-Object System.Management.Automation.PSCredential($sqlDatabaseLogin,$sqlDatabasePW)

        $sqlDatabaseServerName = (New-AzureRmSqlServer `
                                    -ResourceGroupName $resourceGroupName `
                                    -ServerName $sqlDatabaseServerName `
                                    -SqlAdministratorCredentials $sqlLoginCredentials `
                                    -Location $location).ServerName
        Write-Host "`tThe new SQL database server name is $sqlDatabaseServerName." -ForegroundColor Cyan

        Write-Host "`nCreating firewall rule, $fireWallRuleName ..." -ForegroundColor Green
        $workstationIPAddress = Invoke-RestMethod $ipAddressRestService
        New-AzureRmSqlServerFirewallRule `
            -ResourceGroupName $resourceGroupName `
            -ServerName $sqlDatabaseServerName `
            -FirewallRuleName "$fireWallRuleName-workstation" `
            -StartIpAddress $workstationIPAddress `
            -EndIpAddress $workstationIPAddress

        #To allow other Azure services to access the server add a firewall rule and set both the StartIpAddress and EndIpAddress to 0.0.0.0. 
        #Note that this allows Azure traffic from any Azure subscription to access the server.
        New-AzureRmSqlServerFirewallRule `
            -ResourceGroupName $resourceGroupName `
            -ServerName $sqlDatabaseServerName `
            -FirewallRuleName "$fireWallRuleName-Azureservices" `
            -StartIpAddress "0.0.0.0" `
            -EndIpAddress "0.0.0.0"
    }
    #endregion

    #region - Create and validate Azure SQL database
    Write-Host "`nCreating SQL Database, $sqlDatabaseName ..."  -ForegroundColor Green

    try {
        Get-AzureRmSqlDatabase `
            -ResourceGroupName $resourceGroupName `
            -ServerName $sqlDatabaseServerName `
            -DatabaseName $sqlDatabaseName
    }
    catch {
        New-AzureRMSqlDatabase `
            -ResourceGroupName $resourceGroupName `
            -ServerName $sqlDatabaseServerName `
            -DatabaseName $sqlDatabaseName `
            -Edition "Standard" `
            -RequestedServiceObjectiveName "S1"
    }
    #endregion

    #region - Create SQL database tables
    Write-Host "Creating the log4jlogs table  ..." -ForegroundColor Green

    $sqlDatabaseTableName = "log4jLogsCount"
    $cmdCreateLog4jCountTable = " CREATE TABLE [dbo].[$sqlDatabaseTableName](
            [Level] [nvarchar](10) NOT NULL,
            [Total] float,
        CONSTRAINT [PK_$sqlDatabaseTableName] PRIMARY KEY CLUSTERED
        (
        [Level] ASC
        )
        )"

    $conn = New-Object System.Data.SqlClient.SqlConnection
    $conn.ConnectionString = $sqlDatabaseConnectionString
    $conn.Open()

    # Create the log4jlogs table and index
    $cmd = New-Object System.Data.SqlClient.SqlCommand
    $cmd.Connection = $conn
    $cmd.CommandText = $cmdCreateLog4jCountTable
    $cmd.ExecuteNonQuery()

    $conn.close()
    #endregion

    #region - Create HDInsight cluster

    Write-Host "Creating the HDInsight cluster and the dependent services ..." -ForegroundColor Green

    # Create the default storage account
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_LRS

    # Create the default Blob container
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext `
                                        -StorageAccountName $defaultStorageAccountName `
                                        -StorageAccountKey $defaultStorageAccountKey 
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageAccountContext 

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
        -DefaultStorageContainer $defaultBlobContainerName 

    # Validate the cluster
    Get-AzureRmHDInsightCluster -ClusterName $hdinsightClusterName
    #endregion

    #region - copy Oozie workflow and HiveQL files

    Write-Host "Copy workflow definition and HiveQL script file ..." -ForegroundColor Green

    # Both files are stored in a public Blob
    $publicBlobContext = New-AzureStorageContext -StorageAccountName "hditutorialdata" -Anonymous

    # WASB folder for storing the Oozie tutorial files.
    $destFolder = "tutorials/useoozie"  # Do NOT use the long path here

    Start-CopyAzureStorageBlob `
        -Context $publicBlobContext `
        -SrcContainer "useoozie" `
        -SrcBlob "useooziewf.hql"  `
        -DestContext $defaultStorageAccountContext `
        -DestContainer $defaultBlobContainerName `
        -DestBlob "$destFolder/useooziewf.hql" `
        -Force

    Start-CopyAzureStorageBlob `
        -Context $publicBlobContext `
        -SrcContainer "useoozie" `
        -SrcBlob "workflow.xml"  `
        -DestContext $defaultStorageAccountContext `
        -DestContainer $defaultBlobContainerName `
        -DestBlob "$destFolder/workflow.xml" `
        -Force

    #validate the copy
    Get-AzureStorageBlob `
        -Context $defaultStorageAccountContext `
        -Container $defaultBlobContainerName `
        -Blob $destFolder/workflow.xml

    Get-AzureStorageBlob `
        -Context $defaultStorageAccountContext `
        -Container $defaultBlobContainerName `
        -Blob $destFolder/useooziewf.hql

    #endregion

    #region - copy the sample.log file

    Write-Host "Make a copy of the sample.log file ... " -ForegroundColor Green

    Start-CopyAzureStorageBlob `
        -Context $defaultStorageAccountContext `
        -SrcContainer $defaultBlobContainerName `
        -SrcBlob "example/data/sample.log"  `
        -DestContext $defaultStorageAccountContext `
        -DestContainer $defaultBlobContainerName `
        -destBlob "$destFolder/data/sample.log" 

    #validate the copy
    Get-AzureStorageBlob `
        -Context $defaultStorageAccountContext `
        -Container $defaultBlobContainerName `
        -Blob $destFolder/data/sample.log

    #endregion

    #region - submit Oozie job

    $storageUri="wasb://$defaultBlobContainerName@$defaultStorageAccountName.blob.core.windows.net"

    $oozieJobName = $namePrefix + "OozieJob"

    #Oozie WF variables
    $oozieWFPath="$storageUri/tutorials/useoozie"  # The default name is workflow.xml. And you don't need to specify the file name.
    $waitTimeBetweenOozieJobStatusCheck=10

    #Hive action variables
    $hiveScript = "$storageUri/tutorials/useoozie/useooziewf.hql"
    $hiveTableName = "log4jlogs"
    $hiveDataFolder = "$storageUri/tutorials/useoozie/data"
    $hiveOutputFolder = "$storageUri/tutorials/useoozie/output"

    #Sqoop action variables
    $sqlDatabaseConnectionString = "jdbc:sqlserver://$sqlDatabaseServerName.database.windows.net;user=$sqlDatabaseLogin@$sqlDatabaseServerName;password=$sqlDatabasePassword;database=$sqlDatabaseName"

    $OoziePayload =  @"
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>

    <property>
        <name>nameNode</name>
        <value>$storageUrI</value>
    </property>

    <property>
        <name>jobTracker</name>
        <value>jobtrackerhost:9010</value>
    </property>

    <property>
        <name>queueName</name>
        <value>default</value>
    </property>

    <property>
        <name>oozie.use.system.libpath</name>
        <value>true</value>
    </property>

    <property>
        <name>hiveScript</name>
        <value>$hiveScript</value>
    </property>

    <property>
        <name>hiveTableName</name>
        <value>$hiveTableName</value>
    </property>

    <property>
        <name>hiveDataFolder</name>
        <value>$hiveDataFolder</value>
    </property>

    <property>
        <name>hiveOutputFolder</name>
        <value>$hiveOutputFolder</value>
    </property>

    <property>
        <name>sqlDatabaseConnectionString</name>
        <value>&quot;$sqlDatabaseConnectionString&quot;</value>
    </property>

    <property>
        <name>sqlDatabaseTableName</name>
        <value>$SQLDatabaseTableName</value>
    </property>

    <property>
        <name>user.name</name>
        <value>$httpUserName</value>
    </property>

    <property>
        <name>oozie.wf.application.path</name>
        <value>$oozieWFPath</value>
    </property>

    </configuration>
    "@

    Write-Host "Checking Oozie server status..." -ForegroundColor Green
    $clusterUriStatus = "https://$hdinsightClusterName.azurehdinsight.net:443/oozie/v2/admin/status"
    $response = Invoke-RestMethod -Method Get -Uri $clusterUriStatus -Credential $httpCredential -OutVariable $OozieServerStatus

    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
    $oozieServerSatus = $jsonResponse[0].("systemMode")
    Write-Host "Oozie server status is $oozieServerSatus."

    # create Oozie job
    Write-Host "Sending the following Payload to the cluster:" -ForegroundColor Green
    Write-Host "`n--------`n$OoziePayload`n--------"
    $clusterUriCreateJob = "https://$hdinsightClusterName.azurehdinsight.net:443/oozie/v2/jobs"
    $response = Invoke-RestMethod -Method Post -Uri $clusterUriCreateJob -Credential $httpCredential -Body $OoziePayload -ContentType "application/xml" -OutVariable $OozieJobName #-debug

    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
    $oozieJobId = $jsonResponse[0].("id")
    Write-Host "Oozie job id is $oozieJobId..."

    # start Oozie job
    Write-Host "Starting the Oozie job $oozieJobId..." -ForegroundColor Green
    $clusterUriStartJob = "https://$hdinsightClusterName.azurehdinsight.net:443/oozie/v2/job/" + $oozieJobId + "?action=start"
    $response = Invoke-RestMethod -Method Put -Uri $clusterUriStartJob -Credential $httpCredential | Format-Table -HideTableHeaders #-debug

    # get job status
    Write-Host "Sleeping for $waitTimeBetweenOozieJobStatusCheck seconds until the job metadata is populated in the Oozie metastore..." -ForegroundColor Green
    Start-Sleep -Seconds $waitTimeBetweenOozieJobStatusCheck

    Write-Host "Getting job status and waiting for the job to complete..." -ForegroundColor Green
    $clusterUriGetJobStatus = "https://$hdinsightClusterName.azurehdinsight.net:443/oozie/v2/job/" + $oozieJobId + "?show=info"
    $response = Invoke-RestMethod -Method Get -Uri $clusterUriGetJobStatus -Credential $httpCredential
    $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
    $JobStatus = $jsonResponse[0].("status")

    while($JobStatus -notmatch "SUCCEEDED|KILLED")
    {
        Write-Host "$(Get-Date -format 'G'): $oozieJobId is in $JobStatus state...waiting $waitTimeBetweenOozieJobStatusCheck seconds for the job to complete..."
        Start-Sleep -Seconds $waitTimeBetweenOozieJobStatusCheck
        $response = Invoke-RestMethod -Method Get -Uri $clusterUriGetJobStatus -Credential $httpCredential
        $jsonResponse = ConvertFrom-Json (ConvertTo-Json -InputObject $response)
        $JobStatus = $jsonResponse[0].("status")
        $jobStatus
    }

    Write-Host "$(Get-Date -format 'G'): $oozieJobId is in $JobStatus state!" -ForegroundColor Green

    #endregion


**重新執行教學課程**

若要重新執行工作流程，您必須刪除以下項目：

* Hive 指令碼輸出檔案
* log4jLogsCount 資料表中的資料

以下提供可供使用的範例 PowerShell 指令碼：

    $resourceGroupName = "<AzureResourceGroupName>"

    $defaultStorageAccountName = "<AzureStorageAccountName>"
    $defaultBlobContainerName = "<ContainerName>"

    #SQL database variables
    $sqlDatabaseServerName = "<SQLDatabaseServerName>"
    $sqlDatabaseLogin = "<SQLDatabaseLoginName>"
    $sqlDatabasePassword = "<SQLDatabaseLoginPassword>"
    $sqlDatabaseName = "<SQLDatabaseName>"
    $sqlDatabaseTableName = "log4jLogsCount"

    Write-host "Delete the Hive script output file ..." -ForegroundColor Green
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                -ResourceGroupName $resourceGroupName `
                                -Name $defaultStorageAccountName)[0].Value
    $destContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey
    Remove-AzureStorageBlob -Context $destContext -Blob "tutorials/useoozie/output/000000_0" -Container $defaultBlobContainerName

    Write-host "Delete all the records from the log4jLogsCount table ..." -ForegroundColor Green
    $conn = New-Object System.Data.SqlClient.SqlConnection
    $conn.ConnectionString = "Data Source=$sqlDatabaseServerName.database.windows.net;Initial Catalog=$sqlDatabaseName;User ID=$sqlDatabaseLogin;Password=$sqlDatabasePassword;Encrypt=true;Trusted_Connection=false;"
    $conn.open()
    $cmd = New-Object System.Data.SqlClient.SqlCommand
    $cmd.connection = $conn
    $cmd.commandtext = "delete from $sqlDatabaseTableName"
    $cmd.executenonquery()

    $conn.close()

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何定義 Oozie 工作流程，以及如何使用 PowerShell 執行 Oozie 工作。 若要深入了解，請參閱下列文章：

* [搭配 HDInsight 使用以時間為基礎的 Hadoop Oozie 協調器][hdinsight-oozie-coordinator-time]
* [開始在 HDInsight 中搭配 Hive 使用 Hadoop 以分析行動電話使用][hdinsight-get-started]
* [搭配 HDInsight 使用 Azure Blob 儲存體][hdinsight-storage]
* [使用 PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [在 HDInsight 中上傳 Hadoop 作業的資料][hdinsight-upload-data]
* [搭配使用 Sqoop 與 HDInsight 中的 Hadoop][hdinsight-use-sqoop]
* [搭配使用 Hive 與 HDInsight 上的 Hadoop][hdinsight-use-hive]
* [搭配使用 Pig 與 HDInsight 上的 Hadoop][hdinsight-use-pig]
* [開發 HDInsight 的 Java MapReduce 程式][hdinsight-develop-mapreduce]

[hdinsight-cmdlets-download]: http://go.microsoft.com/fwlink/?LinkID=325563



[azure-data-factory-pig-hive]: ../data-factory/transform-data.md
[hdinsight-oozie-coordinator-time]: hdinsight-use-oozie-coordinator-time.md
[hdinsight-versions]:  hdinsight-component-versioning.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md


[hdinsight-use-sqoop]:hadoop/hdinsight-use-sqoop.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[hdinsight-develop-mapreduce]:hadoop/apache-hadoop-develop-deploy-java-mapreduce-linux.md

[sqldatabase-create-configue]: ../sql-database-create-configure.md
[sqldatabase-get-started]: ../sql-database-get-started.md

[azure-management-portal]: https://portal.azure.com/
[azure-create-storageaccount]:../storage/common/storage-create-storage-account.md

[apache-hadoop]: http://hadoop.apache.org/
[apache-oozie-400]: http://oozie.apache.org/docs/4.0.0/
[apache-oozie-332]: http://oozie.apache.org/docs/3.3.2/

[powershell-download]: http://azure.microsoft.com/downloads/
[powershell-about-profiles]: http://go.microsoft.com/fwlink/?LinkID=113729
[powershell-install-configure]: /powershell/azureps-cmdlets-docs
[powershell-start]: http://technet.microsoft.com/library/hh847889.aspx
[powershell-script]: https://technet.microsoft.com/library/ee176961.aspx

[cindygross-hive-tables]: http://blogs.msdn.com/b/cindygross/archive/2013/02/06/hdinsight-hive-internal-and-external-tables-intro.aspx

[img-workflow-diagram]: ./media/hdinsight-use-oozie/HDI.UseOozie.Workflow.Diagram.png
[img-preparation-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.Preparation.Output1.png  
[img-runworkflow-output]: ./media/hdinsight-use-oozie/HDI.UseOozie.RunWF.Output.png

[technetwiki-hive-error]: http://social.technet.microsoft.com/wiki/contents/articles/23047.hdinsight-hive-error-unable-to-rename.aspx
