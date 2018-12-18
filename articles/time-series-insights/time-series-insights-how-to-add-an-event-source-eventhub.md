---
title: 如何將事件中樞的事件來源新增至 Azure Time Series Insights | Microsoft Docs
description: 本文說明如何將與事件中樞連線的事件來源新增至時間序列深入解析環境。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/21/2017
ms.openlocfilehash: ce4bf1ab74e4203f0deb7b2984ffa6a66d5efd4a
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627105"
---
# <a name="how-to-add-an-event-hub-event-source-to-time-series-insights-environment"></a>如何將事件中樞事件來源新增至時間序列深入解析環境

本文說明如何使用 Azure 入口網站，將從事件中樞讀取資料的事件來源新增至時間序列深入解析環境。

## <a name="prerequisites"></a>必要條件
- 建立時間序列深入解析環境。 如需詳細資訊，請參閱[建立 Azure 時間序列深入解析環境](time-series-insights-get-started.md) 
- 建立事件中樞。 如需事件中樞的詳細資料，請參閱[使用 Azure 入口網站建立事件中樞命名空間和事件中樞](../event-hubs/event-hubs-create.md)
- 事件中樞必須要有傳入的作用中訊息事件。 如需詳細資訊，請參閱[使用 .NET Framework 將事件傳送至 Azure 事件中樞](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md)。
- 在事件中樞建立專用取用者群組，讓時間序列深入解析環境從中取用。 每個時間序列深入解析事件必須有自身專屬的取用者群組，且不可與任何其他取用者共用。 如果多個讀取器從同一個取用者群組取用事件，則所有讀取器都可能會看到錯誤。 請注意，每一個事件中樞另外還有 20 個用戶群組的限制。 如需詳細資訊，請參閱 [事件中樞程式設計指南](../event-hubs/event-hubs-programming-guide.md)。

### <a name="add-a-consumer-group-to-your-event-hub"></a>將取用者群組新增至事件中樞
應用程式會使用取用者群組從 Azure 事件中樞提取資料。 只為時間序列深入解析環境提供專用取用者群組，以便可靠地從事件中樞讀取資料。

若要將新的取用者群組新增至事件中樞，請遵循下列步驟：
1. 在 Azure 入口網站中，找出事件中樞並開啟。

2. 在 [實體] 標題底下，選取 [取用者群組]。

   ![事件中樞 - 新增取用者群組](media/time-series-insights-how-to-add-an-event-source-eventhub/5-event-hub-consumer-group.png)

3. 選取 [+ 取用者群組] 以新增取用者群組。 

4. 在 [取用者群組] 頁面上，提供新的唯一 [名稱]。  在時間序列深入解析環境中建立新的事件來源時，使用此相同名稱。

5. 選取 [建立] 以建立新的取用者群組。

## <a name="add-a-new-event-source"></a>新增事件來源
1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找出您的現有時間序列深入解析環境。 按一下 Azure 入口網站左側功能表中的 [所有資源]。 選取 Time Series Insights 環境。

3. 在 [環境拓撲] 標題下，按一下 [事件來源]。

   ![事件來源 + 新增](media/time-series-insights-how-to-add-an-event-source-eventhub/1-event-sources.png)

4. 按一下 [+ 新增]。

5. 提供對於這個時間序列深入解析環境而言唯一的**事件來源名稱**，例如 **event-stream**。

   ![填入表單中的前三個參數。](media/time-series-insights-how-to-add-an-event-source-eventhub/2-import-option.png)

6. 將 [來源] 選取為 [事件中樞]。

7. 選取適當的**匯入選項**。 
   - 當您在其中一個訂用帳戶上已經有現有事件中樞時，選擇 [使用可用訂閱中的事件中樞]。 這是最簡單的方法。
   - 當事件中樞對於您的訂用帳戶是外部，或者您想要選擇進階選項時，選擇 [手動提供事件中樞設定]。 

8. 如果您已選取 [使用可用訂閱中的事件中樞] 選項，下表說明每個必要屬性：

   ![訂用帳戶和事件中樞詳細資料](media/time-series-insights-how-to-add-an-event-source-eventhub/3-new-event-source.png)

   | 屬性 | 說明 |
   | --- | --- |
   | 訂用帳戶識別碼 | 選取建立此事件中樞的訂用帳戶。
   | 服務匯流排命名空間 | 選取包含事件中樞的服務匯流排命名空間。
   | 事件中樞名稱 | 選取事件中樞的名稱。
   | 事件中樞原則名稱 | 選取共用的存取原則，可以在事件中樞的 [設定] 索引標籤上建立。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 事件來源的共用存取原則必須有**讀取**權限。
   | 事件中樞原則金鑰 | 系統可能會預先填入金鑰值。
   | 事件中樞取用者群組 | 要從事件中樞讀取事件的取用者群組。 強烈建議您使用專屬於您事件來源的取用者群組。 |
   | 事件序列化格式 | JSON 是目前唯一可用的序列化。 事件訊息必須是這種格式，否則無法讀取任何資料。 |
   | 時間戳記屬性名稱 | 若要判斷此值，您需要了解傳送至事件中樞的訊息資料訊息格式。 這個值是您想要使用作為事件時間戳記之訊息資料中特定事件屬性的**名稱**。 此值區分大小寫。 保留空白時，會使用事件來源內的**事件加入佇列時間**作為事件時間戳記。 |


9. 如果您已選取 [手動提供事件中樞設定] 選項，下表說明每個必要屬性：

   | 屬性 | 說明 |
   | --- | --- |
   | 訂用帳戶識別碼 | 建立此事件中樞的訂用帳戶。
   | 資源群組 | 建立此事件中樞所在的資源群組。
   | 服務匯流排命名空間 | 服務匯流排命名空間是一個容器，包含一組訊息實體。 建立新的事件中樞時，也會建立服務匯流排命名空間。
   | 事件中樞名稱 | 事件中樞的名稱。 當您建立事件中心時，您也會指定特定名稱。
   | 事件中樞原則名稱 | 共用的存取原則，可以在事件中樞的 [設定] 索引標籤上建立。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 事件來源的共用存取原則必須有**讀取**權限。
   | 事件中樞原則金鑰 | 用來驗證服務匯流排命名空間之存取權的共用存取金鑰。 在此輸入主要金鑰或次要金鑰。
   | 事件中樞取用者群組 | 從事件中樞讀取資料的取用者群組。 強烈建議您使用專屬於您事件來源的取用者群組。
   | 事件序列化格式 | JSON 是目前唯一可用的序列化。 事件訊息必須是這種格式，否則無法讀取任何資料。 |
   | 時間戳記屬性名稱 | 若要判斷此值，您需要了解傳送至事件中樞的訊息資料訊息格式。 這個值是您想要使用作為事件時間戳記之訊息資料中特定事件屬性的**名稱**。 此值區分大小寫。 保留空白時，會使用事件來源內的**事件加入佇列時間**作為事件時間戳記。 |

10. 新增您新增至事件中樞的專用 TSI 取用者群組名稱。

11. 選取 [建立] 以新增事件來源。
   
   ![按一下 [建立專案]。](media/time-series-insights-how-to-add-an-event-source-eventhub/4-create-button.png)

   建立事件來源之後，Time Series Insights 會自動開始將資料串流處理至您的環境。


## <a name="next-steps"></a>後續步驟
- [定義資料存取原則](time-series-insights-data-access.md)來保護資料。
- [將事件傳送](time-series-insights-send-events.md)到事件來源。
- 在[時間序列深入解析總管](https://insights.timeseries.azure.com)中存取您的環境。
