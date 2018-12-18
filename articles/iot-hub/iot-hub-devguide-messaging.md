---
title: 了解 Azure IoT 中樞傳訊 | Microsoft Docs
description: 開發人員指南 - IoT 中樞的裝置到雲端及雲端到裝置傳訊。 其中包括訊息格式和支援之通訊協定的相關資訊。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: dobett
ms.openlocfilehash: 1915c4bc6cd611479c7575179d8fe64def8895eb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46956367"
---
# <a name="send-device-to-cloud-and-cloud-to-device-messages-with-iot-hub"></a>使用 IoT 中樞傳送裝置到雲端與雲端到裝置訊息

IoT 中樞能夠與您的裝置進行雙向通訊。 使用 IoT 中樞傳訊來與您的裝置通訊的方式是，將訊息從您的裝置傳送到您的解決方案後端，並將命令從您的 IoT 解決方案後端傳送到您的裝置。 深入了解 [IoT 中樞訊息格式](../iot-hub/iot-hub-devguide-messages-construct.md)。

## <a name="sending-device-to-cloud-messages-to-iot-hub"></a>將裝置到雲端訊息傳送到 IoT 中樞

IoT 中樞具有內建的服務端點，可供後端服務用於從您的裝置讀取遙測訊息。 此端點與[事件中樞](https://docs.microsoft.com/azure/event-hubs/)相容，而您可以使用標準 IoT 中樞 SDK，[從此內建端點讀取]((https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin))。

IoT 中樞也支援[自訂端點](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-endpoints#custom-endpoints)，可由使用者定義來使用[訊息路由](iot-hub-devguide-messages-d2c.md)，以將裝置遙測資料與事件傳送到 Azure 服務。

## <a name="sending-cloud-to-device-messages-from-iot-hub"></a>從 IoT 中樞傳送雲端到裝置訊息

您可以將[雲端到裝置](iot-hub-devguide-messages-c2d.md)訊息從解決方案後端傳送到您的裝置。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

IoT 中樞傳訊功能的核心屬性是訊息的可靠性和持久性。 這些屬性可在裝置端上恢復間歇性連線，以及在雲端恢復事件處理的負載尖峰。 IoT 中樞會針對裝置到雲端和雲端到裝置訊息，實作「至少一次」  傳遞保證。

## <a name="choosing-the-right-type-of-iot-hub-messaging"></a>選擇正確類型的 IoT 中樞傳訊

使用裝置到雲端訊息可傳送裝置應用程式所傳來的時間序列遙測和警示，使用雲端到裝置訊息則可用來傳送單向通知給您的裝置應用程式。

* 請參閱[裝置到雲端通訊指導方針](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-d2c-guidance)，以便在裝置到雲端訊息、回報的屬性或檔案上傳之間進行選擇。
* 請參閱[雲端到裝置通訊指導方針](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-c2d-guidance)，以便在雲端到裝置訊息、預期屬性或直接方法之間進行選擇。

## <a name="next-steps"></a>後續步驟

* 了解 IoT 中樞[訊息路由](iot-hub-devguide-messages-d2c.md)。
* 了解 IoT 中樞[雲端到裝置傳訊](iot-hub-devguide-messages-c2d.md)。