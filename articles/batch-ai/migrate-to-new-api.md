---
title: 遷移至更新的 Azure Batch AI API | Microsoft Docs
description: 如何更新 Azure Batch AI 程式碼和指令碼以使用工作區及實驗資源
services: batch-ai
documentationcenter: na
author: dlepow
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.custom: ''
ms.service: batch-ai
ms.workload: ''
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 06/08/2018
ms.author: danlep
ms.openlocfilehash: c5e4c1569464d2e204edf13fe7534d80780524e8
ms.sourcegitcommit: 1438b7549c2d9bc2ace6a0a3e460ad4206bad423
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36294953"
---
# <a name="migrate-to-the-updated-batch-ai-api"></a>遷移至更新的批次 Batch AI API

Batch AI REST API 2018-05-01 版和相關的 Batch AI SDK 與工具已引入重大變更和新功能。

如果您已使用舊版的 Batch AI API，本文會說明如何修改程式碼和指令碼以使用新的 API。 

## <a name="whats-changing"></a>有哪些變更項目？

為回應客戶的意見反應，我們將移除作業數量的限制，並讓您可以更輕鬆地管理 Batch AI 資源。 新 API 會導入了兩個新資源：「工作區」和「實驗」。 在實驗底下收集所有相關的訓練作業，並在工作區底下組織所有相關的 Batch AI 資源 (叢集、檔案伺服器、實驗、作業)。  

* **工作區** - 所有 Batch AI 資源類型的最高層集合。 工作區中包含叢集和檔案伺服器。 工作區通常會分開屬於不同群組或專案的工作。 例如，您可能會有開發和測試工作區。 每個訂用帳戶可只需要有限的工作區數目。 

* **實驗** - 相關作業的集合，您可以同時查詢和管理這些作業。 例如，可使用實驗來群組作為超參數調整掃描一部份執行的所有作業。 

下圖顯示資源階層的範例。 

![](./media/migrate-to-new-api/batch-ai-resource-hierarchy.png)

## <a name="monitor-and-manage-existing-resources"></a>監視及管理現有資源
在您建立 Batch AI 叢集、作業或檔案伺服器所在的每個資源群組中，Batch AI 服務會建立名為 `migrated-<region>` 的工作區 (例如 `migrated-eastus`) 和名為 `migrated` 的實驗。 若要存取先前建立的作業、叢集或檔案伺服器，您必須使用已遷移的工作區和實驗。 

### <a name="portal"></a>入口網站 
若要使用入口網站存取先前建立的作業、叢集或檔案伺服器，請先選取 `migrated-<region>` 工作區。 選取工作區之後，可執行調整大小或刪除叢集等作業，然後檢視作業狀態和輸出。 

### <a name="sdks"></a>SDK 
若要透過 Batch AI SDK 存取先前建立的作業、叢集或檔案伺服器，請提供工作區名稱和實驗名稱參數。 

如果您使用 Python SDK，相關變更如下列範例所示。 其他 Batch AI SDK 中的變更也是類似的。 


#### <a name="get-old-cluster"></a>取得舊叢集 

```python
cluster = client.clusters.get(resource_group_name, 'migrated-<region>', cluster_name)
```

#### <a name="delete-old-cluster"></a>刪除舊叢集 

```python
client.clusters.delete(resource_group_name, 'migrated-<region>', cluster_name)
```

#### <a name="get-old-file-server"></a>取得舊檔案伺服器 

```python
cluster = client.fileservers.get(resource_group_name, 'migrated-<region>', fileserver_name)
```

#### <a name="delete-old-file-server"></a>刪除舊檔案伺服器  

```python
client.fileservers.delete(resource_group_name, 'migrated-<region>', fileserver_name)
``` 


#### <a name="get-old-job"></a>取得舊作業 

```python
cluster = client.jobs.get(resource_group_name, 'migrated-<region>', 'migrated', job_name)
```

