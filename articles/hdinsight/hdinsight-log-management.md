---
title: 管理 HDInsight 叢集的記錄 - Azure HDInsight
description: 判斷 HDInsight 活動記錄檔的類型、大小及保留原則。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/11/2018
ms.author: ashishth
ms.openlocfilehash: 339d5d39c637369420e197acf65df802cefd5cb9
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988479"
---
# <a name="manage-logs-for-an-hdinsight-cluster"></a>管理 HDInsight 叢集的記錄

HDInsight 叢集會產生各式各樣的記錄檔。 例如 Apache Hadoop 及相關服務 (例如 Apache Spark) 會產生詳細的作業執行記錄。 記錄檔管理是讓 HDInsight 叢集保持良好狀況的一環。 此外，可能也有將記錄封存的法規需求。  由於對記錄檔會有數目和大小方面的考量，因此將記錄儲存體最佳化和封存會有助於管理服務成本。

管理 HDInsight 叢集記錄包括保留有關叢集環境所有方面的資訊。 此資訊包括所有關聯的 Azure 服務記錄、叢集組態、作業執行資訊、任何錯誤狀態，以及其他所需的資料。

典型的 HDInsight 記錄管理步驟包括：

* 步驟 1：判斷記錄保留原則
* 步驟 2：管理叢集服務版本組態記錄
* 步驟 3：管理叢集作業執行記錄檔
* 步驟 4：預測記錄磁碟區儲存體大小和成本
* 步驟 5：判斷記錄封存原則和程序

## <a name="step-1-determine-log-retention-policies"></a>步驟 1：判斷記錄保留原則

建立 HDInsight 叢集記錄管理策略的第一步，就是收集有關業務案例和作業執行歷程記錄儲存體需求的資訊。

### <a name="cluster-details"></a>叢集詳細資料

下列叢集詳細資料相當實用，可協助收集您記錄管理策略中的資訊。 請從您已在特定 Azure 帳戶中建立的所有 HDInsight 叢集收集此資訊。

* 叢集名稱
* 叢集區域和 Azure 可用性區域
* 叢集狀態，包括上次狀態變更的詳細資料
* 為主要、核心及工作節點指定的 HDInsight 執行個體類型和數目

您可以使用 Azure 入口網站來取得此最上層資訊的大部分。  或者，您也可以使用 Azure 傳統 CLI 來取得有關 HDInsight 叢集的資訊：

