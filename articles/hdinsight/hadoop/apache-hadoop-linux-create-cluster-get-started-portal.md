---
title: '快速入門：使用 Azure 入口網站在 Azure HDInsight 中開始使用 Hadoop 和 Hive '
description: 了解如何使用 Azure 入口網站來建立 HDInsight 叢集，以及使用 Hive 來查詢資料。
keywords: 開始使用,hadoop linux,hadoop 快速入門,開始使用 hive,hive 快速入門
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017,mvc
ms.topic: quickstart
ms.date: 05/07/2018
ms.author: jasonh
ms.openlocfilehash: 802c35f94457cc60ec27b3dd061d763d1acab4f5
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042231"
---
# <a name="quickstart-get-started-with-hadoop-and-hive-in-azure-hdinsight-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站在 Azure HDInsight 中開始使用 Hadoop 和 Hive

在本文中，您會了解如何使用 Azure 入口網站在 HDInsight 中建立 [Hadoop](http://hadoop.apache.org/) \(英文\) 叢集，然後在 HDInsight 中執行 Hive 作業。 大部分 Hadoop 作業都是批次作業。 您會建立叢集、執行一些工作，然後刪除叢集。 在本文中，您會執行所有這三個工作。

在本快速入門中，您會使用 Azure 入口網站來建立 HDInsight Hadoop 叢集。 您也可以使用 [Azure Resource Manager 範本](apache-hadoop-linux-tutorial-get-started.md)來建立叢集。

HDInsight 目前隨附 [7 個不同的叢集類型](./apache-hadoop-introduction.md#cluster-types-in-hdinsight)。 每種叢集類型都支援一組不同的元件。 所有叢集類型都支援 Hive。 如需 HDInsight 中支援的元件清單，請參閱 [HDInsight 在 Hadoop 叢集版本中提供的新功能](../hdinsight-component-versioning.md)  

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="create-a-hadoop-cluster"></a>建立 Hadoop 叢集

在本節中，您會使用 Azure 入口網站，在 HDInsight 中建立 Hadoop 叢集。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 在 Azure 入口網站中，選取 [建立資源] > [資料 + 分析] > [HDInsight]。 

    ![Azure 入口網站上的 Databricks](./media/apache-hadoop-linux-create-cluster-get-started-portal/azure-hdinsight-on-portal.png "Azure 入口網站上的 Databricks")

2. 在 [HDInsight] > [快速建立] > [基本] 下方，提供如下列螢幕擷取畫面所建議的值：

    ![HDInsight Linux 開始提供叢集基本值](./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-portal-provide-basic-values.png "提供用於建立 HDInsight 叢集的基本值")

    輸入或選取下列值：
    
    |屬性  |說明  |
    |---------|---------|
    |**叢集名稱**     | 輸入 Hadoop 叢集的名稱。 由於 HDInsight 中的所有叢集共用相同的 DNS 命名空間，因此這個名稱必須是唯一的。 名稱最多可包含 59 個字元，而這些字元可以是字母、數字和連字號。 名稱的第一個和最後一個字元不可以是連字號。 |
    |**訂用帳戶**     |  選取 Azure 訂用帳戶。 |
    |**叢集類型**     | 暫時略過此欄位。 您會在此程序的下一個步驟中提供此輸入。|
    |**叢集登入使用者名稱和密碼**     | 預設登入名稱為 **admin**。密碼長度至少必須為 10 個字元，且必須包含至少一個數字、一個大寫字母及一個小寫字母、一個非英數字元 (除了字元 ' " ` \)。 確定您**不會提供**常見密碼，例如 "Pass@word1"。|
    |**SSH 使用者名稱** | 預設的使用者名稱為 **sshuser**。  您可以為 SSH 使用者名稱提供另一個名稱。 |
    | **使用與叢集登入相同的密碼** | 選取此核取方塊，讓 SSH 使用者所使用的密碼相同於您提供給叢集登入使用者的密碼。|
    |**資源群組**     | 建立資源群組，或選取現有的資源群組。  資源群組是 Azure 元件的容器。  在此案例中，資源群組包含 HDInsight 叢集和相依的 Azure 儲存體帳戶。 |
    |**位置**     | 選取您要建立叢集的 Azure 位置。  選擇靠近您的位置，以獲得最佳效能。 |
        
3. 選取 [叢集類型]，然後提供輸入值，如下列螢幕擷取畫面所示：

    ![HDInsight Linux 開始提供叢集基本值](./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-portal-provide-cluster-type.png "提供用於建立 HDInsight 叢集的基本值")

    輸入下列值：
    
    |屬性  |說明  |
    |---------|---------|
    |**叢集類型**     | 選取 [Hadoop] |
    |**作業系統**     |  選取 Azure 訂用帳戶。 |
    |**版本**     | 選取 [Hadoop 2.7.3 (HDI 3.6)]。|

    按一下 [選取]，然後按一下 [下一步]。

4. 在 [儲存體] 索引標籤中，提供輸入值，如下列螢幕擷取畫面所示：

    ![HDInsight Linux 開始提供叢集存放區值](./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-portal-select-storage.png "提供用於建立 HDInsight 叢集的存放區值")

    輸入下列值：
    
    |屬性  |說明  |
    |---------|---------|
    |**主要儲存體類型**     | 針對本文，請選取 [Azure 儲存體]，以使用 Azure 儲存體 Blob 作為預設儲存體帳戶。 您也可以使用 Azure Data Lake Storage 作為預設儲存體。 |
    |**選取方法**     |  針對本文，請選取 [我的訂用帳戶]，以使用您 Azure 訂用帳戶的儲存體帳戶。 若要使用其他訂用帳戶的儲存體帳戶，請選取 [存取金鑰]，然後提供該帳戶的存取金鑰。 |
    |**建立新的儲存體帳戶**     | 提供儲存體帳戶的名稱。|

    接受所有其他預設值，然後選取 [下一步]。

5. 在 [摘要] 索引標籤上，確認您在先前步驟中所選取的值。

    ![HDInsight Linux 開始使用叢集摘要](./media/apache-hadoop-linux-create-cluster-get-started-portal/hdinsight-linux-get-started-portal-summary.png "HDInsight Linux 開始使用叢集摘要")
      
4. 選取 [建立] 。 您應該會在入口網站儀表板上看到標題為**正在提交適用於 HDInsight 的部署**的新圖格。 大約需要 20 分鐘的時間來建立叢集。

    ![HDInsight Linux 開始使用的資源群組](./media/apache-hadoop-linux-create-cluster-get-started-portal/deployment-progress-tile.png "Azure HDInsight 叢集資源群組")

4. 一旦建立叢集之後，您就會在 Azure 入口網站中看到叢集概觀頁面。
   
    ![HDInsight Linux 開始使用的叢集設定](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-cluster-settings.png "Azure HDInsight 屬性")    
    
    每個叢集都具備 [Azure 儲存體帳戶](../hdinsight-hadoop-use-blob-storage.md)或 [Azure Data Lake 帳戶](../hdinsight-hadoop-use-data-lake-store.md)相依性。 也稱為預設儲存體帳戶。 HDInsight 叢集與其預設儲存體帳戶必須並存於相同的 Azure 區域。 刪除叢集並不會刪除儲存體帳戶。

    > [!NOTE]
    > 如需其他叢集建立方法及了解本教學課程中使用的屬性，請參閱 [建立 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。       
    > 
    >

## <a name="run-hive-queries"></a>執行 Hive 查詢

[Apache Hive](hdinsight-use-hive.md) 是 HDInsight 中使用的最受歡迎元件。 有許多方法可在 HDInsight 上執行 Hive 工作。 在本教學課程中，您將從入口網站使用 Ambari Hive 檢視。 如需提交 Hive 工作的其他方法，請參閱 [在 HDInsight 中使用 Hive](hdinsight-use-hive.md)。

1. 若要開啟 Ambari，請從上一個螢幕擷取畫面中，選取 [叢集儀表板]。  您也可以瀏覽至 **https://&lt;ClusterName>.azurehdinsight.net**，其中 &lt;ClusterName> 是您在上一節建立的叢集。

    ![HDInsight Linux 開始使用叢集儀表板](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-get-started-open-cluster-dashboard.png "HDInsight Linux 開始使用叢集儀表板")

2. 輸入您在建立叢集時所指定的 Hadoop 使用者名稱和密碼。 預設的使用者名稱為 **admin**。

3. 開啟 [Hive 檢視]  ，如下列螢幕擷取畫面所示：
   
    ![選取 Ambari 檢視](./media/apache-hadoop-linux-tutorial-get-started/selecthiveview.png "HDInsight Hive 檢視器功能表")

4. 在 [查詢] 索引標籤中，將下列 HiveQL 陳述式貼到工作表中：
   
        SHOW TABLES;

    ![HDInsight Hive 檢視](./media/apache-hadoop-linux-tutorial-get-started/hiveview-1.png "HDInsight Hive 檢視查詢編輯器")
   
   > [!NOTE]
   > Hive 需要分號。       
   > 
   > 

5. 選取 [執行] 。 [結果] 索引標籤會出現 [查詢] 索引標籤下方，並顯示作業相關資訊。 
   
    查詢完成之後，[查詢] 索引標籤會顯示作業的結果。 您應該會看到一個名為 hivesampletable 的資料表。 所有 HDInsight 叢集都提供此範例 Hive 資料表。
   
    ![HDInsight Hive 檢視](./media/apache-hadoop-linux-tutorial-get-started/hiveview.png "HDInsight Hive 檢視查詢編輯器")

6. 重複步驟 4 和 5，以執行下列查詢：
   
        SELECT * FROM hivesampletable;
   
7. 您也可以儲存查詢的結果。 選取右側功能表按鈕，然後指定您是否要以 CSV 檔案格式下載結果，或將其儲存至與叢集相關聯的儲存體帳戶。

    ![儲存 Hive 查詢的結果](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-linux-hive-view-save-results.png "儲存 Hive 查詢的結果")

完成 Hive 工作之後，您可以[將結果匯出至 Azure SQL 資料庫或 SQL Server 資料庫](apache-hadoop-use-sqoop-mac-linux.md)，也可以[使用 Excel 將結果視覺化](apache-hadoop-connect-excel-power-query.md)。 如需在 HDInsight 中使用 Hive 的詳細資訊，請參閱 [搭配 HDInsight 中的 Hadoop 使用 Hive 和 HiveQL 來分析範例 Apache log4j 檔案](hdinsight-use-hive.md)。

## <a name="troubleshoot"></a>疑難排解

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](../hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a name="clean-up-resources"></a>清除資源
完成本教學課程之後，您可以刪除叢集。 利用 HDInsight，您的資料會儲存在 Azure 儲存體中，以便您在未使用叢集時安全地進行刪除。 您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。 

> [!NOTE]
> 如果您要「立即」繼續下一個教學課程，以了解如何使用 HDInsight 上的 Hadoop 來執行 ETL 作業，則建議讓叢集保持執行狀態。 這是因為在教學課程中，您必須重新建立 Hadoop 叢集。 不過，如果您不會立即開始下一個教學課程，則現在就必須刪除該叢集。
> 
>  

**刪除叢集和/或預設儲存體帳戶**

1. 回到瀏覽器索引標籤，您可在其中存取 Azure 入口網站。 您應該位於叢集的 [概觀] 頁面上。 如果您只想刪除叢集，但保留預設儲存體帳戶，請選取 [刪除]。

    ![HDInsight 刪除叢集](./media/apache-hadoop-linux-tutorial-get-started/hdinsight-delete-cluster.png "刪除 HDInsight 叢集")

2. 如果您想要刪除叢集以及預設儲存體帳戶，則選取資源群組名稱 (上一個螢幕擷取畫面中醒目提示的項目) 來開啟資源群組頁面。

3. 選取 [刪除資源群組] 以刪除資源群組，其中包含了叢集和預設儲存體帳戶。 請注意，刪除資源群組會刪除儲存體帳戶。 如果您想要保留儲存體帳戶，請選擇只刪除叢集。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何使用 Resource Manager 範本來建立以 Linux 為基礎的 HDInsight 叢集，以及如何執行基本的 Hive 查詢。 在下一篇文章中，您將了解如何使用 HDInsight 上的 Hadoop 來執行擷取、轉換及載入 (ETL) 作業。

> [!div class="nextstepaction"]
>[使用 HDInsight 上的 Apache Hive 來擷取、轉換和載入資料](../hdinsight-analyze-flight-delay-data-linux.md)

如果您準備好開始使用您自己的資料，並需要進一步了解 HDInsight 儲存資料的方式或如何將資料匯入 HDInsight，請參閱下列文章：

* 如需有關 HDInsight 如何使用 Azure 儲存體的資訊，請參閱 [搭配 HDInsight 使用 Azure 儲存體](../hdinsight-hadoop-use-blob-storage.md)。
* 如需有關如何使用 Data Lake Storage 建立 HDInsight 叢集的資訊，請參閱[快速入門：在 HDInsight 中設定叢集](../../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。
* 如需關於如何上傳資料到 HDInsight 的資訊，請參閱[將資料上傳到 HDInsight](../hdinsight-upload-data.md)。

若要深入了解如何使用 HDInsight 分析資料，請參閱下列文章：

* 若要深入了解如何搭配 HDInsight 使用 Hive，包括如何從 Visual Studio 執行 Hive 查詢，請參閱[搭配 HDInsight 使用 Hive](hdinsight-use-hive.md)。
* 若要了解用來轉換資料的 Pig 語言，請參閱[搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)。
* 若要了解 MapReduce (一種撰寫程式以處理 Hadoop 資料的方式)，請參閱[搭配 HDInsight 使用 MapReduce](hdinsight-use-mapreduce.md)。
* 若要了解如何使用適用於 Visual Studio 的 HDInsight 工具來分析 HDInsight 資料，請參閱 [開始使用 Visual Studio Hadoop tools for HDInsight](apache-hadoop-visual-studio-tools-get-started.md)。


如果您想要深入了解如何建立或管理 HDInsight 叢集，請參閱下列文章：

* 若要了解如何管理以 Linux 為基礎的 HDInsight 叢集，請參閱 [使用 Ambari 管理 HDInsight 叢集](../hdinsight-hadoop-manage-ambari.md)。
* 若要深入了解建立 HDInsight 叢集時可選取的選項，請參閱 [使用自訂選項在 Linux 上建立 HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)。


[1]: ../HDInsight/apache-hadoop-visual-studio-tools-get-started.md

[hdinsight-provision]: hdinsight-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md


