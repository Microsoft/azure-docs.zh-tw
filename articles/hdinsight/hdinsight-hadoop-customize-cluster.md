---
title: 使用指令碼動作自訂 HDInsight 叢集 - Azure
description: 深入了解使用指令碼動作來自訂 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 10/05/2016
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 6c4652e65f0f320063d989f97a5428510913005b
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105919"
---
# <a name="customize-windows-based-hdinsight-clusters-using-script-action"></a>使用指令碼動作自訂 Windows 型 HDInsight 叢集
**指令碼動作** 可用來叫用 [自訂指令碼](hdinsight-hadoop-script-actions.md) 。

本文的資訊是針對以 Windows 為基礎的 HDInsight 叢集。 如果是以 Linux 為基礎的叢集，請參閱 [使用指令碼動作自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

您也可以使用多種其他方法來自訂 HDInsight 叢集，例如包括額外的 Azure 儲存體帳戶、變更 Hadoop 組態檔 (core-site.xml、hive-site.xml 等)，或是將共用程式庫 (例如 Hive、Oozie) 加入至叢集中的共同位置。 這些自訂可以透過 Azure PowerShell、Azure HDInsight .NET SDK 或 Azure 入口網站來完成。 如需詳細資訊，請參閱[在 HDInsight 中建立 Hadoop 叢集][hdinsight-provision-cluster]。

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell-cli-and-dotnet-sdk.md)]

## <a name="script-action-in-the-cluster-creation-process"></a>叢集建立程序中的指令碼動作
只有正在建立叢集時，才會使用「指令碼動作」。 下圖說明在建立程序期間執行指令碼動作的時間：

![HDInsight 叢集自訂和叢集建立期間的階段][img-hdi-cluster-states]

當指令碼執行時，叢集會進入 **ClusterCustomization** 階段。 在此階段，指令碼會在系統管理員帳戶下，以平行方式在叢集中所有指定的節點上執行，而在節點上提供完整的系統管理員權限。

> [!NOTE]
> 因為您在 **ClusterCustomization** 階段中於叢集節點上擁有系統管理員權限，所以您可以使用指令碼來執行作業，例如停止和啟動服務，包括 Hadoop 相關服務。 因此，在指令碼中，您必須在指令碼完成執行之前，確定 Ambari 服務及其他 Hadoop 相關服務已啟動並且正在執行。 這些服務必須在叢集建立時，成功地確定叢集的健康情況和狀態。 如果您變更叢集上的任何會影響這些服務的組態，就必須使用所提供的協助程式函式。 如需 Helper 函式的詳細資訊，請參閱[開發 HDInsight 的指令碼動作指令碼][hdinsight-write-script]。
>
>

指令碼的輸出和錯誤記錄檔會儲存在您為叢集指定的預設儲存體帳戶中。 記錄檔是以 **u<\cluster-name-fragment><\time-stamp>setuplog** 的名稱儲存在資料表中。 這些是從叢集中所有節點上 (前端節點和背景工作節點) 執行之指令碼彙總的記錄檔。

每個叢集可接受多個指令碼動作，這些指令碼會依其指定順序被叫用。 指令碼可在前端節點、背景工作節點或同時在兩者執行。

HDInsight 提供數個指令碼在 HDInsight 叢集上安裝下列元件：

