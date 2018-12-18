---
title: Azure IoT 通訊協定閘道 | Microsoft Docs
description: 如何使用 Azure IoT 通訊協定閘道器來擴充 IoT 中樞功能和通訊協定支援，讓裝置能夠使用 IoT 中樞原本不支援的通訊協定來連接至您的中樞。
author: fsautomata
manager: ''
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/11/2017
ms.author: elioda
ms.openlocfilehash: 2c90ee899d0002d41ca21ed4a4927470ee53b2e1
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34635299"
---
# <a name="support-additional-protocols-for-iot-hub"></a>讓 IoT 中樞支援其他通訊協定
「Azure IoT 中樞」原生就支援透過 MQTT、AMQP 及 HTTPS 通訊協定進行通訊。 在某些情況下，裝置或現場閘道可能無法使用這些標準通訊協定之一，因此需要通訊協定調適。 在這種情況下，您可以使用自訂閘道。 自訂閘道可以橋接進出 IoT 中樞的流量，藉此為 IoT 中樞端點啟用通訊協定調適。 您可以使用 [Azure IoT 通訊協定閘道](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md) 做為自訂閘道，來為 IoT 中樞啟用通訊協定調適。

## <a name="azure-iot-protocol-gateway"></a>Azure IoT 通訊協定閘道
Azure IoT 通訊協定閘道是通訊協定調適的架構，專門用來與 IoT 中樞的高延展性雙向裝置通訊。 通訊協定閘道是一種傳遞元件，會透過特定通訊協定接受裝置的連接。 透過 AMQP 1.0 橋接 IoT 中樞的流量。 

您可以使用 Azure Service Fabric、「Azure 雲端服務」背景工作角色或「Windows 虛擬機器」，以可靈活調整的方式，在 Azure 中部署通訊協定閘道。 此外，通訊協定閘道可以部署在內部部署環境中，例如現場閘道。

Azure IoT 通訊協定閘道包含可讓您視需要自訂 MQTT 通訊協定行為的 MQTT 通訊協定配接器。 由於「IoT 中樞」針對 MQTT v3.1.1 通訊協定提供內建支援，因此您應該只有在需要自訂通訊協定或有額外功能的特定需求時，才考慮使用 MQTT 通訊協定配接器。

MQTT 配接器也會示範程式設計模型，此模型可為其他通訊協定建置通訊協定配接器。 此外，Azure IoT 通訊協定閘道程式設計模型還可讓您針對特製化處理 (例如自訂驗證、訊息轉換、壓縮/解壓縮，或將裝置與「IoT 中樞」之間的流量加密/解密) 插入自訂元件。

為了讓您有彈性空間，Azure IoT 通訊協定閘道和 MQTT 實作是以開放原始碼軟體專案的方式來提供。 您可以使用開放原始碼專案，以新增對各種通訊協定和通訊協定版本的支援，或是自訂案例的實作。 

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure IoT 通訊協定閘道，以及如何使用並將其部署為 IoT 方案的一部分，請參閱：

* [GitHub 上的 Azure IoT 通訊協定閘道儲存機制 (英文)](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)
* [Azure IoT 通訊協定閘道開發人員指南 (英文)](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)

若要深入了解如何規劃 IoT 中樞部署，請參閱：

* [與事件中樞比較][lnk-compare]
* [調整規格、高可用性和災害復原][lnk-scaling]
* [IoT 中樞開發人員指南][lnk-devguide]

[lnk-compare]: iot-hub-compare-event-hubs.md
[lnk-scaling]: iot-hub-scaling.md
[lnk-devguide]: iot-hub-devguide.md
