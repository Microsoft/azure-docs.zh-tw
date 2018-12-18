---
title: 建置機器學習管線 - Azure Machine Learning 服務
description: 在本文中，您將深入了解機器學習管線 (使用適用於 Python 的 Azure Machine Learning 來建置)，以及了解使用管線的優點。 資料科學家會使用機器學習 (ML) 管線來建置、最佳化及管理他們的機器學習工作流程。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: sanpil
author: sanpil
ms.date: 09/24/2018
ms.openlocfilehash: e1181d07ed51b1edf5da68d759d871703bd1540f
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162775"
---
# <a name="pipelines-and-azure-machine-learning"></a>管線和 Azure Machine Learning

在本文中，您將深入了解機器學習管線 (使用適用於 Python 的 Azure Machine Learning 來建置)，以及了解使用管線的優點。

## <a name="what-are-machine-learning-pipelines"></a>什麼是機器學習管線？

資料科學家會使用機器學習 (ML) 管線來建置、最佳化及管理他們的機器學習工作流程。 一般管線會包含一連串的步驟，其中涵蓋下列領域：

+ 資料準備，例如正規化和轉換
+ 模型訓練，例如超參數調整和驗證
+ 模型部署和評估  

下圖顯示管線範例：

[![Azure Machine Learning 服務中的機器學習管線](./media/concept-ml-pipelines/pipelines.png) ] (./media/concept-ml-pipelines/machine-learning-pipelines-big.png#lightbox)

## <a name="why-build-pipelines-with-azure-machine-learning"></a>為何使用 Azure Machine Learning 來建置管線？

[適用於 Python 的 Azure Machine Learning SDK](#the-python-sdk-for-pipelines) 可用來建立 ML 管線，以及送出和追蹤個別管線執行。

透過管線，您可以從簡易性、速度、可攜性和重複使用率方面來最佳化工作流程。 使用 Azure Machine Learning 建置管線時，您可以專注於最了解的事項 (機器學習)，而非基礎結構。

使用不同步驟，便可在調整和測試工作流程時，只重新執行所需的步驟。 步驟是在管線中的計算單位。 如上圖所示，準備資料的工作可能牽涉到許多步驟，包括 (但不限於) 正規化、轉換、驗證和特徵化。

設計好管線之後，在管線的訓練迴圈上通常會需要更多微調。 當您重新執行管線時，該執行會跳至需要重新執行的步驟，例如更新的訓練指令碼，並略過未變更的項目。 相同範例適用於用來執行步驟的未變更指令碼。 

透過 Azure Machine Learning，您可以對管線中的每個步驟使用各種工具組和架構，例如 Microsoft Cognitive Toolkit 或 TensorFlow。 Azure 會協調您使用的各種[計算目標](concept-azure-machine-learning-architecture.md)，讓您的中繼資料可輕易地與下游計算目標共用。 

## <a name="key-advantages"></a>主要優點

為機器學習工作流程建置管線的主要優點是：

|主要優點|說明|
|:-------:|-----------|
|**自動&nbsp;執行**|以自動且可靠的方式排定要平行執行或依序執行的幾個步驟。 由於資料準備和建立模型工作會持續數天或數週，因此您可以在管線執行的同時專注於其他工作。 |
|**混合和多元計算**|您可以使用多個管線，這些管線可在異質且可調整的計算和儲存體中可靠地進行協調。 個別的管線步驟可在不同的計算目標上執行，例如 HDInsight、GPU 資料科學 VM 及 Databricks，以有效使用可用的計算選項。|
|**再使用性**|管線可以針對特定案例 (例如重新訓練和批次評分) 範本化。  您可以透過簡單的 REST 呼叫從外部系統觸發這些管線。|
|**追蹤和版本控制**|您不用在逐一查看時手動追蹤資料和結果路徑，而是運用管線 SDK 來明確地為資料來源、輸入和輸出建立名稱和版本，以及針對提升的產能各別管理指令碼與資料。|

## <a name="the-python-sdk-for-pipelines"></a>適用於管線的 Python SDK

使用 Python 來建立您的 ML 管線。 沒有資料相依性存在時，Azure Machine Learning SDK 會提供命令式架構來排序和平行化您管線中的步驟。 您可以在 Jupyter Notebook 或另一個慣用的 IDE 中與之互動。 

您可以使用宣告式資料相依性來最佳化您的工作。 SDK 包含一般工作適用的預建模組架構，這類工作包括資料轉送、計算目標建立及模型發行等等。 藉由實作可在管線間重複使用的自訂步驟，即可延伸架構以建立您自己的慣例模型。

管線可以儲存成範本，也可部署到 REST 端點，以便您排程批次評分或重新訓練作業。

請參閱[管線的 Python SDK 參考文件](http://aka.ms/aml-sdk)和下一節中的 Notebook，以了解如何建置您自己的管線。

## <a name="example-notebooks"></a>Notebook 範例
 
下列 Notebook 會示範如何使用 Azure Machine Learning 建立管線：`pipeline/pipeline-batch-scoring.ipynb`。
 
取得以下筆記：
 
[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]
