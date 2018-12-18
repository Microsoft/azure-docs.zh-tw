---
title: Azure 事件方格事件處理常式
description: 描述支援的 Azure 事件方格事件處理常式
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: conceptual
ms.date: 08/21/2018
ms.author: tomfitz
ms.openlocfilehash: 12cc5a918044e3bc74e34f2e05adc9773dee5421
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498652"
---
# <a name="event-handlers-in-azure-event-grid"></a>Azure 事件方格中的事件處理常式

事件處理常式是傳送事件的位置。 處理常式會採取一些進一步的動作來處理事件。 許多 Azure 服務都已自動設定為可處理事件。 您也可以使用任何 WebHook 來處理事件。 WebHook 不需要裝載於 Azure 也能處理處理事件。 事件方格僅支援 HTTPS WebHook 端點。

本文會提供每個事件處理常式的內容連結。

## <a name="azure-automation"></a>Azure 自動化

使用 Azure 自動化，透過自動化的 Runbook 來處理事件。

|標題  |說明  |
|---------|---------|
|[教學課程：Azure 自動化與事件方格和 Microsoft Teams](ensure-tags-exists-on-new-virtual-machines.md) |建立會傳送事件的虛擬機器。 事件會觸發標記虛擬機器的自動化 Runbook，以及觸發傳送給 Microsoft Teams 通道的訊息。 |

## <a name="azure-functions"></a>Azure Functions

使用 Azure Functions 對事件進行無伺服器回應。

使用 Azure Functions 作為處理常式時，請使用 Event Grid 觸發程序而不是泛型 HTTP 觸發程序。 Event Grid 會自動驗證 Event Grid 函式的觸發程序。 若要使用 HTTP 觸發程序，您必須實作[驗證回應](security-authentication.md#webhook-event-delivery)。

|標題  |說明  |
|---------|---------|
| [Azure Functions 的事件方格觸發程序](../azure-functions/functions-bindings-event-grid.md) | 在 Functions 中使用事件方格觸發程序的概觀。 |
| [教學課程：使用事件方格自動調整已上傳映像的大小](resize-images-on-storage-blob-upload-event.md) | 使用者透過 Web 應用程式將映像上傳至儲存體帳戶。 建立儲存體 Blob 時，事件方格會傳送一個事件給函式應用程式，讓其調整上傳映像的大小。 |
| [教學課程：將巨量資料串流處理至資料倉儲](event-grid-event-hubs-integration.md) | 當事件中樞建立「擷取」檔案時，事件方格會傳送一個事件至函式應用程式。 應用程式會對「擷取」檔案進行擷取，並將資料遷移到資料倉儲。 |
| [教學課程：Azure 服務匯流排與 Azure 事件方格的整合範例](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | 事件方格會從服務匯流排主題傳送訊息至函式應用程式和邏輯應用程式。 |

## <a name="event-hubs"></a>事件中樞

解決方案收到事件的速度比處理事件的速度更快時，請使用事件中樞。 應用程式會按照自己的排程，透過事件中樞處理事件。 您可以調整事件流程來處理傳入事件。

事件中樞可以做為事件來源或事件處理常式。 下列文章會示範如何使用事件中樞做為處理常式。

|標題  |說明  |
|---------|---------|
| [快速入門：使用 Azure CLI 和事件方格將自訂事件路由至 Azure 事件中樞](custom-event-to-eventhub.md) | 將自訂事件傳送至事件中樞交給應用程式處理。 |
| [Resource Manager 範本：自訂主題和事件中樞端點](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-event-hubs-handler)| 建立自訂主題訂用帳戶的 Resource Manager 範本。 它會將事件傳送到 Azure 事件中樞。 |

如需事件中樞做為來源的範例，請參閱[事件中樞來源](event-sources.md#event-hubs)。

## <a name="hybrid-connections"></a>混合式連線

使用 Azure 轉送的混合式連線將事件傳送至應用程式，這些應用程式都位於企業網路中，而且沒有可公開存取的端點。

|標題  |說明  |
|---------|---------|
| [教學課程：將事件傳送至混合式連線](custom-event-to-hybrid-connection.md) | 將自訂事件傳送至現有的混合式連線，以透過接聽應用程式進行處理。 |

## <a name="logic-apps"></a>Logic Apps

使用 Logic Apps 來自動化回應事件的商務程序。

|標題  |說明  |
|---------|---------|
| [教學課程：使用 Azure 事件方格和 Logic Apps 監視虛擬機器變更](monitor-virtual-machine-changes-event-grid-logic-app.md) | 邏輯應用程式會監視虛擬機器的變更，並傳送有關這些變更的電子郵件。 |
| [教學課程：使用 Logic Apps 來傳送 Azure IoT 中樞事件的相關電子郵件通知](publish-iot-hub-events-to-logic-apps.md) | 每當有裝置新增至您的 IoT 中樞時，邏輯應用程式就會傳送電子郵件通知。 |
| [教學課程：Azure 服務匯流排與 Azure 事件方格的整合範例](../service-bus-messaging/service-bus-to-event-grid-integration-example.md?toc=%2fazure%2fevent-grid%2ftoc.json) | 事件方格會從服務匯流排主題傳送訊息至函式應用程式和邏輯應用程式。 |

## <a name="queue-storage"></a>佇列儲存體

使用佇列儲存體來接收需要提取的事件。 若執行中的流程太過冗長導致回應時間過久，您可以使用佇列儲存體。 藉由傳送事件至佇列儲存體，應用程式即可按照自己的排程提取並處理事件。

|標題  |說明  |
|---------|---------|
| [快速入門：使用 Azure CLI 和事件方格將自訂事件路由至 Azure 佇列儲存體](custom-event-to-queue-storage.md) | 描述如何將自訂事件傳送至佇列儲存體。 |

## <a name="webhooks"></a>Webhook

針對回應事件的自訂端點使用 Webhook。

|標題  |說明  |
|---------|---------|
| 快速入門：使用 [Azure CLI](custom-event-quickstart.md)、[PowerShell](custom-event-quickstart-powershell.md) 及[入口網站](custom-event-quickstart-portal.md)建立和路由傳送自訂事件。 | 示範如何將自訂事件傳送至 WebHook。 |
| 快速入門：使用 [Azure CLI](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json)、[PowerShell](../storage/blobs/storage-blob-event-quickstart-powershell.md?toc=%2fazure%2fevent-grid%2ftoc.json) 及[入口網站](blob-event-quickstart-portal.md)將 Blob 儲存體事件路由至自訂的 Web 端點。 | 示範如何將 Blob 儲存體事件傳送至 WebHook。 |
| [快速入門：傳送容器登錄事件](../container-registry/container-registry-event-grid-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) | 示範如何使用 Azure CLI 傳送 Container Registry 事件。 |
| [概觀：接收 HTTP 端點的事件](receive-events.md) | 說明如何驗證 HTTP 端點以從事件訂閱接收事件，然後接收和還原序列化事件。 |

## <a name="next-steps"></a>後續步驟

* 如需 Event Grid 的簡介，請參閱[關於 Event Grid](overview.md)。
* 若要快速地開始使用 Event Grid，請參閱[使用 Azure Event Grid 建立和路由傳送自訂事件](custom-event-quickstart.md)。
