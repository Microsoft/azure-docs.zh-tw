---
title: Azure IoT 中樞簡介 | Microsoft Docs
description: 了解 Azure IoT 中樞。 此 IoT 服務是針對可調整規模的資料擷取、裝置管理和安全性所建置。
author: nberdy
ms.author: nberdy
ms.date: 07/04/2018
ms.topic: overview
ms.custom: mvc
ms.service: iot-hub
services: iot-hub
manager: briz
ms.openlocfilehash: f4254cd90d8cf3b9f4cd206b729a3d44784b377a
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39343622"
---
# <a name="what-is-azure-iot-hub"></a>何謂 Azure IoT 中心？

IoT 中樞是託管於雲端中的受控服務，可做為 IoT 應用程式與其管理裝置之間雙向通訊的中央訊息中樞。 您可以使用 Azure IoT 中樞，藉由數百萬部 IoT 裝置與雲端主控方案後端之間可靠且安全的通訊，以建置 IoT 解決方案。 您可以將大部分裝置連線到 IoT 中樞。

IoT 中樞支援從裝置到雲端以及從雲端到裝置的通訊。 IoT 中樞支援多種傳訊模式，如裝置到雲端遙測、從裝置上傳檔案，以及透過「要求-回覆」方法從雲端控制您的裝置。 IoT 中樞監視可追蹤裝置建立、裝置失敗，以及裝置連線等事件，以協助您維護解決方案的健全狀況。

IoT 中樞的功能可協助您建置可調整規模且功能完整的 IoT 解決方案，例如管理製造業中使用的工業設備、追蹤醫療保健中的重要資產，以及監視辦公大樓使用狀況。

## <a name="scale-your-solution"></a>調整您的解決方案規模

IoT 中樞可以調整規模以因應數百萬個同時連線的裝置，以及每秒數百萬個事件，以支援您的 IoT 工作負載。 IoT 中樞提供了多種服務級別，以滿足您的擴充需求。 若要深入了解，請查看[定價頁面](https://azure.microsoft.com/pricing/details/iot-hub/)。

## <a name="secure-your-communications"></a>保護您的通訊

IoT 中樞提供了安全的通訊通道，以便讓裝置傳送資料。

* 針對裝置的驗證可讓每個裝置安全地連線到 IoT 中樞，並安全地管理每個裝置。

* 您可以完全控制裝置存取，並且可以控制裝置層級的連線。

* 裝置第一次開機時，[IoT 中樞裝置佈建服務](https://docs.microsoft.com/azure/iot-dps/)會自動將裝置佈建到正確的 IoT 中樞。

* 多個驗證類型可支援各式各樣的裝置功能：

  * 可快速開始使用 IoT 解決方案的 SAS 權杖型驗證。

  * 可用於安全標準式驗證的個別 X.509 憑證驗證。

  * 可用於簡單標準式註冊的 X.509 CA 驗證。

## <a name="route-device-data"></a>路由裝置資料

內建訊息路由功能可讓您彈性地設定自動化規則型訊息展開傳送：

* 使用訊息路由來控制中樞傳送裝置遙測的位置。

* 將訊息路由傳送至多個端點不會有額外成本。

* 無程式碼路由規則取代了自訂訊息發送器程式碼。

## <a name="integrate-with-other-services"></a>與其他服務整合

您可以將 IoT 中樞與其他 Azure 服務整合，以建置完整的解決方案。 例如，使用：

* [Azure Event Grid](https://docs.microsoft.com/azure/event-grid/) 可讓您的企業以可靠、可調整規模且安全的方式快速回應重大事件。

* [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/) 可自動化商業流程。

* [Azure Machine Learning](https://docs.microsoft.com/azure/machine-learning/) 可將機器學習和 AI 模型加入您的解決方案。

* [Azure 串流分析](https://docs.microsoft.com/azure/stream-analytics/)可針對來自裝置的資料流執行即時分析運算。

## <a name="configure-and-control-your-devices"></a>設定及控制您的裝置

您可以使用一系列內建功能來管理連線到 IoT 中樞的裝置。

* 針對您的所有裝置，儲存、同步處理，以及查詢裝置中繼資料與狀態資訊。

* 依裝置或根據裝置一般特性來設定裝置狀態。

* 透過訊息路由整合，自動回應裝置回報的狀態變更。

## <a name="make-your-solution-highly-available"></a>讓您的解決方案保持高可用性

我們提供[適用於 IoT 中樞的 99.9% 服務等級協定](https://azure.microsoft.com/support/legal/sla/iot-hub/)。 完整 [Azure SLA](https://azure.microsoft.com/support/legal/sla/) 說明保證的 Azure 整體可用性。

## <a name="connect-your-devices"></a>連線您的裝置

使用 [Azure IoT 裝置 SDK](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-sdks) 程式庫，以建置在您的裝置上執行並與 IoT 中樞互動的應用程式。 支援的平台包括多個 Linux 散發套件、Windows 和即時作業系統。 支援的語言包括：

* C
* C#
* Java
* Python
* Node.js。

IoT 中樞和裝置 SDK 支援下列連線裝置的通訊協定：

* HTTPS
* AMQP
* 透過 WebSocket 的 AMQP
* MQTT
* 透過 WebSocket 的 MQTT

如果您的解決方案不能使用裝置程式庫，則裝置可以使用 MQTT 3.1.1 版、HTTPS 1.1 或 AMQP 1.0 通訊協定，以原生方式連線至您的中樞。

如果您的解決方案無法使用任一種支援的通訊協定，則您可以擴充 IoT 中樞以支援自訂通訊協定：

* 使用 [Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/) 建立一個現場閘道，以在 Edge 上執行通訊協定轉譯。

* 自訂 [Azure IoT 通訊協定閘道](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)，以在雲端中執行通訊協定轉譯。

## <a name="quotas-and-limits"></a>配額和限制

每個 Azure 訂用帳戶都有預設的配額限制，以避免濫用服務。而這些限制會對 IoT 解決方案的範圍造成影響。 依訂用帳戶而定的目前限制是每一訂用帳戶 10 個 IoT 中樞。 您可以連絡支援人員，以要求增加此配額。 如需有關配額限制的詳細資料：

* [Azure 訂用帳戶服務限制](../azure-subscription-service-limits.md)

* [IoT 中樞節流與您](https://azure.microsoft.com/blog/iot-hub-throttling-and-you/) \(英文\)

## <a name="next-steps"></a>後續步驟

若要嘗試完整的 IoT 解決方案，請參閱 IoT 中樞快速入門：

* [快速入門：將遙測從裝置傳送到 IoT 中樞](quickstart-send-telemetry-node.md)