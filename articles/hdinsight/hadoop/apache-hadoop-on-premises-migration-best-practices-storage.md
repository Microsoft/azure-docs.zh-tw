---
title: 儲存體：將內部部署 Apache Hadoop 遷移至 Azure HDInsight
description: 了解將內部部署 Hadoop 叢集移轉至 Azure HDInsight 的儲存體最佳做法。
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/10/2019
ms.openlocfilehash: 8d87d2164a5131b71a2000243c37553610497750
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98944852"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight"></a>將內部部署 Apache Hadoop 叢集遷移至 Azure HDInsight

本文提供 Azure HDInsight 系統中的資料儲存建議。 本文是系列文章中的一篇，提供有助於將內部部署 Apache Hadoop 系統移轉至 Azure HDInsight 的最佳做法。

## <a name="choose-right-storage-system-for-hdinsight-clusters"></a>為 HDInsight 叢集選擇正確的儲存體系統

您可以在 Azure Blob 儲存體或 Azure Data Lake Storage 中重新建立內部部署 Apache Hadoop 檔案系統 (HDFS) 目錄結構。 接著，您即可安全地刪除用於計算的 HDInsight 叢集，而不會遺失使用者資料。 這兩項服務都可以作為 HDInsight 叢集的預設檔案系統和其他檔案系統。 HDInsight 叢集與儲存體帳戶必須裝載於相同的區域內。

### <a name="azure-storage"></a>Azure 儲存體

HDInsight 叢集可以使用 Azure 儲存體中的 Blob 容器作為預設檔案系統或其他檔案系統。 目前支援將標準層儲存體帳戶用於 HDInsight 叢集。 不支援頂級層。 預設 Blob 容器會儲存叢集特定資訊，例如作業歷程記錄和記錄。 不支援多個叢集共用一個 Blob 容器作為預設檔案系統。

建立程序中定義的儲存體帳戶及其各自的金鑰會儲存在叢集節點的 `%HADOOP_HOME%/conf/core-site.xml` 中。 您也可以在 Ambari UI 中，從 HDFS 組態中的 [自訂核心網站] 區段下加以存取。 依預設會為儲存體帳戶金鑰加密，並在金鑰傳至 Hadoop 精靈之前使用自訂解密指令碼將金鑰解密。 包括 Hive、MapReduce、Hadoop 串流和 Pig 在內的各項作業，可夾帶儲存體帳戶的說明和中繼資料。

Azure 儲存體可以進行異地複寫。 雖然異地複寫可提供地理位置復原和資料備援性，但容錯移轉至異地複寫的位置會嚴重影響效能，且可能產生額外的成本。 建議您明智地選擇異地複寫，且最好在資料的價值大於額外成本時才這樣做。

您可以使用下列其中一種格式來存取儲存在 Azure 儲存體中的資料：

|資料存取格式 |描述 |
|---|---|
|`wasb:///`|使用未加密通訊存取預設儲存體。|
|`wasbs:///`|使用加密通訊存取預設儲存體。|
|`wasb://<container-name>@<account-name>.blob.core.windows.net/`|與非預設儲存體帳戶進行通訊時使用。 |

[標準儲存體帳戶的擴充性目標](../../storage/common/scalability-targets-standard-account.md) 會列出 Azure 儲存體帳戶目前的限制。 如果應用程式的需求超出單一儲存體帳戶的延展性目標，您可以建置使用多個儲存體帳戶的應用程式，並將資料物件分割到這些儲存體帳戶中。

[Azure 儲存體分析](../../storage/common/storage-analytics.md) 提供所有儲存體服務的度量，而 Azure 入口網站可以設定為透過圖表來呈現要視覺化的度量。 您可以建立警示，以在儲存體資源計量達到閾值時獲得通知。

Azure 儲存體提供 [blob 物件](../../storage/blobs/soft-delete-blob-overview.md) 的虛刪除，以在應用程式或其他儲存體帳戶使用者不小心修改或刪除資料時，協助復原資料。

您可以建立 [Blob 快照集](/rest/api/storageservices/creating-a-snapshot-of-a-blob)。 快照集是在某個點時間建立的唯讀 Blob 版本，可作為備份 Blob 的途徑。 一旦建立快照集後，即可加以讀取、複製或刪除，但不能修改。

> [!Note]
> 舊版的 Hadoop 散發套件若沒有 "wasbs" 憑證，就必須將該憑證匯入至 Java 信任存放區。

下列方法可用來將憑證匯入至 Java 信任存放區：

將 Azure Blob TLS/SSL 憑證下載至檔案

```bash
echo -n | openssl s_client -connect <storage-account>.blob.core.windows.net:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > Azure_Storage.cer
```

