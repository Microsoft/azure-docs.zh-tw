---
title: Azure Machine Learning 模型資料收集 API 參考 | Microsoft Docs
description: Azure Machine Learning 模型資料收集 API 參考。
services: machine-learning
author: aashishb
ms.author: aashishb
manager: hjerez
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 09/12/2017
ROBOTS: NOINDEX
ms.openlocfilehash: c047555c30481f34b607197f71483197bc64620c
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961462"
---
# <a name="azure-machine-learning-model-data-collection-api-reference"></a>Azure Machine Learning 模型資料收集 API 參考

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

模型資料收集可讓您保存模型輸入以及來自 Machine Learning Web 服務的預測。 請參閱[模型資料收集操作說明指南](how-to-use-model-data-collection.md)來了解如何在 Windows 和 Linux 機器上安裝 `azureml.datacollector`。

在此 API 參考指南中，我們會使用逐步說明的方法，來讓您了解如何從 Machine Learning Web 服務收集模型的輸入和預測。

## <a name="enable-model-data-collection-in-azure-ml-workbench-environment"></a>在 Azure ML Workbench 環境中啟用模型資料收集

 在專案的 aml_config 資料夾底下找到 conda\_dependencies.yml 檔案，然後讓 conda\_dependencies.yml 檔案在 pip 區段底下包含 azureml.datacollector 模組，如下所示。 請注意，這只是完整的 conda\_dependencies.yml 檔案的子區段：

    dependencies:
      - python=3.5.2
      - pip:
        - azureml.datacollector==0.1.0a13

>[!NOTE] 
>您目前可以在 docker 模式下執行，以在 Azure ML Workbench 中使用資料收集器模組。 本機模式可能不適用於所有環境。




## <a name="enable-model-data-collection-in-the-scoring-file"></a>在評分檔案中啟用模型資料收集

在用於運作的評分檔案中，匯入資料收集器模組和 ModelDataCollector 類別：

    from azureml.datacollector import ModelDataCollector


## <a name="model-data-collector-instantiation"></a>模型資料收集器具現化
具現化新的 ModelDataCollector 執行個體：

    dc = ModelDataCollector(model_name, identifier='default', feature_names=None, model_management_account_id='unknown', webservice_name='unknown', model_id='unknown', model_version='unknown')

請參閱類別和參數的詳細資料：

### <a name="class"></a>類別
| 名稱 | 說明 |
|--------------------|--------------------|
| ModelDataCollector | azureml.datacollector 命名空間中的類別。 這個類別的執行個體將用來收集模型資料。 單一評分檔案可以包含多個 ModelDataCollectors。 每個執行個體都應該用於收集評分檔案中某個個別位置的資料，以便讓收集到之資料的結構描述保持一致 (亦即輸入和預測)|


### <a name="parameters"></a>參數

| 名稱 | 類型 | 說明 |
|-------------|------------|-------------------------|
| model_name | 字串 | 要收集資料之模型的名稱 |
| 識別碼 | 字串 | 程式碼中可識別此資料的位置，也就是'RawInput' 或 'Prediction' |
| feature_names | 字串的清單 | 會在提供時變成 csv 標頭的功能名稱清單 |
| model_management_account_id | 字串 | 此模型儲存所在之模型管理帳戶的識別碼。 當模型透過 AML 來運作時，系統會自動填入此參數 |
| webservice_name | 字串 | 此模型目前部署所在之 Web 服務的名稱。 當模型透過 AML 來運作時，系統會自動填入此參數 |
| model_id | 字串 | 此模型在模型管理帳戶的內容中的唯一識別碼。 當模型透過 AML 來運作時，系統會自動填入此參數 |
| model_version | 字串 | 此模型在模型管理帳戶的內容中的版本號碼。 當模型透過 AML 來運作時，系統會自動填入此參數 |



 

## <a name="collecting-the-model-data"></a>收集模型資料

您可以使用上面所建立的 ModelDataCollector 執行個體來收集模型資料。

    dc.collect(input_data, user_correlation_id="")

請參閱方法和參數的詳細資料：

### <a name="method"></a>方法
| 名稱 | 說明 |
|--------------------|--------------------|
| 收集 | 用來收集資料以供模型的輸入或預測使用|


### <a name="parameters"></a>參數

| 名稱 | 類型 | 說明 |
|-------------|------------|-------------------------|
| input_data | 多個類型 | 要收集的資料 (目前可接受類型清單、numpy.array、pandas.DataFrame、pyspark.sql.DataFrame)。 若為資料框架類型，如果有這些功能名稱的標頭存在，則此資訊會包含在資料目的地內 (而不必明確地在 ModelDataCollector 建構函式中傳遞功能名稱) |
| user_correlation_id | 字串 | 選擇性的相互關聯識別碼，可由使用者提供以便與此預測相互關聯 |

