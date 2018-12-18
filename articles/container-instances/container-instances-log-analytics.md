---
title: 使用 Azure Log Analytics 的容器執行個體記錄
description: 了解如何將容器輸出 (STDOUT 與 STDERR) 傳送至 Azure Log Analytics。
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: overview
ms.date: 07/17/2018
ms.author: marsma
ms.openlocfilehash: 9d967cc7509ef0d558327da2299b0478b62de430
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46960714"
---
# <a name="container-instance-logging-with-azure-log-analytics"></a>使用 Azure Log Analytics 的容器執行個體記錄

Log Analytics 工作區提供集中式位置，不僅可讓您從 Azure 資源儲存及查詢記錄資料，對於內部部署資源與其他雲端中的資源，也可執行這些作業。 Azure 容器執行個體包含將資料傳送至 Log Analytics 的內建支援。

若要將容器執行個體資料傳送至 Log Analytics，您必須使用 Azure CLI (或 Cloud Shell) 和 YAML 檔案建立容器群組。 以下幾節將說明如何建立已啟用記錄的容器群組，以及如何查詢記錄。

## <a name="prerequisites"></a>必要條件

若要在您的容器執行個體中啟用記錄，您必須具備下列項目：

* [Log Analytics 工作區](../log-analytics/log-analytics-quick-create-workspace.md)
* [Azure CLI](/cli/azure/install-azure-cli) (或 [Cloud Shell](/azure/cloud-shell/overview))

## <a name="get-log-analytics-credentials"></a>取得 Log Analytics 認證

Azure 容器執行個體必須具備將資料傳送至 Log Analytics 工作區的權限。 若要授與此權限並啟用記錄，您必須在建立容器群組時提供 Log Analytics 工作區識別碼，以及它的其中一個金鑰 (主要或次要)。

若要取得 Log Analytics 工作區識別碼和主要金鑰：

1. 在 Azure 入口網站中瀏覽至您的 Log Analytics 工作區
1. 在 [設定] 下，選取 [進階設定]
1. 選取 [連線的來源] > [Windows 伺服器] (或 **Linux 伺服器** - 兩者的識別碼和金鑰是相同的)
1. 記下：
   * **工作區識別碼**
   * **主要金鑰**

## <a name="create-container-group"></a>建立容器群組

現在您已有 Log Analytics 工作區識別碼和主要金鑰，接下來即可建立已啟用記錄的容器群組。

下列範例示範兩種以單一 [fluentd][fluentd] 容器建立容器群組的方式：Azure CLI，以及搭配使用 YAML 範本的 Azure CLI。 Fluentd 容器在其預設組態中會產生數行輸出。 此輸出會傳送到您的 Log Analytics 工作區，因此很適合用來示範記錄的檢視和查詢。

### <a name="deploy-with-azure-cli"></a>使用 Azure CLI 進行部署

若要使用 Azure CLI 進行部署，請在 [az container create][az-container-create] 命令中指定 `--log-analytics-workspace` 和 `--log-analytics-workspace-key` 參數。 在執行下列命令之前，請先將兩個工作區值取代為您在先前的步驟中取得的值 (並更新資源群組名稱)。

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainergroup001 \
    --image fluent/fluentd \
    --log-analytics-workspace <WORKSPACE_ID> \
    --log-analytics-workspace-key <WORKSPACE_KEY>
```

### <a name="deploy-with-yaml"></a>使用 YAML 進行部署

如果您想要使用 YAML 部署容器群組，請使用此方法。 下列 YAML 會定義具有單一容器的容器群組。 請將 YAML 複製到新檔案中，然後將 `LOG_ANALYTICS_WORKSPACE_ID` 和 `LOG_ANALYTICS_WORKSPACE_KEY` 取代為您在先前的步驟中取得的值。 將檔案儲存為 **deploy-aci.yaml**。

```yaml
apiVersion: 2018-06-01
location: eastus
name: mycontainergroup001
properties:
  containers:
  - name: mycontainer001
    properties:
      environmentVariables: []
      image: fluent/fluentd
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
  diagnostics:
    logAnalytics:
      workspaceId: LOG_ANALYTICS_WORKSPACE_ID
      workspaceKey: LOG_ANALYTICS_WORKSPACE_KEY
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

