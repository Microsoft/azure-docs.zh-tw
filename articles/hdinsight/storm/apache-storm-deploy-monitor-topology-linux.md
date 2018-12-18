---
title: 部署和管理 HDInsight 上的 Apache Storm 拓撲
description: 了解如何使用以 Linux 為基礎 HDInsight 上的 Storm Dashboard 部署、監視和管理 Apache Storm 拓撲。 使用適用於 Visual Studio 的 Hadoop 工具。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 02/22/2018
ms.openlocfilehash: 486fcdfecf70b13d01c259f36b74676fb8e4d54f
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042951"
---
# <a name="deploy-and-manage-apache-storm-topologies-on-azure-hdinsight"></a>部署和管理 HDInsight 上的 Apache Storm 拓撲 

在本文件中，您可以了解管理和監視在 Storm on HDInsight 叢集上執行之 Storm 拓撲的基本概念。

> [!IMPORTANT]
> 本文中的步驟需要 HDInsight 叢集上以 Linux 為基礎的 Storm。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。 
>
> 如需部署和監視以 Windows 為基礎的 HDInsight 上的拓撲的詳細資訊，請參閱 [部署和管理以 Windows 為基礎的 HDInsight 上的Apache Storm 拓撲](apache-storm-deploy-monitor-topology.md)


## <a name="prerequisites"></a>必要條件

* **HDInsight 叢集上以 Linux 為基礎的 Storm**：請參閱 [開始使用 Apache Storm on HDInsight](apache-storm-tutorial-get-started-linux.md) 以取得建立叢集的步驟

* (選擇性) **熟悉 SSH 和 SCP**︰如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

