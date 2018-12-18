---
title: 'Azure Toolkit for IntelliJ：在 HDInsight Spark 中遠端偵錯應用程式 '
description: 了解如何使用適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具，透過 VPN 對 HDInsight 叢集上執行的 Spark 應用程式進行遠端偵錯。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 11/28/2017
ms.openlocfilehash: 1a7fcc11bf1b98d5feaeae07264e556acce2b14b
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45736067"
---
# <a name="use-azure-toolkit-for-intellij-to-debug-spark-applications-remotely-in-hdinsight-through-vpn"></a>使用適用於 IntelliJ 的 Azure 工具組透過 VPN 對 HDInsight 中的 Spark 應用程式進行遠端偵錯

建議透過 SSH 對 Spark 應用程式進行遠端偵錯。 如需指示，請參閱[使用適用於 IntelliJ 的 Azure 工具組透過 SSH 對 HDInsight 叢集上的 Spark 應用程式進行遠端偵錯](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-intellij-tool-debug-remotely-through-ssh)。

本文提供逐步指引，說明如何使用適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具，來提交 HDInsight Spark 叢集上的 Spark 作業，然後從桌上型電腦遠端偵錯。 若要完成這些工作，您必須執行下列高階步驟：

1. 建立站對站或點對站 Azure 虛擬網路。 本文件中的步驟假設您使用站對站網路。
1. 在 HDInsight 中建立 Spark 叢集是站對站虛擬網路的一部分。
1. 請確認叢集前端節點與您桌上型電腦之間的連線能力。
1. 在 IntelliJ IDEA 中建立 Scala 應用程式，然後設定它以進行遠端偵錯。
1. 執行和偵錯應用程式。

