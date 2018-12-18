---
title: 在 Azure HDInsight 中使用安全傳輸儲存體帳戶建立 Hadoop 叢集
description: 了解如何使用已啟用安全傳輸的 Azure 儲存體帳戶建立 HDInsight 叢集。
services: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 07/24/2018
ms.openlocfilehash: 253455d892d38a6e8356b775306b238ebd566da4
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43104495"
---
# <a name="create-hadoop-cluster-with-secure-transfer-storage-accounts-in-azure-hdinsight"></a>在 Azure HDInsight 中使用安全傳輸儲存體帳戶建立 Hadoop 叢集

[需要安全傳輸](../storage/common/storage-require-secure-transfer.md)功能透過安全連線來強制對您帳戶的所有要求，以增強 Azure 儲存體帳戶的安全性。 只有 HDInsight 叢集 3.6 版或更新版本支援這項功能和 wasbs 配置。 

## <a name="prerequisites"></a>必要條件
開始進行本教學課程之前，您必須具備：

* **Azure 訂用帳戶**︰若要建立一個月的免費試用帳戶，請瀏覽至 [azure.microsoft.com/free](https://azure.microsoft.com/free)。
* **已啟用安全傳輸的 Azure 儲存體帳戶**。 如需相關指示，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)和[需要安全傳輸](../storage/common/storage-require-secure-transfer.md)。
* **儲存體帳戶上的 Blob 容器**。 

## <a name="create-cluster"></a>建立叢集

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]


在本節中，您會在 HDInsight 中使用 [Azure Resource Manager 範本](../azure-resource-manager/resource-group-template-deploy.md)建立 Hadoop 叢集。 這個範本位於 [Github](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-with-existing-default-storage-account/) 中。 進行本教學課程並不需要具備 Resource Manager 範本經驗。 如需其他叢集建立方法及了解本教學課程中使用的屬性，請參閱 [建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

1. 按一下以下影像，在 Azure 入口網站中登入 Azure 並開啟 Resource Manager 範本。 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-existing-default-storage-account%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-hadoop-linux-tutorial-get-started/deploy-to-azure.png" alt="Deploy to Azure"></a>

2. 請依照指示建立具有下列規格的叢集： 

    - 指定 HDInsight 3.6 版。  需要 3.6 版或更新版本。
    - 指定已啟用安全傳輸的儲存體帳戶。
    - 使用儲存體帳戶的簡短名稱。
    - 必須事先建立儲存體帳戶和 Blob 容器。 

    如需相關指示，請參閱[建立叢集](hadoop/apache-hadoop-linux-tutorial-get-started.md#create-cluster)。 

如果您使用指令碼動作來提供自己的組態檔，您必須在下列設定中使用 wasbs：

- fs.defaultFS (core-site)
- spark.eventLog.dir 
- spark.history.fs.logDirectory

## <a name="add-additional-storage-accounts"></a>新增其他儲存體帳戶

有數個選項可新增其他已啟用安全傳輸的儲存體帳戶：

- 在最後一節中修改 Azure Resource Manager 範本。
- 使用 [Azure 入口網站](https://portal.azure.com)建立叢集並指定連結的儲存體帳戶。
- 使用指令碼動作，將其他已啟用安全傳輸的儲存體帳戶新增至現有的 HDInsight 叢集。  如需詳細資訊，請參閱[將其他儲存體帳戶新增至 HDInsight](hdinsight-hadoop-add-storage.md)。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何建立 HDInsight 叢集，並啟用儲存體帳戶的安全傳輸。

若要深入了解如何使用 HDInsight 分析資料，請參閱下列文章：

* 若要深入了解如何搭配 HDInsight 使用 Hive，包括如何從 Visual Studio 執行 Hive 查詢，請參閱[搭配 HDInsight 使用 Hive][hdinsight-use-hive]。
* 若要了解用來轉換資料的 Pig 語言，請參閱[搭配 HDInsight 使用 Pig][hdinsight-use-pig]。
* 若要了解 MapReduce (一種撰寫程式以處理 Hadoop 資料的方式)，請參閱[搭配 HDInsight 使用 MapReduce][hdinsight-use-mapreduce]。
* 若要了解如何使用適用於 Visual Studio 的 HDInsight 工具來分析 HDInsight 資料，請參閱 [開始使用 Visual Studio Hadoop tools for HDInsight](hadoop/apache-hadoop-visual-studio-tools-get-started.md)。

若要進一步了解 HDInsight 如何儲存資料或如何將資料匯入 HDInsight 中，請參閱下列文章：

* 如需有關 HDInsight 如何使用 Azure 儲存體的資訊，請參閱 [搭配 HDInsight 使用 Azure 儲存體](hdinsight-hadoop-use-blob-storage.md)。
* 如需如何上傳資料到 HDInsight 的資訊，請參閱[將資料上傳到 HDInsight][hdinsight-upload-data]。

若要深入了解如何建立或管理 HDInsight 叢集，請參閱下列文章：

* 若要了解如何管理以 Linux 為基礎的 HDInsight 叢集，請參閱 [使用 Ambari 管理 HDInsight 叢集](hdinsight-hadoop-manage-ambari.md)。
* 若要深入了解建立 HDInsight 叢集時可選取的選項，請參閱 [使用自訂選項在 Linux 上建立 HDInsight](hdinsight-hadoop-provision-linux-clusters.md)。
* 如果您已熟悉 Linux 和 Hadoop，但想要知道在 HDInsight 上有關 Hadoop 的特定資訊，請參閱 [在 Linux 上使用 HDInsight](hdinsight-hadoop-linux-information.md)。 本文提供的資訊如下：
  
  * 裝載於叢集上的服務 (例如 Ambari 和 WebHCat) URL
  * Hadoop 檔案和範例在本機檔案系統上的位置
  * Azure 儲存體 (WASB) (而非 HDFS) 做為預設資料儲存體的使用方式

[1]: ../HDInsight/hadoop/apache-hadoop-visual-studio-tools-get-started.md

[hdinsight-provision]: hdinsight-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-mapreduce]:hadoop/hdinsight-use-mapreduce.md
[hdinsight-use-hive]:hadoop/hdinsight-use-hive.md
[hdinsight-use-pig]:hadoop/hdinsight-use-pig.md