* (選擇性) **Visual Studio**：Azure SDK 2.5.1 或更新版本，以及 Data Lake Tools for Visual Studio。 如需詳細資訊，請參閱[開始使用 Data Lake Tools for Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

    下列其中一個 Visual Studio 版本：

  * Visual Studio 2012 (含 Update 4)

  * Visual Studio 2013 (含 Update 4) 或 [Visual Studio 2013 Community](http://go.microsoft.com/fwlink/?LinkId=517284)
  * [Visual Studio 2015](https://www.visualstudio.com/downloads/)

  * Visual Studio 2015 (任何版本)

  * Visual Studio 2017 (任何版本)。 Data Lake Tools for Visual Studio 2017 會安裝為 Azure 工作負載的一部分。

## <a name="submit-a-topology-visual-studio"></a>提交拓撲︰Visual Studio

HDInsight Tools 可以用來將 C# 或混合式拓撲提交至 Storm 叢集。 下列步驟使用範例應用程式。 如需使用 HDInsight Tools 建立的詳細資訊，請參閱 [使用 HDInsight Tools for Visual Studio 開發 C# 拓撲](apache-storm-develop-csharp-visual-studio-topology.md)。

1. 如果您尚未安裝最新版的 Data Lake Tools for Visual Studio，請參閱[開始使用 Data Lake Tools for Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

    > [!NOTE]
    > Data Lake Tools for Visual Studio 先前稱為 HDInsight Tools for Visual Studio。
    >
    > Data Lake Tools for Visual Studio 隨附於適用於 Visual Studio 2017 的 __Azure 工作負載__中。

2. 開啟 Visual Studio，選取 [檔案]  >  [新增]  >  [專案]。

3. 在 [新增專案] 對話方塊中，依序展開 [已安裝]  >  [範本]，然後選取 [HDInsight]。 從範本清單中，選取 [Storm 範例] 。 在對話方塊底部，輸入應用程式的名稱。

    ![映像](./media/apache-storm-deploy-monitor-topology-linux/sample.png)

4. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，然後選取 [提交至 Storm on HDInsight]。

   > [!NOTE]
   > 如果出現提示，請輸入您 Azure 訂閱的登入認證。 如果您有多個訂用帳戶，請登入包含 Storm on HDInsight 叢集的訂用帳戶。

5. 從 [Storm 叢集] 下拉式清單中選取 Storm on HDInsight 叢集，然後選取 [提交]。 您可以使用 [輸出]  視窗監視提交是否成功。

## <a name="submit-a-topology-ssh-and-the-storm-command"></a>提交拓撲︰SSH 和 Storm 命令

1. 使用 SSH 連接到 HDInsight 叢集。 使用您的 SSH 登入名稱來取代 **USERNAME**。 將 **CLUSTERNAME** 取代為 HDInsight 叢集名稱：

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net

    如需使用 SSH 連線至 HDInsight 叢集的詳細資訊，請參閱[搭配 HDInsight 使用 SSH](../hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用下列命令以啟動範例拓撲：

        storm jar /usr/hdp/current/storm-client/contrib/storm-starter/storm-starter-topologies-*.jar org.apache.storm.starter.WordCountTopology WordCount

    這個命令會在叢集上啟動範例 WordCount 拓撲。 這個拓撲會隨機產生句子，並計算句子中每個字詞的出現次數。

   > [!NOTE]
   > 將拓撲提交至叢集時，您必須先複製包含叢集的 jar 檔案，再使用 `storm` 命令。 若要將檔案複製到叢集，您可以使用 `scp` 命令。 例如， `scp FILENAME.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.jar`
   >
   > WordCount 範例和其他 Storm 入門範例都已經包含在叢集中，位置是 `/usr/hdp/current/storm-client/contrib/storm-starter/`。

## <a name="submit-a-topology-programmatically"></a>提交拓撲︰以程式設計的方式

您可以透過程式設計的方式，使用 Nimbus 服務部署拓撲。 [https://github.com/Azure-Samples/hdinsight-java-deploy-storm-topology](https://github.com/Azure-Samples/hdinsight-java-deploy-storm-topology) 提供範例 Java 應用程式，以示範如何透過 Nimbus 服務部署和啟動拓撲。

## <a name="monitor-and-manage-visual-studio"></a>監視和管理︰Visual Studio

使用 Visual Studio 提交拓撲之後，隨即會出現 [Storm 拓撲] 檢視。 從清單中選取拓撲，以檢視執行中拓撲的詳細資訊。

![Visual Studio 監視器](./media/apache-storm-deploy-monitor-topology-linux/vsmonitor.png)

> [!NOTE]
> 您也可以透過展開 [Azure]  >  [HDInsight]，並在 Storm on HDInsight 叢集上按一下滑鼠右鍵，然後選取 [檢視 Storm 拓撲] 以從**伺服器總管**中檢視 [Storm 拓撲]。

選取 Spout 或 Bolt 的圖形以檢視這些元件的資訊。 隨即會針對每個選取的項目開啟新的視窗。

### <a name="deactivate-and-reactivate"></a>停用和重新啟動

停用拓撲會暫停它，直到刪除或重新啟動。 若要執行這些作業，請使用 [拓撲摘要] 頂端的 [停用] 和 [重新啟用] 按鈕。

### <a name="rebalance"></a>重新平衡

重新平衡拓撲可以讓系統修訂拓撲的平行處理原則。 例如，如果您已調整叢集的大小來新增更多節點，重新平衡可讓拓撲看見新的節點。

若要重新平衡拓撲，請使用 [拓撲摘要] 頂端的 [重新平衡] 按鈕。

> [!WARNING]
> 重新平衡拓撲首先會停用拓撲，然後跨叢集平均重新分佈背景工作角色，最後讓拓撲返回發生重新平衡之前的狀態。 因此，如果拓撲是作用中，它會再次變成作用中。 如果已停用，它就會保持停用。

### <a name="kill-a-topology"></a>終止拓撲

除非停止 Storm 拓撲或刪除叢集，否則 Storm 拓撲會繼續執行。 若要停止拓撲，請使用 [拓撲摘要] 頂端的 [終止] 按鈕。

## <a name="monitor-and-manage-ssh-and-the-storm-command"></a>監視和管理：SSH 和 Storm 命令

`storm` 公用程式可讓您從命令列使用執行中拓撲。 使用 `storm -h` 以取得完整的命令清單。

### <a name="list-topologies"></a>列出拓撲

使用下列命令來列出所有執行中拓撲：

    storm list

此命令會傳回類似以下文字的資訊：

    Topology_name        Status     Num_tasks  Num_workers  Uptime_secs
    -------------------------------------------------------------------
    WordCount            ACTIVE     29         2            263

### <a name="deactivate-and-reactivate"></a>停用和重新啟動

停用拓撲會暫停它，直到刪除或重新啟動。 使用下列命令來停用和重新啟動：

    storm Deactivate TOPOLOGYNAME

    storm Activate TOPOLOGYNAME

### <a name="kill-a-running-topology"></a>刪除執行中拓撲

Storm 拓撲一旦啟動之後，就會繼續執行直到停止。 若要停止拓撲，請使用下列命令：

    storm kill TOPOLOGYNAME

### <a name="rebalance"></a>重新平衡

重新平衡拓撲可以讓系統修訂拓撲的平行處理原則。 例如，如果您已調整叢集的大小來新增更多節點，重新平衡可讓拓撲看見新的節點。

> [!WARNING]
> 重新平衡拓撲首先會停用拓撲，然後跨叢集平均重新分佈背景工作角色，最後讓拓撲返回發生重新平衡之前的狀態。 因此，如果拓撲是作用中，它會再次變成作用中。 如果已停用，它就會保持停用。

    storm rebalance TOPOLOGYNAME

## <a name="monitor-and-manage-storm-ui"></a>監視和管理︰Storm UI

Storm UI 提供 Web 介面來處理執行中的拓撲，包含在您的 HDInsight 叢集中。 若要檢視 Storm UI，請使用網頁瀏覽器開啟 **https://CLUSTERNAME.azurehdinsight.net/stormui**，其中 **CLUSTERNAME** 是叢集的名稱。

> [!NOTE]
> 如果要求您提供使用者名稱和密碼，請輸入叢集系統管理員 (admin) 和建立叢集時使用的密碼。

### <a name="main-page"></a>主頁面

Storm UI 的主頁面會提供下列資訊：

* **叢集摘要**：Storm 叢集的基本資訊。
* **拓撲摘要**：執行中拓撲的清單。 使用本節中的連結來檢視特定拓撲的詳細資訊。
* **監督員摘要**：Storm 監督員的資訊。
* **Nimbus 組態**：叢集的 Nimbus 組態。

### <a name="topology-summary"></a>拓撲摘要

選取 [拓撲摘要]  區段中的連結會顯示拓撲的下列資訊：

* **拓撲摘要**：拓撲的基本資訊。
* **拓撲動作**：您可以針對拓撲執行的管理動作。

  * **啟用**：繼續處理已停用的拓撲。
  * **停用**：暫停執行中的拓撲。
  * **重新平衡**：調整拓撲的平行處理原則。 變更叢集中的節點數目之後，您應該重新平衡執行中拓撲。 這個作業可讓拓撲調整平行處理原則，以彌補叢集中增加或減少的節點數目。

    如需詳細資訊，請參閱 <a href="http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html" target="_blank">了解 Storm 拓撲的平行處理原則</a>。
  * **終止**：在指定的逾時之後結束 Storm 拓撲。
* **拓撲統計資料**：拓撲的統計資料。 若要設定頁面上其餘項目的時間範圍，請使用 [視窗] 資料行中的連結。
* **Spout**：拓撲所使用的 Spout。 使用本節中的連結檢視特定 Spout 的詳細資訊。
* **Bolt**：拓撲所使用的 Bolt。 使用本節中的連結檢視特定 Bolt 的詳細資訊。
* **拓撲組態**：所選取拓撲的組態。

### <a name="spout-and-bolt-summary"></a>Spout 和 Bolt 摘要

從 [Spout] 或 [Bolt] 區段中選取 Spout 會顯示所選取項目的下列資訊：

* **元件摘要**：Spout 或 Bolt 的基本資訊。
* **Spout/Bolt 統計資料**：Spout 或 Bolt 的統計資料。 若要設定頁面上其餘項目的時間範圍，請使用 [視窗] 資料行中的連結。
* **輸入統計資料** (僅限 Bolt)：Bolt 所使用輸入資料流的資訊。
* **輸出統計資料**：Spout 或 Bolt 所發出資料流的資訊。
* **執行程式**：Spout 或 Bolt 執行個體的資訊。 選取特定執行程式的 [連接埠]  項目，以檢視針對此執行個體所產生之診斷資訊的記錄。
* **錯誤**：Spout 或 Bolt 的任何錯誤資訊。

## <a name="monitor-and-manage-rest-api"></a>監視和管理︰REST API

Storm UI 是以 REST API 為建置基礎，因此您可以使用 REST API 執行類似的管理和監視功能。 您可以使用 REST API 建立自訂工具來管理和監視 Storm 拓撲。

如需詳細資訊，請參閱 [Storm UI REST API](http://storm.apache.org/releases/current/STORM-UI-REST-API.html)。 下列資訊專用於搭配使用 REST API 與 Apache Storm on HDInsight。

> [!IMPORTANT]
> Storm REST API 不是透過網際網路公開可用，而是必須使用 HDInsight 叢集前端節點的 SSH 通道來存取。 如需建立及使用 SSH 通道的詳細資訊，請參閱[使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](../hdinsight-linux-ambari-ssh-tunnel.md)。

### <a name="base-uri"></a>基底 URI

以 Linux 為基礎的 HDInsight 叢集上 REST API 的基底 URI 可在前端節點 (位於 **https://HEADNODEFQDN:8744/api/v1/**) 上取得。 前端節點的網域名稱是在叢集建立期間產生，而不是靜態的。

您可以用幾種不同的方式尋找叢集前端節點的完整網域名稱 (FQDN)：

* **從 SSH 工作階段**：使用命令 `headnode -f` (從 SSH 工作階段到叢集)。
* **從 Ambari Web**：從頁面頂端選取 [服務]，然後選取 [Storm]。 從 [摘要] 索引標籤，選取 [Storm UI 伺服器]。 託管 Storm UI 和 REST API 的節點 FQDN 位於頁面頂端。
* **從 Ambari REST API**：使用命令 `curl -u admin -G "https://CLUSTERNAME.azurehdinsight.net/api/v1/clusters/CLUSTERNAME/services/STORM/components/STORM_UI_SERVER"` 來擷取 Storm UI 和 REST API 執行所在節點的相關資訊。 將 **CLUSTERNAME** 取代為叢集名稱。 出現提示時，請輸入登入 (系統管理員) 帳戶的密碼。 在回應中，"host_name" 項目包含節點的 FQDN。

### <a name="authentication"></a>驗證

REST API 的要求必須使用 **基本驗證**，因此請使用 HDInsight 叢集管理員名稱和密碼。

> [!NOTE]
> 因為使用純文字傳送基本驗證，所以您應該 **一律** 使用 HTTPS 來保護與叢集通訊的安全。

### <a name="return-values"></a>傳回值

從 REST API 傳回的資訊可能只可在叢集中使用。 例如，無法從網際網路存取針對 Zookeeper 伺服器傳回的完整網域名稱 (FQDN)。

## <a name="next-steps"></a>後續步驟

了解如何[使用 Maven 開發 Java 型拓撲](apache-storm-develop-java-topology.md)。

若需更多範例拓撲的清單，請參閱 [Storm on HDInsight 的範例拓撲](apache-storm-example-topology.md)。
