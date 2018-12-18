---
title: 使用 Azure 傳統 CLI 管理 Hadoop 叢集 - Azure HDInsight
description: 了解如何使用 Azure 傳統 CLI 來管理 Azure HDInsight 中的 Hadoop 叢集。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 2586b9219eb145b2033fe2d8fc64b8ae72f34eda
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46958283"
---
# <a name="manage-hadoop-clusters-in-hdinsight-using-the-azure-classic-cli"></a>使用 Azure 傳統 CLI 來管理 HDInsight 中的 Hadoop 叢集
[!INCLUDE [selector](../../includes/hdinsight-portal-management-selector.md)]

了解如何使用 [Azure 傳統 CLI](../cli-install-nodejs.md) 來管理 Azure HDInsight 中的 Hadoop 叢集。 傳統 CLI 實作於 Node.js 中。 此工具可在任何支援 Node.js 的平台上使用，包括 Windows、Mac 和 Linux。

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

## <a name="prerequisites"></a>必要條件
開始閱讀本文之前，您必須符合下列必要條件：

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **Azure 傳統 CLI** - 請參閱[安裝及設定 Azure 傳統 CLI](../cli-install-nodejs.md) 以取得安裝和設定資訊。
* **連線到 Azure**，使用下列命令：

    ```cli
    azure login
    ```
  
    如需使用公司或學校帳戶驗證的詳細資訊，請參閱[從 Azure 傳統 CLI 連線至 Azure 訂用帳戶](/cli/azure/authenticate-azure-cli)。
* 使用下列命令，**切換至 Azure 資源管理員模式**：
  
    ```cli
    azure config mode arm
    ```

若要取得說明，請使用 **-h** 參數。  例如︰

```cli
azure hdinsight cluster create -h
```

## <a name="create-clusters-with-the-cli"></a>使用 CLI 建立叢集
請參閱[使用 Azure 傳統 CLI 建立 HDInsight 中的叢集](hdinsight-hadoop-create-linux-clusters-azure-cli.md)。

## <a name="list-and-show-cluster-details"></a>列出和顯示叢集詳細資料
使用下列命令，以列出並顯示叢集詳細資料：

```cli
azure hdinsight cluster list
azure hdinsight cluster show <Cluster Name>
```

![叢集清單的命令列檢視][image-cli-clusterlisting]

## <a name="delete-clusters"></a>刪除叢集
使用下列命令來刪除叢集：

```cli
azure hdinsight cluster delete <Cluster Name>
```

您也可以刪除包含叢集的資源群組來刪除叢集。 請注意，這將會刪除群組中的所有資源 (包括預設儲存體帳戶)。

```cli
azure group delete <Resource Group Name>
```

## <a name="scale-clusters"></a>調整叢集
若要變更 Hadoop 叢集大小：

```cli
azure hdinsight cluster resize [options] <clusterName> <Target Instance Count>
```


## <a name="enabledisable-http-access-for-a-cluster"></a>啟用/停用叢集 HTTP 存取

```cli
azure hdinsight cluster enable-http-access [options] <Cluster Name> <userName> <password>
azure hdinsight cluster disable-http-access [options] <Cluster Name>
```

## <a name="enabledisable-rdp-access-for-a-cluster"></a>啟用/停用叢集 RDP 存取

```cli
azure hdinsight cluster enable-rdp-access [options] <Cluster Name> <rdpUserName> <rdpPassword> <rdpExpiryDate>
azure hdinsight cluster disable-rdp-access [options] <Cluster Name>
```

## <a name="next-steps"></a>後續步驟
本文中，您學到如何執行不同的 HDInsight 叢集管理工作。 若要深入了解，請參閱下列文章：

* [使用 Azure 入口網站管理 HDInsight][hdinsight-admin-portal]
* [使用 Azure PowerShell 管理 HDInsight][hdinsight-admin-powershell]
* [開始使用 Azure HDInsight][hdinsight-get-started]
* [如何使用 Azure 傳統 CLI][azure-command-line-tools]

[azure-command-line-tools]: ../cli-install-nodejs.md
[azure-create-storageaccount]:../storage/common/storage-create-storage-account.md
[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/


[hdinsight-admin-portal]: hdinsight-administer-use-management-portal.md
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md
[hdinsight-get-started]:hadoop/apache-hadoop-linux-tutorial-get-started.md

[image-cli-account-download-import]: ./media/hdinsight-administer-use-command-line/HDI.CLIAccountDownloadImport.png
[image-cli-clustercreation]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreation.png
[image-cli-clustercreation-config]: ./media/hdinsight-administer-use-command-line/HDI.CLIClusterCreationConfig.png
[image-cli-clusterlisting]: ./media/hdinsight-administer-use-command-line/command-line-list-of-clusters.png "列出和顯示叢集"
