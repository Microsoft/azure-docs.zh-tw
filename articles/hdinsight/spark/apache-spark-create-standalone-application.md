---
title: 教學課程：使用 IntelliJ 為 Azure HDInsight 上的 Spark 建立 Scala Maven 應用程式
description: 建立以 Scala 撰寫的 Spark 應用程式，搭配 Apache Maven 作為建置系統，以及由 IntelliJ IDEA 提供之適用於 Scala 的現有 Maven 原型。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,mvc
ms.topic: tutorial
ms.date: 05/07/2018
ms.openlocfilehash: fc1f952128b4cfbb082f4c539a102f40d3b85e8d
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43040293"
---
# <a name="tutorial-create-a-scala-maven-application-for-spark-in-hdinsight-using-intellij"></a>教學課程：使用 IntelliJ 為 HDInsight 上的 Spark 建立 Scala Maven 應用程式

在本教學課程中，您將了解如何使用 Maven 與 IntelliJ IDEA 建立以 Scala 撰寫的 Spark 應用程式。 本文以 Apache Maven 作為建置系統，並且以 IntelliJ IDEA 為 Scala 提供的現有 Maven 原型作為起始點。  要在 IntelliJ IDEA 中建立 Scala 應用程式，必須執行下列步驟：

* 以 Maven 做為建置系統。
* 更新專案物件模型 (POM) 檔案，以解析 Spark 模組相依性。
* 以 Scala 撰寫應用程式。
* 產生可提交至 HDInsight Spark 叢集的 jar 檔案。
* 使用 Livy 在 Spark 叢集上執行應用程式。

> [!NOTE]
> HDInsight 也提供 IntelliJ IDEA 外掛程式工具，可簡化建立和提交應用程式至 Linux 上之 HDInsight Spark 叢集的程序。 如需詳細資訊，請參閱 [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark 應用程式](apache-spark-intellij-tool-plugin.md)。
> 

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 IntelliJ 開發 Scala Maven 應用程式

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。


## <a name="prerequisites"></a>必要條件

