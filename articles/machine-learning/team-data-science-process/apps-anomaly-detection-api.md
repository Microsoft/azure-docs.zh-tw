---
title: Azure Machine Learning 異常偵測 API | Microsoft Docs
description: 「異常偵測 API」是一個搭配 Microsoft Azure Machine Learning 建置的範例，此 API 使用固定時間間隔的數值，偵測時間序列資料中的異常狀況。
services: machine-learning
documentationcenter: ''
author: alokkirpal
manager: jhubbard
editor: cgronlun
ms.assetid: 52fafe1f-e93d-47df-a8ac-9a9a53b60824
ms.service: machine-learning
ms.component: team-data-science-process
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 06/05/2017
ms.author: alok
ms.openlocfilehash: 9f1f3c2c808558124548008927219191469a412b
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34836461"
---
# <a name="machine-learning-anomaly-detection-api"></a>Machine Learning 異常偵測 API
## <a name="overview"></a>概觀
[異常偵測 API](https://gallery.cortanaintelligence.com/MachineLearningAPI/Anomaly-Detection-2) 是一個搭配 Azure Machine Learning 建置的範例，此 API 使用固定時間間隔的數值，偵測時間序列資料中的異常狀況。

此 API 可偵測時間序列資料的下列異常模式類型︰

* **正向和負向趨勢**：例如，在監視運算中所使用的記憶體數量時，趨勢向上可能會引起您的注意，因為這可能表示有記憶體流失。
* **動態範圍值有所變更**：例如，在監視雲端服務所擲回的例外狀況時，若動態範圍值有任何變更，可能表示服務的健康狀態不穩定。
* **尖峰和下降**：例如，在監視服務的登入失敗數目或電子商務網站的結帳數目時，若有尖峰或下降可能表示發生異常行為。

這些 Machine Learning 偵測器會追蹤數值在不同時間所發生的這一類變化，並以異常分數的形式報告數值正在進行的變化。 它們不需要臨機操作調整臨界值，而且其分數可用來控制偽陽性率。 異常偵測 API 可用在許多案例，例如追蹤不同時間的 KPI 以監視服務、透過搜尋次數、點擊次數等度量來監視使用量、透過不同時間的記憶體、CPU、檔案讀取數等計數器來監視效能。

異常偵測供應項目在供應時會隨附有用的工具以讓您快速入門。

* [Web 應用程式](http://anomalydetection-aml.azurewebsites.net/) 可協助您評估和視覺化將異常偵測 API 套用在您的資料後所產生的結果。

> [!NOTE]
> 請嘗試採用[這個 API](https://gallery.cortanaintelligence.com/MachineLearningAPI/Anomaly-Detection-2) 的 **IT 異常洞察解決方案**
> 
> 若要將這個端對端解決方案部署到您的 Azure 訂用帳戶，請<a href="https://gallery.cortanaintelligence.com/Solution/Anomaly-Detection-Pre-Configured-Solution-1" target="_blank">**從這裡開始 >**</a>
> 
>

## <a name="api-deployment"></a>API 部署
若要使用 API，您必須將它部署到 Azure 訂用帳戶，以在其中裝載成 Azure Machine Learning Web 服務。  您可以從 [Azure AI 資源庫](https://gallery.cortanaintelligence.com/MachineLearningAPI/Anomaly-Detection-2).\(英文\) 執行此動作。  這會將兩個 AzureML Web 服務 (與其相關的資源) 部署到您的 Azure 訂用帳戶 - 一個用於異常偵測 (含季節性偵測)，另一個則不含季節性偵測。  部署完成後，您就能夠從 [AzureML Web 服務](https://services.azureml.net/webservices/)頁面管理您的 API。  從這個頁面，您可以找到您的端點位置、API 金鑰，以及用於呼叫 API 的範例程式碼。  在[這裡](https://docs.microsoft.com/azure/machine-learning/machine-learning-manage-new-webservice)可取得更詳細的指示。

## <a name="scaling-the-api"></a>調整 API
根據預設，部署將有免費的開發/測試計費方案，其中包括 1,000 筆交易/月和 2 個計算時數/月。  根據您的需求，您可以升級到另一個方案。  在[這裡](https://azure.microsoft.com/pricing/details/machine-learning/) (「生產 Web API 價格」之下) 可取得不同方案的價格詳細資料。

## <a name="managing-aml-plans"></a>管理 AML 方案 
您可以在[這裡](https://services.azureml.net/plans/)管理您的計費方案。  方案名稱會以您在部署 API 時選擇的資源群組名稱為主，加上您的訂用帳戶中的唯一字串。  在[這裡](https://docs.microsoft.com/azure/machine-learning/machine-learning-manage-new-webservice) (「管理計費方案」一節之下) 可取得有關如何升級方案的指示。

## <a name="api-definition"></a>API 定義
Web 服務提供透過 HTTPS 的 REST 型 API，可以各種不同方式使用，包括 Web 或行動應用程式、R、Python、Excel 等等。您可以透過 REST API 呼叫將您的時間序列資料傳送到此服務，它會執行上述三個異常類型的組合。

## <a name="calling-the-api"></a>呼叫 API
若要呼叫 API，您必須知道端點位置和 API 金鑰。  從 [AzureML Web 服務](https://services.azureml.net/webservices/)頁面可取得這兩者，以及用於呼叫 API 的範例程式碼。  瀏覽至所需的 API，然後按一下 [取用] 索引標籤以找出它們。  請注意，您可以呼叫 API 做為 Swagger API (即包含 URL 參數 `format=swagger`) 或做為非 Swagger API (即不含 `format` URL 參數)。  範例程式碼會使用 Swagger 格式。  以下是非 Swagger 格式的範例要求和回應。  這些範例適用於季節性端點。  非季節性端點很類似。

### <a name="sample-request-body"></a>範例要求本文
要求包含兩個物件︰`Inputs` 和 `GlobalParameters`。  在下列範例要求中，某些參數會明確傳送，有些則不會 (向下捲動以取得每個端點的完整參數清單)。  不會在要求中明確傳送的參數會使用下面所列的預設值。

    {
                "Inputs": {
                        "input1": {
                                "ColumnNames": ["Time", "Data"],
                                "Values": [
                                        ["5/30/2010 18:07:00", "1"],
                                        ["5/30/2010 18:08:00", "1.4"],
                                        ["5/30/2010 18:09:00", "1.1"]
                                ]
                        }
                },
        "GlobalParameters": {
            "tspikedetector.sensitivity": "3",
            "zspikedetector.sensitivity": "3",
            "bileveldetector.sensitivity": "3.25",
            "detectors.spikesdips": "Both"
        }
    }

### <a name="sample-response"></a>範例回應
請注意，若要查看 `ColumnNames` 欄位，您必須將 `details=true` 納入為要求中的 URL 參數。  請參閱下表，以了解每個欄位背後的意義。

    {
        "Results": {
            "output1": {
                "type": "table",
                "value": {
                    "Values": [
                        ["5/30/2010 6:07:00 PM", "1", "1", "0", "0", "-0.687952590518378", "0", "-0.687952590518378", "0", "-0.687952590518378", "0"],
                        ["5/30/2010 6:08:00 PM", "1.4", "1.4", "0", "0", "-1.07030497733224", "0", "-0.884548154298423", "0", "-1.07030497733224", "0"],
                        ["5/30/2010 6:09:00 PM", "1.1", "1.1", "0", "0", "-1.30229513613974", "0", "-1.173800281031", "0", "-1.30229513613974", "0"]
                    ],
                    "ColumnNames": ["Time", "OriginalData", "ProcessedData", "TSpike", "ZSpike", "BiLevelChangeScore", "BiLevelChangeAlert", "PosTrendScore", "PosTrendAlert", "NegTrendScore", "NegTrendAlert"],
                    "ColumnTypes": ["DateTime", "Double", "Double", "Double", "Double", "Double", "Int32", "Double", "Int32", "Double", "Int32"]
                }
            }
        }
    }


## <a name="score-api"></a>分數 API
分數 API 可用來對非季節性的時間序列資料執行異常偵測。 API 會對此資料執行許多異常偵測器，並傳回其異常分數。 下圖顯示分數 API 可偵測到的異常範例。 此時間序列有 2 個不同的層級變更和 3 個尖峰。 紅點顯示偵測到層級變更時的時間，黑點則顯示偵測到的尖峰。
![分數 API][1]

### <a name="detectors"></a>偵測器
異常偵測 API 支援 3 大類別的偵測器。 下表中可以找到每個偵測器的特定輸入參數和輸出的詳細資料。

| 偵測器類別 | 偵測器 | 說明 | 輸入參數 | 輸出 |
| --- | --- | --- | --- | --- |
| 尖峰偵測器 |TSpike 偵測器 |偵測尖峰和下降是根據值與第一個和第三個四分位數的差距 |*tspikedetector.sensitivity*：接受範圍 1 - 10 的整數值，預設值︰3；值愈高，會捕捉到愈極端的值，因此降低敏感度 |TSpike︰二進位值 - 如果偵測到尖峰/下降則為 ‘1’，否則為 ‘0’ |
| 尖峰偵測器 | ZSpike 偵測器 |偵測尖峰和下降是根據資料點與平均數的差距 |*zspikedetector.sensitivity*：接受範圍 1 - 10 的整數值，預設值︰3；值愈高，會捕捉到愈極端的值，而降低敏感度 |ZSpike︰二進位值 - 如果偵測到尖峰/下降則為 ‘1’，否則為 ‘0’ | |
| 緩慢趨勢偵測器 |緩慢趨勢偵測器 |根據所設定的敏感度偵測緩慢的正向趨勢 | 偵測器分數的臨界值 (預設值︰3.25，3.25 – 5 是合理值的選取範圍；值愈高，敏感度越低) |tscore︰代表趨勢異常分數的浮動數字 |
| 層級變更偵測器 | 雙向層級變更偵測器 |根據所設定的敏感度偵測向上和向下層級變更 | 偵測器分數的臨界值 (預設值︰3.25，3.25 – 5 是合理值的選取範圍；值愈高，敏感度越低) |rpscore︰代表向上和向下層級變更異常分數的浮動數字 | |

### <a name="parameters"></a>參數
下表列出這些輸入參數的詳細資訊：

| 輸入參數 | 說明 | 預設設定 | 類型 | 有效範圍 | 建議範圍 |
| --- | --- | --- | --- | --- | --- |
| detectors.historyWindow |用於計算異常分數的歷程記錄 (以資料點數目為單位) |500 |integer |10 - 2000 |取決於時間序列 |
| detectors.spikesdips | 是否偵測只有尖峰、只有下降，或兩者 |兩者 |列舉 |兩者、尖峰、下降 |兩者 |
| bileveldetector.sensitivity |雙向層級變更偵測器的敏感度。 |3.25 |double |None |3.25-5 (值愈低代表敏感度越高) |
| trenddetector.sensitivity |正向趨勢偵測器的敏感度。 |3.25 |double |None |3.25-5 (值愈低代表敏感度越高) |
| tspikedetector.sensitivity |TSpike 偵測器的敏感度 |3 |integer |1 - 10 |3-5 (值愈低代表敏感度越高) |
| zspikedetector.sensitivity |ZSpike 偵測器的敏感度 |3 |integer |1 - 10 |3-5 (值愈低代表敏感度越高) |
| postprocess.tailRows |輸出結果中要保留的最新資料點數目 |0 |integer |0 (保留所有資料點)，或指定要在結果中保留的資料點數目 |N/A |

### <a name="output"></a>輸出
API 會對您的時間序列資料執行所有偵測器，然後傳回每個時間點的異常分數和二進位尖峰指示器。 下表列出 API 的輸出。 

| 輸出 | 說明 |
| --- | --- |
| 時間 |未經處理資料或彙總 (和/或) 插補資料 (如果套用彙總 (和/或) 遺漏資料插補) 的時間戳記 |
| 資料 |未經處理資料或彙總 (和/或) 插補資料 (如果套用彙總 (和/或) 遺漏資料插補) 的值 |
| TSpike |指出 TSpike 偵測器是否要偵測尖峰的二進位指示器 |
| ZSpike |指出 Zspike 偵測器是否要偵測尖峰的二進位指示器 |
| rpscore |代表雙向層級變更異常分數的浮動數字 |
| rpalert |根據輸入敏感度指出有雙向層級變更異常的 1/0 值 |
| tscore |代表正向趨勢異常分數的浮動數字 |
| talert |根據輸入敏感度指出有正向趨勢異常的 1/0 值 |

## <a name="scorewithseasonality-api"></a>ScoreWithSeasonality API
ScoreWithSeasonality API 可用來對具有季節性模式的時間序列執行異常偵測。 這個 API 適用於偵測季節性模式的偏差。  
下圖顯示季節性時間序列中所偵測到的異常範例。 時間序列有一個尖峰 (第 1 個黑點)、兩個下降 (第 2 個黑點和尾端的黑點)，以及一個層級變更 (紅點)。 請注意，從時間序列中移除季節性元件後，才能看到序列中間的兩個下降和該層級變更。
![季節性 API][2]

### <a name="detectors"></a>偵測器
季節性端點中的偵測器類似於非季節性端點中的偵測器，但參數名稱稍有不同 (如下所列)。

### <a name="parameters"></a>參數

下表列出這些輸入參數的詳細資訊：

| 輸入參數 | 說明 | 預設設定 | 類型 | 有效範圍 | 建議範圍 |
| --- | --- | --- | --- | --- | --- |
| preprocess.aggregationInterval |用來彙總輸入時間序列的彙總間隔 (秒) |0 (不執行彙總) |integer |0︰略過彙總，否則 > 0 |5 分鐘到 1 天，取決於時間序列 |
| preprocess.aggregationFunc |用來將資料彙總到指定 AggregationInterval 的函式 |平均值 |列舉 |平均值、總和、長度 |N/A |
| preprocess.replaceMissing |用來插補遺漏資料的值 |lkv (上一個已知值) |列舉 |零、lkv、平均值 |N/A |
| detectors.historyWindow |用於計算異常分數的歷程記錄 (以資料點數目為單位) |500 |integer |10 - 2000 |取決於時間序列 |
| detectors.spikesdips | 是否偵測只有尖峰、只有下降，或兩者 |兩者 |列舉 |兩者、尖峰、下降 |兩者 |
| bileveldetector.sensitivity |雙向層級變更偵測器的敏感度。 |3.25 |double |None |3.25-5 (值愈低代表敏感度越高) |
| postrenddetector.sensitivity |正向趨勢偵測器的敏感度。 |3.25 |double |None |3.25-5 (值愈低代表敏感度越高) |
| negtrenddetector.sensitivity |負向趨勢偵測器的敏感度。 |3.25 |double |None |3.25-5 (值愈低代表敏感度越高) |
| tspikedetector.sensitivity |TSpike 偵測器的敏感度 |3 |integer |1 - 10 |3-5 (值愈低代表敏感度越高) |
| zspikedetector.sensitivity |ZSpike 偵測器的敏感度 |3 |integer |1 - 10 |3-5 (值愈低代表敏感度越高) |
| seasonality.enable |是否要執行季節性分析 |true |布林值 |true、false |取決於時間序列 |
| seasonality.numSeasonality |要偵測的定期循環數目上限 |1 |integer |1、2 |1 - 2 |
| seasonality.transform |在套用異常偵測之前，是否應該移除季節性 (和) 趨勢元件 |deseason |列舉 |無、deseason、deseasontrend |N/A |
| postprocess.tailRows |輸出結果中要保留的最新資料點數目 |0 |integer |0 (保留所有資料點)，或指定要在結果中保留的資料點數目 |N/A |

### <a name="output"></a>輸出
API 會對您的時間序列資料執行所有偵測器，然後傳回每個時間點的異常分數和二進位尖峰指示器。 下表列出 API 的輸出。 

| 輸出 | 說明 |
| --- | --- |
| 時間 |未經處理資料或彙總 (和/或) 插補資料 (如果套用彙總 (和/或) 遺漏資料插補) 的時間戳記 |
| OriginalData |未經處理資料或彙總 (和/或) 插補資料 (如果套用彙總 (和/或) 遺漏資料插補) 的值 |
| ProcessedData |下列任一項︰ <ul><li>已進行季節性調整的時間序列 (在已偵測到明顯季節性並選取了 deseason 選項的前提下)</li><li>已進行季節性調整並去趨勢化的時間序列 (在已偵測到明顯季節性並選取了 deseasontrend 選項的前提下)</li><li>否則，與 OriginalData 相同</li> |
| TSpike |指出 TSpike 偵測器是否要偵測尖峰的二進位指示器 |
| ZSpike |指出 Zspike 偵測器是否要偵測尖峰的二進位指示器 |
| BiLevelChangeScore |代表層級變更異常分數的浮動數字 |
| BiLevelChangeAlert |根據輸入敏感度指出有層級變更異常的 1/0 值 |
| PosTrendScore |代表正向趨勢異常分數的浮動數字 |
| PosTrendAlert |根據輸入敏感度指出有正向趨勢異常的 1/0 值 |
| NegTrendScore |代表負向趨勢異常分數的浮動數字 |
| NegTrendAlert |根據輸入敏感度指出有負向趨勢異常的 1/0 值 |

[1]: ./media/apps-anomaly-detection-api/anomaly-detection-score.png
[2]: ./media/apps-anomaly-detection-api/anomaly-detection-seasonal.png

