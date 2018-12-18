---
title: 在 HDInsight 中的 Hadoop 叢集上使用空白邊緣節點 - Azure
description: 如何將空白邊緣節點新增至可作為用戶端的 HDInsight 叢集，然後測試/主控 HDInsight 應用程式。
services: hdinsight
ms.reviewer: jasonh
author: jasonwhowell
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 04/23/2018
ms.author: jasonh
ms.openlocfilehash: 1111f3c21e3c3718a9a010284a42ea469e04473d
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43090383"
---
# <a name="use-empty-edge-nodes-on-hadoop-clusters-in-hdinsight"></a>在 HDInsight 中的 Hadoop 叢集上使用空白邊緣節點

了解如何將空白邊緣節點新增至 HDInsight 叢集。 空白邊緣節點是一部 Linux 虛擬機器，其中已安裝及設定和前端節點相同的用戶端工具，但未執行任何 Hadoop 服務。 您可以使用邊緣節點來存取叢集、測試用戶端應用程式，以及裝載用戶端應用程式。 

您可以將空白邊緣節點新增至現有 HDInsight 叢集，以及在建立叢集時新增到新的叢集。 空白邊緣節點是使用 Azure Resource Manager 範本來新增。  下列範例示範如何使用範本來新增︰

    "resources": [
        {
            "name": "[concat(parameters('clusterName'),'/', variables('applicationName'))]",
            "type": "Microsoft.HDInsight/clusters/applications",
            "apiVersion": "2015-03-01-preview",
            "dependsOn": [ "[concat('Microsoft.HDInsight/clusters/',parameters('clusterName'))]" ],
            "properties": {
                "marketPlaceIdentifier": "EmptyNode",
                "computeProfile": {
                    "roles": [{
                        "name": "edgenode",
                        "targetInstanceCount": 1,
                        "hardwareProfile": {
                            "vmSize": "Standard_D3"
                        }
                    }]
                },
                "installScriptActions": [{
                    "name": "[concat('emptynode','-' ,uniquestring(variables('applicationName')))]",
                    "uri": "[parameters('installScriptAction')]",
                    "roles": ["edgenode"]
                }],
                "uninstallScriptActions": [],
                "httpsEndpoints": [],
                "applicationType": "CustomApplication"
            }
        }
    ],

如範例所示，您可以選擇性地呼叫[指令碼動作](hdinsight-hadoop-customize-cluster-linux.md)來執行其他設定，例如在邊緣節點中安裝 [Apache Hue](hdinsight-hadoop-hue-linux.md)。 指令碼動作指令碼必須可在網站上公開存取。  例如，如果指令碼儲存在 Azure 儲存體中，則需使用公用容器或公用 Blob。

