---
title: 從以 Windows 為基礎的 HDInsight 移轉至以 Linux 為基礎的 HDInsight - Azure
description: 了解如何從以 Windows 為基礎的 HDInsight 叢集移轉至以 Linux 為基礎的 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: jasonh
ms.openlocfilehash: ee97d2005752c9e0cd40de238d1f4a946bb9ee3d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948412"
---
# <a name="migrate-from-a-windows-based-hdinsight-cluster-to-a-linux-based-cluster"></a>從以 Windows 為基礎的 HDInsight 叢集移轉至以 Linux 為基礎的叢集

本文件提供 Windows 和 Linux 上的 HDInsight 之間差異的詳細資料。 它也提供有關如何將現有工作負載移轉至以 Linux 作為基礎之叢集的指引。

儘管以 Windows 為基礎的 HDInsight 提供一種簡單方式來使用雲端中的 Hadoop，但您可能需要移轉到以 Linux 為基礎的叢集。 例如，充分利用您解決方案所需且以 Linux 為基礎的工具和技術。 Hadoop 生態系統的許多內容都是在以 Linux 為基礎的系統上開發，因此可能無法與以 Windows 為基礎的 HDInsight 搭配使用。 許多針對 Hadoop 的書籍、影片及其他訓練材料，都會假設您正在使用 Linux 系統。

> [!NOTE]
> HDInsight 叢集使用 Ubuntu 長期支援 (LTS) 做為叢集中節點的作業系統。 如需 HDInsight 中可用 Ubuntu 版本的相關資訊以及其他元件版本設定資訊，請參閱 [HDInsight 元件版本](hdinsight-component-versioning.md)。

## <a name="migration-tasks"></a>移轉工作

下列為移轉的一般工作流程。

![移轉工作流程圖表](./media/hdinsight-migrate-from-windows-to-linux/workflow.png)

1. 閱讀此文件的每一節，以了解在移轉時，可能需要進行的變更。

2. 建立以 Linux 為基礎的叢集做為測試/品質保證環境。 如需建立以 Linux 為基礎之叢集的詳細資訊，請參閱 [在 HDInsight 中建立以 Linux 為基礎的叢集](hdinsight-hadoop-provision-linux-clusters.md)。

3. 將現有的作業、資料來源與接收複製到新的環境。

4. 執行驗證測試以確保您的工作在新叢集上會如預期般運作。

當您已驗證一切都會如預期般運作之後，請為移轉排定停機時間。 在停機期間，執行下列動作：

1. 備份所有儲存在本機叢集節點上的暫時性資料。 例如，如果您的資料是直接儲存在前端節點上。

2. 刪除以 Windows 為基礎的叢集。

3. 使用和以 Windows 為基礎之叢集所使用的相同預設資料存放區，建立以 Linux 為基礎的叢集。 以 Linux 為基礎的叢集可以針對現有的生產資料繼續運作。

4. 匯入任何已備份的暫時性資料。

5. 使用新叢集啟動工作/繼續處理。

### <a name="copy-data-to-the-test-environment"></a>將資料複製到測試環境

雖然複製資料和工作的方法有很多，但是本區段所討論的兩個方法，是將檔案直接移至測試叢集最簡單的方法。

#### <a name="hdfs-copy"></a>HDFS 複製

使用下列步驟，將資料從生產環境叢集複製到測試叢集。 這些步驟使用 HDInsight 隨附的 `hdfs dfs` 公用程式。

1. 尋找現有叢集的儲存體帳戶和預設容器資訊。 下列範例使用 PowerShell 來擷取此資訊：

    ```powershell
    $clusterName="Your existing HDInsight cluster name"
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    write-host "Storage account name: $clusterInfo.DefaultStorageAccount.split('.')[0]"
    write-host "Default container: $clusterInfo.DefaultStorageContainer"
    ```

2. 若要建立測試環境，請依照＜在 HDInsight 中建立以 Linux 為基礎的叢集＞文件中的步驟執行。 於建立叢集之前停止遵循步驟，並改為選取 [選擇性組態] 。

3. 從 [選擇性組態] 區段中，選取 [連結的儲存體帳戶]。

4. 選取 [新增儲存體金鑰] ，並在出現提示時，選取步驟 1 中由 PowerShell 指令碼傳回的儲存體帳戶。 按一下每個區段中的 [選取]。 最後，建立叢集。

