---
title: 搭配 Azure HDInsight 叢集使用 Azure Data Lake Storage Gen2 預覽
description: 了解如何從 Azure Data Lake Storage Gen2 預覽查詢資料，並儲存分析的結果。
author: jamesbak
ms.component: data-lake-storage-gen2
ms.service: storage
ms.topic: article
ms.date: 06/27/2018
ms.author: jamesbak
ms.openlocfilehash: 3869d83ada1cbe0b234694b6acae88b6f68fc2dd
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43782272"
---
# <a name="use-azure-data-lake-storage-gen2-preview-with-azure-hdinsight-clusters"></a>搭配 Azure HDInsight 叢集使用 Azure Data Lake Storage Gen2 預覽

若要分析 HDInsight 叢集中的資料，您可以將資料儲存在 Azure 儲存體、Azure Data Lake Storage Gen1 或 Azure Data Lake Storage Gen2 預覽的任何組合中。 所有儲存體選項都可讓您安全地刪除用於計算的 HDInsight 叢集，而不會遺失使用者資料。

Hadoop 支援預設檔案系統的概念。 預設檔案系統意指預設配置和授權。 也可用來解析相對路徑。 進行 HDInsight 叢集建立程序期間，您可以指定 Azure 儲存體或 Azure Data Lake Storage 中的 Blob 容器作為預設檔案系統。 或者，您也可以使用 HDInsight 3.5，選取 Azure 儲存體或 Azure Data Lake Storage 作為預設的檔案系統，但有一些例外狀況。

在本文中，您將了解 Azure Data Lake Storage Gen2 與 HDInsight 叢集搭配運作的方式。 如需建立 HDInsight 叢集的詳細資訊，請參閱[使用 Azure Data Lake Storage 搭配 Hadoop、Spark、Kafka 等等來設定 HDInsight 叢集](quickstart-create-connect-hdi-cluster.md)。

Azure 儲存體是強大的一般用途儲存體解決方案，其完美整合了 HDInsight。 HDInsight 可以使用 Azure Data Lake Storage 作為叢集的預設檔案系統。 透過 Hadoop 分散式檔案系統 (HDFS) 介面，可直接在 Azure Data Lake Storage 的檔案上運作 HDInsight 中的整套元件。

不建議您使用預設的檔案系統來儲存商務資料。 最好在每次使用後刪除預設的檔案系統，以減少儲存成本。 請注意，預設容器包含應用程式與系統記錄檔。 請務必先擷取記錄檔再刪除容器。

不支援多個叢集共用一個檔案系統。

## <a name="hdinsight-storage-architecture"></a>HDInsight 儲存架構

下圖提供使用 Azure 儲存體之 HDInsight 儲存架構的摘要檢視：

![Hadoop 叢集會使用 HDFS API 來存取和儲存 Blob 儲存體中的結構化和非結構化資料。](./media/use-hdi-cluster/HDI.ABFS.Arch.png "HDInsight 儲存體架構")

HDInsight 可以存取本機連接至計算節點的分散式檔案系統。 可使用完整 URI 來存取此檔案系統，例如：

    hdfs://<NAME_NODE_HOST>/<PATH>

此外，HDInsight 也能讓您存取儲存在 Azure Data Lake Storage 中的資料。 語法為：

    abfs[s]://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/<path>

以下是搭配 HDInsight 叢集使用 Azure 儲存體帳戶時的一些考量。

* **連線到叢集的儲存體帳戶中檔案**在建立期間，會擁有與叢集相關聯的帳戶名稱和金鑰。 此設定可讓您完整存取檔案系統中的檔案。