接下來，執行下列命令以部署容器群組；請將 `myResourceGroup` 取代為您訂用帳戶中的資源群組 (或先建立名為 "myResourceGroup" 的資源群組)：

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainergroup001 --file deploy-aci.yaml
```

在發出命令不久後，您應該就會收到 Azure 的回應，其中包含部署詳細資料。

## <a name="view-logs-in-log-analytics"></a>檢視 Log Analytics 中的記錄

在您部署容器群組後，可能需要幾分鐘的時間 (最多 10 分鐘)，第一個記錄項目才會出現在 Azure 入口網站中。 若要檢視容器群組的記錄，請開啟 Log Analytics 工作區，然後：

1. 在 [OMS 工作區] 概觀中，選取 [記錄搜尋]
1. 在 [再多試一些查詢] 下，選取 [所有收集的資料] 連結

您應該會看到 `search *` 查詢所顯示的數個結果。 如果一開始未看到任何結果，請等候數分鐘，然後再選取 [執行] 按鈕重新執行查詢。 根據預設，記錄項目會顯示在 [清單] 檢視中 -- 選取 [資料表] 可查看簡略格式的記錄檔項目。 接著，您可以展開資料列來查看個別記錄項目的內容。

![Azure 入口網站中的記錄搜尋結果][log-search-01]

## <a name="query-container-logs"></a>查詢容器記錄

Log Analytics 包含涵蓋範圍廣大的[查詢語言][query_lang]，可從可能高數千行的記錄輸出中提取資訊。

Azure 容器執行個體記錄代理程式會將項目傳送至 Log Analytics 工作區中的 `ContainerInstanceLog_CL` 資料表。 查詢的基本結構是一個來源資料表 (`ContainerInstanceLog_CL`)，後面接著一系列由管道字元 (`|`) 隔開的運算子。 您可以鏈結數個運算子，以找出更精確的結果及執行進階函式。

若要查看範例查詢結果，請將下列查詢貼到查詢文字方塊中 (在 [顯示舊版語言轉換器] 下方)，然後選取 [執行] 按鈕以執行查詢。 此查詢會顯示 [訊息] 欄位中包含「警告」一詞的所有記錄項目：

```query
ContainerInstanceLog_CL
| where Message contains("warn")
```

此外也支援更複雜的查詢。 例如，下列查詢只會顯示過去一小時內為 "mycontainergroup001" 容器群組產生的記錄項目：

```query
ContainerInstanceLog_CL
| where (ContainerGroup_s == "mycontainergroup001")
| where (TimeGenerated > ago(1h))
```

## <a name="next-steps"></a>後續步驟

### <a name="log-analytics"></a>Log Analytics

如需關於在 Azure Log Analytics 中查詢記錄和設定警示的詳細資訊，請參閱：

* [了解 Log Analytics 中的記錄搜尋](../log-analytics/log-analytics-log-search.md)
* [Azure 監視器中的整合警示](../monitoring-and-diagnostics/monitoring-overview-unified-alerts.md)

### <a name="monitor-container-cpu-and-memory"></a>監視容器 CPU 和記憶體

如需監視容器執行個體 CPU 和記憶體資源的相關資訊，請參閱：

* [在 Azure 容器執行個體中監視容器資源](container-instances-monitor.md)。

<!-- IMAGES -->
[log-search-01]: ./media/container-instances-log-analytics/portal-query-01.png

<!-- LINKS - External -->
[fluentd]: https://hub.docker.com/r/fluent/fluentd/
[query_lang]: https://aka.ms/LogAnalyticsLanguage

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create