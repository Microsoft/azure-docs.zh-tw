---
title: 在 Azure HDInsight 上安裝您的自訂 Apache Hadoop 應用程式
description: 瞭解如何在 Azure HDInsight 中安裝適用于 Apache Hadoop 叢集的 HDInsight 應用程式。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: 9591551b7d5657803e770bd71b47acb141b40fb6
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98942688"
---
# <a name="install-custom-apache-hadoop-applications-on-azure-hdinsight"></a>在 Azure HDInsight 上安裝自訂的 Apache Hadoop 應用程式

在本文中，您將瞭解如何在未發佈至 Azure 入口網站的 Azure HDInsight 上安裝 [Apache Hadoop](https://hadoop.apache.org/) 應用程式。 您將在本文中安裝的應用程式為 [色調](https://gethue.com/)。

HDInsight 應用程式是使用者可以在 HDInsight 叢集上安裝的應用程式。  Microsoft 獨立軟體廠商 (ISV) 或您可以自己開發這些應用程式。  

## <a name="prerequisites"></a>Prerequisites

如果您想要在現有的 HDInsight 叢集上安裝 HDInsight 應用程式，您必須有 HDInsight 叢集。 若要建立叢集，請參閱 [建立叢集](hadoop/apache-hadoop-linux-tutorial-get-started.md)。 您也可以在建立 HDInsight 叢集時安裝 HDInsight 應用程式。

## <a name="install-hdinsight-applications"></a>安裝 HDInsight 應用程式

HDInsight 應用程式可以在您建立叢集時安裝，或安裝至現有的 HDInsight 叢集。 如需定義 Azure Resource Manager 範本，請參閱 [MSDN：安裝 HDInsight 應用程式](/rest/api/hdinsight/hdinsight-application)。

部署此應用程式 (Hue) 所需的檔案︰

* [azuredeploy.json](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/azuredeploy.json)：可供安裝 HDInsight 應用程式的 Resource Manager 範本。 如需開發您自己的 Resource Manager 範本，請參閱 [MSDN：安裝 HDInsight 應用程式](/rest/api/hdinsight/hdinsight-application) 。
* [hue-install_v0.sh](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/scripts/Hue-install_v0.sh)：Resource Manager 範本為了設定邊緣節點所呼叫的指令碼動作。
* [hue-binaries.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz)︰從 hui-install_v0.sh 呼叫的 Hue 二進位檔。
* [hue-binaries-14-04.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz)︰從 hui-install_v0.sh 呼叫的 Hue 二進位檔。
* [webwasb-tomcat.tar.gz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/webwasb-tomcat.tar.gz)：從 hui-install_v0.sh 呼叫的範例 Web 應用程式 (Tomcat)。

### <a name="to-install-hue-to-an-existing-hdinsight-cluster"></a>將 Hue 安裝至現有的 HDInsight 叢集

1. 選取下圖以登入 Azure，並在 Azure 入口網站中開啟 Resource Manager 範本。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FHue%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Resource Manager 範本位於 [https://github.com/hdinsight/Iaas-Applications/tree/master/Hue](https://github.com/hdinsight/Iaas-Applications/tree/master/Hue) 。  若要了解如何撰寫此 Resource Manager 範本，請參閱 [MSDN：安裝 HDInsight 應用程式](/rest/api/hdinsight/hdinsight-application)。

1. 從下拉式清單中選取包含您的叢集的現有 **資源群組** 。 您必須使用與叢集相同的資源群組。

1. 輸入您要安裝應用程式的叢集名稱。 此叢集必須是現有的叢集。

1. 選取 [ **我同意上方所述的條款及條件**] 核取方塊。

1. 選取 [購買]  。

您可以從釘選到入口網站儀表板和入口網站通知的圖格查看安裝狀態 (按一下入口網站頂端的鈴鐺圖示)。  安裝此應用程式需要 10 分鐘左右。

### <a name="to-install-hue-while-creating-a-cluster"></a>在建立叢集時安裝 Hue

1. 選取下圖以登入 Azure，並在 Azure 入口網站中開啟 Resource Manager 範本。

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhdinsightapps%2Fcreate-linux-based-hadoop-cluster-in-hdinsight.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Resource Manager 範本位於 [https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json](https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json) 。  若要了解如何撰寫此 Resource Manager 範本，請參閱 [MSDN：安裝 HDInsight 應用程式](/rest/api/hdinsight/hdinsight-application)。

2. 請依照指示來建立叢集和安裝 Hue。 如需建立 HDInsight 叢集的詳細資訊，請參閱 [在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

### <a name="other-installation-methods"></a>其他安裝方法

除了 Azure 入口網站，您也可以使用 [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-powershell) 和 [Azure CLI](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-azure-cli) 來呼叫 Resource Manager 範本。

## <a name="validate-the-installation"></a>驗證安裝

您可以在 Azure 入口網站上檢查應用程式狀態以驗證應用程式安裝。 此外，您也可以驗證所有 HTTP 端點是否如預期般出現，以及網頁（如果有的話）。

針對 **色調**，您可以使用下列步驟：

### <a name="azure-portal"></a>Azure 入口網站

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取您安裝應用程式的叢集。
1. 從 [ **設定** ] 功能表中選取 [ **應用程式**]。
1. 從清單中選取 [ **色調** ] 以查看屬性。  
1. 選取 [網頁] 連結來驗證網站。

### <a name="azure-cli"></a>Azure CLI

取代 `CLUSTERNAME` ，並 `RESOURCEGROUP` 使用相關的值，然後輸入下列命令：

* 列出 HDInsight 叢集的所有應用程式。

    ```azurecli
    az hdinsight application list --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

* 取得指定之應用程式的屬性。

    ```azurecli
    az hdinsight application show --name hue --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

## <a name="troubleshoot-the-installation"></a>安裝疑難排解

您可以從入口網站通知檢查應用程式安裝狀態 (按一下入口網站頂端的鈴鐺圖示)。

如果應用程式安裝失敗，您可以從三個位置查看錯誤訊息和偵錯工具資訊：

* HDInsight 應用程式︰一般錯誤資訊。

    從入口網站開啟叢集，然後從 [設定] 中選取 [應用程式]：

    ![hdinsight 應用程式的應用程式安裝錯誤](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-error.png)

* HDInsight 指令碼動作︰如果 HDInsight 應用程式的錯誤訊息指出指令碼動作失敗，則 [指令碼動作] 窗格會顯示指令碼失敗的詳細資訊。

    從 [設定] 選取 [腳本動作]。 指令碼動作歷程記錄會顯示錯誤訊息

    ![hdinsight 應用程式的指令碼動作錯誤](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-script-action-error.png)

* Apache Ambari Web UI：如果安裝腳本是失敗的原因，請使用 Ambari Web UI 來檢查有關安裝腳本的完整記錄。

    如需詳細資訊，請參閱 [疑難排解腳本動作](./troubleshoot-script-action.md)。

## <a name="remove-hdinsight-applications"></a>移除 HDInsight 應用程式

### <a name="azure-portal"></a>Azure 入口網站

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 選取您安裝應用程式的叢集。
1. 從 [ **設定** ] 功能表中選取 [ **應用程式**]。
1. 以滑鼠右鍵按一下您要移除的應用程式，然後選取 [ **刪除**]。
1. 選取 [是]  加以確認。

### <a name="azure-cli"></a>Azure CLI

`NAME`將、 `CLUSTERNAME` 和取代為 `RESOURCEGROUP` 相關的值，然後輸入下列命令：

```azurecli
az hdinsight application delete --name NAME --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
```

## <a name="next-steps"></a>後續步驟

* [MSDN：安裝 HDInsight 應用程式](/rest/api/hdinsight/hdinsight-application)︰了解如何開發 Resource Manager 範本以供部署 HDInsight 應用程式。
* [安裝 HDInsight 應用程式](hdinsight-apps-install-applications.md)：了解如何將 HDInsight 應用程式安裝到您的叢集。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)：了解如何將自訂 HDInsight 應用程式發佈到 Azure Marketplace。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [使用 Resource Manager 範本在 HDInsight 中建立以 Linux 為基礎的 Apache Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)︰了解如何呼叫 Resource Manager 範本來建立 HDInsight 叢集。
* [在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)︰了解如何使用空白邊緣節點來存取 HDInsight 叢集、測試 HDInsight 應用程式，以及裝載 HDInsight 應用程式。
