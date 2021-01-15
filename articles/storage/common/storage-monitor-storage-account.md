---
title: 如何監視 Azure 入口網站中的 Azure 儲存體帳戶 | Microsoft Docs
description: 瞭解如何使用 Azure 入口網站和 Azure 儲存體分析來監視 Azure 中的儲存體帳戶。
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 01/09/2020
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring
ms.openlocfilehash: e5495b466bf9b16319b788ec32c7b3a03100f505
ms.sourcegitcommit: d59abc5bfad604909a107d05c5dc1b9a193214a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98218137"
---
# <a name="monitor-a-storage-account-in-the-azure-portal"></a>在 Azure 入口網站中監視儲存體帳戶

[Azure 儲存體分析](storage-analytics.md)會提供所有儲存體服務的計量以及 Blob、佇列和資料表的記錄。 您可以使用 [Azure 入口網站](https://portal.azure.com)設定要為帳戶記錄哪些計量和記錄，並設定可利用視覺方式呈現計量資料的圖表。 

建議您檢閱[適用於儲存體的 Azure 監視器](../../azure-monitor/insights/storage-insights-overview.md) (預覽)。 其是 Azure 監視器的一項功能，藉由統一檢視 Azure 儲存體服務效能、容量和可用性，提供 Azure 儲存體帳戶的全面監視。 您不需要啟用或設定任何項目，而且可以立即從預先定義的互動式圖表和其他包含的視覺效果中檢視這些計量。

> [!NOTE]
> 在 Azure 入口網站中查看監視資料會衍生相關成本。 如需詳細資訊，請參閱[儲存體分析](storage-analytics.md)。
>
> Azure 檔案服務目前支援儲存體分析計量，但還不支援記錄。
>
> Premium 效能區塊 Blob 儲存體帳戶不支援儲存體分析計量，但支援記錄。 您可以透過 REST API 或用戶端程式庫以程式設計的方式啟用記錄。 如果您想要使用進階效能 Blob 儲存體帳戶來檢視計量，請考慮使用 [Azure 監視器中 Azure 儲存體計量](../blobs/monitor-blob-storage.md)。
>
> 如需使用儲存體分析和其他工具來識別、診斷及疑難排解 Azure 儲存體相關問題的深入指南，請參閱 [監視、診斷及疑難排解 Microsoft Azure 儲存體](storage-monitoring-diagnosing-troubleshooting.md)。
>

<a id="modify-retention-policy"></a>

## <a name="configure-monitoring-for-a-storage-account"></a>設定儲存體帳戶的監視

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [儲存體帳戶]，接著選取儲存體帳戶名稱以開啟帳戶儀表板。
1. 在功能表刀鋒視窗的 [監視] 區段中選取 [診斷]。

    ![在監視 (傳統) 區段下，反白顯示診斷設定 (傳統) 選項的螢幕擷取畫面。](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. 針對您想要監視的每個 [服務] 選取計量資料的 [類型]，並選取資料的 [保留原則]。 您也可以將 [狀態] 設定為 [關閉] 來停用監視。

    ![MonitoringOptions](./media/storage-monitor-storage-account/storage-enable-metrics-01.png)

   若要設定資料保留原則，請移動 [保留期 (天)] 滑桿，或輸入要保留資料的天數，範圍從 1 到 365 天。 新儲存體帳戶的預設值是 7 天。 如果不想設定保留原則，則請輸入零。 如果沒有保留原則，您可以決定是否刪除監視資料。

   > [!WARNING]
   > 若您以手動方式刪除計量資料，將需要付費。 過時的分析資料 (存在時間超過保留原則的資料) 則會由系統刪除，不必付費。 建議您根據要將帳戶的儲存體分析資料保留多久來設定保留原則。 如需詳細資訊，請參閱[儲存體計量的計量](storage-analytics-metrics.md#billing-on-storage-metrics)。
   >

1. 監視組態完成時，選取 [儲存]。

[儲存體帳戶] 刀鋒視窗以及個別服務的刀鋒視窗 (Blob、佇列、資料表和檔案) 會以圖表顯示一組預設的計量。 為服務啟用計量後，資料最慢可能需要一小時才會出現在其圖表中。 您可以在任何計量圖表上選取 [編輯]，以設定哪些計量要顯示在圖表中。

您可以將 [狀態] 設定為 [關閉] 來停用計量收集和記錄。

> [!NOTE]
> Azure 儲存體使用[表格儲存體](storage-introduction.md#table-storage)來儲存儲存體帳戶的計量，並且會將計量以資料表形式儲存在帳戶中。 如需詳細資訊，請參閱。 [度量的儲存方式](storage-analytics-metrics.md#how-metrics-are-stored)。
>

## <a name="customize-metrics-charts"></a>自訂計量圖表

使用下列程序可選擇要在計量圖表中檢視哪些儲存體計量。

1. 一開始先在 Azure 入口網站中顯示儲存體計量圖表。 您可以在 [儲存體帳戶] 刀鋒視窗上和個別服務 (Blob、佇列、資料表及檔案) 的 [計量] 刀鋒視窗中找到圖表。

   在此範例中，請使用下列會出現在 [儲存體帳戶] 刀鋒視窗上的圖表：

   ![Azure 入口網站中的圖表選擇](./media/storage-monitor-storage-account/stg-customize-chart-00.png)

1. 按一下圖表內的任一處，以編輯圖表。

1. 接著，針對要在圖表中顯示的計量選取 [時間範圍]，以及要顯示計量的 [服務] (Blob、佇列、資料表、檔案)。 在此，我們選擇要顯示 Blob 服務過去一週的計量︰

   ![[編輯圖表] 刀鋒視窗中的時間範圍和服務選擇](./media/storage-monitor-storage-account/storage-edit-metric-time-range.png)

1. 選取要在圖表中顯示的個別 [計量]，然後按一下 [確定]。

   ![[編輯圖表] 刀鋒視窗中的個別計量選擇](./media/storage-monitor-storage-account/storage-edit-metric-selections.png)

您的圖表設定不會影響對儲存體帳戶中的監視資料所做的收集、彙總或儲存。

### <a name="metrics-availability-in-charts"></a>計量在圖表中的可用性

可用計量清單會根據您在下拉式功能表中所選擇的服務以及您正在編輯之圖表的單位類型而有所變更。 例如，只有在編輯以百分比顯示單位的圖表時，才能選取百分比計量，例如 PercentNetworkError 和 PercentThrottlingError︰

![Azure 入口網站中的要求錯誤百分比圖表](./media/storage-monitor-storage-account/stg-customize-chart-04.png)

### <a name="metrics-resolution"></a>計量解析

您在 [診斷] 中選取的計量會決定帳戶可用計量的解析︰

* **彙總** 監視會提供入口流量/出口流量、可用性、延遲和成功百分比等計量。 這些計量是從 Blob、資料表、檔案和佇列服務彙總而來。
* **依 API** 會提供更精細的解析，除了服務層級的彙總外，還能呈現個別儲存體作業可用的計量。

## <a name="configure-metrics-alerts"></a>設定計量警示

您可以建立警示，以在儲存體資源計量達到閾值時通知您。

1. 若要開啟 [警示規則] 刀鋒視窗，請向下捲動至 [功能表] 刀鋒視窗的 [監視] 區段，然後選取 [警示 (傳統)]。
2. 選取 [新增計量警示 (傳統)]，以開啟 [新增警示規則] 刀鋒視窗
3. 為新的警示規則輸入 [名稱] 和 [描述]。
4. 選取要為其新增警示的 [計量]、警示 [條件] 和 [閾值]。 閾值的單位類型會隨您所選擇的計量而變更。 例如，「計數」是 ContainerCount 的單位類型，PercentNetworkError 計量的單位則是百分比。
5. 選取 [期間]。 在期間內達到或超過閾值的計量會觸發警示。
6. (選擇性) 設定 [電子郵件] 和 [Webhook] 通知。 如需 Webhook 的詳細資訊，請參閱[針對 Azure 度量警示設定 Webhook](../../azure-monitor/platform/alerts-webhooks.md)。 如果未設定電子郵件或 Webhook 通知，則警示只會出現在 Azure 入口網站。

![Azure 入口網站中的 [新增警示規則] 刀鋒視窗](./media/storage-monitor-storage-account/add-alert-rule.png)

## <a name="add-metrics-charts-to-the-portal-dashboard"></a>在入口網站儀表板中新增計量圖表

您可以在入口網站儀表板中新增任何儲存體帳戶的 Azure 儲存體計量圖表。

1. 在 [Azure 入口網站](https://portal.azure.com)中檢視儀表板時按一下 [編輯儀表板]。
1. 在 [圖格庫] 中，選取 [圖格尋找依據] > [類型]。
1. 選取 [類型] > [儲存體帳戶]。
1. 在 [資源] 中，選取您想要在儀表板中新增其計量的儲存體帳戶。
1. 選取 [類別] > [監視]。
1. 將想要顯示之計量的圖表圖格拖放到儀表板。 針對想要顯示在儀表板上的所有計量重複此操作。 下圖醒目提示「Blob - 要求總數」圖表來做為範例，但所有圖表都可放置在儀表板上。

   ![Azure 入口網站中的圖格庫](./media/storage-monitor-storage-account/storage-customize-dashboard.png)
1. 新增完圖表時，請選取儀表板頂端附近的 [自訂完成]。

在儀表板中新增圖表後，您可以進一步自訂這些圖表，如「自訂計量圖表」所述。

## <a name="configure-logging"></a>設定記錄

您可以指示 Azure 儲存體，讓它將 Blob、資料表和佇列服務之讀取、寫入及刪除要求的診斷記錄儲存起來。 您已設定的資料保留原則同樣適用於這些記錄。

> [!NOTE]
> Azure 檔案服務目前支援儲存體分析計量，但還不支援記錄。
>

1. 在 [Azure 入口網站](https://portal.azure.com)中，選取 [儲存體帳戶]，然後選取儲存體帳戶名稱以開啟 [儲存體帳戶] 刀鋒視窗。
1. 在功能表刀鋒視窗的 [監視 (傳統)] 區段中選取 [診斷設定 (傳統)]。

    ![Azure 入口網站中 [監視] 底下的 [診斷] 功能表項目。](./media/storage-monitor-storage-account/storage-enable-metrics-00.png)

1. 確定 [狀態] 已設為 [開啟]，然後選取要為其啟用記錄的 [服務]。

    ![在 Azure 入口網站中設定記錄。](./media/storage-monitor-storage-account/enable-diagnostics.png)
1. 按一下 [檔案] 。

診斷記錄會儲存在儲存體帳戶中名為 *$logs* 的 Blob 容器內。 您可以使用儲存體 explorer （例如 [Microsoft Azure 儲存體總管](https://storageexplorer.com)）來查看記錄資料，或使用儲存體用戶端程式庫或 PowerShell 以程式設計方式查看記錄資料。

如需存取 $logs 容器的詳細資訊，請參閱[儲存體分析記錄](storage-analytics-logging.md)。

## <a name="next-steps"></a>後續步驟

* 尋找更多關於儲存體分析之[計量、記錄和帳單](storage-analytics.md)的詳細資料。