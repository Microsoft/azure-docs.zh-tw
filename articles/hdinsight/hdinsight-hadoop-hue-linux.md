---
title: 在 HDInsight Linux 叢集上使用 Hue 與 Hadoop - Azure - Azure
description: 了解如何在 HDInsight 叢集上安裝 Hue，並且使用通道將要求路由至 Hue。 使用 Hue 瀏覽儲存體並執行 Hive 或 Pig。
keywords: Hue hadoop
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 12/11/2017
ms.author: jasonh
ms.openlocfilehash: db2c4281f23685c00e63058a5cfd8caaf663b0fb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948735"
---
# <a name="install-and-use-hue-on-hdinsight-hadoop-clusters"></a>在 HDInsight Hadoop 叢集上安裝和使用 Hue

了解如何在 HDInsight 叢集上安裝 Hue，並且使用通道將要求路由至 Hue。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="what-is-hue"></a>何謂 Hue？
色調是一組 Web 應用程式，用來與 Hadoop 叢集互動。 您可以使用 Hue 以瀏覽與 Hadoop 叢集 (在 HDInsight 叢集的案例中為 WASB) 相關聯的儲存體、執行 Hive 工作和 Pig 指令碼等等。 HDInsight Hadoop 叢集上的 Hue 安裝可使用下列元件。

* Beeswax Hive 編輯器
* Pig
* Metastore 管理員
* Oozie
* FileBrowser (與 WASB 預設容器進行通訊)
* 工作瀏覽器

> [!WARNING]
> 透過 HDInsight 叢集提供的元件會受到完整支援，且 Microsoft 支援服務將協助釐清與解決這些元件的相關問題。
>
> 自訂元件則獲得商務上合理的支援，協助您進一步疑難排解問題。 如此可能會進而解決問題，或要求您利用可用管道，以找出開放原始碼技術，從中了解該技術的深度專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。
>
>

## <a name="install-hue-using-script-actions"></a>使用指令碼動作安裝 Hue

您可以從 https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh \(英文\) 取得可在 Linux 型 HDInsight 叢集上安裝 Hue 的指令碼。您可以使用這個指令碼在叢集上安裝色調，並以 Azure 儲存體 Blob (WASB) 或 Azure Data Lake Store 做為預設儲存體。

本節提供如何在使用 Azure 入口網站佈建叢集時使用指令碼的指示。

> [!NOTE]
> Azure PowerShell、Azure 傳統 CLI、HDInsight .NET SDK 或 Azure Resource Manager 範本也可用來套用指令碼動作。 您也可以將指令碼動作套用到執行中的叢集上。 如需詳細資訊，請參閱 [使用指令碼動作自訂 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)。
>
>

1. 使用[在 Linux 上佈建 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)中的步驟開始佈建叢集，但是不完成佈建。

   > [!NOTE]
   > 若要在 HDInsight 叢集上安裝 Hue，建議的 HeadNode 大小為至少 A4 (8 核心、14 GB 記憶體)。
   >
   >
2. 在 [選用組態] 刀鋒視窗中，選取 [指令碼動作]，並提供如下所示的資訊：

    ![提供 Hue 的指令碼動作參數](./media/hdinsight-hadoop-hue-linux/hue-script-action.png "提供 Hue 的指令碼動作參數")

   * **名稱**：輸入指令碼動作的易記名稱。
   * **指令碼 URI**： https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh
   * **HEAD**：勾選此選項
   * **背景工作角色**：將此選項保留空白。
   * **ZOOKEEPER**：將此選項保留空白。
   * **參數**：將此選項保留空白。
3. 在 [指令碼動作] 底部，使用 [選取] 按鈕以儲存組態。 最後，使用 [選用組態] 刀鋒視窗底部的 [選取] 按鈕，儲存選用組態資訊。
4. 繼續如[在 Linux 上佈建 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)中所述佈建叢集。

## <a name="use-hue-with-hdinsight-clusters"></a>搭配使用 Hue 與 HDInsight 叢集

執行 Hue 之後，SSH 通道是在叢集上存取 Hue 的唯一方式。 透過 SSH 的通道允許直接至在其中執行 Hue 之叢集的前端節點的流量。 叢集完成佈建之後，請使用下列步驟，在 HDInsight Linux 叢集上使用 Hue。

> [!NOTE]
> 建議您使用 Firefox 網頁瀏覽器來遵循下列指示。
>
>

1. 請以 [使用 SSH 通道來存取 Ambari Web UI、ResourceManager、JobHistory、NameNode、Oozie 及其他 Web UI](hdinsight-linux-ambari-ssh-tunnel.md) 中的資訊，建立用戶端系統至 HDInsight 叢集的 SSH 通道，然後設定網頁瀏覽器以便將 SSH 通道當做 Proxy 使用。

2. 一旦您建立了 SSH 通道，並設定您的瀏覽器將流量以 Proxy 通過該通道傳送後，您必須找到主要前端節點的主機名稱。 若要這麼做，您可以使用 SSH 在連接埠 22 上連線到叢集。 例如，`ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`，其中 **USERNAME** 是您的 SSH 使用者名稱，**CLUSTERNAME** 是您的叢集名稱。

    如需詳細資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

3. 連線之後，請使用下列命令來取得主要前端節點的完整網域名稱︰

        hostname -f

    此命令會傳回類似以下的名稱：

        hn0-myhdi-nfebtpfdv1nubcidphpap2eq2b.ex.internal.cloudapp.net

    這是 Hue 網站所在之主要前端節點的主機名稱。
