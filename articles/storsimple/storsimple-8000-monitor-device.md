---
title: 監視 StorSimple 8000 系列裝置 | Microsoft Docs
description: 說明如何使用 StorSimple 裝置管理員服務來監視使用量、I/O 效能和容量使用率。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 10/17/2017
ms.author: alkohli
ms.openlocfilehash: 679c1fc8775ad4481bc99c9aea79fe16e9bcac8f
ms.sourcegitcommit: cf4c0ad6a628dfcbf5b841896ab3c78b97d4eafd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/21/2017
ms.locfileid: "23479264"
---
# <a name="use-the-storsimple-device-manager-service-to-monitor-your-storsimple-device"></a>使用 StorSimple 裝置管理員服務監視 StorSimple 裝置

## <a name="overview"></a>概觀
您可以使用 StorSimple 裝置管理員服務來監視您 StorSimple 解決方案中的特定裝置。 您可以根據 I/O 效能、容量使用率、網路輸送量，以及裝置效能計量建立自訂圖表，並且將這些圖表釘選到儀表板。 如需詳細資訊，請移至[自訂入口網站儀表板](../azure-portal/azure-portal-dashboards.md)。

若要檢視特定裝置的監視資訊，請在 Azure 入口網站中，選取 [StorSimple 裝置管理員服務]。 從裝置清單中選取您的裝置，然後移至 [監視]。 然後您會看見所選裝置的 [容量]、[使用量] 和 [效能] 圖表。

## <a name="capacity"></a>Capacity
**容量**追蹤裝置上佈建的空間與剩餘的空間。 然後剩餘的容量會顯示為固定在本機或階層式。

佈建的容量和剩餘的容量會進一步細分為階層式磁碟區和固定在本機的磁碟區。 針對每個磁碟區，會顯示裝置上佈建的容量和剩餘的容量。

![IO 容量](./media/storsimple-8000-monitor-device/device-capacity.png)



## <a name="usage"></a>使用量
**使用量**追蹤由磁碟區、磁碟區容器，或裝置所使用之資料儲存體空間數量的相關計量。 您可以根據您的主要儲存體、雲端儲存體，或裝置儲存空間的容量使用率來建立報告。 容量使用率可以在特定磁碟區、特定磁碟區容器，或所有磁碟區容器上測量。
預設會報告過去 24 小時的使用量。 您可以從下列項目選取來編輯圖表，以變更報告使用量的持續時間：
* 過去 24 小時
* 過去 7 天
* 過去 30 天
* 過去 90 天
* 去年

使用圖表會報告兩個關鍵計量 - 成長和範圍。 範圍是指在所選持續期間 (例如過去 7 天) 報告的使用量的最大值和最小值。

成長則指在所選持續期間從第一天到最後一天使用量的增加。 

成長和範圍也可以透過下列方程式表示：

```
Range = {Usage(minimum), Usage(maximum)}

Growth = Usage(Last day) - Usage(first day)

Growth (%) = [{Usage(last day) - Usage(first day)} X 100]/Usage(first day)
```

使用的主要、雲端及本機儲存體的說明如下：

### <a name="primary-storage-usage"></a>主要儲存體使用量
這些圖表顯示在進行重複資料刪除和壓縮之前寫入 StorSimple 磁碟區的資料量。 您可以檢視磁碟區容器中所有磁碟區或單一磁碟區所使用的主要儲存體。 使用的主要儲存體會進一步細分為使用的主要階層式存放區，以及使用的主要本機固定存放區。

下表顯示建立雲端快照集前後 StorSimple 裝置的主要儲存體。 由於這只是磁碟區資料，因此雲端快照不應變更主要儲存體。 如您所見，由於建立雲端快照集的緣故，圖表顯示使用的主要階層式存放區或使用的主要本機固定存放區並沒有差別。 雲端快照是在下午 11:50 左右開始在該裝置上建立。

![建立雲端快照集後的主要容量使用率](./media/storsimple-8000-monitor-device/device-primary-storage-after-cloudsnapshot.png)

