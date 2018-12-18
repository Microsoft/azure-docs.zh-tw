---
title: 使用 Azure 儲存體封存 Azure 計量和記錄資料
description: 將 Azure 內產生的記錄和計量資料封存至儲存體帳戶。
author: johnkemnetz
services: azure-monitor
ms.service: azure-monitor
ms.topic: tutorial
ms.date: 09/25/2017
ms.author: johnkem
ms.custom: mvc
ms.component: metrics
ms.openlocfilehash: be6f3efd74d013e9ddb02c44031dd6a402f00871
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409485"
---
# <a name="archive-azure-metric-and-log-data-using-azure-storage"></a>使用 Azure 儲存體封存 Azure 計量和記錄資料

Azure 環境的許多層會產生記錄和計量資料，可封存至 Azure 儲存體帳戶。 當一段時間的監視資料超過保留期之後，建議您將此資料的記錄保留在低成本、不可搜尋的存放區。 

- Azure 監視器平台的計量會保留 93 天。 
- 資源診斷記錄只會在路由至 Log Analytics 時顯示，其中記錄有最少 30 天的可設定保留期。 
- 活動記錄項目會保留90 天。  

本教學課程逐步引導您設定 Azure 環境，以將資料封存至儲存體帳戶。

> [!div class="checklist"]
> * 建立儲存體帳戶以放置監視資料
> * 將訂用帳戶記錄路由傳送至儲存體帳戶
> * 將資源資料路由傳送至儲存體帳戶
> * 將虛擬機器 (客體 OS) 資料路由傳送至儲存體帳戶
> * 檢視其中的監視資料
> * 清除資源

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-storage-account"></a>建立儲存體帳戶

首先，您必須設定用來封存監視資料的儲存體帳戶。 若要這樣做，[請遵循這裡的步驟](../storage/common/storage-create-storage-account.md)。

## <a name="route-subscription-logs-to-the-storage-account"></a>將訂用帳戶記錄路由傳送至儲存體帳戶

您現在已準備好開始設定 Azure 環境，以將監視資料路由傳送至儲存體帳戶。 首先，我們會設定要路由傳送至儲存體帳戶的訂用帳戶層級資料 (包含在 Azure 活動記錄中)。 [**Azure 活動記錄**](monitoring-overview-activity-logs.md)提供 Azure 中的訂用帳戶層級事件的記錄。 您可以在 Azure 入口網站瀏覽，以查明「誰」建立、更新或刪除「什麼」資源及「何時」那樣做。

1. 按一下左側瀏覽清單上的 [監視] 按鈕，然後按一下 [活動記錄]。

   ![活動記錄區段](media/monitor-tutorial-archive-monitoring-data/activity-log-home.png)

2. 在顯示的 [活動記錄] 區段中，按一下 [匯出] 按鈕。

3. 在出現的 [匯出活動記錄] 區段中，勾選 [匯出至儲存體帳戶] 方塊，然後按一下 [選取儲存體帳戶]。

   ![活動記錄匯出](media/monitor-tutorial-archive-monitoring-data/activity-log-export.png)

4. 在出現的區段中，使用 [儲存體帳戶] 下拉式清單，選取您在先前的**建立儲存體帳戶**步驟中建立的儲存體帳戶名稱，然後按一下 [確定]。

   ![選取儲存體帳戶](media/monitor-tutorial-archive-monitoring-data/activity-log-storage.png)

5. 將 [保留期 (天數)] 滑桿設為 30。 此滑桿可設定監視資料在儲存體帳戶中的保留天數。 Azure 監視器會自動刪除比指定天數更舊的資料。 保留天數為 0 會無限期地儲存資料。

6. 按一下 [儲存] 並關閉此區段。

來自訂用帳戶的監視資料現在開始流入儲存體帳戶。

## <a name="route-resource-data-to-the-storage-account"></a>將資源資料路由傳送至儲存體帳戶

現在，我們要設定**資源診斷設定**，以設定要路由傳送至儲存體帳戶的資源層級資料 (資源計量和診斷記錄)。

1. 按一下左側瀏覽清單上的 [監視] 按鈕，然後按一下 [診斷設定]。 在這裡，您會看到訂用帳戶中所有透過 Azure 監視器而產生監視資料的資源清單。 如果您在這份清單中沒有任何資源，您可以先[建立邏輯應用程式](../logic-apps/quickstart-create-first-logic-app-workflow.md)，再繼續進行，這樣就有可進行診斷設定的資源。

