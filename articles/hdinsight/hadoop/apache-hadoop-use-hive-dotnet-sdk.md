---
title: 使用 HDInsight .NET SDK 執行 Hive 查詢 - Azure
description: 了解如何使用 HDInsight .NET SDK 將 Hadoop 工作提交至 Azure HDInsight Hadoop。
ms.reviewer: jasonh
services: hdinsight
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: ac02e67791cde4d67f126da46c86896fb80ee1b9
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43049911"
---
# <a name="run-hive-queries-using-hdinsight-net-sdk"></a>使用 HDInsight .NET SDK 執行 Hive 查詢
[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 HDInsight .NET SDK 提交 Hive 查詢。 您要撰寫 C# 程式來提交用於列出 Hive 資料表的 Hive 查詢，並顯示結果。

> [!NOTE]
> 此文章中的步驟必須從 Windows 用戶端執行。 如需搭配 Linux、OS X 或 Unix 用戶端使用 Hive 的資訊，請使用本文頂端顯示的索引標籤選取器。

## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須有下列各項：

* **HDInsight 中的 Hadoop 叢集**。 請參閱[開始在 Hdinsight 中使用 Linux 型 Hadoop](apache-hadoop-linux-tutorial-get-started.md)。

    > [!WARNING]
    > 2017 年 9 月 15 起，HDInsight.NET SDK 僅支援從 Microsoft Azure 儲存體帳戶傳回的 Hive 查詢結果。 如果您使用此範例，並搭配將 Azure Data Lake Store 用作主要儲存體的 HDInsight 叢集使用，您無法擷取使用 .NET SDK 的搜尋結果。

* **Visual Studio 2013/2015/2017**。

## <a name="run-a-hive-query"></a>執行 Hive 查詢
HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。 

**提交工作**

1. 在 Visual Studio 建立 C# 主控台應用程式。
2. 從 NuGet Package Manager 主控台執行下列命令：
   
        Install-Package Microsoft.Azure.Management.HDInsight.Job
3. 使用下列程式碼：

    ```csharp
        using System.Collections.Generic;
        using System.IO;
        using System.Text;
        using System.Threading;
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
                
                // Only Azure Storage accounts are supported by the SDK
                private const string DefaultStorageAccountName = "<Default Storage Account Name>";
                private const string DefaultStorageAccountKey = "<Default Storage Account Key>";
                private const string DefaultStorageContainerName = "<Default Blob Container Name>";
   
                static void Main(string[] args)
                {
                    System.Console.WriteLine("The application is running ...");
   
                    var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = ExistingClusterUsername, Password = ExistingClusterPassword };
                    _hdiJobManagementClient = new HDInsightJobManagementClient(ExistingClusterUri, clusterCredentials);
   
                    SubmitHiveJob();
   
                    System.Console.WriteLine("Press ENTER to continue ...");
                    System.Console.ReadLine();
                }
   
                private static void SubmitHiveJob()
                {
                    Dictionary<string, string> defines = new Dictionary<string, string> { { "hive.execution.engine", "tez" }, { "hive.exec.reducers.max", "1" } };
                    List<string> args = new List<string> { { "argA" }, { "argB" } };
                    var parameters = new HiveJobSubmissionParameters
                    {
                        Query = "SHOW TABLES",
                        Defines = defines,
                        Arguments = args
                    };
   
                    System.Console.WriteLine("Submitting the Hive job to the cluster...");
                    var jobResponse = _hdiJobManagementClient.JobManagement.SubmitHiveJob(parameters);
                    var jobId = jobResponse.JobSubmissionJsonResponse.Id;
                    System.Console.WriteLine("Response status code is " + jobResponse.StatusCode);
                    System.Console.WriteLine("JobId is " + jobId);
   
                    System.Console.WriteLine("Waiting for the job completion ...");
   
                    // Wait for job completion
                    var jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                    while (!jobDetail.Status.JobComplete)
                    {
                        Thread.Sleep(1000);
                        jobDetail = _hdiJobManagementClient.JobManagement.GetJob(jobId).JobDetail;
                    }
   
                    // Get job output
                    var storageAccess = new AzureStorageAccess(DefaultStorageAccountName, DefaultStorageAccountKey,
                        DefaultStorageContainerName);
                    var output = (jobDetail.ExitValue == 0)
                        ? _hdiJobManagementClient.JobManagement.GetJobOutput(jobId, storageAccess) // fetch stdout output in case of success
                        : _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess); // fetch stderr output in case of failure
   
                    System.Console.WriteLine("Job output is: ");
   
                    using (var reader = new StreamReader(output, Encoding.UTF8))
                    {
                        string value = reader.ReadToEnd();
                        System.Console.WriteLine(value);
                    }
                }
            }
        }
    ```
4. 按 **F5** 鍵執行應用程式。

應用程式的輸出應該類似這樣：

![HDInsight Hadoop Hive 作業輸出](./media/apache-hadoop-use-hive-dotnet-sdk/hdinsight-hadoop-use-hive-net-sdk-output.png)

## <a name="next-steps"></a>後續步驟
在本文中，您學到幾種建立 HDInsight 叢集的方法。 若要深入了解，請參閱下列文章：

* [開始使用 Azure HDInsight](apache-hadoop-linux-tutorial-get-started.md)
* [在 HDInsight 中建立 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)
* [使用 Azure 入口網站管理 HDInsight 中的 Hadoop 叢集](../hdinsight-administer-use-management-portal.md)
* [HDInsight .NET SDK 參考](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [搭配 HDInsight 使用 Sqoop](apache-hadoop-use-sqoop-mac-linux.md)
* [建立非互動式驗證 .NET HDInsight 應用程式](../hdinsight-create-non-interactive-authentication-dotnet-applications.md)
 