#### <a name="delete-old-job"></a>刪除舊作業

```python
client.jobs.delete(resource_group_name, 'migrated-<region>', 'migrated', job_name)
```
 
### <a name="azure-cli"></a>Azure CLI 
 
如果使用 Azure CLI 來存取先前建立的作業、叢集或檔案伺服器，請使用 `-w` 和 `-e` 參數來提供工作區和實驗名稱。 


#### <a name="get-old-cluster"></a>取得舊叢集

```azurecli
az batchai cluster show -g resource-group-name -w migrated-<region> -n cluster-name
```


#### <a name="delete-old-cluster"></a>刪除舊叢集 

```azurecli
az batchai cluster delete -g resource-group-name -w migrated-<region> -n cluster-name
```

#### <a name="get-old-file-server"></a>取得舊檔案伺服器

```azurecli
az batchai file-server show -g resource-group-name -w migrated-<region> -n fileserver-name
```


#### <a name="delete-old-file-server"></a>刪除舊檔案伺服器 

```azurecli
az batchai file-server delete -g resource-group-name -w migrated-<region> -n fileserver-name
``` 


#### <a name="get-old-job"></a>取得舊作業

```azurecli
az batchai job show -g resource-group-name -w migrated-<region> -e migrated -n job-name
```


#### <a name="delete-old-job"></a>刪除舊作業 

```azurecli
az batchai job delete -g resource-group-name -w migrated-<region> -e migrated -n job-name
``` 

## <a name="create-batch-ai-resources"></a>建立 Batch AI 資源 
 
如果您使用其中一個現有的 Batch AI SDK，您可以繼續使用它來建立 Batch AI 資源 (作業、叢集或檔案伺服器)，而不需要變更程式碼。 不過，在升級到新的 SDK 之後，您需要進行下列變更。
 
### <a name="create-workspace"></a>建立工作區 
視您的狀況而定，您可以透過入口網站或 CLI，以手動方式一次建好工作區。 如果您使用 CLI，請使用下列命令建立工作區： 

```azurecli
az batchai workspace create -g resource-group-name -n workspace-name
```

### <a name="create-experiment"></a>建立實驗 


視您的狀況而定，您可以透過入口網站或 CLI，以手動方式一次建好實驗。 如果您使用 CLI，請使用下列命令建立實驗： 

```azurecli
az batchai experiment create -g resource-group-name -w workspace-name -n experiment-name

```

### <a name="create-clusters-file-servers-and-jobs"></a>建立叢集、檔案伺服器和作業
 
如果您使用入口網站來建立作業、叢集或檔案伺服器，入口網站會在建立體驗中引導您提供工作區名稱和實驗名稱參數。

若要透過更新的 SDK 建立作業、叢集或檔案伺服器，請提供工作區名稱參數。 如果您使用 Python SDK，相關變更如下列範例所示。 以您先前建立的工作區和實驗取代 workspace_name 和 experiment_name。 其他 Batch AI SDK 中的變更也是類似的。 


#### <a name="create-cluster"></a>建立叢集 

```python
_ = client.clusters.create(resource_group_name, workspace_name, cluster_name, cluster_create_parameters).result()
```

#### <a name="create-file-server"></a>建立檔案伺服器 

```python
_ = client.fileservers.create(resource_group_name, workspace_name, fileserver_name, fileserver_create_parameters).result()
```

#### <a name="create-job"></a>建立作業 

```python
_ = client.jobs.create(resource_group_name, workspace_name, experiment_name, job_name job_create_parameters).result()
```


## <a name="next-steps"></a>後續步驟

* 請參閱 Batch AI API 的參考：[CLI](/cli/azure/batchai)、[.NET](/dotnet/api/overview/azure/batchai)、[Java](/java/api/overview/azure/batchai)、[Node.js](/javascript/api/overview/azure/batchai)、[Python](/python/api/overview/azure/batchai) 和 [REST](/rest/api/batchai)