2. 按一下清單中的資源，然後按一下 [開啟診斷]。

   ![開啟診斷](media/monitor-tutorial-archive-monitoring-data/diagnostic-settings-turn-on.png)

   如果有已完成的設定，則您看到現有的設定，以及 [新增診斷設定] 按鈕。 按一下此按鈕。

   資源診斷設定可定義應該從特定資源路由傳送「什麼」監視資料，以及監視資料應該傳送至「何處」。

3. 在出現的區段中，為您的設定指定**名稱**，並勾選 [封存至儲存體帳戶] 方塊。

   ![診斷設定區段](media/monitor-tutorial-archive-monitoring-data/diagnostic-settings-home.png)

4. 按一下 [封存至儲存體帳戶] 下的 [設定] 按鈕，並選取您在上一節所建立的儲存體帳戶。 按一下 [確定]。

   ![診斷設定儲存體帳戶](media/monitor-tutorial-archive-monitoring-data/diagnostic-settings-storage.png)

5. 勾選 [記錄] 和 [計量] 下的所有方塊。 根據資源類型而定，您可能只有其中一個選項可用。 這些核取方塊可控制要將該資源類型的哪幾種記錄和計量資料傳送至您選取的目的地，在此案例中是儲存體帳戶。

   ![診斷設定類別](media/monitor-tutorial-archive-monitoring-data/diagnostic-settings-categories.png)

6. 將 [保留期 (天數)] 滑桿設為 30。 此滑桿可設定監視資料在儲存體帳戶中的保留天數。 Azure 監視器會自動刪除比指定天數更舊的資料。 保留天數為 0 會無限期地儲存資料。

7. 按一下 [檔案] 。

來自資源的監視資料現在開始流入儲存體帳戶。

> [!NOTE]
> 目前不支援透過診斷設定傳送多維度計量。 跨維度值所彙總的維度計量會匯出為扁平化單一維度計量。
>
> 「例如」：可以在每個佇列層級瀏覽並繪製事件中樞上的「內送郵件」計量。 不過，當您透過診斷設定匯出時，計量將會呈現為事件中樞內所有佇列的所有內送郵件。
>
>

## <a name="route-virtual-machine-guest-os-data-to-the-storage-account"></a>將虛擬機器 (客體 OS) 資料路由傳送至儲存體帳戶

1. 如果您的訂用帳戶中還沒有虛擬機器，請[建立虛擬機器](../virtual-machines/windows/quick-create-portal.md)。

2. 在入口網站的左側瀏覽清單中，按一下 [虛擬機器]。

3. 在顯示的虛擬機器清單中，按一下您所建立的虛擬機器。

4. 在出現的區段中，按一下左側導覽上的 [診斷設定]。 這個區段可讓您在虛擬機器上設定 Azure 監視器現成的監視擴充，並將 Windows 或 Linux 產生的資料路由傳送至儲存體帳戶。

   ![瀏覽至診斷設定](media/monitor-tutorial-archive-monitoring-data/guest-navigation.png)

5. 在出現的區段中，按一下 [啟用來賓層級監視]。

   ![啟用診斷設定](media/monitor-tutorial-archive-monitoring-data/guest-enable.png)

6. 正確儲存診斷設定之後，[概觀] 索引標籤會顯示要收集的資料清單及其儲存位置。 按一下 [效能計數器] 區段，以檢閱要收集的 Windows 效能計數器集合。

   ![效能計數器設定](media/monitor-tutorial-archive-monitoring-data/guest-perf-counters.png)

7. 按一下 [記錄] 索引標籤，然後勾選應用程式和系統記錄的 [資訊] 層級記錄的核取方塊。

   ![記錄設定](media/monitor-tutorial-archive-monitoring-data/guest-logs.png)

8. 按一下 [代理程式] 索引標籤，然後在 [儲存體帳戶] 下，按一下顯示的儲存體帳戶名稱。

   ![更新儲存體帳戶](media/monitor-tutorial-archive-monitoring-data/guest-storage-home.png)

9. 在出現的區段中，選取您在先前的**建立儲存體帳戶**步驟中建立的儲存體帳戶。

10. 按一下 [檔案] 。

來自虛擬機器的監視資料現在開始流入儲存體帳戶。

