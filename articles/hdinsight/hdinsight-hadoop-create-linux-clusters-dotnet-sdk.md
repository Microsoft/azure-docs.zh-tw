---
title: 使用 .NET 建立 Hadoop 叢集 - Azure HDInsight
description: 了解如何在 Linux 上使用 HDInsight .NET SDK 為 HDInsight 建立 Hadoop、HBase、Storm 或 Spark 叢集。
services: hdinsight
author: mamccrea
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 08/16/2018
ms.author: mamccrea
ms.openlocfilehash: ea5c0d3b9f444601607ebcda8d139797eb8dd2ec
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43093906"
---
# <a name="create-linux-based-clusters-in-hdinsight-using-the-net-sdk"></a>在 HDInsight 中使用 .NET SDK 建立以 Linux 為基礎的叢集

[!INCLUDE [selector](../../includes/hdinsight-create-linux-cluster-selector.md)]


了解如何使用 .NET SDK 在 Azure HDInsight 叢集中建立 Hadoop 叢集。

> [!IMPORTANT]
> 本文中的步驟會建立具有一個背景工作節點的叢集。 如果您在建立叢集時或在建立後調整叢集時規劃有 32 個以上的背景工作節點，您就必須選取具有至少 8 個核心和 14 GB RAM 的前端節點大小。
>
> 如需節點大小和相關成本的詳細資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **Azure 儲存體帳戶**。 請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)。
* **Visual Studio 2013、Visual Studio 2015 或 Visual Studio 2017**。

## <a name="create-clusters"></a>建立叢集

1. 開啟 Visual Studio 2017。
2. 建立新的 Visual C# 主控台應用程式。
3. 在 [工具] 功能表中按一下 [NuGet 套件管理員]，然後按一下 [套件管理員主控台]。
4. 在主控台中執行下列命令，以安裝套件：

    ```powershell
    Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
    Install-Package Microsoft.Azure.Management.ResourceManager -Pre
    Install-Package Microsoft.Azure.Management.HDInsight
    ```

    這些命令會將 .NET 程式庫及其參考新增至目前的 Visual Studio 專案。
