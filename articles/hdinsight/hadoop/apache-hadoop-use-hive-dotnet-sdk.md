---
title: 使用 HDInsight .NET SDK 執行 Apache Hive 查詢 - Azure
description: 了解如何使用 HDInsight .NET SDK 將 Apache Hadoop 作業提交至 Azure HDInsight Apache Hadoop。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/24/2019
ms.openlocfilehash: 17264fc2a7d0a51de545e329ccf12ad45ad49d46
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98927247"
---
# <a name="run-apache-hive-queries-using-hdinsight-net-sdk"></a>使用 HDInsight .NET SDK 執行 Apache Hive 查詢

[!INCLUDE [hive-selector](../../../includes/hdinsight-selector-use-hive.md)]

了解如何使用 HDInsight .NET SDK 提交 Apache Hive 查詢。 您要撰寫 C# 程式來提交用於列出 Hive 資料表的 Hive 查詢，並顯示結果。

> [!NOTE]  
> 此文章中的步驟必須從 Windows 用戶端執行。 如需搭配 Linux、OS X 或 Unix 用戶端使用 Hive 的資訊，請使用本文頂端顯示的索引標籤選取器。

## <a name="prerequisites"></a>必要條件

開始閱讀本文之前，您必須有下列各項：

* HDInsight 中的 Apache Hadoop 叢集。 請參閱[開始在 Hdinsight 中使用 Linux 型 Hadoop](apache-hadoop-linux-tutorial-get-started.md)。

    > [!IMPORTANT]  
    > 2017 年 9 月 15 起，HDInsight.NET SDK 僅支援從 Microsoft Azure 儲存體帳戶傳回的 Hive 查詢結果。 如果您使用此範例，並搭配將 Azure Data Lake Store 用作主要儲存體的 HDInsight 叢集使用，您無法擷取使用 .NET SDK 的搜尋結果。

* [Visual Studio](https://visualstudio.microsoft.com/vs/community/) 2013 以上。 至少應安裝工作負載 **.net 桌面開發** 。

## <a name="run-a-hive-query"></a>執行 Hive 查詢

HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。

1. 在 Visual Studio 建立 C# 主控台應用程式。

1. 從 NuGet Package Manager 主控台執行下列命令：

    ```console
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```

1. 編輯下列程式碼，以初始化變數的值： `ExistingClusterName, ExistingClusterUsername, ExistingClusterPassword,DefaultStorageAccountName,DefaultStorageAccountKey,DefaultStorageContainerName` 。 然後使用修訂過的程式碼，做為 Visual Studio 中 **Program.cs** 的整個內容。

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
            private const string ExistingClusterUsername = "<Cluster Username>";
            private const string ExistingClusterPassword = "<Cluster User Password>";

            // Only Azure Storage accounts are supported by the SDK
            private const string DefaultStorageAccountName = "<Default Storage Account Name>";
            private const string DefaultStorageAccountKey = "<Default Storage Account Key>";
            private const string DefaultStorageContainerName = "<Default Blob Container Name>";

            private const string ExistingClusterUri = ExistingClusterName + ".azurehdinsight.net";

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

1. 按 **F5** 執行應用程式。

應用程式的輸出應該類似于：

![HDInsight Hadoop Hive 作業輸出](./media/apache-hadoop-use-hive-dotnet-sdk/hdinsight-hadoop-use-hive-net-sdk-output.png)

## <a name="next-steps"></a>後續步驟

在本文中，您已瞭解如何使用 HDInsight .NET SDK 提交 Apache Hive 查詢。 如需詳細資訊，請參閱下列文章：

* [開始使用 Azure HDInsight](apache-hadoop-linux-tutorial-get-started.md)
* [在 HDInsight 中建立 Apache Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)
* [HDInsight .NET SDK 參考](/dotnet/api/overview/azure/hdinsight)
* [搭配 HDInsight 使用 Apache Sqoop](apache-hadoop-use-sqoop-mac-linux.md)
* [建立非互動式驗證 .NET HDInsight 應用程式](../hdinsight-create-non-interactive-authentication-dotnet-applications.md)
