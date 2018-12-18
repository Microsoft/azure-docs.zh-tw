---
title: 使用指令碼動作自訂 HDInsight 叢集 - Azure
description: 使用指令碼動作在以 Linux 為基礎的 HDInsight 叢集上新增自訂元件。 指令碼動作是 Bash 指令碼，可用來自訂叢集設定，或新增其他服務和公用程式，如 Hue、Solr 或 R。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/01/2018
ms.author: jasonh
ms.openlocfilehash: e9662a0a0def91fa2cace1eb47dceffed5ff1ae6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978024"
---
# <a name="customize-linux-based-hdinsight-clusters-using-script-actions"></a>使用指令碼動作自訂 Linux 型 HDInsight 叢集

HDInsight 提供一個稱為**指令碼動作**的設定方法，此方法會叫用自訂指令碼來自訂叢集。 這些指令碼可用來安裝其他元件和變更組態設定。 叢集建立期間或叢集建立之後，可以使用指令碼動作。

> [!IMPORTANT]
> 只有以 Linux 為基礎的 HDInsight 叢集能夠在已在執行中的叢集上使用指令碼動作。
>
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

指令碼動作也可以發佈到 Azure Marketplace 做為 HDInsight 應用程式。 如需 HDInsight 應用程式的詳細資訊，請參閱 [將 HDInsight 應用程式發佈到 Azure Marketplace](hdinsight-apps-publish-applications.md)。

## <a name="permissions"></a>權限

如果您要使用已加入網域的 HDInsight 叢集，則在對叢集使用指令碼動作時必須有兩個 Ambari 權限︰

* **AMBARI.RUN\_CUSTOM\_COMMAND**：依預設，Ambari 系統管理員角色會具有此權限。
* **CLUSTER.RUN\_CUSTOM\_COMMAND**：依預設，HDInsight 叢集系統管理員和 Ambari 系統管理員會具有此權限。

如需使用已加入網域之 HDInsight 權限的詳細資訊，請參閱[管理已加入網域的 HDInsight 叢集](./domain-joined/apache-domain-joined-manage.md)。

## <a name="access-control"></a>存取控制

如果您不是 Azure 訂用帳戶的系統管理員/擁有者，您的帳戶必須至少有包含 HDInsight 叢集之資源群組的**參與者**存取權。

