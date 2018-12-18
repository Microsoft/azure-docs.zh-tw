---
title: 使用擴充的 Spark 記錄伺服器針對 Spark 應用程式進行偵錯和診斷 - Azure HDInsight
description: 使用擴充的 Spark 記錄伺服器對 Spark 應用程式進行偵錯和診斷 - Azure HDInsight。
services: hdinsight
ms.service: hdinsight
author: jejiang
ms.author: jejiang
ms.reviewer: jasonh
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 09/14/2018
ms.openlocfilehash: 4627593e4ab96c63423a7afd6152f3a004bc6c3f
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47042358"
---
# <a name="use-extended-spark-history-server-to-debug-and-diagnose-spark-applications"></a>使用擴充的 Spark 記錄伺服器對 Spark 應用程式進行偵錯和診斷

此文章將說明如何使用擴充的 Spark 記錄伺服器，對已完成和執行中的 Spark 應用程式進行偵錯及診斷。 此延伸模組包含 [資料] 索引標籤和 [圖表] 索引標籤和 [診斷] 索引標籤。在 [資料] 索引標籤上，使用者可以檢查 Spark 作業的輸入與輸出。 在 [圖表] 索引標籤上，使用者可以檢查資料流程，並重新執行作業圖表。 在 [診斷] 索引標籤上，使用者可以參考 [資料扭曲]、[時間扭曲] 與 [執行程式使用狀況分析]。

## <a name="get-access-to-spark-history-server"></a>存取 Spark 歷程記錄伺服器

「Spark 記錄伺服器」是已完成和執行中 Spark 應用程式的 Web UI。 

### <a name="open-the-spark-history-server-web-ui-from-azure-portal"></a>從 Azure 入口網站開啟 Spark 歷程記錄伺服器 Web UI

