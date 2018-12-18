---
title: 安裝已發佈的應用程式 - Dataiku DDS - Azure HDInsight
description: 安裝及使用 Dataiku DDS 協力廠商 Hadoop 應用程式。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: ashish
ms.openlocfilehash: 64a6f393498ca90675712747afc8f9befc4b932f
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105164"
---
# <a name="install-published-application---dataiku-dds"></a>安裝已發佈的應用程式 - Dataiku DDS

本文說明如何在 Azure HDInsight 上安裝及執行 [Dataiku DDS](https://www.dataiku.com/) \(英文\) 已發佈的 Hadoop 應用程式。 如需 HDInsight 應用程式平台的概觀，以及可用獨立軟體廠商 (ISV) 已發佈的應用程式清單，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

## <a name="about-dataiku-dss"></a>關於 Dataiku DSS

Dataiku [Data Science Studio (DSS)](https://www.dataiku.com/dss/features/connectivity/) \(英文\) 是一個共同作業資料科學平台，可讓資料科學家建置並提供分析解決方案。 提供 DSS 作為 HDInsight 應用程式可讓您使用資料科學來建置巨量資料解決方案，並以企業等級和規模來執行這些解決方案。

您可以使用 DSS，從資料的擷取、準備及處理開始，實作完整的分析解決方案。 DSS 解決方案也可以包含訓練和套用機器學習模型、視覺效果，然後使其能夠實際運作。

您可以使用 Hadoop 或 Spark 叢集來安裝 HDInsight 上的 DSS。 您可以在現有的執行中叢集上安裝 DSS，或在建立新叢集時安裝 DSS。 DSS 也支援使用 Azure Blob 儲存體作為讀取資料的連接器。

您可以使用 DSS 來建置專案，然後這些專案將可以產生 MapReduce 或 Spark 作業。 這些作業會在 HDInsight 上以一般 MapReduce 或 Spark 作業的形式執行，因此您可以視需要調整叢集規模。

## <a name="prerequisites"></a>必要條件

若要在新的 HDInsight 叢集或現有叢集上安裝此應用程式，您必須具有下列設定：

* 叢集層：標準、進階
* 叢集類型：Hadoop、Spark
* 叢集版本：3.4、3.5

## <a name="install-the-dataiku-dss-published-application"></a>安裝 Dataiku DSS 已發佈的應用程式

如需有關安裝這個 ISV 應用程式和其他可用 ISV 應用程式的逐步指示，請參閱[安裝協力廠商 Hadoop 應用程式](hdinsight-apps-install-applications.md)。

## <a name="launch-dataiku-dss"></a>啟動 Dataiku DSS

1. 安裝之後，您可以從 Azure 入口網站中的叢集啟動 DSS，方法是前往 [設定] 窗格，然後按一下 [一般] 類別底下的 [應用程式]。 [已安裝的應用程式] 窗格會列出已安裝的應用程式。

    ![已安裝的 Dataiku DSS 應用程式](./media/hdinsight-apps-install-dataiku/app.png)

2. 當您選取 [DSS on HDInsight] \(HDInsight 上的 DSS\) 時，會看到網頁連結和 SSH 端點路徑。 選取 [WEBPAGE] \(網頁\) 連結。

3. 第一次啟動時，您會看到一個可免費建立新 Dataiku 帳戶或登入現有帳戶的表單。 您也可以選擇啟動一個可免費試用 2 週的 [Enterprise 版](https://www.dataiku.com/dss/editions/) \(英文\)。 從這裡，您可以選擇繼續輸入 Enterprise 版的授權金鑰，或使用 Community 版。

4. 完成所選取的授權選項之後，您會看到一個登入表單。 輸入在登入表單之前顯示的預設認證。

下列步驟提供簡單的示範。

1. [下載範例訂單 CSV](https://doc.dataiku.com/tutorials/data/101/haiku_shirt_sales.csv)。

2. 從 DSS 儀表板中，選取左側功能表上的 [+] (新增專案) 連結來建立新專案。

    ![新增專案連結](./media/hdinsight-apps-install-dataiku/new-project.png)

3. 在 [New project] \(新增專案\) 表單中，輸入 [Name] \(名稱\)。 [Project Key] \(專案金鑰\) 中會自動填入建議值。 在此案例中，請輸入 "Orders"。 按一下 [建立]。

    ![新增專案表單](./media/hdinsight-apps-install-dataiku/new-project-form.png)

4. 在新增專案頁面上，選取 [+ IMPORT YOUR FIRST DATASET] \(+ 匯入您的第一個資料集\)。

    ![檔案上傳](./media/hdinsight-apps-install-dataiku/import-dataset.png)

5. 選取 [Files] \(檔案\) 資料集清單底下的 [Upload your files] \(上傳您的檔案\)。 您會看到 [Upload] \(上傳\) 對話方塊。 按一下 [Add a file] \(新增檔案\)，選取您所下載的 `haiku_shirt_sales.csv` 檔案並驗證。

6. 檔案會上傳到 DSS。 請按一下 [Preview] \(預覽\) 按鈕來檢查 DSS 是否正確偵測到 CSV 格式。

    ![檔案上傳](./media/hdinsight-apps-install-dataiku/preview.png)

7. 此匯入作業幾乎完美。 CSV 檔案使用 Tab 分隔符號。 您可以看到資料採用表格式格式，有名為功能的資料行和代表觀測值的行。 其中有一個錯誤，就是顯然檔案的標頭和資料之間包含一列空白行。 若要修正此錯誤，請在 [Skip next lines] \(略過接下來的行\) 欄位中輸入 `1`。

    ![儲存](./media/hdinsight-apps-install-dataiku/skip-lines.png)

8. 為新資料集提供一個名稱。 在畫面頂端的欄位中輸入 **haiku_shirt_sales**，然後選取 [Create] \(建立\) 按鈕。

9. 您會看到一個可供您開始進行探索的表格式資料檢視。 針對每一欄，您應該會看到 Dataiku Science Studio 已偵測到資料類型 (以「藍色」表示) - 在此案例中為「文字」、「數字」或「日期 (未剖析)」。 量測計會指出資料行值似乎與類型不符 (以紅色表示) 或遺失 (以黑色表示) 的比例。 在此範例資料集中，department 同時具有空值和無效的資料。

    ![表格式檢視](./media/hdinsight-apps-install-dataiku/viewing-dataset.png)

## <a name="data-manipulation"></a>資料操作

真實世界的資料幾乎總是雜亂無章，而很少會以井然有序的方式封裝。 清理資料通常需要一系列的指令碼和關聯的商務邏輯。 Dataiku DSS 提供一個專用的**實驗室**工具，可讓使用者更容易進行此工作。

1. 按一下右上角的 [Lab] \(實驗室\)。

    ![[Lab] \(實驗室\) 按鈕](./media/hdinsight-apps-install-dataiku/lab-button.png)

2. [Lab] \(實驗室\) 視窗隨即開啟。 此實驗室是您反覆處理資料集來進一步深入了解資料集的地方。 本教學課程示範的是 Visual analysis (視覺分析) 層面。 選取 [Visual analysis] \(視覺分析\) 底下的 [New] \(新增\) 按鈕。 系統會提示您指定分析名稱。 目前保留預設名稱，然後按一下 [CREATE] \(建立\)。

    ![建立實驗室](./media/hdinsight-apps-install-dataiku/create-lab.png)

3. 選取頁面右上角的 [Quick columns stats] \(快速資料行統計資料\) 按鈕。

    ![Quick columns stats (快速資料行統計資料)](./media/hdinsight-apps-install-dataiku/quick-column-stats.png)

4. 您會看到資料類型和值的統計資料顯示在 [Columns quick view] \(資料行快速檢視\) 窗格底下的時間軸型圖表中。

    ![Columns quick view (資料行快速檢視)](./media/hdinsight-apps-install-dataiku/columns-quick-view.png)

您現在可以使用範例資料來探索 DSS。 您可以清理和使用資料，然後建立新的視覺效果。

如需深入的教學課程，請參閱[了解 Dataiku DSS](https://www.dataiku.com/learn/) \(英文\)。

## <a name="next-steps"></a>後續步驟

* [Dataiku DSS 文件](https://doc.dataiku.com/dss/latest/) \(英文\)。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集，以及測試和裝載 HDInsight 應用程式。
