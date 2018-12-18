---
title: 搭配 Hortonworks 沙箱使用 Data Lake tools for Visual Studio - Azure HDInsight
description: 了解如何搭配在本機 VM 中執行的 Hortonworks 沙箱使用 Azure Data Lake tools for Visual Studio。 您可以使用這些工具，在沙箱上建立和執行 Hive 和 Pig 作業，以及檢視作業輸出和歷程記錄。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: jasonh
ms.openlocfilehash: c657048d22ca3bee6b41b7351d8fdc5bacdb44be
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106289"
---
# <a name="use-the-azure-data-lake-tools-for-visual-studio-with-the-hortonworks-sandbox"></a>搭配 Hortonworks 沙箱使用 Azure Data Lake tools for Visual Studio

Azure Data Lake 包含使用於一般 Hadoop 叢集的工具。 針對在本機虛擬機器中執行的 Hortonworks 沙箱，本文提供搭配使用 Data Lake 工具所需的步驟。

使用 Hortonworks 沙箱，可讓您在本機開發環境上使用 Hadoop。 開發解決方案之後並想要進行大規模部署時，您可以接著移至 HDInsight 叢集。

## <a name="prerequisites"></a>必要條件

* 在您的開發環境上虛擬機器中執行的 Hortonworks 沙箱。 此文件是使用在 Oracle VirtualBox 上執行的沙箱所撰寫並測試。 若要了解沙箱的設定，請參閱 [Hortonworks 沙箱使用者入門](hadoop/apache-hadoop-emulator-get-started.md) 文件。

* Visual Studio 2013、Visual Studio 2015 或 Visual Studio 2017 (任一版本)。

