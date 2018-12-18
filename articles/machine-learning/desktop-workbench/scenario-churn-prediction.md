---
title: 使用 Azure Machine Learning 進行客戶流失預測 | Microsoft Docs
description: 如何使用 Azure ML Workbench 執行流失分析。
services: machine-learning
documentationcenter: ''
author: miprasad
manager: kristin.tolle
editor: miprasad
ms.assetid: ''
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/17/2017
ms.author: miprasad
ROBOTS: NOINDEX
ms.openlocfilehash: 0210e65c0859b00caac0fe66baa1c73063f644c8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46947936"
---
# <a name="customer-churn-prediction-using-azure-machine-learning"></a>使用 Azure Machine Learning 進行客戶流失預測

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 



平均而言，維持現有客戶比招收新客戶的成本便宜五倍。 因此，行銷主管通常會發現自己需要評估客戶流失的可能性，並尋找必要的動作將流失率降到最低。

此解決方案旨在使用 Azure Machine Learning Workbench 示範預測流失分析。 此解決方案提供的易於使用範本可開發零售商的流失預測資料管線。 範本可以搭配使用不同的資料集和不同的流失定義。 實際操作範例的目的是要：

1. 了解 Azure Machine Learning Workbench 的資料準備工具，來清除和內嵌流失分析的客戶關聯性資料。

2. 執行功能轉換來處理含有雜訊的異質資料。

3. 整合第三方程式庫 (例如 `scikit-learn` 和 `azureml`) 來開發可預測流失的貝氏和以樹狀結構作為基礎的分類器。

4. 部署。

## <a name="link-of-the-gallery-github-repository"></a>資源庫 GitHub 存放庫的連結
以下是裝載所有程式碼的公用 GitHub 存放庫之連結：

[https://github.com/Azure/MachineLearningSamples-ChurnPrediction](https://github.com/Azure/MachineLearningSamples-ChurnPrediction)

## <a name="use-case-overview"></a>使用案例概觀
公司需要有效的策略來管理客戶流失。 客戶流失包含客戶停止使用服務、切換至競爭對手的服務、切換到服務中較低層的體驗，或減少參與服務。

在此使用案例中，我們要查看來自法國電信公司 Orange 的資料，來識別為了改善服務而可能在近期流失的客戶，並建立可協助保留客戶的自訂拓展活動。

電信公司面臨具競爭力的市場。 許多電訊廠商由於客戶流失而失去來自後付型客戶的營收。 因此，能夠精確地識別客戶流失會是極大的競爭優勢。

導致電信客戶流失的一些因素包括：

* 察覺到服務中斷頻繁
* 在線上/零售商店的客戶服務體驗不佳
* 其他競爭電訊廠商的供應項目 (更好的家庭方案、行動數據方案等)。

在此解決方案中，我們將使用具體範例來建立電信公司的預測客戶流失模型。

## <a name="prerequisites"></a>必要條件

* [Azure 帳戶](https://azure.microsoft.com/free/) (提供免費試用)

* 遵循[快速入門安裝指南](quickstart-installation.md)所安裝的 [Azure Machine Learning Workbench](../service/overview-what-is-azure-ml.md) 版本，以便安裝程式並建立 Workbench

* 若要進行運作，您最好是執行在本機安裝並執行 Docker 引擎。 如果不是，您可以使用叢集選項，但請注意，Azure Container Service (ACS) 的執行成本會很高。

* 這個解決方案是假設您使用本機安裝的 Docker 引擎，在 Windows 10 上執行 Azure Machine Learning Workbench。 如果您是使用 macOS，指示大致相同。

## <a name="create-a-new-workbench-project"></a>建立新的 Workbench 專案

使用此範例作為範本來建立新專案：
1.  開啟 Azure Machine Learning Workbench
2.  在 [專案] 頁面上，按一下 **+** 符號，然後選取 [新增專案]
3.  在 [建立新專案] 窗格中，填入新專案的資訊
4.  在 [搜尋專案範本] 搜尋方塊中輸入「客戶流失預測」並選取範本
5.  按一下 [建立] 

## <a name="data-description"></a>資料說明

解決方案中使用的資料集是來自 SIDKDD 2009 競爭。 它稱為 `CATelcoCustomerChurnTrainingSample.csv`，且位於 [`data`](https://github.com/Azure/MachineLearningSamples-ChurnPrediction/tree/master/data) 資料夾中。 資料集包含來自法國電信公司 Orange 的異質雜訊資料 (數值/類別變數)，並且是匿名的。

變數會擷取客戶人口統計資訊、通話統計資料 (例如平均的通話持續時間、通話失敗率等)、合約資訊、相容統計資料。 流失變數是二進位檔案 (0 - 未流失，1 - 已流失)。

## <a name="scenario-structure"></a>案例結構

資料夾結構的排列方式如下所示：

__data__：包含解決方案中使用的資料集  

__docs__：包含所有線上實習實驗室

實現解決方案的實際操作實驗室順序如下所示：
1. 資料準備：[data] 資料夾中與「資料準備」相關的主要檔案是 `CATelcoCustomerChurnTrainingSample.csv`
2. 模型化和評估：根資料夾中與模型化和評估相關的主要檔案是 `CATelcoCustomerChurnModeling.py`
3. 不使用 .dprep 進行模型化和評估：根資料夾中此工作的主要檔案是 `CATelcoCustomerChurnModelingWithoutDprep.py`
4. 作業：部署的主要檔案是模型 (`model.pkl`) 和 `churn_schema_gen.py`

| 順序| 檔案名稱 | 相關的檔案 |
|--|-----------|------|
| 1 | [`DataPreparation.md`](https://github.com/Azure/MachineLearningSamples-ChurnPrediction/blob/master/docs/DataPreparation.md) | 'data/CATelcoCustomerChurnTrainingSample.csv' |
| 2 | [`ModelingAndEvaluation.md`](https://github.com/Azure/MachineLearningSamples-ChurnPrediction/blob/master/docs/ModelingAndEvaluation.md) | 'CATelcoCustomerChurnModeling.py' |
| 3 | [`ModelingAndEvaluationWithoutDprep.md`](https://github.com/Azure/MachineLearningSamples-ChurnPrediction/blob/master/docs/ModelingAndEvaluationWithoutDprep.md) | 'CATelcoCustomerChurnModelingWithoutDprep.py' |
| 4 | [`Operationalization.md`](https://github.com/Azure/MachineLearningSamples-ChurnPrediction/blob/master/docs/Operationalization.md) | 'model.pkl'<br>'churn_schema_gen.py' |

請依上述的循序方式遵循實驗室。

## <a name="conclusion"></a>結論
此實際操作情節示範如何使用 Azure Machine Learning Workbench 執行流失預測。 我們會先執行資料清除來處理雜訊和異質資料，接著使用資料準備工具進行功能設計。 然後，我們會使用開放原始碼機器學習工具來建立和評估分類模型，然後使用本機 Docker 容器來部署讓它準備好實際執行的模型。