## <a name="view-the-monitoring-data-in-the-storage-account"></a>檢視儲存體帳戶中的監視資料

> [!WARNING]
> 2018 年 11 月 1 日起，儲存體帳戶中的記錄資料格式將變更為 JSON 資料行。 [請參閱本文以了解影響的描述，以及如何更新您的工具，來處理新的格式。](./monitor-diagnostic-logs-append-blobs.md) 
>
> 

如果您遵循上述步驟操作，資料已經開始流向儲存體帳戶。

1. 對於某些資料類型，例如活動記錄，需要有一些活動在儲存體帳戶中產生事件。 若要活動記錄中產生活動，請遵循[這些指示](./monitor-quick-audit-notify-action-in-subscription.md)。 您可能需要等候長達五分鐘，事件才會出現在儲存體帳戶中。

2. 在入口網站中，在左側導覽列尋找並瀏覽至 [儲存體帳戶] 區段。

3. 識別您在上一節所建立的儲存體帳戶並按一下。

4. 按一下 [Blob]，再按一下標示為 **insights-operational-logs** 的容器，最後按一下標示為 **name=default** 的容器。 這是包含活動記錄的容器。 監視資料先依資源識別碼 (就是活動記錄的訂用帳戶識別碼) 分組到容器，再依日期和時間分組。 這些 Blob 的完整格式為：

   insights-operational-logs/name=default/resourceId=/SUBSCRIPTIONS/{subscription ID}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json

5. 依資源識別碼、日期和時間按一下容器，以瀏覽至 PT1H.json 檔案。 按一下 PT1H.json 檔案，然後按一下 [下載]。 每個 PT1H.json blob 有一個 JSON blob，包含在 blob URL 指定時數內 (例如 h = 12) 發生的事件。 在目前這一小時，事件一發生就會附加到 PT1H.json 檔案。 分鐘值 (m=00) 一定是 00，因為記錄事件每小時會分組到個別 Blob。

   您現在可以檢視已儲存在儲存體帳戶中的 JSON 事件。 若是資源診斷記錄，Blob 的格式為：

   insights-logs-{log category name}/resourceId=/{resource ID}/y={four-digit numeric year}/m={two-digit numeric month}/d={two-digit numeric day}/h={two-digit 24-hour clock hour}/m=00/PT1H.json

6. 客體 OS 監視資料會儲存在資料表中。 瀏覽回到儲存體帳戶首頁，然後按一下 [資料表]。 有計量、效能計數器和事件記錄的資料表。

您現在已成功設定要封存至儲存體帳戶的監視資料。

## <a name="clean-up-resources"></a>清除資源

1. 從先前的**將訂用帳戶記錄路由傳送至儲存體帳戶**步驟瀏覽回到 [匯出活動記錄] 區段，然後按一下 [重設]。

2. 瀏覽至 [診斷設定] 區段，按一下您在先前的**將資源資料路由傳送至儲存體帳戶**步驟中建立診斷設定的資源，然後找出您建立的設定，按一下 [編輯設定] 按鈕，並按一下 [刪除]。

3. 瀏覽至您在先前的**將虛擬機器 (客體 OS) 資料路由傳送至儲存體帳戶**步驟中設定之虛擬機器上的 [診斷設定] 區段，然後在 [代理程式] 索引標籤下按一下 [移除] \(在 [移除 Azure 診斷代理程式] 區段下方)。

4. 瀏覽至您在先前的**建立儲存體帳戶**步驟中建立的儲存體帳戶，然後按一下 [刪除儲存體帳戶]。 輸入儲存體帳戶的名稱，然後按一下 [刪除]。

5. 如果您在先前的步驟中也建立虛擬機器或邏輯應用程式，請一併刪除。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何從 Azure 環境 (訂用帳戶、資源及客體 OS) 設定要封存至儲存體帳戶的監視資料。


> [!div class="checklist"]
> * 建立儲存體帳戶以放置監視資料
> * 將訂用帳戶記錄路由傳送至儲存體帳戶
> * 將資源資料路由傳送至儲存體帳戶
> * 將虛擬機器 (客體 OS) 資料路由傳送至儲存體帳戶
> * 檢視其中的監視資料
> * 清除資源

為了更妥善利用您的資料和衍生額外的深入解析，也請將資料傳送到 Log Analytics。

> [!div class="nextstepaction"]
> [開始使用 Log Analytics](../log-analytics/log-analytics-get-started.md)
