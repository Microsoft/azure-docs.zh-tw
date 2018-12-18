---
title: 使用 .NET SDK 管理 HDInsight 中的 Hadoop 叢集 - Azure
description: 了解如何使用 HDInsight .NET SDK 對 HDInsight 中的 Hadoop 叢集執行管理工作。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 0d8238d5f1dca15aaba0f8f5a6580df97164006a
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43110794"
---
# <a name="manage-hadoop-clusters-in-hdinsight-by-using-net-sdk"></a>使用 .NET SDK 管理 HDInsight 中的 Hadoop 叢集
[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [HDInsight.NET SDK](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight)管理 HDInsight 叢集。

**先決條件**

開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

## <a name="connect-to-azure-hdinsight"></a>連接至 Azure HDInsight

您需要下列 NuGet 套件：

```
Install-Package Microsoft.Rest.ClientRuntime.Azure.Authentication -Pre
Install-Package Microsoft.Azure.Management.ResourceManager -Pre
Install-Package Microsoft.Azure.Management.HDInsight
```

下列程式碼範例示範如何先連接至 Azure，然後才透過 Azure 訂用帳戶管理 HDInsight 叢集。

```csharp
using System;
using Microsoft.Azure;
using Microsoft.Azure.Management.HDInsight;
using Microsoft.Azure.Management.HDInsight.Models;
using Microsoft.Azure.Management.ResourceManager;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.Rest;
using Microsoft.Rest.Azure.Authentication;

namespace HDInsightManagement
{
    class Program
    {
        private static HDInsightManagementClient _hdiManagementClient;
        // Replace with your AAD tenant ID if necessary
        private const string TenantId = UserTokenProvider.CommonTenantId; 
        private const string SubscriptionId = "<Your Azure Subscription ID>";
        // This is the GUID for the PowerShell client. Used for interactive logins in this example.
        private const string ClientId = "1950a258-227b-4e31-a9cf-717495945fc2";

        static void Main(string[] args)
        {
            // Authenticate and get a token
            var authToken = Authenticate(TenantId, ClientId, SubscriptionId);
            // Flag subscription for HDInsight, if it isn't already.
            EnableHDInsight(authToken);
            // Get an HDInsight management client
            _hdiManagementClient = new HDInsightManagementClient(authToken);

            // insert code here

            System.Console.WriteLine("Press ENTER to continue");
            System.Console.ReadLine();
        }

        /// <summary>
        /// Authenticate to an Azure subscription and retrieve an authentication token
        /// </summary>
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
    }
}
```

當您執行此程式時，應該會看到提示。  如果您不想要看到提示，請參閱 [建立非互動式驗證 .NET HDInsight 應用程式](hdinsight-create-non-interactive-authentication-dotnet-applications.md)。

## <a name="create-clusters"></a>建立叢集
請參閱 [使用 .NET SDK 在 HDInsight 中建立 Linux 型叢集](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md)

## <a name="list-clusters"></a>列出叢集
下列程式碼片段列出叢集和一些屬性︰

```csharp
var results = _hdiManagementClient.Clusters.List();
foreach (var name in results.Clusters) {
    Console.WriteLine("Cluster Name: " + name.Name);
    Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
    Console.WriteLine("\t Cluster location: " + name.Location);
    Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
}
```

## <a name="delete-clusters"></a>刪除叢集
使用下列程式碼片段，同步或非同步地刪除叢集︰ 

```csharp
_hdiManagementClient.Clusters.Delete("<Resource Group Name>", "<Cluster Name>");
_hdiManagementClient.Clusters.DeleteAsync("<Resource Group Name>", "<Cluster Name>");
```

## <a name="scale-clusters"></a>調整叢集
叢集調整功能可讓您變更在 Azure HDInsight 中執行的叢集所用的背景工作節點數目，而不需要重新建立叢集。

> [!NOTE]
> 只支援使用 HDInsight 3.1.3 版或更高版本的叢集。 如果不確定您的叢集版本，您可以檢查 [屬性] 頁面。  請參閱 [列出和顯示叢集](hdinsight-administer-use-portal-linux.md#list-and-show-clusters)。
> 
> 

變更 HDInsight 支援的每一種叢集所用的資料節點數目會有何影響：

* Hadoop
  
    您可以順暢地增加正在執行的 Hadoop 叢集中背景工作節點數目，而不會影響任何擱置或執行中的工作。 您也可以在作業進行當中提交新工作。 系統會順暢處理失敗的調整作業，讓叢集永保正常運作狀態。
  
    減少資料節點數目以縮減 Hadoop 叢集時，系統會重新啟動叢集中的部分服務。 這會導致所有執行中和擱置的工作在調整作業完成時失敗。 但您可以在作業完成後重新提交這些工作。
* hbase
  
    您可以順暢地在 HBase 叢集運作時對其新增或移除資料節點。 區域伺服器會在完成調整作業的數分鐘之內自動取得平衡。 但是，您也可以手動平衡區域伺服器，方法是登入叢集的前端節點，然後從命令提示字元視窗執行下列命令：
  
    ```bash
    >pushd %HBASE_HOME%\bin
    >hbase shell
    >balancer
    ```
* Storm
  
    您可以順暢地在 Storm 叢集運作時對其新增或移除資料節點。 但在調整作業順利完成後，您需要重新平衡拓撲。
  
    您可以使用兩種方式來完成重新平衡作業：
  
  * Storm Web UI
  * 命令列介面 (CLI) 工具
    
    如需詳細資訊，請參閱 [Apache Storm 文件](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) 。
    
    HDInsight 叢集上有提供 Storm Web UI：
    
    ![HDInsight Storm 調整重新平衡](./media/hdinsight-administer-use-management-portal/hdinsight-portal-scale-cluster-storm-rebalance.png)
    
    以下是如何使用 CLI 命令重新平衡 Storm 拓撲的範例：
    
    ```cli
    ## Reconfigure the topology "mytopology" to use 5 worker processes,
    ## the spout "blue-spout" to use 3 executors, and
    ## the bolt "yellow-bolt" to use 10 executors
    $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10
    ```

下列程式碼片段示範如何同步或非同步地調整叢集大小︰

```csharp
_hdiManagementClient.Clusters.Resize("<Resource Group Name>", "<Cluster Name>", <New Size>);   
_hdiManagementClient.Clusters.ResizeAsync("<Resource Group Name>", "<Cluster Name>", <New Size>);   
```

## <a name="grantrevoke-access"></a>授與/撤銷存取權
HDInsight 叢集具有下列 HTTP Web 服務 (所有這些服務都有 RESTful 端點)：

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

預設會授與這些服務的存取權。 您可以撤銷/授與存取權。 撤銷：

```csharp
var httpParams = new HttpSettingsParameters
{
    HttpUserEnabled = false,
    HttpUsername = "admin",
    HttpPassword = "*******",
};
_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);
```

授與：

```csharp
var httpParams = new HttpSettingsParameters
{
    HttpUserEnabled = enable,
    HttpUsername = "admin",
    HttpPassword = "*******",
};
_hdiManagementClient.Clusters.ConfigureHttpSettings("<Resource Group Name>, <Cluster Name>, httpParams);
```

> [!NOTE]
> 透過授與/撤銷存取權，您將重設叢的使用者名稱和密碼。
> 
> 

這也可以透過入口網站完成。 請參閱[使用 Azure 入口網站管理 HDInsight][hdinsight-admin-portal]。

## <a name="update-http-user-credentials"></a>更新 HTTP 使用者認證
與[授與/撤銷 HTTP 存取權](#grant/revoke-access)程序一樣。若已授與叢集 HTTP 存取權，則必須先將其撤銷。  然後再使用新的 HTTP 使用者認證授與存取權。

## <a name="find-the-default-storage-account"></a>尋找預設的儲存體帳戶
下列程式碼片段示範如何取得叢集的預設儲存體帳戶名稱和預設儲存體帳戶金鑰。

```csharp
var results = _hdiManagementClient.Clusters.GetClusterConfigurations(<Resource Group Name>, <Cluster Name>, "core-site");
foreach (var key in results.Configuration.Keys)
{
    Console.WriteLine(String.Format("{0} => {1}", key, results.Configuration[key]));
}
```

## <a name="submit-jobs"></a>提交工作
**提交 MapReduce 作業**

請參閱 [在 HDInsight 中執行 Hadoop MapReduce 範例](hadoop/apache-hadoop-run-samples-linux.md)。

**提交 Hive 作業** 

請參閱 [使用 .NET SDK 執行 Hive 查詢](hadoop/apache-hadoop-use-hive-dotnet-sdk.md)。

**提交 Pig 作業**

請參閱 [使用 .NET SDK 執行 Pig 作業](hadoop/apache-hadoop-use-pig-dotnet-sdk.md)。

**提交 Sqoop 作業**

請參閱 [在 HDInsight 上使用 Sqoop](hadoop/apache-hadoop-use-sqoop-dotnet-sdk.md)。

**提交 Oozie 作業**

請參閱 [在 HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程](hdinsight-use-oozie-linux-mac.md)。

## <a name="upload-data-to-azure-blob-storage"></a>將資料上傳至 Azure Blob 儲存體
請參閱[將資料上傳至 HDInsight][hdinsight-upload-data]。

## <a name="see-also"></a>另請參閱
* [HDInsight .NET SDK 參考文件](https://docs.microsoft.com/dotnet/api/overview/azure/hdinsight)
* [使用 Azure 入口網站管理 HDInsight][hdinsight-admin-portal]
* [使用命令列介面管理 HDInsight][hdinsight-admin-cli]
* [建立 HDInsight 叢集][hdinsight-provision]
* [將資料上傳至 HDInsight][hdinsight-upload-data]
* [開始使用 Azure HDInsight][hdinsight-get-started]

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-provision-custom-options]: hdinsight-hadoop-provision-linux-clusters.md#configuration
[hdinsight-submit-jobs]:hadoop/submit-apache-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-portal-linux.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md