5. 在 [方案總管] 中按兩下 **Program.cs** 來開啟該檔案、貼上下列程式碼，然後提供變數的值：

    ```csharp
    using System;
    using Microsoft.Rest;
    using Microsoft.Rest.Azure.Authentication;
    using Microsoft.Azure;
    using Microsoft.Azure.Management.HDInsight;
    using Microsoft.Azure.Management.HDInsight.Models;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    namespace CreateHDInsightCluster
    {
        class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;

            private const string SubscriptionId = "<Your Azure Subscription ID>";
            // Replace with your AAD tenant ID if necessary
            private const string TenantId = UserTokenProvider.CommonTenantId; 
            // This is the GUID for the PowerShell client. Used for interactive logins in this example.
            private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";

            private const string ExistingResourceGroupName = "<Enter Resource Group Name>";
            private const string ExistingStorageName = "<Enter Default Storage Account Name>.blob.core.windows.net";
            private const string ExistingStorageKey = "<Enter Default Storage Account Key>";
            private const string ExistingBlobContainer = "<Enter Default Bob Container Name>";

            private const string NewClusterName = "<Enter HDInsight Cluster Name>";
            private const int NewClusterNumNodes = 2;
            private const string NewClusterLocation = "EAST US 2";     // Must be the same as the default Storage account
            private const OSType NewClusterOSType = OSType.Linux;
            private const string NewClusterType = "Hadoop";
            private const string NewClusterVersion = "3.6";
            private const string NewClusterUsername = "admin";
            private const string NewClusterPassword = "<Enter HTTP User Password>";
            private const string NewClusterSshUserName = "sshuser";

            // You can use eitehr password or public key. See https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix
            private const string NewClusterSshPassword = "<Enter SSH User Password>";
            private const string NewClusterSshPublicKey = @"---- BEGIN SSH2 PUBLIC KEY ----
                Comment: ""rsa-key-20150731""
                AAAAB3NzaC1yc2EAAAABJQAAAQEA4QiCRLqT7fnmUA5OhYWZNlZo6lLaY1c+IRsp
                gmPCsJVGQLu6O1wqcxRqiKk7keYq8bP5s30v6bIljsLZYTnyReNUa5LtFw7eauGr
                yVt3Pve6ejfWELhbVpi0iq8uJNFA9VvRkz8IP1JmjC5jsdnJhzQZtgkIrdn3w0e6
                WVfu15kKyY8YAiynVbdV51EB0SZaSLdMZkZQ81xi4DDtCZD7qvdtWEFwLa+EHdkd
                pzO36Mtev5XvseLQqzXzZ6aVBdlXoppGHXkoGHAMNOtEWRXpAUtEccjpATsaZhQR
                zZdZlzHduhM10ofS4YOYBADt9JohporbQVHM5w6qUhIgyiPo7w==
                ---- END SSH2 PUBLIC KEY ----"; //replace the public key with your own

            static void Main(string[] args)
            {
                System.Console.WriteLine("Creating a cluster.  The process takes 10 to 20 minutes ...");

                // Authenticate and get a token
                var authToken = GetTokenCloudCredentials(TenantId, ClientId, SubscriptionId);
                // Flag subscription for HDInsight, if it isn't already.
                EnableHDInsight(authToken);
                // Get an HDInsight management client
                _hdiManagementClient = new HDInsightManagementClient(authToken);

                // Set parameters for the new cluster
                var parameters = new ClusterCreateParameters
                {
                    ClusterSizeInNodes = NewClusterNumNodes,
                    UserName = NewClusterUsername,
                    ClusterType = NewClusterType,
                    OSType = NewClusterOSType,
                    Version = NewClusterVersion,

                    // Use an Azure storage account as the default storage
                    DefaultStorageInfo = new AzureStorageInfo(ExistingStorageName, ExistingStorageKey, ExistingBlobContainer),

                    // Is the cluster type RServer? If so, you can set the EdgeNodeSize.
                    // Otherwise, the default VM size is used.
                    //EdgeNodeSize = "Standard_D12_v2",

                    Password = NewClusterPassword,
                    Location = NewClusterLocation,

                    SshUserName = NewClusterSshUserName,
                    SshPassword = NewClusterSshPassword,
                    //SshPublicKey = NewClusterSshPublicKey
                };

                // Is the cluster type RServer? If so, add the RStudio configuration option.
                /*
                parameters.Configurations.Add(
                    "rserver",
                    new Dictionary<string, string>()
                    {
                        { "rstudio", "true" }
                    }
                );
                */

                // Create the cluster
                _hdiManagementClient.Clusters.Create(ExistingResourceGroupName, NewClusterName, parameters);

                System.Console.WriteLine("The cluster has been created. Press ENTER to continue ...");
                System.Console.ReadLine();
            }

            /// <summary>
            /// Authenticate to an Azure subscription and retrieve an authentication token
            /// </summary>
            static TokenCloudCredentials GetTokenCloudCredentials(string TenantId, string ClientId, string SubscriptionId)
            {
                var authContext = new AuthenticationContext("https://login.microsoftonline.com/" + TenantId);
                var tokenAuthResult = authContext.AcquireTokenAsync("https://management.core.windows.net/",
                    ClientId, 
                    new Uri("urn:ietf:wg:oauth:2.0:oob"), 
                    new PlatformParameters(PromptBehavior.Always), 
                    UserIdentifier.AnyUser);
                return new TokenCloudCredentials(SubscriptionId, tokenAuthResult.Result.AccessToken);
            }
            /// <summary>
            /// Marks your subscription as one that can use HDInsight, if it has not already been marked as such.
            /// </summary>
            /// <remarks>This is essentially a one-time action; if you have already done something with HDInsight
            /// on your subscription, then this isn't needed at all and will do nothing.</remarks>
            /// <param name="authToken">An authentication token for your Azure subscription</param>
            static void EnableHDInsight(TokenCloudCredentials authToken)
            {
                // Create a client for the Resource manager and set the subscription ID
                var resourceManagementClient = new ResourceManagementClient(new TokenCredentials(authToken.Token));
                resourceManagementClient.SubscriptionId = SubscriptionId;
                // Register the HDInsight provider
                var rpResult = resourceManagementClient.Providers.Register("Microsoft.HDInsight");
            }
        }
    }
    ```

6. 取代類別成員值。
7. 按 **F5** 鍵執行應用程式。 主控台視窗會開啟並顯示應用程式的狀態。 系統會提示您輸入 Azure 帳號認證。 建立 HDInsight 叢集可能需要幾分鐘的時間，通常約 15 分鐘。

