---
title: 'Azure Toolkit for Eclipse：：建立適用於 HDInsight Spark 的 Scala 應用程式 '
description: 使用 HDInsight 工具 (位於 Eclipse 的 Azure 工具組中) 來開發以 Scala 撰寫的 Spark 應用程式，並直接從 Eclipse IDE 將它們提交到 HDInsight Spark 叢集。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/30/2017
ms.author: jasonh
ms.openlocfilehash: 836bdccbf3f8887a47da38b47b414722c878be04
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43046007"
---
# <a name="use-azure-toolkit-for-eclipse-to-create-spark-applications-for-an-hdinsight-cluster"></a>使用適用於 Eclipse 的 Azure 工具組建立適用於 HDInsight 叢集的 Spark 應用程式

使用 HDInsight 工具 (位於適用於 Eclipse 的 Azure 工具組中) 來開發以 Scala 撰寫的 Spark 應用程式，並直接從 Eclipse IDE 將它們提交到 Azure HDInsight Spark 叢集。 您能以數種不同的方式使用 HDInsight 工具外掛程式：

* 在 HDInsight Spark 叢集上開發並提交 Scala Spark 應用程式
* 存取您的 Azure HDInsight Spark 叢集資源
* 在本機開發並執行 Scala Spark 應用程式

> [!IMPORTANT]
> 您可以使用此工具，僅針對 Linux 上的 HDInsight Spark 叢集建立並提交應用程式。
> 
> 

## <a name="prerequisites"></a>必要條件

* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。
* Oracle Java Development Kit 第 8 版，可用於 Eclipse IDE 執行階段。 您可以從 [Oracle 網站](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下載。
* Eclipse IDE。 本文使用的是 Eclipse Neon。 您可以從 [Eclipse 網站](https://www.eclipse.org/downloads/)下載。



## <a name="install-hdinsight-tools-in-azure-toolkit-for-eclipse-and-the-scala-plug-in"></a>安裝適用於 Eclipse 和 Scala 外掛程式之 Azure 工具組中的 HDInsight 工具

### <a name="install-azure-toolkit-for-eclipse"></a>安裝適用於 Eclipse 的 Azure 工具組
適用於 Eclipse 的 HDInsight 工具是適用於 Eclipse 的 Azure 工具組的一部分。 如需安裝指示，請參閱[安裝適用於 Eclipse 的 Azure 工具組](https://docs.microsoft.com/java/azure/eclipse/azure-toolkit-for-eclipse-installation)。

### <a name="install-the-scala-plug-in"></a>安裝 Scala 外掛程式
當您開啟 Eclipse 時，HDInsight 工具會自動偵測您是否已安裝 Scala 外掛程式。 選取 [確定] 以繼續，然後遵循指示從 Eclipse Marketplace 安裝外掛程式。

![自動安裝 Scala 外掛程式](./media/apache-spark-eclipse-tool-plugin/auto-install-scala.png)

使用者可以[登入 Azure 訂用帳戶](#Sign-in-to-your-Azure-subscription)，或是使用 Ambari 使用者名稱/密碼或加入網域的認證開始[連結 HDInsight 叢集](#Link-a-cluster)。 

## <a name="sign-in-to-your-azure-subscription"></a>登入您的 Azure 訂用帳戶：
1. 啟動 Eclipse IDE，然後開啟 [Azure Explorer]。 在 [視窗] 功能表上，選取 [顯示檢視]，然後選取 [其他]。 在開啟的對話方塊中展開 [Azure]，然後依序選取 [Azure Explorer] 和 [確定]。

   ![顯示檢視對話方塊](./media/apache-spark-eclipse-tool-plugin/view-explorer-1.png)
1. 以滑鼠右鍵按一下 [Azure] 節點，然後選取 [登入]。
1. 在 [Azure 登入] 對話方塊中，選擇 [驗證方法]、選取 [登入]，並輸入您的 Azure 認證。
   
   ![[Azure 登入] 對話方塊](./media/apache-spark-eclipse-tool-plugin/view-explorer-2.png)
1. 登入之後，[選取訂用帳戶] 對話方塊會列出與認證相關聯的所有 Azure 訂用帳戶。 按一下 [選取] 以關閉對話方塊。

   ![[選取訂用帳戶] 對話方塊](./media/apache-spark-eclipse-tool-plugin/Select-Subscriptions.png)
1. 在 [Azure Explorer] 索引標籤中展開 [HDInsight]，可查看您訂用帳戶下的 HDInsight Spark 叢集。
   
   ![Azure Explorer 中的 HDInsight Spark 叢集](./media/apache-spark-eclipse-tool-plugin/view-explorer-3.png)
1. 您可以進一步展開叢集名稱節點，查看與叢集相關聯的資源 (例如，儲存體帳戶)。
   
   ![展開叢集名稱以查看資源](./media/apache-spark-eclipse-tool-plugin/view-explorer-4.png)

## <a name="link-a-cluster"></a>連結叢集
您可以使用 Ambari 受控使用者名稱來連結一般的叢集。 同樣地，對於已加入網域的 HDInsight 叢集，您可以使用網域和使用者名稱進行連結，例如 user1@contoso.com。

1. 從 [Azure 總管] 中選取 [連結叢集]。

   ![連結叢集操作功能表](./media/apache-spark-intellij-tool-plugin/link-a-cluster-context-menu.png)

1. 輸入 [叢集名稱]、[使用者名稱] 及 [密碼]，然後按一下 [確定] 按鈕連結叢集。 您也可以選擇輸入 [儲存體帳戶]、[儲存體金鑰]，然後選取 [儲存體容器]，讓儲存體總管在左側樹狀檢視中工作
   
   ![連結叢集對話方塊](./media/apache-spark-eclipse-tool-plugin/link-cluster-dialog.png)
   
   > [!NOTE]
   > 如果叢集已登入 Azure 訂用帳戶並連結叢集，我們會使用連結的儲存體金鑰、使用者名稱和密碼。
   > ![Eclipse 中的儲存體總管](./media/apache-spark-eclipse-tool-plugin/storage-explorer-in-Eclipse.png)

1. 如果輸入資訊正確無誤，則按一下 [確定] 按鈕之後，您就可以在 [HDInsight] 節點中看見連結的叢集。 您現在可以將應用程式提交至此連結的叢集。

   ![連結的叢集](./media/apache-spark-intellij-tool-plugin/linked-cluster.png)

1. 您也可以從 [Azure 總管] 取消連結叢集。
   
   ![取消連結的叢集](./media/apache-spark-intellij-tool-plugin/unlink.png)


## <a name="set-up-a-spark-scala-project-for-an-hdinsight-spark-cluster"></a>設定 HDInsight Spark 叢集的 Spark Scala 專案

1. 在 Eclipse IDE 工作區中，依序選取 [檔案]、[新增]，然後選取 [專案]。 
1. 在 [新增專案] 精靈中展開 [HDInsight]、選取 [HDInsight 上的 Spark (Scala)]，然後選取 [下一步]。

   ![選取 Spark HDInsight (Scala) 專案](./media/apache-spark-eclipse-tool-plugin/create-hdi-scala-app-2.png)
1. Scala 專案建立精靈會自動偵測您是否已安裝 Scala 外掛程式。 選取 [確定] 以繼續下載 Scala 外掛程式，並遵循指示重新啟動 Eclipse。

   ![Scala 檢查](./media/apache-spark-eclipse-tool-plugin/auto-install-scala-2.png)
1. 在 [新增 HDInsight Scala 專案] 對話方塊中，提供下列值，然後選取 [下一步]：
   * 輸入專案的名稱。
   * 在 [JRE] 區域中，請確定已將 [使用執行環境 JRE] 設為 [JavaSE-1.7] 或更新版本。
   * 在 [Spark 程式庫] 區域中，您可以選擇 [使用 Maven 設定 Spark SDK ] 選項。  我們的工具為 Spark SDK 和 Scala SDK 整合正確的版本。 您也可以選擇 [手動新增 Spark SDK] 選項，手動下載並新增 Spark SDK。

   ![[新增 HDInsight Scala 專案] 對話方塊](./media/apache-spark-eclipse-tool-plugin/create-hdi-scala-app-3.png)
1. 在下一個對話方塊中，選取 完成。 
   
  
## <a name="create-a-scala-application-for-an-hdinsight-spark-cluster"></a>建立 HDInsight Spark 叢集的 Scala 應用程式

1. 在 Eclipse IDE 中，從 [套件總管] 將您稍早建立的專案展開，以滑鼠右鍵按一下 [src]、指向 [新增]，然後選取 [其他]。
1. 在 [選取精靈] 對話方塊方塊中，展開 [Scala 精靈]、選取 [Scala 物件]，然後選取 [下一步]。
   
   ![選取精靈對話方塊](./media/apache-spark-eclipse-tool-plugin/create-scala-proj-1.png)
1. 在 [建立新檔案] 對話方塊中輸入物件的名稱，然後選取 [完成]。
   
   ![[建立新檔案] 對話方塊](./media/apache-spark-eclipse-tool-plugin/create-scala-proj-2.png)
1. 在文字編輯器中貼上下列程式碼：
   
        import org.apache.spark.SparkConf
        import org.apache.spark.SparkContext
   
        object MyClusterApp{
          def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("MyClusterApp")
            val sc = new SparkContext(conf)
   
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
   
            //find the rows that have only one digit in the seventh column in the CSV
            val rdd1 =  rdd.filter(s => s.split(",")(6).length() == 1)
   
            rdd1.saveAsTextFile("wasb:///HVACOut")
          }        
        }
1. 在 HDInsight Spark 叢集上執行應用程式：
   
   a. 從 [封裝總管] 中，以滑鼠右鍵按一下專案名稱，然後選取 [將 Spark 應用程式提交給 HDInsight]。        
   b. 在 [提交 Spark] 對話方塊中提供下列值，然後選取 [提交]：
      
      * 針對 **叢集名稱**，選取您要在其上執行應用程式的 HDInsight Spark 叢集。
      * 從 Eclipse 專案中選取構件，或從硬碟中選取一個。 預設值取決於您在套件總管中以滑鼠右鍵按一下的項目。
      * 在 [主要類別名稱] 下拉式清單中，提交精靈會顯示您的所有物件名稱。 選取或輸入您需要執行的物件名稱。 如果您從硬碟選取構件，就必須手動輸入主要的類別名稱。 
      * 因為此範例中的應用程式程式碼不需要任何命令列引數，或是參考 JAR 或檔案，所以您可以將其餘的文字方塊保留空白。
        
      ![[提交 Spark] 對話方塊](./media/apache-spark-eclipse-tool-plugin/create-scala-proj-3.png)
1. [Spark Submission (提交 Spark)]  索引標籤應會開始顯示進度。 您可以選取 [提交 Spark] 視窗中的紅色按鈕，就能將應用程式停止。 您也可以選取全球圖示 (以映像中的藍色方塊表示)，檢視此特定應用程式執行的記錄。
      
   ![[提交 Spark] 視窗](./media/apache-spark-eclipse-tool-plugin/create-scala-proj-4.png)


## <a name="access-and-manage-hdinsight-spark-clusters-by-using-hdinsight-tools-in-azure-toolkit-for-eclipse"></a>使用適用於 Eclipse 的 Azure 工具組中的 HDInsight 工具來存取和管理 HDInsight Spark 叢集
您可以使用 HDInsight 工具來執行各種作業，包括存取作業輸出。

### <a name="access-the-job-view"></a>存取作業檢視
1. 在 [Azure Explorer] 中，依序展開 [HDInsight] 和 Spark 叢集名稱，然後選取 [作業]。 

   ![作業檢視節點](./media/apache-spark-eclipse-tool-plugin/job-view-node.png)

1. 選取 [作業] 節點。 如果 Java 版本低於 **1.8**，HDInsight 工具會自動提醒您安裝 **E(fx)clipse** 外掛程式。 選取 [確定] 繼續作業，然後遵循精靈安裝 Eclipse Marketplace 並重新啟動 Eclipse。 

   ![安裝 E(fx)clipse](./media/apache-spark-eclipse-tool-plugin/auto-install-efxclipse.png)

1. 從 [作業] 節點開啟 [作業] 檢視。 在右窗格中，[Spark 作業檢視]  索引標籤會顯示已在叢集上執行的所有應用程式。 選取您想要查看更多詳細資料的應用程式名稱。

   ![應用程式詳細資料](./media/apache-spark-eclipse-tool-plugin/view-job-logs.png)

   然後您可以採取下列任何動作：

   * 將滑鼠停留於作業圖表。 它會顯示關於執行中作業的基本資訊。 選取作業圖表，就可以看到每項作業產生的階段和資訊。

     ![作業階段詳細資料](./media/apache-spark-eclipse-tool-plugin/Job-graph-stage-info.png)

   * 選取 [記錄] 索引標籤可檢視經常使用的記錄 (包括「驅動程式 Stderr」、「驅動程式 Stdout」和「目錄資訊」)。

     ![記錄詳細資料](./media/apache-spark-eclipse-tool-plugin/Job-log-info.png)

   * 選取視窗頂端的超連結，在應用程式層級開啟 Spark 歷程記錄 UI 和 YARN UI。

### <a name="access-the-storage-container-for-the-cluster"></a>存取叢集的儲存體容器
1. 在 [Azure Explorer] 中展開 [HDInsight] 根節點，查看可用的 HDInsight Spark 叢集清單。
1. 展開叢集名稱以查看叢集的儲存體帳戶和預設儲存體容器。
   
   ![儲存體帳戶和預設儲存體容器](./media/apache-spark-eclipse-tool-plugin/view-explorer-5.png)
1. 選取與叢集相關聯的儲存體容器名稱。 在右窗格中，按兩下 **HVACOut** 資料夾。 開啟其中一個 **part-** 檔案，可查看應用程式的輸出。

### <a name="access-the-spark-history-server"></a>存取 Spark 歷程記錄伺服器
1. 在 [Azure Explorer] 中，以滑鼠右鍵按一下您的 Spark 叢集名稱，然後選取 [開啟 Spark 歷程記錄 UI]。 出現提示時，輸入叢集的系統管理員認證。 在佈建叢集時，您已指定這些項目。
1. 在 [Spark 歷程記錄伺服器] 儀表板中，您可以使用應用程式名稱，尋找您剛完成執行的應用程式。 在上述程式碼中，您可以使用 `val conf = new SparkConf().setAppName("MyClusterApp")`來設定應用程式名稱。 因此，Spark 應用程式名稱為 **MyClusterApp**。

### <a name="start-the-ambari-portal"></a>啟動 Ambari 入口網站
1. 在 [Azure Explorer] 中，以滑鼠右鍵按一下您的 Spark 叢集名稱，然後選取 [開啟叢集管理入口網站 (Ambari)]。 
1. 出現提示時，輸入叢集的系統管理員認證。 在佈建叢集時，您已指定這些項目。

### <a name="manage-azure-subscriptions"></a>管理 Azure 訂用帳戶
根據預設，Azure 工具組中適用於 Eclipse 的 HDInsight 工具會列出您所有 Azure 訂用帳戶中的 Spark 叢集。 如有必要，您可以指定要存取其叢集的訂用帳戶。 

1. 在 [Azure Explorer] 中，以滑鼠右鍵按一下 [Azure] 根節點，然後選取 [管理訂用帳戶]。 
1. 在對話方塊中，清除您不需要存取的訂用帳戶核取方塊，然後選取 [關閉]。 如果您想要登出 Azure 訂用帳戶，也可以選取 [登出]。

## <a name="run-a-spark-scala-application-locally"></a>在本機執行 Spark Scala 應用程式
您可以使用適用於 Eclipse 的 Azure 工具組中之 HDInsight 工具，在工作站上本機執行 Spark Scala 應用程式。 一般而言，這些應用程式不需要存取叢集資源 (例如儲存體容器)，而且您可在本機上執行並測試。

### <a name="prerequisite"></a>必要條件
在 Windows 電腦上執行本機 Spark Scala 應用程式時，可能會發生如 [SPARK-2356](https://issues.apache.org/jira/browse/SPARK-2356) 中所述的例外狀況。 發生這個例外狀況是因為 Windows 中遺失 **WinUtils.exe**。 

若要解決這個錯誤，必須[下載可執行檔](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe)到例如 **C:\WinUtils\bin** 的位置，然後新增 **HADOOP_HOME** 環境變數，並將變數的值設為 **C\WinUtils**。

### <a name="run-a-local-spark-scala-application"></a>執行本機 Spark Scala 應用程式
1. 啟動 Eclipse，然後建立專案。 在 [新增專案] 對話方塊中選取下列選項，然後選取 [下一步]。
   
   * 在左窗格中，選取 [HDInsight]。
   * 在右窗格中，選取 [HDInsight 上的 Spark 本機執行範例 (Scala)]。

   ![New Project dialog box](./media/apache-spark-eclipse-tool-plugin/hdi-spark-app-local-run.png)
   
1. 若要提供專案的詳細資料，請遵循稍早的[設定 HDInsight Spark 叢集的 Spark Scala 專案](#set-up-a-spark-scala-project-for-an-hdinsight-spark-cluster)一節中的步驟 3 到 6。

1. 範本會在 **src** 資料夾下方新增可在電腦本機執行的程式碼範例 (**LogQuery**)。
   
   ![LogQuery 的位置](./media/apache-spark-eclipse-tool-plugin/local-app.png)
   
1. 以滑鼠右鍵按一下 **LogQuery** 應用程式、指向 [執行身分]，然後選取 [1 Scala 應用程式]。 在 [主控台] 索引標籤上隨即顯示像這樣的輸出：
   
   ![Spark 應用程式本機執行結果](./media/apache-spark-eclipse-tool-plugin/hdi-spark-app-local-run-result.png)

## <a name="known-problems"></a>已知的問題
連結叢集時，建議您提供儲存體認證。

![互動式登入](./media/apache-spark-eclipse-tool-plugin/link-cluster-with-storage-credential-eclipse.png)

提交工作有兩種模式。 如果提供儲存體認證，將使用批次模式提交工作。 否則，將使用互動模式。 如果叢集忙碌，您可能會收到以下錯誤。

![Eclipse 會在叢集忙碌時收到錯誤](./media/apache-spark-eclipse-tool-plugin/eclipse-interactive-cluster-busy-upload.png)

![Eclipse 會在叢集忙碌時收到錯誤](./media/apache-spark-eclipse-tool-plugin/eclipse-interactive-cluster-busy-submit.png)

## <a name="feedback"></a>意見反應
如果您有任何意見反應，或在使用此工具時遇到任何其他問題，請傳送電子郵件到 hdivstool@microsoft.com。

## <a name="seealso"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](apache-spark-custom-library-website-log-analysis.md)

### <a name="creating-and-running-applications"></a>建立和執行應用程式
* [使用 Scala 建立獨立應用程式](apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [使用適用於 IntelliJ 的 Azure 工具組中來建立和提交 Spark Scala 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用適用於 IntelliJ 的 Azure 工具組透過 VPN 對 Spark 應用程式進行遠端偵錯](../hdinsight-apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [使用適用於 IntelliJ 的 Azure 工具組透過 SSH 對 Spark 應用程式進行遠端偵錯](../hdinsight-apache-spark-intellij-tool-debug-remotely-through-ssh.md)
* [透過 Hortonworks 沙箱使用 HDInsight Tools for IntelliJ](../hadoop/hdinsight-tools-for-intellij-with-hortonworks-sandbox.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="managing-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)

