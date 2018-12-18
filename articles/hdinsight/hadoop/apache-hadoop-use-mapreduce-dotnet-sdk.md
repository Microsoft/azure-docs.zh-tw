---
title: 使用 HDInsight .NET SDK 提交 MapReduce 工作 - Azure
description: 了解如何使用 HDInsight .NET SDK 將 MapReduce 作業提交至 Azure HDInsight Hadoop。
ms.reviewer: jasonh
services: hdinsight
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: b2a79b73015419dc48a1a731e01ac9c8e1fc27f8
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041495"
---
# <a name="run-mapreduce-jobs-using-hdinsight-net-sdk"></a>使用 HDInsight .NET SDK 執行 MapReduce 作業
[!INCLUDE [mapreduce-selector](../../../includes/hdinsight-selector-use-mapreduce.md)]

了解如何使用 HDInsight .NET SDK 提交 MapReduce 作業。 HDInsight 叢集隨附內含一些 MapReduce 範例的 jar 檔案。 此 jar 檔案是 /example/jars/hadoop-mapreduce-examples.jar。  其中一個範例是 wordcount。 您可開發 C# 主控台應用程式以提交 wordcount 作業。  此作業會讀取 /example/data/gutenberg/davinci.txt 檔案，並將結果輸出至 /example/data/davinciwordcount。  如果您想要重新執行應用程式，您必須清除輸出資料夾。

> [!NOTE]
> 此文章中的步驟必須從 Windows 用戶端執行。 如需搭配 Linux、OS X 或 Unix 用戶端使用 Hive 的資訊，請使用本文頂端顯示的索引標籤選取器。
> 
> 

## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須有下列各項：

* **HDInsight 中的 Hadoop 叢集**。 請參閱[開始在 Hdinsight 中使用 Linux 型 Hadoop](apache-hadoop-linux-tutorial-get-started.md)。
* **Visual Studio 2013/2015/2017**。

## <a name="submit-mapreduce-jobs-using-hdinsight-net-sdk"></a>使用 HDInsight .NET SDK 提交 MapReduce 工作
HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。 

**提交工作**

1. 在 Visual Studio 建立 C# 主控台應用程式。
2. 從 NuGet Package Manager Console 執行下列命令：

    ```   
    Install-Package Microsoft.Azure.Management.HDInsight.Job
    ```
3. 使用下列程式碼：

    ```csharp
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using System.Threading;
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;

    namespace SubmitHDInsightJobDotNet
    {
        class Program
        {
            private static HDInsightJobManagementClient _hdiJobManagementClient;

            private const string existingClusterName = "<Your HDInsight Cluster Name>";
            private const string existingClusterUri = existingClusterName + ".azurehdinsight.net";
            private const string existingClusterUsername = "<Cluster Username>";
            private const string existingClusterPassword = "<Cluster User Password>";

            private const string defaultStorageAccountName = "<Default Storage Account Name>"; //<StorageAccountName>.blob.core.windows.net
            private const string defaultStorageAccountKey = "<Default Storage Account Key>";
            private const string defaultStorageContainerName = "<Default Blob Container Name>";

            private const string sourceFile = "/example/data/gutenberg/davinci.txt";  
            private const string outputFolder = "/example/data/davinciwordcount";

            static void Main(string[] args)
            {
                System.Console.WriteLine("The application is running ...");

                var clusterCredentials = new BasicAuthenticationCloudCredentials { Username = existingClusterUsername, Password = existingClusterPassword };
                _hdiJobManagementClient = new HDInsightJobManagementClient(existingClusterUri, clusterCredentials);

                SubmitMRJob();

                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }

            private static void SubmitMRJob()
            {
                List<string> args = new List<string> { { "/example/data/gutenberg/davinci.txt" }, { "/example/data/davinciwordcount" } };

                var paras = new MapReduceJobSubmissionParameters
                {
                    JarFile = @"/example/jars/hadoop-mapreduce-examples.jar",
                    JarClass = "wordcount",
                    Arguments = args
                };

                System.Console.WriteLine("Submitting the MR job to the cluster...");
                var jobResponse = _hdiJobManagementClient.JobManagement.SubmitMapReduceJob(paras);
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
                System.Console.WriteLine("Job output is: ");
                var storageAccess = new AzureStorageAccess(defaultStorageAccountName, defaultStorageAccountKey,
                    defaultStorageContainerName);
    
                if (jobDetail.ExitValue == 0)
                {
                    // Create the storage account object
                    CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=" + 
                        defaultStorageAccountName + 
                        ";AccountKey=" + defaultStorageAccountKey);
    
                    // Create the blob client.
                    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
    
                    // Retrieve reference to a previously created container.
                    CloudBlobContainer container = blobClient.GetContainerReference(defaultStorageContainerName);
    
                    CloudBlockBlob blockBlob = container.GetBlockBlobReference(outputFolder.Substring(1) + "/part-r-00000");
    
                    using (var stream = blockBlob.OpenRead())
                    {
                        using (StreamReader reader = new StreamReader(stream))
                        {
                            while (!reader.EndOfStream)
                            {
                                System.Console.WriteLine(reader.ReadLine());
                            }
                        }
                    }
                }
                else
                {
                    // fetch stderr output in case of failure
                    var output = _hdiJobManagementClient.JobManagement.GetJobErrorLogs(jobId, storageAccess); 
    
                    using (var reader = new StreamReader(output, Encoding.UTF8))
                    {
                        string value = reader.ReadToEnd();
                        System.Console.WriteLine(value);
                    }
    
                }
            }
        }
    }
    ```

4. 按 **F5** 鍵執行應用程式。

若要再次執行作業，您必須變更範例中的作業輸出資料夾名稱，在此範例中，它是 "/example/data/davinciwordcount"。

作業順利完成時，應用程式會顯示檔案 "part-r-00000" 的內容。

## <a name="next-steps"></a>後續步驟
在本文中，您學到幾種建立 HDInsight 叢集的方法。 若要深入了解，請參閱下列文章：

* 對於提交 Hive 作業，請參閱[使用 HDInsight.NET SDK 執行 Hive 查詢](apache-hadoop-use-hive-dotnet-sdk.md)。
* 如需建立 HDInsight 叢集，請參閱[在 HDInsight 中建立 Linux 型 Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。
* 如需管理 HDInsight 叢集，請參閱[在 HDInsight 中管理 Hadoop 叢集](../hdinsight-administer-use-portal-linux.md)。
* 如需了解 HDInsight .NET SDK，請參閱 [HDInsight .NET SDK 參考](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight)。
* 若要向 Azure 進行非互動式驗證，請參閱[建立非互動式驗證 .NET HDInsight 應用程式](../hdinsight-create-non-interactive-authentication-dotnet-applications.md)。

