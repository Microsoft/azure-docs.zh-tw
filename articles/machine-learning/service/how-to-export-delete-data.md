---
title: 匯出或刪除工作區資料 - Azure Machine Learning | Microsoft Docs
description: 在 Azure Machine Learning 中，您可以使用 Azure 入口網站、CLI、SDK 及已驗證的 REST API 以匯出或刪除工作區。 本文將說明如何做到。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
author: ph-com
ms.author: pahusban
ms.date: 09/24/2018
ms.openlocfilehash: aef7bae7d2c511766bdbebbf1180e19263d7ccec
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46986297"
---
# <a name="export-or-delete-your-workspace-data-in-machine-learning"></a>匯出或刪除 Machine Learning 中的工作區資料

在 Azure Machine Learning 中，您可以使用已驗證的 REST API，匯出或刪除工作區。 本文會說明做法。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="control-your-workspace-data"></a>控制您的工作區資料
透過 Azure 入口網站、CLI、SDK 和經過驗證的 REST API，可以匯出和刪除 Azure Machine Learning 服務所儲存的產品中資料。 透過 Azure 隱私權入口網站可以存取遙測資料。 

在 Azure Machine Learning 服務中，個人資料會包含執行歷程記錄文件中的使用者資訊，以及使用者與服務部分互動的遙測記錄。

## <a name="delete-workspace-data-with-the-rest-api"></a>使用 REST API 來刪除工作區資料 

為了刪除資料，可以使用 HTTP DELETE 動詞來進行下列 API 呼叫。 這些可透過在要求中指定 `Authorization: Bearer <arm-token>` 標頭來進行授權，其中 `<arm-token>` 是 `https://management.core.windows.net/` 端點的 AAD 存取權杖。  

若要了解如何取得此權杖及呼叫 Azure 端點，請參閱 [Azure REST API 文件](https://docs.microsoft.com/rest/api/azure/) \(英文\)。  

在下列範例中，請以決定相關資源的執行個體名稱取代 {} 中的文字。

### <a name="delete-an-entire-workspace"></a>刪除整個工作區

使用此呼叫以刪除整個工作區。  
> [!WARNING]
> 將刪除所有資訊，而且工作區將不再可用。

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}?api-version=2018-03-01-preview

### <a name="delete-models"></a>刪除模型

使用此呼叫來取得模型及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/models?api-version=2018-03-01-preview

可以藉由下列方式刪除個別模型：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/models/{id}?api-version=2018-03-01-preview

### <a name="delete-assets"></a>刪除資產

使用此呼叫來取得資產及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/assets?api-version=2018-03-01-preview

可以藉由下列方式刪除個別資產：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/assets/{id}?api-version=2018-03-01-preview

### <a name="delete-images"></a>刪除映像

使用此呼叫來取得映像及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/images?api-version=2018-03-01-preview

可以藉由下列方式刪除個別映像：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/images/{id}?api-version=2018-03-01-preview

### <a name="delete-services"></a>刪除服務

使用此呼叫來取得服務及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/services?api-version=2018-03-01-preview

可以藉由下列方式刪除個別服務：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/services/{id}?api-version=2018-03-01-preview

## <a name="export-service-data-with-the-rest-api"></a>使用 REST API 來匯出服務資料

為了匯出資料，可以使用 HTTP GET 動詞來進行下列 API 呼叫。 這些可透過在要求中指定 `Authorization: Bearer <arm-token>` 標頭來進行授權，其中 `<arm-token>` 是 `https://management.core.windows.net/` 端點的 AAD 存取權杖  

若要了解如何取得此權杖及呼叫 Azure 端點，請參閱 [Azure REST API 文件](https://docs.microsoft.com/rest/api/azure/) \(英文\)。   

在下列範例中，請以決定相關資源的執行個體名稱取代 {} 中的文字。

### <a name="export-workspace-information"></a>匯出工作區資訊

使用此呼叫來取得所有工作區的清單：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces?api-version=2018-03-01-preview

可以藉由下列方式取得個別工作區的相關資訊：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}?api-version=2018-03-01-preview

### <a name="export-compute-information"></a>匯出計算資訊

可以藉由下列方式取得附加至工作區的所有計算目標：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroup/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/computes?api-version=2018-03-01-preview

可以藉由下列方式取得單一計算目標的相關資訊：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroup/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/computes/{computeName}?api-version=2018-03-01-preview

### <a name="export-run-history-data"></a>匯出執行歷程記錄資料
使用此呼叫來取得所有實驗及其資訊：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/experiments 

可以藉由下列方式取得特定實驗的所有執行：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/experiments/{experimentName}/runs 

可以藉由下列方式取得執行歷程記錄項目：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/experiments/{experimentName}/runs/{runId} 

可以藉由下列方式取得實驗的所有執行計量：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/experiments/{experimentName}/metrics 

可以藉由下列方式取得單一執行計量：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/experiments/{experimentName}/metrics/{metricId}
    
### <a name="export-artifacts"></a>匯出成品

使用此呼叫來取得成品及其路徑的清單：

    https://{location}.experiments.azureml.net/artifact/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/artifacts/origins/ExperimentRun/containers/{runId}
    
### <a name="export-notifications"></a>匯出通知

使用此呼叫來取得已儲存工作的清單：     

    https://{location}.experiments.azureml.net/notification/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/tasks

可以藉由下列方式取得單一工作的通知：

    https://{location}.experiments.azureml.net/notification/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}tasks/{taskId}

### <a name="export-data-stores"></a>匯出資料存放區

使用此呼叫來取得資料存放區的清單：

    https://{location}.experiments.azureml.net/datastore/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/datastores

可以藉由下列方式取得個別資料存放區：

    https://{location}.experiments.azureml.net/datastore/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/datastores/{name}

### <a name="export-models"></a>匯出模型

使用此呼叫來取得模型及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/models?api-version=2018-03-01-preview

可以藉由下列方式取得個別模型：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/models/{id}?api-version=2018-03-01-preview

### <a name="export-assets"></a>匯出資產

使用此呼叫來取得資產及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/assets?api-version=2018-03-01-preview

可以藉由下列方式取得個別資產：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/assets/{id}?api-version=2018-03-01-preview

### <a name="export-images"></a>匯出映像

使用此呼叫來取得映像及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/images?api-version=2018-03-01-preview

可以藉由下列方式取得個別映像：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/images/{id}?api-version=2018-03-01-preview

### <a name="export-services"></a>匯出服務

使用此呼叫來取得服務及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/services?api-version=2018-03-01-preview

可以藉由下列方式取得個別服務：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspace}/services/{id}?api-version=2018-03-01-preview

### <a name="export-pipeline-experiments"></a>匯出管線實驗

可以藉由下列方式取得個別實驗：

    https://{location}.aether.ms/api/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/Experiments/{experimentId}

### <a name="export-pipeline-graphs"></a>匯出管線圖表

可以藉由下列方式取得個別圖表：

    https://{location}.aether.ms/api/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/Graphs/{graphId}

### <a name="export-pipeline-modules"></a>匯出管線模組

可以藉由下列方式取得模組：

    https://{location}.aether.ms/api/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/Modules/{id}

### <a name="export-pipeline-templates"></a>匯出管線範本

可以藉由下列方式取得範本：

    https://{location}.aether.ms/api/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/Templates/{templateId}

### <a name="export-pipeline-data-sources"></a>匯出管線資料來源

可以藉由下列方式取得資料來源：

    https://{location}.aether.ms/api/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/DataSources/{id}