* **「未」連線到叢集的儲存體帳戶中公用檔案**會對檔案系統中的檔案公開唯讀權限。
  
  > [!NOTE]
  > 公用檔案系統可讓您取得檔案系統中所有可用檔案的清單，並讓您可存取中繼資料。 只有在您知道確切的 URL 時，公用檔案系統才可讓您存取檔案。 如需詳細資訊，請參閱[限制對容器和 Blob 的存取](http://msdn.microsoft.com/library/windowsazure/dd179354.aspx) (容器和 Blob 的規則與檔案和檔案系統的運作方式相同)。
 
* **儲存體帳戶中「未」連線到叢集的私人檔案系統**不允許存取檔案系統中的檔案，除非您在提交 WebHCat 作業時定義了儲存體帳戶。 本文稍後將會說明這項限制的原因。

建立程序及其金鑰中定義的儲存體帳戶會儲存在叢集節點的 *%HADOOP_HOME%/conf/core-site.xml* 中。 HDInsight 的預設行為是使用 *core-site.xml* 檔案中定義的儲存體帳戶。 您可以使用 [Ambari](../../hdinsight/hdinsight-hadoop-manage-ambari.md) 來修改此設定

多個 WebHCat 工作 (包括 Hive、MapReduce、Hadoop 資料流和 Pig) 可隨身夾帶儲存體帳戶的說明和中繼資料。 (這個方法目前適用於具有儲存體帳戶的 Pig，但不適用於中繼資料)。如需詳細資訊，請參閱 [在其他儲存體帳戶和 Metastores 上使用 HDInsight 叢集](http://social.technet.microsoft.com/wiki/contents/articles/23256.using-an-hdinsight-cluster-with-alternate-storage-accounts-and-metastores.aspx)。

## <a id="benefits"></a>Azure 儲存體的優點

計算叢集和儲存體叢集未並存於同處所隱含的效能損失，可經由將計算叢集建立到靠近 Azure 區域內的儲存體帳戶資源來彌補，其中的高速網路可讓計算節點有效率地存取 Azure 儲存體內的資料。

將資料儲存在 Azure 儲存體而非 HDFS 有許多優點：

* **資料重複使用和共用：** HDFS 中的資料位於計算叢集內。 只有可存取計算叢集的應用程式，才能利用 HDFS API 來使用資料。 可透過 HDFS API 或 [Blob 儲存體 REST API][blob-storage-restAPI] 來存取 Azure 儲存體中的資料。 因此，許多應用程式 (包括其他 HDInsight 叢集) 和工具都可用來產生和取用資料。

* **資料封存：** 將資料儲存在 Azure 儲存體中，可安全地刪除用於計算的 HDInsight 叢集，而不會遺失使用者資料。

* **資料儲存成本：** 長期將資料儲存在原生 DFS 中的成本高於將資料儲存在 Azure 儲存體中，因為計算叢集的成本高於 Azure 儲存體的成本。 此外，因為不需要每次產生計算叢集時都重新載入資料，也能節省資料載入成本。

* **彈性向外延展：** 雖然HDFS 提供向外延展的檔案系統，但延展程度取決於您建立給叢集的節點數目。 變更延展程度較為複雜，可改用 Azure 儲存體自動提供的彈性延展功能。

* **異地複寫：** Azure 儲存體資料可以進行異地複寫。 雖然此功能可支援地理位置復原和資料備援，但容錯移轉至異地複寫的位置會嚴重影響效能，且可能產生額外的成本。 因此，請謹慎選擇異地複寫，且最好在資料的價值大於額外成本時才這樣做。

* **資料生命週期管理：** 任何檔案系統中的所有資料都會有其本身的生命週期。 資料一開始通常會非常有價值且常被存取，然後逐漸失去其價值且較少受到存取，最終需要封存或刪除。 Azure 儲存體會提供資料階層處理和生命週期管理原則，將資料依其生命週期階段適當分層。

某些 MapReduce 工作和封裝可能會產生中繼結果，但您並不真的想要將這些結果儲存在 Azure 儲存體中。 在此情況下，您仍可選擇將資料儲存在本機 HDFS。 事實上，在 Hive 工作和其他程序中，HDInsight 會使用 HDFS 實作 (稱為 DFS) 來儲存其中某些中繼結果。

> [!NOTE]
> 大部分 HDFS 命令 (例如，`ls`、`copyFromLocal` 和 `mkdir`) 仍可正常運作。 只有 DFS 的特定命令 (例如 `fschk` 和 `dfsadmin`) 才會在 Azure 儲存體上出現不同的行為。

## <a name="create-an-data-lake-storage-file-system"></a>建立 Data Lake Storage 檔案系統

若要使用檔案系統，您必須先建立 [Azure 儲存體帳戶][azure-storage-create]。 在這個程序中，您可以指定建立儲存體帳戶所在的 Azure 區域。 叢集與儲存體帳戶必須在相同區域內託管。 Hive 中繼存放區 SQL Server 資料庫和 Oozie 中繼存放區 SQL Server 資料庫也必須位在相同的區域內。

您所建立的每個 Blob 不論位於何處，都屬於 Azure Data Lake Storage 帳戶中的某個檔案系統。 

預設 Data Lake Storage 檔案系統會儲存叢集特定資訊，例如作業記錄和記錄。 請勿將預設 Data Lake Storage 檔案系統與多個 HDInsight 叢集共用。 這可能會損毀作業歷程記錄。 建議您為每個叢集使用不同的檔案系統，並在所有相關叢集的部署中指定的連結儲存體帳戶 (而不是預設儲存體帳戶) 上放置共用的資料。 如需如何設定連結儲存體帳戶的詳細資訊，請參閱[建立 HDInsight 叢集][hdinsight-creation]。 不過，在刪除原始的 HDInsight 叢集後，您可以重複使用預設儲存檔案系統。 對於 HBase 叢集，您可以利用已刪除的 HBase 叢集所使用的預設 Blob 容器來建立一個新的 HBase 叢集，藉以保留 HBase 資料表結構描述和資料。

[!INCLUDE [secure-transfer-enabled-storage-account](../../../includes/hdinsight-secure-transfer.md)]

### <a name="use-the-azure-portal"></a>使用 Azure 入口網站

從入口網站建立 HDInsight 叢集時，您可以選擇提供存體帳戶詳細資料 (如下列螢幕擷取畫面所示)。 您也可以指定是否要將額外的儲存體帳戶與叢集相關聯；若是如此，請針對額外的儲存體選擇任何可用的儲存體選項。

![HDinsight hadoop 建立資料來源](./media/use-hdi-cluster/create-storage-account.png)

> [!WARNING]
> 不支援在與 HDInsight 叢集不同的位置中使用其他儲存體帳戶。

### <a name="use-azure-powershell"></a>使用 Azure PowerShell

如果您已[安裝和設定 Azure PowerShell][powershell-install]，可以從 Azure PowerShell 命令提示字元使用下列程式碼來建立儲存體帳戶和容器：

[!INCLUDE [upgrade-powershell](../../../includes/hdinsight-use-latest-powershell.md)]

    $SubscriptionID = "<Your Azure Subscription ID>"
    $ResourceGroupName = "<New Azure Resource Group Name>"
    $Location = "WEST US 2"

    $StorageAccountName = "<New Azure Storage Account Name>"
    $containerName = "<New Azure Blob Container Name>"

    Connect-AzureRmAccount
    Select-AzureRmSubscription -SubscriptionId $SubscriptionID

    # Create resource group
    New-AzureRmResourceGroup -name $ResourceGroupName -Location $Location

    # Create default storage account
    New-AzureRmStorageAccount -ResourceGroupName $ResourceGroupName `
      -Name StorageAccountName `
      -Location $Location `
      -SkuName Standard_LRS `
      -Kind StorageV2 
      -HierarchialNamespace $True

    # Create default blob containers
    $storageAccountKey = (Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -StorageAccountName $StorageAccountName)[0].Value
    $destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  
    New-AzureStorageContainer -Name $containerName -Context $destContext

> [!NOTE]
> 建立容器與在 Azure Data Lake Storage 中建立檔案系統為同義詞。

### <a name="use-azure-cli"></a>使用 Azure CLI

[!INCLUDE [use-latest-version](../../../includes/hdinsight-use-latest-cli.md)]

如果您已 [安裝和設定 Azure CLI](../../cli-install-nodejs.md)，下列命令即可用於儲存體帳戶和容器。

```bash
az storage account create \
    --name <STORAGE_ACCOUNT_NAME> \
    --resource-group <RESOURCE_GROUP_NAME> \
    --location westus2 \
    --sku Standard_LRS \
    --kind StorageV2 \
    --Enable-hierarchical-namespace true
```

> [!NOTE]
> Data Lake Storage Gen2 的公開預覽期間僅支援 `--sku Standard_LRS`。

系統會提示您指定將建立儲存體帳戶的地理區域。 請在您計劃要建立 HDInsight 叢集的相同區域中建立儲存體帳戶。

建立儲存體帳戶後，使用下列命令來抓取儲存體帳戶金鑰：

    azure storage account keys list <STORAGE_ACCOUNT_NAME>

若要建立容器，請使用下列命令：

    azure storage container create <CONTAINER_NAME> --account-name <STORAGE_ACCOUNT_NAME> --account-key <STORAGE_ACCOUNT_KEY>

> [!NOTE]
> 建立容器與在 Azure Data Lake Storage 中建立檔案系統為同義詞。

## <a name="address-files-in-azure-storage"></a>定址 Azure 儲存體中的檔案

從 HDInsight 存取 Azure 儲存體中的檔案的 URI 配置如下：

    abfs[s]://<FILE_SYSTEM_NAME>@<ACCOUNT_NAME>.dfs.core.windows.net/<PATH>

URI 配置提供未加密存取 (使用 abfs: 首碼) 和 SSL 加密存取 (使用 abfss 首碼)。 建議盡可能使用 abfss，即使是存取 Azure 中相同區域內的資料也一樣。

* &lt;FILE_SYSTEM_NAME&gt; 可識別檔案系統 Azure Data Lake Storage 的路徑。
* &lt;ACCOUNT_NAME&gt; 可識別 Azure 儲存體帳戶名稱。 需要使用完整網域名稱 (FQDN)。

    如果 &lt;FILE_SYSTEM_NAME&gt; 和 &lt;ACCOUNT_NAME&gt; 的值皆未指定，則會使用預設檔案系統。 對於預設檔案系統上的檔案，您可以使用相對路徑或絕對路徑。 例如，可使用下列其中一個路徑來參考 HDInsight 叢集隨附的 hadoop-mapreduce-examples.jar 檔案：
    
        abfs://myfilesystempath@myaccount.dfs.core.windows.net/example/jars/hadoop-mapreduce-examples.jar
        abfs:///example/jars/hadoop-mapreduce-examples.jar
        /example/jars/hadoop-mapreduce-examples.jar

> [!NOTE]
> 在 HDInsight 2.1 和 1.6 版叢集中的檔案名稱是 *hadoop-examples.jar*。

* &lt;PATH&gt; 是檔案或目錄 HDFS 路徑名稱。

> [!NOTE]
> 使用 HDInsight 外部的檔案時，大部分的公用程式會無法辨識 ABFS 格式，因而預期基本的路徑格式，例如 `example/jars/hadoop-mapreduce-examples.jar`。
 
## <a name="use-additional-storage-accounts"></a>使用其他儲存體帳戶

在建立 HDInsight 叢集時，您會指定要與它相關聯的 Azure 儲存體帳戶。 除了此儲存體帳戶，您也可以在建立過程中或在叢集建立後，從相同或不同的 Azure 訂用帳戶中新增其他儲存體帳戶。 如需關於新增其他儲存體帳戶的指示，請參閱[建立 HDInsight 叢集](../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。

> [!WARNING]
> 不支援在與 HDInsight 叢集不同的位置中使用其他儲存體帳戶。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何搭配 HDInsight 使用 HDFS 相容的 Azure 儲存體。 這種方法可讓您建立可調整、長期封存的資料取得解決方案，並利用 HDInsight 來將儲存的結構化和非結構化資料內的資訊解除鎖定。

如需詳細資訊，請參閱

* [Azure Data Lake Storage Gen2 的 ABFS Hadoop Filesystem 驅動程式](abfs-driver.md)
* [Azure Data Lake Storage 簡介](introduction.md)
* [使用 Azure Data Lake Storage 搭配 Hadoop、Spark、Kafka 等等來設定 HDInsight 叢集](quickstart-create-connect-hdi-cluster.md)
* [使用 Distcp 將資料內嵌到 Azure Data Lake Storage](use-distcp.md)

[powershell-install]: /powershell/azureps-cmdlets-docs
[hdinsight-creation]: ../../hdinsight/hdinsight-hadoop-provision-linux-clusters.md

[blob-storage-restAPI]: http://msdn.microsoft.com/library/windowsazure/dd135733.aspx
[azure-storage-create]: ../common/storage-create-storage-account.md

[img-hdi-powershell-blobcommands]: ./media/use-hdi-cluster/HDI.PowerShell.BlobCommands.png
