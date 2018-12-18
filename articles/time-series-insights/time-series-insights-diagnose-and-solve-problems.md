---
title: 在 Azure 時間序列深入解析中診斷與解決問題 | Microsoft Docs
description: 本文說明如何在 Azure 時間序列深入解析環境中診斷、疑難排解與解決可能會遇到的常見問題。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 04/09/2018
ms.openlocfilehash: 399c7b000360a73a9bab06b046be21c9d93a1c70
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46367115"
---
# <a name="diagnose-and-solve-problems-in-your-time-series-insights-environment"></a>在 Time Series Insights 環境中診斷與解決問題

本文說明您可能會在時間序列深入解析環境中看到的部分問題。 其提供可能的原因和解決方案以便於解析。

## <a name="video"></a>視訊： 

### <a name="in-this-video-we-cover-common-time-series-insights-customer-challenges-and-mitigationsbr"></a>在本影片中，我們將說明常見的 Time Series Insights 客戶挑戰和緩和措施。</br>

> [!VIDEO https://www.youtube.com/embed/7U0SwxAVSKw]

## <a name="problem-1-no-data-is-shown"></a>問題 1：未顯示資料
您無法在 [Azure 時間序列深入解析總管](https://insights.timeseries.azure.com) 中看到資料有幾個常見原因：

### <a name="possible-cause-a-event-source-data-is-not-in-json-format"></a>可能原因 A：事件來源資料格式不是 JSON
Azure 時間序列深入解析現在只支援 JSON 資料。 如需 JSON 範例，請參閱[支援的 JSON 樣貌](time-series-insights-send-events.md#supported-json-shapes)。

### <a name="possible-cause-b-event-source-key-is-missing-a-required-permission"></a>可能原因 B：事件來源索引鍵遺漏必要的權限
* 針對 IoT 中樞，您必須提供具有「服務連線」權限的金鑰。

   ![IoT 中樞服務連線權限](media/diagnose-and-solve-problems/iothub-serviceconnect-permissions.png)

   如上圖所示，iothubowner 與 service 原則都會運作，因為兩者皆有「服務連接」權限。

* 針對事件中樞，您必須提供具有「接聽」權限的索引鍵。

   ![事件中樞接聽權限](media/diagnose-and-solve-problems/eventhub-listen-permissions.png)

   如上圖所示，read 與 manage 原則皆可運作，因為兩者皆有「接聽」權限。

### <a name="possible-cause-c-the-consumer-group-provided-is-not-exclusive-to-time-series-insights"></a>可能原因 C：提供的取用者群組並非時間序列深入解析專用
在註冊 IoT 中樞或事件中樞期間，您要指定應用於讀取資料的取用者群組。 **不可**共用此取用者群組。 如果共用取用者群組，基礎事件中樞會自動且隨機地與其中一個讀取器中斷連線。 提供唯一的取用者群組讓時間序列深入解析讀取。

## <a name="problem-2-some-data-is-shown-but-some-is-missing"></a>問題 2：顯示部分資料，但遺漏部分資料
當您可以看到部分資料，但是資料延遲時，有幾個要考慮的可能性：

### <a name="possible-cause-a-your-environment-is-getting-throttled"></a>可能原因 A：您的環境正在進行節流
於建立具有資料的事件來源之後佈建環境時，常會發生這個問題。  Azure IoT 中樞和事件中樞會將資料儲存長達七天。  TSI 將一律從事件來源內最舊的事件 (FIFO) 開始。  因此，如果您連線到 S1 (單一單位 TSI 環境) 時，事件來源有 5 百萬個事件，TSI 每天大約會讀取 1 百萬個事件。  乍看之下，TSI 好像發生五天的延遲。  實際的情況是環境正在進行節流。  如果您的事件來源中有舊的事件，可以進行兩種方式之一：

- 變更事件來源的保留期限制，以協助您將不想要在 TSI 中顯示的舊事件移除
- 佈建較大的環境大小 (以單位數來說)，以增加舊事件的輸送量。  使用上述範例時，如果您將同一個 S1 環境增加至一天五個單位，環境現在應該可以趕在一天之內讀取完成。  如果穩定狀態事件生產環境為每天 1 百萬個以下的事件，則您可以在其趕上之後，將事件的容量降回一個單位。  

節流限制會根據環境 SKU 類型和容量來強制執行。 環境中所有的事件來源皆共用此容量。 如果IoT 中樞或事件中樞的事件來源正在推送超過強制限制的資料，您就會看到節流和延遲情形。

下圖顯示有 SKU 的 S1 且容量為 3 的 Time Series Insights 環境。 該環境可以每日輸入 3 百萬個事件。

![環境 SKU 目前容量](media/diagnose-and-solve-problems/environment-sku-current-capacity.png)

例如，假設此環境正在從事件中樞擷取訊息。 觀察下圖所示的輸入率：

![事件中樞的範例輸入率](media/diagnose-and-solve-problems/eventhub-ingress-rate.png)

如圖所示，每日輸入速率為 ~67,000 則訊息。 此速率可轉譯為大約每分鐘 46 則訊息。 如果每個事件中樞訊息被壓平合併為單一 Time Series Insights 事件，則此環境看不見節流。 如果每個事件中樞訊息被壓平合併為 100 個 Time Series Insights 事件，則每分鐘應內嵌 4,600 個事件。 容量為 3 的 S1 SKU 環境每分鐘只能輸入 2,100 個事件 (每天 1 百萬個事件 = 每 3 單位每分鐘 700 個事件 = 每分鐘 2,100 個事件)。 因此您會看見延遲情形是因為節流。 

若要概略了解壓平合併邏輯的運作方式，請參閱[支援的 JSON 樣貌](time-series-insights-send-events.md#supported-json-shapes)。

### <a name="recommended-resolution-steps-for-excessive-throttling"></a>針對過度節流的建議解決步驟
若要修正延遲情形，請增加您環境的 SKU 容量。 如需詳細資訊，請參閱[如何調整您的 Time Series Insights 環境規模](time-series-insights-how-to-scale-your-environment.md)。

### <a name="possible-cause-b-initial-ingestion-of-historical-data-is-causing-slow-ingress"></a>可能原因 B：歷程記錄資料的初始擷取造成輸入緩慢
如果您正與現有事件來源連線，則您的 IoT 中樞或事件中樞裡可能已有資料。 環境會從事件來源訊息保留期間開始時開始提取資料。

此行為是預設行為且無法覆寫。 您可以進行節流，並且可能需要一段時間才能趕上正在輸入的歷程記錄資料。

#### <a name="recommended-resolution-steps-of-large-initial-ingestion"></a>大型初始擷取的建議解決步驟
若要修正延遲，請遵循下列步驟：
1. 將 SKU 容量增加到最大允許值 (在此案例中是 10)。 增加容量後，輸入程序會開始加快速度來趕上。 您可以從[時間序列深入解析總管](https://insights.timeseries.azure.com)的可用性圖表中看到我們趕上的速度有多快。 增加容量需支付費用。
2. 趕上延遲時間後，即可將 SKU 容量降回到正常的輸入速率。

## <a name="problem-3-my-event-sources-timestamp-property-name-setting-doesnt-work"></a>問題 3：我的事件來源的時間戳記屬性名稱設定沒有作用
請確定名稱和值符合下列規則︰
* 時間戳記屬性名稱有_區分大小寫_。
* 來自事件來源的時間戳記屬性值 (JSON 字串) 格式應是「yyyy-MM-ddTHH:mm:ss.FFFFFFFK」。 字串的範例如 “2008-04-12T12:53Z”。

確保您已擷取「時間戳記屬性名稱」並正常運作的最簡單方式是使用 TSI 總管。  在 TSI 總管中使用圖表的情況下，選取您在提供「時間戳記屬性名稱」之後的一段時間。  以滑鼠右鍵按一下選取項目，然後選擇 [瀏覽事件] 選項。  第一個資料行標頭應該是您的「時間戳記屬性名稱」，且「時間戳記」字組旁邊應該要有 ($ts)，而非：
- (abc)，這表示 TSI 將資料值讀取為字串
- 「行事曆圖示」，這表示 TSI 將資料值讀取為「日期時間」
- #，這表示 TSI 將資料值讀取為整數


## <a name="next-steps"></a>後續步驟
- 如需其他協助，請在 [MSDN 論壇](https://social.msdn.microsoft.com/Forums/home?forum=AzureTimeSeriesInsights)或 [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-timeseries-insights) 上啟動交談。 
- 您也可以使用 [Azure 支援](https://azure.microsoft.com/support/options/)以取得協助的支援選項。