如果您現在在連線到 StorSimple 裝置的主機上執行 IO，視您將資料寫入哪個磁碟區 (階層式或本機固定) 而定，將看到使用的主要階層式存放區或本機固定存放區會增加。 以下是 StorSimple 裝置的主要儲存體使用量圖表。 在此裝置上，StorSimple 主機在下午大約 2:30，在裝置上的階層式磁碟區開始執行寫入。 您可以看到寫入的每秒位元組出現峰值，對應至主機上執行的 IO。

![在階層式磁碟區執行 IO 時的效能](./media/storsimple-8000-monitor-device/device-io-from-initiator.png)

如果您看使用的主要階層式存放區已經增加，但是主要本機固定的使用量還是維持不變，這是因為沒有執行寫入到裝置上的本機固定磁碟區。

![階層式磁碟區上執行 IO 時的主要容量使用率](./media/storsimple-8000-monitor-device/device-primary-storage-io-from-initiator.png)

如果您執行的是 Update 3 或更新版本，就能夠依個別磁碟區、所有磁碟區、所有階層式磁碟區以及所有固定在本機的磁碟區，細分主要儲存體容量使用率，如以下所示。 如果依所有固定在本機的磁碟區來細分，您將能快速地確認已用完多少本機層。

![所有階層式磁碟區的主要容量使用率](./media/storsimple-8000-monitor-device/monitor-usage3.png)

![所有固定在本機之磁碟區的主要容量使用率](./media/storsimple-8000-monitor-device/monitor-usage4.png)

您可以進一步按一下清單中的每個磁碟區，查看對應的使用量。

![所有固定在本機之磁碟區的主要容量使用率](./media/storsimple-8000-monitor-device/device-primary-storage-usage-by-volume.png)

### <a name="cloud-storage-usage"></a>雲端儲存體使用量
這些圖表顯示雲端儲存體使用量。 這項資料會進行重複資料刪除和壓縮。 此數量包含雲端快照集，其可能包含未反映在任何主要磁碟區且針對舊版或必要保留用途而保留的資料。 您可以比較主要儲存體和雲端儲存體的耗用圖表來了解資料減少速率，雖然數字並不精確。

下表顯示建立雲端快照集時，StorSimple 裝置的雲端儲存體使用率。

* 雲端快照集在上午大約 11:50 在該裝置上啟動，您可以看到在雲端快照集之前，還沒有使用的雲端儲存體。 
* 雲端快照集完成之後，雲端儲存體使用率瞬間上升到 0.89 GB。 
* 雲端快照集進行時，從裝置到雲端的 IO 也有一個對應的峰值。

    ![建立雲端快照集之前的雲端儲存體使用率](./media/storsimple-8000-monitor-device/device-cloud-storage-before-cloudsnapshot.png)

    ![建立雲端快照集之後的雲端儲存體使用率](./media/storsimple-8000-monitor-device/device-cloud-storage-after-cloudsnapshot.png)

    ![建立雲端快照集期間，從裝置到雲端的 IO](./media/storsimple-8000-monitor-device/device-io-to-cloud-during-cloudsnapshot.png)


### <a name="local-storage-usage"></a>本機儲存體使用量
這些圖表顯示裝置的總使用率，此使用率因為包含了 SSD 線性層，所以會高於主要儲存體使用量。 這個階層包含也存在於裝置上其他階層中的資料量。 SSD 線性層中的容量是重複使用的，因此當新資料進入時，舊資料會移至 HDD 層 (此時會進行重複資料刪除並壓縮)，再移至雲端。

隨著時間過去，使用的主要儲存體和使用的本機儲存體很可能會一起增加，直到資料開始分層到雲端。 此時，使用的本機儲存體可能開始穩定停滯，但使用的主要儲存體將會因更多的資料寫入而增加。

