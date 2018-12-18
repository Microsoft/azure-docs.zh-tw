---
title: 使用 Azure Machine Learning 搭配來自 IoT 中樞的資料進行氣象預報 | Microsoft Docs
description: 使用 Azure Machine Learning，根據 IoT 中樞透過感應器收集的溫度和溼度資料來預測下雨的機會。
author: rangv
manager: ''
keywords: 氣象預報機器學習
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/11/2018
ms.author: rangv
ms.openlocfilehash: a331f8a8a69ffe41a368c1b36f1680890aaac8bf
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38666869"
---
# <a name="weather-forecast-using-the-sensor-data-from-your-iot-hub-in-azure-machine-learning"></a>在 Azure Machine Learning 中使用 IoT 中樞的感應器資料進行氣象預報

![端對端圖表](media/iot-hub-get-started-e2e-diagram/6.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

機器學習服務是一項資料科學技術，協助電腦從現有的資料學習，以便預測未來的行為、結果和趨勢。 Azure Machine Learning 是雲端預測性分析服務，可讓您快速地建立預測模型，並將其部署為分析解決方案。

## <a name="what-you-learn"></a>您學到什麼

了解如何使用 Azure Machine Learning 透過來自您的 Azure IoT 中樞的溫度和溼度資料執行氣象預報 (下雨的機會)。 下雨的機會是備妥的氣象預報模型的輸出。 此模型的建置是根據溫度和溼度的歷史資料來預測下雨的機會。

## <a name="what-you-do"></a>您要做什麼

- 將氣象預報模型部署為 Web 服務。
- 新增取用者群組，讓您的 IoT 中樞準備好進行資料存取。
- 建立串流分析作業，以及設定作業：
  - 從 IoT 中樞讀取溫度和溼度資料。
  - 呼叫 Web 服務來取得降雨機會。
  - 將結果儲存至 Azure Blob 儲存體。
- 使用 Microsoft Azure 儲存體總管來檢視氣象預報。

## <a name="what-you-need"></a>您需要什麼

- 完成涵蓋下列需求的[設定裝置](iot-hub-raspberry-pi-kit-node-get-started.md)教學課程︰
  - 有效的 Azure 訂用帳戶。
  - 位於您訂用帳戶中的 Azure IoT 中樞。
  - 將訊息傳送到您 Azure IoT 中樞的用戶端應用程式。
- Azure Machine Learning Studio 帳戶。 ([免費試用 Machine Learning Studio](https://studio.azureml.net/))。

## <a name="deploy-the-weather-prediction-model-as-a-web-service"></a>將氣象預報模型部署為 Web 服務

1. 移至 [氣象預報模型頁面](https://gallery.cortanaintelligence.com/Experiment/Weather-prediction-model-1)。
1. 在 Microsoft Azure Machine Leaning Studio 中按一下 [在 Studio 中開啟]。
   ![在 Cortana Intelligence 資源庫中開啟氣象預報模型頁面](media/iot-hub-weather-forecast-machine-learning/2_weather-prediction-model-in-cortana-intelligence-gallery.png)
1. 按一下 [執行] 來驗證模型中的步驟。 此步驟可能需要 2 分鐘才能完成。
   ![在 Azure Machine Learning Studio 中開啟氣象預報模型](media/iot-hub-weather-forecast-machine-learning/3_open-weather-prediction-model-in-azure-machine-learning-studio.png)
1. 按一下 [設定 Web 服務] > [預測性 Web 服務]。
   ![在 Azure Machine Learning Studio 中部署氣象預報模型](media/iot-hub-weather-forecast-machine-learning/4-deploy-weather-prediction-model-in-azure-machine-learning-studio.png)
1. 在圖表中，將 [Web 服務輸入] 模組拖曳至接近 [評分模型] 模組附近的位置。
1. 將 [Web 服務輸入] 模組連線至 [評分模型] 模組。
   ![在 Azure Machine Learning Studio 中連線兩個模組](media/iot-hub-weather-forecast-machine-learning/13_connect-modules-azure-machine-learning-studio.png)
1. 按一下 [執行] 來驗證模型中的步驟。
1. 按一下 [部署 Web 服務] 來將模型部署為 Web 服務。
1. 在模型的儀表板上，下載**要求/回應**的 **Excel 2010 或舊版活頁簿**。

   > [!Note]
   > 即使您電腦上執行較新版的 Excel，也務必下載**Excel 2010 或舊版活頁簿**。

   ![下載要求回應端點的 Excel](media/iot-hub-weather-forecast-machine-learning/5_download-endpoint-app-excel-for-request-response.png)

1. 開啟 Excel 活頁簿，請記下 **Web 服務 URL** 和**存取金鑰**。

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="create-configure-and-run-a-stream-analytics-job"></a>設定、設定及執行串流分析作業

### <a name="create-a-stream-analytics-job"></a>建立串流分析作業

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [建立資源] > [物聯網] > [串流分析作業]。
1. 輸入作業的以下資訊。

   **作業名稱**：作業名稱。 此名稱必須是全域唯一的。

   **資源群組**︰使用 IoT 中樞所用的相同資源群組。

   **位置**︰使用與資源群組相同的位置。

   **釘選至儀表板**︰核取此選項可讓您從儀表板輕鬆地存取 IoT 中樞。

   ![在 Azure 中建立串流分析作業](media/iot-hub-weather-forecast-machine-learning/7_create-stream-analytics-job-azure.png)

1. 按一下頁面底部的 [新增] 。

### <a name="add-an-input-to-the-stream-analytics-job"></a>將輸入新增至串流分析作業

1. 開啟串流分析作業。
1. 在 [作業拓撲] 之下，按一下 [輸入]。
1. 在 [輸入] 窗格中，按一下 [新增]，然後輸入下列資訊︰

   **輸入別名**︰輸入的唯一別名。

   **來源**︰選取 [IoT 中樞]。

   **取用者群組**：選取您建立的取用者群組。

   ![在 Azure 中將輸入新增至串流分析作業](media/iot-hub-weather-forecast-machine-learning/8_add-input-stream-analytics-job-azure.png)

1. 按一下頁面底部的 [新增] 。

### <a name="add-an-output-to-the-stream-analytics-job"></a>將輸出新增至串流分析作業

1. 在 [作業拓撲] 之下，按一下 [輸出]。
1. 在 [輸出] 窗格中，按一下 [新增]，然後輸入下列資訊︰

   **輸出別名**︰輸出的唯一別名。

   **接收器**：選取 [Blob 儲存體]。

   **儲存體帳戶**：您 Blob 儲存體的儲存體帳戶。 您可以建立儲存體帳戶，或使用現有的帳戶。

   **容器**：儲存 Blob 所在位置的容器。 您可以建立容器或是使用現有的容器。

   **事件序列化格式**：選取 [CSV]。

   ![在 Azure 中將輸出新增至串流分析作業](media/iot-hub-weather-forecast-machine-learning/9_add-output-stream-analytics-job-azure.png)

1. 按一下頁面底部的 [新增] 。

### <a name="add-a-function-to-the-stream-analytics-job-to-call-the-web-service-you-deployed"></a>將功能新增至串流分析作業，以呼叫您所部署的 Web 服務

1. 在 [作業拓撲] 下，按一下 [函式] > [新增]。
1. 輸入以下資訊：

   **函式別名**：輸入 `machinelearning`。

   **函式類型**：選取 [Azure ML]。

   **匯入選項**：選取 [從不同的訂用帳戶匯入]。

   **URL**：輸入您從 Excel 活頁簿記下的 Web 服務 URL。

   **金鑰**：輸入您從 Excel 活頁簿記下的存取金鑰。

   ![在 Azure 中將功能新增至串流分析作業](media/iot-hub-weather-forecast-machine-learning/10_add-function-stream-analytics-job-azure.png)

1. 按一下頁面底部的 [新增] 。

### <a name="configure-the-query-of-the-stream-analytics-job"></a>設定串流分析作業的查詢

1. 在 [作業拓撲] 之下，按一下 [查詢]。
1. 將現有程式碼取代為下列程式碼：

   ```sql
   WITH machinelearning AS (
      SELECT EventEnqueuedUtcTime, temperature, humidity, machinelearning(temperature, humidity) as result from [YourInputAlias]
   )
   Select System.Timestamp time, CAST (result.[temperature] AS FLOAT) AS temperature, CAST (result.[humidity] AS FLOAT) AS humidity, CAST (result.[Scored Probabilities] AS FLOAT ) AS 'probabalities of rain'
   Into [YourOutputAlias]
   From machinelearning
   ```

   使用作業的輸入別名取代 `[YourInputAlias]`。

   使用作業的輸出別名取代 `[YourOutputAlias]`。

1. 按一下 [檔案] 。

### <a name="run-the-stream-analytics-job"></a>執行串流分析作業

在串流分析作業中，按一下 [啟動] > [立即] > [啟動]。 成功啟動作業後，作業狀態會從 [已停止] 變更為 [執行中]。

![執行串流分析作業](media/iot-hub-weather-forecast-machine-learning/11_run-stream-analytics-job-azure.png)

## <a name="use-microsoft-azure-storage-explorer-to-view-the-weather-forecast"></a>使用 Microsoft Azure 儲存體總管來檢視氣象預報

執行用戶端應用程式來開始收集和傳送溫度和溼度資料至 IoT 中樞。 對於 IoT 中樞收到的每個訊息，串流分析作業會呼叫氣象預報 Web 服務，以產生下雨的機會。 接著會將結果儲存到您的 Azure Blob 儲存體。 Azure 儲存體總管是一個工具，可供您用來檢視結果。

1. [下載並安裝 Microsoft Azure 儲存體總管](http://storageexplorer.com/)。
1. 開啟 [Azure 儲存體總管]。
1. 登入您的 Azure 帳戶。
1. 選取您的訂用帳戶。
1. 按一下您的訂用帳戶 > [儲存體帳戶] > 您的儲存體帳戶 > [Blob 容器] > 您的容器。
1. 開啟 .csv 檔案來查看結果。 最後一個資料行記錄下雨的機會。

   ![使用 Azure Machine Learning 取得氣象預報結果](media/iot-hub-weather-forecast-machine-learning/12_get-weather-forecast-result-azure-machine-learning.png)

## <a name="summary"></a>總結

您已成功使用 Azure Machine Learning 根據 IoT 中樞收到的溫度和溼度資料來產生下雨的機會。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]