邊緣節點虛擬機器大小必須符合 HDInsight 叢集背景工作節點 VM 大小需求。 如需建議的背景工作節點 VM 大小，請參閱[在 HDInsight 中建立 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md#cluster-types)。

建立邊緣節點之後，您可以使用 SSH 連線到邊緣節點，並執行用戶端工具來存取 HDInsight 中的 Hadoop 叢集。

> [!WARNING] 
> 邊緣節點上安裝的自訂元件會收到來自 Microsoft 的商業上合理支援。 可能可以解決您遇到的問題。 或者，您可能需要參考社群資源以取得進一步協助。 以下是從社群取得協助的一些最活躍網站：
>
> * [HDInsight 的 MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight)
> * [http://stackoverflow.com](http://stackoverflow.com)。
>
> 如果您使用 Apache 技術，您可以透過位於 [http://apache.org](http://apache.org) 的 Apache 專案網站 (例如 [Hadoop](http://hadoop.apache.org/) 網站) 尋求協助。

> [!NOTE]
> 和叢集一樣，邊緣節點也會受到修補程式的管理。  如需詳細資訊，請參閱 [HDInsight 的作業系統修補](./hdinsight-os-patching.md)。

## <a name="add-an-edge-node-to-an-existing-cluster"></a>將邊緣節點新增至現有叢集
在本節中，您會使用 Resource Manager 範本將邊緣節點新增至現有 HDInsight 叢集。  Resource Manager 範本可在 [GitHub](https://azure.microsoft.com/resources/templates/101-hdinsight-linux-add-edge-node/)中找到。 Resource Manager 範本會呼叫位於 https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-add-edge-node/scripts/EmptyNodeSetup.sh 的指令碼動作。此指令碼不會執行任何動作。  其目的只是為了示範從 Resource Manager 範本呼叫指令碼動作。

**將空白邊緣節點新增至現有叢集**

1. 按一下以下影像，在 Azure 入口網站中登入 Azure 並開啟 Azure Resource Manager 範本。 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-add-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>
3. 設定下列屬性：
   
   * **訂用帳戶**：選取用來建立叢集的 Azure 訂用帳戶。
   * **資源群組**：選取用於現有 HDInsight 叢集的資源群組。
   * **位置**：選取現有 HDInsight 叢集的位置。
   * **叢集名稱**︰輸入現有 HDInsight 叢集的名稱。
   * **邊緣節點大小**︰選取其中一個 VM 大小。 VM 大小必須符合背景工作節點 VM 大小需求。 如需建議的背景工作節點 VM 大小，請參閱[在 HDInsight 中建立 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md#cluster-types)。
   * **邊緣節點前置詞**︰預設值是 **new**。  若使用預設值，邊緣節點名稱是 **new-edgenode**。  您可以從入口網站自訂前置詞。 您也可以從範本自訂完整名稱。

4. 核取 [我同意上方所述的條款及條件]，然後按一下 [購買] 以建立邊緣節點。

>[!IMPORTANT]
> 請務必選取用於現有 HDInsight 叢集的 Azure 資源群組。  否則，您會收到錯誤訊息：「無法對巢狀資源執行所要求的作業。 找不到父資源 '&lt;叢集名稱>'」。

## <a name="add-an-edge-node-when-creating-a-cluster"></a>在建立叢集時新增邊緣節點
在本節中，您會使用 Resource Manager 範本對邊緣節點建立 HDInsight 叢集。  Resource Manager 範本可在 [Azure 快速入門範本資源庫](https://azure.microsoft.com/documentation/templates/101-hdinsight-linux-with-edge-node/)中找到。 Resource Manager 範本會呼叫位於 https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-hdinsight-linux-with-edge-node/scripts/EmptyNodeSetup.sh 的指令碼動作。此指令碼不會執行任何動作。  其目的只是為了示範從 Resource Manager 範本呼叫指令碼動作。

**使用邊緣節點建立 HDInsight 叢集**

1. 如果您還沒有 HDInsight 叢集，請予以建立。  請參閱[開始在 HDInsight 中使用 Hadoop](hadoop/apache-hadoop-linux-tutorial-get-started.md)。
2. 按一下以下影像，在 Azure 入口網站中登入 Azure 並開啟 Azure Resource Manager 範本。 
   
    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-linux-with-edge-node%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-use-edge-node/deploy-to-azure.png" alt="Deploy to Azure"></a>
3. 設定下列屬性：
   
   * **訂用帳戶**：選取用來建立叢集的 Azure 訂用帳戶。
   * **資源群組**：建立用於叢集的新資源群組。
   * **位置**：選取資源群組的位置。
   * **叢集名稱**︰輸入要建立之新叢集的名稱。
   * **叢集登入使用者名稱**︰輸入 Hadoop HTTP 使用者名稱。  預設名稱為 **admin**。
   * **叢集登入密碼**︰輸入 Hadoop HTTP 使用者密碼。
   * **SSH 使用者名稱**︰輸入 SSH 使用者名稱。 預設名稱為 **sshuser**。
   * **SSH 密碼**︰輸入 SSH 使用者密碼。
   * **安裝指令碼動作**︰請保留預設值以便完成本教學課程。
     
     某些屬性已被硬式編碼在範本中︰叢集類型、叢集背景工作角色節點計數、邊緣節點大小與邊緣節點名稱。
4. 核取 [我同意上方所述的條款及條件]，然後按一下 [購買] 以建立具有邊緣節點的叢集。

## <a name="add-multiple-edge-nodes"></a>新增多個邊緣節點

您可以將多個邊緣節點新增至 HDInsight 叢集。  多個邊緣節點的設定只能使用 Azure Resource Manager 範本來執行。  請參閱本文開頭處的範本範例。  您必須更新 **targetInstanceCount**，以反映您要建立的邊緣節點數目。

## <a name="access-an-edge-node"></a>存取邊緣節點
邊緣節點 ssh 端點是 &lt;EdgeNodeName>.&lt;ClusterName>-ssh.azurehdinsight.net:22。  例如，new-edgenode.myedgenode0914-ssh.azurehdinsight.net:22。

在 Azure 入口網站上，邊緣節點會顯示為應用程式。  入口網站可提供您相關資訊，以便使用 SSH 存取邊緣節點。

**確認邊緣節點 SSH 端點**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟具有邊緣節點的 HDInsight 叢集。
3. 按一下 [應用程式] 。 您應該會看到邊緣節點。  預設名稱為 **new-edgenode**中找到。
4. 按一下邊緣節點。 您應該會看到 SSH 端點。

**在邊緣節點上使用 Hive**

1. 使用 SSH 連線到邊緣節點。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 在使用 SSH 連線到邊緣節點之後，使用下列命令來開啟 Hive 主控台︰
   
        hive
3. 執行下列命令，以顯示叢集中的 Hive 資料表︰
   
        show tables;

## <a name="delete-an-edge-node"></a>刪除邊緣節點
您可以從 Azure 入口網站刪除邊緣節點。

**存取邊緣節點**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 開啟具有邊緣節點的 HDInsight 叢集。
3. 按一下 [應用程式] 。 您應該會看到邊緣節點清單。  
4. 以滑鼠右鍵按一下您想要刪除的邊緣節點，然後按一下 [刪除] 。
5. 按一下 [ **是** ] 以確認。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何新增邊緣節點以及如何存取邊緣節點。 若要深入了解，請參閱下列文章：

* [安裝 HDInsight 應用程式](hdinsight-apps-install-applications.md)︰了解如何將 HDInsight 應用程式安裝到您的叢集。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)︰了解如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* [發佈 HDInsight 應用程式](hdinsight-apps-publish-applications.md)︰了解如何將自訂 HDInsight 應用程式發佈至 Azure Marketplace。
* [MSDN：安裝 HDInsight 應用程式](https://msdn.microsoft.com/library/mt706515.aspx)︰了解如何定義 HDInsight 應用程式。
* [使用指令碼動作自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)：了解如何使用指令碼動作來安裝其他應用程式。
* [使用 Resource Manager 範本在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)︰了解如何呼叫 Resource Manager 範本來建立 HDInsight 叢集。

