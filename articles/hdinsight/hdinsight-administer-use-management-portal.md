---
title: 使用 Azure 入口網站在 HDInsight 中管理 Windows 型 Hadoop 叢集
description: 了解如何管理 HDInsight 服務。 建立 HDInsight 叢集、開啟互動式 JavaScript 主控台，以及開啟 Hadoop 命令主控台。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/25/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 130f684b69a9cfa1936c2a9aa5175f3d5a7631ac
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970272"
---
# <a name="manage-windows-based-hadoop-clusters-in-hdinsight-by-using-the-azure-portal"></a>使用 Azure 入口網站管理 HDInsight 中的 Windows 型 Hadoop 叢集

您可以使用 [Azure 入口網站][azure-portal]，在 Azure HDInsight 中建立 Windows 型 Hadoop 叢集、變更 Hadoop 使用者密碼，以及啟用遠端桌面通訊協定 (RDP)，以存取叢集上的 Hadoop 命令主控台。

本文的資訊僅適用於 Windows 型 HDInsight 叢集。 如需管理 Linux 型叢集的相關資訊，請參閱[使用 Azure 入口網站管理 HDInsight 上的 Hadoop 叢集](hdinsight-administer-use-portal-linux.md)。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。


## <a name="prerequisites"></a>必要條件

開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **Azure 儲存體帳戶** - HDInsight 叢集使用 Azure Blob 儲存體容器做為預設檔案系統。 如需 Azure Blob 儲存體如何提供順暢 HDInsight 叢集使用體驗的詳細資訊，請參閱 [搭配使用 Azure Blob 儲存體與 HDInsight](hdinsight-hadoop-use-blob-storage.md)。 如需建立 Azure 儲存體帳戶的詳細資訊，請參閱 [如何建立儲存體帳戶](../storage/common/storage-create-storage-account.md)。

