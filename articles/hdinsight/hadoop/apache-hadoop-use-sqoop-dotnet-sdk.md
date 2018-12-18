---
title: 使用 .NET 和 HDInsight 執行 Sqoop 作業 - Azure
description: 了解如何在 Hadoop 叢集與 Azure SQL Database 之間使用 HDInsight .NET SDK 執行 Sqoop 匯入和匯出。
keywords: Sqoop 作業
ms.reviewer: jasonh
services: hdinsight
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: ffcc2bff17fbe07c58df032dbb8994edb04ae5d0
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046772"
---
# <a name="run-sqoop-jobs-by-using-net-sdk-for-hadoop-in-hdinsight"></a>在 HDInsight 中使用 .NET SDK for Hadoop 執行 Sqoop 作業
[!INCLUDE [sqoop-selector](../../../includes/hdinsight-selector-use-sqoop.md)]

了解如何在 HDInsight 上使用 Azure HDInsight .NET SDK 執行 Sqoop 作業，以進行 HDInsight 叢集與 Azure SQL Database 或 SQL Server Database 之間的匯入和匯出作業。

> [!NOTE]
> 雖然本文中的程序可與以 Windows 為基礎或以 Linux 為基礎的 HDInsight 叢集搭配使用，不過，這些程序只能從 Windows 用戶端運作。 若要選擇其他方法，您可使用本文頂端的索引標籤選取器。
> 

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備下列項目：

* HDInsight 中的 Hadoop 叢集。 如需詳細資訊，請參閱[建立叢集和 SQL 資料庫](hdinsight-use-sqoop.md#create-cluster-and-sql-database)。

## <a name="use-sqoop-on-hdinsight-clusters-with-the-net-sdk"></a>使用 .NET SDK 在 HDInsight 叢集上使用 Sqoop
HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。 在本節中，您會建立 C# 主控台應用程式，將 hivesampletable 匯出至您稍早在本教學課程中建立的 Azure SQL Database 資料表。

## <a name="submit-a-sqoop-job"></a>提交 Sqoop 作業

1. 在 Visual Studio 建立 C# 主控台應用程式。

2. 從 Visual Studio Package Manager 主控台執行下列 NuGet 命令以匯入套件：
   
        Install-Package Microsoft.Azure.Management.HDInsight.Job

3. 在 Program.cs 檔案中使用下列程式碼：
   
        using System.Collections.Generic;
        using Microsoft.Azure.Management.HDInsight.Job;
        using Microsoft.Azure.Management.HDInsight.Job.Models;
        using Hyak.Common;
   
        namespace SubmitHDInsightJobDotNet
        {
            class Program
            {
                private static HDInsightJobManagementClient _hdiJobManagementClient;
   
                private const string ExistingClusterName = "<Your HDInsight Cluster Name>";
                private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";
                private const string ExistingClusterUsername = "<Cluster Username>";
                private const string ExistingClusterPassword = "<Cluster User Password>";
   
                static void Main(string[] args)
                {
                    System.Console.WriteLine("The application is running ...");
   
                    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
   
                    SubmitSqoopJob();
   
                    System.Console.WriteLine("Press ENTER to continue ...");
                    System.Console.ReadLine();
                }
   
                private static void SubmitSqoopJob()
                {
                    var sqlDatabaseServerName = "<SQLDatabaseServerName>";
                    var sqlDatabaseLogin = "<SQLDatabaseLogin>";
                    var sqlDatabaseLoginPassword = "<SQLDatabaseLoginPassword>";
                    var sqlDatabaseDatabaseName = "<DatabaseName>";
   
                    var tableName = "<TableName>";
                    var exportDir = "/tutorials/usesqoop/data";
   
                    // Connection string for using Azure SQL Database.
                    // Comment if using SQL Server
                    var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ".database.windows.net;user=" + sqlDatabaseLogin + "@" + sqlDatabaseServerName + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
                    // Connection string for using SQL Server.
                    // Uncomment if using SQL Server
                    //var connectionString = "jdbc:sqlserver://" + sqlDatabaseServerName + ";user=" + sqlDatabaseLogin + ";password=" + sqlDatabaseLoginPassword + ";database=" + sqlDatabaseDatabaseName;
   
                    var parameters = new SqoopJobSubmissionParameters
                    {
                        Files = new List<string> { "/user/oozie/share/lib/sqoop/sqljdbc41.jar" }, // This line is required for Linux-based cluster.
                        Command = "export --connect " + connectionString + " --table " + tableName + "_mobile --export-dir " + exportDir + "_mobile --fields-terminated-by \\t -m 1"
                    };
   
                    System.Console.WriteLine("Submitting the Sqoop job to the cluster...");
                    var response = _hdiJobManagementClient.JobManagement.SubmitSqoopJob(parameters);
                    System.Console.WriteLine("Validating that the response is as expected...");
                    System.Console.WriteLine("Response status code is " + response.StatusCode);
                    System.Console.WriteLine("Validating the response object...");
                    System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
                }
            }
        }

4. 若要執行程式，請選取 **F5** 鍵。 

## <a name="limitations"></a>限制
以 Linux 為基礎的 HDInsight 存在下列限制：

* 大量匯出：用來將資料匯出至 Microsoft SQL Server 或 Azure SQL Database 的 Sqoop 連接器目前不支援大量插入。

* 批次處理：執行插入時若使用 `-batch` 參數，Sqoop 將會執行多個插入，而不是將插入作業批次處理。

## <a name="next-steps"></a>後續步驟
現在，您已了解如何使用 Sqoop。 若要深入了解，請參閱：

* [搭配 HDInsight 使用 Oozie](../hdinsight-use-oozie.md)：在 Oozie 工作流程中使用 Sqoop 動作。
* [使用 HDInsight 分析航班延誤資料](../hdinsight-analyze-flight-delay-data.md)：使用 Hive 分析航班誤點資料，然後使用 Sqoop 將資料匯出至 Azure SQL Database。
* [將資料上傳至 HDInsight](../hdinsight-upload-data.md)：尋找其他方法將資料上傳至 HDInsight 或 Azure Blob 儲存體。

