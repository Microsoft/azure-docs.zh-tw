---
title: 安裝已發佈的應用程式 - Datameer - Azure HDInsight
description: 安裝及使用 Datameer 第三方 Hadoop 應用程式。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: ashish
ms.openlocfilehash: a428bb7bc9cc6a6a2e28989271ad1998700438cf
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43107743"
---
# <a name="install-published-application---datameer"></a>安裝已發佈的應用程式 - Datameer

本文說明如何在 Azure HDInsight 上安裝及執行 [Datameer](https://www.datameer.com/) 已發佈的 Hadoop 應用程式。 如需 HDInsight 應用程式平台的概觀，以及可用獨立軟體廠商 (ISV) 已發佈的應用程式清單，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

## <a name="about-datameer"></a>關於 Datameer

Datameer 是 Hadoop 平台的原生應用程式，可擴充現有的 Azure HDInsight 功能，並可快速整合、準備及分析結構化和非結構化的資料。 Datameer 可以存取超過 70 種來源和格式：結構化、半結構化和非結構化。 您可以直接上傳資料，也可以使用其獨特的資料連結來隨需提取資料。 Datameer 的自助式功能和熟悉的試算表介面，可降低巨量資料技術的複雜性並加快深入解析的時間。 此試算表介面可提供簡易機制，讓您輸入宣告式公式，然後轉換成最佳化的 Hadoop 作業。 運用 Datameer 和商業智慧 (BI) 與 Excel 技巧，您即可快速地在雲端中使用 Hadoop。 如需詳細資訊，請參閱 [Datameer 文件](http://www.datameer.com/documentation/display/DAS50/Home?ls=Partners&lsd=Microsoft&c=Partners&cd=Microsoft)。

## <a name="prerequisites"></a>必要條件

若要在新的 HDInsight 叢集或現有叢集上安裝此應用程式，您必須具有下列設定：

* 叢集層：標準
* 叢集類型：Hadoop
* 叢集版本：3.4

## <a name="install-the-datameer-published-application"></a>安裝 Datameer 已發佈的應用程式

如需有關安裝這個 ISV 應用程式和其他可用 ISV 應用程式的逐步指示，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。

## <a name="launch-datameer"></a>啟動 Datameer

1. 安裝之後，您可以從 Azure 入口網站中的叢集啟動 Datameer，方法是前往 [設定] 窗格，然後按一下 [一般] 分類底下的 [應用程式]。 [已安裝的應用程式] 窗格會列出已安裝的應用程式。

    ![已安裝的 Datameer 應用程式](./media/hdinsight-apps-install-datameer/datameer-app.png)

2. 當您選取 [Datameer] 時，您會看到網頁連結和 SSH 端點路徑。 選取 [網頁] 連結。

3. 第一次啟動時有兩個授權選項：14 天免費試用，或啟動現有授權。

    ![授權選項](./media/hdinsight-apps-install-datameer/license.png)

4. 完成所選取的授權選項之後，您會看到一個登入表單。 輸入在登入表單之前顯示的預設認證。 登入之後，接受軟體合約以繼續。

    ![登入](./media/hdinsight-apps-install-datameer/login.png)

下列步驟顯示 "Hello World" 示範。

1. [下載 CSV 範例](https://datameer.box.com/s/wzzw27za3agic4yjj8zrn6vfrph0ppnf)。

2. 按一下 Datameer 儀表板頂端的 [+] 符號，然後按一下 [檔案上傳]。

    ![檔案上傳](./media/hdinsight-apps-install-datameer/upload.png)

3. 在 [上傳] 對話方塊中，瀏覽並選取您已下載的 **Hello World.csv** 檔案。 確定 [檔案類型] 設定為 [CSV/TSV]。 選取 [下一步] 。 持續按 [下一步] 直到精靈結束。

    ![檔案上傳](./media/hdinsight-apps-install-datameer/upload-browse.png)

4. 將新資料夾下的檔案命名為 **Hello World**。 將新資料夾重新命名為 "Demo"。 選取 [ **儲存**]。

    ![儲存](./media/hdinsight-apps-install-datameer/save.png)

5. 再次按一下 [+] 符號，然後選取 [活頁簿] 為資料建立新活頁簿。

    ![新增活頁簿](./media/hdinsight-apps-install-datameer/add-workbook.png)

6. 依序展開 **Data** 資料夾、**FileUploads**，然後展開您在儲存 "Hello World" 檔案時建立的 **Demo** 資料夾。 從檔案清單選取 **Hello World** 表單，然後按一下 [新增資料]。

    ![儲存](./media/hdinsight-apps-install-datameer/select-file.png)

7. 您會看到資料載入到試算表介面。 若要選取資料的子集，請選取工具列中的 [篩選] 按鈕。

    ![[篩選] 按鈕](./media/hdinsight-apps-install-datameer/filter-button.png)

8. 在 [套用篩選] 對話方塊中，選取 [城市] 資料行、[等於] 運算子，然後在篩選文字方塊中輸入**芝加哥**。 勾選 [在新工作表中建立篩選] 核取方塊，然後選取 [建立篩選]。

    ![套用篩選](./media/hdinsight-apps-install-datameer/apply-filter.png)

9. 依序按一下 [檔案] 和 [儲存] 以儲存活頁簿。 提供名稱，例如 "Hello World Workbook"。

10. 您會看到活頁簿執行方式和時間的選項。 目前先讓所有選項保持使用其預設值，然後勾選 [儲存後立即啟動計算程序]，並選取 [儲存]。

    ![儲存活頁簿](./media/hdinsight-apps-install-datameer/save-workbook.png)

11. Datameer 提供了功能強大的視覺效果工具。 若要顯示資料，請建立資訊圖。 選取儀表板頂端的 [+] 符號，然後選取 [資訊圖]。

    ![新增資訊圖](./media/hdinsight-apps-install-datameer/infographic-button.png)

12. 從左側的小工具清單拖曳「長條圖」小工具，如下圖中的步驟 1 所示。 接下來，瀏覽右邊資料瀏覽器下的 [Data] 資料夾，展開您的活頁簿，然後展開您使用篩選所新增的工作表 (步驟 2)。 將 [名稱] 資料行拖曳到橫條圖上，並放置到 [標籤] 目標，將活頁簿的 [名稱] 資料行設定為圖表的 [標籤] 欄位 (步驟 3)。

    ![資訊圖](./media/hdinsight-apps-install-datameer/infographic.png)

13. 若要將「年齡」設為圖表的 Y 軸，請將 [年齡] 資料行拖曳到圖表的 [資料] 欄位。

    ![資訊圖](./media/hdinsight-apps-install-datameer/infographic-age.png)

恭喜！ 您已在未撰寫任何程式碼的情況下建立資料的視覺效果。 您現在可以探索各種變化和其他視覺效果。

## <a name="next-steps"></a>後續步驟

* [Datameer 文件](http://www.datameer.com/documentation/display/DAS50/Home?ls=Partners&lsd=Microsoft&c=Partners&cd=Microsoft)。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集，以及測試和裝載 HDInsight 應用程式。
