---
title: 遠端監視解決方案加速器概觀 - Azure | Microsoft Docs
description: 遠端監視解決方案加速器的概觀。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 11/10/2017
ms.author: dobett
ms.openlocfilehash: dfe584532efeab1dbc0d2928b7afb0a6695a21ee
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39184940"
---
# <a name="remote-monitoring-solution-accelerator-overview"></a>遠端監視解決方案加速器概觀

遠端監視[解決方案加速器](../iot-accelerators/about-iot-accelerators.md)會在遠端位置實作多部機器的端對端監視解決方案。 此解決方案結合了主要的 Azure 服務以提供一般的商務案例實作。 您可以將此解決方案作為自己實作的起點並加以[自訂](../iot-accelerators/iot-accelerators-remote-monitoring-customize.md)，以符合自己特有的商務需求。

本文將逐步介紹遠端監視解決方案的一些重要元素，讓您瞭解它的運作方式。 這項知識能協助您︰

* 在解決方案中進行疑難排解。
* 規劃如何自訂解決方案以滿足您的特定需求。
* 設計使用 Azure 服務的 IoT 解決方案。

## <a name="logical-architecture"></a>邏輯架構

下圖概述 [IoT 架構](../iot-fundamentals/iot-introduction.md)上重疊的遠端監視解決方案加速器的邏輯元件：

![邏輯架構](./media/iot-accelerators-remote-monitoring-sample-walkthrough/remote-monitoring-architecture.png)

## <a name="why-microservices"></a>使用微服務的理由？

從 Microsoft 發行第一個解決方案加速器開始，就已發展雲端架構。 [微服務](https://azure.microsoft.com/blog/microservices-an-application-revolution-powered-by-the-cloud/)問世以來都是經證實的做法，可達成延展及彈性，而不必犧牲開發速度。 多項在內部使用此架構模式的 Microsoft 服務，都獲得絕佳的可靠性與延展性結果。 更新的解決方案加速器已將這些知識付諸實現，因此您也可以從中獲益。

> [!TIP]
> 若要了解微服務架構的詳細資訊，請參閱 [.NET 應用程式架構](https://www.microsoft.com/net/learn/architecture)和[微服務︰採用雲端技術的應用程式革命](https://azure.microsoft.com/blog/microservices-an-application-revolution-powered-by-the-cloud/)。

## <a name="device-connectivity"></a>裝置連線能力

解決方案在邏輯架構的裝置連線部分中包含下列元件：

### <a name="simulated-devices"></a>模擬的裝置

解決方案所包含的微服務可讓您管理模擬裝置的集區，從而測試解決方案中的端對端流程。 模擬的裝置：

* 產生裝置到雲端的遙測。
* 回應來自 IoT 中樞的雲端到裝置方法呼叫。

微服務會提供 RESTful 端點，讓您可建立、啟動及停止模擬。 每個模擬都是由一組不同類型的虛擬裝置所組成，可傳送遙測並回應方法呼叫。

您可以在解決方案入口網站的儀表板佈建模擬的裝置。

### <a name="physical-devices"></a>實體裝置

您可以將實體裝置連線至解決方案。 您可以使用 Azure IoT 裝置 SDK 來實作模擬裝置的行為。

您可以在解決方案入口網站的儀表板佈建實體裝置。

### <a name="iot-hub-and-the-iot-manager-microservice"></a>IoT 中樞與 IoT 管理員微服務

[IoT 中樞](../iot-hub/index.yml)會內嵌從裝置傳送至雲端的資料，並提供給 `telemetry-agent` 微服務。

解決方案中的 IoT 中樞也可︰

* 維護身分識別登錄，以儲存允許連線至入口網站之所有裝置的驗證金鑰。 您可以透過身分識別登錄來啟用和停用裝置。
* 代表解決方案入口網站在裝置上叫用方法。
* 維護所有已註冊裝置的裝置對應項。 裝置對應項會儲存裝置所報告的屬性值。 裝置對應項也會儲存在解決方案入口網站中設定的所需屬性，以便裝置在後續連接時擷取。
* 排程作業以設定多個裝置的屬性，或在多個裝置上叫用方法。

解決方案所包含的 `iot-manager` 微服務可處理與 IoT 中樞的互動，例如：

* 建立及管理 IoT 裝置。
* 管理裝置對應項。
* 在裝置上叫用方法。
* 管理 IoT 認證。

此服務也會執行 IoT 中樞查詢，以擷取屬於使用者定義群組的裝置。

微服務會提供 RESTful 端點來管理裝置和裝置對應項、叫用方法，以及執行 IoT 中樞查詢。

## <a name="data-processing-and-analytics"></a>資料處理和分析

解決方案在邏輯架構的資料處理和分析部分中包含下列元件：

### <a name="device-telemetry"></a>裝置遙測

解決方案包含兩個微服務可處理裝置遙測。

[telemetry-agent](https://github.com/Azure/telemetry-agent-dotnet) 微服務：

* 儲存 Azure Cosmos DB 中的遙測。
* 分析來自裝置的遙測串流。
* 會根據定義的規則產生警示。

警示會儲存在 Azure Cosmos DB 中。

[telemetry-agent](https://github.com/Azure/telemetry-agent-dotnet) 微服務可讓解決方案入口網站讀取裝置所傳送的遙測。 解決方案入口網站也會使用此服務來：

* 定義監視規則，例如可觸發警示的臨界值
* 擷取過去警示的清單。

您可以使用這個微服務所提供的 RESTful 端點來管理遙測、規則和警示。

### <a name="storage"></a>儲存體

[storage-adapter](https://github.com/Azure/pcs-storage-adapter-dotnet) \(英文\) 微服務是主要儲存體服務前面的配接器，可用於解決方案加速器。 它提供簡單的集合和機碼值儲存體。

解決方案加速器的標準部署會使用 Azure Cosmos DB 作為其主要的儲存體服務。

Azure Cosmos DB 資料庫會將資料儲存於解決方案加速器中。 **storage-adapter** 微服務在解決方案中是作為其他微服務的配接器，以存取儲存體服務。

## <a name="presentation"></a>展示

解決方案在邏輯架構的展示部分中包含下列元件：

[Web 使用者介面是 React Javascript 應用程式](https://github.com/Azure/pcs-remote-monitoring-webui)。 應用程式：

* 只會使用 Javascript React，並會完全在瀏覽器中執行。
* CSS 樣式。
* 會透過 AJAX 呼叫與公眾對應的微服務互動。

使用者介面會呈現所有解決方案加速器功能，並與其他服務互動，例如：

* [authentication](https://github.com/Azure/pcs-auth-dotnet) 微服務可保護使用者資料。
* [iothub-manager](https://github.com/Azure/iothub-manager-dotnet) 微服務可列出並管理 IoT 裝置。

[ui-config](https://github.com/Azure/pcs-config-dotnet) 微服務可讓使用者介面儲存及擷取組態集。

## <a name="next-steps"></a>後續步驟

如果您需要探索來源程式碼和開發人員文件，請開始使用兩個主要的 GitHub 存放庫其中之一：

* [使用 Azure IoT 的遠端監視解決方案加速器 (.NET)](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/) \(英文\)。
* [使用 Azure IoT 的遠端監視解決方案加速器 (Java)](https://github.com/Azure/azure-iot-pcs-remote-monitoring-java) \(英文\)。

詳細的方案架構圖表：
* [遠端監視架構的解決方案加速器](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Architecture) \(英文\)。

如需關於遠端監視解決方案加速器的詳細概念資訊，請參閱[自訂解決方案加速器](../iot-accelerators/iot-accelerators-remote-monitoring-customize.md)。
