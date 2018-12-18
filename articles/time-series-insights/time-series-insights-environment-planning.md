---
title: 規劃調整您的 Azure 時間序列深入解析環境規模 | Microsoft Docs
description: 此文章說明在規劃 Azure 時間序列深入解析環境 (包括儲存容量、資料保留、輸入容量、監視和業務災害復原 (BCDR)) 時，如何遵循最佳做法。
services: time-series-insights
ms.service: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/15/2017
ms.openlocfilehash: fa178efadf001b70501b132ede67686ae5c06363
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47422553"
---
# <a name="plan-your-azure-time-series-insights-environment"></a>規劃 Azure 時間序列深入解析環境

此文章以預期的輸入速率和資料保留需求作為基礎，說明如何規劃 Azure 時間序列深入解析環境。

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-cover-time-series-insights-data-retention-and-how-to-plan-for-itbr"></a>在此影片中，我們將說明時間序列深入解析資料保留以及如何進行規劃。</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

## <a name="best-practices"></a>最佳做法

若要開始使用時間序列深入解析，您最好知道以分鐘數計您需要推送多少資料，以及需要儲存資料多久。  

如需這兩個時間序列深入解析 SKU 的容量和保留之詳細資訊，請參閱[時間序列深入解析定價](https://azure.microsoft.com/pricing/details/time-series-insights/)。

請考慮下列的屬性，以最佳方式規劃能長期獲致成功的環境： 
- 儲存體容量
- 資料保留期間
- 輸入容量 
- 塑造您的事件
- 確定您已備妥參考資料

## <a name="understand-storage-capacity"></a>了解儲存體容量
依預設，時間序列深入解析會保留資料 以您已佈建的儲存體數量 (單位乘以每個單位的存放裝置數量) 和輸入。

## <a name="understand-data-retention"></a>了解資料保留
您可以設定時間序列深入解析環境的**資料保留時間**設定，從而啟用最多 400 天的保留期。  時間序列深入解析有兩個模式，其中一個進行最佳化以確保您的環境具有最新的資料 (依預設)，另一個最佳化以確保符合保留期限制，其中如果叫用環境的整體儲存體容量，輸入就會暫停。  您可以調整保留期，並在 Azure 入口網站中環境組態頁面的兩個模式之間切換。

您可以在時間序列深入解析環境中設定最大值為 400 天的資料保留期。

## <a name="configure-data-retention"></a>設定資料保留期

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取您的時間序列深入解析環境。

2. 在 [時間序列深入解析環境] **頁面**的 [設定]標題下，選取 [設定]。 

3. 在 [資料保留時間 (天數)] 方塊中，輸入介於 1 到 400 的值。

   ![設定保留期](media/environment-mitigate-latency/configure-retention.png)

## <a name="understand-ingress-capacity"></a>了解輸入容量

在規劃時所需著重的另一個區域是輸入容量，也就是每分鐘配置的衍生。 

就節流的觀點而言，會將封包大小為 32 KB 的輸入資料封包視為 32 個事件，每個大小為 1 KB。 允許的事件大小上限為 32 KB；大於 32 KB 的資料封包會被截斷。

下表摘要說明每個 SKU 的輸入容量：

|SKU  |每個單位每個月的事件計數  |每個單位每個月的事件大小  |每個單位每分鐘的事件計數  | 每個單位每分鐘的大小   |
|---------|---------|---------|---------|---------|
|S1     |   3,000 萬     |  30 GB     |  720    |  720 KB   |
|S2     |   3 億    |   300 GB   | 7,200   | 7,200 KB  |

您可以在單一環境中將 S1 或 S2 SKU 的容量增加至 10 個單位。 您無法從 S1 環境移轉至 S2，或從 S2 環境移轉至 S1。

針對輸入容量，您應該先以每月為基礎判斷所需要的總輸入。 接下來，判斷您每分鐘的需求，因為這是節流和延遲扮演的角色。

如果您資料輸入的高峰持續時間低於 24 小時，時間序列深入解析可能會以上列速率 2 倍的輸入速率來「趕上」。 

例如，如果您有單一 S1 SKU 和以每分鐘 720 個事件速率的輸入資料，且高峰以 1440 個或更少事件的速率低於 1 小時，對您的環境就沒有明顯的延遲。 不過，如果一小時以上超過每分鐘 1440 個事件，您可能會遇到視覺化的資料發生延遲，而且可供您在環境中查詢。 

您可能不會事先知道需要推送多少資料。 在此情況下，您可以在 Azure 入口網站中找到 [Azure IoT 中樞](https://docs.microsoft.com/azure/iot-hub/iot-hub-metrics)和 [Azure 事件中樞](https://blogs.msdn.microsoft.com/cloud_solution_architect/2016/05/25/using-the-azure-rest-apis-to-retrieve-event-hub-metrics/)的資料遙測。 此遙測可協助您判斷如何佈建環境。 在 Azure 入口網站中使用個別事件來源的 [計量] 頁面以檢視其遙測。 如果您了解事件來源的計量，就可以更有效地規劃並佈建時間序列深入解析環境。

### <a name="calculate-ingress-requirements"></a>計算輸入需求

- 確認您的輸入容量高於您每分鐘平均速率，且您的環境夠大，可在 1 小時內處理您預期的輸入，相當於您容量的 2 倍。

- 如果發生持續超過 1 小時的輸入高峰，請使用高峰率作為平均值，並使用處理高峰率的容量來佈建環境。
 
### <a name="mitigate-throttling-and-latency"></a>減少節流和延遲

如需關於如何避免節流和延遲的資訊，請參閱[減少節流和延遲](time-series-insights-environment-mitigate-latency.md)。

## <a name="shaping-your-events"></a>塑造您的事件
請務必確定您將事件傳送至 TSI 的方式可支援要佈建的環境大小 (反過來說，您可以將環境的大小對應至 TSI 可讀取的事件數目和每個事件的大小)。  同樣地，務必考量您在查詢資料時想要進行配量和篩選的屬性。  基於這一點，我們建議您檢閱[傳送事件文件](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-send-events)中的＜JSON 形式＞一節。  該內容位於頁面底部。  

## <a name="ensuring-you-have-reference-data-in-place"></a>確定您已備妥參考資料
參考資料集是許多項目的集合，由這些項目來擴展您事件來源中的事件。 時間序列深入解析輸入引擎會將事件來源的每個事件和參考資料集中的對應資料聯結在一起。 然後此增強的事件可用於查詢。 此聯結是以參考資料集中定義的主索引鍵資料行為基礎。

請注意，參考資料不會回溯加入。 這表示一旦設定和上傳參考資料集之後，只會比對目前和未來的輸入資料並加入參考資料集中。  如果您打算將大量歷程記錄資料傳送至 TSI，但沒有先在 TSI 中上傳或建立參考資料，則您可能要重新執行您的工作 (提醒您，這可不有趣)。  

若要深入了解如何在 TSI 中建立、上傳及管理參考資料，請移至[參考資料集文件](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-add-reference-data-set)。

## <a name="business-disaster-recovery"></a>業務災害復原
作為 Azure 服務，時間序列深入解析可提供高可用性 (HA)，在 Azure 區域層級使用備援，且解決方案不需任何額外的工作。 Microsoft Azure 平台也包含各種功能，以協助您建置具有災害復原 (DR) 功能或跨區域可用性的解決方案。 若想要為裝置或使用者提供全域、跨區域的高可用性，可善用這類 Azure DR 功能。 指導方針[Azure 業務持續性技術指導方針](../resiliency/resiliency-technical-guidance.md) 一文描述業務持續性和 DR 的 Azure 內建功能。 [Azure 應用程式的災害復原與高可用性](https://docs.microsoft.com/azure/architecture/resiliency/index) 一文針對 Azure 應用程式的策略提供架構指導方針以達到 HA 和 DR。

Azure 時間序列深入解析並未內建業務災害復原 (BCDR)。
根據預設，Azure IoT 中樞與事件中樞都內建復原功能。

若要深入了解 IoT 中樞的 BCDR 原則，請移至[這裡](https://docs.microsoft.com/azure/iot-hub/iot-hub-ha-dr)。  

若要深入了解事件中樞的 BCDR 原則，請移至[這裡](https://docs.microsoft.com/azure/event-hubs/event-hubs-geo-dr)。

不過，需要 BCDR 的客戶還是可以使用下列方法實作復原策略。
在備份 Azure 區域中建立第二個時間序列深入解析環境，並從主要事件來源將事件傳送至此第二環境，利用第二個專用取用者群組和該事件來源的 BCDR 指導方針。  

1.  在第二個區域中建立環境。  若要深入了解如何建立時間序列深入解析環境，請參閱[這裡](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-get-started)的內容。
2.  為事件來源建立第二個專用取用者群組，並將該事件來源連線至新環境。  請務必指定第二個專用取用者群組。  您可以遵循 [IoT 中樞文件](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-iothub)或[事件中樞文件](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-data-access)來深入了解這方面的資訊。
3.  如果主要區域會在災害事件發生期間停止運作，請將作業切換到備份時間序列深入解析環境來執行。  

請**務必注意**，在容錯移轉案例期間，TSI 可以再次開始處理訊息之前可能會有延遲：這可能導致訊息處理量突增。 如需詳細資訊，請參閱[此文件](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-environment-mitigate-latency)

## <a name="next-steps"></a>後續步驟
- [如何新增事件中樞的事件來源](time-series-insights-how-to-add-an-event-source-eventhub.md)
- [如何新增 IoT 中樞的事件來源](time-series-insights-how-to-add-an-event-source-iothub.md)
