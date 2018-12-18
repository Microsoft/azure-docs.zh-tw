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
ms.openlocfilehash: 377dbadd7b696e62d8464258d22d0dd5ed926208
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106098"
---
# <a name="install-published-application---cask-data-application-platform-cdap"></a>安裝已發佈的應用程式 - Cask 資料應用程式平台 (CDAP)

本文說明如何在 Azure HDInsight 上安裝及執行 [CDAP](http://cask.co/products/cdap/) 已發佈的 Hadoop 應用程式。 如需 HDInsight 應用程式平台的概觀，以及可用獨立軟體廠商 (ISV) 已發佈的應用程式清單，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

## <a name="about-cdap"></a>關於 CDAP

在 Hadoop 中開發應用程式並不容易。  Hadoop 技術延伸模組的數目相當龐大且日益增加，需要一些時間才能整合。 監視資料流程與收集計量需要建置個別解決方案。

### <a name="how-does-cdap-help"></a>CDAP 有何助益？

Cask 資料應用程式平台 (CDAP) 是巨量資料的整合平台。 CDAP 可讓您將重點放在建置應用程式而非基礎結構。

CDAP 使用開發人員熟悉的高階概念和抽象概念。 這些抽象概可隱藏內部系統的複雜性，並促進重複使用解決方案。

名為 [Cask Hydrator](http://cask.co/products/hydrator/) 的 CDAP 延伸模組提供了使用者介面供您開發和管理資料管線。 資料管線是由各種 *外掛程式組成，這些外掛程式會執行資料擷取、轉換、分析和執行後作業等工作。

每個 CDAP 外掛程式都具有定義完善的介面，評估不同的技術時只要使用另一個外掛程式來取代原本的外掛程式，而不需要動到應用程式的其餘部分。

CDAP 管線可提供應用程式中資料的高階圖示流程。 此視覺效果會顯示從擷取經由轉換資料與分析，最後到外部資料存放區的資料端對端流程。

下列資料管線範例會即時內嵌 Twitter 資料，然後根據預先定義的準則篩選出某些推文。 資料管線會轉換未經處理的推文資料，並且將該資料投影至更容易閱讀的格式，然後根據一組值將推文分組，並將結果寫入至 HBase 存放區。

![CDAP 管線](./media/hdinsight-apps-install-cask/pipeline.png)

此端對端管線是使用 **Cask Hydrator UI** 進行建置的，可使用其外掛程式介面和拖放功能以在每個階段之間建立連線。 您可以個別地隔離及修改每個外掛程式的功能。 使用 CDAP，即可在數小時內建置及驗證類似管線。 在典型的 Hadoop 世界中，建構這類解決方案可能需要好幾天的時間。

CDAP 也會提供名為 [Cask Tracker](http://cask.co/products/tracker/) 的延伸模組，可在資料流經應用程式時以視覺化方式進行追蹤。 Cask Tracker 會將「資料控管」新增至系統，讓整個應用程式的資料資產正式受到管理。 您可以追蹤每個資料點的歷程、收集相關計量，以及稽核整個程序的資料軌跡。

以下是資料如何在上述管線中流動的圖例：

![CDAP Tracker](./media/hdinsight-apps-install-cask/tracker.png)

## <a name="prerequisites"></a>必要條件

若要在新的 HDInsight 叢集或現有叢集上安裝此應用程式，您必須具有下列設定：

* 叢集層：標準
* 叢集類型：HBase
* 叢集版本：3.4、3.5

## <a name="install-the-cdap-published-application"></a>安裝 CDAP 已發佈的應用程式

如需有關安裝這個 ISV 應用程式和其他可用 ISV 應用程式的逐步指示，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。

## <a name="launch-cdap"></a>啟動 CDAP

1. 安裝之後，從 Azure 入口網站中的叢集啟動 CDAP，方法是前往 [設定] 窗格，然後選取 [一般] 分類底下的 [應用程式]。 [已安裝的應用程式] 窗格會列出所有已安裝的應用程式。

    ![已安裝的 CDAP 應用程式](./media/hdinsight-apps-install-cask/cdap-app.png)

2. 當您選取 CDAP 時，您會看到網頁連結和 HTTP 端點，以及 SSH 端點路徑。 選取 [網頁] 連結。

3. 出現提示時，輸入您的叢集系統管理員認證。

    ![驗證](./media/hdinsight-apps-install-cask/auth.png)

4. 登入之後，您會看到 Cask CDAP GUI 首頁。

    ![Cask GUI 首頁](./media/hdinsight-apps-install-cask/gui.png)

5. 若要探索 CDAP 介面，請按一下網頁頂端的 [Cask Market] 功能表連結。

    ![Cask Market 連結](./media/hdinsight-apps-install-cask/cask-market.png)

6. 從清單中選取 [存取記錄範例]。

    ![存取記錄範例](./media/hdinsight-apps-install-cask/market-log-sample.png)

7. 按一下 [載入] 以確認。

    ![按一下 [載入]](./media/hdinsight-apps-install-cask/market-load.png)

8. 隨附的資料範例檢視隨即顯示。 選取 [下一步]。

    ![存取記錄範例 - 檢視資料](./media/hdinsight-apps-install-cask/market-view-data.png)

9. 選取 [串流] 作為 [目的地類型]，輸入 [目的地名稱]，然後選取 [完成]。

    ![存取記錄範例 - 選取目的地](./media/hdinsight-apps-install-cask/market-destination.png)

10. 成功載入資料包 (datapack) 之後，請選取 [檢視串流詳細資料]。

    ![已成功上傳資料包 (datapack)](./media/hdinsight-apps-install-cask/market-view-details.png)

11. 若要啟用命名空間的中繼資料，請在 [存取記錄] 詳細資料頁面上的 [使用方式] 索引標籤中選取 [啟用]。

    ![存取記錄範例 - 已載入 - 啟用中繼資料](./media/hdinsight-apps-install-cask/log-loaded.png)

12. 啟用中繼資料之後，您會看到圖表顯示稽核訊息資訊。

    ![存取記錄範例 - 中繼資料已啟用](./media/hdinsight-apps-install-cask/log-metadata.png)

13. 若要探索記錄資料，請選取網頁頂端的 [探索] 圖示。

    ![存取記錄範例 - 探索](./media/hdinsight-apps-install-cask/log-explore.png)

14. 您會看到 SQL 查詢範例。 放心地依照需要修改，然後選取 [執行]。

    ![存取記錄範例 - 使用查詢探索資料集](./media/hdinsight-apps-install-cask/log-query.png)

15. 查詢完成之後，請選取 [動作] 資料行下方的 [檢視] 圖示。

    ![存取記錄範例 - 檢視已完成的查詢](./media/hdinsight-apps-install-cask/log-query-view.png)

16. 您會看到查詢結果。

    ![存取記錄範例 - 查詢結果](./media/hdinsight-apps-install-cask/log-query-results.png)

## <a name="next-steps"></a>後續步驟

* [Cask 文件](http://cask.co/resources/documentation/)。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集，以及測試和裝載 HDInsight 應用程式。