5. 建立叢集之後，使用 **SSH** 來連線至該叢集。 如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

6. 從 SSH 工作階段中，使用下列命令來將檔案從已連結的儲存體帳戶複製到新的預設儲存體帳戶。 使用 PowerShell 傳回的容器資訊來取代 CONTAINER。 使用帳戶名稱來取代 __ACCOUNT__。 將資料路徑取代為資料檔案路徑。

    ```bash
    hdfs dfs -cp wasb://CONTAINER@ACCOUNT.blob.core.windows.net/path/to/old/data /path/to/new/location
    ```

    > [!NOTE]
    > 如果包含資料的目錄結構不存在於測試環境，您可以使用下列命令建立它：

    ```bash
    hdfs dfs -mkdir -p /new/path/to/create
    ```

    `-p` 參數可讓您在路徑中建立所有目錄。

#### <a name="direct-copy-between-blobs-in-azure-storage"></a>在 Azure 儲存體中的 Blob 之間直接複製

此外，您也可能會想要使用 `Start-AzureStorageBlobCopy` Azure PowerShell Cmdlet 在 HDInsight 之外的儲存體帳戶之間複製 Blob。 如需詳細資訊，請參閱＜搭配使用 Azure PowerShell 與 Azure 儲存體＞一文中的＜如何管理 Azure Blob＞一節。

## <a name="client-side-technologies"></a>用戶端技術

