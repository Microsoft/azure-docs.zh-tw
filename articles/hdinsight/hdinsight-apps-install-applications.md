---
title: 在 Azure HDInsight 上安裝協力廠商 Hadoop 應用程式
description: 了解如何在 Azure HDInsight 上安裝協力廠商 Hadoop 應用程式。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: c50bd7c314c88c0950478cc3068d9a5873b65263
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996957"
---
# <a name="install-third-party-hadoop-applications-on-azure-hdinsight"></a>在 Azure HDInsight 上安裝協力廠商 Hadoop 應用程式

了解如何在 Azure HDInsight 上安裝協力廠商 Hadoop 應用程式。 如需您自己的應用程式的安裝指示，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。

HDInsight 應用程式是使用者可以在 HDInsight 叢集上安裝的應用程式。 Microsoft 獨立軟體廠商 (ISV) 或您可以自己開發這些應用程式。  

下列清單顯示已發佈的應用程式：

* **AtScale 智慧平台**將您的 HDInsight 叢集變成向外延展的 OLAP 伺服器。 此應用程式可讓您使用 Microsoft Excel、PowerBI、Tableau Software 或 QlikView 的 BI 工具，以互動方式查詢無數資料列。
* **適用於 HDInsight 的 Cask CDAP**：提供第一個巨量資料統一整合平台，可減少資料應用程式與 Data Lake 80% 的產出時間。 此應用程式僅支援 Standard HBase 3.4 叢集。
* **HDInsight 上的 DATAIKU DDS**可讓資料專業人員製作原型、建置及部署非常特定的服務，以將未經處理資料轉換成有影響力的商務預測。
* **H2O Artificial Intelligence for HDInsight (Beta)** H2O Sparkling Water 支援下列分散式演算法：GLM、Naïve Bayes、分散式隨機樹系、梯度提升機、深度類神經網路、深度學習、k-means、PCA、Generalized Low Rank Model、異常偵測和 Autoencoders。
* **Kyligence Analytics Platform**：Kyligence Analytics Platform (KAP) 是由 Apache Kylin 和 Apache Hadoop 支援之符合企業需求的資料倉儲，對大規模資料集的查詢延遲不到一秒，而且可簡化企業用戶和分析師的資料分析作業。 
* **Paxata 自助資料準備**
* **Spark Job Server for KNIME Spark Executor**：Spark Job Server for KNIME Spark Executor 可用來將 KNIME Analytics Platform 連線到 HDInsight 叢集。
* **適用於 HDInsight 的 Streamsets 資料收集器**提供功能完整的整合式開發環境 (IDE)，可讓您設計、測試、部署和管理任意至任意內嵌管線 (它會協調資料流與批次資料)，並包括各種資料流內轉換，而完全不必撰寫自訂程式碼。 
* **[Trifacta](http://www.trifacta.com/)** 可讓資料工程師和分析師利用機器學習來提供突破性的使用者體驗、工作流程和架構，更有效率地探索及準備現今多元的資料。
* **WANdisco Fusion HDI 應用程式**可隨環境變更，和資料維持持續一致的連線。 它可讓您隨時隨地存取資料，無停機和中斷的時間。

本文提供的指示將使用 Azure 入口網站。 您也可以從入口網站匯出 Azure Resource Manager 範本或從廠商取得 Resource Manage 範本的複本，然後使用 Azure PowerShell 與 Azure 傳統 CLI 來部署範本。  請參閱[使用 Resource Manager 範本在 HDInsight 中建立 Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)。

## <a name="prerequisites"></a>必要條件
如果您想要在現有的 HDInsight 叢集上安裝 HDInsight 應用程式，您必須有 HDInsight 叢集。 若要建立叢集，請參閱 [建立叢集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。 您也可以在建立 HDInsight 叢集時安裝 HDInsight 應用程式。

## <a name="install-applications-to-existing-clusters"></a>將應用程式安裝到現有的叢集
下列程序示範如何將 HDInsight 應用程式安裝到現有的 HDInsight 叢集。

**安裝 HDInsight 應用程式**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表中的 [HDInsight 叢集]  。
3. 按一下 HDInsight 叢集。  如果您沒有叢集，則必須先建立一個。  請參閱 [建立叢集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。
4. 按一下 [設定] 類別下的 [應用程式]。 您可以會看到已安裝的應用程式清單。 如果您無法找到 [應用程式]，就表示沒有任何應用程式適用於這個版本的 HDInsight 叢集。
   
    ![HDInsight 應用程式入口網站功能表](./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png)
5. 按一下功能表中的 [新增]。 您可以看到現有 HDInsight 應用程式的清單。
   
    ![HDinsight 應用程式可用的應用程式](./media/hdinsight-apps-install-applications/hdinsight-apps-list.png)
6. 按一下其中一個可用的應用程式，並依照指示接受法律條款。

您可以從入口網站通知看到安裝狀態 (按一下入口網站頂端的鈴鐺圖示)。 安裝應用程式之後，應用程式會出現在 [已安裝的應用程式] 清單上。

## <a name="install-applications-during-cluster-creation"></a>在叢集建立期間安裝應用程式
您可以選擇在建立叢集時安裝 HDInsight 應用程式。 在此過程中，HDInsight 應用程式會在叢集建立並處於執行中狀態後安裝。 若要使用 Azure 入口網站在叢集建立期間安裝應用程式，您可以使用「自訂」選項而非預設的「快速建立」選項。

## <a name="list-installed-hdinsight-apps-and-properties"></a>列出已安裝的 HDInsight 應用程式和屬性
入口網站會顯示叢集的已安裝 HDInsight 應用程式清單，以及每個已安裝應用程式的屬性。

**列出 HDInsight 應用程式並顯示屬性**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 按一下左側功能表中的 [HDInsight 叢集]  。 
3. 按一下 HDInsight 叢集。
4. 在 [設定] 中，按一下 [設定] 類別之下的 [應用程式]。 安裝的應用程式會列在右邊。 
   
    ![HDinsight 應用程式安裝的應用程式](./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png)
5. 按一下其中一個已安裝的應用程式，以顯示屬性。 此屬性會列出：
   
   * 應用程式名稱：應用程式名稱。
   * 狀態︰應用程式狀態。 
   * 網頁︰您已部署到邊緣節點之 Web 應用程式的 URL。 此認證與您針對叢集設定的 HTTP 使用者認證相同。
   * HTTP 端點︰此認證與您針對叢集設定的 HTTP 使用者認證相同。 
   * SSH 端點︰您可以使用 SSH 連接到邊緣節點。 SSH 認證與您針對叢集設定的 SSH 使用者認證相同。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。
6. 若要刪除應用程式，請以滑鼠右鍵按一下應用程式，然後按一下操作功能表中的 [刪除]。

## <a name="connect-to-the-edge-node"></a>連接到邊緣節點
您可以使用 HTTP 和 SSH 連接到邊緣節點。 您可以在 [入口網站](#list-installed-hdinsight-apps-and-properties)中找到端點資訊。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

HTTP 端點認證是您已針對 HDInsight 叢集設定的 HTTP 使用者認證；SSH 端點認證就是您已針對 HDInsight 叢集設定的 SSH 認證。

## <a name="troubleshoot"></a>疑難排解
請參閱[針對安裝問題進行疑難排解](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation)。

## <a name="next-steps"></a>後續步驟
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [使用 Resource Manager 範本在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)︰了解如何呼叫 Resource Manager 範本來建立 HDInsight 叢集。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集、測試 HDInsight 應用程式，以及裝載 HDInsight 應用程式。

