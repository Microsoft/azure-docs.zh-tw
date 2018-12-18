---
title: 在 Azure HDInsight 上使用 Grafana
description: 了解如何在 Azure HDInsight 中存取 Grafana。
services: hdinsight
ms.service: hdinsight
author: jasonwhowell
ms.author: jasonh
ms.reviewer: jasonh
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/17/2018
ms.openlocfilehash: cd6c8147788ccc9abc1f002137c2b1a6c4622ce1
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042440"
---
# <a name="access-grafana-in-azure-hdinsight"></a>在 Azure HDInsight 中存取 Grafana


Grafana 是廣受使用的開放原始碼圖形和儀表板產生器。 Grafana 的功能非常豐富；不僅可讓使用者建立可自訂和可共用的儀表板，也提供樣板化/指令碼式儀表板、LDAP 整合、多種資料來源等功能。

目前，只有 Azure HDInsight 中的互動式查詢叢集類型可支援 Grafana。


如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="create-a-hadoop-cluster"></a>建立 Hadoop 叢集

在本節中，您會使用 Azure Resource Manager 範本，在 HDInsight 中建立互動式查詢叢集。 進行本文並不需要具備 Resource Manager 範本體驗。 

1. 按一下以下的 [部署至 Azure] 按鈕，在 Azure 入口網站中登入 Azure 並開啟 Resource Manager 範本。 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-interactive-hive%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-grafana/deploy-to-azure.png" alt="Deploy to Azure"></a>