4. 使用瀏覽器開啟位於 http://HOSTNAME:8888 的 Hue 入口網站。 以您在先前步驟取得的名稱取代 HOSTNAME。

   > [!NOTE]
   > 當您第一次登入時，系統會提示您建立帳戶來登入 Hue 入口網站。 您在此處指定的認證會限制為入口網站，並且與佈建叢集時您指定的系統管理員或 SSH 使用者認證不相關。
   >
   >

    ![登入 Hue 入口網站](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-login.png "指定 Hue 入口網站的認證")

### <a name="run-a-hive-query"></a>執行 HIVE 查詢
1. 從 Hue 入口網站中，按一下 [查詢編輯器]，然後按一下 [Hive] 開啟 Hive 編輯器。

    ![使用 Hive](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-use-hive.png "使用 Hive")
2. 在 [協助] 索引標籤中，於 [資料庫] 底下，您應該會看到 **hivesampletable**。 這是 HDInsight 上的所有 Hadoop 叢集隨附的範例資料表。 在右窗格中輸入範例查詢，然後在下方窗格的 [結果]  索引標籤中查看輸出，如螢幕擷取畫面所示。

    ![執行 Hive 查詢](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-hive-query.png "執行 Hive 查詢")

    您也可以使用 [圖表]  索引標籤來查看結果的視覺表示。

### <a name="browse-the-cluster-storage"></a>瀏覽叢集儲存體
1. 從 Hue 入口網站中，按一下功能表列右上角的 [檔案瀏覽器]  。
2. 根據預設，檔案瀏覽器會在 **/user/myuser** 目錄中開啟。 按一下路徑中使用者目錄前面的正斜線，以移至與叢集相關聯的 Azure 儲存體容器的根目錄。

    ![使用檔案瀏覽器](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-file-browser.png "使用檔案瀏覽器")
3. 以滑鼠右鍵按一下檔案或資料夾，以查看可用的作業。 使用右邊的 [上傳]  按鈕，將檔案上傳至目前的目錄。 使用 [新增]  按鈕建立新的檔案或目錄。

> [!NOTE]
> Hue 檔案瀏覽器只會顯示與 HDInsight 叢集相關聯的預設容器的內容。 已與叢集相關聯的任何額外儲存體帳戶/容器將無法使用檔案瀏覽器存取。 不過，與叢集相關聯的其他容器一律可供 Hive 工作存取。 例如，如果您在 Hive 編輯器中輸入 `dfs -ls wasb://newcontainer@mystore.blob.core.windows.net` 命令，則您也可以看到其他容器的內容。 在這個命令中， **newcontainer** 不是與叢集相關聯的預設容器。
>
>

## <a name="important-considerations"></a>重要考量︰
1. 用來安裝 Hue 的指令碼只會將它安裝在叢集的主要前端節點上。

2. 在安裝期間，會重新啟動多個 Hadoop 服務 (HDFS、YARN、MR2、Oozie) 以更新與設定。 指令碼完成安裝 Hue 之後，可能需要一些時間讓其他 Hadoop 服務啟動。 一開始可能會影響 Hue 的效能。 所有服務啟動之後，Hue 就可以完全正常運作。
3. Hue 不了解 Tez 作業，這是 Hive 目前的預設值。 如果您想要使用 MapReduce 做為 Hive 執行引擎，請更新指令碼以在您的指令碼中使用下列命令：

        set hive.execution.engine=mr;

4. 使用 Linux 叢集，您就可以擁有在主要前端節點上執行服務，在次要前端節點上執行 Resource Manager 的案例。 使用 Hue 以檢視叢集上「執行中」工作的詳細資料時，此類案例可能會導致錯誤 (如下所示)。 不過，您可以在工作完成時檢視工作詳細資料。

   ![Hue 入口網站錯誤](./media/hdinsight-hadoop-hue-linux/hdinsight-hue-portal-error.png "Hue 入口網站錯誤")

   這是由已知問題造成的。 因應措施是修改 Ambari，讓作用中的 Resource Manager 也在主要前端節點上執行。
5. 當 HDInsight 叢集使用 Azure 儲存體 (使用 `wasb://`) 時，色調能了解 WebHDFS。 因此，搭配指令碼動作使用的自訂指令碼會安裝 WebWasb，這是針對與 WASB 通訊的 WebHDFS 相容服務。 所以，即使在 Hue 入口網站顯示有 HDFS (例如將滑鼠移至 [檔案瀏覽器] 時)，應將它解讀成 WASB。

## <a name="next-steps"></a>後續步驟
* [在 HDInsight 叢集上安裝 Giraph](hdinsight-hadoop-giraph-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Giraph。 Giraph 可讓您利用 Hadoop 執行圖形處理，且可以搭配 Azure HDInsight 一起使用。
* [在 HDInsight 叢集上安裝 Solr](hdinsight-hadoop-solr-install-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 Solr。 Solr 可讓您對儲存的資料執行功能強大的搜尋作業。
* [在 HDInsight 叢集上安裝 R](hdinsight-hadoop-r-scripts-linux.md)。 在 HDInsight Hadoop 叢集上使用叢集自訂安裝 R。 R 是一個用於統計計算的開放原始碼語言和環境。 它提供數百個內建的統計函式及它自己的程式設計語言，此語言結合了函式型和物件導向程式設計的層面。 它也提供廣泛的圖形功能。

[powershell-install-configure]: install-configure-powershell-linux.md
[hdinsight-provision]: hdinsight-provision-clusters-linux.md
[hdinsight-cluster-customize]: hdinsight-hadoop-customize-cluster-linux.md
