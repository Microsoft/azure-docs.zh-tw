---
title: 使用 PowerShell 管理 HDInsight 中的 Hadoop 叢集 - Azure
description: 了解如何使用 Azure PowerShell 對 HDInsight 中的 Hadoop 叢集執行管理工作。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 7abd75347d7bd2673a9e6040ee15def6638de440
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43096577"
---
# <a name="manage-hadoop-clusters-in-hdinsight-by-using-azure-powershell"></a>使用 Azure PowerShell 管理 HDInsight 上的 Hadoop 叢集
[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

Azure PowerShell 可讓您在 Azure 中用來控制和自動化工作負載的部署及管理。 在本文中，您將了解如何使用 Azure PowerShell 來管理Azure HDInsight 中的 Hadoop 叢集。 如需 HDInsight PowerShell Cmdlet 的清單，請參閱 [HDInsight Cmdlet 參考文件](https://msdn.microsoft.com/library/azure/dn479228.aspx)。

**先決條件**

開始閱讀本文之前，您必須有下列各項：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

## <a name="install-azure-powershell"></a>安裝 Azure PowerShell
[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

如果您已安裝 Azure PowerShell 0.9 x 版，必須先解除安裝然後再安裝較新版本。

檢查已安裝的 PowerShell 版本：

```powershell
Get-Module *azure*
```

若要解除安裝較舊的版本，請在控制台中執行 [程式和功能]。

## <a name="create-clusters"></a>建立叢集
請參閱 [使用 Azure PowerShell 在 HDInsight 中建立以 Linux 為基礎的叢集](hdinsight-hadoop-create-linux-clusters-azure-powershell.md)

## <a name="list-clusters"></a>列出叢集
使用下列命令列出目前訂用帳戶中的所有叢集：

```powershell
Get-AzureRmHDInsightCluster
```

## <a name="show-cluster"></a>顯示叢集
使用下列命令顯示目前訂用帳戶中特定叢集的詳細資料：

```powershell
Get-AzureRmHDInsightCluster -ClusterName <Cluster Name>
```

## <a name="delete-clusters"></a>刪除叢集
使用下列命令來刪除叢集：

```powershell
Remove-AzureRmHDInsightCluster -ClusterName <Cluster Name>
```

您也可以移除包含叢集的資源群組來刪除叢集。 刪除資源群組會刪除群組中的所有資源 (包括預設儲存體帳戶)。

```powershell
Remove-AzureRmResourceGroup -Name <Resource Group Name>
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

    減少資料節點數目以縮減 Hadoop 叢集時，系統會重新啟動叢集中的部分服務。 重新啟動服務會導致所有執行中和擱置的工作在調整作業完成時失敗。 但您可以在作業完成後重新提交這些工作。
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

    ![hdinsight storm scale rebalance](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.png)

    以下是如何使用 CLI 命令重新平衡 Storm 拓撲的範例：

    ```cli
    ## Reconfigure the topology "mytopology" to use 5 worker processes,
    ## the spout "blue-spout" to use 3 executors, and
    ## the bolt "yellow-bolt" to use 10 executors
    $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10
    ```

若要使用 Azure PowerShell 變更 Hadoop 叢集大小，請從用戶端電腦執行下列命令：

```powershell
Set-AzureRmHDInsightClusterSize -ClusterName <Cluster Name> -TargetInstanceCount <NewSize>
```


## <a name="grantrevoke-access"></a>授與/撤銷存取權
HDInsight 叢集具有下列 HTTP Web 服務 (所有這些服務都有 RESTful 端點)：

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

預設會授與這些服務的存取權。 您可以撤銷/授與存取權。 撤銷：

```powershell
Revoke-AzureRmHDInsightHttpServicesAccess -ClusterName <Cluster Name>
```

授與：

```powershell
$clusterName = "<HDInsight Cluster Name>"

# Credential option 1
$hadoopUserName = "admin"
$hadoopUserPassword = "<Enter the Password>"
$hadoopUserPW = ConvertTo-SecureString -String $hadoopUserPassword -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$hadoopUserPW)

# Credential option 2
#$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"

Grant-AzureRmHDInsightHttpServicesAccess -ClusterName $clusterName -HttpCredential $credential
```

> [!NOTE]
> 透過授與/撤銷存取權，您將重設叢集使用者名稱和密碼。
>
>

授與及撤銷存取權也可以透過入口網站完成。 請參閱[使用 Azure 入口網站管理 HDInsight][hdinsight-admin-portal]。

## <a name="update-http-user-credentials"></a>更新 HTTP 使用者認證
這是與[授與/撤銷 HTTP 存取權](#grant/revoke-access)相同的程序。 如果已將 HTTP 存取權授與叢集，您必須先將它撤銷。  然後再使用新的 HTTP 使用者認證授與存取權。

## <a name="find-the-default-storage-account"></a>尋找預設的儲存體帳戶
下列 PowerShell 指令碼示範如何取得預設儲存體帳戶名稱和相關資訊：

```powershell
#Connect-AzureRmAccount
$clusterName = "<HDInsight Cluster Name>"

$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$storageInfo = $clusterInfo.DefaultStorageAccount.split('.')
$defaultStoreageType = $storageInfo[1]
$defaultStorageName = $storageInfo[0]

echo "Default Storage account name: $defaultStorageName"
echo "Default Storage account type: $defaultStoreageType"

if ($defaultStoreageType -eq "blob")
{
    $defaultBlobContainerName = $cluster.DefaultStorageContainer
    $defaultStorageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccountName)[0].Value
    $defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey

    echo "Default Blob container name: $defaultBlobContainerName"
    echo "Default Storage account key: $defaultStorageAccountKey"
}
```


## <a name="find-the-resource-group"></a>尋找資源群組
在 Resource Manager 模式中，每個 HDInsight 叢集皆屬於一個 Azure 資源群組。  尋找資源群組：

```powershell
$clusterName = "<HDInsight Cluster Name>"

$cluster = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroupName = $cluster.ResourceGroup
```


## <a name="submit-jobs"></a>提交工作
**提交 MapReduce 作業**

請參閱 [在以 Windows 為基礎的 HDInsight 中執行 Hadoop MapReduce 範例](hdinsight-run-samples.md)。

**提交 Hive 作業**

請參閱 [使用 PowerShell 執行 Hive 查詢](hadoop/apache-hadoop-use-hive-powershell.md)。

**提交 Pig 作業**

請參閱 [使用 PowerShell 執行 Pig 作業](hadoop/apache-hadoop-use-pig-powershell.md)。

**提交 Sqoop 作業**

請參閱 [在 HDInsight 上使用 Sqoop](hadoop/hdinsight-use-sqoop.md)。

**提交 Oozie 作業**

請參閱 [在 HDInsight 上搭配 Hadoop 使用 Oozie 來定義並執行工作流程](hdinsight-use-oozie.md)。

## <a name="upload-data-to-azure-blob-storage"></a>將資料上傳至 Azure Blob 儲存體
請參閱[將資料上傳至 HDInsight][hdinsight-upload-data]。

## <a name="see-also"></a>另請參閱
* [HDInsight Cmdlet 參考文件](https://msdn.microsoft.com/library/azure/dn479228.aspx)
* [使用 Azure 入口網站管理 HDInsight][hdinsight-admin-portal]
* [使用命令列介面管理 HDInsight][hdinsight-admin-cli]
* [建立 HDInsight 叢集][hdinsight-provision]
* [將資料上傳至 HDInsight][hdinsight-upload-data]
* [以程式設計方式提交 Hadoop 工作][hdinsight-submit-jobs]
* [開始使用 Azure HDInsight][hdinsight-get-started]

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-provision-custom-options]: hdinsight-hadoop-provision-linux-clusters.md#configuration
[hdinsight-submit-jobs]:hadoop/submit-apache-hadoop-jobs-programmatically.md

[hdinsight-admin-cli]: hdinsight-administer-use-command-line.md
[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-flight]: hdinsight-analyze-flight-delay-data.md

[hdinsight-powershell-reference]: https://msdn.microsoft.com/library/dn858087.aspx

[powershell-install-configure]: /powershell/azureps-cmdlets-docs

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png