2. 輸入或選取如下列螢幕擷取畫面所建議的值：

    > [!NOTE]
    > 您提供的值必須是唯一且應該遵循命名方針。 範本不會執行驗證檢查。 如果您提供的值已在使用中或未遵循方針，則會在提交範本之後收到錯誤。       
    > 
    >
    
    ![HDInsight Linux 開始使用入口網站上的 Resource Manager 範本](./media/hdinsight-grafana/hdinsight-linux-get-started-arm-template-on-portal.png "使用 Azure 入口網站和資源群組管理員範本在 HDInsigut 中部署 Hadoop 叢集")

    輸入或選取下列值：
    
    |屬性  |說明  |
    |---------|---------|
    |**訂用帳戶**     |  選取 Azure 訂用帳戶。 |
    |**資源群組**     | 建立資源群組，或選取現有的資源群組。  資源群組是 Azure 元件的容器。  在此案例中，資源群組包含 HDInsight 叢集和相依的 Azure 儲存體帳戶。 |
    |**位置**     | 選取您要建立叢集的 Azure 位置。  選擇靠近您的位置，以獲得最佳效能。 |
    |**叢集類型**     | 選取 [Hadoop]。 |
    |**叢集名稱**     | 輸入 Hadoop 叢集的名稱。 由於 HDInsight 中的所有叢集共用相同的 DNS 命名空間，因此這個名稱必須是唯一的。 名稱最多可包含 59 個字元，而這些字元可以是字母、數字和連字號。 名稱的第一個和最後一個字元不可以是連字號。 |
    |**叢集登入名稱和密碼**     | 預設登入名稱為 **admin**。密碼長度至少必須為 10 個字元，且必須包含至少一個數字、一個大寫字母及一個小寫字母、一個非英數字元 (除了字元 ' " ` \)。 確定您**不會提供**常見密碼，例如 "Pass@word1"。|
    |**SSH 使用者名稱和密碼**     | 預設的使用者名稱為 **sshuser**。  您可以將 SSH 使用者名稱重新命名。  SSH 使用者密碼與叢集登入密碼具有相同的需求。|
       
    某些屬性已硬式編碼在範本中。  您可以從範本設定這些值。 如需這些屬性的詳細說明，請參閱[在 HDInsight 中建立Hadoop 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。

3. 選取 [我同意上方所述的條款及條件] 和 [釘選到儀表板]，然後選取 [購買]。 您應該會在入口網站儀表板上看到標題為**正在提交部署**的新圖格。 大約需要 20 分鐘的時間來建立叢集。

    ![範本部署進度](./media/hdinsight-grafana/deployment-progress-tile.png "Azure 範本部署進度")

4. 建立叢集後，此圖格的標題會變更為您指定的資源群組名稱。 此圖格也會列出資源群組內所建立的 HDInsight 叢集。 
   
    ![HDInsight Linux 開始使用的資源群組](./media/hdinsight-grafana/hdinsight-linux-get-started-resource-group.png "Azure HDInsight 叢集資源群組")
    
5. 此圖格也會列出與叢集相關聯的預設儲存體。 每個叢集都具備 [Azure 儲存體帳戶](../hdinsight-hadoop-use-blob-storage.md)或 [Azure Data Lake 帳戶](../hdinsight-hadoop-use-data-lake-store.md)相依性。 也稱為預設儲存體帳戶。 HDInsight 叢集與其預設儲存體帳戶必須並存於相同的 Azure 區域。 刪除叢集並不會刪除儲存體帳戶。
    

> [!NOTE]
> 如需其他叢集建立方法及了解本教學課程中使用的屬性，請參閱 [建立 HDInsight 叢集](../hdinsight-hadoop-provision-linux-clusters.md)。       
> 
>

## <a name="access-the-grafana-dashboard"></a>存取 Grafana 儀表板

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [HDInsight 叢集]，然後選取您在上一節建立的叢集名稱。
3. 在 [快速連結] 下，按一下 [叢集儀表板]。

    ![HDInsight 叢集儀表板入口網站](./media/hdinsight-grafana/hdinsight-portal-cluster-dashboard.png "入口網站上的 HDInsight 叢集儀表板")

4. 在儀表板中，按一下 [Grafana] 圖格。
5. 輸入 Hadoop 叢集使用者認證。
6. Grafana 儀表板顯示如下：

    ![HDInsight Grafana 儀表板](./media/hdinsight-grafana/hdinsight-grafana-dashboard.png "HDInsight Grafana 儀表板")

## <a name="clean-up-resources"></a>清除資源
完成本文之後，您可能想要刪除叢集。 利用 HDInsight，您的資料會儲存在 Azure 儲存體中，以便您在未使用叢集時安全地進行刪除。 您也需支付 HDInsight 叢集的費用 (即使未使用)。 由於叢集費用是儲存體費用的許多倍，所以刪除未使用的叢集符合經濟效益。 

> [!NOTE]
> 如果您要「立即」繼續下一個教學課程，以了解如何使用 HDInsight 上的 Hadoop 來執行 ETL 作業，則建議讓叢集保持執行狀態。 這是因為在教學課程中，您必須重新建立 Hadoop 叢集。 不過，如果您不會立即開始下一個教學課程，則現在就必須刪除該叢集。
> 
> 

**刪除叢集和/或預設儲存體帳戶**

1. 回到瀏覽器索引標籤，您可在其中存取 Azure 入口網站。 您應該位於叢集的 [概觀] 頁面上。 如果您只想刪除叢集，但保留預設儲存體帳戶，請選取 [刪除]。

    ![HDInsight 刪除叢集](./media/hdinsight-grafana/hdinsight-delete-cluster.png "刪除 HDInsight 叢集")

2. 如果您想要刪除叢集以及預設儲存體帳戶，則選取資源群組名稱 (上一個螢幕擷取畫面中醒目提示的項目) 來開啟資源群組頁面。

3. 選取 [刪除資源群組] 以刪除資源群組，其中包含了叢集和預設儲存體帳戶。 請注意，刪除資源群組會刪除儲存體帳戶。 如果您想要保留儲存體帳戶，請選擇只刪除叢集。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何使用 Resource Manager 範本來建立以 Linux 為基礎的 HDInsight 叢集，以及如何執行基本的 Hive 查詢。 在下一篇文章中，您將了解如何使用 HDInsight 上的 Hadoop 來執行擷取、轉換及載入 (ETL) 作業。

> [!div class="nextstepaction"]
>[使用 HDInsight 上的 Apache Hive 來擷取、轉換和載入資料](../hdinsight-analyze-flight-delay-data-linux.md)

如果您準備好開始使用您自己的資料，並需要進一步了解 HDInsight 儲存資料的方式或如何將資料匯入 HDInsight，請參閱下列文章：

* 如需有關 HDInsight 如何使用 Azure 儲存體的資訊，請參閱 [搭配 HDInsight 使用 Azure 儲存體](../hdinsight-hadoop-use-blob-storage.md)。
* 如需關於如何上傳資料到 HDInsight 的資訊，請參閱[將資料上傳到 HDInsight](../hdinsight-upload-data.md)。

若要深入了解如何使用 HDInsight 分析資料，請參閱下列文章：

* 若要深入了解如何搭配 HDInsight 使用 Hive，包括如何從 Visual Studio 執行 Hive 查詢，請參閱[搭配 HDInsight 使用 Hive](../hdinsight-use-hive.md)。
* 若要了解用來轉換資料的 Pig 語言，請參閱[搭配 HDInsight 使用 Pig](../hdinsight-use-pig.md)。
* 若要了解 MapReduce (一種撰寫程式以處理 Hadoop 資料的方式)，請參閱[搭配 HDInsight 使用 MapReduce](../hdinsight-use-mapreduce.md)。
* 若要了解如何使用適用於 Visual Studio 的 HDInsight 工具來分析 HDInsight 資料，請參閱 [開始使用 Visual Studio Hadoop tools for HDInsight](../hadoop/apache-hadoop-visual-studio-tools-get-started.md)。



如果您想要深入了解如何建立或管理 HDInsight 叢集，請參閱下列文章：

* 若要了解如何管理以 Linux 為基礎的 HDInsight 叢集，請參閱 [使用 Ambari 管理 HDInsight 叢集](../hdinsight-hadoop-manage-ambari.md)。
* 若要深入了解建立 HDInsight 叢集時可選取的選項，請參閱 [使用自訂選項在 Linux 上建立 HDInsight](../hdinsight-hadoop-provision-linux-clusters.md)。


[1]: ../HDInsight/apache-hadoop-visual-studio-tools-get-started.md

[hdinsight-provision]: hdinsight-provision-linux-clusters.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-use-hive]: hdinsight-use-hive.md
[hdinsight-use-pig]: hdinsight-use-pig.md


