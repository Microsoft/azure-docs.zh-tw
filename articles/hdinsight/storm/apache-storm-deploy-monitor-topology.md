---
title: 部署和管理 HDInsight 中的 Apache Storm 拓撲
description: 了解如何使用以 Windows 為基礎 HDInsight 上的 Storm Dashboard 部署、監視及管理 Apache Storm 拓撲。 使用適用於 Visual Studio 的 Hadoop 工具。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 03/01/2017
ms.author: jasonh
ROBOTS: NOINDEX
ms.openlocfilehash: 97dfa2ffc103de377b4c510d2a3a7404b5e96747
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43050724"
---
# <a name="deploy-and-manage-apache-storm-topologies-on-windows-based-hdinsight"></a>部署和管理以 Windows 為基礎的 HDInsight 上的 Apache Storm 拓撲

Storm Dashboard 可讓您使用網頁瀏覽器輕鬆地部署和執行 Apache Storm 拓撲至 HDInsight 叢集。 您也可以使用儀表板來監視和管理執行中拓撲。 如果您使用 Visual Studio，則 HDInsight Tools for Visual Studio 會提供 Visual Studio 中的類似功能。

HDInsight Tools 的 Storm Dashboard 和 Storm 功能依賴 Storm REST API，此 API 可用來建立您專屬的監視和管理方案。

