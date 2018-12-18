---
title: 存取 Linux 型 HDInsight 上的 Hadoop YARN 應用程式記錄 - Azure
description: 了解如何使用命令列和網頁瀏覽器存取以 Linux 為基礎之 HDInsight (Hadoop) 叢集上的 YARN 應用程式記錄檔。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 03/22/2018
ms.author: jasonh
ms.openlocfilehash: 179349d059fd75e2da01eb908a786e2e7ac91307
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43092255"
---
# <a name="access-yarn-application-logs-on-linux-based-hdinsight"></a>存取以 Linux 為基礎之 HDInsight 上的 YARN 應用程式記錄

了解如何存取 Azure HDInsight 的 Hadoop 叢集上之 YARN (Yet Another Resource Negotiator) 應用程式的記錄。

> [!IMPORTANT]
> 此文件中的步驟需要使用 Linux 的 HDInsight 叢集。 Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [HDInsight 元件版本設定](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

## <a name="YARNTimelineServer"></a>YARN Timeline Server

[YARN Timeline Server (英文)](http://hadoop.apache.org/docs/r2.7.3/hadoop-yarn/hadoop-yarn-site/TimelineServer.html) 透過兩個不同的介面，提供完整應用程式的一般資訊，以及架構的特定應用程式資訊。 具體而言：

* 儲存及擷取 HDInsight 叢集上泛型應用程式資訊的功能已在版本 3.1.1.374 或更新版本上啟用。
* Timeline Server 的架構特定應用程式資訊元件目前在 HDInsight 叢集上並未提供。

應用程式的一般資訊包括下列類型的資料：

* 應用程式識別碼，應用程式的唯一識別碼
* 啟動應用程式的使用者
* 為完成應用程式而進行之嘗試的相關資訊
* 任何指定之應用程式嘗試所使用的容器

## <a name="YARNAppsAndLogs"></a>YARN 應用程式和記錄檔

YARN 藉由將資源管理從應用程式排程/監視分離，支援多種程式設計模型 (MapReduce 為其中之一)。 YARN 使用全域 *ResourceManager* (RM)、每一背景工作節點 *ResourceManager* (NM) 及每一應用程式 *ResourceManager* (AM)。 每一應用程式 AM 會與 RM 交涉用來執行您應用程式的資源 (CPU、記憶體、磁碟、網路)。 RM 會與 NM 合作來授與這些資源 (以「 *容器*」的形式授與)。 AM 則是負責追蹤 RM 指派給它之容器的進度。 視應用程式的本質而定，一個應用程式可能會需要許多容器。

每個應用程式都可能包含多個「應用程式嘗試」。 如果應用程式失敗，系統可能會以新的嘗試來重試它。 每個嘗試都在容器中執行。 從某方面來說，容器提供由 YARN 應用程式所執行之作業的基本單位內容。 所有於容器內容之內所執行的作業，都會在容器所配置的單一背景工作節點上執行。 請參閱 [YARN 概念][YARN-concepts]，以取得進一步的參考資料。

應用程式記錄檔 (和關聯的容器記錄檔) 在對有問題的 Hadoop 應用程式進行偵錯上相當重要。 YARN 以 [記錄檔彙總][log-aggregation] 功能提供一個良好的架構，以收集、彙總及儲存應用程式記錄檔。 「記錄檔彙總」功能可使存取應用程式記錄更具確定性。 它能彙總背景工作節點上所有容器的記錄檔，並將它們依每個背景工作節點儲存成單一彙總記錄檔。 在應用程式完成之後，記錄檔會儲存在預設檔案系統上。 您的應用程式可能使用數百或數千個容器，但在單一背景工作節點上執行之所有容器的記錄一律彙總成單一檔案。 因此，應用程式使用的每一背景工作節點只會有 1 個記錄。 根據預設，HDInsight 叢集 3.0 版和更新版本上會啟用記錄彙總。 彙總記錄位於叢集的預設儲存體。 下列路徑是記錄的 HDFS 路徑︰

    /app-logs/<user>/logs/<applicationId>

在路徑中，`user` 是啟動應用程式的使用者名稱。 `applicationId` 是 YARN RM 指派給應用程式的唯一識別碼

您無法直接閱讀彙總的記錄檔，因為它們是以 [TFile][T-file]、由容器編製索引的 [二進位格式][binary-format] 撰寫。 對於您感興趣的應用程式或容器，請使用 YARN ResourceManager 記錄或 CLI 工具來檢視純文字記錄。

## <a name="yarn-cli-tools"></a>YARN CLI 工具

若要使用 YARN CLI 工具，您必須先使用 SSH 連接到 HDInsight 叢集。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

您可以執行下列其中一個命令，以檢視這些純文字記錄：

    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application>
    yarn logs -applicationId <applicationId> -appOwner <user-who-started-the-application> -containerId <containerId> -nodeAddress <worker-node-address>

執行這些命令時，請指定 &lt;applicationId>、&lt;user-who-started-the-application>、&lt;containerId> 及 &lt;worker-node-address> 資訊。

## <a name="yarn-resourcemanager-ui"></a>YARN ResourceManager UI

YARN ResourceManager UI 是在叢集前端節點上執行。 可透過 Ambari Web UI 存取。 請使用下列步驟來檢視 YARN 記錄：

1. 在您的網頁瀏覽器中，瀏覽至 https://CLUSTERNAME.azurehdinsight.net。 將 CLUSTERNAME 取代為 HDInsight 叢集的名稱。
2. 從左邊的服務清單中選取 [YARN] 。

    ![選取的 Yarn 服務](./media/hdinsight-hadoop-access-yarn-app-logs-linux/yarnservice.png)
3. 在 [快速連結] 下拉式清單中，選取其中一個叢集前端節點，然後選取 [ResourceManager 記錄]。

    ![Yarn 快速連結](./media/hdinsight-hadoop-access-yarn-app-logs-linux/yarnquicklinks.png)

    您會看到一份 YARN 記錄檔的連結清單。

[YARN-timeline-server]:http://hadoop.apache.org/docs/r2.4.0/hadoop-yarn/hadoop-yarn-site/TimelineServer.html
[log-aggregation]:http://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/
[T-file]:https://issues.apache.org/jira/secure/attachment/12396286/TFile%20Specification%2020081217.pdf
[binary-format]:https://issues.apache.org/jira/browse/HADOOP-3315
[YARN-concepts]:http://hortonworks.com/blog/apache-hadoop-yarn-concepts-and-applications/
