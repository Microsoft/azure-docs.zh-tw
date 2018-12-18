---
title: 安裝已發佈的應用程式 - StreamSets Data Collector - Azure HDInsight
description: 安裝及使用 StreamSets Data Collector 第三方 Hadoop 應用程式。
services: hdinsight
author: ashishthaps
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: ashish
ms.openlocfilehash: 72ace99a8124b0a288e8facf630e947151169d0b
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43106877"
---
# <a name="install-published-application---streamsets-data-collector"></a>安裝已發佈的應用程式 - StreamSets Data Collector

本文說明如何在 Azure HDInsight 上安裝及執行 [StreamSets Data Collector for HDInsight](https://streamsets.com/) 已發佈的 Hadoop 應用程式。 如需 HDInsight 應用程式平台的概觀，以及可用獨立軟體廠商 (ISV) 已發佈的應用程式清單，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

## <a name="about-streamsets-data-collector"></a>關於 StreamSets Data Collector

StreamSets Data Collector 可部署在 Azure HDInsight 應用程式之上。 StreamSets Data Collector 提供完整功能的整合式開發環境 (IDE)，可讓您設計、測試、部署及管理「任何對任何」內嵌管線。 這些管線可以讓串流和批次資料進行網狀傳輸，並包含各種串流內轉換，而您卻不必撰寫自訂程式碼。

StreamSets Data Collector 可讓您使用許多巨量資料元件 (例如 HDFS、Kafka、Solr、Hive、HBASE 和 Kudu) 來建置資料流程。 一旦 StreamSets Data Collector 在 Edge Server 或者 Hadoop 叢集上執行，您就能即時監控資料異常和資料流程作業。 此監控包括閾值型警示、異常偵測，以及錯誤記錄的自動補救。

依據設計，StreamSets Data Collector 在邏輯上會隔離管線中的各個階段，讓您不必編碼即可投入新處理器和連接器，且所需停機時間最少，來符合新的商務需求。

### <a name="streamsets-resource-links"></a>StreamSets 資源連結

* [Documentation](https://streamsets.com/documentation/datacollector/latest/help/#Getting_Started/GettingStarted_Title.html)
* [部落格](https://streamsets.com/blog/)
* [教學課程](https://github.com/streamsets/tutorials)
* [開發人員支援論壇](https://groups.google.com/a/streamsets.com/forum/#!forum/sdc-user)
* [Slack 公用 StreamSets 通道](https://streamsetters.slack.com/)
* [原始程式碼](https://github.com/streamsets)

## <a name="prerequisites"></a>必要條件

若要在新的 HDInsight 叢集或現有叢集上安裝此應用程式，您必須具有下列設定：

* 叢集層：標準或進階
* 叢集版本：3.5 和以上版本

## <a name="install-the-streamsets-data-collector-published-application"></a>安裝 StreamSets Data Collector 已發佈的應用程式

如需有關安裝這個 ISV 應用程式和其他可用 ISV 應用程式的逐步指示，請參閱[安裝第三方 Hadoop 應用程式](hdinsight-apps-install-applications.md)。

## <a name="launch-streamsets-data-collector"></a>啟動 StreamSets Data Collector

1. 安裝之後，您可以從 Azure 入口網站中的叢集啟動 StreamSets，方法是前往 [設定] 窗格，然後選取 [一般] 分類底下的 [應用程式]。 [已安裝的應用程式] 窗格會列出已安裝的應用程式。

    ![已安裝的 StreamSets 應用程式](./media/hdinsight-apps-install-streamsets/streamsets.png)

2. 當您選取 StreamSets Data Collector 時，您會看到網頁的連結，以及 SSH 端點路徑。 選取 [網頁] 連結。

3. 在 [登入] 對話方塊中，使用下列認證來登入：`admin` 和 `admin`。

4. 在 [開始使用] 頁面中，按一下 [建立新管線]。

    ![建立新管線](./media/hdinsight-apps-install-streamsets/get-started.png)

5. 在 [新增管線] 視窗中，輸入管線的名稱 ("Hello World")，選擇性地輸入描述，然後選取 [儲存]。

6. Data Collector 主控台隨即顯示。 [屬性] 面板會顯示管線屬性。
 
    ![Data Collector 主控台](./media/hdinsight-apps-install-streamsets/pipeline-canvas.png)

7. 您現在已準備好遵循 [StreamSets 教學課程](https://streamsets.com/documentation/datacollector/latest/help/#Tutorial/Tutorial-title.html)。 本教學課程會提供逐步指示，說明如何建立您的第一個管線。

## <a name="next-steps"></a>後續步驟

* [StreamSets Data Collector 文件](https://streamsets.com/documentation/datacollector/latest/help/#Getting_Started/GettingStarted_Title.html#concept_htw_ghg_jq)。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集，以及測試和裝載 HDInsight 應用程式。