將上述檔案匯入至所有節點上的 Java 信任存放區

```bash
keytool -import -trustcacerts -keystore /path/to/jre/lib/security/cacerts -storepass changeit -noprompt -alias blobtrust -file Azure_Storage.cer
```

確認已新增的憑證位於信任存放區中

```bash
keytool -list -v -keystore /path/to/jre/lib/security/cacerts
```

如需詳細資訊，請參閱下列文章：

- [使用 Azure 儲存體搭配 Azure HDInsight 叢集](../hdinsight-hadoop-use-blob-storage.md)
- [標準儲存體帳戶的擴充性目標](../../storage/common/scalability-targets-standard-account.md)
- [Blob 儲存體的延展性和效能目標](../../storage/blobs/scalability-targets.md)
- [Microsoft Azure 儲存體效能與延展性檢查清單](../../storage/blobs/storage-performance-checklist.md)
- [監視、診斷與疑難排解 Microsoft Azure 儲存體](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md)
- [在 Azure 入口網站中監視儲存體帳戶](../../storage/common/storage-monitor-storage-account.md)

### <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 會實行 HDFS 和 POSIX 樣式存取控制模型。 它會提供與 Azure AD 的第一個類別整合，以進行更細微的存取控制。 它可儲存的資料並沒有大小的限制，且其執行大量平行分析的能力也不受限。

如需詳細資訊，請參閱下列文章：

- [使用 Azure 入口網站建立具有 Data Lake Storage Gen1 的 HDInsight 叢集](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)
- [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen1](../hdinsight-hadoop-use-data-lake-storage-gen1.md)

### <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 是最新的儲存體供應專案。 它結合了第一代 Azure Data Lake Storage Gen1 的核心功能，以及與 Hadoop 相容的檔案系統端點，直接整合到 Azure Blob 儲存體。 這項增強功能結合了物件儲存體在規模和成本上的優勢，以及一般只有內部部署檔案系統所能帶來的可靠性和效能。

Azure Data Lake Storage Gen 2 建置於 [Azure Blob 儲存體](../../storage/blobs/storage-blobs-introduction.md) 之上，並可讓您使用檔案系統和物件儲存架構來與資料進行介面。 [Azure Data Lake Storage Gen1](../../data-lake-store/index.yml)的功能（例如檔案系統語義、檔案層級安全性和級別）結合了低成本、分層式儲存體、高可用性/嚴重損壞修復功能，以及來自[Azure Blob 儲存體](../../storage/blobs/storage-blobs-introduction.md)的大型 SDK/工具生態系統。 在 Data Lake Storage Gen2 中，所有物件儲存體的品質都維持不變，另外還增加了經過分析工作負載最佳化的檔案系統介面優點。

Data Lake Storage Gen2 的基本功能是將 [階層命名空間](../../storage/blobs/data-lake-storage-namespace.md) 新增至 Blob 儲存體服務，以將物件/檔案組織成目錄階層，以進行高效能資料存取。 階層式結構可讓重新命名或刪除目錄之類的作業成為目錄中單一不可部分完成的中繼資料作業，而不是列舉及處理共用目錄名稱前置詞的所有物件。

以往雲端式分析必須在效能、管理及安全性方面妥協。 Azure Data Lake Storage (ADLS) Gen2 的主要功能如下：