* [Azure SDK for .NET](https://azure.microsoft.com/downloads/) 2.7.1 或更新版本。

* [Azure Data Lake tools for Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504)。

## <a name="configure-passwords-for-the-sandbox"></a>設定沙箱的密碼

確定 Hortonworks 沙箱正在執行。 依照 [Hortonworks 沙箱使用者入門](hadoop/apache-hadoop-emulator-get-started.md#set-sandbox-passwords)文件中的步驟執行。 這些步驟會設定 SSH `root` 帳戶以及 Ambari `admin` 帳戶的密碼。 從 Visual Studio 連線至沙箱時會使用這些密碼。

## <a name="connect-the-tools-to-the-sandbox"></a>將工具連線至沙箱

1. 開啟 Visual Studio，選取 [檢視]，然後選取 [伺服器總管]。

2. 從 [伺服器總管]，以滑鼠右鍵按一下 [HDInsight] 項目，然後選取 [連線至 HDInsight Emulator]。

    ![[伺服器總管] 的螢幕擷取畫面，其中 [連線到 HDInsight 模擬器] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/connect-emulator.png)

3. 從 [連線到 HDInsight 模擬器] 對話方塊，輸入您為 Ambari 設定的密碼。

    ![對話方塊的螢幕擷取畫面，其中密碼文字方塊已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/enter-ambari-password.png)

    選取 [下一步] 以繼續操作。

4. 使用 [密碼] 欄位來輸入您為 `root` 帳戶所設定的密碼。 讓其他欄位保持預設值。

    ![對話方塊的螢幕擷取畫面，其中密碼文字方塊已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/enter-root-password.png)

    選取 [下一步] 以繼續操作。

5. 等候服務的驗證完成。 在某些情況下，驗證可能會失敗並提示您更新設定。 如果驗證失敗，請選取 [更新] 並等候服務的設定和驗證完成。

    ![對話方塊的螢幕擷取畫面，其中 [更新] 按鈕已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/fail-and-update.png)

    > [!NOTE]
    > 更新程序會使用 Ambari，將 Hortonworks 沙箱設定修改成 Data Lake tools for Visual Studio 所預期的組態。

6. 驗證完成後，請選取 [完成] 以完成設定。
    ![對話方塊的螢幕擷取畫面，其中 [完成] 按鈕已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/finished-connect.png)

     >[!NOTE]
     > 視您的開發環境的速度，以及配置給虛擬機器的記憶體數量而定，可能需要幾分鐘的時間來設定及驗證服務。

完成這些步驟後，您會發現 [伺服器總管] 的 [HDInsight] 區段下出現 [HDInsight 本機叢集] 項目。

## <a name="write-a-hive-query"></a>撰寫 Hive 查詢

Hive 會提供類似 SQL 的查詢語言 (HiveQL)，以便處理結構化資料。 執行下列步驟，了解如何針對本機叢集執行特定查詢。

1. 在 [伺服器總管] 中，以滑鼠右鍵按一下您先前新增的本機叢集項目，然後選取 [撰寫 Hive 查詢]。

    ![[伺服器總管] 的螢幕擷取畫面，其中 [撰寫 Hive 查詢] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/write-hive-query.png)

    新的查詢視窗隨即開啟。 您可以在此視窗迅速寫入查詢並提交到本機叢集。

2. 在新的查詢視窗中，輸入下列命令︰

        select count(*) from sample_08;

    若要執行查詢，請選取視窗頂端的 [提交]。 讓其他值 ([批次] 和伺服器名稱) 保持預設值。

    ![查詢視窗的螢幕擷取畫面，其中 [提交] 按鈕已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/submit-hive.png)

    您也可以使用 [提交] 旁邊的下拉式功能表來選取 [進階]。 進階選項可讓您在提交作業時提供其他選項。

    ![[提交指令碼] 對話方塊的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/advanced-hive.png)

3. 提交查詢後，將會顯示作業狀態。 作業狀態會顯示 Hadoop 處理作業時的相關資訊。 [作業狀態] 提供作業的狀態。 狀態會定期更新，您也可以使用重新整理圖示來手動重新整理狀態。

    ![[作業檢視] 對話方塊的螢幕擷取畫面，其中 [作業狀態] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/job-state.png)

    當 [作業狀態] 變更為 [已完成] 之後，有向非循環圖 (DAG) 隨即顯示。 此圖說明處理 Hive 查詢時由 Tez 所決定的執行路徑。 Tez 是本機叢集上 Hive 的預設執行引擎。

    > [!NOTE]
    > Tez 也是當您使用 Linux 型 HDInsight 叢集時的預設值。 它不是 Windows 型 HDInsight 的預設值。 若要在該處使用它，您必須將 `set hive.execution.engine = tez;` 行新增到 Hive 查詢的開頭。

    使用 [作業輸出] 連結來檢視輸出。 在此案例中，它是 823，亦即 sample_08 資料表中的資料列數目。 您可以使用 [作業記錄] 和 [下載 YARN 記錄] 連結，檢視作業的相關診斷資訊。

4. 您也可以將 [批次] 欄位變更為 [互動式]，以互動方式執行 Hive 作業。 接著，選取 [執行]。

    ![[互動式] 與 [執行] 按鈕已反白顯示的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/interactive-query.png)

    互動式查詢會將處理期間所產生的輸出記錄串流處理至 [HiveServer2 輸出] 視窗。

    > [!NOTE]
    > 此資訊與作業完成後可從 [作業記錄] 連結取得的資訊相同。

    ![輸出記錄的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/hiveserver2-output.png)

## <a name="create-a-hive-project"></a>建立 Hive 專案

您也可以建立包含多個 Hive 指令碼的專案。 若有相關的指令碼或想要儲存指令碼於有版本控制的系統，您可以使用專案。

1. 在 Visual Studio 中，選取 [檔案]、[新增]，然後選取 [專案]。

2. 從專案清單，展開 [範本]，展開 [Azure Data Lake]，然後選取 [HIVE (HDInsight)]。 從範本清單中，選取 [Hive 範例]。 輸入名稱和位置，然後選取 [確定]。

    ![[新增專案] 視窗的螢幕擷取畫面，其中 [Azure Data Lake]、[HIVE]、[Hive 範例] 與 [確定] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/new-hive-project.png)

[Hive 範例] 專案包含兩個指令碼：**WebLogAnalysis.hql** 和 **SensorDataAnalysis.hql**。 您可以使用前述視窗頂端的 [提交] 按鈕提交這些指令碼。

## <a name="create-a-pig-project"></a>建立 Pig 專案

Hive 提供類似 SQL 的語言來處理結構化的資料，Pig 的運作方式是藉由對資料執行轉換。 Pig 提供可讓您開發轉換管線的語言 (Pig Latin)。 若要搭配本機叢集使用 Pig，請依照這些步驟執行：

1. 開啟 Visual Studio，並依序選取 [檔案] > [新增] 和 [專案]。 從專案清單，展開 [範本]、[Azure Data Lake]，然後選取 [Pig (HDInsight)]。 從範本清單中，選取 [Pig 應用程式]。 輸入名稱、位置，然後選取 [確定]。

    ![[新增專案] 視窗的螢幕擷取畫面，其中 [Azure Data Lake]、[Pig]、[Pig 應用程式] 與 [確定] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/new-pig.png)

2. 輸入下列文字作為與此專案一起建立之檔案 **script.pig** 的內容。

        a = LOAD '/demo/data/Website/Website-Logs' AS (
            log_id:int,
            ip_address:chararray,
            date:chararray,
            time:chararray,
            landing_page:chararray,
            source:chararray);
        b = FILTER a BY (log_id > 100);
        c = GROUP b BY ip_address;
        DUMP c;

    雖然 Pig 使用與 Hive 不同的語言，但您透過 [提交] 按鈕在這兩種語言之間執行作業的方式一致。 選取 [提交] 旁邊的下拉式清單，即可顯示 Pig 的進階提交對話方塊。

    ![[提交指令碼] 對話方塊的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/advanced-pig.png)

3. 如同 Hive 查詢，也會顯示作業狀態和輸出。

    ![已完成 Pig 作業的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/completed-pig.png)

## <a name="view-jobs"></a>檢視作業

Data Lake 工具也可讓您輕鬆地檢視已在 Hadoop 上執行之作業的相關資訊。 使用下列步驟來查看已在本機叢集上執行的作業。

1. 從 [伺服器總管]，在本機叢集上按一下滑鼠右鍵，然後選取 [檢視作業]。 已提交至叢集的作業清單隨即顯示。

    ![[伺服器總管] 的螢幕擷取畫面，其中 [檢視作業] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/view-jobs.png)

2. 選取作業清單中的作業，以檢視作業詳細資料。

    ![[作業瀏覽器] 的螢幕擷取畫面，其中一個作業已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/view-job-details.png)

    顯示的資訊與您在執行 Pig 或 Hive 查詢後看到的資訊類似，且包括用於檢視輸出和記錄資訊的連結。

3. 您也可以從這裡修改和重新提交作業。

## <a name="view-hive-databases"></a>檢視 Hive 資料庫

1. 在 [伺服器總管] 中，展開 [HDInsight 本機叢集] 項目，然後展開 [Hive 資料庫]。 本機叢集上的 [預設] 和 [xademo] 資料庫隨即顯示。 展開資料庫即可顯示資料庫內的資料表。

    ![[伺服器總管] 的螢幕擷取畫面，其中資料庫已展開](./media/hdinsight-hadoop-emulator-visual-studio/expanded-databases.png)

2. 展開資料表即可顯示該資料表的資料行。 若要快速檢視資料，請以滑鼠右鍵按一下資料表，然後選取 [檢視前 100 個資料列]。

    ![[伺服器總管] 的螢幕擷取畫面，其中資料表已展開且 [檢視前 100 個資料列] 已選取](./media/hdinsight-hadoop-emulator-visual-studio/view-100.png)

### <a name="database-and-table-properties"></a>資料庫和資料表屬性

您可以檢視資料庫或資料表的屬性。 選取 [屬性] 會在 [屬性] 視窗中顯示所選取項目的詳細資料。 請參考下列螢幕擷取畫面中顯示的資訊為例：

![[屬性] 視窗的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/properties.png)

### <a name="create-a-table"></a>建立資料表

若要建立資料表，請以滑鼠右鍵按一下資料庫，然後選取 [建立資料表]。

![[伺服器總管] 的螢幕擷取畫面，其中 [建立資料表] 已反白顯示](./media/hdinsight-hadoop-emulator-visual-studio/create-table.png)

接著，您可以使用表單建立資料表。 在下列螢幕擷取畫面底部，您可以看到用於建立資料表的原始 HiveQL。

![用於建立資料表之表單的螢幕擷取畫面](./media/hdinsight-hadoop-emulator-visual-studio/create-table-form.png)

## <a name="next-steps"></a>後續步驟

* [了解 Hortonworks 沙箱的訣竅](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) \(英文\)
* [Hadoop 教學課程 - 開始使用 HDP](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/)
