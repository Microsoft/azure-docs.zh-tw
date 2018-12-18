---
title: 發佈 Azure HDInsight 應用程式
description: 了解如何建立 HDInsight 應用程式，然後將其發佈到 Azure Marketplace。
services: hdinsight
author: jasonwhowell
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 05/14/2018
ms.author: jasonh
ms.openlocfilehash: 1cfefc3463fee6e432170a16034175bb4267fad5
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43103565"
---
# <a name="publish-an-hdinsight-application-in-the-azure-marketplace"></a>將 HDInsight 應用程式發佈到 Azure Marketplace
您可以在以 Linux 為基礎的 HDInsight 叢集上安裝 Azure HDInsight 應用程式。 在此文章中，您會學習如何將 HDInsight 應用程式發佈到 Azure Marketplace。 如需發佈到 Azure Marketplace 的一般資訊，請參閱[將供應項目發佈到 Azure Marketplace](../marketplace/marketplace-publishers-guide.md)。

HDInsight 應用程式會使用「自備授權 (BYOL)」模型。 在 BYOL 案例中，應用程式提供者負責將應用程式授權給應用程式使用者。 應用程式使用者只需支付他們建立的 Azure 資源費用，例如在 HDInsight 叢集以及叢集的 VM 和節點。 目前，Azure 不經手應用程式本身的計費。

如需詳細資訊，請參閱這些 HDInsight 應用程式相關文章：

* [安裝 HDInsight 應用程式](hdinsight-apps-install-applications.md)。 了解如何在您的叢集上安裝 HDInsight 應用程式。
* [安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。 了解如何安裝和測試自訂 HDInsight 應用程式。

## <a name="prerequisites"></a>必要條件
若要將自訂應用程式提交至 Marketplace，首先，[建立並測試您的自訂應用程式](hdinsight-apps-install-custom-applications.md)。

您還必須註冊開發人員帳戶。 如需詳細資訊，請參閱[將供應項目發佈到 Azure Marketplace](../marketplace/marketplace-publishers-guide.md) 和[建立 Microsoft 開發人員帳戶](../marketplace/marketplace-publishers-guide.md)。

## <a name="define-the-application"></a>定義應用程式
將應用程式發佈到 Marketplace 包括兩個步驟。 首先，定義 createUiDef.json 檔案。 CreateUiDef.json 檔案表示您的應用程式與哪些叢集相容。 接著，從 Azure 入口網站發佈範本。 以下是範例 createUiDef.json 檔案：

```json
{
    "handler": "Microsoft.HDInsight",
    "version": "0.0.1-preview",
    "clusterFilters": {
        "types": ["Hadoop", "HBase", "Storm", "Spark"],
        "versions": ["3.6"]
    }
}
```

| 欄位 | 說明 | 可能的值 |
| --- | --- | --- |
| types |與應用程式相容的叢集類型。 |Hadoop、HBase、Storm、Spark (或這些類型的任意組合) |
| versions |與應用程式相容的 HDInsight 叢集類型。 |3.4 |

## <a name="application-installation-script"></a>應用程式安裝指令碼
當應用程式安裝在叢集上時 (現有叢集或新的叢集)，會建立邊緣節點。 應用程式安裝指令碼會在邊緣節點上執行。

  > [!IMPORTANT]
  > 應用程式安裝指令碼的名稱在特定叢集上必須是唯一的。 指令碼名稱必須具有以下格式：
  > 
  > "name": "[concat('hue-install-v0','-' ,uniquestring(‘applicationName’)]"
  > 
  > 指令碼名稱有三個部分：
  > 
  > * 指令碼名稱前置詞必須包含應用程式名稱或與該應用程式相關的名稱。
  > * 連字號，以方便閱讀。
  > * 唯一的字串函式，並以應用程式名稱作為參數。
  > 
  > 在保存的指令碼動作清單中，上述範例會顯示為 **hue-install-v0-4wkahss55hlas**。 請參閱[範例 JSON 承載](https://raw.githubusercontent.com/hdinsight/Iaas-Applications/master/Hue/azuredeploy.json)。
  > 

安裝指令碼必須具有下列特性：
* 指令碼具有等冪性。 指令碼的多個呼叫產生相同的結果。
* 指令碼是正確版本。 當您升級或測試變更時，針對指令碼使用不同的位置。 這可確保安裝應用程式的客戶不會受到您的更新或測試影響。 
* 指令碼在每個點都有適當的記錄。 指令碼記錄檔通常是對應用程式安裝問題進行偵錯的唯一方法。
* 外部服務或資源的呼叫有足夠的重試次數，讓安裝不會受到暫時性網路問題的影響。
* 如果您的指令碼在節點上啟動服務，服務會受到監視並設定為在節點重新開機時自動啟動。

## <a name="package-the-application"></a>封裝應用程式
建立 .zip 檔案，其中包含安裝 HDInsight 應用程式所需的所有檔案。 使用 .zip 檔案來[發佈應用程式](#publish-application)。 .zip 檔案包含下列檔案：

* [createUiDefinition.json](#define-application)
* mainTemplate.json (如需範例，請參閱[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)。)
* 所有必要的指令碼

> [!NOTE]
> 您可以在任何可公開存取的端點上，裝載應用程式檔案 (包括任何 Web 應用程式檔案)。
> 

## <a name="publish-the-application"></a>發佈應用程式
若要安裝 HDInsight 應用程式：

1. 登入 [Azure 發佈](https://publish.windowsazure.com/)。
2. 在左窗格中，選取 [解決方案範本]。
3. 輸入標題，然後選取 [建立新的解決方案範本]。
4. 如果您尚未註冊您的組織，請選取 [建立開發人員中心帳戶並加入 Azure 方案]。  如需詳細資訊，請參閱 [建立 Microsoft 開發人員帳戶](../marketplace/marketplace-publishers-guide.md)。
5. 選取 [定義一些拓撲以便開始使用] 。 方案範本是所有其拓撲的「父項」。 您可以在一個供應項目或解決方案範本中定義多個拓撲。 當供應項目推送到預備環境時，它的所有拓撲也會一起推入。 
6. 輸入拓撲名稱，然後選取 [+]。
7. 輸入新的版本，然後選取 [+]。
8. 上傳您在[封裝應用程式](#package-application)時建立的 .zip 檔案。  
9. 選取 [要求認證]。 Microsoft 認證團隊會檢閱檔案並認證拓撲。

## <a name="next-steps"></a>後續步驟
* 了解如何在您的叢集上[安裝 HDInsight 應用程式](hdinsight-apps-install-applications.md)。
* 了解如何[安裝自訂 HDInsight 應用程式](hdinsight-apps-install-custom-applications.md)，以及如何將未發佈的 HDInsight 應用程式部署到 HDInsight。
* 了解如何[使用指令碼動作來自訂以 Linux 為基礎的 HDInsight 叢集](hdinsight-hadoop-customize-cluster-linux.md)，以及新增更多應用程式。 
* 了解如何[使用 Azure Resource Manager 範本在 HDInsight 中建立以 Linux 為基礎的 Hadoop 叢集](hdinsight-hadoop-create-linux-clusters-arm-templates.md)。
* 了解如何[在 HDInsight 中使用空白邊緣節點](hdinsight-apps-use-edge-node.md)，以存取 HDInsight 叢集、測試 HDInsight 應用程式，以及裝載 HDInsight 應用程式。