```
    azure hdinsight cluster list
    azure hdinsight cluster show <ClusterName>
```
[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

您還可以使用 PowerShell 來檢視此資訊。  如需詳細資訊，請參閱[使用 Azure PowerShell 來管理 HDInsight 中的 Hadoop 叢集](hdinsight-administer-use-powershell.md)。

### <a name="understand-the-workloads-running-on-your-clusters"></a>了解在您叢集上執行的工作負載

您必須了解在 HDInsight 叢集上執行的工作負載類型，才能為每一種類型設計出適當的記錄策略。

* 工作負載是實驗性的 (例如開發或測試)，還是生產環境性質的？
* 生產環境性質工作負載一般多久執行一次？
* 是否有任何工作負載需要大量資源和/或長時間執行？
* 是否有任何工作負載使用系統會為其產生多種記錄的一組複雜 Hadoop 服務？
* 是否有任何工作負載有相關規定的執行歷程需求？

### <a name="example-log-retention-patterns-and-practices"></a>範例記錄保留模式和做法

* 考量藉由為每個記錄項目新增識別碼或透過其他技術，來維護資料歷程追蹤。 這可讓您回溯至資料和作業的原始來源，並追蹤資料至每個階段，以了解資料是否一致和有效。

* 考量如何從該叢集或從多個叢集收集記錄，然後整理這些記錄以供稽核、監視、規劃及警示用途使用。 您可以使用自訂的解決方案來定期存取和下載記錄檔，然後對記錄檔進行結合和分析以在儀表板上顯示。 您也可以新增額外的功能以進行安全性警示或失敗偵測。 您可以使用 PowerShell、HDInsight SDK 或可存取 Azure 傳統部署模型的程式碼，來建置這些公用程式。

* 考量監視解決方案或服務是否能提供實用的效益。 Microsoft System Center 提供 [HDInsight 管理組件](https://www.microsoft.com/download/details.aspx?id=42521)。 您也可以使用協力廠商工具 (例如 Chukwa 和 Ganglia) 來收集和集中記錄。 許多公司都有提供可監視 Hadoop 型巨量資料解決方案的服務，例如 Centerity、Compuware APM、Sematext SPM 及 Zettaset Orchestrator。

## <a name="step-2-manage-cluster-service-versions-and-view-script-action-logs"></a>步驟 2：管理叢集服務版本及檢視指令碼動作記錄

典型的 HDInsight 叢集會使用數個服務和開放原始碼軟體套件 (例如 Apache HBase 和 Apache Spark 等)。 針對某些工作負載 (例如生物資訊學)，您可能除了作業執行記錄之外，還必須保留服務組態記錄歷程記錄。

### <a name="view-cluster-configuration-settings-with-the-ambari-ui"></a>使用 Ambari UI 來檢視叢集組態設定

Apache Ambari 提供 Web UI 和 REST API，可簡化 HDInsight 叢集的管理、組態及監視作業。 以 Linux 為基礎的 HDInsight 叢集會隨附 Ambari。 請在 Azure 入口網站 HDInsight 頁面上，選取 [叢集儀表板] 窗格以開啟 [叢集儀表板] 連結頁面。  接著，選取 [HDInsight 叢集儀表板] 窗格以開啟 Ambari UI。  系統會提示您輸入叢集登入認證。

若要開啟服務檢視清單，請在 Azure 入口網站的 HDInsight 頁面上選取 [Ambari 檢視]。  此清單會依您已安裝的程式庫而有所不同。  例如，您可能會看到 [YARN 佇列管理員]、[Hive 檢視] 及 [Tez 檢視]。  請選取任何服務連結來查看組態和服務資訊。  Ambari UI [Stack and Version] \(堆疊與版本\) 頁面會提供有關叢集服務組態與服務版本歷程記錄的資訊。 若要瀏覽至 Ambari UI 的這個區段，請選取 [Admin] \(系統管理\) 功能表，然後選取 [Stack and Version] \(堆疊與版本\)。  選取 [Versions] \(版本\) 索引標籤以查看服務版本資訊。

![堆疊與版本](./media/hdinsight-log-management/stack-versions.png)

使用 Ambari UI 時，您可以下載在叢集中特定主機 (或節點) 上執行之任何 (或所有) 服務的組態。  請選取 [Hosts] \(主機\) 功能表，然後選取感興趣之主機的連結。 在該主機的頁面上，選取 [Host Actions] \(主機動作\) 按鈕，然後選取 [Download Client Configs] \(下載用戶端組態\)。 

![主機用戶端組態](./media/hdinsight-log-management/client-configs.png)

### <a name="view-the-script-action-logs"></a>檢視指令碼動作記錄

HDInsight [指令碼動作](hdinsight-hadoop-customize-cluster-linux.md)會以手動方式或在指定時，於叢集上執行指令碼。 例如，指令碼動作可用來在叢集上安裝額外的軟體，或是更改組態設定的預設值。 指令碼動作記錄可讓您深入了解在設定叢集時發生的錯誤，還可了解可能影響叢集效能和可用性的組態設定變更。  若要查看指令碼動作的狀態，請選取 Ambari UI 上的 [ops] 按鈕，或是存取預設儲存體帳戶中的狀態記錄。 儲存體記錄檔位於 `/STORAGE_ACCOUNT_NAME/DEFAULT_CONTAINER_NAME/custom-scriptaction-logs/CLUSTER_NAME/DATE`。

## <a name="step-3-manage-the-cluster-job-execution-log-files"></a>步驟 3：管理叢集作業執行記錄檔

下一步是檢閱各種服務的作業執行記錄檔。  服務可能包括 Apache HBase、Apache Spark 及許多其他服務。 Hadoop 叢集會產生大量的詳細資訊記錄，因此判斷哪些記錄有用 (以及哪些記錄沒用) 會相當耗時。  對於目標性的記錄檔管理來說，了解記錄系統相當重要。  以下是一個範例記錄檔。

![HDInsight 記錄檔範例](./media/hdinsight-troubleshoot-failed-cluster/logs.png)

### <a name="access-the-hadoop-log-files"></a>存取 Hadoop 記錄檔

HDInsight 會將其記錄檔同時儲存在叢集檔案系統和 Azure 儲存體中。 您可以開啟與叢集的 SSH 連線並瀏覽檔案系統，或是使用遠端前端節點伺服器上的 Hadoop YARN 狀態入口網站，來檢查叢集中的記錄檔。 您可以使用任何能夠存取和下載 Azure 儲存體中資料的工具，來檢查 Azure 儲存體中的記錄檔。 範例包括 AZCopy、CloudXplorer 及「Visual Studio 伺服器總管」。 您也可以使用 PowerShell 和「Azure 儲存體用戶端」程式庫或 Azure .NET SDK，來存取 Azure Blob 儲存體中的資料。

Hadoop 會將作業的工作以「工作嘗試」的形式在叢集中的各種節點上執行。 HDInsight 可以起始理論式工作嘗試，其中會先終止所有其他未完成的工作嘗試。 這會產生將即時記錄至控制器、stderr 及 syslog 記錄檔的重要活動。 此外，多個工作嘗試會同時執行，但記錄檔只能以線性方式顯示結果。

#### <a name="hdinsight-logs-written-to-azure-blob-storage"></a>寫入至 Azure Blob 儲存體的 HDInsight 記錄檔

HDInsight 叢集已設定成針對使用 Azure PowerShell Cmdlet 或 .NET 作業提交 API 來提交的所有工作，將工作記錄寫入至 Azure Blob 儲存體帳戶。  如果您透過 SSH 連線至叢集來提交作業，則執行記錄資訊會如上節所述儲存在 Azure 資料表中。

除了 HDInsight 所產生的核心記錄檔之外，已安裝的服務 (例如 YARN) 也會產生作業執行記錄檔。  記錄檔的數目和類型會依所安裝的服務而有所不同。  常見的服務包括 Apache HBase、Apache Spark 等。  請調查每個服務的作業記錄執行檔案，以了解您叢集上可用的全部記錄檔。  每個服務都有自己的獨特記錄方法和記錄檔儲存位置。  為舉例說明，在下一節中，將會詳細探討從 YARN 存取最常見的服務記錄檔。

### <a name="hdinsight-logs-generated-by-yarn"></a>YARN 所產生的 HDInsight 記錄檔

YARN 會彙總背景工作節點上所有容器的記錄，然後將這些記錄依每個背景工作節點儲存成單一彙總記錄檔。 該記錄會在應用程式完成之後，儲存在預設的檔案系統上。 您的應用程式可以使用數百或數千個容器，但在單一背景工作節點上執行之所有容器的記錄一律會彙總成單一檔案。 您應用程式使用的每一背景工作節點只會有一個記錄。 在 HDInsight 叢集 3.0 版和更新版本上，預設會啟用記錄彙總。 彙總記錄位於叢集的預設儲存體。

```
    /app-logs/<user>/logs/<applicationId>
```

您無法直接閱讀彙總的記錄，因為其撰寫格式是由容器編製索引的 TFile 二進位格式。 對於您感興趣的應用程式或容器，請使用 YARN ResourceManager 記錄或 CLI 工具來檢視純文字記錄。

#### <a name="yarn-cli-tools"></a>YARN CLI 工具

若要使用 YARN CLI 工具，您必須先使用 SSH 連接到 HDInsight 叢集。 請在執行這些命令時，指定 `<applicationId>``<user-who-started-the-application>``<containerId>`及 `<worker-node-address>` 資訊。 您可以使用下列其中一個命令，以純文字的形式檢視這些記錄：

```bash
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>
```

#### <a name="yarn-resourcemanager-ui"></a>YARN ResourceManager UI

YARN ResourceManager UI 會在叢集前端節點上執行，您可以透過 Ambari Web UI 來存取它。 請使用下列步驟來檢視 YARN 記錄：

1. 在網頁瀏覽器中，瀏覽至 `https://CLUSTERNAME.azurehdinsight.net`。 將 CLUSTERNAME 取代為 HDInsight 叢集的名稱。
2. 從左邊的服務清單中，選取 [YARN]。
3. 從 [Quick Links] \(快速連結\) 下拉式清單中，選取其中一個叢集前端節點，然後選取 [ResourceManager logs] \(ResourceManager 記錄\)。 您會看到一份 YARN 記錄檔的連結清單。

## <a name="step-4-forecast-log-volume-storage-sizes-and-costs"></a>步驟 4：預測記錄磁碟區儲存體大小和成本

完成上述步驟之後，您便會了解 HDInsight 叢集所要產生之記錄檔的類型和磁碟區。

接著，請分析一段時間內主要記錄儲存體位置中的記錄資料磁碟區。 例如，您可以分析 30-60-90 天期間內的磁碟區和成長情況。  請將此資訊記錄在試算表中，或使用其他工具 (例如 Visual Studio、「Azure 儲存體總管」或 Power Query for Excel) 來記錄此資訊。 如需詳細資訊，請參閱[分析 HDInsight 記錄](hdinsight-debug-jobs.md)。  

您現在已有足夠的資訊來建立主要記錄的記錄管理策略。  請使用您的試算表 (或所選擇的工具) 來預測記錄大小成長及 Azure 服務成本增加的記錄儲存體。  此外，針對您要檢查的一組記錄，也請考量是否有任何記錄保留需求。  在判斷哪些記錄檔可以刪除 (如果有的話) 及哪些記錄應該保留並封存至較低廉的 Azure 儲存體之後，現在您即可重新預測未來的記錄儲存體成本。

## <a name="step-5-determine-log-archive-policies-and-processes"></a>步驟 5：判斷記錄封存原則和程序

在判斷哪些記錄檔可以刪除之後，您可以調整許多 Hadoop 服務上的記錄參數，以在指定的期間之後，便自動刪除記錄檔。

針對特定的記錄檔，您可以使用較低廉的記錄檔封存方法。 針對 Azure Resource Manager 活動記錄，您可以使用 Azure 入口網站來探索此功能。  請在 HDInsight 執行個體的 Azure 入口網站中選取 [活動記錄] 連結，來設定 ARM 記錄的封存。  在 [活動記錄] 搜尋頁面的頂端，選取 [匯出] 功能表項目來開啟 [匯出活動記錄] 窗格。  填入訂用帳戶、區域、是否要匯出至儲存體帳戶，以及記錄的保留天數。 在這同一個窗格上，您也可以指出是否要匯出至事件中樞。 

![匯出記錄檔](./media/hdinsight-log-management/archive.png)

或者，您也可以使用 PowerShell 來撰寫記錄封存指令碼。  如需範例 PowerShell 指令碼，請參閱[將 Azure 自動化記錄封存至 Azure Blob 儲存體](https://gallery.technet.microsoft.com/scriptcenter/Archive-Azure-Automation-898a1aa8)(英文\)。

### <a name="accessing-azure-storage-metrics"></a>存取 Azure 儲存體計量

您可以設定讓 Azure 儲存體記錄儲存體作業和存取。 您可以使用這些非常詳細的記錄來監視和規劃容量，以及稽核對儲存體發出的要求。 所記錄的資訊包括延遲詳細資料，可讓您監視和微調解決方案的效能。
您可以使用適用於 Hadoop 的 .NET SDK，來檢查針對儲存 HDInsight 叢集之資料的 Azure 儲存體產生的記錄檔。

### <a name="control-the-size-and-number-of-backup-indexes-for-old-log-files"></a>控制舊記錄檔的備份索引大小和數目

若要控制所保留之記錄檔的大小和數目，請設定 `RollingFileAppender` 的下列屬性：

* `maxFileSize` 是檔案的關鍵大小，超出此大小時就會變換檔案。 預設值是 10 MB。
* `maxBackupIndex` 指定要建立的備份檔案數目，預設值為 1。

### <a name="other-log-management-techniques"></a>其他記錄管理技術

若要避免磁碟空間用完，您可以使用一 OS 工具 (例如 `logrotate`) 來管理記錄檔的處理。 您可以設定讓 `logrotate` 每天執行，以壓縮記錄檔並移除舊記錄檔。 您的方法取決於您的需求，例如要在本機節點上保留記錄檔多久。 

您也可以檢查是否已針對一或多個服務啟用 DEBUG 記錄功能，此功能會大幅增加輸出記錄大小。 

若要將記錄從所有節點收集到一個集中位置，您可以建立資料流程，例如將所有記錄項目擷取至 Solr。

## <a name="next-steps"></a>後續步驟

* [HDInsight 的監視和記錄做法](https://msdn.microsoft.com/library/dn749790.aspx) \(英文\)
* [存取以 Linux 為基礎之 HDInsight 上的 YARN 應用程式記錄](hdinsight-hadoop-access-yarn-app-logs-linux.md)
* [如何控制各種 Hadoop 元件的記錄檔大小](https://community.hortonworks.com/articles/8882/how-to-control-size-of-log-files-for-various-hdp-c.html) \(英文\)
