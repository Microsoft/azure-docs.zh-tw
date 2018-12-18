---
title: 如何將 IoT 中樞的事件來源新增至 Azure Time Series Insights | Microsoft Docs
description: 本文說明如何將與 IoT 中樞連線的事件來源新增至時間序列深入解析環境
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/21/2017
ms.openlocfilehash: b6beecbf64cee925f62ac4c82919926fcb79940a
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627367"
---
# <a name="how-to-add-an-iot-hub-event-source-to-time-series-insights-environment"></a>如何將 IoT 中樞事件來源新增至時間序列深入解析環境
本文說明如何使用 Azure 入口網站，將從 IoT 中樞讀取資料的事件來源新增至時間序列深入解析環境。

## <a name="prerequisites"></a>必要條件
- 建立時間序列深入解析環境。 如需詳細資訊，請參閱[建立 Azure 時間序列深入解析環境](time-series-insights-get-started.md) 
- 建立 IoT 中樞。 如需有關 IoT 中樞的詳細資訊，請參閱[使用 Azure 入口網站建立 IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)
- IoT 中樞必須要有傳入的作用中訊息事件。
- 在 IoT 中樞建立專用取用者群組，讓時間序列深入解析環境從中取用。 每個時間序列深入解析事件必須有自身專屬的取用者群組，且不可與任何其他取用者共用。 如果多個讀取器從同一個取用者群組取用事件，則所有讀取器都可能會看到錯誤。 如需詳細資料，請參閱 [IoT 中樞開發人員指南](../iot-hub/iot-hub-devguide.md)。

### <a name="add-a-consumer-group-to-your-iot-hub"></a>將取用者群組新增至 IoT 中樞
應用程式會使用取用者群組從 Azure IoT 中樞提取資料。 只為時間序列深入解析環境提供專用取用者群組，以便可靠地從 IoT 中樞讀取資料。

若要將新的取用者群組新增至 IoT 中樞，請遵循下列步驟：
1. 在 Azure 入口網站中，找出 IoT 中樞並開啟。

2. 在 [傳訊] 標題下，選取 [端點]。 

   ![新增取用者群組](media/time-series-insights-how-to-add-an-event-source-iothub/5-add-consumer-group.png)

3. 選取 [事件]端點，[屬性] 頁面隨即開啟。

4. 在 [取用者群組] 標題下，提供取用者群組新的唯一名稱。 建立新的事件來源時，在時間序列深入解析環境中使用這個相同的名稱。

5. 選取 [儲存] 以儲存新的取用者群組。

## <a name="add-a-new-event-source"></a>新增事件來源
1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找出您的現有時間序列深入解析環境。 按一下 Azure 入口網站左側功能表中的 [所有資源]。 選取 Time Series Insights 環境。

3. 在 [環境拓撲] 標題下，按一下 [事件來源]。
   ![事件來源 + 新增](media/time-series-insights-how-to-add-an-event-source-iothub/1-event-sources.png)

4. 按一下 [+ 新增]。

5. 提供對於這個時間序列深入解析環境而言唯一的**事件來源名稱**，例如 **event-stream**。

   ![填入表單中的前三個參數。](media/time-series-insights-how-to-add-an-event-source-iothub/2-import-option.png)

6. 將 [來源] 選取為 [IoT 中樞]。

7. 選取適當的**匯入選項**。 
   - 當您在其中一個訂用帳戶上已經有現有 IoT 中樞時，選擇 [使用可用訂閱中的 IoT 中樞]。 這是最簡單的方法。
   - 當 IoT 中樞對於您的訂用帳戶是外部，或者您想要選擇進階選項時，選擇 [手動提供 IoT 中樞設定]。 

8. 如果您已選取 [使用可用訂閱中的 IoT 中樞] 選項，下表說明每個必要屬性：

   ![訂用帳戶和事件中樞詳細資料](media/time-series-insights-how-to-add-an-event-source-iothub/3-new-event-source.png)

   | 屬性 | 說明 |
   | --- | --- |
   | 訂用帳戶識別碼 | 選取建立此 IoT 中樞的訂用帳戶。
   | IoT 中樞名稱 | 選取 IoT 中樞的名稱。
   | IoT 中樞原則名稱 | 選取 [IoT 中樞設定] 索引標籤下的共用存取原則。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 事件來源的共用存取原則必須有**服務連線**權限。
   | IoT 中樞原則金鑰 | 索引鍵已預先填入。
   | IoT 中樞取用者群組 | 從 IoT 中樞讀取事件的取用者群組。 強烈建議您使用專屬於您事件來源的取用者群組。
   | 事件序列化格式 | JSON 是目前唯一可用的序列化。 事件訊息必須是這種格式，否則無法讀取任何資料。 |
   | 時間戳記屬性名稱 | 若要判斷此值，您需要了解傳送至 IoT 中樞的訊息資料訊息格式。 這個值是您想要使用作為事件時間戳記之訊息資料中特定事件屬性的**名稱**。 此值區分大小寫。 保留空白時，會使用事件來源內的**事件加入佇列時間**作為事件時間戳記。 |

9. 如果您已選取 [手動提供 IoT 中樞設定] 選項，下表說明每個必要屬性：

   | 屬性 | 說明 |
   | --- | --- |
   | 訂用帳戶識別碼 | 建立此 IoT 中樞的訂用帳戶。
   | 資源群組 | 建立此 IoT 中樞所在的資源群組名稱。
   | IoT 中樞名稱 | IoT 中樞的名稱。 當您建立 IoT 中樞時，您也會指定特定名稱。
   | IoT 中樞原則名稱 | 共用存取原則，可在 [IoT 中樞設定] 索引標籤下加以建立。每一個共用存取原則都會有名稱、權限 (由您設定) 和存取金鑰。 事件來源的共用存取原則必須有**服務連線**權限。
   | IoT 中樞原則金鑰 | 用來驗證服務匯流排命名空間之存取權的共用存取金鑰。 在此輸入主要金鑰或次要金鑰。
   | IoT 中樞取用者群組 | 從 IoT 中樞讀取事件的取用者群組。 強烈建議您使用專屬於您事件來源的取用者群組。
   | 事件序列化格式 | JSON 是目前唯一可用的序列化。 事件訊息必須是這種格式，否則無法讀取任何資料。 |
   | 時間戳記屬性名稱 | 若要判斷此值，您需要了解傳送至 IoT 中樞的訊息資料訊息格式。 這個值是您想要使用作為事件時間戳記之訊息資料中特定事件屬性的**名稱**。 此值區分大小寫。 保留空白時，會使用事件來源內的**事件加入佇列時間**作為事件時間戳記。 |

10. 新增您新增至 IoT 中樞的專用 TSI 取用者群組名稱。

11. 選取 [建立] 以新增事件來源。

   ![按一下 [建立專案]。](media/time-series-insights-how-to-add-an-event-source-iothub/4-create-button.png)

   建立事件來源之後，Time Series Insights 會自動開始將資料串流處理至您的環境。

## <a name="next-steps"></a>後續步驟
- [定義資料存取原則](time-series-insights-data-access.md)來保護資料。
- [將事件傳送](time-series-insights-send-events.md)到事件來源。
- 在[時間序列深入解析總管](https://insights.timeseries.azure.com)中存取您的環境。
