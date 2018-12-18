---
title: Azure IoT Central 中的裝置連線能力 | Microsoft Docs
description: 本文介紹 Azure IoT Central 裝置連線能力的重要相關概念
author: dominicbetts
ms.author: dobett
ms.date: 11/30/2017
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: timlt
ms.openlocfilehash: dc9fe144c2258f33ce59c61ce63c15835cc3fa53
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34628329"
---
# <a name="device-connectivity-in-azure-iot-central"></a>Azure IoT Central 中的裝置連線能力

本文介紹 Microsoft Azure IoT Central 裝置連線能力的重要相關概念。

連線至 Azure IoT Central 應用程式的每個裝置，都會連線至 Azure IoT Central 所用 IoT 中樞所公開的[端點](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-endpoints)。 IoT 中樞可讓您從連網裝置建立可擴充、安全且可靠的連線。

## <a name="sdk-support"></a>SDK 支援

Azure 裝置 SDK 能讓您以最簡單的方式，在連線至 Azure IoT Central 應用程式的裝置上實作程式碼。 可用的裝置 SDK 如下：

- [Azure IoT SDK for C](https://github.com/azure/azure-iot-sdk-c)
- [Azure IoT SDK for Python](https://github.com/azure/azure-iot-sdk-python)
- [Azure IoT SDK for Node.js](https://github.com/azure/azure-iot-sdk-node)
- [Azure IoT SDK for Java](https://github.com/azure/azure-iot-sdk-java)
- [Azure IoT SDK for .NET](https://github.com/azure/azure-iot-sdk-csharp)

每個裝置在連線時，都會使用可識別裝置的唯一連接字串。 裝置只能連線至其註冊所在的 IoT 中樞。 當您在 Azure IoT Central 應用程式中建立真實裝置時，應用程式會產生連接字串供您使用。

## <a name="sdk-features-and-iot-hub-connectivity"></a>SDK 的功能和 IoT 中樞連線能力

所有裝置在與 IoT 中樞通訊時，都會使用下列 IoT 中樞連線能力選項：

- [裝置到雲端傳訊](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c)
- [裝置對應項](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-device-twins)

下表摘要說明 Azure IoT Central 裝置功能與 IoT 中樞功能的對應方式：

| Azure IoT 中心 | Azure IoT 中樞 |
| ----------- | ------- |
| 量測：遙測 | 裝置到雲端傳訊 |
| 裝置屬性 | 裝置對應項的報告屬性 |
| 設定 | 裝置對應項所需和所報告的屬性 |

若要深入了解如何使用裝置 SDK，請參閱下列其中一篇文章，以取得程式碼範例：

- [將一般 Node.js 用戶端連線至 Azure IoT Central 應用程式](howto-connect-nodejs.md)
- [將 Raspberry Pi 裝置連線到 Azure IoT Central 應用程式](howto-connect-raspberry-pi-python.md)
- [將 DevDiv kit 裝置連線到 Azure IoT Central 應用程式](howto-connect-devkit.md)

下列影片會介紹如何將真實裝置連線至 Azure IoT Central 的概觀：

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Connect-real-devices-to-Microsoft-IoT-Central/Player]

## <a name="protocols"></a>通訊協定

裝置 SDK 支援使用下列網路通訊協定來連線至 IoT 中樞：

- MQTT
- AMQP
- HTTPS

如需這些差異通訊協定和其選擇方針的資訊，請參閱[選擇通訊協定](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-protocols)。

如果裝置無法使用任何所支援的通訊協定，則可以使用 Azure IoT Edge 來轉換通訊協定。 IoT Edge 支援其他 intelligence-on-the-edge 案例，可從 Azure IoT Central 應用程式將處理卸載至邊緣。

## <a name="security"></a>安全性

在裝置與 Azure IoT Central 之間交換的資料會都加密。 IoT 中樞會對每個要從裝置連線至任何裝置面向 IoT 中樞端點的要求進行驗證。 為了避免透過網路交換認證，裝置會使用已簽署的權杖來進行驗證。 如需詳細資訊，請參閱[控制 IoT 中樞的存取權](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-security)。

> [!NOTE]
> 目前，連線至 Azure IoT Central 的裝置必須使用 SAS 權杖。 連線至 Azure IoT Central 的裝置不支援 X.509 憑證。

## <a name="next-steps"></a>後續步驟

現在，您已了解 Azure IoT Central 中的裝置連線能力，以下是建議的後續步驟：

- [準備及連線 DevKit 裝置](howto-connect-devkit.md)
- [準備及連線 Raspberry Pi](howto-connect-raspberry-pi-python.md)
- [將一般 Node.js 用戶端連線至 Azure IoT Central 應用程式](howto-connect-nodejs.md)
