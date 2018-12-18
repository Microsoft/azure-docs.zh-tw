---
title: 使用 .NET SDK for Hadoop 執行 Apache Pig 工作 - Azure HDInsight
description: 了解如何使用 .NET SDK for Hadoop 將 Pig 工作提交至 HDInsight 上的 Hadoop。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: jasonh
ms.openlocfilehash: 06d2633556e149f48c9750b5df0408b601bf3da3
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046755"
---
# <a name="run-pig-jobs-using-the-net-sdk-for-hadoop-in-hdinsight"></a>在 HDInsight 中使用 .NET SDK for Hadoop 執行 Pig 工作

[!INCLUDE [pig-selector](../../../includes/hdinsight-selector-use-pig.md)]

了解如何使用 .NET SDK for Hadoop 將 Apache Pig 工作提交至 Azure HDInsight 上的 Hadoop。

HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。 Pig 可讓您透過建立一系列資料轉換的模型，來建立 MapReduce 作業。 在本文件中，您會學習如何使用基本 C# 應用程式將 Pig 作業提交至 HDInsight 叢集。

## <a name="prerequisites"></a>必要條件

若要完成這篇文章中的步驟，您需要下列項目。

* Azure HDInsight (HDInsight 上的 Hadoop) 叢集 (Windows 或 Linux 型)。

  > [!IMPORTANT]
  > Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。

* Visual Studio 2012、2013、2015 或 2017。

## <a name="create-the-application"></a>建立應用程式

HDInsight .NET SDK 提供 .NET 用戶端程式庫，讓您輕鬆地從 .NET 使用 HDInsight 叢集。

1. 從 Visual Studio 的 [檔案] 功能表中，選取 [新增]，然後選取 [專案]。

2. 對於新的專案，輸入或選取下列值：

   | 屬性 | 值 |
   | ------ | ------ |
   | 類別 | 範本/Visual C#/Windows |
   | 範本 | 主控台應用程式 |
   | 名稱 | SubmitPigJob |

3. 按一下 [確定]  以建立專案。

4. 從 [工具] 功能表中，選取 [程式庫封裝管理員] 或 [NuGet 封裝管理員]，然後選取 [封裝管理員主控台]。

5. 若要安裝 .NET SDK 封裝，請使用下列命令︰

        Install-Package Microsoft.Azure.Management.HDInsight.Job

6. 在 [方案總管] 中，按兩下 **Program.cs** 加以開啟。 將現有程式碼取代為下者。

    ```csharp
    using Microsoft.Azure.Management.HDInsight.Job;
    using Microsoft.Azure.Management.HDInsight.Job.Models;
    using Hyak.Common;

    namespace SubmitPigJob
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

                SubmitPigJob();

                System.Console.WriteLine("Press ENTER to continue ...");
                System.Console.ReadLine();
            }

            private static void SubmitPigJob()
            {
                var parameters = new PigJobSubmissionParameters
                {
                    Query = @"LOGS = LOAD '/example/data/sample.log';
                                LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
                                FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
                                GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
                                FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
                                RESULT = order FREQUENCIES by COUNT desc;
                                DUMP RESULT;"
                };

                System.Console.WriteLine("Submitting the Pig job to the cluster...");
                var response = _hdiJobManagementClient.JobManagement.SubmitPigJob(parameters);
                System.Console.WriteLine("Validating that the response is as expected...");
                System.Console.WriteLine("Response status code is " + response.StatusCode);
                System.Console.WriteLine("Validating the response object...");
                System.Console.WriteLine("JobId is " + response.JobSubmissionJsonResponse.Id);
            }
        }
    }
    ```

7. 若要啟動應用程式，請按 **F5**。

8. 若要結束應用程式，請按 **ENTER**。

## <a name="next-steps"></a>後續步驟

如需 HDInsight 中 Pig 的詳細資訊，請參閱[搭配使用 Pig 與 HDInsight 上的 Hadoop](hdinsight-use-pig.md)。

如需在 HDInsight 上使用 Hadoop 的詳細資訊，請參閱下列文件：

* [搭配使用 Hive 與 HDInsight 上的 Hadoop](hdinsight-use-hive.md)
* [搭配使用 MapReduce 與 HDInsight 上的 Hadoop](hdinsight-use-mapreduce.md)

[preview-portal]: https://portal.azure.com/
