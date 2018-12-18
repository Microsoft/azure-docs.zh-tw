---
title: 在生產環境中啟用模型資料收集 - Azure Machine Learning
description: 了解如何在 Azure Blob 儲存體中收集 Azure Machine Learning 輸入模型資料。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: marthalc
author: marthalc
ms.date: 09/24/2018
ms.openlocfilehash: 412871c0c692f60e690f61fa4e6f67f836cd3ef7
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47158202"
---
# <a name="collect-data-for-models-in-production"></a>在生產環境中收集模型資料

在本文中，您可以了解如何將輸入模型資料從您已部署至 Azure Kubernetes Cluster (AKS) 的 Azure Machine Learning 服務收集到 Azure Blob 儲存體。 

啟用之後，您收集的此資料可協助您：
* 監視生產資料進入您模型時的資料變化

* 制定何時重新訓練或最佳化您模型的更佳決策

* 使用收集到的資料重新訓練模型

## <a name="what-is-collected-and-where-does-it-go"></a>收集的內容和其放置位置？

下列是可收集的資料：
* 來自 Azure Kubernetes Cluster (AKS) 中所部署 Web 服務的模型**輸入**資料 (**不會**收集語音、影像和影片) 
  
* 使用生產輸入資料的模型預測。

> [!Note]
> 對此資料的預先彙總或預先計算，目前不屬於服務的一部分。   

輸出會儲存至 Azure Blob。 因為資料會新增至 Azure Blob，所以您接著可以選擇慣用的工具來執行分析。 

Blob 中輸出資料的路徑遵循此語法：

```
/modeldata/<subscriptionid>/<resourcegroup>/<workspace>/<webservice>/<model>/<version>/<identifier>/<year>/<month>/<day>/data.csv
# example: /modeldata/1a2b3c4d-5e6f-7g8h-9i10-j11k12l13m14/myresourcegrp/myWorkspace/aks-w-collv9/best_model/10/inputs/2018/12/31/data.csv
```

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- 已安裝 Azure Machine Learning 工作區、包含您指令碼的本機目錄，以及適用於 Python 的 Azure Machine Learning SDK。 了解如何使用[如何設定開發環境](how-to-configure-environment.md)文件來取得這些必要條件。

- 要部署至 Azure Kubernetes Service (AKS) 的訓練過機器學習模型。 如果您沒有該模型，請參閱[訓練影像分類模型](tutorial-train-models-with-aml.md)教學課程。

- [AKS 叢集](how-to-deploy-to-aks.md)。

- 下列是安裝[在環境中](how-to-configure-environment.md)的相依性和模組：
  + 在 Linux 上：
    ```shell
    sudo apt-get install libxml++2.6-2v5
    pip install azureml-monitoring
    ```

  + 在 Windows 上：
    ```shell
    pip install azureml-monitoring
    ```

## <a name="enable-data-collection"></a>啟用資料收集
不論透過 Azure Machine Learning Service 或其他工具所部署的模型為何，都可以啟用資料收集。 

若要予以啟用，您需要：

1. 開啟評分檔案。 

1. 在檔案開頭處新增下列程式碼：

   ```python 
   from azureml.monitoring import ModelDataCollector
   ```

2. 在 `init()` 函式中宣告資料收集變數：

    ```python
    global inputs_dc, prediction_dc
    inputs_dc = ModelDataCollector("best_model", identifier="inputs", feature_names=["feat1", "feat2", "feat3". "feat4", "feat5", "feat6"])
    prediction_dc = ModelDataCollector("best_model", identifier="predictions", feature_names=["prediction1", "prediction2"])
    ```

    CorrelationId 是選用參數，如果您的模型不需要它，則不需要設定。 備妥 correlationId 可協助您更輕鬆地對應其他資料  (範例包含：LoanNumber、CustomerId 等等)。
    
    Identifier 稍後用於建置 Blob 中的資料夾結構，可用來劃分「原始」資料與「已處理」。

3.  將下列程式碼行新增至 `run(input_df)` 函式：

    ```python
    data = np.array(data)
    result = model.predict(data)
    inputs_dc.collect(data) #this call is saving our input data into Azure Blob
    prediction_dc.collect(result) #this call is saving our input data into Azure Blob
    ```

4. 當您在 AKS 中部署服務而必須更新設定檔案時，資料收集**不會**自動設定為 **true**，例如： 

    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True)
    ```
    透過變更此設定，也可以開啟用於監視服務的 AppInsights：
    ```python
    aks_config = AksWebservice.deploy_configuration(collect_model_data=True, enable_app_insights=True)
    ``` 

5. [建立新映像並部署服務。](how-to-deploy-to-aks.md) 


如果您的現有服務已在**環境檔案**和**評分檔案**中安裝相依性，則請透過下列方式啟用資料收集：

1. 移至 [Azure 入口網站 ](https://portal.azure.com)。

1. 開啟工作區。

1. 移至 [部署] -> [選取服務] -> [編輯]。

   ![編輯服務](media/how-to-enable-data-collection/EditService.png)

1. 在 [進階設定] 中，取消選取 [啟用模型資料收集]。 

   ![取消核取資料收集](media/how-to-enable-data-collection/CheckDataCollection.png)

   在此視窗中，您也可以選擇 [啟用 AppInsights 診斷] 來追蹤您服務的健康狀態。  

1. 選取 [更新] 以套用變更。


## <a name="disable-data-collection"></a>停用資料收集
您隨時可以停止收集資料。 使用 Python 程式碼或 Azure 入口網站來停用資料收集。

+ 選項 1 - 在 Azure 入口網站中停用： 
  1. 登入 [Azure 入口網站](https://portal.azure.com)。

  1. 開啟工作區。

  1. 移至 [部署] -> [選取服務] -> [編輯]。

     ![編輯服務](media/how-to-enable-data-collection/EditService.png)

  1. 在 [進階設定] 中，取消選取 [啟用模型資料收集]。 

     ![取消核取資料收集](media/how-to-enable-data-collection/UncheckDataCollection.png) 

  1. 選取 [更新] 以套用變更。

* 選項 2 - 使用 Python 停用資料收集：

  ```python 
  ## replace <service_name> with the name of the web service
  <service_name>.update(collect_model_data=False)
  ```

## <a name="example-notebook"></a>範例筆記本

`00.Getting Started/12.enable-data-collection-for-models-in-aks.ipynb` 筆記本示範本文中的概念。  

取得此筆記本：
 
[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]