1. 從 [Azure 入口網站](https://portal.azure.com/)，開啟 Spark 叢集。 如需詳細資訊，請參閱[列出和顯示叢集](../hdinsight-administer-use-portal-linux.md#list-and-show-clusters)。
2. 從 [快速連結]，按一下 [叢集儀表板]，然後按一下 [Spark 記錄伺服器]。 出現提示時，輸入 Spark 叢集的系統管理員認證。 

    ![Spark 歷程記錄伺服器](./media/apache-azure-spark-history-server/launch-history-server.png "Spark 歷程記錄伺服器")

### <a name="open-the-spark-history-server-web-ui-by-url"></a>透過 URL 開啟 Spark 記錄伺服器 Web UI
藉由瀏覽至下列 URL 來開啟 Spark 記錄伺服器，並以客戶的 Spark 叢集名稱取代 <ClusterName>。

   ```
   https://<ClusterName>.azurehdinsight.net/sparkhistory
   ```

「Spark 記錄伺服器」Web UI 看起來像這樣：

![HDInsight Spark 記錄伺服器](./media/apache-azure-spark-history-server/hdinsight-spark-history-server.png)


## <a name="data-tab-in-spark-history-server"></a>Spark 歷程記錄伺服器中的 [資料] 索引標籤
選取作業識別碼，然後按一下工具功能表上的 [資料]，以取得資料檢視。

+ 您可以分別選取 [輸入]、[輸出]和 [資料表作業] 索引標籤來檢查這些項目。

    ![[資料] 索引標籤](./media/apache-azure-spark-history-server/sparkui-data-tabs.png)

+ 按一下 [複製] 按鈕可複製所有資料列。

    ![資料複製](./media/apache-azure-spark-history-server/sparkui-data-copy.png)

+ 按一下 [csv] 按鈕，可將所有資料儲存為 CSV 檔案。

    ![資料儲存](./media/apache-azure-spark-history-server/sparkui-data-save.png)

+ 在 [搜尋] 欄位中輸入關鍵字，即可進行搜尋，而搜尋結果會立即顯示。

    ![資料搜尋](./media/apache-azure-spark-history-server/sparkui-data-search.png)

+ 按一下資料行標頭可排序資料表，按一下加號可展開資料列，以顯示更多詳細資料，或按一下減號來摺疊資料列。

    ![資料表](./media/apache-azure-spark-history-server/sparkui-data-table.png)

+ 若要下載單一檔案，請按一下右側的 [部分下載] 按鈕，然後選取要下載到本機的檔案，如果檔案已不存在，顯示錯誤訊息的新索引標籤會隨即開啟。

    ![資料的下載資料列](./media/apache-azure-spark-history-server/sparkui-data-download-row.png)

+ 若要複製完整路徑或相對路徑，可從展開的下載功能表中選取 [複製完整路徑] 或 [複製相對路徑]。 針對 Azure Data Lake Storage 檔案，[在 Azure 儲存體總管中開啟] 會啟動 Azure 儲存體總管，並在登入時移至該資料夾。

    ![資料的複製路徑](./media/apache-azure-spark-history-server/sparkui-data-copy-path.png)

+ 按一下資料表下方的編號，可在單一頁面上顯示太多資料列時，瀏覽多個頁面。 

    ![資料頁面](./media/apache-azure-spark-history-server/sparkui-data-page.png)

+ 將滑鼠停留在 [資料] 旁的問號上方可顯示工具提示，或按一下問號以取得詳細資訊。

    ![資料的詳細資訊](./media/apache-azure-spark-history-server/sparkui-data-more-info.png)

+ 按一下 [提供意見反應給我們]，可將意見反應和問題傳送給我們。

    ![圖表的意見反應](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)


## <a name="graph-tab-in-spark-history-server"></a>Spark 歷程記錄伺服器中的 [圖表] 索引標籤
選取作業識別碼，然後按一下工具功能表上的 [圖表]，以取得作業圖表檢視。

+ 透過產生的作業圖表來檢查您的作業概觀。 

+ 根據預設，作業圖表會顯示所有作業，並且可依據**作業識別碼**進行篩選。

    ![圖表作業識別碼](./media/apache-azure-spark-history-server/sparkui-graph-jobid.png)

+ 系統會預設為選取 [進度]，使用者可以在 [顯示] 的下拉式清單中選取 [讀取]/[寫入] 來檢查資料流程。

    ![圖表顯示](./media/apache-azure-spark-history-server/sparkui-graph-display.png)

    圖表節點會以表示熱度圖的色彩來顯示。

    ![圖表熱度圖](./media/apache-azure-spark-history-server/sparkui-graph-heatmap.png)

+ 按一下 [播放] 按鈕可播放作業，而按一下 [停止] 按鈕可隨時停止。 播放時，工作會以不同色彩來顯示，以表示不同狀態：

    + 綠色表示成功：作業已順利完成。
    + 橘色表示重試：失敗但不會影響作業最終結果的工作執行個體。 這些工作有之後可能會成功的重複或重試執行個體。
    + 藍色表示執行中：工作正在執行中。
    + 白色表示等候中或已略過：工作在等候執行，或已略過該階段。
    + 紅色表示失敗：工作失敗。

    ![正在執行的圖表色彩範例](./media/apache-azure-spark-history-server/sparkui-graph-color-running.png)
 
    已略過的階段會以白色顯示。
    ![圖表色彩範例，略過](./media/apache-azure-spark-history-server/sparkui-graph-color-skip.png)

    ![失敗的圖表色彩範例](./media/apache-azure-spark-history-server/sparkui-graph-color-failed.png)
 
    > [!NOTE]
    > 每個作業都可播放。 針對不完整的作業，不支援播放。


+ 捲動滑鼠滾輪可縮放作業圖表，或按一下 [縮放至適當比例]，以調整成符合螢幕的大小。
 
    ![圖表縮放至適當比例](./media/apache-azure-spark-history-server/sparkui-graph-zoom2fit.png)

+ 如果有失敗的工作，將滑鼠停留在圖表節點上方可查看工具提示，然後按一下階段可開啟階段頁面。

    ![圖表工具提示](./media/apache-azure-spark-history-server/sparkui-graph-tooltip.png)

+ 在 [作業圖表] 索引標籤中，若階段有符合以下條件的工作，將會顯示工具提示與小圖示：
    + 資料扭曲：資料讀取大小 > 此階段中所有工作的平均資料讀取大小 * 2 與資料讀取大寫 > 10 MB
    + 時間扭曲：執行時間 > 此階段中包含之所有工作的平均執行時間 * 2 與執行時間 > 2 分鐘

    ![圖表扭曲圖示](./media/apache-azure-spark-history-server/sparkui-graph-skew-icon.png)

+ 作業圖表節點會顯示每個階段的下列資訊：
    + 識別碼。
    + 名稱或描述。
    + 工作總數。
    + 讀取的資料：輸入大小和隨機讀取大小的總和。
    + 寫入的資料：輸出大小和隨機寫入大小的總和。
    + 執行時間：第一次嘗試開始時和最後一次嘗試完成時之間的時間。
    + 資料列計數：輸入記錄、輸出記錄、隨機讀取記錄和隨機寫入記錄的總和。
    + 進度。

    > [!NOTE]
    > 根據預設，作業圖表節點會顯示每個階段 (不包括階段執行時間) 最後一次嘗試時的資訊 ，但是播放期間的圖表節點會顯示每一次嘗試的資訊。

    > [!NOTE]
    > 針對讀取和寫入的資料大小，我們使用 1MB = 1000 KB = 1000 * 1000 個位元組。

+ 按一下 [提供意見反應給我們]，可將意見反應和問題傳送給我們。

    ![圖表的意見反應](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)


## <a name="diagnosis-tab-in-spark-history-server"></a>Spark 歷程記錄伺服器中的 [診斷] 索引標籤
選取作業識別碼，然後按一下工具功能表上的 [診斷]，以取得作業診斷檢視。 [診斷] 所有標籤包括 [資料扭曲]、[時間扭曲] 與 [執行程式使用狀況分析]。
    
+ 透過選取對應的索引標籤以查看 [資料扭曲]、[時間扭曲] 與 [執行程式使用狀況分析]。

    ![[診斷] 索引標籤](./media/apache-azure-spark-history-server/sparkui-diagnosis-tabs.png)

### <a name="data-skew"></a>資料扭曲
按一下 [資料扭曲] 索引標籤，即會根據指定的參數顯示對應的扭曲工作。 

+ **指定參數** - 第一個區段會顯示用來偵測資料扭曲的參數。 內建規則是：工作資料讀取大於平均工作資料讀取的 3 倍，而工作資料讀取超過 10MB。 若要為扭曲工作定義您的自己的規則，您可以選擇您的參數，[扭曲階段] 與 [扭曲字元] 區段將相應重新整理。

+ **扭曲階段** - 第二個區段會顯示具有符合上面指定條件之扭曲工作的階段。 若階段中有多個扭曲工作，扭曲階段表格只會顯示最扭曲的工作 (例如要用於資料扭曲的最大資料)。

    ![資料扭曲區段 2](./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section2.png)

+ **扭曲圖表** – 當扭曲階段表格中的列已被選取時，扭曲表格會根據資料讀取與執行時間來顯示更多工作分派詳細資料。 扭曲工作會標示為紅色，且正常工作會標示為藍色。 針對效能考量，圖表只會顯示最多 100 個範例工作。 工作詳細資料會顯示在右下角的窗格中。

    ![資料扭曲區段 3](./media/apache-azure-spark-history-server/sparkui-diagnosis-dataskew-section3.png)

### <a name="time-skew"></a>時間扭曲
[時間扭曲] 索引標籤會根據工作執行時間來顯示扭曲工作。 

+ **指定參數** - 第一個區段會顯示用來偵測時間扭曲的參數。 偵測時間扭曲的預設條件：工作執行時間大於平均執行時間 3 倍，而工作執行時間大於 30 秒。 您可以根據您的需求來變更參數。 [扭曲階段] 與 [扭曲圖表] 會顯示對應的階段與工作資序，就像上面的 [資料扭曲] 索引標籤一樣。

+ 按一下 [時間扭曲]，然後系統會根據在 [指定參數] 區段中設定的參數在 [扭曲階段] 區段中顯示篩選的結果。 按一下 [扭曲階段] 區段中的某個項目，接著對應的圖表會在區段 3 中顯示為草稿，而且工作詳細資料會顯示在右下角的窗格。

    ![匙時間扭曲區段 2](./media/apache-azure-spark-history-server/sparkui-diagnosis-timeskew-section2.png)

### <a name="executor-usage-analysis"></a>執行程式使用狀況分析
執行程式使用狀況圖表會將 Spark 作業實際執行程式配置與執行狀態視覺化。  

+ 按一下 [執行程式使用狀況分析]，接著會顯示四個關於執行程式使用狀況的類型曲線草稿，包括 [已配置的執行程式]、[執行中的執行程式]、[閒置執行程式] 與 [執行程式執行個體上限] 。 關於已配置的執行程式，「已新增執行程式」或「已移除執行程式」事件將會使得已配置的執行程式數目增加或減少，您可以查看「作業」中的「事件時間表」以取得更多比較。

    ![[執行程式] 索引標籤](./media/apache-azure-spark-history-server/sparkui-diagnosis-executors.png)

+ 按一下色彩圖示以選取或取消選取所有草稿中的對應內容。

    ![選取圖表](./media/apache-azure-spark-history-server/sparkui-diagnosis-select-chart.png)


## <a name="faq"></a>常見問題集

### <a name="1-revert-to-community-version"></a>1.還原為社群版本

若要還原為社群版本，請執行下列步驟：

1. 在 Ambari 中開啟叢集。 按一下左側面板中的 [Spark2]。
2. 按一下 [設定] 索引標籤。
3. 展開 **Custom spark2-defaults** 群組。
4. 按一下 [新增屬性]，新增 **spark.ui.enhancement.enabled=false**，然後儲存。
5. 屬性現在會設定為 **false**。
6. 按一下 **[儲存]** 儲存組態。

    ![功能關閉](./media/apache-azure-spark-history-server/sparkui-turn-off.png)

7. 按一下左側面板中的 [Spark2]，在 [摘要] 索引標籤下方，按一下 [Spark2 記錄伺服器]。

    ![重新啟動伺服器 1](./media/apache-azure-spark-history-server/sparkui-restart-1.png) 

8. 按一下 **Spark2 記錄伺服器**的 [重新啟動]，以重新啟動記錄伺服器。

    ![重新啟動伺服器 2](./media/apache-azure-spark-history-server/sparkui-restart-2.png)  

9. Spark 記錄伺服器 Web UI 在重新整理後就會還原成社群版本。

### <a name="2-upload-history-server-event"></a>2.上傳記錄伺服器事件

如果您遇到記錄伺服器錯誤，請依照下列步驟來提供事件：
1. 按一下記錄伺服器 Web UI 中的 [下載] 來下載事件。

    ![下載事件](./media/apache-azure-spark-history-server/sparkui-download-event.png)

2. 從 [資料]/[圖表] 索引標籤中按一下 [提供意見反應給我們]。

    ![圖表的意見反應](./media/apache-azure-spark-history-server/sparkui-graph-feedback.png)

3. 提供錯誤的標題和描述，並將 zip 檔拖曳至 [編輯] 欄位中，然後按一下 [提交新問題]。

    ![檔案問題](./media/apache-azure-spark-history-server/sparkui-file-issue.png)


### <a name="3-upgrade-jar-file-for-hotfix-scenario"></a>3.針對 hotfix 案例升級 jar 檔案

如果您想要使用 hotfix 來進行升級，請使用下列指令碼，這將會升級 spark-enhancement.jar*。

**upgrade_spark_enhancement.sh**：

   ```bash
    #!/usr/bin/env bash

    # Copyright (C) Microsoft Corporation. All rights reserved.

    # Arguments:
    # $1 Enhancement jar path

    if [ "$#" -ne 1 ]; then
        >&2 echo "Please provide the upgrade jar path."
        exit 1
    fi

    install_jar() {
        tmp_jar_path="/tmp/spark-enhancement-hotfix-$( date +%s )"

        if wget -O "$tmp_jar_path" "$2"; then
            for FILE in "$1"/spark-enhancement*.jar
            do
                back_up_path="$FILE.original.$( date +%s )"
                echo "Back up $FILE to $back_up_path"
                mv "$FILE" "$back_up_path"
                echo "Copy the hotfix jar file from $tmp_jar_path   to $FILE"
                cp "$tmp_jar_path" "$FILE"

                "Hotfix done."
                break
            done
        else    
            >&2 echo "Download jar file failed."
            exit 1
        fi
    }

    jars_folder="/usr/hdp/current/spark2-client/jars"
    jar_path=$1

    if ls ${jars_folder}/spark-enhancement*.jar 1>/dev/null 2>&1;   then
        install_jar "$jars_folder" "$jar_path"
    else
        >&2 echo "There is no target jar on this node. Exit with no action."
        exit 0
    fi
   ```

**使用量**： 

`upgrade_spark_enhancement.sh https://${jar_path}`

**範例**：

`upgrade_spark_enhancement.sh https://${account_name}.blob.core.windows.net/packages/jars/spark-enhancement-${version}.jar` 

**從 Azure 入口網站中使用 Bash 檔案**

1. 啟動 [Azure 入口網站](https://ms.portal.azure.com)，然後選取您的叢集。
2. 按一下 [指令碼動作]，然後按一下 [提交新項目]。 完成 [提交指令碼動作] 表單，然後按一下 [建立] 按鈕。
    
    + **指令碼類型**：選取 [自訂]。
    + **名稱**：指定指令碼名稱。
    + **Bash 指令碼 URI**：將 Bash 檔案上傳到私人叢集，然後複製此處的 URL。 或者，使用提供的 URI。
    
   ```upgrade_spark_enhancement
    https://hdinsighttoolingstorage.blob.core.windows.net/shsscriptactions/upgrade_spark_enhancement.sh
   ```

    + 核取 [標頭] 和 [背景工作角色]。
    + **參數**：設定 Bash 使用量後方的參數。

    ![上傳記錄或升級 hotfix](./media/apache-azure-spark-history-server/sparkui-upload2.png)


## <a name="known-issues"></a>已知問題

1.  目前，此功能只適用於 Spark 2.3 叢集。

2.  使用 RDD 的輸入/輸出資料不會顯示在 [資料] 索引標籤中。

## <a name="next-steps"></a>後續步驟

* [在 HDInsight 上管理 Spark 叢集的資源](apache-spark-resource-manager.md)
* [設定 Spark 設定](apache-spark-settings.md)


## <a name="contact-us"></a>與我們連絡

如果您有任何意見反應，或在使用此工具時遇到任何其他問題，請傳送電子郵件到 ([hdivstool@microsoft.com](mailto:hdivstool@microsoft.com))。
