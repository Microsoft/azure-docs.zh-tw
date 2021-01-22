---
title: 新增事件中樞事件來源-Azure 時間序列深入解析 |Microsoft Docs
description: 瞭解如何將 Azure 事件中樞事件來源新增至您的 Azure 時間序列深入解析環境。
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 01/21/2021
ms.custom: seodec18
ms.openlocfilehash: f4b5d4915cd6520edd7a45af85a836c3360eee32
ms.sourcegitcommit: 77afc94755db65a3ec107640069067172f55da67
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98696324"
---
# <a name="add-an-event-hub-event-source-to-your-azure-time-series-insights-environment"></a>將事件中樞事件來源新增至您的 Azure 時間序列深入解析環境

此文章說明如何使用 Azure 入口網站，將從 Azure 事件中樞讀取資料的事件來源新增至您的 Azure 時間序列深入解析環境。

> [!NOTE]
> 本文所述的步驟適用于 Azure 時間序列深入解析 Gen 1 和 Azure 時間序列深入解析 Gen 2 環境。

## <a name="prerequisites"></a>必要條件

- 建立 [Azure 時間序列深入解析環境](./tutorials-set-up-tsi-environment.md)中所述的 Azure 時間序列深入解析環境。
- 建立事件中樞。 請參閱 [使用 Azure 入口網站來建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)。
- 事件中樞必須要有傳入的作用中訊息事件。 瞭解如何 [使用 .NET Framework 將事件傳送至 Azure 事件中樞](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md)。
- 在事件中樞內建立 Azure 時間序列深入解析環境可以取用的專用取用者群組。 每個 Azure 時間序列深入解析事件來源都必須有自己專用的取用者群組，而該群組不會與任何其他取用者共用。 如果有多個讀取器取用來自同一個取用者群組的事件，則所有讀取器可能會出現失敗。 另外也限制每一個事件中樞最多有 20 個取用者群組。 如需詳細資訊，請參閱 [事件中樞程式設計指南](../event-hubs/event-hubs-programming-guide.md)。

### <a name="add-a-consumer-group-to-your-event-hub"></a>將取用者群組新增至事件中樞

應用程式會使用取用者群組從 Azure 事件中樞提取資料。 若要可靠地從事件中樞讀取資料，請提供僅供此 Azure 時間序列深入解析環境使用的專用取用者群組。

將新的取用者群組新增至事件中樞：

