---
title: 預測性維護解決方案加速器概觀 - Azure | Microsoft Docs
description: Azure IoT 預測性維護解決方案加速器的概觀。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 07/12/2018
ms.author: dobett
ms.openlocfilehash: e7c6c8d017e4371919088ec414d3108939ca4a19
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036249"
---
# <a name="predictive-maintenance-solution-accelerator-overview"></a>預測性維護解決方案加速器概觀

預測性維護解決方案加速器是一個端對端解決方案，適用於預測可能發生失敗之時間點的商務案例。 您可以針對最佳化維護等活動，主動使用此解決方案加速器。 此解決方案結合了主要 Azure IoT 解決方案加速器服務，例如 IoT 中樞、串流分析和 [Azure Machine Learning][lnk-machine-learning] 工作區。 此工作區包含以公開範例資料集為基礎的模型，用來預測飛機引擎的剩餘使用年限 (RUL)。 此解決方案完整提供 IoT 商務案例的實作作為起點，讓您規劃和實作能滿足特定商務需求的解決方案。

## <a name="logical-architecture"></a>邏輯架構

下圖概述解決方案加速器的邏輯元件：

![][img-architecture]

藍色項目是在您佈建解決方案加速器的區域中所佈建的 Azure 服務。 您可以在其中部署解決方案加速器的地區清單會顯示於[佈建頁面][lnk-azureiotsuite]。

綠色項目是表示飛機引擎的模擬裝置。 您可以在[模擬裝置](#simulated-devices)一節中進一步了解這些模擬裝置。

灰色項目代表可實作「裝置管理」功能的元件。 目前的預測性維護解決方案加速器版本不會佈建這些資源。 若要深入了解裝置管理，請參閱[遠端監視解決方案加速器][lnk-remote-monitoring]。

## <a name="azure-resources"></a>Azure 資源

在 Azure 入口網站中，瀏覽至具有您所選之解決方案名稱的資源群組以檢視已佈建的資源。

![加速器資源][img-resource-group]

當您佈建解決方案加速器時，您會收到一封電子郵件，其中包含機器學習服務工作區的連結。 您也可以從已佈建解決方案的 [Microsoft Azure IoT 解決方案加速器][lnk-azureiotsuite]頁面，瀏覽至 Machine Learning 工作區。 當解決方案處於**就緒**狀態時，此頁面上會出現一個圖格。

![機器學習服務模型][img-machine-learning]

## <a name="simulated-devices"></a>模擬的裝置

在解決方案加速器中，模擬裝置代表飛機引擎。 此解決方案已佈建兩個對應至單一飛機的引擎。 每個引擎會發出四種類型的遙測：感應器 9、感應器 11、感應器 14 和感應器 15 會提供 Machine Learning 模型來計算該引擎的 RUL 所需的資料。 每個模擬的裝置會將下列遙測訊息傳送至 IoT 中樞：

*週期計數*。 週期表示已完成持續期間介於 2 小時與 10 小時之間的飛行。 在飛行期間，每半小時擷取一次遙測資料。

*遙測*。 有四個代表引擎屬性的感應器。 這些感應器會一般會標示為感應器 9、感應器 11、感應器 14 和感應器 15。 這 4 個感應器代表足以從 RUL 模型取得有用結果的遙測。 用於解決方案加速器中的模型是根據包含實際引擎感應器資料的公用資料集建立而來。 如需有關如何從原始資料集建立模型的詳細資訊，請參閱 [Cortana Intelligence Gallery 預測性維護範本][lnk-cortana-analytics]。

模擬的裝置可以處理從解決方案中 IoT 中樞傳送的下列命令：

| 命令 | 說明 |
| --- | --- |
| StartTelemetry |控制模擬的狀態。<br/>傳送遙測以啟動裝置 |
| StopTelemetry |控制模擬的狀態。<br/>傳送遙測以停止裝置 |

IoT 中樞會提供裝置命令通知。

## <a name="azure-stream-analytics-job"></a>Azure 串流分析作業

**作業：遙測**會使用兩個陳述式來操作傳入裝置遙測串流：

* 第一個陳述式會從裝置選取所有遙測資料，然後將此資料傳送至 bob 儲存體。 從這裡，它會呈現在 Web 應用程式中。
* 第二個陳述式會透過兩分鐘的滑動視窗計算感應器平均值，然後透過事件中樞將此資料傳送至**事件處理器**。

## <a name="event-processor"></a>事件處理器
**事件處理器主機**會在 Azure Web 作業中執行。 **事件處理器** 需要一個完整的週期來處理平均感應器值。 接著將這些值傳遞至 API，該 API 會公開定型模型來計算引擎的 RUL。 此 API 是由佈建為方案一部分的 Machine Learning 工作區所公開。

## <a name="machine-learning"></a>Machine Learning
Machine Learning 元件使用的模型衍生自從真實飛機引擎收集而來的資料。 您可以在 [azureiotsuite.com][lnk-azureiotsuite] 頁面上，您從的解決方案圖格瀏覽至 Machine Learning 工作區。 當解決方案處於**就緒**狀態時，就會出現此圖格。

Azure Machine Learning 模型可以作為範本，示範這些功能如何運用透過 IoT 解決方案加速器服務收集而來的裝置遙測資料。 Microsoft 已根據公開可用的資料<sup>\[1\]</sup>，以及如何使用模型的逐步指引，建置了一套飛機引擎的[迴歸模型][lnk_regression_model]。

Azure IoT 預測性維護解決方案加速器會利用從這個範本建立的迴歸模型。 此模型會部署到您的 Azure 訂用帳戶，並透過自動產生的 API 加以公開。 此解決方案包含了代表 4 具 (全部共 100 具) 引擎之測試資料與 4 個 (全部共 21 個) 感應器之資料流的交集。 此資料足以從定型的模型提供精確的結果。

*\[1\] A. Saxena and K. Goebel (2008)。《渦輪風扇發動機性能下降模擬資料集》(Turbofan Engine Degradation Simulation Data Set)，NASA Ames Prognostics Data Repository (https://c3.nasa.gov/dashlink/resources/139/), NASA Ames Research Center, Moffett Field, CA*

## <a name="next-steps"></a>後續步驟
您現在已看到預測性維護解決方案加速器的主要元件，您可加以自訂。

您也可以探索 IoT 解決方案加速器的一些其他特性與功能：

* [IoT 解決方案加速器的常見問題集][lnk-faq]
* [從頭建立 IoT 安全性][lnk-security-groundup]

[img-architecture]: media/iot-accelerators-predictive-walkthrough/architecture.png
[img-resource-group]: media/iot-accelerators-predictive-walkthrough/resource-group.png
[img-machine-learning]: media/iot-accelerators-predictive-walkthrough/machine-learning.png

[lnk-remote-monitoring]: quickstart-predictive-maintenance-deploy.md
[lnk-cortana-analytics]: http://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Template-3
[lnk-azureiotsuite]: https://www.azureiotsolutions.com/
[lnk-faq]: iot-accelerators-faq.md
[lnk-security-groundup]:/azure/iot-fundamentals/iot-security-ground-up
[lnk-machine-learning]: https://azure.microsoft.com/services/machine-learning/
[lnk_regression_model]: http://gallery.cortanaanalytics.com/Collection/Predictive-Maintenance-Template-3