- **Hadoop 相容存取**： Azure Data Lake Storage Gen2 可讓您管理及存取資料，就如同使用 [HADOOP 分散式檔案系統 (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)一樣。 新的 [ABFS 驅動程式](../../storage/blobs/data-lake-storage-abfs-driver.md) 可在包含于 [Azure HDInsight](../index.yml)的所有 Apache Hadoop 環境中使用。 此驅動程式可讓您存取儲存在 Data Lake Storage Gen2 中的資料。

- **POSIX 權限的超集合**：Data Lake Gen2 的安全性模型可完全支援 ACL 和 POSIX 權限，以及一些 Data Lake Storage Gen2 特有的額外細微姓。 這些設定可透過系統管理工具或 Hive 和 Spark 這類架構來配置。

- **符合成本效益**：Data Lake Storage Gen2 具有低成本儲存體容量和異動功能。 隨著資料在整個生命週期中進行轉換，計費率也會變更，以透過 [Azure Blob 儲存體生命週期](../../storage/blobs/storage-lifecycle-management-concepts.md)等內建功能將成本降至最低。

- **可搭配使用 Blob 儲存體工具、架構及應用程式**：Data Lake Storage Gen2 可繼續搭配使用現今仍有的各種 Blob 儲存體工具、架構及應用程式。

- **優化驅動程式**： Azure Blob 檔案系統驅動程式 (ABFS) 專門針對大型資料分析進行 [優化](../../storage/blobs/data-lake-storage-abfs-driver.md) 。 對應的 REST API 會透過 dfs 端點 dfs.core.windows.net 呈現。

您可以使用下列其中一種格式來存取儲存在 ADLS Gen2 中的資料：
- `abfs:///`:存取叢集的預設 Data Lake Storage。
- `abfs://file_system@account_name.dfs.core.windows.net`:與非預設 Data Lake Storage 進行通訊時使用。

如需詳細資訊，請參閱下列文章：

- [Azure Data Lake Storage Gen2 簡介](../../storage/blobs/data-lake-storage-introduction.md)
- [Azure Blob Filesystem 驅動程式 (ABFS.md)](../../storage/blobs/data-lake-storage-abfs-driver.md)
- [搭配 Azure HDInsight 叢集使用 Data Lake Storage Gen2](../hdinsight-hadoop-use-data-lake-storage-gen2.md)

## <a name="secure-azure-storage-keys-within-on-premises-hadoop-cluster-configuration"></a>保護內部部署 Hadoop 叢集組態內的 Azure 儲存體金鑰

新增至 Hadoop 設定檔的 Azure 儲存體金鑰，可建立內部部署 HDFS 與 Azure Blob 儲存體之間的連線能力。 這些金鑰可透過以 Hadoop 認證提供者架構進行加密而受到保護。 加密之後，即可安全地儲存及存取這些金鑰。

**若要佈建認證：**

```bash
hadoop credential create fs.azure.account.key.account.blob.core.windows.net -value <storage key> -provider jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks/file
```

**若要將上述提供者路徑新增至 core-site.xml 或自訂核心網站下的 Ambari 組態：**

```xml
<property>
    <name>hadoop.security.credential.provider.path</name>
        <value>
        jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks
        </value>
    <description>Path to interrogate for protected credentials.</description>
</property>
```

> [!Note]
> 您也可以將提供者路徑屬性新增至 distcp 命令列，而不是在叢集層級將金鑰儲存在 core-site.xml 上，如下所示：

```bash
hadoop distcp -D hadoop.security.credential.provider.path=jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks /user/user1/ wasb:<//yourcontainer@youraccount.blob.core.windows.net/>user1
```

## <a name="restrict-azure-storage-data-access-using-sas"></a>使用 SAS 限制 Azure 儲存體資料存取

根據預設，HDInsight 對於與叢集相關聯的 Azure 儲存體帳戶具有其資料的完整存取權。 Blob 容器上的共用存取簽章 (SAS) 可用來限制對資料的存取，例如，為使用者提供資料的唯讀存取權。

### <a name="using-the-sas-token-created-with-python"></a>使用以 Python 建立的 SAS 權杖

1. 開啟 [SASToken.py](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature/blob/master/Python/SASToken.py) 檔案並變更下列值：

    |權杖屬性|描述|
    |---|---|
    |policy_name|要建立的預存原則所要使用的名稱。|
    |storage_account_name|儲存體帳戶的名稱。|
    |storage_account_key|儲存體帳戶的金鑰。|
    |storage_container_name|您想要限制存取的儲存體帳戶中的容器。|
    |example_file_path|上傳至容器之檔案的路徑。|

2. SASToken.py 檔案附有 `ContainerPermissions.READ + ContainerPermissions.LIST` 權限，並且可根據使用案例進行調整。

3. 依照下列方式執行指令碼：`python SASToken.py`

4. 指令碼完成時，會顯示類似下列文字的 SAS 權杖：`sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14`

5. 若要使用共用存取簽章來限制對容器的存取，請在 Ambari HDFS 的 [設定] > [進階] > [自訂核心網站] > [新增屬性] 下，將自訂項目新增至叢集的核心網站組態。

6. 針對 [金鑰] 和 [值] 欄位使用下列值：

    機 **碼**： `fs.azure.sas.YOURCONTAINER.YOURACCOUNT.blob.core.windows.net` **值**： Python 應用程式從上述步驟4傳回的 SAS 金鑰。

7. 按一下 [新增] 按鈕以儲存這個金鑰和值，然後按一下 [儲存] 按鈕以儲存組態變更。 出現提示時，加入變更的描述 (例如，「新增 SAS 儲存體存取權」)，然後按一下 [儲存]。

8. 在 Ambari Web UI 中，選取左側清單中的 [HDFS]，然後從右側 [服務動作] 下拉式清單中選取 [重新啟動所有受影響項目]。 出現提示時，選取 [確認全部重新啟動]。

9. 對 MapReduce2 和 YARN 重複此程序。

在 Azure 中使用 SAS 權杖有三個重要事項要記住：

1. 以「讀取 + 列出」權限建立 SAS 權杖時，使用該 SAS 權杖存取 Blob 容器的使用者將無法「寫入和刪除」資料。 使用該 SAS 權杖存取 Blob 容器的使用者若嘗試寫入或刪除作業，將會收到類似於 `"This request is not authorized to perform this operation"` 的訊息。

2. 以 `READ + LIST + WRITE` 權限 (僅限制 `DELETE`) 產生 SAS 權杖時，會先將 `hadoop fs -put` 之類的命令寫入至 `\_COPYING\_` 檔案，然後嘗試將該檔案重新命名。 此 HDFS 作業對應於 WASB 的 `copy+delete`。 由於未 `DELETE` 提供許可權，因此「put」會失敗。 `\_COPYING\_` 作業是一項 Hadoop 功能，用以提供某種程度的並行存取控制。 目前沒有任何方法可以限制「刪除」作業，也不會影響「寫入」作業。

3. 可惜的是，hadoop 認證提供者和解密金鑰提供者 (ShellDecryptionKeyProvider) 目前無法使用 SAS 權杖，因此目前無法受到保護，無法從可見度中加以保護。

如需詳細資訊，請參閱[使用 Azure 儲存體共用存取簽章來限制 HDInsight 對資料的存取](../hdinsight-storage-sharedaccesssignature-permissions.md)。

## <a name="use-data-encryption-and-replication"></a>使用資料加密和複寫

所有寫入至「Azure 儲存體」的資料都會使用[儲存體服務加密 (SSE)](../../storage/common/storage-service-encryption.md)來自動加密。 Azure 儲存體帳戶中的資料一律會進行複寫以提供高可用性。 建立儲存體帳戶時，您可以選擇下列其中一個複寫選項：

- [本機備援儲存體 (LRS)](../../storage/common/storage-redundancy.md#locally-redundant-storage)
- [區域備援儲存體 (ZRS)](../../storage/common/storage-redundancy.md#zone-redundant-storage)
- [異地備援儲存體 (GRS)](../../storage/common/storage-redundancy.md#geo-redundant-storage)
- [讀取權限異地備援儲存體 (RA-GRS)](../../storage/common/storage-redundancy.md)

Azure 儲存體提供本機冗余的儲存體 (LRS) 但您也應該將重要資料複製到另一個區域中的另一個 Azure 儲存體帳戶，且頻率與嚴重損壞修復計畫的需求一致。 複製資料的方法有很多種，包括 [ADLCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md)、 [DistCp](https://hadoop.apache.org/docs/current/hadoop-distcp/DistCp.html)、 [Azure PowerShell](../../data-lake-store/data-lake-store-get-started-powershell.md)或 [Azure Data Factory](../../data-factory/connector-azure-data-lake-store.md)。 也建議您強制執行 Azure 儲存體帳戶的存取原則，以防止意外刪除。

如需詳細資訊，請參閱下列文章：

- [Azure 儲存體複寫](../../storage/common/storage-redundancy.md)
- [Azure Data Lake Storage Gen1 (ADLS) 的災難指引 ](../../data-lake-store/data-lake-store-disaster-recovery-guidance.md)

## <a name="attach-additional-azure-storage-accounts-to-cluster"></a>將其他 Azure 儲存體帳戶附加至叢集

在 HDInsight 建立程式期間，會將 Azure 儲存體帳戶、Azure Data Lake Storage Gen1 或 Azure Data Lake Storage Gen2 選為預設檔案系統。 除了這個預設的儲存體帳戶以外，也可以在叢集建立的過程中或在叢集建立之後，從相同或不同的 Azure 訂用帳戶中新增其他儲存體帳戶。

額外的儲存體帳戶可透過下列其中一種方法來新增：
- Ambari HDFS > [設定] > [進階] > [自訂核心網站] > [新增儲存體帳戶] > [名稱和金鑰] > [重新啟動服務]
- 藉由傳入儲存體帳戶的名稱和金鑰使用[指令碼動作](../hdinsight-hadoop-add-storage.md)

> [!Note]
> 在有效的使用案例中，您可以透過對 [Azure 支援](https://azure.microsoft.com/support/faq/)的要求增加 azure 儲存體的限制。

如需詳細資訊，請參閱[將其他儲存體帳戶新增至 HDInsight](../hdinsight-hadoop-add-storage.md)。

## <a name="next-steps"></a>後續步驟

閱讀本系列的下一篇文章： [內部部署到 Azure HDInsight Hadoop 遷移的資料移轉最佳做法](apache-hadoop-on-premises-migration-best-practices-data-migration.md)。
