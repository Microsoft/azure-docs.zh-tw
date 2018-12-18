---
title: 'Azure Toolkit for IntelliJ：透過 SSH 針對 Spark 應用程式進行遠端偵錯 '
description: 逐步指引如何使用適用於 IntelliJ 之 Azure 工具組中的 HDInsight 工具，透過 SSH 對 HDInsight 叢集上的應用程式進行遠端偵錯
keywords: 遠端偵錯 intellij, 進行 intellij 遠端偵錯, ssh, intellij, hdinsight, 偵錯 intellij, 偵錯
services: hdinsight
ms.service: hdinsight
author: jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 11/25/2017
ms.openlocfilehash: f26355f32eeeb6b6e195b838bed502271533f4f4
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047735"
---
# <a name="debug-spark-applications-locally-or-remotely-on-an-hdinsight-cluster-with-azure-toolkit-for-intellij-through-ssh"></a>使用適用於 IntelliJ 的 Azure 工具組透過 SSH 對 HDInsight 叢集上的 Spark 應用程式進行本機或遠端偵錯

本文提供的逐步指引，是關於如何使用適用於 IntelliJ 之 Azure 工具組中的 HDInsight 工具，對 HDInsight 叢集上的應用程式進行遠端偵錯。 若要對您的專案進行偵錯，您也可以觀看 [Debug HDInsight Spark applications with Azure Toolkit for IntelliJ](https://channel9.msdn.com/Series/AzureDataLake/Debug-HDInsight-Spark-Applications-with-Azure-Toolkit-for-IntelliJ) (使用適用於 IntelliJ 的 Azure 工具組對 HDInsight Spark 應用程式進行偵錯) 影片。

**先決條件**
* **適用於 IntelliJ 的 Azure 工具組中的 HDInsight 工具**。 此工具是適用於 IntelliJ 之 Azure 工具組的一部分。 如需詳細資訊，請參閱[安裝適用於 IntelliJ 的 Azure 工具組](https://docs.microsoft.com/azure/azure-toolkit-for-intellij-installation)。 以及**適用於 IntelliJ 的 Azure 工具組**。 使用此工具組建立適用於 HDInsight 叢集的 Spark 應用程式。 如需詳細資訊，請遵循[使用適用於 IntelliJ 的 Azure 工具組建立適用於 HDInsight 叢集的 Spark 應用程式](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-intellij-tool-plugin)中的指示進行。

* **HDInsight SSH 服務與使用者名稱和密碼管理**。 如需詳細資訊，請參閱[使用 SSH 連線到 HDInsight (Hadoop)](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-linux-use-ssh-unix) 和[使用 SSH 通道來存取 Ambari Web UI、JobHistory、NameNode、Oozie 及其他 Web UI](https://docs.microsoft.com/azure/hdinsight/hdinsight-linux-ambari-ssh-tunnel)。 
 
## <a name="learn-how-to-perform-local-run-and-debugging"></a>了解如何執行本機執行和偵錯
### <a name="scenario-1-create-a-spark-scala-application"></a>案例 1：建立 Spark Scala 應用程式 

1. 啟動 IntelliJ IDEA，然後建立專案。 在 [新增專案]  對話方塊中，執行下列操作：

   a. 選取 [HDInsight]。 

   b. 根據您的喜好設定選取 Java 或 Scala 範本。 在下列選項之間進行選取：

      - **Spark on HDInsight (Scala) \(HDInsight 上的 Spark (Scala)\)**

      - **Spark on HDInsight (Java) \(HDInsight 上的 Spark (Java)\)**

      - **HDInsight Sample (Scala) 上的 Spark**

      此範例會使用 [HDInsight 範本上的 Spark (Scala)] 範本。

   c. 在 [建置工具] 清單中，根據您的需求選取下列任一項目：

      - **Maven**，以支援 Scala 專案建立精靈

      -  **SBT**，以管理相依性並建置 Scala 專案 

      ![建立偵錯專案](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-create-projectfor-debug-remotely.png)

   d. 選取 [下一步] 。     
 
1. 在下一個 [新增專案] 視窗中，執行下列動作：

   ![選取 Spark SDK](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-new-project.png)

   a. 輸入專案名稱和專案位置。

   b. 在 [專案 SDK] 下拉式清單中，選取 [Java 1.8] \(適用於 **Spark 2.x** 叢集)，或選取 [Java 1.7] \(適用於 **Spark 1.x** 叢集)。

   c. 在 [Spark 版本] 下拉式清單中，Scala 專案建立精靈會為 Spark SDK 和 Scala SDK 整合正確的版本。 如果 Spark 叢集是 2.0 以前的版本，請選取 [Spark 1.x]。 否則，請選取 **Spark 2.x**。 此範例使用 **Spark 2.0.2 (Scala 2.11.8)**。

   d. 選取 [完成]。

1. 選取 **src** > **main \(主要\)** > scala 以在專案中開啟您的程式碼。 此範例使用 **SparkCore_wasbloTest** 指令碼。

### <a name="prerequisite-for-windows"></a>Windows 的必要條件
在 Windows 電腦上執行本機 Spark Scala 應用程式時，可能會發生如 [SPARK-2356](https://issues.apache.org/jira/browse/SPARK-2356) 中所述的例外狀況。 發生這個例外狀況是因為 Windows 上遺失 WinUtils.exe。 

若要解決這個錯誤，請[下載可執行檔](http://public-repo-1.hortonworks.com/hdp-win-alpha/winutils.exe)，並將其放在 **C:\WinUtils\bin** 之類的位置。 然後，新增環境變數 **HADOOP_HOME**，並將變數的值設為 **C:\WinUtils**。

### <a name="scenario-2-perform-local-run"></a>案例 2：執行本機執行
1. 開啟 **SparkCore_wasbloTest** 指令碼，以滑鼠右鍵按一下指令碼編輯器，然後選取 [執行 '[Spark 作業]XXX'] 選項以執行本機執行。
1. 一旦本機執行完成，您會看到輸出檔案儲存至目前的專案總管**資料** > **__預設__**。

    ![本機執行結果](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/local-run-result.png)
1. 我們的工具已在您執行本機執行和本機偵錯時自動設定預設本機執行設定。 在右上角開啟 [[Spark 作業] XXX]，您可以看到 [[Spark 作業]XXX] 已在 [Azure HDInsight Spark 作業] 底下建立。 切換至 [本機執行] 索引標籤。

    ![本機執行設定](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/local-run-configuration.png)
    - [環境變數](#prerequisite-for-windows)：如果您已將系統環境變數 **HADOOP_HOME** 設定為 **C:\WinUtils**，它可以自動偵測，不需要手動新增。
    - [WinUtils.exe 位置](#prerequisite-for-windows)：如果您尚未設定系統環境變數，您可以按一下此按鈕以找到位置。
    - 只要選擇兩個選項之一，在 MacOS 和 Linux 上不需要它們。
1. 您也可以在執行本機執行和本機偵錯之前手動設定。 在上述螢幕擷取畫面中，選取加號 (**+**)。 然後選取 [Azure HDInsight Spark 作業] 選項。 輸入 [名稱] 的資訊、要儲存的**主要類別名稱**，然後按一下本機執行按鈕。

### <a name="scenario-3-perform-local-debugging"></a>案例 3：執行本機偵錯
1. 開啟 **SparkCore_wasbloTest** 指令碼，設定中斷點。
1. 以滑鼠右鍵按一下指令碼編輯器，然後選取 [偵錯 '[Spark 作業]XXX'] 選項以執行本機偵錯。   



## <a name="learn-how-to-perform-remote-run-and-debugging"></a>了解如何執行遠端執行和偵錯
### <a name="scenario-1-perform-remote-run"></a>案例 1：執行遠端執行

1. 若要存取 [編輯設定] 功能表，請選取右上角的圖示。 從這個功能表中，您可以建立或編輯遠端偵錯的設定。

   ![編輯設定](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-edit-configurations.png) 

1. 在 [Run/Debug Configurations] \(執行/偵錯設定) 對話方塊中，選取加號 (**+**)。 然後選取 [Azure HDInsight Spark 作業] 選項。

   ![新增設定](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-add-new-Configuration.png)
1. 切換至 [在叢集中遠端執行] 索引標籤。輸入 [Name] \(名稱\)、**[Spark cluster] \(Spark 叢集\)** 和 [Main class name] \(主要類別名稱\)。 然後選取 [Advanced configuration] \(進階設定\)。 我們的工具支援使用**執行程式**進行偵錯。 **NumExectors**，預設值為 5。 您最好不要設定高於 3。

   ![[Run/Debug Configurations] \(執行/偵錯設定\)](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-run-debug-configurations.png)

1. 在 [Spark Submission Advanced Configuration] \(Spark 提交進階設定\) 對話方塊中，選取 [Enable Spark remote debug] \(啟用 Spark 遠端偵錯\)。 輸入 SSH 使用者名稱，然後輸入密碼或使用私密金鑰檔案。 若要儲存設定，請選取 [確定]。 如果您想要執行遠端偵錯，您需要設定它。 如果您只想要使用遠端執行，則不需要設定它。

   ![[Enable Spark remote debug] \(啟用 Spark 遠端偵錯\)](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-enable-spark-remote-debug.png)

1. 設定現在會使用您提供的名稱儲存。 若要檢視設定詳細資訊，請選取設定名稱。 若要進行變更，請選取 [Edit Configurations] \(編輯設定\)。 

1. 完成組態設定之後，您可以針對遠端叢集執行專案，或執行遠端偵錯。
   
   ![遠端執行按鈕](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/perform-remote-run.png)

1. 按一下送出記錄檔未出現在左面板中的 [中斷連線] 按鈕。 不過，它仍在後端執行。

   ![遠端執行按鈕](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/remote-run-result.png)



### <a name="scenario-2-perform-remote-debugging"></a>案例 2：執行遠端偵錯
1. 設定一個中斷點，然後按一下 [遠端偵錯] 圖示。 與遠端提交的差異是，必須設定 SSH 使用者名稱/密碼。

   ![選取偵錯圖示](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-icon.png)

1. 當程式執行觸及中斷點時，您會在 [偵錯工具] 窗格中看到一個 [驅動程式] 索引標籤和兩個 [執行程式] 索引標籤。 選取**繼續程式**圖示以繼續執行程式碼，然後到達下一個中斷點。 您必須切換到正確的 [執行程式] 索引標籤，才能尋找要偵錯的目標執行程式。 您可以在對應的 [主控台] 索引標籤中檢視執行記錄。

   ![[Debug] \(偵錯\) 索引標籤](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debugger-tab.png)

### <a name="scenario-3-perform-remote-debugging-and-bug-fixing"></a>案例 3：執行遠端偵錯和錯誤修正
1. 設定兩個中斷點，然後選取**偵錯**圖示以啟動遠端偵錯程序。

1. 程式碼會在第一個中斷點停止，然後在 [變數] 窗格中顯示參數和變數資訊。 

1. 選取**繼續程式**圖示以繼續。 程式碼會在第二個點停止。 正如預期，會攔截到例外狀況。

   ![擲回錯誤](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-throw-error.png) 

1. 再次選取**繼續程式**圖示。 [HDInsight Spark Submission] \(HDInsight Spark 提交) 視窗會顯示「作業執行失敗」錯誤。

   ![提交錯誤](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-error-submission.png) 

1. 若要使用 IntelliJ 偵錯功能動態更新變數值，請再次選取 [偵錯]。 [變數] 窗格會再次出現。 

1. 以滑鼠右鍵按一下 [Debug] \(偵錯\) 索引標籤上的目標，然後選取 [Set Value] \(設定值\)。 接下來，輸入變數的新值。 然後選取 **Enter** 儲存值。 

   ![設定值](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-set-value.png) 

1. 選取**繼續程式**圖示以繼續執行程式。 此時，不會攔截到任何例外狀況。 您可以看到專案成功執行而未發生任何例外狀況。

   ![偵錯而未發生例外狀況](./media/apache-spark-intellij-tool-debug-remotely-through-ssh/hdinsight-debug-without-exception.png)

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
* [使用適用於 IntelliJ 的 Azure 工具組透過 VPN 對 Spark 應用程式進行遠端偵錯](apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [透過 Hortonworks 沙箱使用 HDInsight Tools for IntelliJ](../hadoop/hdinsight-tools-for-intellij-with-hortonworks-sandbox.md)
* [使用適用於 Eclipse 的 Azure 工具組中的 HDInsight 工具建立 Spark 應用程式](../hdinsight-apache-spark-eclipse-tool-plugin.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](apache-spark-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](apache-spark-job-debugging.md)