## <a name="open-the-portal"></a>開啟入口網站
1. 登入 [https://portal.azure.com](https://portal.azure.com)。
2. 開啟入口網站之後，您可以：

   * 按一下左側功能表中的 [建立資源] 以建立新的叢集：

       ![新的 HDInsight 叢集按鈕](./media/hdinsight-administer-use-management-portal/azure-portal-new-button.png)
   * 按一下左功能表中的 [HDInsight 叢集]  。

       ![Azure 入口網站 HDInsight 叢集按鈕](./media/hdinsight-administer-use-management-portal/azure-portal-hdinsight-button.png)

     如果 **HDInsight** 未出現在左功能表中，請按一下 [瀏覽]。

     ![Azure 入口網站瀏覽叢集按鈕](./media/hdinsight-administer-use-management-portal/azure-portal-browse-button.png)

## <a name="create-clusters"></a>建立叢集
如需使用入口網站建立的相關指示，請參閱[建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

HDInsight 可以與很多 Hadoop 元件搭配使用。 如需已驗證和所支援元件的清單，請參閱 [Azure HDInsight 提供 Hadoop 的什麼版本？](hdinsight-component-versioning.md)(英文)。 您可以使用下列其中一個選項來自訂 HDInsight：

* 使用 [指令碼動作] 來執行可自訂叢集的自訂指令碼，以變更叢集組態或安裝自訂元件 (如 Giraph 或 Solr)。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster.md)。
* 在叢集建立期間，使用 HDInsight .NET SDK 或 Azure PowerShell 中的叢集自訂參數。 即會在叢集的存留期保留這些組態變更，而且它們不受叢集節點重新製作映像的影響，而 Azure 平台會定期執行重新製作映像以進行維護。 如需使用叢集自訂參數的詳細資訊，請參閱 [建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。
* 您可以使用 JAR 檔案形式在叢集上執行一些原生 Java 元件 (例如 Mahout 和 Cascading)。 這些 JAR 檔案可以配送至 Azure Blob 儲存體，並透過 Hadoop 工作提交機制提交至 HDInsight 叢集。 如需詳細資訊，請參閱 [以程式設計方式提交 Hadoop 工作](hadoop/submit-apache-hadoop-jobs-programmatically.md)。

  > [!NOTE]
  > 如果您在將 JAR 檔案部署至 HDInsight 叢集或在 HDInsight 叢集上呼叫 JAR 檔案時發生問題，請連絡 [Microsoft 支援](https://azure.microsoft.com/support/options/)。
  >
  > Cascading 不受 HDInsight 支援，而且不符合「Microsoft 支援」的資格。 如需所支援元件的清單，請參閱 [HDInsight 所提供叢集版本的新功能](hdinsight-component-versioning.md)。
  >
  >

不支援使用遠端桌面連線在叢集上安裝自訂軟體。 您應該避免在前端節點的磁碟機上儲存任何檔案，因為這些檔案會在您需要重新建立叢集時遺失。 建議您將檔案儲存至 Azure Blob 儲存體。 Blob 儲存體是持續性的。

## <a name="list-and-show-clusters"></a>列出和顯示叢集
1. 登入 [https://portal.azure.com](https://portal.azure.com)。
2. 按一下左功能表中的 [HDInsight 叢集]  。
3. 按一下叢集名稱。 如果叢集清單很長，您可以使用頁面頂端的篩選器。
4. 按兩下清單中的叢集以顯示詳細資料。

    **功能表和基本功能**：

    ![Azure portal HDInsight cluster essentials](./media/hdinsight-administer-use-management-portal/hdinsight-essentials.png)

   * 若要自訂功能表，請在功能表上的任意處按一下滑鼠右鍵，然後按一下 [自訂] 。
   * **設定**和**所有設定**：顯示該叢集的 [設定] 刀鋒視窗，可讓您存取該叢集的詳細組態資訊。
   * **儀表板**、**叢集儀表板**和 **URL**：這些是存取叢集儀表板 (也就是適用於 Linux 型叢集的 Ambari Web) 的所有方法。
   * **安全殼層**︰顯示使用安全殼層 (SSH) 連線連接到叢集的指示。
   * **調整叢集**：可讓您變更此叢集的背景工作節點數目。
   * **刪除**：刪除叢集。
   * **快速啟動**：顯示可協助您開始使用 HDInsight 的資訊。
   * **使用者**：可讓您為自己 Azure 訂用帳戶的其他使用者設定針對此叢集的「入口網站管理」權限。

     > [!IMPORTANT]
     > 這「只」會影響在 Azure 入口網站對此叢集的存取和權限，對於連線到 HDInsight 叢集或將工作提交到 HDInsight 叢集的使用者沒有影響。
     >
     >
   * **標記**：標記可讓您設定索引鍵/值組，以定義自訂的雲端服務分類法。 例如，您可建立名為 **project**的索引鍵，然後使用與特定專案相關聯之所有服務的通用值。
   * **Ambari 檢視**：Ambari Web 的連結。

     > [!IMPORTANT]
     > 若要管理 HDInsight 叢集所提供的服務，您必須使用 Ambari Web 或 Ambari REST API。 如需使用 Ambari 的詳細資訊，請參閱 [使用 Ambari 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)。
     >
     >

     **使用量**：

     ![Azure 入口網站 HDInsight 叢集使用量](./media/hdinsight-administer-use-management-portal/hdinsight-portal-cluster-usage.png)
5. 按一下 [設定] 。

    ![Azure 入口網站 HDInsight 叢集使用量](./media/hdinsight-administer-use-management-portal/hdinsight.portal.cluster.settings.png)

   * **屬性**：檢視叢集屬性。
   * **叢集 AAD 身分識別**：
   * **Azure 儲存體金鑰**：檢視預設儲存體帳戶與其金鑰。 儲存體帳戶是叢集建立程序期間的組態。
   * **叢集登入**：變更叢集 HTTP 使用者名稱和密碼。
   * **外部中繼存放區**：檢視 Hive 和 Oozie 中繼存放區。 中繼存放區只可以在叢集建立程序期間進行設定。
   * **調整叢集**：增加和減少叢集背景工作角色節點的數目。
   * **遠端桌面**： 啟用和停用遠端桌面 (RDP) 存取，以及設定 RDP 使用者名稱。  RDP 使用者名稱必須與 HTTP 使用者名稱不同。
   * **記錄可查夥伴**：

     > [!NOTE]
     > 這是可用設定的泛型清單，並不會針對所有叢集類型顯示所有的可用設定。
     >
     >
6. 按一下 [屬性] ：

    屬性區段如下所列：

   * **主機名稱**：叢集名稱。
   * **叢集 URL**。
   * **狀態**：包括Aborted、Accepted、ClusterStorageProvisioned、AzureVMConfiguration、HDInsightConfiguration、Operational、Running、Error、Deleting、Deleted、Timedout、DeleteQueued、DeleteTimedout、DeleteError、PatchQueued、CertRolloverQueued、ResizeQueued、ClusterCustomization
   * **區域**：Azure 位置。 如需支援的 Azure 位置清單，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)上的 [區域] 下拉式清單方塊。
   * **資料已建立**。
   * **作業系統**：**Windows** 或 **Linux**。
   * **類型**：Hadoop、HBase、Storm、Spark。
   * **版本**。 請參閱 [HDInsight 版本](hdinsight-component-versioning.md)
   * **訂用帳戶**︰訂用帳戶名稱。
   * **訂用帳戶識別碼**。
   * **主要資料來源**。 用來做為預設 Hadoop 檔案系統的 Azure Blob 儲存體帳戶。
   * **背景工作節點定價層**。
   * **前端節點定價層**。

## <a name="delete-clusters"></a>刪除叢集
刪除叢集時，並不會刪除預設的儲存體帳戶或任何連結的儲存體帳戶。 您可以使用相同的儲存體帳戶和相同的中繼存放區重新建立叢集。

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 按一下頂端功能表中的 [刪除]  ，然後依照指示執行。

另請參閱 [暫停/關閉叢集](#pauseshut-down-clusters)。

## <a name="scale-clusters"></a>調整叢集
叢集調整功能可讓您變更在 Azure HDInsight 中執行的叢集所用的背景工作節點數目，而不需要重新建立叢集。

> [!NOTE]
> 只支援使用 HDInsight 3.1.3 版或更高版本的叢集。 如果不確定您的叢集版本，您可以檢查 [屬性] 頁面。  請參閱[列出和顯示叢集](#list-and-show-clusters)。
>
>

變更 HDInsight 支援的每一種叢集所用的資料節點數目會有何影響：

* Hadoop

    您可以順暢地增加正在執行的 Hadoop 叢集中背景工作節點數目，而不會影響任何擱置或執行中的工作。 您也可以在作業進行當中提交新工作。 系統會順暢處理失敗的調整作業，讓叢集永保正常運作狀態。

    減少資料節點數目以縮減 Hadoop 叢集時，系統會重新啟動叢集中的部分服務。 這會導致所有執行中和擱置的工作在調整作業完成時失敗。 但您可以在作業完成後重新提交這些工作。
* hbase

    您可以順暢地在 HBase 叢集運作時對其新增或移除資料節點。 區域伺服器會在完成調整作業的數分鐘之內自動取得平衡。 但是，您也可以手動平衡區域伺服器，方法是登入叢集的前端節點，然後從命令提示字元視窗執行下列命令：

        >pushd %HBASE_HOME%\bin
        >hbase shell
        >balancer

    如需使用 HBase 殼層的詳細資訊，請參閱 []
* Storm

    您可以順暢地在 Storm 叢集運作時對其新增或移除資料節點。 但在調整作業順利完成後，您需要重新平衡拓撲。

    您可以使用兩種方式來完成重新平衡作業：

  * Storm Web UI
  * 命令列介面 (CLI) 工具

    如需詳細資訊，請參閱 [Apache Storm 文件](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html) 。

    HDInsight 叢集上有提供 Storm Web UI：

    ![hdinsight storm scale rebalance](./media/hdinsight-administer-use-management-portal/hdinsight-portal-scale-cluster-storm-rebalance.png)

    以下是如何使用 CLI 命令重新平衡 Storm 拓撲的範例：

        ## Reconfigure the topology "mytopology" to use 5 worker processes,
        ## the spout "blue-spout" to use 3 executors, and
        ## the bolt "yellow-bolt" to use 10 executors
        $ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

**調整叢集**

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 在頂端功能表按一下 [設定]，然後按一下 [調整叢集]。
4. 輸入 **背景工作節點的數目**。 叢集節點的數目限制會因 Azure 訂用帳戶而有所不同。 請連絡帳務支援提高限制。  成本資訊會反映您對節點數目所做的變更。

    ![HDInsight Hadoop HBase Storm Spark scale](./media/hdinsight-administer-use-management-portal/hdinsight.portal.scale.cluster.png)

## <a name="pauseshut-down-clusters"></a>暫停/關閉叢集
大部分 Hadoop 工作是只會偶爾執行的批次工作。 對於大部分的 Hadoop 叢集而言，叢集長時間並未用於處理。 利用 HDInsight，您的資料會儲存在 Azure 儲存體中，以便您在未使用叢集時安全地進行刪除。
您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。

有許多方法可以設計程序：

* 使用 Azure Data Factory。 請參閱 [Azure HDInsight 連結服務](../data-factory/compute-linked-services.md)和[使用 Azure Data Factory 進行轉換和分析](../data-factory/transform-data.md)，以取得隨選和自行定義的 HDInsight 連結服務。
* 使用 Azure PowerShell。  請參閱 [分析航班延誤資料](hdinsight-analyze-flight-delay-data.md)。
* 使用 Azure 傳統 CLI。 請參閱[使用 Azure 傳統 CLI 管理 HDInsight 叢集](hdinsight-administer-use-command-line.md)。
* 使用 HDInsight .NET SDK。 請參閱 [提交 Hadoop 工作](hadoop/submit-apache-hadoop-jobs-programmatically.md)。

如需定價資訊，請參閱 [HDInsight 定價](https://azure.microsoft.com/pricing/details/hdinsight/)。 若要從入口網站刪除叢集，請參閱 [刪除叢集](#delete-clusters)

## <a name="change-cluster-username"></a>變更叢集使用者名稱
HDInsight 叢集可以有兩個使用者帳戶。 HDInsight 叢集使用者帳戶是在建立程序期間所建立。 您也可以建立 RDP 使用者帳戶，以透過 RDP 來存取叢集。 請參閱 [啟用遠端桌面](#connect-to-hdinsight-clusters-by-using-rdp)。

**變更 HDInsight 叢集使用者名稱和密碼**

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 在頂端功能表按一下 [設定]，然後按一下 [叢集登入]。
4. 如果已啟用 [叢集登入]，您必須先按一下 [停用]，然後按一下 [啟用]，之後才能變更使用者名稱和密碼。
5. 變更 [叢集登入名稱] 及/或 [叢集登入密碼]，然後按一下 [儲存]。

    ![HDInsight change cluster user username password http user](./media/hdinsight-administer-use-management-portal/hdinsight.portal.change.username.password.png)

## <a name="grantrevoke-access"></a>授與/撤銷存取權
HDInsight 叢集具有下列 HTTP Web 服務 (所有這些服務都有 RESTful 端點)：

* ODBC
* JDBC
* Ambari
* Oozie
* Templeton

預設會授與這些服務的存取權。 您可以從 Azure 入口網站撤銷/授與存取權。

> [!NOTE]
> 透過授與/撤銷存取權，您將重設叢的使用者名稱和密碼。
>
>

**授與/撤銷 HTTP Web 服務存取**

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 在頂端功能表按一下 [設定]，然後按一下 [叢集登入]。
4. 如果已啟用 [叢集登入]，您必須先按一下 [停用]，然後按一下 [啟用]，之後才能變更使用者名稱和密碼。
5. 針對 [叢集登入使用者名稱] 和 [叢集登入密碼]，分別輸入叢集的新使用者名稱和密碼。
6. 按一下 [儲存] 。

    ![HDInsight grand remove http web service access](./media/hdinsight-administer-use-management-portal/hdinsight.portal.change.username.password.png)

## <a name="find-the-default-storage-account"></a>尋找預設的儲存體帳戶
每個 HDInsight 叢集都有預設的儲存體帳戶。 叢集的預設儲存體帳戶與其金鑰會顯示在 [設定] /**屬性**/**Azure 儲存體金鑰**。 請參閱[列出和顯示叢集](#list-and-show-clusters)。

## <a name="find-the-resource-group"></a>尋找資源群組
在 Azure Resource Manager 模式中，每個 HDInsight 叢集是隨著 Azure 資源群組一起建立。 叢集所屬的 Azure 資源群組會出現於：

* 叢集清單含有 [資源群組]  資料行。
* 叢集 [基本資料]  磚。  

請參閱[列出和顯示叢集](#list-and-show-clusters)。

## <a name="open-hdinsight-query-console"></a>開啟 HDInsight 查詢主控台
HDInsight 查詢主控台包括下列功能：

* **Hive 編輯器**：用於提交 Hive 工作的 GUI Web 介面。  請參閱 [使用查詢主控台執行 Hive 查詢](hadoop/apache-hadoop-use-hive-query-console.md)。

    ![HDInsight portal hive editor](./media/hdinsight-administer-use-management-portal/hdinsight-hive-editor.png)
* **工作歷程記錄**：監視 Hadoop 工作。  

    ![HDInsight portal job history](./media/hdinsight-administer-use-management-portal/hdinsight-job-history.png)

    按一下 [查詢名稱] 來顯示詳細資料，包括工作屬性、[作業查詢] 和 [作業輸出]。 您也可以將查詢和輸出下載至您的工作站。
* **檔案瀏覽器**：瀏覽預設的儲存體帳戶和連結的儲存體帳戶。

    ![HDInsight portal file browser browse](./media/hdinsight-administer-use-management-portal/hdinsight-file-browser.png)

    在螢幕擷取畫面上， **<Account>** 類型表示此項目是 Azure 儲存體帳戶。  按一下帳戶名稱以瀏覽檔案。
* **Hadoop UI**。

    ![HDInsight portal Hadoop UI](./media/hdinsight-administer-use-management-portal/hdinsight-hadoop-ui.png)

    從「Hadoop UI」，您可以瀏覽檔案，並檢查記錄檔。
* **Yarn UI**。

    ![HDInsight portal YARN UI](./media/hdinsight-administer-use-management-portal/hdinsight-yarn-ui.png)

## <a name="run-hive-queries"></a>執行 Hive 查詢
若要從入口網站執行 Hive 作業，請按一下 HDInsight 查詢主控台中的 [Hive 編輯器]。 請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

## <a name="monitor-jobs"></a>監視工作
若要從入口網站監視 Hive 作業，請按一下 HDInsight 查詢主控台中的 [ **工作歷程記錄** ]。 請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

## <a name="browse-files"></a>瀏覽檔案
若要瀏覽預設儲存體帳戶和連結儲存體帳戶中儲存的檔案，請按一下 HDInsight 查詢主控台中的 [檔案瀏覽器]  。 請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

您也可以從 HDInsight 主控台中的 [Hadoop UI] 使用 [瀏覽檔案系統] 公用程式。  請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

## <a name="monitor-cluster-usage"></a>監視叢集使用量
HDInsight 叢集刀鋒視窗的 [使用量] 區段會顯示以下資訊：訂用帳戶可搭配 HDInsight 使用的核心數目，以及配置給此叢集的核心數目和它們在此叢集中配置給節點的方式。 請參閱 [列出和顯示叢集](#list-and-show-clusters)。

> [!IMPORTANT]
> 若要監視 HDInsight 叢集所提供的服務，您必須使用 Ambari Web 或 Ambari REST API。 如需使用 Ambari 的詳細資訊，請參閱 [使用 Ambari 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)
>
>

## <a name="open-hadoop-ui"></a>開啟 Hadoop UI
若要監視叢集、瀏覽檔案系統及檢查記錄檔，請按一下 HDInsight 查詢主控台中的 [Hadoop UI]  。 請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

## <a name="open-yarn-ui"></a>開啟 Yarn UI
若要使用 Yarn 使用者介面，請按一下 HDInsight 查詢主控台中的 [Yarn UI]  。 請參閱 [開啟 HDInsight 查詢主控台](#open-hdinsight-query-console)。

## <a name="connect-to-clusters-using-rdp"></a>使用 RDP 連接到叢集
您在建立叢集時所提供的叢集認證可以存取叢集上的服務，但是無法透過遠端桌面來存取叢集本身。 當您佈建叢集時或叢集佈建完成後，可以開啟遠端桌面存取。 如需在建立時啟用遠端桌面的指示，請參閱 [建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

**啟用遠端桌面**

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 在頂端功能表按一下 [設定]，然後按一下 [遠端桌面]。
4. 輸入 [到期日]、[遠端桌面使用者名稱] 和 [遠端桌面密碼]，然後按一下 [啟用]。

    ![HDInsight 啟用停用設定遠端桌面](./media/hdinsight-administer-use-management-portal/hdinsight.portal.remote.desktop.png)

    [到期日] 的預設值是一週。

   > [!NOTE]
   > 您也可以使用 HDInsight .NET SDK，在叢集上啟用遠端桌面。 以下列方式在 HDInsight 用戶端物件上使用 **EnableRdp** 方法：**client.EnableRdp(clustername, location, "rdpuser", "rdppassword", DateTime.Now.AddDays(6))**。 同樣地，若要在叢集上停用遠端桌面，您可以使用 **client.DisableRdp(clustername, location)**。 如需這些方法的詳細資訊，請參閱 [HDInsight .NET SDK 參考](http://go.microsoft.com/fwlink/?LinkId=529017)。 這僅適用於在 Windows 上執行的 HDInsight 叢集。
   >
   >

**使用 RDP 連線到叢集**

1. 登入[入口網站][azure-portal]。
2. 依序按一下左側功能表的 [瀏覽全部]、[HDInsight 叢集] 及您的叢集名稱。
3. 在頂端功能表按一下 [設定]，然後按一下 [遠端桌面]。
4. 按一下 [連接]  並遵循指示。 如果 [連接] 已停用，您必須先加以啟用。 請務必使用遠端桌面使用者的使用者名稱和密碼。  不可使用叢集使用者的認證。

## <a name="open-hadoop-command-line"></a>開啟 Hadoop 命令列
若要使用遠端桌面連線到叢集，並使用 Hadoop 命令列，則您必須先啟用對叢集的遠端桌面存取 (如上一節所述)。

**開啟 Hadoop 命令列**

1. 使用遠端桌面連接到叢集。
2. 從桌面上，按兩下 [Hadoop 命令列] 。

    ![HDI.HadoopCommandLine][image-hadoopcommandline]

    如需 Hadoop 命令的詳細資訊，請參閱 [Hadoop 命令參考](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/CommandsManual.html)。

在前一個螢幕擷取畫面上，資料夾名稱已內嵌 Hadoop 版本號碼。 版本號碼會根據叢集上所安裝的 Hadoop 元件版本而變更。 您可以使用 Hadoop 環境變數來參照那些資料夾。 例如︰

    cd %hadoop_home%
    cd %hive_home%
    cd %hbase_home%
    cd %pig_home%
    cd %sqoop_home%
    cd %hcatalog_home%

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用入口網站建立 HDInsight 叢集，以及如何開啟 Hadoop 命令列工具。 若要深入了解，請參閱下列文章：

* [使用 Azure PowerShell 管理 HDInsight](hdinsight-administer-use-powershell.md)
* [使用 Azure 傳統 CLI 管理 HDInsight](hdinsight-administer-use-command-line.md)
* [建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)
* [以程式設計方式提交 Hadoop 工作](hadoop/submit-apache-hadoop-jobs-programmatically.md)
* [開始使用 Azure HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Azure HDInsight 提供 Hadoop 的什麼版本？](hdinsight-component-versioning.md)

[azure-portal]: https://portal.azure.com
[image-hadoopcommandline]: ./media/hdinsight-administer-use-management-portal/hdinsight-hadoop-command-line.png "Hadoop 命令列"
