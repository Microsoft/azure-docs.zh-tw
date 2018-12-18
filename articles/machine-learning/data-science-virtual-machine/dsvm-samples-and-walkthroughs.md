---
title: 資料科學虛擬機器的範例和逐步介紹 - Azure | Microsoft Docs
description: 資料科學虛擬機器的範例和逐步介紹。
keywords: 資料科學工具、資料科學虛擬機器、資料科學工具、linux 資料科學
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: gokuma
ms.openlocfilehash: a1f15b805d2f27152d9ba85608ce0dc1d1aac21e
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392560"
---
# <a name="samples-on-the-data-science-virtual-machines-dsvm"></a>資料科學虛擬機器 (DSVM) 上的範本

DSVM 包含一組完整的範例程式碼，其格式除了 Jupyter Notebook，還有以 Python 和 R 等語言撰寫的指令碼。    
> [!NOTE]
> 如需在 DSVM 上執行 Jupyter Notebook 的詳細資訊，請參閱[存取 Jupyter](#access-jupyter) 一節。

## <a name="quick-reference-of-samples"></a>範例的快速參考
| 範例類別 | 說明 | 位置 |
| ------------- | ------------- | ------------- |
| **R** 語言  | **R** 中的範例說明的情況包括連接 Azure 雲端資料庫存放區、比較開放原始碼 R 與 Microsoft R，以及在 Microsoft R Server 或 SQL Server 上運用模型。 <br/> [螢幕擷取畫面](#r-language) | <br/>`~notebooks` <br/> <br/> `~samples/MicrosoftR` <br/> <br/> `~samples/RSqlDemo` <br/> <br/> `~samples/SQLRServices`<br/> <br/>|
| **Python** 語言  | **Python** 中的範例說明的情況包括連接 Azure 雲端資料存放區，以及使用 **Azure Machine Learning**。  <br/> [螢幕擷取畫面](#python-language) | <br/>`~notebooks` <br/><br/>|
| **Julia** 語言  | **Julia** 中的範例詳細說明 Julia 中的繪圖、Julia 中的深入學習、從 Julia 呼叫 C 和 Python 等等。 <br/> [螢幕擷取畫面](#julia-language) |<br/> **Windows**：<br/> `~notebooks/Julia_notebooks`<br/><br/> **Linux**：<br/> `~notebooks/julia`<br/><br/> |
| **Azure Machine Learning** AzureML  | 使用 **Azure Machine Learning** 服務建置 ML 和深度學習模型，並在任何地方部署模型。 利用自動化 ML、智慧型超參數微調、模型管理和分散式定型等功能。 <br/> [螢幕擷取畫面](#azureml) | <br/>`~notebooks/AzureML`<br/> <br/>|
| **PyTorch** Notebook  | 運用 **PyTorch** 型類神經網路的深度學習範例。 有從初學者到進階案例的各種不同 Notebook。  <br/> [螢幕擷取畫面](#pytorch) | <br/>`~notebooks/Deep_learning_frameworks/pytorch`<br/> <br/>|
| **TensorFlow**  | 使用 **TensorFlow** 架構實作的多個不同神經網路範例和技術。 <br/> [螢幕擷取畫面](#tensorflow) | <br/>`~notebooks/Deep_learning_frameworks/tensorflow`<br/><br/> |
| **CNTK** <br/> (Microsoft 辨識工具組)  | Microsoft 的辨識工具組小組發佈的深入學習範例。  <br/> [螢幕擷取畫面](#cntk) | <br/> `~notebooks/DeepLearningTools/CNTK/Tutorials`<br/><br/> **Linux**：<br/> `~notebooks/CNTK`<br/> <br/>|
| **caffe2** | 運用 **caffe2** 型神經網路的深入學習範例。 有一些 Notebook 的設計是讓使用者熟悉 caffe2 以及如何有效使用它，包括映像預先處理、資料集建立、迴歸和使用預先定型的模型之類的範例。 <br/> [螢幕擷取畫面](#caffe2) | <br/>`~notebooks/Deep_learning_frameworks/caffe2`<br/><br/> |
| **H2O**   | 使用 **H2O** 的 Python 架構範例處理許多真實案例問題。 <br/> [螢幕擷取畫面](#h2o) | <br/>`~notebooks/h2o`<br/><br/> |
| **SparkML** 語言  | 在 **Apache Spark 2.x** 上透過 **pySpark** 和 **MMLSpark (適用於 Apache Spark 的 Microsoft 機器學習)** 利用 Spark **MLlib** 工具組特色和功能的範例。  <br/> [螢幕擷取畫面](#sparkml) | <br/>`~notebooks/SparkML/pySpark`<br/>`~notebooks/MMLSpark`<br/><br/>  |
| **XGBoost** | **XGBoost** 中對於分類、迴歸等案例的標準機器學習範例。 <br/> [螢幕擷取畫面](#xgboost) | <br/>**Windows**：<br/>`\dsvm\samples\xgboost\demo`<br/><br/> |

<br/>

## <a name="access-jupyter"></a>存取 Jupyter 

您可以按一下桌面或應用程式功能表上的 `Jupyter` 圖示存取 Jupyter。 您也可以在 Ubuntu 上瀏覽 **`https://<Full Domain Name or IP Address of the DSVM>:8000`**，從網頁瀏覽器遠端存取 DSVM Linux 版本上的 Jupyter。

查看螢幕擷取畫面以新增例外狀況，並透過瀏覽器啟用 Jupyter 存取。


![啟用 Jupyter 例外狀況](./media/ubuntu-jupyter-exception.png)


使用登入 DSVM 所用的同一組密碼登入。
<br/>

**Jupyter 首頁**
<br/>![Jupyter 首頁](./media/jupyter-home.png)<br/>

## <a name="r-language"></a>R 語言 
<br/>![R 範例](./media/r-language-samples.png)<br/>

## <a name="python-language"></a>Python 語言
<br/>![Python 範例](./media/python-language-samples.png)<br/>

## <a name="julia-language"></a>Julia 語言 
<br/>![Julia 範例](./media/julia-samples.png)<br/>

## <a name="azureml"></a>AzureML 
<br/>![AzurekML 範例](./media/azureml-samples.png)<br/>

## <a name="pytorch"></a>PyTorch
<br/>![PyTorch 範例](./media/pytorch-samples.png)<br/>

## <a name="tensorflow"></a>TensorFlow 
<br/>![TensorFlow 範例](./media/tensorflow-samples.png)<br/>


## <a name="cntk"></a>CNTK 
<br/>![CNTK 範例](./media/cntk-samples.png)<br/>


## <a name="caffe2"></a>caffe2 
<br/>![caffe2 範例](./media/caffe2-samples.png)<br/>

## <a name="h2o"></a>H2O 
<br/>![H2O 範例](./media/h2o-samples.png)<br/>

## <a name="sparkml"></a>SparkML 
<br/>![SparkML 範例](./media/sparkml-samples.png)<br/>

## <a name="xgboost"></a>XGBoost 
<br/>![XGBoost 範例](./media/xgboost-samples.png)<br/>

