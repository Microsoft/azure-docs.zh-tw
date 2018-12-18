---
title: 使用 Azure Data Factory 中的 Spark 轉換資料 | Microsoft Docs
description: 本教學課程提供逐步指示，說明如何使用 Azure Data Factory 中的 Spark 活動來轉換資料。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/10/2018
ms.author: douglasl
ms.openlocfilehash: f1cc1b728a91c22f9b4b2062ed5c423314e561c8
ms.sourcegitcommit: 7bc4a872c170e3416052c87287391bc7adbf84ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48017579"
---
# <a name="transform-data-in-the-cloud-by-using-a-spark-activity-in-azure-data-factory"></a>使用 Azure Data Factory 中的 Spark 活動來轉換雲端中的資料
在本教學課程中，您會使用 Azure 入口網站來建立 Azure Data Factory 管線。 此管線使用 Spark 活動和隨選 Azure HDInsight 連結服務來轉換資料。 

您會在本教學課程中執行下列步驟：

> [!div class="checklist"]
> * 建立資料處理站。 
> * 建立使用 Spark 活動的管線。
> * 觸發管線執行。
> * 監視管道執行。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="prerequisites"></a>必要條件
* **Azure 儲存體帳戶**。 您需要建立 Python 指令碼和輸入檔案，並上傳至 Azure 儲存體。 Spark 程式的輸出會儲存在這個儲存體帳戶中。 隨選 Spark 叢集與其主要儲存體是使用相同的儲存體帳戶。  

> [!NOTE]
> HdInsight 僅支援標準層的一般用途儲存體帳戶。 請確定帳戶不是進階或僅限 Blob 儲存體帳戶。

