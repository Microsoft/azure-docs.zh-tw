---
title: 在 Azure 監視器中建立具有動態閾值的警示
description: 建立警示，內含以機器學習為基礎的動態閾值
author: antonfrMSFT
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 04/27/2018
ms.author: mbullwin
ms.reviewer: antonfr
ms.component: alerts
ms.openlocfilehash: 01f924e0b3a2976a3f537cb5acac842eeeaccb4b
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263308"
---
# <a name="alerts-with-dynamic-thresholds-in-azure-monitor-limited-public-preview"></a>在 Azure 監視器中具有動態閾值的警示 (有限的公開預覽)

具有動態閾值的警示是對於 Azure 監視器中 Azure 計量警示的增強功能，它會利用進階的機器學習 (ML) 功能，學習計量的歷史行為，以自動計算基準線並且使用它們作為警示閾值。

使用動態閾值的優點如下：

- 省去設定預先定義固定界限相關的麻煩，因為監視器會自動學習歷史計量效能並且套用 ML 演算法，以判斷警示閾值。
- 它們可識別季節性的行為，並且只對與預期季節性的行為有偏差的項目發出警示。 如果您的服務定期在週末閒置而在每週一達到尖峰，則具有動態閾值的計量警示不會觸發。 目前支援：每小時、每日及每週的季節性。
- 持續學習計量效能，並且根據計量變更進行調整。

以動態閾值為基礎的警示可用於此[文章](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-near-real-time-metric-alerts#what-resources-can-i-create-near-real-time-metric-alerts-for)所列的所有以 Azure 監視器為基礎的計量來源。

## <a name="sign-up-to-access-the-preview"></a>登入以存取預覽

若要試用這項功能，請[註冊預覽](http://aka.ms/DynamicThresholdMetricAlerts)。 一如既往，我們很希望聽聽您的意見反應，來信請寄：[azurealertsfeedback@microsoft.com](mailto:azurealertsfeedback@microsoft.com)

## <a name="how-to-configure-alerts-with-dynamic-thresholds"></a>如何設定具有動態閾值的警示

具有動態閾值的警示可以透過 Azure 監視器中的警示進行設定

![警示預覽](./media/monitoring-alerts-dynamic-thresholds/0001.png)

## <a name="creating-an-alert-rule-with-dynamic-thresholds"></a>建立具有動態閾值的警示規則

1. 從 [監視器] 底下的 [警示] 窗格，選取 [新增警示規則] 按鈕，以在 Azure 中建立新的警示。

   ![新增警示規則](./media/monitoring-alerts-dynamic-thresholds/002.png)

2. [建立規則] 區段會以三個部分顯示：「定義警示條件」、「定義警示詳細資料」和「定義動作群組」。 首先從「定義警示條件」區段開始，使用「選取目標」連結，藉由選取資源來指定目標。 一旦選擇了適當的資源，請按一下 [完成] 按鈕。

   ![選取目標](./media/monitoring-alerts-dynamic-thresholds/0003.png)

3. 接下來使用 [新增準則] 按鈕，以檢視可用於資源的訊號選項清單，然後從訊號清單選擇適當的 [計量] 選項。 (例如 CPU 百分比。)

   ![新增準則](./media/monitoring-alerts-dynamic-thresholds/004.png)

4. 在 [設定訊號邏輯] 畫面上的 [警示邏輯] 區段中，您可以選擇將條件切換為「動態」類型，這樣會自動產生動態閾值 (紅線) 以及計量 (藍線)。

   ![動態](./media/monitoring-alerts-dynamic-thresholds/005.png)

5. 圖表中顯示的閾值是根據歷史記錄資料的最後 7 天進行計算。一旦建立警示，「動態」閾值會取得可用的其他歷史資料，並且根據新資料持續學習，讓閾值更準確。

6. 其他警示邏輯設定：
   - 條件 - 您可以選擇讓警示在下列三個條件其中一個發生時觸發：
       - 大於閾值上限或低於閾值下限 (預設值)
       - 大於閾值上限
       - 低於閾值下限。
   - 時間彙總：平均值 (預設)、加總、最小值、最大值。
   - 警示敏感度：
       - 高 - 較多警示，因為警示會在最小的偏差時觸發。
       - 中 - 比高的敏感度低，比高敏感度的警示少 (預設值)
       - 低 - 最不敏感的閾值。

    ![警示邏輯設定](./media/monitoring-alerts-dynamic-thresholds/00007.png)

7. 評估依據：
    -  持續時間為何，該警示應藉由從 [期間] 選擇，來查詢指定條件。

    ![評估依據](./media/monitoring-alerts-dynamic-thresholds/007.png)

   > [!NOTE]
   > 支援的期間值：5 分鐘、10 分鐘、30 分鐘及 1 小時。

   若要減少暫時性尖峰所產生的警示雜訊，我們建議使用「觸發警示的違規次數」設定。 這項功能可讓您只有在連續違反閾值 X 次，或者在最後一個 Z 期間違規 Y 次，才會收到警示。 例如︰

    若要在問題持續 15 分鐘、在 5 分鐘的指定時間內連續 3 次時觸發警示，請使用下列設定：

   ![評估依據](./media/monitoring-alerts-dynamic-thresholds/0008.png)

    若要以 5 分鐘為期間，在最後 30 分鐘的 15 分鐘內違反動態閾值時觸發警示，請使用下列設定：

   ![評估依據](./media/monitoring-alerts-dynamic-thresholds/0009.png)

8. 目前使用者可以使用具有動態閾值準則的警示作為單一準則。

   ![建立規則](./media/monitoring-alerts-dynamic-thresholds/010.png)

## <a name="q--a"></a>問答集

- 問：如果計量隨著時間緩慢變更，這樣會觸發具有動態閾值的警示嗎？

- 答：可能不會。 動態閾值適合用於偵測明顯的偏差，而不是緩慢發展的問題。

- 問：是否可以透過 API 設定動態閾值？

- 答：我們正在處理中。
