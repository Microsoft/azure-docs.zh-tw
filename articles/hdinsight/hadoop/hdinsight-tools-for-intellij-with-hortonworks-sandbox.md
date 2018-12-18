---
title: 透過 Hortonworks 沙箱使用 Azure Toolkit for IntelliJ
description: 了解如何透過 Hortonworks 沙箱使用 Azure Toolkit for IntelliJ 中的 HDInsight 工具。
keywords: hadoop 工具,hive 查詢,intellij,hortonworks 沙箱,azure toolkit for intellij
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/16/2018
ms.author: jasonh
ms.openlocfilehash: 9e87392ad7730571b973dbec809f64487eefa849
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048586"
---
# <a name="use-hdinsight-tools-for-intellij-with-hortonworks-sandbox"></a>透過 Hortonworks 沙箱使用 HDInsight Tools for IntelliJ

瞭解如何使用 HDInsight Tools for IntelliJ 開發 Apache Scala 應用程式，然後在電腦上執行的 [Hortonworks 沙箱](http://hortonworks.com/products/sandbox/)上測試應用程式。 

[IntelliJ IDEA](https://www.jetbrains.com/idea/) 是 Java 整合式開發環境 (IDE)，可用來開發電腦軟體。 在 Hortonworks 沙箱上開發並測試應用程式之後，您可以將應用程式移至 [Azure HDInsight](apache-hadoop-introduction.md)。

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前，您必須具備下列項目：

- 在您的本機電腦執行的 Hortonworks 沙箱上要有 Hortonworks Data Platform (HDP) 2.4。 若要設定 HDP，請參閱[透過虛擬機器的 Hadoop 沙箱開始使用 Hadoop 生態系統](apache-hadoop-emulator-get-started.md)。 
    > [!NOTE]
    > HDInsight Tools for IntelliJ 只經過 HDP 2.4 測試。 若要取得 HDP 2.4，請在 **Hortonworks 沙箱下載網站**展開 [Hortonworks 沙箱封存](http://hortonworks.com/downloads/#sandbox)。

- [Java Developer Kit (JDK) 1.8 版或更新版本](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)。 Azure Toolkit for IntelliJ 需有 JDT。

- [IntelliJ IDEA community 版本](https://www.jetbrains.com/idea/download)搭配使用 [Scala](https://plugins.jetbrains.com/idea/plugin/1347-scala) 外掛程式與 [Azure Toolkit for IntelliJ](https://docs.microsoft.com/java/azure/intellij/azure-toolkit-for-intellij) 外掛程式。 HDInsight Tools for IntelliJ 是 Azure Toolkit for IntelliJ 的一部分。 

若要安裝外掛程式：

  1. 開啟 IntelliJ IDEA。
  2. 在 [歡迎使用] 頁面上，選取 [設定]，然後按一下 [外掛程式]。
  3. 選取左下角的 [安裝 JetBrains 外掛程式]。
  4. 使用搜尋功能來搜尋 **Scala**，然後選取 [安裝]。
  5. 選取 [重新啟動 IntelliJ IDEA] 以完成安裝。
  6. 重複步驟 4 和 5 以安裝 [Azure Toolkit for IntelliJ]。 如需詳細資訊，請參閱[安裝適用於 IntelliJ 的 Azure 工具組](https://docs.microsoft.com/azure/azure-toolkit-for-intellij-installation)。

## <a name="create-a-spark-scala-application"></a>建立 Spark Scala 應用程式

在本節中，您會使用 IntelliJ IDEA 建立範例 Scala 專案。 在下節中，您要在提交專案之前，將 IntelliJ IDEA 連結至 Hortonworks 沙箱 (模擬器)。

1. 在電腦上開啟 IntelliJ IDEA。 在 [新增專案] 對話方塊中，完成下列步驟：

   1. 選取 [HDInsight] > [HDInsight 上的 Spark (Scala)]。
   2. 在 [建置工具] 清單中，根據您的狀況選取下列項目：

    * **Maven**：建立 Scala 專案的精靈支援。
    * **SBT**：管理 Scala 的相依性和建置。

   ![[新增專案] 對話方塊](./media/hdinsight-tools-for-intellij-with-hortonworks-sandbox/intellij-create-scala-project.png)

2. 選取 [下一步] 。
3. 在下一個 [新增專案] 對話方塊中，完成下列步驟：

    1. 在 [專案名稱] 方塊中，輸入專案名稱。
    2. 在 [專案位置] 方塊中，輸入專案位置。
    3. 在 [專案 SDK] 下拉式清單旁邊，選取 [新增]，並選取 [JDK]，然後指定 Java JDK 1.7 版或更新版本的資料夾。 如果是 Spark 2.x 叢集，選取 **Java 1.8**。 如果是 Spark 1.x 叢集，選取 **Java 1.7**。 預設位置是 C:\Program Files\Java\jdk1.8.x_xxx。
    4. 在 [Spark 版本] 下拉式清單中，Scala 專案建立精靈會為 Spark SDK 和 Scala SDK 整合正確的版本。 如果 Spark 叢集是 2.0 以前的版本，請選取 [Spark 1.x]。 否則，請選取 [Spark2.x]。 此範例使用 Spark 1.6.2 (Scala 2.10.5)。 確定您使用標示為 **Scala 2.10.x** 的儲存機制。 請勿使用標示為 Scala 2.11.x 的儲存機制。
    
    ![建立 IntelliJ Scala 專案屬性](./media/hdinsight-tools-for-intellij-with-hortonworks-sandbox/intellij-create-scala-project-properties.png)


4. 選取 [完成]。
5. 如果**專案**檢視尚未開啟，請按下 **Alt + 1** 加以開啟。
6. 在 [專案總管] 中，展開專案，然後選取 [src]。
7. 以滑鼠右鍵按一下 [src]，指向 [新增]，然後選取 [Scala 類別]。
8. 在 [名稱] 方塊中，輸入名稱。 在 [種類] 方塊中，選取 [物件]。 然後選取 [確定]。

    ![建立新 Scala 類別的對話方塊](./media/hdinsight-tools-for-intellij-with-hortonworks-sandbox/intellij-create-new-scala-class.png)

9. 在 .scala 檔案中，貼上下列程式碼：

        import java.util.Random
        import org.apache.spark.{SparkConf, SparkContext}
        import org.apache.spark.SparkContext._

        /**
        * Usage: GroupByTest [numMappers] [numKVPairs] [valSize] [numReducers]
        */
        object GroupByTest {
            def main(args: Array[String]) {
                val sparkConf = new SparkConf().setAppName("GroupBy Test")
                var numMappers = 3
                var numKVPairs = 10
                var valSize = 10
                var numReducers = 2

                val sc = new SparkContext(sparkConf)

                val pairs1 = sc.parallelize(0 until numMappers, numMappers).flatMap { p =>
                val ranGen = new Random
                var arr1 = new Array[(Int, Array[Byte])](numKVPairs)
                for (i <- 0 until numKVPairs) {
                    val byteArr = new Array[Byte](valSize)
                    ranGen.nextBytes(byteArr)
                    arr1(i) = (ranGen.nextInt(Int.MaxValue), byteArr)
                }
                arr1
                }.cache
                // Enforce that everything has been calculated and in cache.
                pairs1.count

                println(pairs1.groupByKey(numReducers).count)
            }
        }

10. 在 [建置] 功能表上，選取 [建置專案]。 確定編譯已成功完成。


## <a name="link-to-the-hortonworks-sandbox"></a>連結至 HortonWorks 沙箱

您連結至 Hortonworks 沙箱 (模擬器) 前，必須有現有的 IntelliJ 應用程式。

若要連結至模擬器：

1. 在 IntelliJ 中開啟專案。
2. 在 [檢視] 功能表上，選取 [工具視窗]，然後選取 [Azure Explorer]。
3. 展開 [Azure]，以滑鼠右鍵按一下 [HDInsight]，然後選取 [連結模擬器]。
4. 在 [連結新模擬器] 對話方塊中，輸入您為 Hortonworks 沙箱根帳戶設定的密碼。 接下來，輸入類似於下方螢幕擷取畫面的值。 然後選取 [確定]。 

   ![連結新模擬器的對話方塊](./media/hdinsight-tools-for-intellij-with-hortonworks-sandbox/intellij-link-an-emulator.png)

5. 若要設定模擬器，請選取[是]。

成功連接模擬器後，[HDInsight] 節點會列出該模擬器 (Hortonworks 沙箱)。

## <a name="submit-the-spark-scala-application-to-the-hortonworks-sandbox"></a>將 Spark Scala 應用程式提交至 Hortonworks 沙箱

在將 IntelliJ IDEA 連結至模擬器之後，您就可以提交專案。

若要將專案提交至模擬器：

1. 在 [專案總管] 中，以滑鼠右鍵按一下專案，然後選取 [將 Spark 應用程式提交給 HDInsight]。
2. 完成下列步驟：

    1. 在 [Spark 叢集 (僅限 Linux)] 下拉式清單中，選取您的本機 Hortonworks 沙箱。
    2. 在 [主要類別名稱] 方塊中，選取或輸入主要類別名稱。 在本教學課程中，名稱是 **GroupByTest**。

3. 選取 [提交]。 Spark 提交工具視窗會顯示工作提交記錄檔。

## <a name="next-steps"></a>後續步驟

- 了解如何[使用 Azure Toolkit for IntelliJ 中的 HDInsight 工具建立 HDInsight Spark Linux 叢集的 Spark 應用程式](../spark/apache-spark-intellij-tool-plugin.md)。

- 如需有關 HDInsight Tools for IntelliJ 的影片，請參閱[介紹用於開發 Spark 的 HDInsight Tools for IntelliJ](https://www.youtube.com/watch?v=YTZzYVgut6c)。

- 了解如何[使用 Azure Toolkit for IntelliJ 透過 SSH 對 HDInsight 叢集上的 Spark 應用程式進行遠端偵錯](../spark/apache-spark-intellij-tool-debug-remotely-through-ssh.md)。

- 了解如何[使用 Azure Toolkit for IntelliJ 中的 HDInsight 工具遠端偵錯 HDInsight Spark Linux 叢集上的 Spark 應用程式](../spark/apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)。

- 了解如何[使用適用於 Azure Toolkit for Eclipse 中的 HDInsight 工具建立 Spark 應用程式](../spark/apache-spark-eclipse-tool-plugin.md)。

- 如需有關 HDInsight Tools for Eclipse 的影片，請參閱[使用 HDInsight Tool for Eclipse 建立 Spark 應用程式](https://mix.office.com/watch/1rau2mopb6fha)。