> [!IMPORTANT]
> 這份文件中的步驟需要使用 Windows 做為作業系統的 Storm on HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](../hdinsight-component-versioning.md#hdinsight-windows-retirement)。
>
> 如需透過使用 Linux 的 HDInsight 叢集來部署和管理 Storm 拓撲的詳細資訊，請參閱[部署和管理以 Linux 為基礎的 HDInsight 上的 Apache Storm 拓撲](apache-storm-deploy-monitor-topology-linux.md)

## <a name="prerequisites"></a>必要條件

* **Apache Storm on HDInsight** - 請參閱[開始使用 Apache Storm on HDInsight](apache-storm-tutorial-get-started-linux.md)，以取得建立叢集的步驟。

* 針對 **Storm Dashboard**：支援 HTML5 的新型網頁瀏覽器。

* 針對 **Visual Studio** - Azure SDK 2.5.1 或更新版本，以及 HDInsight Tools for Visual Studio。 請參閱[開始使用 HDInsight Tools for Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md) 以安裝及設定 HDInsight Tools for Visual Studio。

    下列其中一個 Visual Studio 版本：

  * Visual Studio 2012 (含 Update 4)

  * Visual Studio 2013 (含 Update 4) 或 Visual Studio 2013 Community

  * Visual Studio 2015 (任何版本)

  * Visual Studio 2017 (任何版本)

## <a name="storm-dashboard"></a>Storm Dashboard

Storm Dashboard 是可以在 Storm 叢集上使用的網頁。 URL 是 **https://&lt;clustername>.azurehdinsight.net/**，其中 **clustername** 是 Storm on HDInsight 叢集的名稱。

從 Storm Dashboard 的頂端，選取 [提交拓撲] 。 依照頁面上的指示來執行範例拓撲，或者上傳及執行您已建立的拓撲。

![提交拓撲頁面][storm-dashboard-submit]

### <a name="storm-ui"></a>Storm UI

從 Storm Dashboard 中選取 [Storm UI]  連結。 這會顯示叢集和任何執行中拓撲的詳細資訊。

![Storm UI][storm-dashboard-ui]

> [!NOTE]
> 在 Internet Explorer 的某些版本中，您可能會發現 Storm UI 在您最初到訪之後不會重新整理。 例如，可能不會顯示您提交的新拓撲，或是可能將先前已停用的拓撲顯示為使用中。 Microsoft 知道這個問題，並正在找出解決方案。

#### <a name="main-page"></a>主頁面

Storm UI 的主頁面會提供下列資訊：

* **叢集摘要**：Storm 叢集的基本資訊。

* **拓撲摘要**：執行中拓撲的清單。 使用本節中的連結來檢視特定拓撲的詳細資訊。

* **監督員摘要**：Storm 監督員的資訊。

* **Nimbus 組態**：叢集的 Nimbus 組態。

#### <a name="topology-summary"></a>拓撲摘要

選取 [拓撲摘要]  區段中的連結會顯示拓撲的下列資訊：

* **拓撲摘要**：拓撲的基本資訊。

* **拓撲動作**：您可以針對拓撲執行的管理動作。

  * **啟用**：繼續處理已停用的拓撲。

  * **停用**：暫停執行中的拓撲。

  * **重新平衡**：調整拓撲的平行處理原則。 變更叢集中的節點數目之後，您應該重新平衡執行中拓撲。 這可讓拓撲調整平行處理原則，以彌補叢集中增加或減少的節點數目。

      如需詳細資訊，請參閱[了解 Storm 拓撲的平行處理原則](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。

  * **終止**：在指定的逾時之後結束 Storm 拓撲。

* **拓撲統計資料**：拓撲的統計資料。 使用 [視窗]  資料行中的連結，以設定頁面上其餘項目的時間範圍。

* **Spout**：拓撲所使用的 Spout。 使用本節中的連結檢視特定 Spout 的詳細資訊。

* **Bolt**：拓撲所使用的 Bolt。 使用本節中的連結檢視特定 Bolt 的詳細資訊。

* **拓撲組態**：所選取拓撲的組態。

#### <a name="spout-and-bolt-summary"></a>Spout 和 Bolt 摘要

從 [Spout] 或 [Bolt] 區段中選取 Spout 會顯示所選取項目的下列資訊：

* **元件摘要**：Spout 或 Bolt 的基本資訊。

* **Spout/Bolt 統計資料**：Spout 或 Bolt 的統計資料。 使用 [視窗]  資料行中的連結，以設定頁面上其餘項目的時間範圍。

* **輸入統計資料** (僅限 Bolt)：Bolt 所使用輸入資料流的資訊。

* **輸出統計資料**：此 Spout 或 Bolt 所發出資料流的資訊。

* **執行程式**：Spout 或 Bolt 執行個體的資訊。 選取特定執行程式的 [連接埠]  項目，以檢視針對此執行個體所產生之診斷資訊的記錄。

* **錯誤**：此 Spout 或 Bolt 的任何錯誤資訊。

## <a name="hdinsight-tools-for-visual-studio"></a>HDInsight Tools for Visual Studio

HDInsight Tools 可以用來將 C# 或混合式拓撲提交至 Storm 叢集。 下列步驟使用範例應用程式。 如需使用 HDInsight Tools 建立您專屬拓撲的詳細資訊，請參閱 [使用 HDInsight Tools for Visual Studio 開發 C# 拓撲](apache-storm-develop-csharp-visual-studio-topology.md)。

使用下列步驟，將範例部署至 Storm on HDInsight 叢集，然後檢視和管理拓撲。

1. 如果您尚未安裝最新版本的 HDInsight Tools for Visual Studio，請參閱[開始使用 HDInsight Tools for Visual Studio](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

2. 開啟 Visual Studio，選取 [檔案]  >  [新增]  >  [專案]。

3. 在 [新增專案] 對話方塊中，依序展開 [已安裝]  >  [範本]，然後選取 [HDInsight]。 從範本清單中，選取 [Storm 範例] 。 在對話方塊底部，輸入應用程式的名稱。

    ![映像](./media/apache-storm-deploy-monitor-topology/sample.png)

4. 在 [方案總管] 中，於專案上按一下滑鼠右鍵，然後選取 [提交至 Storm on HDInsight]。

   > [!NOTE]
   > 如果出現提示，請輸入您 Azure 訂閱的登入認證。 如果您有多個訂用帳戶，請登入包含 Storm on HDInsight 叢集的訂用帳戶。

5. 從 [Storm 叢集] 下拉式清單中選取 Storm on HDInsight 叢集，然後選取 [提交]。 您可以使用 [輸出]  視窗監視提交是否成功。

6. 順利提交拓撲時，應該會出現叢集的 [Storm 拓撲]  。 從清單中選取拓撲，以檢視執行中拓撲的詳細資訊。

    ![Visual Studio 監視器](./media/apache-storm-deploy-monitor-topology/vsmonitor.png)

   > [!NOTE]
   > 您也可以透過展開 [Azure]  >  [HDInsight]，並在 Storm on HDInsight 叢集上按一下滑鼠右鍵，然後選取 [檢視 Storm 拓撲] 以從**伺服器總管**中檢視 [Storm 拓撲]。

    選取 Spout 或 Bolt 的圖形以檢視這些元件的資訊。 隨即會針對每個選取的項目開啟新的視窗。

   > [!NOTE]
   > 拓撲的名稱是拓撲的類別名稱 (在此案例中為 `HelloWord`) 加上時間戳記。

7. 從 [拓撲摘要] 檢視中，選取 [終止] 停止拓撲。

   > [!NOTE]
   > 除非停止 Storm 拓撲或刪除叢集，否則 Storm 拓撲會繼續執行。


## <a name="rest-api"></a>REST API

Storm UI 是以 REST API 為建置基礎，因此您可以使用 REST API 執行類似的管理和監視功能。 您可以使用 REST API 建立自訂工具來管理和監視 Storm 拓撲。

如需詳細資訊，請參閱 [Storm UI REST API](https://github.com/apache/storm/blob/0.9.3-branch/STORM-UI-REST-API.md)。 下列資訊專用於搭配使用 REST API 與 Apache Storm on HDInsight。

### <a name="base-uri"></a>基底 URI

REST API on HDInsight 叢集的基底 URI 是 **https://&lt;clustername>.azurehdinsight.net/stormui/api/v1/**，其中 **clustername** 是 Storm on HDInsight 叢集的名稱。

### <a name="authentication"></a>驗證

REST API 的要求必須使用 **基本驗證**，因此請使用 HDInsight 叢集管理員名稱和密碼。

> [!NOTE]
> 因為使用純文字傳送基本驗證，所以您應該 **一律** 使用 HTTPS 來保護與叢集通訊的安全。

### <a name="return-values"></a>傳回值

從 REST API 傳回的資訊只能從叢集或與叢集相同之 Azure 虛擬網路上的虛擬機器內使用。 例如，無法從網際網路存取針對 Zookeeper 伺服器所傳回的完整網域名稱 (FQDN)。

## <a name="next-steps"></a>後續步驟

現在，您已了解如何使用 Storm Dashboard 部署和監視拓撲，接著請了解如何：

* [使用 HDInsight Tools for Visual Studio 開發 C# 拓撲](apache-storm-develop-csharp-visual-studio-topology.md)

* [使用 Maven 開發 Java 型拓撲](apache-storm-develop-java-topology.md)

若需更多範例拓撲的清單，請參閱 [Storm on HDInsight 的範例拓撲](apache-storm-example-topology.md)。

[hdinsight-dashboard]: ./media/apache-storm-deploy-monitor-topology/dashboard-link.png
[storm-dashboard-submit]: ./media/apache-storm-deploy-monitor-topology/submit.png
[storm-dashboard-ui]: ./media/apache-storm-deploy-monitor-topology/storm-ui-summary.png