| 名稱 | 指令碼 |
| --- | --- |
| **安裝 Spark** | `https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1`。 請參閱[在 HDInsight 叢集上安裝和使用 Spark][hdinsight-install-spark]。 |
| **安裝 R** | `https://hdiconfigactions.blob.core.windows.net/rconfigactionv02/r-installer-v02.ps1`。 請參閱 [在 HDInsight 叢集上安裝及使用 R](r-server/r-server-hdinsight-manage.md#install-additional-r-packages-on-the-cluster)。 |
| **安裝 Solr** | `https://hdiconfigactions.blob.core.windows.net/solrconfigactionv01/solr-installer-v01.ps1`。 請參閱 [在 HDInsight 叢集上安裝及使用 Solr](hdinsight-hadoop-solr-install.md)。 |
| **安裝 Giraph** | `https://hdiconfigactions.blob.core.windows.net/giraphconfigactionv01/giraph-installer-v01.ps1`。 請參閱 [在 HDInsight 叢集上安裝及使用 Giraph](hdinsight-hadoop-giraph-install.md)。 |
| **預先載入 Hive 程式庫** | `https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1`。 請參閱 [在 HDInsight 叢集上新增 Hive 程式庫](hdinsight-hadoop-add-hive-libraries.md) |

## <a name="call-scripts-using-the-azure-portal"></a>使用 Azure 入口網站來呼叫指令碼
**從 Azure 入口網站**

1. 依[在 HDInsight 建立 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)中的描述開始建立叢集。
2. 在 [選擇性組態] 下方的 [指令碼動作] 刀鋒視窗中，按一下 [加入指令碼動作] 以提供有關指令碼動作的詳細資料，如下所示：

    ![使用指令碼動作以自訂叢集](./media/hdinsight-hadoop-customize-cluster/HDI.CreateCluster.8.png "使用指令碼動作以自訂叢集")

    <table border='1'>
        <tr><th>屬性</th><th>值</th></tr>
        <tr><td>名稱</td>
            <td>指定指令碼動作的名稱。</td></tr>
        <tr><td>指令碼 URI</td>
            <td>對自訂叢集所叫用的指令碼指定 URI。 s</td></tr>
        <tr><td>Head/Worker</td>
            <td>指定執行自訂指令碼的節點 (**Head** 或 **Worker**)。</b>
        <tr><td>參數</td>
            <td>如果指令碼要求，請指定參數。</td></tr>
    </table>

    請按 ENTER 加入一個以上的指令碼動作，以在叢集上安裝多個元件。
3. 按一下 [ **選取** ] 以儲存指令碼動作組態，並繼續建立叢集。

## <a name="call-scripts-using-azure-powershell"></a>使用 Azure PowerShell 呼叫指令碼
接下來的這個 PowerShell 指令碼示範如何在以 Windows 為基礎的 HDInsight 叢集上安裝 Spark。  

    # Provide values for these variables
    $subscriptionID = "<Azure Suscription ID>" # After "Connect-AzureRmAccount", use "Get-AzureRmSubscription" to list IDs.

    $nameToken = "<Enter A Name Token>"  # The token is use to create Azure service names.
    $namePrefix = $nameToken.ToLower() + (Get-Date -Format "MMdd")

    $resourceGroupName = $namePrefix + "rg"
    $location = "EAST US 2" # used for creating resource group, storage account, and HDInsight cluster.

    $hdinsightClusterName = $namePrefix + "spark"
    $httpUserName = "admin"
    $httpPassword = "<Enter a Password>"

    $defaultStorageAccountName = "$namePrefix" + "store"
    $defaultBlobContainerName = $hdinsightClusterName

    #############################################################
    # Connect to Azure
    #############################################################

    Try{
        Get-AzureRmSubscription
    }
    Catch{
        Connect-AzureRmAccount
    }
    Select-AzureRmSubscription -SubscriptionId $subscriptionID

    #############################################################
    # Prepare the dependent components
    #############################################################

    # Create resource group
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location

    # Create storage account
    New-AzureRmStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $defaultStorageAccountName `
        -Location $location `
        -Type Standard_GRS
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey `
                                    -ResourceGroupName $resourceGroupName `
                                    -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext `
                                    -StorageAccountName $defaultStorageAccountName `
                                    -StorageAccountKey $storageAccountKey  
    New-AzureStorageContainer `
        -Name $defaultBlobContainerName `
        -Context $defaultStorageAccountContext

    #############################################################
    # Create cluster with ApacheSpark
    #############################################################

    # Specify the configuration options
    $config = New-AzureRmHDInsightClusterConfig `
                -DefaultStorageAccountName "$defaultStorageAccountName.blob.core.windows.net" `
                -DefaultStorageAccountKey $defaultStorageAccountKey


    # Add a script action to the cluster configuration
    $config = Add-AzureRmHDInsightScriptAction `
                -Config $config `
                -Name "Install Spark" `
                -NodeType HeadNode `
                -Uri https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1 `

    # Start creating a cluster with Spark installed
    New-AzureRmHDInsightCluster `
            -ResourceGroupName $resourceGroupName `
            -ClusterName $hdinsightClusterName `
            -Location $location `
            -ClusterSizeInNodes 2 `
            -ClusterType Hadoop `
            -OSType Windows `
            -DefaultStorageContainer $defaultBlobContainerName `
            -Config $config


若要安裝其他軟體，您必須取代指令碼中的指令碼檔案：

出現提示時，請輸入叢集的認證。 建立叢集可能需要幾分鐘的時間。

## <a name="call-scripts-using-net-sdk"></a>使用 .NET SDK 呼叫指令碼
下列範例示範如何在以 Windows 為基礎的 HDInsight 叢集上安裝 Spark。 若要安裝其他軟體，您必須取代程式碼中的指令碼檔案。

**使用 Spark 建立 HDInsight 叢集**

1. 在 Visual Studio 建立 C# 主控台應用程式。
2. 從 NuGet Package Manager Console 執行下列命令：

        Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
        Install-Package Microsoft.Azure.Management.ResourceManager -Pre
        Install-Package Microsoft.Azure.Management.HDInsight
3. 在 Program.cs 檔案中使用下列 using 陳述式：

        using System;
        using System.Security;
        using Microsoft.Azure;
        using Microsoft.Azure.Management.HDInsight;
        using Microsoft.Azure.Management.HDInsight.Models;
        using Microsoft.Azure.Management.ResourceManager;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Rest;
        using Microsoft.Rest.Azure.Authentication;
4. 使用下列命令將程式碼放置在類別中：

        private static HDInsightManagementClient _hdiManagementClient;

        // Replace with your AAD tenant ID if necessary
        private const string TenantId = UserTokenProvider.CommonTenantId;
        private const string SubscriptionId = "<Your Azure Subscription ID>";
        // This is the GUID for the PowerShell client. Used for interactive logins in this example.
        private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";
        private const string ResourceGroupName = "<ExistingAzureResourceGroupName>";
        private const string NewClusterName = "<NewAzureHDInsightClusterName>";
        private const int NewClusterNumWorkerNodes = 2;
        private const string NewClusterLocation = "East US";
        private const string NewClusterVersion = "3.2";
        private const string ExistingStorageName = "<ExistingAzureStorageAccountName>";
        private const string ExistingStorageKey = "<ExistingAzureStorageAccountKey>";
        private const string ExistingContainer = "<ExistingAzureBlobStorageContainer>";
        private const string NewClusterType = "Hadoop";
        private const OSType NewClusterOSType = OSType.Windows;
        private const string NewClusterUsername = "<HttpUserName>";
        private const string NewClusterPassword = "<HttpUserPassword>";

        static void Main(string[] args)
        {
            System.Console.WriteLine("Running");

            // Authenticate and get a token
            var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
            // Flag subscription for HDInsight, if it isn't already.
            EnableHDInsight(authToken);
            // Get an HDInsight management client
            _hdiManagementClient = new HDInsightManagementClient(authToken);

            CreateCluster();
        }

        private static void CreateCluster()
        {
            var parameters = new ClusterCreateParameters
            {
                ClusterSizeInNodes = NewClusterNumWorkerNodes,
                Location = NewClusterLocation,
                ClusterType = NewClusterType,
                OSType = NewClusterOSType,
                Version = NewClusterVersion,
                DefaultStorageInfo = new AzureStorageInfo(ExistingStorageName, ExistingStorageKey, ExistingContainer),
                UserName = NewClusterUsername,
                Password = NewClusterPassword,
            };

            ScriptAction sparkScriptAction = new ScriptAction("Install Spark",
                new Uri("https://hdiconfigactions.blob.core.windows.net/sparkconfigactionv03/spark-installer-v03.ps1"), "");

            parameters.ScriptActions.Add(ClusterNodeType.HeadNode, new System.Collections.Generic.List<ScriptAction> { sparkScriptAction });
            parameters.ScriptActions.Add(ClusterNodeType.WorkerNode, new System.Collections.Generic.List<ScriptAction> { sparkScriptAction });

            _hdiManagementClient.Clusters.Create(ResourceGroupName, NewClusterName, parameters);
        }

        /// <summary>
        /// Authenticate to an Azure subscription and retrieve an authentication token
        /// </summary>
        /// <param name="TenantId">The AAD tenant ID</param>
        /// <param name="ClientId">The AAD client ID</param>
        /// <param name="SubscriptionId">The Azure subscription ID</param>
        /// <returns></returns>
        static TokenCloudCredentials Authenticate(string TenantId, string ClientId, string SubscriptionId)
        {
            var authContext = new AuthenticationContext("https://login.microsoftonline.com/" + TenantId);
            var tokenAuthResult = authContext.AcquireToken("https://management.core.windows.net/",
                ClientId,
                new Uri("urn:ietf:wg:oauth:2.0:oob"),
                PromptBehavior.Always,
                UserIdentifier.AnyUser);
            return new TokenCloudCredentials(SubscriptionId, tokenAuthResult.AccessToken);
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
5. 按 **F5** 鍵執行應用程式。

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>支援在 HDInsight 叢集上使用開放原始碼軟體
Microsoft Azure HDInsight 服務是彈性的平台，可讓您使用以 Hadoop 形成之開放原始碼技術的生態系統，在雲端中建置巨量資料應用程式。 Microsoft Azure 提供對開放原始碼技術的一般層級支援，如 **Azure 支援常見問題集網站** 的＜ <a href="http://azure.microsoft.com/support/faq/" target="_blank">支援範圍</a>＞章節中所述。 HDInsight 服務對於某些元件提供額外層級的支援，如下所述。

HDInsight 服務中有兩種類型的開放原始碼元件可用：

* **內建元件** - 這些元件預先安裝在 HDInsight 叢集上，並且提供叢集的核心功能。 例如，YARN ResourceManager、Hive 查詢語言 (HiveQL) 及 Mahout 程式庫都屬於這個類別。 叢集元件的完整清單可於 [HDInsight 所提供 Hadoop 叢集版本的新功能](hdinsight-component-versioning.md)中取得</a>。
* **自訂元件** - 身為叢集使用者的您可以安裝社群中可用或是您建立的任何元件，或者在工作負載中使用。

內建元件受到完整支援，且 Microsoft 支援服務將會協助釐清與解決這些元件的相關問題。

> [!WARNING]
> 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務將協助釐清與解決這些元件的相關問題。
>
> 自訂元件則獲得商務上合理的支援，協助您進一步疑難排解問題。 如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)、[Spark](http://spark.apache.org/)。
>
>

HDInsight 服務提供數種方式以使用自訂元件。 無論元件如何使用或如何安裝在叢集上，都適用相同層級的支援。 以下是自訂元件可用於 HDInsight 叢集之最常見方式的清單：

1. 工作提交 - Hadoop 或其他類型的工作，執行或使用可以提交給叢集的自訂元件。
2. 叢集自訂 - 在叢集建立期間，您可以指定額外設定和將會安裝在叢集節點上的自訂元件。
3. 範例 - 對於熱門自訂元件，Microsoft 和其他提供者可能會提供如何在 HDInsight 叢集上使用這些元件的範例。 提供這些範例，但是沒有支援。

## <a name="develop-script-action-scripts"></a>開發指令碼動作指令碼
請參閱[開發 HDInsight 的指令碼動作指令碼][hdinsight-write-script]。

## <a name="see-also"></a>另請參閱
* [在 HDInsight 中建立 Hadoop 叢集][hdinsight-provision-cluster]提供如何使用其他自訂選項建立 HDInsight 叢集的指示。
* [開發 HDInsight 的指令碼動作指令碼][hdinsight-write-script]
* [在 HDInsight 叢集上安裝和使用 Spark][hdinsight-install-spark]
* [在 HDInsight 叢集上安裝及使用 Solr](hdinsight-hadoop-solr-install.md)。
* [在 HDInsight 叢集上安裝及使用 Giraph](hdinsight-hadoop-giraph-install.md)。

[hdinsight-install-spark]: hdinsight-hadoop-spark-install.md
[hdinsight-write-script]: hdinsight-hadoop-script-actions.md
[hdinsight-provision-cluster]: hdinsight-hadoop-provision-linux-clusters.md
[powershell-install-configure]: /powershell/azureps-cmdlets-docs


[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster/HDI-Cluster-state.png "叢集建立期間的階段"
