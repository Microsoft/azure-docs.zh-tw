---
title: 了解 Azure IoT 中樞內建端點 | Microsoft Docs
description: 開發人員指南：說明如何使用內建的事件中樞相容端點來讀取裝置到雲端訊息。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/18/2018
ms.author: dobett
ms.openlocfilehash: 02624b4f3b0fceb1816f4f43b1f435356f8d5235
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984036"
---
# <a name="read-device-to-cloud-messages-from-the-built-in-endpoint"></a>從內建端點讀取裝置到雲端訊息

根據預設，系統會將訊息路由至內建的服務面向端點 (**messages/events**)，此端點與[事件中樞][lnk-event-hubs]相容。 此端點目前只公開使用連接埠 5671 上的 [AMQP][lnk-amqp] 通訊協定。 IoT 中樞會公開下列屬性，讓您控制與事件中樞相容的內建訊息端點 **messages/events**。

| 屬性            | 說明 |
| ------------------- | ----------- |
| **分割計數** | 在建立時設定此屬性，以定義裝置到雲端事件擷取的[分割區][lnk-event-hub-partitions]數目。 |
| **保留時間**  | 此屬性可指定 IoT 中樞保留訊息的天數。 預設值是一天，但它可以增加到七天。 |

IoT 中樞也可讓您管理內建裝置到雲端接收端點上的取用者群組。

如果您使用[訊息路由](iot-hub-devguide-messages-d2c.md)且[後援路由](iot-hub-devguide-messages-d2c.md#fallback-route)已啟用，就會將未符合任何路由上之查詢的所有訊息寫入到內建端點。 如果您停用此後援路由，則會捨棄不符合任何查詢的訊息。

您可以使用 [IoT 中樞資源提供者 REST API][lnk-resource-provider-apis] 以程式設計方式修改保留時間，或使用 [Azure 入口網站][lnk-management-portal]來修改。

IoT 中樞會公開您後端服務的 **messages/events** 內建端點，以讀取您的中樞收到的裝置到雲端訊息。 此端點與事件中樞相容，可讓您使用事件中樞服務支援的任何機制讀取訊息。

## <a name="read-from-the-built-in-endpoint"></a>從內建端點讀取

使用[適用於 .NET 的 Azure 服務匯流排 SDK][lnk-servicebus-sdk] 或[事件中樞 - 事件處理器主機][lnk-eventprocessorhost]時，您可以使用任何 IoT 中樞連接字串搭配正確的權限。 然後使用 **messages/events** 做為事件中樞名稱

當您使用無法感知「IoT 中樞」的 SDK (或產品整合) 時，必須擷取「事件中樞」相容端點和「事件中樞」相容名稱︰

1. 登入 [Azure 入口網站][lnk-management-portal]，然後瀏覽至 IoT 中樞。
1. 按一下 [內建端點]。
1. [事件] 區段包含下列值：[事件中樞相容端點]、[事件中樞相容名稱]、[分割區]、[保留時間] 與 [取用者群組]。

    ![裝置到雲端設定][img-eventhubcompatible]

IoT 中樞 SDK 需要 IoT 中樞端點名稱，也就是 [端點] 下顯示的 **messages/events**。

如果您正在使用的 SDK 需要**主機名稱**或**命名空間**值，請從 [事件中樞相容端點] 中移除配置。 例如，如果您的事件中樞相容端點為 **sb://iothub-ns-myiothub-1234.servicebus.windows.net/**，**主機名稱**會是 **iothub-ns-myiothub-1234.servicebus.windows.net**。 **命名空間**將是 **iothub-ns-myiothub-1234**。

然後，您可以使用具有 **ServiceConnect** 權限的任何共用存取原則，連接至指定的事件中樞。

如果您需要使用先前資訊來建置事件中樞連接字串，請使用下列模式：

`Endpoint={Event Hub-compatible endpoint};SharedAccessKeyName={iot hub policy name};SharedAccessKey={iot hub policy key}`

您可以搭配 IoT 中樞公開之事件中樞相容端點使用的 SDK 和整合項目包含下列清單中的項目：

* [Java 事件中樞用戶端](https://github.com/Azure/azure-event-hubs-java)。
* [Apache Storm Spout](../hdinsight/storm/apache-storm-develop-csharp-event-hub-topology.md)。 您可以檢視 GitHub 上的 [Spout 原始檔](https://github.com/apache/storm/tree/master/external/storm-eventhubs) 。
* [Apache Spark 整合](../hdinsight/spark/apache-spark-eventhub-streaming.md)。

## <a name="next-steps"></a>後續步驟

* 如需 IoT 中樞端點的詳細資訊，請參閱 [IoT 中樞端點][lnk-endpoints]。
* [快速入門][lnk-get-started]示範如何從模擬裝置傳送裝置到雲端訊息和從內建端點讀取訊息。 如需詳細資訊，請參閱[使用路由處理 IoT 中樞的裝置到雲端訊息][lnk-d2c-tutorial]教學課程。
* 如果您想要將裝置到雲端訊息路由至自訂端點，請參閱[針對裝置到雲端訊息使用訊息路由和自訂端點][lnk-custom]。

[img-eventhubcompatible]: ./media/iot-hub-devguide-messages-read-builtin/eventhubcompatible.png

[lnk-custom]: iot-hub-devguide-messages-read-custom.md
[lnk-get-started]: quickstart-send-telemetry-node.md
[lnk-endpoints]: iot-hub-devguide-endpoints.md
[lnk-resource-provider-apis]: https://docs.microsoft.com/rest/api/iothub/iothubresource
[lnk-event-hubs]: http://azure.microsoft.com/documentation/services/event-hubs/
[lnk-management-portal]: https://portal.azure.com
[lnk-d2c-tutorial]: tutorial-routing.md
[lnk-event-hub-partitions]: ../event-hubs/event-hubs-features.md#partitions
[lnk-servicebus-sdk]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-eventprocessorhost]: https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-receive-eph
[lnk-amqp]: https://www.amqp.org/