## <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶**。 如需詳細資訊，請參閱[取得 Azure 的免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **HDInsight 中的 Apache Spark 叢集**。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。
* **Oracle JAVA Development Kit**。 您可以從 [Oracle 網站](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下載。
* **IntelliJ IDEA**。 本文章使用 2017.1 版。 您可以從 [JetBrains 網站](https://www.jetbrains.com/idea/download/)下載。
* **適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具**。 適用於 IntelliJ 的 HDInsight 工具是適用於 IntelliJ 的 Azure 工具組的一部分。 如需有關如何安裝 Azure 工具組的指示，請參閱[安裝適用於 IntelliJ 的 Azure 工具組](https://docs.microsoft.com/java/azure/intellij/azure-toolkit-for-intellij-installation)。
* **從 IntelliJ IDEA 登入您的 Azure 訂用帳戶**。 請遵循[使用適用於 IntelliJ 的 Azure 工具組建立適用於 HDInsight 叢集的 Spark 應用程式](apache-spark-intellij-tool-plugin.md)中的指示。
* **例外狀況的因應措施**。 在 Windows 電腦上執行 Spark Scala 應用程式以進行遠端偵錯時，可能會發生例外狀況。 這個例外狀況會在 [SPARK 2356](https://issues.apache.org/jira/browse/SPARK-2356) 中說明，並會因 Windows 中的 WinUtils.exe 檔案遺失而發生。 若要解決這個錯誤，您必須[下載可執行檔](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe)，並將其放至如 **C:\WinUtils\bin** 的位置。 新增環境變數 **HADOOP_HOME**，然後將變數的值設為 **C\WinUtils**。

## <a name="step-1-create-an-azure-virtual-network"></a>步驟 1：建立 Azure 虛擬網路
請遵循下列連結的指示來建立 Azure 虛擬網路，然後確認桌上型電腦和虛擬網路之間的連線能力：

* [使用 Azure 入口網站建立具有站對站 VPN 連線的 VNet](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [使用 PowerShell 建立具有站對站 VPN 連線的 VNet](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md)
* [使用 PowerShell 設定虛擬網路的點對站連線](../../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

## <a name="step-2-create-an-hdinsight-spark-cluster"></a>步驟 2：建立 HDInsight Spark 叢集
建議您也要在 Azure HDInsight 上建立 Apache Spark 叢集，這是您所建立之 Azure 虛擬網路的一部分。 使用[在 HDInsight 中建立以 Linux 作為基礎的叢集](../hdinsight-hadoop-provision-linux-clusters.md)中可取得的資訊。 作為選擇性組態的一部分，選取您在上一個步驟中建立的 Azure 虛擬網路。

## <a name="step-3-verify-the-connectivity-between-the-cluster-head-node-and-your-desktop"></a>步驟 3：請確認叢集前端節點與您桌上型電腦之間的連線能力
1. 取得前端節點的 IP 位址。 開啟叢集的 Ambari UI。 從叢集刀鋒視窗中，選取 [儀表板]。

    ![在 Ambari 中選取儀表板](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/launch-ambari-ui.png)
1. 從 Ambari UI 中，選取 [主機]。

    ![在 Ambari 中選取主機](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/ambari-hosts.png)
1. 您會看到前端節點、背景工作角色節點和 zookeeper 節點的清單。 前端節點有 **hn*** 前置詞。 選取第一個前端節點。

    ![在 Ambari 中尋找前端節點](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/cluster-headnodes.png)
1. 從開啟的頁面底部 [摘要] 窗格，複製前端節點的 **IP 位址**和**主機名稱**。

    ![在 Ambari 中尋找 IP 位址](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/headnode-ip-address.png)
1. 將前端節點的 IP 位址和主機名稱新增到電腦 (您需要從中執行和遠端偵錯 Spark 作業) 上的**主機**檔案。 這可讓您使用 IP 位址和主機名稱，與前端節點進行通訊。

   a. 以提高的權限開啟 [記事本] 檔案。 從 [檔案] 功能表中，選取 [開啟]，然後尋找主機檔案的位置。 在 Windows 電腦上的位置是 **C:\Windows\System32\Drivers\etc\hosts**。

   b. 將下列資訊新增至**主機**檔案：

           # For headnode0
           192.xxx.xx.xx hn0-nitinp
           192.xxx.xx.xx hn0-nitinp.lhwwghjkpqejawpqbwcdyp3.gx.internal.cloudapp.net

           # For headnode1
           192.xxx.xx.xx hn1-nitinp
           192.xxx.xx.xx hn1-nitinp.lhwwghjkpqejawpqbwcdyp3.gx.internal.cloudapp.net
1. 從您連線到 HDInsight 叢集所使用之 Azure 虛擬網路的電腦，確認您可以使用 IP 位址和主機名稱來偵測兩個前端節點。
1. 遵循[使用 SSH 連線到 HDInsight 叢集](../hdinsight-hadoop-linux-use-ssh-unix.md)中的指示，使用 SSH 連線至叢集前端節點。 從叢集前端節點，偵測桌上型電腦的 IP 位址。 測試指派給電腦之兩個 IP 位址的連線能力：

    - 一個是網路連線
    - 一個是 Azure 虛擬網路

1. 對其他前端節點重複這些步驟。

## <a name="step-4-create-a-spark-scala-application-by-using-hdinsight-tools-in-azure-toolkit-for-intellij-and-configure-it-for-remote-debugging"></a>步驟 4：使用適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具來建立 Spark Scala 應用程式，並設定它以進行遠端偵錯
1. 開啟 IntelliJ IDEA，並建立新的專案。 在 [新增專案]  對話方塊中，執行下列操作：

    ![在 IntelliJ IDEA 中選取新的專案範本](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/create-hdi-scala-app.png)

    a. 選取 [HDInsight] > [HDInsight 上的 Spark (Scala)]。

    b. 選取 [下一步] 。
1. 在下一個 [新增專案] 對話方塊中，執行下列操作，然後選取 [完成]：

    - 輸入專案名稱和位置。

    - 在 [專案 SDK] 下拉式清單中，選取 [Java 1.8] \(適用於 Spark 2.x 叢集)，或選取 [Java 1.7] \(適用於 Spark 1.x 叢集)。

    - 在 [Spark 版本] 下拉式清單中，Scala 專案建立精靈會為 Spark SDK 和 Scala SDK 整合正確的版本。 如果 Spark 叢集是 2.0 以前的版本，請選取 [Spark 1.x]。 否則，請選取 [Spark2.x]。 此範例使用 **Spark 2.0.2 (Scala 2.11.8)**。
  
   ![選取專案 SDK 和 Spark 版本](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/hdi-scala-project-details.png)
  
1. Spark 專案會自動為您建立構件。 若要檢視構件，請執行下列動作：

    a. 從 [檔案] 功能表，選取 [專案結構]。

    b. 在 [專案結構] 對話方塊中，選取 [構件]，以檢視建立的預設構件。 您也可以建立自己的構件，方法是選取加號 (**+**)。

   ![建立 JAR](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/default-artifact.png)


1. 將程式庫新增至專案。 若要新增文件庫，請執行下列操作：

    a. 以滑鼠右鍵按一下專案樹狀結構中的專案名稱，然後選取 [開啟模組設定]。 

    b. 在 [專案結構] 對話方塊中，依序選取 [文件庫]**、**([+]) 符號，然後選取 [從 Maven]。

    ![新增文件庫](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/add-library.png)

    c. 在 [從 Maven 存放庫下載文件庫]  對話方塊中，搜尋並新增下列文件庫：

   * `org.scalatest:scalatest_2.10:2.2.1`
   * `org.apache.hadoop:hadoop-azure:2.7.1`
1. 從叢集前端節點複製 `yarn-site.xml` 和 `core-site.xml`，並將它們新增至專案。 使用下列命令來複製檔案。 您可以使用 [Cygwin](https://cygwin.com/install.html) 來執行下列 `scp` 命令，從叢集前端節點複製檔案：

        scp <ssh user name>@<headnode IP address or host name>://etc/hadoop/conf/core-site.xml .

    因為我們已新增桌上型電腦之主機檔案的叢集前端節點 IP 位址及主機名稱，因此可以下列方式使用 `scp` 命令：

        scp sshuser@hn0-nitinp:/etc/hadoop/conf/core-site.xml .
        scp sshuser@hn0-nitinp:/etc/hadoop/conf/yarn-site.xml .

    若要將這些檔案新增至專案，請在您專案樹狀結構的 **/src** 資料夾下複製這些檔案，例如 `<your project directory>\src`。
1. 更新 `core-site.xml` 檔案以進行下列變更：

   a. 取代加密的金鑰。 `core-site.xml` 檔案包含與叢集相關聯之儲存體帳戶的加密金鑰。 在您新增至專案的 `core-site.xml` 檔案中，以與預設儲存體帳戶相關聯的實際儲存體金鑰取代加密金鑰。 如需詳細資訊，請參閱[管理儲存體存取金鑰](../../storage/common/storage-account-manage.md#access-keys)。

           <property>
                 <name>fs.azure.account.key.hdistoragecentral.blob.core.windows.net</name>
                 <value>access-key-associated-with-the-account</value>
           </property>
   b. 從 `core-site.xml` 移除下列項目：

           <property>
                 <name>fs.azure.account.keyprovider.hdistoragecentral.blob.core.windows.net</name>
                 <value>org.apache.hadoop.fs.azure.ShellDecryptionKeyProvider</value>
           </property>

           <property>
                 <name>fs.azure.shellkeyprovider.script</name>
                 <value>/usr/lib/python2.7/dist-packages/hdinsight_common/decrypt.sh</value>
           </property>

           <property>
                 <name>net.topology.script.file.name</name>
                 <value>/etc/hadoop/conf/topology_script.py</value>
           </property>
   c. 儲存檔案。
1. 新增您應用程式的主要類別。 從**專案總管**，以滑鼠右鍵按一下 [src]、指向 [新增]，然後選取 [Scala 類別]。

    ![選取主要類別](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/hdi-spark-scala-code.png)
1. 在 [建立新的 Scala 類別] 對話方塊中，提供一個名稱，並在 [種類] 方塊中選取 [物件]，然後選取 [確定]。

    ![建立新的 Scala 類別](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/hdi-spark-scala-code-object.png)
1. 在 `MyClusterAppMain.scala` 檔案中，貼上下列程式碼。 此程式碼會建立 Spark 內容，並且從 `SparkSample` 物件開啟 `executeJob` 方法。

        import org.apache.spark.{SparkConf, SparkContext}

        object SparkSampleMain {
          def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("SparkSample")
                                      .set("spark.hadoop.validateOutputSpecs", "false")
            val sc = new SparkContext(conf)

            SparkSample.executeJob(sc,
                                   "wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv",
                                   "wasb:///HVACOut")
          }
        }

1. 重複步驟 8 和 9，以新增稱為 `*SparkSample` 的新 Scala 物件。 將下列程式碼新增至這個類別。 此程式碼會從 HVAC.csv (所有 HDInsight Spark 叢集上均有提供) 讀取資料。 它會擷取在 CSV 檔案的第七個資料行中只有個位數的資料列，然後將輸出寫入叢集預設儲存體容器下的 **/HVACOut** 中。

        import org.apache.spark.SparkContext

        object SparkSample {
         def executeJob (sc: SparkContext, input: String, output: String): Unit = {
           val rdd = sc.textFile(input)

           //find the rows which have only one digit in the 7th column in the CSV
           val rdd1 =  rdd.filter(s => s.split(",")(6).length() == 1)

           val s = sc.parallelize(rdd.take(5)).cartesian(rdd).count()
           println(s)

           rdd1.saveAsTextFile(output)
           //rdd1.collect().foreach(println)
         }
        }
1. 重複步驟 8 和 9，以新增稱為 `RemoteClusterDebugging` 的新類別。 這個類別會實作 Spark 測試架構，可用於偵錯應用程式。 將下列程式碼新增至 `RemoteClusterDebugging` 類別：

        import org.apache.spark.{SparkConf, SparkContext}
        import org.scalatest.FunSuite

        class RemoteClusterDebugging extends FunSuite {

         test("Remote run") {
           val conf = new SparkConf().setAppName("SparkSample")
                                     .setMaster("yarn-client")
                                     .set("spark.yarn.am.extraJavaOptions", "-Dhdp.version=2.4")
                                     .set("spark.yarn.jar", "wasb:///hdp/apps/2.4.2.0-258/spark-assembly-1.6.1.2.4.2.0-258-hadoop2.7.1.2.4.2.0-258.jar")
                                     .setJars(Seq("""C:\workspace\IdeaProjects\MyClusterApp\out\artifacts\MyClusterApp_DefaultArtifact\default_artifact.jar"""))
                                     .set("spark.hadoop.validateOutputSpecs", "false")
           val sc = new SparkContext(conf)

           SparkSample.executeJob(sc,
             "wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv",
             "wasb:///HVACOut")
         }
        }

     以下有幾個重點值得注意：

      * 對於 `.set("spark.yarn.jar", "wasb:///hdp/apps/2.4.2.0-258/spark-assembly-1.6.1.2.4.2.0-258-hadoop2.7.1.2.4.2.0-258.jar")`，請確定 Spark 組件 JAR 可用於位於指定路徑的叢集存放區。
      * 對於 `setJars`，指定要建立構件 JAR 的位置。 通常是 `<Your IntelliJ project directory>\out\<project name>_DefaultArtifact\default_artifact.jar`。
1. 在 `*RemoteClusterDebugging` 類別中，以滑鼠右鍵按一下 `test` 關鍵字，然後選取 [建立 RemoteClusterDebugging 組態]。

    ![建立遠端組態](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/create-remote-config.png)

1. 在 [建立 RemoteClusterDebugging 組態] 對話方塊中，提供組態的名稱，然後選取 [測試種類] 作為 [測試名稱]。 保留所有其他值作為預設設定。 選取 [套用]，然後選取 [確定]。

    ![新增組態詳細資料](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/provide-config-value.png)
1. 您現在應該會在功能表列中看到 [遠端執行] 組態下拉式清單。

    ![[遠端執行] 下拉式清單](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/config-run.png)

## <a name="step-5-run-the-application-in-debug-mode"></a>步驟 5：在偵錯模式中執行應用程式
1. 在您的 IntelliJ IDEA 專案中，開啟 `SparkSample.scala`，並在 `val rdd1` 旁邊建立中斷點。 在快顯功能表的 [建立中斷點] 中，選取 **executeJob 函式中的一行**。

    ![新增中斷點](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/create-breakpoint.png)
1. 若要執行應用程式，請選取 [遠端執行] 組態下拉式清單旁的 [偵錯執行] 按鈕。

    ![選取 [偵錯執行] 按鈕](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-run-mode.png)
1. 當程式執行觸達中斷點時，您會在下方窗格中看到 [偵錯工具] 索引標籤。

    ![檢視 [偵錯工具] 索引標籤](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-add-watch.png)
1. 若要新增監看式，請選取 ([+]) 圖示。

    ![選取 [+] 圖示](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-add-watch-variable.png)

    在此範例中，會在 `rdd1` 變數建立之前中斷應用程式。 使用此監看式，我們可以看到變數 `rdd` 中的前五個資料列。 選取 [輸入]。

    ![在偵錯模式中執行程式](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-add-watch-variable-value.png)

    先前映像中看到的內容是在執行階段，您可以查詢數 TB 的資料，並且針對應用程式的進度進行偵錯。 例如，您可以在先前映像中顯示的輸出，看到輸出的第一個資料列是標頭。 您可以此輸出作為基礎來修改應用程式程式碼，視需要跳過標頭資料列。
1. 您現在可以選取 [繼續程式] 圖示，以繼續執行應用程式。

    ![選取繼續程式](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-continue-run.png)
1. 如果應用程式順利完成，就會看到類似下列的輸出：

    ![主控台輸出](./media/apache-spark-intellij-tool-plugin-debug-jobs-remotely/debug-complete.png)

## <a name="seealso"></a>後續步驟
* [概觀：Azure HDInsight 上的 Apache Spark](apache-spark-overview.md)

### <a name="demo"></a>示範
* 建立 Scala 專案 (影片)：[Create Spark Scala Applications](https://channel9.msdn.com/Series/AzureDataLake/Create-Spark-Applications-with-the-Azure-Toolkit-for-IntelliJ) (建立 Spark Scala 應用程式)
* 遠端偵錯 (影片)：[Use Azure Toolkit for IntelliJ to debug Spark applications remotely on an HDInsight cluster](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) (使用適用於 IntelliJ 的 Azure 工具組對 HDInsight 叢集上的 Spark 應用程式進行遠端偵錯)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](apache-spark-machine-learning-mllib-ipython.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](../hdinsight-apache-spark-custom-library-website-log-analysis.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](../hdinsight-apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [使用適用於 IntelliJ 的 Azure 工具組建立適用於 HDInsight 叢集的 Spark 應用程式](apache-spark-intellij-tool-plugin.md)
* [使用適用於 IntelliJ 的 Azure 工具組透過 SSH 對 Spark 應用程式進行遠端偵錯](apache-spark-intellij-tool-debug-remotely-through-ssh.md)
* [透過 Hortonworks 沙箱使用 HDInsight Tools for IntelliJ](../hadoop/hdinsight-tools-for-intellij-with-hortonworks-sandbox.md)
* [使用適用於 Eclipse 的 Azure 工具組中的 HDInsight 工具建立 Spark 應用程式](../hdinsight-apache-spark-eclipse-tool-plugin.md)
* [利用 HDInsight 中的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