此外，如果您要建立 HDInsight 叢集，至少具備 Azure 訂用帳戶之**參與者**存取權的某人，必須已在先前註冊 HDInsight 的提供者。 具有訂用帳戶之參與者存取權的使用者第一次在訂用帳戶上建立資源時，便會註冊提供者。 透過[使用 REST 註冊提供者](https://msdn.microsoft.com/library/azure/dn790548.aspx)，也可以不需要建立資源就完成註冊作業。

如需使用存取管理的詳細資訊，請參閱下列文件：

* [開始在 Azure 入口網站中使用存取管理](../role-based-access-control/overview.md)
* [使用角色指派來管理 Azure 訂用帳戶資源的存取權](../role-based-access-control/role-assignments-portal.md)

## <a name="understanding-script-actions"></a>了解指令碼動作

指令碼動作是在 HDInsight 叢集中節點上執行的 Bash 指令碼。 以下是指令碼動作的特性和功能。

* 必須儲存在可從 HDInsight 叢集存取的 URI 上。 以下是可能的儲存位置：

    * HDInsight 叢集可存取的 **Azure Data Lake Store** 帳戶。 如需有關使用 Azure Data Lake Store 與 HDInsight 的詳細資訊，請參閱[快速入門：在 HDInsight 中設定叢集](../storage/data-lake-storage/quickstart-create-connect-hdi-cluster.md)。

        使用 Data Lake Store 中儲存的指令碼時，URI 格式為 `adl://DATALAKESTOREACCOUNTNAME.azuredatalakestore.net/path_to_file`。

        > [!NOTE]
        > 用來存取 Data Lake Store 的服務主體 HDInsight 必須具有指令碼的讀取權限。

    * 本身是 HDInsight 叢集主要或其他儲存體帳戶之 **Azure 儲存體帳戶**中的 Blob。 在建立叢集期間，已將這兩種儲存體帳戶的存取權都授與 HDInsight。

    * 公用檔案共用服務，例如 Azure Blob、GitHub、OneDrive、Dropbox 等。

        如需範例 URI，請參閱[範例指令碼動作指令碼](#example-script-action-scripts)一節。

        > [!WARNING]
        > HDInsight 僅支援__一般用途__的 Azure 儲存體帳戶。 目前不支援 __Blob 儲存體__帳戶類型。

* 可以限制為**只在特定節點類型上執行**，例如前端節點或背景工作節點。

* 可以是**持續性**或**臨時性**。

    **持續性**指令碼可用來自訂透過調整規模作業新增至叢集的新背景工作節點。 持續性指令碼也可以在進行調整規模作業時，將變更套用至另一個節點類型，例如前端節點。

  > [!IMPORTANT]
  > 持續性指令碼動作必須有唯一的名稱。

    **臨時性**指令碼不會持續留存。 在指令碼執行後，它們不會套用至新增至叢集的背景工作角色節點。 您可以在之後將臨時性指令碼升階為持續性指令碼，或將持續性指令碼降階為臨時性指令碼。

  > [!IMPORTANT]
  > 建立叢集期間使用的指令碼動作會自動保存下來。
  >
  > 即使您特別指出應予保存，仍然不會保存失敗的指令碼。

* 可以接受指令碼在執行期間所使用的**參數**。

* 在叢集節點上以**根層級權限**執行。

* 可以透過 **Azure 入口網站**、**Azure PowerShell**、**Azure 傳統 CLI** 或 **HDInsight .NET SDK** 使用

叢集會保留所有已執行指令碼的歷程記錄。 當您需要尋找升級或降級作業之指令碼的識別碼時，歷程記錄就很有用。

> [!IMPORTANT]
> 沒有任何自動方式可復原指令碼動作所做的變更。 請手動回復變更，或提供可回復變更的指令碼。

### <a name="script-action-in-the-cluster-creation-process"></a>叢集建立程序中的指令碼動作

在叢集建立期間使用的指令碼動作與在現有叢集上執行的指令碼動作稍微不同︰

* 此指令碼會**自動保存**。

* 指令碼中若發生**失敗**，可能會導致叢集建立程序失敗。

下圖說明在建立程序期間執行指令碼動作的時間：

![HDInsight 叢集自訂和叢集建立期間的階段][img-hdi-cluster-states]

設定 HDInsight 時會執行此指令碼。 指令碼會以平行方式在叢集中所有指定的節點上執行，並且在節點上以根權限執行。

> [!NOTE]
> 您可以執行停止和啟動服務 (包括 Hadoop 相關服務) 等作業。 如果您停止服務，就必須確保 Ambari 服務及其他 Hadoop 相關服務在指令碼完成前處於執行狀態。 這些服務必須在叢集建立時，成功地判斷叢集的健康情況和狀態。


在叢集建立期間，您可以同時使用多個指令碼動作。 這些指令碼會以指定的順序叫用。

> [!IMPORTANT]
> 指令碼動作必須在 60 分鐘內完成，否則就會逾時。 在叢集佈建期間，同時執行指令碼與其他安裝和組態程序。 與在您開發環境中的執行時間相較，爭用 CPU 時間和網路頻寬等資源可能會導致指令碼需要較長的時間才能完成。
>
> 若要讓執行指令碼所花費的時間降到最低，請避免從原始程式碼下載和編譯應用程式之類的工作。 預先編譯應用程式，並在 Azure 儲存體中儲存二進位檔。


### <a name="script-action-on-a-running-cluster"></a>執行中叢集上的指令碼動作

在已處於執行中狀態的叢集上執行的指令碼如果發生失敗，並不會自動導致叢集變成失敗狀態。 指令碼完成後，叢集應該會回到「執行中」狀態。

> [!IMPORTANT]
> 即使叢集狀態為「執行中」，失敗的指令碼仍可能已造成問題。 例如，指令碼可能會刪除叢集所需的檔案。
>
> 指令碼動作會以根權限執行。 請先確定您了解指令碼的作用，再將它套用到您的叢集。

將指令碼套用到叢集時，如果指令碼執行成功，叢集狀態會從 [執行中] 變更為 [已接受]，再變更為 [HDInsight 設定]，最後回到 [執行中]。 指令碼狀態會記錄在指令碼動作歷程記錄中，您可以使用此資訊來判斷指令碼是成功或失敗。 例如， `Get-AzureRmHDInsightScriptActionHistory` PowerShell Cmdlet 可用來檢視指令碼的狀態。 它會傳回類似以下文字的資訊：

    ScriptExecutionId : 635918532516474303
    StartTime         : 8/14/2017 7:40:55 PM
    EndTime           : 8/14/2017 7:41:05 PM
    Status            : Succeeded

> [!IMPORTANT]
> 如果您在叢集建立後變更叢集使用者 (管理員) 密碼，針對此叢集執行的指令碼動作可能會失敗。 如果您有任何以背景工作角色節點為目標的持續性指令碼動作，當您調整叢集的規模時，這些指令碼動作可能會失敗。

## <a name="example-script-action-scripts"></a>範例指令碼動作指令碼

指令碼動作的指令碼可以透過下列公用程式使用：

* Azure 入口網站
* Azure PowerShell
* Azure 傳統 CLI
* HDInsight .NET SDK

HDInsight 提供一些指令碼以在 HDInsight 叢集上安裝下列元件：

| 名稱 | 指令碼 |
| --- | --- |
| **新增 Azure 儲存體帳戶** |https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh 。請參閱 [在 HDInsight 叢集新增儲存體](hdinsight-hadoop-add-storage.md)。 |
| **安裝色調** |https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh 。請參閱 [在 HDInsight 叢集上安裝及使用色調](hdinsight-hadoop-hue-linux.md)。 |
| **安裝 Presto** |https://raw.githubusercontent.com/hdinsight/presto-hdinsight/master/installpresto.sh 。請參閱 [在 HDInsight 叢集上安裝和使用 Presto](hdinsight-hadoop-install-presto.md)。 |
| **安裝 Solr** |https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh 。請參閱 [在 HDInsight 叢集上安裝及使用 Solr](hdinsight-hadoop-solr-install-linux.md)。 |
| **安裝 Giraph** |https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh 。請參閱 [在 HDInsight 叢集上安裝及使用 Giraph](hdinsight-hadoop-giraph-install-linux.md)。 |
| **預先載入 Hive 程式庫** |https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh 。請參閱 [在 HDInsight 叢集上新增 Hive 程式庫](hdinsight-hadoop-add-hive-libraries.md)。 |
| **安裝或更新 Mono** | https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash 。 請參閱 [在 HDInsight 上安裝或更新 Mono](hdinsight-hadoop-install-mono.md)。 |

## <a name="use-a-script-action-during-cluster-creation"></a>在建立叢集期間使用指令碼動作

本節提供建立 HDInsight 叢集時以不同的方式使用指令碼動作的範例。

### <a name="use-a-script-action-during-cluster-creation-from-the-azure-portal"></a>在建立叢集期間從 Azure 入口網站使用指令碼動作

1. 依[在 HDInsight 建立 Hadoop 叢集](hdinsight-hadoop-provision-linux-clusters.md)中的描述開始建立叢集。 叢集建立期間，您會抵達 [叢集摘要] 頁面。 請從 [叢集摘要] 頁面中，選取 [進階設定] 的 [編輯] 連結。

    ![[Advanced settings] \(進階設定\) 連結](./media/hdinsight-hadoop-customize-cluster-linux/advanced-settings-link.png)

3. 從 [進階設定] 區段中，選取 [指令碼動作]。 從 [指令碼動作] 區段中，選取 [+ 送出新的]

    ![送出新的指令碼動作](./media/hdinsight-hadoop-customize-cluster-linux/add-script-action.png)

4. 使用 [Select a script] \(選取指令碼\) 項目選取預先製作的指令碼。 若要使用自訂指令碼，請選取 [Custom] \(自訂\)，然後為您的指令碼提供 [Name] \(名稱\) 和 [Bash script URI] \(Bash 指令碼 URI\)。

    ![在選取指令碼表單中加入指令碼](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    下表說明表單上的元素：

    | 屬性 | 值 |
    | --- | --- |
    | 選取指令碼 | 若要使用自己的指令碼，請選取 [自訂]。 或是選取其中一個提供的指令碼。 |
    | 名稱 |指定指令碼動作的名稱。 |
    | Bash 指令碼 URI |指定指令碼的 URI。 |
    | Head/Worker/Zookeeper |指定執行指令碼的節點 (**前端**、**背景工作**或 **ZooKeeper**)。 |
    | 參數 |如果指令碼要求，請指定參數。 |

    使用 [保存此指令碼動作] 項目，可確保在執行規模調整作業時套用此指令碼。

5. 選取 [Create] \(建立\) 以儲存指令碼。 您可以接著使用 [+ Submit new] \(+ 送出新的\) 以加入另一個指令碼。

    ![多個指令碼動作](./media/hdinsight-hadoop-customize-cluster-linux/multiple-scripts.png)

    當您完成加入指令碼後，使用 [選取] 按鈕，然後使用 [下一步] 按鈕返回 [叢集摘要] 區段。

3. 若要建立叢集，請從 [叢集摘要] 區段選取 [建立]。

### <a name="use-a-script-action-from-azure-resource-manager-templates"></a>從 Azure Resource Manager 範本使用指令碼動作

指令碼動作可搭配 Azure Resource Manager 範本使用。 如需範例，請參閱 [https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/](https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/)。

在此範例中，指令碼動作是使用下列程式碼加入：

```json
"scriptActions": [
    {
        "name": "setenvironmentvariable",
        "uri": "[parameters('scriptActionUri')]",
        "parameters": "headnode"
    }
]
```

如需如何部署範本的資訊，請參閱下列文件：

* [使用範本與 Azure PowerShell 來部署資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)

* [使用範本與 Azure CLI 來部署資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli)

### <a name="use-a-script-action-during-cluster-creation-from-azure-powershell"></a>在建立叢集期間從 Azure PowerShell 使用指令碼動作

本節中，您使用 [Add-AzureRmHDInsightScriptAction](https://docs.microsoft.com/powershell/module/azurerm.hdinsight/add-azurermhdinsightscriptaction) Cmdlet 叫用指令碼以自訂叢集。 在繼續之前，請確認您已安裝和設定 Azure PowerShell。 如需設定工作站以執行 HDInsight PowerShell Cmdlet 的相關資訊，請參閱 [安裝並設定 Azure PowerShell](/powershell/azure/overview)。

下列指令碼示範使用 PowerShell 建立叢集時如何套用指令碼動作：

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=5-90)]

建立叢集可能需要幾分鐘的時間。

### <a name="use-a-script-action-during-cluster-creation-from-the-hdinsight-net-sdk"></a>在建立叢集期間從 HDInsight .NET SDK 使用指令碼動作

HDInsight .NET SDK 提供用戶端程式庫，讓您輕鬆地從 .NET 應用程式使用 HDInsight。 如需程式碼範例，請參閱 [在 HDInsight 中使用 .NET SDK 建立以 Linux 為基礎的叢集](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md#use-script-action)。

## <a name="apply-a-script-action-to-a-running-cluster"></a>將指令碼動作套用到執行中的叢集

在本節中，了解如何套用指令碼動作至執行中的叢集。

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-portal"></a>從 Azure 入口網站將指令碼動作套用到執行中的叢集

1. 從 [Azure 入口網站](https://portal.azure.com)，選取您的 HDInsight 叢集。

2. 從 HDInsight 叢集概觀中，選取 [指令碼動作] 磚。

    ![指令碼動作磚](./media/hdinsight-hadoop-customize-cluster-linux/scriptactionstile.png)

   > [!NOTE]
   > 您也可以從 [設定] 區段中，依序選取 [所有設定] 和 [指令碼動作]。

3. 從 [指令碼動作] 區段的頂端，選取 [送出新的]。

    ![將指令碼加入執行中的叢集](./media/hdinsight-hadoop-customize-cluster-linux/add-script-running-cluster.png)

4. 使用 [Select a script] \(選取指令碼\) 項目選取預先製作的指令碼。 若要使用自訂指令碼，請選取 [Custom] \(自訂\)，然後為您的指令碼提供 [Name] \(名稱\) 和 [Bash script URI] \(Bash 指令碼 URI\)。

    ![在選取指令碼表單中加入指令碼](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    下表說明表單上的元素：

    | 屬性 | 值 |
    | --- | --- |
    | 選取指令碼 | 若要使用自己的指令碼，請選取 [自訂]。 否則，請選取提供的指令碼。 |
    | 名稱 |指定指令碼動作的名稱。 |
    | Bash 指令碼 URI |指定指令碼的 URI。 |
    | Head/Worker/Zookeeper |指定執行指令碼的節點 (**前端**、**背景工作**或 **ZooKeeper**)。 |
    | 參數 |如果指令碼要求，請指定參數。 |

    使用 [保存此指令碼動作] 項目，可確保在執行規模調整作業時套用此指令碼。

5. 最後，使用 [建立] 按鈕將指令碼套用到叢集。

### <a name="apply-a-script-action-to-a-running-cluster-from-azure-powershell"></a>從 Azure PowerShell 將指令碼動作套用到執行中的叢集

在繼續之前，請確認您已安裝和設定 Azure PowerShell。 如需設定工作站以執行 HDInsight PowerShell Cmdlet 的相關資訊，請參閱 [安裝並設定 Azure PowerShell](/powershell/azure/overview)。

下列範例示範如何將指令碼動作套用至執行中的叢集：

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=105-117)]

作業完成後，您會收到類似以下的訊息：

    OperationState  : Succeeded
    ErrorMessage    :
    Name            : Giraph
    Uri             : https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh
    Parameters      :
    NodeTypes       : {HeadNode, WorkerNode}

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-cli"></a>從 Azure CLI 將指令碼動作套用到執行中的叢集

在繼續之前，請確認您已安裝和設定 Azure CLI。 如需詳細資訊，請參閱[安裝 Azure 傳統 CLI](../cli-install-nodejs.md)。

[!INCLUDE [classic-cli-warning](../../includes/requires-classic-cli.md)]

1. 若要切換至 Azure Resource Manager 模式，請於命令列使用下列命令：

    ```bash
    azure config mode arm
    ```

2. 使用下列命令來驗證您的 Azure 訂用帳戶。

    ```bash
    azure login
    ```

3. 使用下列命令將指令碼動作套用至執行中的叢集

    ```bash
    azure hdinsight script-action create <clustername> -g <resourcegroupname> -n <scriptname> -u <scriptURI> -t <nodetypes>
    ```

    如果省略這個命令的參數，系統會提示您使用。 如果您以 `-u` 指定的指令碼接受參數，您可以使用 `-p` 參數來指定它們。

    有效的節點類型為 `headnode``workernode` 和 `zookeeper`。 如果指令碼應該要套用到多個節點類型，請指定以 ';' 分隔類型。 例如： `-n headnode;workernode`。

    若要保留指令碼，請新增 `--persistOnSuccess`。 您之後也可以使用 `azure hdinsight script-action persisted set` 來保存指令碼。

    作業完成後，您會收到類似下列文字的輸出：

        info:    Executing command hdinsight script-action create
        + Executing Script Action on HDInsight cluster
        data:    Operation Info
        data:    ---------------
        data:    Operation status:
        data:    Operation ID:  b707b10e-e633-45c0-baa9-8aed3d348c13
        info:    hdinsight script-action create command OK

### <a name="apply-a-script-action-to-a-running-cluster-using-rest-api"></a>使用 REST API 將指令碼動作套用到執行中的叢集

請參閱[在執行中的叢集執行指令碼動作](https://msdn.microsoft.com/library/azure/mt668441.aspx)。

### <a name="apply-a-script-action-to-a-running-cluster-from-the-hdinsight-net-sdk"></a>從 HDInsight .NET SDK 將指令碼動作套用到執行中的叢集

如需使用 .NET SDK 將指令碼套用至叢集的範例，請參閱 [https://github.com/Azure-Samples/hdinsight-dotnet-script-action](https://github.com/Azure-Samples/hdinsight-dotnet-script-action)。

## <a name="view-history-promote-and-demote-script-actions"></a>檢視歷程記錄、升級和降級指令碼動作

### <a name="using-the-azure-portal"></a>使用 Azure 入口網站

1. 從 [Azure 入口網站](https://portal.azure.com)，選取您的 HDInsight 叢集。

2. 從 HDInsight 叢集概觀中，選取 [指令碼動作] 磚。

    ![指令碼動作磚](./media/hdinsight-hadoop-customize-cluster-linux/scriptactionstile.png)

   > [!NOTE]
   > 您也可以從 [設定] 區段中，依序選取 [所有設定] 和 [指令碼動作]。

4. 此叢集的指令碼歷程記錄會顯示在 [指令碼動作] 區段。 此資訊包含持續性指令碼清單。 在以下的螢幕擷取畫面中，您可以看到 Solr 指令碼已在此叢集上執行。 此螢幕擷取畫面不會顯示任何持續性指令碼。

    ![[指令碼動作] 區段](./media/hdinsight-hadoop-customize-cluster-linux/script-action-history.png)

5. 選取歷程記錄中的指令碼，便會顯示此指令碼的 [屬性] 區段。 從視窗的頂端，您可以重新執行指令碼或將其升階。

    ![指令碼動作屬性](./media/hdinsight-hadoop-customize-cluster-linux/promote-script-actions.png)

6. 您也可以使用 [指令碼動作] 區段上項目右邊的 **...** 來執行動作。

    ![指令碼動作...使用情況](./media/hdinsight-hadoop-customize-cluster-linux/deletepromoted.png)

### <a name="using-azure-powershell"></a>使用 Azure PowerShell

| 使用下列... | 來 ... |
| --- | --- |
| Get-AzureRmHDInsightPersistedScriptAction |擷取持續性指令碼動作的資訊 |
| Get-AzureRmHDInsightScriptActionHistory |擷取已套用到叢集的指令碼動作歷程記錄，或特定指令碼的詳細資料 |
| Set-AzureRmHDInsightPersistedScriptAction |將臨時性指令碼動作升級為持續性指令碼動作 |
| Remove-AzureRmHDInsightPersistedScriptAction |將持續性指令碼動作降級為臨時性指令碼動作 |

> [!IMPORTANT]
> 使用 `Remove-AzureRmHDInsightPersistedScriptAction` 無法復原指令碼執行的動作。 此 Cmdlet 只會移除持續性旗標。

下列範例指令碼示範如何使用 Cmdlet 來升級而後降級指令碼。

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=123-140)]

### <a name="using-the-azure-classic-cli"></a>使用 Azure 傳統 CLI

| 使用下列... | 來 ... |
| --- | --- |
| `azure hdinsight script-action persisted list <clustername>` |擷取保存的指令碼動作清單 |
| `azure hdinsight script-action persisted show <clustername> <scriptname>` |擷取特定的保存指令碼動作資訊 |
| `azure hdinsight script-action history list <clustername>` |擷取已套用到叢集的指令碼動作歷程記錄 |
| `azure hdinsight script-action history show <clustername> <scriptname>` |擷取特定指令碼動作的資訊 |
| `azure hdinsight script action persisted set <clustername> <scriptexecutionid>` |將臨時性指令碼動作升級為持續性指令碼動作 |
| `azure hdinsight script-action persisted delete <clustername> <scriptname>` |將持續性指令碼動作降級為臨時性指令碼動作 |

> [!IMPORTANT]
> 使用 `azure hdinsight script-action persisted delete` 無法復原指令碼執行的動作。 此 Cmdlet 只會移除持續性旗標。

### <a name="using-the-hdinsight-net-sdk"></a>使用 HDInsight .NET SDK

如需使用 .NET SDK 從叢集擷取指令碼歷程記錄、升級或降級指令碼的範例，請參閱 [https://github.com/Azure-Samples/hdinsight-dotnet-script-action](https://github.com/Azure-Samples/hdinsight-dotnet-script-action)。

> [!NOTE]
> 這個範例也示範如何使用 .NET SDK 安裝 HDInsight 應用程式。

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>支援在 HDInsight 叢集上使用開放原始碼軟體

Microsoft Azure HDInsight 服務使用以 Hadoop 為中心的開放原始碼技術生態系統。 Microsoft Azure 可為開放原始碼技術提供一般層級的支援。 如需詳細資訊，請參閱 [Azure 支援常見問題集網站](https://azure.microsoft.com/support/faq/)中的**支援範圍**一節。 HDInsight 服務針對內建元件提供額外等級的支援。

HDInsight 服務中有兩種類型的開放原始碼元件可用：

* **內建元件** - 這些元件預先安裝在 HDInsight 叢集上，並且提供叢集的核心功能。 例如，YARN ResourceManager、Hive 查詢語言 (HiveQL) 及 Mahout 程式庫都屬於這個類別。 叢集元件的完整清單可於 [HDInsight 所提供 Hadoop 叢集版本的新功能](hdinsight-component-versioning.md)中取得。
* **自訂元件** - 身為叢集使用者的您可以安裝社群中可用或是您建立的任何元件，或者在工作負載中使用。

> [!WARNING]
> 對隨 HDInsight 叢集提供的元件會有完整支援。 Microsoft 支援服務可協助隔離和解決這些元件的相關問題。
>
> 自訂元件則獲得商務上合理的支援，協助您進一步疑難排解問題。 Microsoft 支援服務可能可以解決問題，也可能要求您利用可用的開放原始碼技術管道，找到該技術的深度專業知識。 例如，有許多社群網站可供使用，例如：[MSDN 的 HDInsight 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight) \(英文\)、[http://stackoverflow.com](http://stackoverflow.com) \(英文\)。 此外，Apache 專案在 [http://apache.org](http://apache.org) 上也有專案網站，例如 [Hadoop](http://hadoop.apache.org/)。

HDInsight 服務提供數種方式以使用自訂元件。 無論元件如何使用或如何安裝在叢集上，都適用相同層級的支援。 下列清單描述自訂元件可用於 HDInsight 叢集之最常見方式：

1. 工作提交 - Hadoop 或其他類型的工作，執行或使用可以提交給叢集的自訂元件。

2. 叢集自訂 - 在叢集建立期間，您可以指定額外設定以及會安裝在叢集節點上的自訂元件。

3. 範例 - 對於熱門自訂元件，Microsoft 和其他提供者可能會提供如何在 HDInsight 叢集上使用這些元件的範例。 提供這些範例，但是沒有支援。

## <a name="troubleshooting"></a>疑難排解

您可以使用 Ambari Web UI 來檢視指令碼動作所記錄的資訊。 如果指令碼在叢集建立期間失敗，則與該叢集相關聯的預設儲存體帳戶中也會有記錄檔。 本節提供關於如何使用這兩個選項擷取記錄檔的資訊。

### <a name="using-the-ambari-web-ui"></a>使用 Ambari Web UI

1. 在瀏覽器中，瀏覽至 https://CLUSTERNAME.azurehdinsight.net。 將 CLUSTERNAME 取代為 HDInsight 叢集的名稱。

    出現提示時，輸入管理帳戶名稱 (admin) 和叢集的密碼。 您可能必須在 Web 表單中重新輸入系統管理員認證。

2. 在頁面頂端的列中，選取 **ops** 項目。 隨即顯示透過 Ambari 在叢集上執行的目前和先前作業的清單。

    ![Ambari Web UI 列與選取的 ops](./media/hdinsight-hadoop-customize-cluster-linux/ambari-nav.png)

3. 尋找在 [作業] 欄位中有 **run\_customscriptaction** 的項目。 這些項目是在執行指令碼動作時建立的。

    ![作業的螢幕擷取畫面](./media/hdinsight-hadoop-customize-cluster-linux/ambariscriptaction.png)

    若要檢視 STDOUT 和 STDERR 輸出，請選取 run\customscriptaction 項目，並向下鑽研連結。 執行指令碼時會產生此輸出，其中可能包含實用的資訊。

### <a name="access-logs-from-the-default-storage-account"></a>從預設的儲存體帳戶存取記錄檔

如果叢集建立因指令碼錯誤而失敗，記錄會保存在叢集存放區帳戶中。

* 儲存體記錄檔位於 `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\CLUSTER_NAME\DATE`。

    ![作業的螢幕擷取畫面](./media/hdinsight-hadoop-customize-cluster-linux/script_action_logs_in_storage.png)

    在此目錄下，記錄檔會個別針對前端節點、背景工作節點和 Zookeeper 節點進行組織。 部分範例如下：

    * **前端節點** - `<uniqueidentifier>AmbariDb-hn0-<generated_value>.cloudapp.net`

    * **背景工作節點** - `<uniqueidentifier>AmbariDb-wn0-<generated_value>.cloudapp.net`

    * **Zookeeper 節點** - `<uniqueidentifier>AmbariDb-zk0-<generated_value>.cloudapp.net`

* 對應主機的所有 stdout 和 stderr 都會上傳至儲存體帳戶。 每個指令碼動作分別有一個 **output-\*.txt** 和 **errors-\*.txt**。 output-*.txt 檔案包含在主機上執行之指令碼的 URI 相關資訊。 以下文字是此資訊的範例：

        'Start downloading script locally: ', u'https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh'

* 您有可能重複建立具有相同名稱的指令碼動作叢集。 在這種情況下，您可以根據 DATE 資料夾名稱來區分相關的記錄檔。 例如，在不同的日期為叢集 (mycluster) 建立的資料夾結構會類似下列記錄檔項目：

    `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-04` `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-05`

* 如果您在同一天建立具有相同名稱的指令碼動作叢集，您可以使用唯一的前置詞來識別相關的記錄檔。

* 如果您在接近 12:00 AM (午夜) 時建立叢集，記錄檔可能會橫跨兩天。 在這種情況下，您會看到相同的叢集有兩個不同的日期資料夾。

* 將記錄檔上傳至預設容器可能需要 5 分鐘，特別是大型叢集。 因此，如果您想要存取記錄檔，則不應在指令碼動作失敗時立即刪除叢集。

### <a name="ambari-watchdog"></a>Ambari 看門狗

> [!WARNING]
> 請勿變更以 Linux 為基礎之 HDInsight 叢集上的 Ambari 看門狗 (hdinsightwatchdog) 密碼。 變更此帳戶的密碼會破壞在 HDInsight 叢集上執行新指令碼動作的能力。

### <a name="cant-import-name-blobservice"></a>無法匯入名稱 BlobService

__徵兆__：指令碼動作失敗。 在 Ambari 中檢視作業時，會顯示類似下列錯誤的訊息：

```
Traceback (most recent call list):
  File "/var/lib/ambari-agent/cache/custom_actions/scripts/run_customscriptaction.py", line 21, in <module>
    from azure.storage.blob import BlobService
ImportError: cannot import name BlobService
```

__原因__︰如果您升級隨附於 HDInsight 叢集的 Python Azure 儲存體用戶端，就會發生此錯誤。 HDInsight 需要 Azure 儲存體用戶端 0.20.0。

__解決方案__︰若要解決這個錯誤，請使用 `ssh` 以手動方式連線到每個叢集節點，然後使用下列命令重新安裝正確的儲存體用戶端版本︰

```bash
sudo pip install azure-storage==0.20.0
```

如需使用 SSH 連線到叢集的相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

### <a name="history-doesnt-show-scripts-used-during-cluster-creation"></a>歷程記錄不會顯示在叢集建立期間使用的指令碼

如果您在 2016 年 3 月 15 日之前建立叢集，就可能不會在指令碼動作歷程記錄中看到任何項目。 調整叢集大小會導致指令碼顯示在指令碼動作歷程記錄中。

有兩種例外狀況：

* 如果您的叢集在 2015 年 9 月 1 日之前建立。 此日期是引進指令碼動作的日期。 在此日期之前建立的任何叢集可能都未使用指令碼動作建立叢集。

* 如果您在叢集建立期間使用了多個指令碼動作，並將相同的名稱、相同的 URI 使用於多個指令碼，但將不同的參數使用於多個指令碼。 在這些情況下，您會收到下列錯誤：

    由於現有指令碼的指令碼名稱發生衝突，所以無法在此叢集上執行任何新的指令碼動作。 建立叢集時所提供的指令碼名稱全都必須是唯一的名稱。 既有的指令碼會在調整大小時執行。

## <a name="next-steps"></a>後續步驟

* [開發 HDInsight 的指令碼動作指令碼](hdinsight-hadoop-script-actions-linux.md)
* [在 HDInsight 叢集上安裝及使用 Solr](hdinsight-hadoop-solr-install-linux.md)
* [在 HDInsight 叢集上安裝及使用 Giraph](hdinsight-hadoop-giraph-install-linux.md)
* [在 HDInsight 叢集新增儲存體](hdinsight-hadoop-add-storage.md)

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-linux/HDI-Cluster-state.png "叢集建立期間的階段"