* HDInsight 上的 Apache Spark 叢集。 如需指示，請參閱[在 Azure HDInsight 中建立 Apache Spark 叢集](apache-spark-jupyter-spark-sql.md)。
* Oracle Java Development Kit。 您可以從[這裡](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)安裝它。
* Java IDE。 本文使用 IntelliJ IDEA 18.1.1。 您可以從[這裡](https://www.jetbrains.com/idea/download/)安裝它。

## <a name="use-intellij-to-create-application"></a>使用 IntelliJ 建立應用程式

1. 啟動 IntelliJ IDEA，然後建立專案。 在 [新增專案]  對話方塊中，執行下列操作： 

   a. 選取 [HDInsight] > [HDInsight 上的 Spark (Scala)]。

   b. 在 [建置工具] 清單中，根據您的需求選取下列任一項目：

      * **Maven**，以支援 Scala 專案建立精靈
      * **SBT**，以管理相依性並建置 Scala 專案

   ![[新增專案] 對話方塊](./media/apache-spark-create-standalone-application/create-hdi-scala-app.png)

1. 選取 [下一步] 。

1. Scala 專案建立精靈會自動偵測您是否已安裝 Scala 外掛程式。 選取 [安裝]。

   ![Scala 外掛程式檢查](./media/apache-spark-create-standalone-application/Scala-Plugin-check-Reminder.PNG) 

1. 若要下載 Scala 外掛程式，請選取 [確定]。 依指示重新啟動 IntelliJ。 

   ![Scala 外掛程式安裝對話方塊](./media/apache-spark-create-standalone-application/Choose-Scala-Plugin.PNG)

1. 在 [新增專案] 視窗中，執行下列動作：  

    ![選取 Spark SDK](./media/apache-spark-create-standalone-application/hdi-new-project.png)

   a. 輸入專案名稱和位置。

   b. 在 [專案 SDK] 下拉式清單中，選取 [Java 1.8] \(適用於 Spark 2.x 叢集)，或選取 [Java 1.7] \(適用於 Spark 1.x 叢集)。

   c. 在 [Spark 版本] 下拉式清單中，Scala 專案建立精靈會為 Spark SDK 和 Scala SDK 整合正確的版本。 如果 Spark 叢集是 2.0 以前的版本，請選取 [Spark 1.x]。 否則，請選取 [Spark2.x]。 此範例使用 **Spark 2.0.2 (Scala 2.11.8)**。

1. 選取 [完成]。

## <a name="install-scala-plugin-for-intellij-idea"></a>安裝 IntelliJ IDEA 的 Scala 外掛程式
若要安裝 Scala 外掛程式，請使用下列步驟：

1. 開啟 IntelliJ IDEA。
1. 在 [歡迎使用] 畫面上選取 [設定]，然後選取 [外掛程式]。
   
    ![啟用 Scala 外掛程式](./media/apache-spark-create-standalone-application/enable-scala-plugin.png)
1. 選取左下角的 [安裝 JetBrains 外掛程式]。 
1. 在 [瀏覽 JetBrains 外掛程式] 對話方塊中搜尋 **Scala**，然後選取 [安裝]。
   
    ![安裝 Scala 外掛程式](./media/apache-spark-create-standalone-application/install-scala-plugin.png)
1. 在外掛程式安裝成功後，您必須重新啟動 IDE。

## <a name="create-a-standalone-scala-project"></a>建立獨立 Scala 專案
1. 開啟 IntelliJ IDEA。
1. 從 [檔案] 功能表中選取 [新增] > [專案]，以建立新的專案。
1. 在 [新增專案] 對話方塊中，選取下列選項：
   
    ![建立 Maven 專案](./media/apache-spark-create-standalone-application/create-maven-project.png)
   
   * 選取 [Maven]  做為專案類型。
   * 指定 [專案 SDK] 。 選取 [新增]，然後瀏覽至 Java 安裝目錄 (通常是 `C:\Program Files\Java\jdk1.8.0_66`)。
   * 選取 [從原型建立]  選項。
   * 從原型清單中，選取 **org.scala-tools.archetypes:scala-archetype-simple**。 此 archetype 會建立正確的目錄結構，並下載撰寫 Scala 程式所需的預設相依性。
1. 選取 [下一步] 。
1. 為 [GroupId]、[ArtifactId] 和 [版本] 提供相關值。 本教學課程中使用下列值：

    - GroupId：com.microsoft.spark.example
    - ArtifactId：SparkSimpleApp
1. 選取 [下一步] 。
1. 確認設定，然後選取 [下一步]。
1. 確認專案名稱和位置，然後按一下 [完成]。
1. 在左窗格中選取 **src > 測試 > scala > com > microsoft > spark > 範例**，以滑鼠右鍵按一下 **MySpec**，然後選取 [刪除]。 應用程式並不需要此檔案。
  
1. 在後續步驟中，您會更新 pom.xml，以定義 Spark Scala 應用程式的相依性。 若要自動下載並解析這些相依性，您必須據以設定 Maven。
   
    ![設定 Maven 以進行自動下載](./media/apache-spark-create-standalone-application/configure-maven.png)
   
   1. 在 [檔案] 功能表中，選取 [設定]。
   1. 在 [設定] 對話方塊中，導覽至 [建置、執行、部署] > [建置工具] > [Maven] > [匯入]。
   1. 選取 [自動匯入 Maven 專案] 的選項。
   1. 選取 [套用]，然後選取 [確定]。
1. 在左窗格中選取 **src > 主要 > scala > com.microsoft.spark.example**，然後按兩下 [應用程式] 以開啟 App.scala。

1. 將現有的範例程式碼取代為下列程式碼，然後儲存變更。 此程式碼會從 HVAC.csv (所有 HDInsight Spark 叢集上均有提供) 讀取資料、擷取在第六個資料行中只有個位數的資料列，並將輸出寫入到叢集預設儲存體容器下的 **/HVACOut** 。

        package com.microsoft.spark.example
   
        import org.apache.spark.SparkConf
        import org.apache.spark.SparkContext
   
        /**
          * Test IO to wasb
          */
        object WasbIOTest {
          def main (arg: Array[String]): Unit = {
            val conf = new SparkConf().setAppName("WASBIOTest")
            val sc = new SparkContext(conf)
   
            val rdd = sc.textFile("wasb:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")
   
            //find the rows which have only one digit in the 7th column in the CSV
            val rdd1 = rdd.filter(s => s.split(",")(6).length() == 1)
   
            rdd1.saveAsTextFile("wasb:///HVACout")
          }
        }
1. 在左窗格中按兩下 **pom.xml**。
   
   1. 在 `<project>\<properties>` 內新增下列區段：
      
          <scala.version>2.10.4</scala.version>
          <scala.compat.version>2.10.4</scala.compat.version>
          <scala.binary.version>2.10</scala.binary.version>
   1. 在 `<project>\<dependencies>` 內新增下列區段：
      
           <dependency>
             <groupId>org.apache.spark</groupId>
             <artifactId>spark-core_${scala.binary.version}</artifactId>
             <version>1.4.1</version>
           </dependency>
      
      儲存 pom.xml 的變更。
1. 建立 .jar 檔案。 IntelliJ IDEA 允許將 JAR 建立為專案的構件。 請執行下列步驟：
    
    1. 從 [檔案] 功能表，選取 [專案結構]。
    1. 在 [專案結構] 對話方塊中選取 [構件]，然後選取加號。 在快顯對話方塊中選取 [JAR]，然後選取 [從具有相依性的模組]。
       
        ![建立 JAR](./media/apache-spark-create-standalone-application/create-jar-1.png)
    1. 在 [從模組建立 JAR] 對話方塊中，對 [主要類別] 選取省略符號 (![ellipsis](./media/apache-spark-create-standalone-application/ellipsis.png))。
    1. 在 [選取主要類別] 對話方塊中，選取依預設出現的類別，然後選取 [確定]。
       
        ![建立 JAR](./media/apache-spark-create-standalone-application/create-jar-2.png)
    1. 在 [從模組建立 JAR] 對話方塊中，確定已選取 [擷取至目標 JAR] 選項，然後選取 [確定]。  此設定會建立具有所有相依性的單一 JAR。
       
        ![建立 JAR](./media/apache-spark-create-standalone-application/create-jar-3.png)
    1. [輸出配置] 索引標籤會列出所有納入 Maven 專案中的 jar。 您可以選取並刪除 Scala 應用程式未直接依存的 jar。 對於您在此處建立的應用程式，您可以移除最後一個 (**SparkSimpleApp 編譯輸出**) 以外的所有 jar。 選取要刪除的 jar，然後選取 [刪除] 圖示。
       
        ![建立 JAR](./media/apache-spark-create-standalone-application/delete-output-jars.png)
       
        請確實選取 [包含在專案建置中] 方塊，以確保在每次建置或更新專案時都會建立 jar。 依序選取 [套用] 和 [確定]。
    1. 從 [建置] 功能表中選取 [建置構件]，以建立 jar。 輸出 jar 會建立在 **\out\artifacts** 下。
       
        ![建立 JAR](./media/apache-spark-create-standalone-application/output.png)

## <a name="run-the-application-on-the-spark-cluster"></a>在 Spark 叢集上執行應用程式
若要在叢集上執行應用程式，您可以使用下列方法：

* **將應用程式 jar 複製到與叢集相關聯的 Azure 儲存體 Blob** 。 您可以使用命令列公用程式 [**AzCopy**](../../storage/common/storage-use-azcopy.md) 來執行此動作。 另外也有很多用戶端可用來上傳資料。 您可以在 [在 HDInsight 上將 Hadoop 作業的資料上傳](../hdinsight-upload-data.md)中找到其詳細資訊。
* **使用 Livy 從遠端提交應用程式作業至** Spark 叢集。 HDInsight 上的 Spark 叢集包含會公開 REST 端點以從遠端提交 Spark 作業的 Livy。 如需詳細資訊，請參閱 [搭配 HDInsight 上的 Spark 叢集利用 Livy 遠端提交 Spark 作業](apache-spark-livy-rest-interface.md)。

## <a name="next-step"></a>後續步驟

在本文中，您已了解如何建立 Spark Scala 應用程式。 前往下篇文章，了解如何使用 Livy 在 HDInsight Spark 叢集上執行此應用程式。

> [!div class="nextstepaction"]
>[利用 Livy 在 Spark 叢集上遠端執行作業](./apache-spark-livy-rest-interface.md)