1. 在 [Azure 入口網站](https://portal.azure.com)中，從事件中樞命名空間的 [ **總覽** ] 窗格中，找出並開啟您的事件中樞實例。 選取 [ **實體] > 事件中樞** ，或在 [ **名稱**] 底下尋找您的實例。

    [![開啟您的事件中樞命名空間](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-connect-event-hub-namespace.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-connect-event-hub-namespace.png#lightbox)

1. 在您的事件中樞實例中，選取 [ **實體 > 取用者群組**]。 然後，選取 [ **+ 取用者群組** ] 以新增取用者群組。

   [![事件中樞-新增取用者群組](media/time-series-insights-how-to-add-an-event-source-eventhub/add-event-hub-consumer-group.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/add-event-hub-consumer-group.png#lightbox)

   否則，請選取現有的取用者群組，並跳至下一節。

1. 在 [取用者群組] 頁面上，在 [名稱] 中輸入新的唯一值。  當您在 Azure 時間序列深入解析環境中建立新的事件來源時，請使用相同的名稱。

1. 選取 [建立]  。

## <a name="add-a-new-event-source"></a>新增事件來源

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 找出您現有的 Azure 時間序列深入解析環境。 在左側功能表中，選取 [ **所有資源**]，然後選取您的 Azure 時間序列深入解析環境。

1. 選取 [ **事件來源**]，然後選取 [ **新增**]。

   [![在 [事件來源] 下，選取 [加入] 按鈕](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-add-an-event-source.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-add-an-event-source.png#lightbox)

1. 輸入此 Azure 時間序列深入解析環境唯一的 [ **事件來源名稱** ] 值，例如 `Contoso-TSI-Gen 1-Event-Hub-ES` 。

1. 在 [來源] 中，選取 [事件中樞]。

1. 為 [匯入選項] 選取適當的值：

   - 若您的其中一個訂用帳戶中有現有的事件中樞，請選取 [來自可用訂閱的使用事件中樞]。 此選項是最簡單的方法。

     [![選取事件來源匯入選項](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-event-hub-select-import-option.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-event-hub-select-import-option.png#lightbox)

   - 下表說明 [來自可用訂閱的使用事件中樞] 選項的必要屬性：

       [![訂用帳戶和事件中樞詳細資料](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-configure-create-confirm.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-configure-create-confirm.png#lightbox)

       | 屬性 | 描述 |
       | --- | --- |
       | 訂用帳戶 | 所需的事件中樞實例和命名空間所屬的訂用帳戶。 |
       | 事件中樞命名空間 | 所需事件中樞實例所屬的事件中樞命名空間。 |
       | 事件中樞名稱 | 所需事件中樞實例的名稱。 |
       | 事件中樞原則值 | 選取所需的共用存取原則。 您可以在 [事件中樞 **設定** ] 索引標籤上建立共用存取原則。每個共用存取原則都有名稱、您設定的許可權，以及存取金鑰。 事件來源的共用存取原則必須有 **讀取** 權限。 |
       | 事件中樞原則金鑰 | 從選取的事件中樞原則值預先填入。 |

   - 若事件中樞對於您的訂用帳戶而言是外部，或您想要選取進階選項，請選取 [手動提供事件中樞設定]。

       下表說明 [手動提供事件中樞設定] 選項的必要屬性：

       | 屬性 | 描述 |
       | --- | --- |
       | 訂用帳戶識別碼 | 所需的事件中樞實例和命名空間所屬的訂用帳戶。 |
       | 資源群組 | 所需事件中樞實例和命名空間所屬的資源群組。 |
       | 事件中樞命名空間 | 所需事件中樞實例所屬的事件中樞命名空間。 |
       | 事件中樞名稱 | 所需事件中樞實例的名稱。 |
       | 事件中樞原則值 | 選取所需的共用存取原則。 您可以在 [事件中樞 **設定** ] 索引標籤上建立共用存取原則。每個共用存取原則都有名稱、您設定的許可權，以及存取金鑰。 事件來源的共用存取原則必須有 **讀取** 權限。 |
       | 事件中樞原則金鑰 | 用來驗證服務匯流排命名空間之存取權的共用存取金鑰。 在這裡輸入主要金鑰或次要金鑰。 |

   - 這兩個選項都會共用下列設定選項：

       | 屬性 | 描述 |
       | --- | --- |
       | 事件中樞取用者群組 | 從事件中樞讀取事件的取用者群組。 強烈建議您使用專屬於您事件來源的取用者群組。 |
       | 事件序列化格式 | 目前，JSON 是目前唯一可用的序列化格式。 事件訊息必須是這種格式，否則無法讀取資料。 |
       | 時間戳記屬性名稱 | 若要判斷此值，您需要了解傳送至事件中樞之訊息資料的訊息格式。 這個值是您想要使用作為事件時間戳記之訊息資料中特定事件屬性的 **名稱**。 此值區分大小寫。 若保留空白，會使用事件來源中的 **事件加入佇列時間** 作為事件時間戳記。 |

1. 新增您新增至事件中樞的專用 Azure 時間序列深入解析取用者組名。

1. 選取 [建立]  。

   建立事件來源之後，Azure 時間序列深入解析會自動開始將資料串流至您的環境。

## <a name="next-steps"></a>後續步驟

- [定義資料存取原則](./concepts-access-policies.md)來保護資料。

- [將事件傳送](time-series-insights-send-events.md) 至事件來源。

- 在 [Azure 時間序列深入解析 Explorer](https://insights.timeseries.azure.com)中存取您的環境。