## <a name="use-bootstrap"></a>使用 Bootstrap

藉由使用 Bootstrap，您可以在建立叢集期間設定額外的設定。  如需詳細資訊，請參閱 [使用 Bootstrap 自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-bootstrap.md)。

修改 [建立叢集](#create-clusters) 中的範例，以設定 Hive 設定：

```csharp
static void Main(string[] args)
{
    System.Console.WriteLine("Creating a cluster.  The process takes 10 to 20 minutes ...");

    // Authenticate and get a token
    var authToken = GetTokenCloudCredentials(TenantId, ClientId, SubscriptionId);
    // Flag subscription for HDInsight, if it isn't already.
    EnableHDInsight(authToken);
    // Get an HDInsight management client
    _hdiManagementClient = new HDInsightManagementClient(authToken);

    // Set parameters for the new cluster
    var extendedParameters = new ClusterCreateParametersExtended
    {
        Location = NewClusterLocation,
        Properties = new ClusterCreateProperties
        {
            ClusterDefinition = new ClusterDefinition
            {
                ClusterType = NewClusterType.ToString()
            },
            ClusterVersion = NewClusterVersion,
            OperatingSystemType = NewClusterOSType
        }
    };

    var coreConfigs = new Dictionary<string, string>
    {
        {"fs.defaultFS", string.Format("wasb://{0}@{1}", ExistingBlobContainer, ExistingStorageName)},
        {
            string.Format("fs.azure.account.key.{0}", ExistingStorageName),
            ExistingStorageKey
        }
    };

    // bootstrap
    var hiveConfigs = new Dictionary<string, string>
    {
        { "hive.metastore.client.socket.timeout", "90"}
    };

    var gatewayConfigs = new Dictionary<string, string>
    {
        {"restAuthCredential.isEnabled", "true"},
        {"restAuthCredential.username", NewClusterUsername},
        {"restAuthCredential.password", NewClusterPassword}
    };

    var configurations = new Dictionary<string, Dictionary<string, string>>
    {
        {"core-site", coreConfigs},
        {"gateway", gatewayConfigs},
        {"hive-site", hiveConfigs}
    };

    var serializedConfig = JsonConvert.SerializeObject(configurations);
    extendedParameters.Properties.ClusterDefinition.Configurations = serializedConfig;

    var sshPublicKeys = new List<SshPublicKey>();
    var sshPublicKey = new SshPublicKey
    {
        CertificateData =
            string.Format("ssh-rsa {0}", NewClusterSshPublicKey)
    };
    sshPublicKeys.Add(sshPublicKey);

    var headNode = new Role
    {
        Name = "headnode",
        TargetInstanceCount = 2,
        HardwareProfile = new HardwareProfile
        {
            VmSize = "Large"
        },
        OsProfile = new OsProfile
        {
            LinuxOperatingSystemProfile = new LinuxOperatingSystemProfile
            {
                UserName = NewClusterSshUserName,
                Password = NewClusterSshPassword //,
                // When use a SSH pulbic key, make sure to remove comments, headers and trailers, and concatenate the key into one line 
                //SshProfile = new SshProfile
                //{
                //    SshPublicKeys = sshPublicKeys
                //}
            }
        }
    };

    var workerNode = new Role
    {
        Name = "workernode",
        TargetInstanceCount = NewClusterNumNodes,
        HardwareProfile = new HardwareProfile
        {
            VmSize = "Large"
        },
        OsProfile = new OsProfile
        {
            LinuxOperatingSystemProfile = new LinuxOperatingSystemProfile
            {
                UserName = NewClusterSshUserName,
                Password = NewClusterSshPassword //,
                //SshProfile = new SshProfile
                //{
                //    SshPublicKeys = sshPublicKeys
                //}
            }
        }
    };

    extendedParameters.Properties.ComputeProfile = new ComputeProfile();
    extendedParameters.Properties.ComputeProfile.Roles.Add(headNode);
    extendedParameters.Properties.ComputeProfile.Roles.Add(workerNode);

    _hdiManagementClient.Clusters.Create(ExistingResourceGroupName, NewClusterName, extendedParameters);

    System.Console.WriteLine("The cluster has been created. Press ENTER to continue ...");
    System.Console.ReadLine();
}
```

## <a name="use-script-action"></a>使用指令碼動作

藉由使用指令碼動作，您可以在建立叢集期間設定額外的設定。  如需詳細資訊，請參閱[使用指令碼動作自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

修改 [建立叢集](#create-clusters) 中的範例，以呼叫用來安裝 R 的指令碼動作：

```csharp
static void Main(string[] args)
{
    System.Console.WriteLine("Creating a cluster.  The process takes 10 to 20 minutes ...");

    // Authenticate and get a token
    var authToken = GetTokenCloudCredentials(TenantId, ClientId, SubscriptionId);
    // Flag subscription for HDInsight, if it isn't already.
    EnableHDInsight(authToken);
    // Get an HDInsight management client
    _hdiManagementClient = new HDInsightManagementClient(authToken);

    // Set parameters for the new cluster
    var parameters = new ClusterCreateParameters
    {
        ClusterSizeInNodes = NewClusterNumNodes,
        Location = NewClusterLocation,
        ClusterType = NewClusterType,
        OSType = NewClusterOSType,
        Version = NewClusterVersion,

        DefaultStorageInfo = new AzureStorageInfo(ExistingStorageName, ExistingStorageKey, ExistingBlobContainer),

        UserName = NewClusterUsername,
        Password = NewClusterPassword,
        SshUserName = NewClusterSshUserName,
        SshPublicKey = NewClusterSshPublicKey
    };

    ScriptAction rScriptAction = new ScriptAction("Install R",
        new Uri("https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh"), "");

    parameters.ScriptActions.Add(ClusterNodeType.HeadNode,new System.Collections.Generic.List<ScriptAction> { rScriptAction});
    parameters.ScriptActions.Add(ClusterNodeType.WorkerNode, new System.Collections.Generic.List<ScriptAction> { rScriptAction });

    _hdiManagementClient.Clusters.Create(ExistingResourceGroupName, NewClusterName, parameters);

    System.Console.WriteLine("The cluster has been created. Press ENTER to continue ...");
    System.Console.ReadLine();
}
```

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="next-steps"></a>後續步驟
既然您已成功建立 HDInsight 叢集，請使用下列內容來了解如何使用您的叢集。 

### <a name="hadoop-clusters"></a>Hadoop 叢集
* [〈搭配 HDInsight 使用 Hivet〉](hadoop/hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hadoop/hdinsight-use-pig.md)
* [〈搭配 HDInsight 使用 MapReduce〉](hadoop/hdinsight-use-mapreduce.md)

### <a name="hbase-clusters"></a>HBase 叢集
* [開始在 HDInsight 上使用 HBase](hbase/apache-hbase-tutorial-get-started-linux.md)
* [在 HDInsight 上開發適用於 HBase 的 Java 應用程式](hbase/apache-hbase-build-java-maven-linux.md)

### <a name="storm-clusters"></a>Storm 叢集
* [在 HDInsight 上開發適用於 Storm 的 Java 拓撲](storm/apache-storm-develop-java-topology.md)
* [在 HDInsight 上的 Storm 中使用 Python 元件](storm/apache-storm-develop-python-topology.md)
* [在 HDInsight 上使用 Storm 部署和監視拓撲](storm/apache-storm-deploy-monitor-topology-linux.md)

### <a name="spark-clusters"></a>Spark 叢集
* [使用 Scala 建立獨立應用程式](spark/apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行工作](spark/apache-spark-livy-rest-interface.md)
* [Spark 和 BI：搭配 BI 工具來使用 HDInsight 中的 Spark 以執行互動式資料分析](spark/apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](spark/apache-spark-machine-learning-mllib-ipython.md)

### <a name="run-jobs"></a>執行工作
* [使用 .NET SDK 在 HDInsight 中執行 Hive 工作](hadoop/apache-hadoop-use-hive-dotnet-sdk.md)
* [使用 .NET SDK 在 HDInsight 中執行 Pig 工作](hadoop/apache-hadoop-use-pig-dotnet-sdk.md)
* [使用 .NET SDK 在 HDInsight 中執行 Sqoop 工作](hadoop/apache-hadoop-use-sqoop-dotnet-sdk.md)
* [在 HDInsight 中執行 Oozie 工作](hdinsight-use-oozie.md)