下表顯示建立雲端快照集時，StorSimple 裝置使用的主要儲存體。 雲端快照集在上午 11:50 開始建立，本機儲存體也在該時間開始減少。 使用的本機儲存體從 1.445 GB 下降到 1.09 GB。 這表示很可能在線性 SSD 層中未壓縮的資料已刪除重複資料、壓縮，並移入 HDD 層。 請注意，若裝置已在 SSD 和 HDD 層中有大量資料，您可能看不出減少的情況。 在此範例中，裝置僅有少量的資料。

![建立雲端快照集之後的本機儲存體使用率](./media/storsimple-8000-monitor-device/device-local-storage-after-cloudsnapshot.png)

## <a name="performance"></a>效能
**效能**追蹤主機伺服器上 iSCSI 啟動器介面與裝置之間或裝置與雲端之間，與讀取和寫入作業數目相關的計量。 這項效能可以針對特定磁碟區、特定磁碟區容器，或所有磁碟區容器來測量。 效能也包括裝置上的 CPU 使用率和不同網路介面的網路輸送量。

### <a name="io-performance-for-initiator-to-device"></a>啟動器到裝置的 I/O 效能
下表顯示生產環境裝置之所有磁碟區中的裝置啟動器 I/O。 繪製的計量是每秒讀取和寫入的位元組。 您也可以將讀取、寫入和未完成的 IO，或將讀取延遲和寫入延遲，繪製成圖表。

![啟動器到裝置的 IO 效能](./media/storsimple-8000-monitor-device/device-io-from-initiator.png)

### <a name="io-performance-for-device-to-cloud"></a>裝置到雲端的 I/O 效能
就相同裝置而言，所繪製的 I/O 作業是針對所有磁碟區容器從裝置到雲端的資料。 在此裝置上，資料僅位於線性層內且並未溢出至雲端。 從裝置到雲端並未發生讀寫作業。 因此，圖表中尖峰的間隔是 5 分鐘，這和裝置與服務之間的活動訊號檢查頻率對應。

針對相同的裝置，磁碟區資料的雲端快照集從上午 11:50 開始建立。 這會導致資料從裝置流向雲端。 在這段期間會提供雲端寫入服務。 IO 圖表顯示建立快照集時，在每秒寫入位元組出現峰值所對應的時間。

![建立雲端快照集期間，從裝置到雲端的 IO](./media/storsimple-8000-monitor-device/device-io-to-cloud-during-cloudsnapshot.png)

### <a name="network-throughput-for-device-network-interfaces"></a>裝置網路介面的網路輸送量
**網路輸送量** 會追蹤與從主機伺服器和裝置上的 iSCSI 啟動器網路介面傳輸之資料量有關，以及與在裝置和雲端之間傳輸之資料量有關的計量。 您可以針對裝置上的每一個 iSCSI 網路介面監視此計量。

下列圖表顯示裝置上 Data 0、1 1 GbE 網路的網路輸送量，此網路同時具備雲端功能 (預設) 和 iSCSI 功能。 此裝置在 6 月 14 日下午大約 9 點，資料分層到雲端 (當時沒有建立任何雲端快照集，表示分層是將資料移動到雲端的機制)，產生傳送到雲端的 IO。 網路輸送量圖表在同一時間有對應的峰值，且大部分的網路流量是輸出到雲端。

![Data 0 的網路輸送量](./media/storsimple-8000-monitor-device/device-network-throughput-data0.png)

如果我們看 Data 1 介面的輸送量圖表 (另一個只啟用 iSCSI 的 1 GbE 網路介面)，在這段期間幾乎沒有網路流量。

![Data 1 的網路輸送量](./media/storsimple-8000-monitor-device/device-network-throughput-data1.png)


## <a name="cpu-utilization-for-device"></a>裝置的 CPU 使用率
**CPU 使用率**追蹤裝置上與使用的 CPU 相關的計量。 下列圖表顯示生產環境中裝置的 CPU 使用率統計資料。

![裝置的 CPU 使用率](./media/storsimple-8000-monitor-device/device-cpu-utilization.png)



## <a name="next-steps"></a>後續步驟
* 了解如何[使用 StorSimple 裝置管理員服務裝置儀表板](storsimple-device-dashboard.md)。
* 了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-manager-service-administration.md)。

