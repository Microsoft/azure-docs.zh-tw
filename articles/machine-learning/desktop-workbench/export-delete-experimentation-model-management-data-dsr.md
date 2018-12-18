---
title: 探索或刪除實驗或模型管理資料 - Azure Machine Learning | Microsoft Docs
description: 在 Azure Machine Learning 中，您可以使用 Azure 入口網站、CLI、SDK 及已驗證的 REST API 來探索或刪除與實驗或模型管理相關的帳戶資料。 本文將說明如何做到。
services: machine-learning
author: cjgronlund
ms.author: cgronlun
manager: haining
ms.reviewer: jmartens, mldocs
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.date: 05/22/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 8c5b20bf837491e8b15f7bc8baa018ead584616b
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47159171"
---
# <a name="export-or-delete-your-experimentation-or-model-management-data-in-machine-learning"></a>在機器學習服務中探索或刪除實驗或模型管理資料

>[!Note]
>**此文章已過時。** 針對這個服務之舊版本的支援將逐漸終止。 [檢視支援時間表](../service/overview-what-happened-to-workbench.md#timeline)。 閱讀關於此主題的[最新文章](../service/how-to-export-delete-data.md)。


在 Azure Machine Learning 中，您可以使用已驗證的 REST API 來探索或刪除與實驗或模型管理相關的帳戶資料。 本文會說明做法。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="control-your-account-data"></a>控制您的帳戶資料
透過 Azure 入口網站、CLI、SDK 及已驗證的 REST API，可以匯出和刪除 Azure Machine Learning 實驗和模型管理所儲存的產品中資料。 透過 Azure 隱私權入口網站可以存取遙測資料。 

在 Azure Machine Learning 中，個人資料會包含執行歷程記錄文件中的使用者資訊，以及使用者與服務部分互動的遙測記錄。

## <a name="delete-account-data-with-the-rest-api"></a>使用 REST API 來刪除帳戶資料 

為了刪除資料，可以使用 HTTP DELETE 動詞來進行下列 API 呼叫。 這些可透過在要求中指定 `Authorization: Bearer <arm-token>` 標頭來進行授權，其中 `<arm-token>` 是 `https://management.core.windows.net/` 端點的 AAD 存取權杖。  

若要了解如何取得此權杖及呼叫 Azure 端點，請參閱 [Azure REST API 文件](https://docs.microsoft.com/rest/api/azure/) \(英文\)。  

在下列範例中，請以決定相關資源的執行個體名稱取代 {} 中的文字。

## <a name="delete-from-a-hosting-account"></a>從裝載帳戶刪除

    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.MachineLearningModelManagement/accounts/{account-name}?api-version=2017-09-01-preview      

## <a name="delete-from-the-model-management-service"></a>從模型管理服務刪除

### <a name="model-document"></a>模型文件
使用此呼叫來取得模型及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models?api-version=2017-09-01-preview"

可以藉由下列方式刪除個別模型：  

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models/{modelId}?api-version=2017-09-01-preview

### <a name="manifest-document"></a>資訊清單文件
使用此呼叫來取得所有資訊清單及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests?api-version=2017-09-01-preview

可以藉由下列方式刪除個別資訊清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests/{manifestId}?api-version=2017-09-01-preview

### <a name="service-documents"></a>服務文件
使用此呼叫來取得所有服務及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services?api-version=2017-09-01-preview

可以藉由下列方式刪除個別服務：    

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{serviceName}?api-version=2017-09-01-preview

### <a name="image-document"></a>映像文件
使用此呼叫來取得所有映像及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images?api-version=2017-09-01-preview

可以藉由下列方式刪除個別映像：  

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images/{imageId}?api-version=2017-09-01-preview

## <a name="delete-run-history-artifact-and-notification-data"></a>刪除執行歷程記錄、成品及通知資料
在刪除對應的專案文件後，便會將專案的歷程記錄、成品及通知存放區全部刪除：

    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.MachineLearningExperimentation/accounts/{account-name}/workspaces/{workspace-name}/projects/{project-name}?api-version=2017-05-01-preview
    
## <a name="delete-from-experimentation-account-resource-provider"></a>從實驗帳戶資源提供者刪除
專案文件會以下列方式刪除：

    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.MachineLearningExperimentation/accounts/{account-name}/workspaces/{workspace-name}/projects/{project-name}?api-version=2017-05-01-preview

工作區文件會以下列方式刪除：

    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.MachineLearningExperimentation/accounts/{account-name}/workspaces/{workspace-name}?api-version=2017-05-01-preview

整個實驗帳戶會以下列方式刪除：
    
    https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.MachineLearningExperimentation/accounts/{account-name}?api-version=2017-05-01-preview


## <a name="export-service-data-with-the-rest-api"></a>使用 REST API 來匯出服務資料
為了匯出資料，可以使用 HTTP GET 動詞來進行下列 API 呼叫。 這些可透過在要求中指定 `Authorization: Bearer <arm-token>` 標頭來進行授權，其中 `<arm-token>` 是 `https://management.core.windows.net/` 端點的 AAD 存取權杖  

若要了解如何取得此權杖及呼叫 Azure 端點，請參閱 [Azure REST API 文件](https://docs.microsoft.com/rest/api/azure/) \(英文\)。   

在下列範例中，請以決定相關資源的執行個體名稱取代 {} 中的文字。

## <a name="export-hosting-account-data"></a>匯出裝載帳戶資料

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningModelManagement/accounts/{accountName}?api-version=2017-09-01-preview     

## <a name="export-model-management-service-data"></a>匯出模型管理服務資料
### <a name="model-document"></a>模型文件

使用此呼叫來取得模型及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models?api-version=2017-09-01-preview"

可以藉由下列方式取得個別模型：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models/{modelId}?api-version=2017-09-01-preview 

### <a name="manifests"></a>資訊清單
使用此呼叫來取得所有資訊清單及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests?api-version=2017-09-01-preview

可以藉由下列方式取得個別資訊清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests/{manifestId}?api-version=2017-09-01-preview

### <a name="services"></a>服務
使用此呼叫來取得所有服務及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services?api-version=2017-09-01-preview

可以藉由下列方式取得個別服務： 

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{serviceName}?api-version=2017-09-01-preview

### <a name="images"></a>映像
使用此呼叫來取得所有映像及其識別碼的清單：

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images?api-version=2017-09-01-preview

可以藉由下列方式取得個別服務： 

    https://{location}.modelmanagement.azureml.net/api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images/{imageId}?api-version=2017-09-01-preview     

## <a name="export-compute-data"></a>匯出計算資料
### <a name="compute-clusters"></a>計算叢集
使用此呼叫來取得所有計算叢集及其名稱的清單：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/computes?api-version=2018-03-01-preview

可以藉由下列方式取得個別叢集：

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningServices/workspaces/{workspaceName}/computes/{compute-name}?api-version=2018-03-01-preview

### <a name="operationalization-clusters"></a>作業化叢集
使用此呼叫來取得所有叢集及其名稱的清單：

    https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.MachineLearningCompute/operationalizationClusters?api-version=2017-06-01-preview

可以藉由下列方式取得個別叢集：

    https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.MachineLearningCompute/operationalizationClusters/{clusterName}?api-version=2017-06-01-preview

## <a name="export-run-history-data"></a>匯出執行歷程記錄資料
使用此呼叫來取得所有執行及其識別碼的清單：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runs

使用此呼叫來取得所有實驗及其識別碼的清單：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/experiments

可以藉由下列方式取得執行歷程記錄項目：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runs/{runId}

可以藉由下列方式取得執行計量項目：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runmetrics

可以藉由下列方式取得執行實驗：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/experiments/{experimentId}    

執行歷程記錄成品：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runs/{runId}/artifacts

執行歷程記錄成品 URI：

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runs/{runId}/artifacturi?name={artifactName}

## <a name="export-artifacts"></a>匯出成品
使用此呼叫來取得資產及其名稱的清單：

    https://{location}.experiments.azureml.net/artifact/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/assets

使用此呼叫來取得成品及其路徑的清單：

    https://{location}.experiments.azureml.net/artifact/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/artifacts/origins/{origin}/containers/{runId}
        
        
### <a name="artifact-contents"></a>成品內容

    https://{location}.experiments.azureml.net/artifact/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/artifacts/contentinfo/ExperimentRun/{runId}/{artifactPath}

### <a name="artifact-documents"></a>成品文件

    https://{location}.experiments.azureml.net/history/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/runs/{runId}/artifacts
        
### <a name="assets"></a>Assets

    https://{location}.experiments.azureml.net/artifact/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/assets/name/{name}

## <a name="export-notifications"></a>匯出通知

1. 移至 [Azure 入口網站的 [使用者] 區段](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/)，然後從 [名稱] 資料行中選取使用者。 
2. 記下 [物件識別碼]，然後將它用於下列呼叫中：     

        https://{location}.experiments.azureml.net/notification/v1.0/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}/users/{objectId}/jobs

## <a name="export-experimentation-account-information"></a>匯出實驗帳戶資訊
### <a name="experimentation-account-information"></a>實驗帳戶資訊

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}?api-version=2017-05-01-preview
        
### <a name="workspace-information"></a>工作區資訊

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}?api-version=2017-05-01-preview  

### <a name="project-information"></a>專案資訊

    https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.MachineLearningExperimentation/accounts/{accountName}/workspaces/{workspaceName}/projects/{projectName}?api-version=2017-05-01-preview