* **Azure PowerShell**(英文)。 遵循[如何安裝並設定 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的指示。


### <a name="upload-the-python-script-to-your-blob-storage-account"></a>將 Python 指令碼上傳至 Blob 儲存體帳戶
1. 使用下列內容建立名為 **WordCount_Spark.py** 的 Python 檔案： 

    ```python
    import sys
    from operator import add
    
    from pyspark.sql import SparkSession
    
    def main():
        spark = SparkSession\
            .builder\
            .appName("PythonWordCount")\
            .getOrCreate()
            
        lines = spark.read.text("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/inputfiles/minecraftstory.txt").rdd.map(lambda r: r[0])
        counts = lines.flatMap(lambda x: x.split(' ')) \
            .map(lambda x: (x, 1)) \
            .reduceByKey(add)
        counts.saveAsTextFile("wasbs://adftutorial@<storageaccountname>.blob.core.windows.net/spark/outputfiles/wordcount")
        
        spark.stop()
    
    if __name__ == "__main__":
        main()
    ```
1. 以您的 Azure 儲存體帳戶名稱取代 *&lt;storageAccountName&gt;*。 然後儲存檔案。 
1. 在 Azure Blob 儲存體中，建立名為 **adftutorial** 的容器 (如果不存在)。 
1. 建立名為 **spark** 的資料夾。
1. 在 **spark** 資料夾下，建立名為 **script** 的子資料夾。 
1. 將 **WordCount_Spark.py** 檔案上傳至 **script** 子資料夾。 


### <a name="upload-the-input-file"></a>上傳輸入檔案
1. 建立名為 **minecraftstory.txt** 的檔案並填入一些文字。 Spark 程式會計算這段文字中的字數。 
1. 在 **spark** 資料夾下，建立名為 **inputfiles** 的子資料夾。 
1. 將 **minecraftstory.txt** 檔案上傳至 **inputfiles** 子資料夾。 

## <a name="create-a-data-factory"></a>建立 Data Factory

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
1. 選取左側功能表上的 [新增]、[資料 + 分析]，然後選取 [資料處理站]。 
   
   ![在 [新增] 窗格中選取資料處理站](./media/tutorial-transform-data-spark-portal/new-azure-data-factory-menu.png)
1. 在 [新增資料處理站] 窗格的 [名稱] 下，輸入 **ADFTutorialDataFactory**。 
      
   ![[新增資料處理站] 窗格](./media/tutorial-transform-data-spark-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 *全域唯一的*。 如果您看到下列錯誤，請變更資料處理站的名稱。 (例如，使用 **&lt;yourname&gt;ADFTutorialDataFactory**)。 如需 Data Factory 成品的命名規則，請參閱 [Data Factory - 命名規則](naming-rules.md)一文。
  
   ![名稱無法使用時的錯誤](./media/tutorial-transform-data-spark-portal/name-not-available-error.png)
1. 針對 [訂用帳戶]，選取您要用來建立資料處理站的 Azure 訂用帳戶。 
1. 針對 [資源群組]，採取下列其中一個步驟︰
     
   - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
   - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
   本快速入門的某些步驟是假設您使用 **ADFTutorialResourceGroup** 作為資源群組名稱。 若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
1. 針對 [版本]，選取 [V2]。
1. 針對 [位置]，選取資料處理站的位置。 

   如需目前可使用 Data Factory 的 Azure 區域清單，請在下列頁面上選取您感興趣的區域，然後展開 [分析] 以找出 [Data Factory]：[依區域提供的產品](https://azure.microsoft.com/global-infrastructure/services/)。 Data Factory 所使用的資料存放區 (Azure 儲存體和 Azure SQL Database 等) 和計算 (HDInsight 等) 可位於其他區域。

1. 選取 [建立] 。

1. 建立完成之後，您會看到 [Data Factory] 頁面。 選取 [編寫與監視] 圖格，以在個別的索引標籤上啟動 Data Factory UI 應用程式。

    ![資料處理站的首頁，具有 [編寫與監視] 圖格](./media/tutorial-transform-data-spark-portal/data-factory-home-page.png)

## <a name="create-linked-services"></a>建立連結的服務
在本節中，您會撰寫兩個連結服務： 
    
- 將 Azure 儲存體帳戶連結至資料處理站的 **Azure 儲存體連結服務**。 隨選 HDInsight 叢集會使用此儲存體。 而且也包含要執行的 Spark 指令碼。 
- **隨選 HDInsight 連結服務**。 Azure Data Factory 會自動建立 HDInsight 叢集並執行 Spark 程式。 然後在 HDInsight 叢集的閒置時間達到預先設定的時間後，系統就會刪除該叢集。 

### <a name="create-an-azure-storage-linked-service"></a>建立 Azure 儲存體連結服務

1. 在 [現在就開始吧] 頁面上，切換至左面板中的 [編輯] 索引標籤。 

   ![[現在就開始吧] 頁面](./media/tutorial-transform-data-spark-portal/get-started-page.png)

1. 選取視窗底部的 [連線]，然後選取 [+ 新增]。 

   ![用於建立新連線的按鈕](./media/tutorial-transform-data-spark-portal/new-connection.png)
1. 在 [新增連結服務] 視窗中，選取 [資料存放區] > [Azure Blob 儲存體]，然後選取 [繼續]。 

   ![選取 [Azure Blob 儲存體] 圖格](./media/tutorial-transform-data-spark-portal/select-azure-storage.png)
1. 針對 [儲存體帳戶名稱]，從清單中選取名稱，然後選取 [儲存]。 

   ![用於指定儲存體帳戶名稱的方塊](./media/tutorial-transform-data-spark-portal/new-azure-storage-linked-service.png)


### <a name="create-an-on-demand-hdinsight-linked-service"></a>建立隨選 HDInsight 連結服務

1. 再次選取 [+新增] 按鈕以建立另一個連結服務。 
1. 在 [新增連結服務] 視窗中，選取 [計算] > [Azure HDInsight]，然後選取 [繼續]。 

   ![選取 [Azure HDInsight] 圖格](./media/tutorial-transform-data-spark-portal/select-azure-hdinsight.png)
1. 在 [新增連結服務] 視窗中，完成下列步驟： 

   a. 針對 [名稱]，輸入 **AzureHDInsightLinkedService**。
   
   b. 針對 [類型]，確認已選取 [隨選 HDInsight]。
   
   c. 針對 [Azure 儲存體連結服務]，選取 [AzureStorage1]。 您之前已建立此連結服務。 如果您使用不同的名稱，請在此指定正確的名稱。 
   
   d. 針對 [叢集類型]，選取 [spark]。
   
   e. 針對 [服務主體識別碼]，輸入有權建立 HDInsight 叢集的服務主體識別碼。 
   
      此服務主體必須是訂用帳戶的參與者角色成員，或建立叢集所在的資源群組成員。 如需詳細資訊，請參閱[建立 Azure Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)。
   
   f. 針對 [服務主體金鑰]，輸入金鑰。 
   
   g. 針對 [資源群組]，選取建立資料處理站時使用的相同資源群組。 將在此資源群組中建立 Spark 叢集。 
   
   h. 展開 [OS 類型]。
   
   i. 針對**叢集使用者名稱**輸入名稱。 
   
   j. 輸入使用者的**叢集密碼**。 
   
   k. 選取 [完成]。 

   ![HDInsight 連結服務設定](./media/tutorial-transform-data-spark-portal/azure-hdinsight-linked-service-settings.png)

> [!NOTE]
> Azure HDInsight 會限制您在其支援的每個 Azure 區域中可使用的核心總數。 對於隨選 HDInsight 連結服務，用來建立 HDInsight 叢集的 Azure 儲存體位置與用來作為其主要儲存體的位置相同。 請確定您有足夠的核心配額，才能成功建立叢集。 如需詳細資訊，請參閱[使用 Hadoop、Spark 及 Kafka 等在 HDInsight 中設定叢集](../hdinsight/hdinsight-hadoop-provision-linux-clusters.md)。 

## <a name="create-a-pipeline"></a>建立管線

1. 選取 **+** (加號) 按鈕，然後選取功能表上的 [管線]。

   ![用於建立新管線的按鈕](./media/tutorial-transform-data-spark-portal/new-pipeline-menu.png)
1. 在 [活動] 工具箱中，展開 [HDInsight]。 將 [活動] 工具箱中的 [Spark] 活動拖到管線設計工具介面。 

   ![拖曳 Spark 活動](./media/tutorial-transform-data-spark-portal/drag-drop-spark-activity.png)
1. 在 [Spark] 活動視窗底部的屬性中，完成下列步驟： 

   a. 切換至 [HDI 叢集] 索引標籤。
   
   b. 選取您在上一個程序中建立的 **AzureHDInsightLinkedService**。 
        
   ![指定 HDInsight 連結服務](./media/tutorial-transform-data-spark-portal/select-hdinsight-linked-service.png)
1. 切換至 [指令碼/Jar] 索引標籤並完成下列步驟： 

   a. 針對 [作業連結服務]，選取 [AzureStorage1]。
   
   b. 選取 [瀏覽儲存體]。

   ![在 [指令碼/Jar] 索引標籤上選取 Spark 指令碼](./media/tutorial-transform-data-spark-portal/specify-spark-script.png)
   
   c. 瀏覽至 **adftutorial/spark/script** 資料夾，選取 **WordCount_Spark.py**，然後選取 [完成]。      

1. 若要驗證管線，選取工具列上的 [驗證] 按鈕。 選取 **>>** (右箭頭) 按鈕以關閉驗證視窗。 
    
   ![[驗證] 按鈕](./media/tutorial-transform-data-spark-portal/validate-button.png)
1. 選取 [全部發佈]。 Data Factory UI 會將實體 (連結服務和管線) 發佈至 Azure Data Factory 服務。 
    
   ![[全部發佈] 按鈕](./media/tutorial-transform-data-spark-portal/publish-button.png)


## <a name="trigger-a-pipeline-run"></a>觸發管線執行
選取工具列上的 [觸發程序]，然後選取 [立即觸發]。 

![[觸發程序] 和 [立即觸發程序] 按鈕](./media/tutorial-transform-data-spark-portal/trigger-now-menu.png)

## <a name="monitor-the-pipeline-run"></a>監視管道執行

1. 切換至 [監視] 索引標籤。確認您看到管線執行。 建立 Spark 叢集需要約 20 分鐘。 
   
1. 定期選取 [重新整理] 以檢查管線執行的狀態。 

   ![用來監視管線執行的索引標籤，具有 [重新整理] 按鈕](./media/tutorial-transform-data-spark-portal/monitor-tab.png)

1. 若要檢視與管線執行相關聯的活動執行，請選取 [動作] 資料行中的 [檢視活動執行]。

   ![管線執行狀態](./media/tutorial-transform-data-spark-portal/pipeline-run-succeeded.png) 

   您可以選取頂端的 [管線] 連結，切換回管線執行檢視。

   ![[活動執行] 檢視](./media/tutorial-transform-data-spark-portal/activity-runs.png)

## <a name="verify-the-output"></a>驗證輸出
確認已在 adftutorial 容器的 spark/otuputfiles/wordcount 資料夾中建立輸出檔案。 

![輸出檔案的位置](./media/tutorial-transform-data-spark-portal/verity-output.png)

這個檔案應該有輸入文字檔中的每個字組，以及字組在檔案中出現的次數。 例如︰ 

```
(u'This', 1)
(u'a', 1)
(u'is', 1)
(u'test', 1)
(u'file', 1)
```

## <a name="next-steps"></a>後續步驟
此範例中的管線會使用 Spark 活動和隨選 HDInsight 連結服務來轉換資料。 您已了解如何︰ 

> [!div class="checklist"]
> * 建立資料處理站。 
> * 建立使用 Spark 活動的管線。
> * 觸發管線執行。
> * 監視管道執行。

若要了解如何在虛擬網路中的 Azure HDInsight 叢集上執行 Hive 指令碼來轉換資料，請進入下一個教學課程： 

> [!div class="nextstepaction"]
> [教學課程：在 Azure 虛擬網路中使用 Hive 來轉換資料](tutorial-transform-data-hive-virtual-network-portal.md)。





