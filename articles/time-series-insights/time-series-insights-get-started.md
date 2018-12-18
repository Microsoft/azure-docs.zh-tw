---
title: 建立 Azure Time Series Insights 環境 | Microsoft Docs
description: 本文說明如何使用 Azure 入口網站建立新的時間序列深入解析環境。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/15/2017
ms.openlocfilehash: 909d8a921e46f0f935bb8a059714b1c0170ddee4
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627656"
---
# <a name="create-a-new-time-series-insights-environment-in-the-azure-portal"></a>在 Azure 入口網站中建立新的 Time Series Insights 環境
本文說明如何使用 Azure 入口網站建立新的時間序列深入解析環境。

時間序列深入解析可讓您開始視覺化及查詢流入 Azure IoT 中樞與事件中樞的資料 (以分鐘為單位)，讓您查詢大量的時間序列資料 (以秒為單位)。  它是針對物聯網 (IoT) 規模所設計的，而且可以處理數 TB 的資料。

## <a name="steps-to-create-the-environment"></a>用以建立環境的步驟
請遵循下列步驟來建立環境：

1.  登入 [Azure 入口網站](https://portal.azure.com)。

2.  選取 [+ 新增] 按鈕。

3.  選取 [物聯網] 類別，然後選取 [時間序列深入解析]。

   ![建立 Time Series Insights 環境](media/time-series-insights-get-started/1-new-tsi.png)

4.  在 [時間序列深入解析] 分頁上，選取 [建立]。

5. 填入必要參數。 下表說明每個參數：
   
   ![建立 Time Series Insights 資源群組](media/time-series-insights-get-started/2-create-tsi.png)
   
   設定|建議的值|說明
   ---|---|---
   環境名稱 | 唯一的名稱 | 此名稱代表[時間序列總管](https://insights.timeseries.azure.com)中的環境
   訂用帳戶 | 您的訂用帳戶 | 如果您有多個訂用帳戶，最好是選擇包含事件來源的訂用帳戶。 Time Series Insights 可以自動偵測相同訂用帳戶中現有的 Azure IoT 中樞與事件中樞資源。
   資源群組 | 建立新的或使用現有的 | 資源群組是一起使用之 Azure 資源的集合。 您可以選擇現有的資源群組，例如包含事件中樞或 IoT 中樞的資源群組。 或者，如果此資源與其他資源不相關，您可以製作新的群組。
   位置 | 最接近您的事件來源 | 最好是選擇包含事件來源資料的相同資料中心位置，以避免跨區域和跨地區頻寬成本，以及在將資料移出區域時增加延遲。
   定價層 | S1 | 選擇所需的輸送量。 如需最低成本和入門容量，選取 S1。
   Capacity | 1 | 容量是套用至輸入速率、儲存體容量以及與選取之 SKU 相關聯成本的乘數。  您可以在環境建立後變更其容量。 如需最低成本，選取 1 作為容量。 
  
6. 請勾選 [釘選到儀表板] 以便在未來最輕鬆地存取時間序列環境。

   ![建立 Time Series Insights [釘選到儀表板]](media/time-series-insights-get-started/3-pin-create.png)

7. 選取 [建立] 以開始佈建程序。 這需要幾分鐘的時間。

8. 若要監視部署程序，選取 [通知] 符號 (鈴鐺圖示)。

   ![觀看通知](media/time-series-insights-get-started/4-notifications.png)

當部署成功時，您可以選取 [移至資源] 以設定其他屬性、以資料存取原則設定安全性、新增事件來源和其他動作。

## <a name="next-steps"></a>後續步驟
* [定義資料存取原則](time-series-insights-data-access.md)來保護環境。
* [新增事件中樞事件來源](time-series-insights-how-to-add-an-event-source-eventhub.md)到您的 Azure 時間序列深入解析環境。 
* [傳送事件](time-series-insights-send-events.md)到事件來源。
* 在[時間序列深入解析總管](https://insights.timeseries.azure.com)中檢視您的環境。