[Azure PowerShell Cmdlet](/powershell/azureps-cmdlets-docs)、[Azure 傳統 CLI](../cli-install-nodejs.md) 或 [.NET SDK for Hadoop](https://hadoopsdk.codeplex.com/) 之類的用戶端技術會繼續使用以 Linux 為基礎的叢集。 這些依賴 REST API 的技術在兩種叢集作業系統類型上都相同。

## <a name="server-side-technologies"></a>伺服器端技術

下表提供移轉 Windows 特定之伺服器端元件的指導方針。

| 如果您正在使用這項技術... | 請執行此動作... |
| --- | --- |
| **PowerShell** (伺服器端指令碼，包含於叢集建立期間使用的指令碼動作) |重寫為 Bash 指令碼。 針對指令碼動作，請參閱[使用指令碼動作自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)和[以 Linux 為基礎之 HDInsight 的指令碼動作開發](hdinsight-hadoop-script-actions-linux.md)。 |
| **Azure 傳統 CLI** (伺服器端指令碼) |雖然 Azure 傳統 CLI 可在 Linux 上使用，它並沒有預先安裝在 HDInsight 叢集前端節點上。 如需有關安裝 Azure 傳統 CLI 的詳細資訊，請參閱[開始使用 Azure 傳統 CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli)。 |
| **.NET 元件** |以 Linux 為基礎的 HDInsight 透過 [Mono](https://mono-project.com) 支援 .NET。 如需詳細資訊，請參閱[將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md)。 |
| **Win32 元件或其他僅限 Windows 的技術** |指導方針將視元件或技術而有所不同。 您可以尋找與 Linux 相容的版本。 如果沒有，您就必須找出替代解決方案或重新撰寫此元件。 |

> [!IMPORTANT]
> HDInsight 管理 SDK 不完全相容 Mono。 請勿使用它作為部署到 HDInsight 叢集的解決方案一部分。

## <a name="cluster-creation"></a>叢集建立

本節將提供叢集建立之差異的資訊。

### <a name="ssh-user"></a>SSH 使用者

以 Linux 為基礎的 HDInsight 是使用 **安全殼層 (SSH)** 通訊協定來為叢集節點提供遠端存取功能。 不同於以 Windows 為基礎之叢集的遠端桌面，大部分的 SSH 用戶端不提供圖形化使用者體驗。 SSH 用戶端改為提供命令列，讓您在叢集上執行命令。 某些用戶端 (例如 [MobaXterm](http://mobaxterm.mobatek.net/)) 除了提供遠端命令列之外，也提供圖形化檔案系統瀏覽器。

在叢集建立期間，您必須提供 SSH 使用者，以及**密碼**或**公開金鑰憑證**以進行驗證。

我們建議使用公開金鑰憑證，因為它比密碼更安全。 憑證驗證會產生已簽署的公開/私人金鑰組，然後在建立叢集時提供公開金鑰。 使用 SSH 連線至伺服器時，用戶端上的私人金鑰將會為連線提供驗證。

如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

### <a name="cluster-customization"></a>叢集自訂

**指令碼動作** 必須以 Bash 指令碼撰寫。 以 Linux 作為基礎的叢集可以在建立叢集期間或之後使用指令碼動作。 如需詳細資訊，請參閱[使用指令碼動作自訂 Linux 型 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)和[以 Linux 為基礎之 HDInsight 的指令碼動作開發](hdinsight-hadoop-script-actions-linux.md)。

另一個自訂功能是 **bootstrap**。 針對 Windows 叢集，此功能可讓您指定其他搭配 Hive 使用之程式庫的位置。 在叢集建立之後，這些程式庫將可自動搭配 Hive 查詢使用，而不需使用 `ADD JAR`。

針對以 Linux 為基礎的叢集，Bootstrap 功能不提供此功能。 請改為使用 [在叢集建立期間新增 Hive 程式庫](hdinsight-hadoop-add-hive-libraries.md)中所記錄的指令碼動作。

### <a name="virtual-networks"></a>虛擬網路

以 Windows 為基礎的 HDInsight 僅支援傳統虛擬網路，而以 Linux 為基礎的 HDInsight 則需要資源管理員虛擬網路。 如果資源位於傳統虛擬網路中，且以 Linux 為基礎的 HDInsight 叢集必須連接到這類資源時，請參閱 [將傳統虛擬網路連接到 Resource Manager 虛擬網路](../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md)。

如需設定需求的詳細資訊，請參閱[使用虛擬網路延伸 HDInsight 功能](hdinsight-extend-hadoop-virtual-network.md)文件。

## <a name="management-and-monitoring"></a>管理與監視

有許多您可能曾搭配以 Windows 為基礎之 HDInsight 使用的 Web UI (例如工作歷程記錄或 Yarn UI)，皆可透過 Ambari 使用。 此外，Ambari Hive 檢視能提供使用您的網頁瀏覽器執行 Hive 查詢的方法。 Ambari Web UI 可以在以 Linux 為基礎的叢集上使用，位於： https://CLUSTERNAME.azurehdinsight.net。

如需使用 Ambari 的詳細資訊，請參閱下列文件：

* [Ambari Web](hdinsight-hadoop-manage-ambari.md)
* [Ambari REST API](hdinsight-hadoop-manage-ambari-rest-api.md)

### <a name="ambari-alerts"></a>Ambari 警示

Ambari 擁有能通知您叢集潛在問題的警示系統。 警示將會以紅色或黃色的項目出現在 Ambari Web UI 中，您也可以透過 REST API 擷取它們。

> [!IMPORTANT]
> Ambari 警示代表「可能有」問題，而不表示「已發生」問題。 例如，您可能會收到無法存取 HiveServer2 的警示，但實際上您仍然可以正常存取它。
>
> 許多警示都是針對某項服務實作為以間隔為基礎的查詢，並會預期在特定的時間範圍內收到回應。 因此警示本身並不代表服務已關閉，而只是單純表示該服務沒有在預期的時間範圍內傳回結果。

## <a name="file-system-locations"></a>檔案系統位置

Linux 叢集檔案系統的展開方式和以 Windows 為基礎的 HDInsight 叢集不同。 使用下列表格來尋找常用的檔案。

| 我需要尋找... | 它位於... |
| --- | --- |
| 組態 |`/etc`。 例如， `/etc/hadoop/conf/core-site.xml` |
| 記錄檔 |`/var/logs` |
| Hortonworks Data Platform (HDP) |`/usr/hdp`。此處有兩個目錄，一個是目前的 HDP 版本，另一個則是 `current`。 `current` 目錄包含位於版本號碼目錄之檔案和目錄的符號連結。 `current` 目錄的用途是讓您方便存取 HDP 檔案，因為版本號碼會在更新 HDP 版本時變更。 |
| hadoop-streaming.jar |`/usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar` |

通常來說，如果您知道檔案的名稱，便可以使用下列來自 SSH 工作階段的命令來尋找檔案路徑：

    find / -name FILENAME 2>/dev/null

您也可以搭配檔案名稱使用萬用字元。 例如，`find / -name *streaming*.jar 2>/dev/null` 會傳回任何檔案名稱包含 'streaming' 這個字的 jar 檔案路徑。

## <a name="hive-pig-and-mapreduce"></a>Hive、Pig 及 MapReduce

Pig 和 MapReduce 工作負載在 Linux 為基礎的叢集上很相似。 不過，您可以使用較新版本的 Hadoop、Hive 和 Pig 建立以 Linux 為基礎的 HDInsight 叢集。 這些版本的差異可能會對您現有方案的運作方式造成變更。 如需 HDInsight 包含之元件的版本詳細資訊，請參閱 [HDInsight 元件版本設定](hdinsight-component-versioning.md)。

以 Linux 為基礎的 HDInsight 不提供遠端桌面功能。 您可以改用 SSH 來遠端連線至叢集前端節點。 如需詳細資訊，請參閱下列文件：

* [搭配 SSH 使用 Hive](hdinsight-hadoop-use-hive-ssh.md)
* [搭配 SSH 使用 Pig](hadoop/apache-hadoop-use-pig-ssh.md)
* [搭配 SSH 使用 MapReduce](hadoop/apache-hadoop-use-mapreduce-ssh.md)

### <a name="hive"></a>Hive

> [!IMPORTANT]
> 如果您使用外部 Hive 中繼存放區，您應該先備份中繼存放區，然後才將它搭配以 Linux 為基礎的 HDInsight 使用。 以 Linux 為基礎的 HDInsight 可搭配較新版本的 Hive 使用，但可能會與舊版建立的中繼存放區不相容。

下列圖表提供移轉 Hive 工作負載的指導方針。

| 以 Windows 為基礎時，我是使用... | 以 Linux 為基礎時... |
| --- | --- |
| **Hive 編輯器** |[Ambari 中的 Hive 檢視](hadoop/apache-hadoop-use-hive-ambari-view.md) |
| `set hive.execution.engine=tez;` 以啟用 Tez |Tez 是以 Linux 為基礎之叢集的預設執行引擎，因此已不再需要 SET 陳述式。 |
| C# 使用者定義函數 | 如需驗證以 Linux 為基礎之 HDInsight 的 C# 元件詳細資訊，請參閱[將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md) |
| 伺服器上的 CMD 檔案或指令碼是做為 Hive 工作的一部分進行叫用 |使用 Bash 指令碼 |
| `hive` 命令 |使用 [Beeline](hadoop/apache-hadoop-use-hive-beeline.md) 或是[來自 SSH 工作階段的 Hive](hdinsight-hadoop-use-hive-ssh.md) |

### <a name="pig"></a>Pig

| 以 Windows 為基礎時，我是使用... | 以 Linux 為基礎時... |
| --- | --- |
| C# 使用者定義函數 | 如需驗證以 Linux 為基礎之 HDInsight 的 C# 元件詳細資訊，請參閱[將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md) |
| 伺服器上的 CMD 檔案或指令碼是做為 Pig 作業的一部分叫用 |使用 Bash 指令碼 |

### <a name="mapreduce"></a>MapReduce

| 以 Windows 為基礎時，我是使用... | 以 Linux 為基礎時... |
| --- | --- |
| C# 對應器和歸納器元件 | 如需驗證以 Linux 為基礎之 HDInsight 的 C# 元件詳細資訊，請參閱[將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md) |
| 伺服器上的 CMD 檔案或指令碼是做為 Hive 工作的一部分進行叫用 |使用 Bash 指令碼 |

## <a name="oozie"></a>Oozie

> [!IMPORTANT]
> 如果您使用外部 Oozie 中繼存放區，您應該先備份中繼存放區，然後才將它搭配以 Linux 為基礎的 HDInsight 使用。 以 Linux 為基礎的 HDInsight 可搭配較新版本的 Oozie 使用，但可能會與舊版建立的中繼存放區不相容。

Oozie 工作流程允許殼層動作。 殼層動作會使用作業系統的預設殼層來執行命令列命令。 如果您有依賴 Windows 殼層的 Oozie 工作流程，您必須重新撰寫工作流程，以依賴 Linux 殼層環境 (Bash)。 如需使用 Oozie 殼層動作的詳細資訊，請參閱 [Oozie 殼層動作擴充功能](http://oozie.apache.org/docs/3.3.0/DG_ShellActionExtension.html)。

如果您有使用 C# 應用程式的工作流程，請在 Linux 環境中驗證這些應用程式。 如需詳細資訊，請參閱[將 .NET 方案移轉至以 Linux 為基礎的 HDInsight](hdinsight-hadoop-migrate-dotnet-to-linux.md)。

## <a name="storm"></a>Storm

| 以 Windows 為基礎時，我是使用... | 以 Linux 為基礎時... |
| --- | --- |
| Storm Dashboard |無法使用 Storm Dashboard。 請參閱 [在以 Linux 為基礎的 HDInsight 上部署與管理 Storm 拓撲](storm/apache-storm-deploy-monitor-topology-linux.md) ，以了解提交拓撲的方法。 |
| Storm UI |Storm UI 位於 https://CLUSTERNAME.azurehdinsight.net/stormui |
| Visual Studio 以建立、部署及管理 C# 或混合式拓撲 |在以 Linux 作為基礎的 Storm on HDInsight 上，可以使用 Visual Studio 來建立、部署和管理 C# (SCP.NET) 或混合式拓撲。 僅能搭配使用 10/28/2016 之後建立的叢集。 |

## <a name="hbase"></a>hbase

在以 Linux 為基礎的叢集上，HBase 的 znode 父項目為 `/hbase-unsecure`。 針對任何使用原生 HBase Java API 的 Java 用戶端應用程式，在組態中設定此值。

如需設定此值的範例用戶端，請參閱 [建置以 Java 為基礎的 HBase 應用程式](hdinsight-hbase-build-java-maven.md) 。

## <a name="spark"></a>Spark

Spark 叢集可以在 Windows 叢集預覽期間取得。 Spark GA 只適用於以 Linux 為基礎的叢集。 以 Windows 為基礎的 Spark 預覽叢集和以 Linux 為基礎的 Spark 叢集之間並沒有移轉路徑。

## <a name="known-issues"></a>已知問題

### <a name="azure-data-factory-custom-net-activities"></a>Azure Data Factory 自訂 .NET 活動

Azure Data Factory 自訂 .NET 活動目前並不受以 Linux 為基礎的 HDInsight 叢集所支援。 您應該改為使用下列其中一個方法，來將自訂活動實作為 ADF 管線的一部分。

* 在 Azure Batch 集區上執行 .NET 活動。 請參閱 [在 Azure Data Factory 管線中使用自訂活動](../data-factory/transform-data-using-dotnet-custom-activity.md)
* 將活動實作為 MapReduce 活動。 如需詳細資訊，請參閱[從 Data Factory 叫用 MapReduce 程式](../data-factory/transform-data-using-hadoop-map-reduce.md)。

### <a name="line-endings"></a>行尾結束符號

通常來說，以 Windows 為基礎之系統上的行尾結束符號是使用 CRLF，而以 Linux 為基礎的系統則使用 LF。 您可能需要修改現有的資料產生者和取用者才能使用 LF。

例如，使用 Azure PowerShell，在以 Windows 為基礎的叢集上查詢 HDInsight，會傳回擁有 CRLF 的資料。 在以 Linux 為基礎的叢集上使用相同查詢，會傳回 LF。 請先進行測試，查看行尾結束符號否會導致您的方案發生問題，再將它移轉到以 Linux 作為基礎的叢集。

一律使用 LF 作為叢集節點上執行之指令碼的結束行。 如果您使用 CRLF，便可能會在以 Linux 為基礎的叢集上執行指令碼時遭遇到錯誤。

如果指令碼並沒有包含擁有內嵌 CR 字元的字串，您可以使用下列其中一種方法來大量變更行尾結束符號：

* **上傳至叢集之前**：請在將指令碼上傳至叢集之前，使用下列 PowerShell 陳述式將行尾結束符號從 CRLF 變更為 LF。

    ```powershell
    $original_file ='c:\path\to\script.py'
    $text = [IO.File]::ReadAllText($original_file) -replace "`r`n", "`n"
    [IO.File]::WriteAllText($original_file, $text)
    ```

* **上傳至叢集之後**：請從以 Linux 為基礎之叢集的 SSH 工作階段使用下列命令來修改指令碼。

    ```bash
    hdfs dfs -get wasb:///path/to/script.py oldscript.py
    tr -d '\r' < oldscript.py > script.py
    hdfs dfs -put -f script.py wasb:///path/to/script.py
    ```

## <a name="next-steps"></a>後續步驟

* [了解如何建立 Linux 型 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)
* [使用 SSH 連線到 HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md)
* [使用 Ambari 管理 Linux 型叢集](hdinsight-hadoop-manage-ambari.md)
