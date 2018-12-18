---
title: '教學課程：在 Azure HDInsight 中使用 Power BI 分析 Apache Spark 資料 '
description: 使用 Microsoft Power BI 將儲存在 HDInsight 叢集中的 Spark 資料視覺化
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,mvc
ms.topic: tutorial
ms.date: 05/07/2018
ms.openlocfilehash: b8f952f27b5971704c8202fe80a95026e513b373
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43045817"
---
# <a name="tutorial-analyze-spark-data-using-power-bi-in-hdinsight"></a>教學課程：在 HDInsight 中使用 Power BI 分析 Spark 資料 

了解如何使用 Microsoft Power BI 在 Azure HDInsight 中將 Apache Spark 叢集中的資料視覺化。

在本教學課程中，您了解如何：
> [!div class="checklist"]
> * 使用 Power BI 將 Spark 資料視覺化

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

* **完成本文[教學課程：在 Azure HDInsight 中的 Apache Spark 叢集上載入資料和執行查詢](./apache-spark-load-data-run-query.md)**。
* **Power BI**：[Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) 與 [Power BI 試用訂用帳戶](https://app.powerbi.com/signupredirect?pbi_source=web) (選擇性)。


## <a name="verify-the-data"></a>驗證資料

您在[上一個教學課程](apache-spark-load-data-run-query.md)中建立的 Jupyter Notebook 包含可建立 `hvac` 資料表的程式碼。 本資料表根據的是 **\HdiSamples\HdiSamples\SensorSampleData\hvac\hvac.csv** 上所有 HDInsight Spark 叢集上的可用 CSV 檔案。 使用下列程序來驗證資料。

1. 從 Jupyter Notebook 貼上下列程式碼，然後按下 **SHIFT + ENTER** 鍵。 該程式碼會驗證資料表是否存在。

    ```PySpark
    %%sql
    SHOW TABLES
    ```

    輸出看起來如下：

    ![顯示 Spark 中的資料表](./media/apache-spark-use-bi-tools/show-tables.png)

    如果在開始本教學課程之前關閉 Notebook，則會清除 `hvactemptable`，因此其不會包含在輸出中。  從 BI 工具只可以存取儲存在中繼存放區的 Hive 資料表 (在 **isTemporary** 資料行下表示為 **False**)。 在此教學課程中，您會連線到您所建立的 **hvac** 資料表。

2. 將以下程式碼貼入空白儲存格，然後按下 **SHIFT + ENTER** 鍵。 該程式碼會驗證資料表中的資料。

    ```PySpark
    %%sql
    SELECT * FROM hvac LIMIT 10
    ```

    輸出看起來如下：

    ![顯示 Spark 中 hvac 資料表的資料列](./media/apache-spark-use-bi-tools/select-limit.png)

3. 從 Notebook 的 [檔案] 功能表中，按一下 [關閉並終止]。 關閉 Notebook 來釋放資源。 

## <a name="visualize-the-data"></a>將資料視覺化

在本節中，您會使用 Power BI 從 Spark 叢集資料建立視覺效果、報表及儀表板。 

### <a name="create-a-report-in-power-bi-desktop"></a>在 Power BI Desktop 中建立報表
使用 Spark 的第一步是連接到 Power BI Desktop 中的叢集，從叢集載入資料，並根據該資料建立基本視覺效果。

> [!NOTE]
> 本文中所示範的連接器目前為預覽版本。 請透過 [Power BI 社群](https://community.powerbi.com/) \(英文\) 網站或 [Power BI Ideas](https://ideas.powerbi.com/forums/265200-power-bi-ideas) \(英文\) 提供您的意見反應。

1. 開啟 [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/)。
1. 按一下 [首頁] 索引標籤中的 [取得資料]，然後按一下 [更多]。

    ![將資料從 HDInsight Apache Spark 送入 Power BI Desktop](./media/apache-spark-use-bi-tools/hdinsight-spark-power-bi-desktop-get-data.png "將資料從 Apache Spark BI 送入 Power BI")


2. 在搜尋方塊中輸入 `Spark`，選取 [Azure HDInsight Spark (搶鮮版 (Beta))]，然後按一下 [連線]。

    ![將資料從 Apache Spark BI 送入 Power BI](./media/apache-spark-use-bi-tools/apache-spark-bi-import-data-power-bi.png "將資料從 Apache Spark BI 送入 Power BI")

3. 輸入您的叢集 URL (格式為 `mysparkcluster.azurehdinsight.net`)，選取 [DirectQuery]，然後按一下 [確定]。

    您可以搭配 Spark 使用任一種資料連線模式。 如果您使用 DirectQuery，變更會反映在報表中，而不需重新整理整個資料集。 如果您是匯入資料，則必須重新整理資料集，才能看到變更。 如需如何使用以及該在何時使用 DirectQuery 的詳細資訊，請參閱[使用 Power BI 中的 DirectQuery](https://powerbi.microsoft.com/documentation/powerbi-desktop-directquery-about/)。 

4. 輸入 HDInsight 登入帳戶資訊，然後按一下 [連線]。 預設帳戶名稱為 *admin*。

5. 選取 `hvac` 資料表，等待查看資料的預覽，然後按一下 [載入]。

    ![Spark 叢集使用者名稱和密碼](./media/apache-spark-use-bi-tools/apache-spark-bi-select-table.png "Spark 叢集使用者名稱和密碼")

    Power BI Desktop 擁有連線至 Spark 叢集以及從 `hvac` 資料表載入資料所需的資訊。 資料表及其資料行會顯示在 [欄位] 窗格中。  請參閱下列螢幕擷取畫面：

6. 將每棟建築物之目標溫度和實際溫度間的差異視覺化： 

    1. 在 [視覺效果] 窗格中，選取 [區域圖]。 
    2. 將 [BuildingID] 欄位拖曳至 [軸]，並將 [ActualTemp] 和 [TargetTemp] 欄位拖曳至 [值]。

        ![使用 Apache Spark BI 建立 Spark 資料視覺效果](./media/apache-spark-use-bi-tools/apache-spark-bi-add-value-columns.png "使用 Apache Spark BI 建立 Spark 資料視覺效果")

        圖表看起來如下：

        ![使用 Apache Spark BI 建立 Spark 資料視覺效果](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph.png "使用 Apache Spark BI 建立 Spark 資料視覺效果")

        根據預設，視覺效果會顯示 **ActualTemp** 和 **TargetTemp** 的總和。 在 [視覺效果] 窗格中，按一下 [ActualTemp] 與 [TargetTemp] 旁的向下箭頭，您會看到已選取 [總和]。

    3. 在 [視覺效果] 窗格中，按一下 [ActualTemp] 與 [TargetTemp] 旁的向下箭頭，選取 [平均值] 以取得每棟建築物之實際溫度與目標溫度的平均值。

        ![使用 Apache Spark BI 建立 Spark 資料視覺效果](./media/apache-spark-use-bi-tools/apache-spark-bi-average-of-values.png "使用 Apache Spark BI 建立 Spark 資料視覺效果")

        您的資料視覺效果應該類似於螢幕擷取畫面中的視覺效果。 在視覺效果上移動游標可取得相關資料的工具提示。

        ![使用 Apache Spark BI 建立 Spark 資料視覺效果](./media/apache-spark-use-bi-tools/apache-spark-bi-area-graph-sum.png "使用 Apache Spark BI 建立 Spark 資料視覺效果")

7. 按一下 [檔案]，然後 [儲存]，並輸入檔案名稱 `BuildingTemperature.pbix`。 

### <a name="publish-the-report-to-the-power-bi-service-optional"></a>將報告發佈至 Power BI 服務 (選用)

Power BI 服務可讓您跨組織共用報表和儀表板。 在本節中，您會首次發佈資料集和報表。 然後，您會將報表釘選到儀表板。 儀表板通常用於強調報告中的資料子集；您的報告中只有一個視覺效果，但逐步了解下列步驟仍很有幫助。

1. 開啟 Power BI Desktop。
2. 在 [首頁] 索引標籤上，按一下 [發佈]。

    ![從 Power BI Desktop 發佈](./media/apache-spark-use-bi-tools/apache-spark-bi-publish.png "從 Power BI Desktop 發佈")

2. 選取要發佈您的資料集和報告的目標工作區，然後按一下 [選取]。 在下列映像中，已選取預設值 [我的工作區]。

    ![選取要發佈資料集和報表的目標工作區](./media/apache-spark-use-bi-tools/apache-spark-bi-select-workspace.png "選取要發佈資料集和報表的目標工作區") 

3. 成功發佈之後，按一下 [在 Power BI 中開啟 'BuildingTemperature.pbix']。

    ![發佈成功，按一下以輸入認證](./media/apache-spark-use-bi-tools/apache-spark-bi-publish-success.png "發佈成功，按一下以輸入認證") 

4. 在 Power BI 服務中，按一下 [輸入認證]。

    ![在 Power BI 服務中輸入認證](./media/apache-spark-use-bi-tools/apache-spark-bi-enter-credentials.png "在 Power BI 服務中輸入認證")

5. 按一下 [編輯認證]。

    ![在 Power BI 服務中編輯認證](./media/apache-spark-use-bi-tools/apache-spark-bi-edit-credentials.png "在 Power BI 服務中編輯認證")

6. 輸入 HDInsight 登入帳戶資訊，然後按一下 [登入]。 預設帳戶名稱為 *admin*。

    ![登入 Spark 叢集](./media/apache-spark-use-bi-tools/apache-spark-bi-sign-in.png "登入 Spark 叢集")

7. 在左側窗格中，移至 [工作區] > [我的工作區] > [報表]，然後按一下 [BuildingTemperature]。

    ![左側窗格中報告下列出的報表](./media/apache-spark-use-bi-tools/apache-spark-bi-service-left-pane.png "左側窗格中報表下列出的報告")

    您也應該會看到 [BuildingTemperature] 列於左側窗格中 [資料集] 底下。

    現在您在 Power BI Desktop 中建立的視覺效果已可於 Power BI 服務中使用。 

8. 將游標暫留在視覺效果上，然後按一下右上角的釘選圖示。

    ![Power BI 服務中的報告](./media/apache-spark-use-bi-tools/apache-spark-bi-service-report.png "Power BI 服務中的報告")

9. 選取 [新的儀表板]，輸入名稱 `Building temperature`，然後按一下 [釘選]。

    ![釘選到新的儀表板](./media/apache-spark-use-bi-tools/apache-spark-bi-pin-dashboard.png "釘選到新的儀表板")

10. 在報告中，按一下 [移至儀表板]。 

您的視覺效果已釘選至儀表板 - 您可以將其他視覺效果新增至報告，並將其釘選至相同的儀表板。 如需關於報告與儀表板的詳細資訊，請參閱 [Power BI 中的報告](https://powerbi.microsoft.com/documentation/powerbi-service-reports/)和 [Power BI 中的儀表板](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/)。

<!--
## <a name="tableau"></a>Use Tableau Desktop 

> [!NOTE]
> This section is applicable only for Spark 1.5.2 clusters created in Azure HDInsight.
>
>

1. Install [Tableau Desktop](http://www.tableau.com/products/desktop) on the computer where you are running this Apache Spark BI tutorial.

2. Make sure that computer also has Microsoft Spark ODBC driver installed. You can install the driver from [here](http://go.microsoft.com/fwlink/?LinkId=616229).

1. Launch Tableau Desktop. In the left pane, from the list of server to connect to, click **Spark SQL**. If Spark SQL is not listed by default in the left pane, you can find it by click **More Servers**.
2. In the Spark SQL connection dialog box, provide the values as shown in the screenshot, and then click **OK**.

    ![Connect to a cluster for Apache Spark BI](./media/apache-spark-use-bi-tools/connect-to-tableau-apache-spark-bi.png "Connect to a cluster for Apache Spark BI")

    The authentication drop-down lists **Microsoft Azure HDInsight Service** as an option, only if you installed the [Microsoft Spark ODBC Driver](http://go.microsoft.com/fwlink/?LinkId=616229) on the computer.
3. On the next screen, from the **Schema** drop-down, click the **Find** icon, and then click **default**.

    ![Find schema for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-find-schema-apache-spark-bi.png "Find schema for Apache Spark BI")
4. For the **Table** field, click the **Find** icon again to list all the Hive tables available in the cluster. You should see the **hvac** table you created earlier using the notebook.

    ![Find table for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-find-table-apache-spark-bi.png "Find table for Apache Spark BI")
5. Drag and drop the table to the top box on the right. Tableau imports the data and displays the schema as highlighted by the red box.

    ![Add tables to Tableau for Apache Spark BI](./media/apache-spark-use-bi-tools/tableau-add-table-apache-spark-bi.png "Add tables to Tableau for Apache Spark BI")
6. Click the **Sheet1** tab at the bottom left. Make a visualization that shows the average target and actual temperatures for all buildings for each date. Drag **Date** and **Building ID** to **Columns** and **Actual Temp**/**Target Temp** to **Rows**. Under **Marks**, select **Area** to use an area map for Spark data visualization.

     ![Add fields for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-add-fields.png "Add fields for Spark data visualization")
7. By default, the temperature fields are shown as aggregate. If you want to show the average temperatures instead, you can do so from the drop-down, as shown in the following screenshot:

    ![Take average of temperature for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-average-temperature.png "Take average of temperature for Spark data visualization")

8. You can also super-impose one temperature map over the other to get a better feel of difference between target and actual temperatures. Move the mouse to the corner of the lower area map until you see the handle shape highlighted in a red circle. Drag the map to the other map on the top and release the mouse when you see the shape highlighted in red rectangle.

    ![Merge maps for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-merge-maps.png "Merge maps for Spark data visualization")

     Your data visualization should change as shown in the screenshot:

    ![Tableau output for Spark data visualization](./media/apache-spark-use-bi-tools/spark-data-visualization-tableau-output.png "Tableau output for Spark data visualization")
9. Click **Save** to save the worksheet. You can create dashboards and add one or more sheets to it.
-->

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

- 使用 Power BI 將 Spark 資料視覺化。

前往下一篇文章，以查看如何將您在 Spark 中註冊的資料提取至 BI 分析工具，例如 Power BI。 
> [!div class="nextstepaction"]
> [執行 Spark 串流作業](apache-spark-eventhub-streaming.md)

