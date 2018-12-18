---
title: StorSimple 8000 系列硬體元件和狀態 | Microsoft Docs
description: 了解如何透過 StorSimple 裝置管理員服務監視 StorSimple 裝置的硬體元件。
services: storsimple
documentationcenter: ''
author: alkohli
manager: twooley
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/15/2018
ms.author: alkohli
ms.openlocfilehash: 58007eea9ce25423bc3a9c2847de42db04be43eb
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42141484"
---
# <a name="use-the-storsimple-device-manager-service-to-monitor-hardware-components-and-status"></a>使用 StorSimple 裝置管理員服務監視硬體元件和狀態

## <a name="overview"></a>概觀
本文描述內部部署 StorSimple 8000 系列裝置中的各種實體和邏輯元件。 也會說明如何使用 StorSimple 裝置管理員服務中的 [狀態與硬體健康狀態] 刀鋒視窗監視裝置元件狀態。

[狀態與硬體健康狀態] 刀鋒視窗會顯示所有 StorSimple 裝置元件的硬體狀態。

在 8100 的元件清單下，有三個區段描述：

* **共用元件** – 這些元件不是像磁碟機、機箱、電源和冷卻模組 (PCM) 元件和 PCM 溫度、穩電壓，和電源線電流感應器等屬於控制器的一部分。
* **控制器 0 元件** – 位於控制器 0 的元件，例如控制器、SAS 擴展器和連接器、控制器溫度感應器和不同的網路介面。
* **控制器 1 元件** – 構成控制器 1 的元件，類似於控制器 0 的詳細元件。

8600 裝置有對應至磁碟擴充群 (EBOD) 機箱的其他元件。 在元件清單中，有五個區段。 其中有三個區段包含主要機箱中的元件，並且和 8100 所述元件相同。 EBOD 機箱有兩個其他區段在描述：

* **EBOD 控制器 0 元件** – 位於 EBOD 機箱 0 的元件，例如 EBOD 控制器、SAS 擴展器和連接器、以及控制器溫度感應器。
* **EBOD 控制器 1 元件** – 構成 EBOD 機箱 1 的元件，類似於 EBOD 機箱 0 的詳細元件。
* **EBOD 機箱共用元件** – EBOD 機箱和不屬於 EBOD 控制器的 PCM 中呈現的元件。

> [!NOTE]
> **StorSimple 雲端設備 (8010/8020) 無法使用硬體狀態。**


## <a name="monitor-the-hardware-status"></a>監視硬體狀態
執行下列步驟來檢視裝置元件的硬體狀態：

1. 瀏覽至 **裝置**，選取特定的 StorSimple 裝置。 移至 [監視] > [硬體健康狀態]。

    ![](./media/storsimple-8000-monitor-hardware-status/hw-health1.png)

2. 找出 [硬體元件] 區段並選擇可用的元件。 只要按一下元件標籤，即可展開清單並檢視各種裝置元件的狀態。 請參閱[主要機箱的詳細元件清單](#component-list-for-primary-enclosure-of-storsimple-device)和 [EBOD 機箱的詳細元件清單](#component-list-for-ebod-enclosure-of-storsimple-device)。

    ![](./media/storsimple-8000-monitor-hardware-status/hw-health2.png)

3. 您可以使用下列色彩編碼配置來解譯元件狀態：
   
   * **綠色核取符號** – 代表**正常**狀態的狀況良好元件。
   * **黃色** – 代表在**警告**狀態中的降級元件。
   * **紅色驚嘆號** – 代表有**失敗**狀態的失敗元件。
   * **白底黑字** – 代表不存在的元件。
   
   下列螢幕擷取畫面顯示有元件處於 [正常]、[警告] 和 [失敗] 狀態的裝置。
       
   ![](./media/storsimple-8000-monitor-hardware-status/hw-health3.png)

   展開 [共用的元件清單]，我們可以看到 NVRAM 與叢集已降級。

   ![](./media/storsimple-8000-monitor-hardware-status/hw-health5.png)

   展開 [控制器 1 元件] 清單，我們可以看到叢集節點失敗。  

   ![](./media/storsimple-8000-monitor-hardware-status/hw-health4.png)  

4. 如果您遇到狀態不是「狀況良好」  的元件，請連絡 Microsoft 支援服務。 如果您的裝置上啟用警示，您會收到電子郵件警示。 如果您必須更換失敗的硬體元件，請參閱 [StorSimple 硬體元件更換](storsimple-hardware-component-replacement.md)。

## <a name="component-list-for-primary-enclosure-of-storsimple-device"></a>StorSimple 裝置之主要機箱的元件清單
下表摘要列出您內部部署 StorSimple 裝置之主要機箱 (在 8100 和 8600 中皆為贈品) 中包含的實體和邏輯元件。

| 元件 | 模組 | 類型 | 位置 | 現場可更換單位 (FRU)？ | 說明 |
| --- | --- | --- | --- | --- | --- |
| 插槽 [0-11] 中的磁碟機 |磁碟機 |實體 |共用 |是 |主要機箱中的每個 SSD 或 HDD 磁碟機都表示為一行。 |
| 周圍溫度感應器 |機箱 |實體 |共用 |否 |測量底座內溫度。 |
| 中間板溫度感應器 |機箱 |實體 |共用 |否 |測量中間板的溫度。 |
| 有聲警報器 |機箱 |實體 |共用 |否 |指出底座內的有聲警報器子系統是否正常運作。 |
| 機箱 |機箱 |實體 |共用 |是 |指出底座的目前狀態。 |
| 機箱設定 |機箱 |實體 |共用 |否 |指底座的前板。 |
| 電源線電壓感應器 |PCM |實體 |共用 |否 |許多電源線電壓感應器都會顯示其狀態，指出測量的電壓是否在容許範圍內。 |
| 電源線電流感應器 |PCM |實體 |共用 |否 |許多電源線電流感應器都會顯示其狀態，指出測量的電流是否在容許範圍內。 |
| PCM 中的溫度感應器 |PCM |實體 |共用 |否 |許多溫度感應器 (例如入口和熱點感應器) 都會顯示其狀態，指出測量的溫度是否在容許範圍內。 |
| 電源供應器 [0-1] |PCM |實體 |共用 |是 |位於裝置背面的兩個 PCM 中的每個電源供應器都表示為一行。 |
| 冷卻 [0-1] |PCM |實體 |共用 |是 |位於兩個 PCM 中的四部冷卻風扇都表示為一行。 |
| 電池 [0-1] |PCM |實體 |共用 |是 |位於 PCM 中的每個備份電池模組都表示為一行。 |
| Metis |N/A |邏輯 |共用 |N/A |顯示電池狀態：是否需要充電和生命週期是否即將結束。 |
| 叢集 |N/A |邏輯 |共用 |N/A |顯示兩個整合式控制器模組之間建立的叢集狀態。 |
| 叢集節點 |N/A |邏輯 |共用 |N/A |指出做為叢集一部分的控制器狀態。 |
| 叢集仲裁 |N/A |邏輯 | |N/A |指出 HDD 儲存體集區中大部分磁碟成員資格的目前狀態。 |
| HDD 資料空間 |N/A |邏輯 |共用 |N/A |儲存空間可用於硬碟 (HDD) 儲存體集區中的資料。 |
| HDD 管理空間 |N/A |邏輯 |共用 |N/A |管理工作的 HDD 儲存體集區中保留的空間。 |
| HDD 仲裁空間 |N/A |邏輯 |共用 |N/A |叢集仲裁的 HDD 儲存體集區中保留的空間。 |
| HDD 更換空間 |N/A |邏輯 |共用 |N/A |控制器更換的 HDD 儲存體集區中保留的空間。 |
| SSD 資料空間 |N/A |邏輯 |共用 |N/A |儲存空間可用於固態硬碟 (SSD) 存放集區中的資料。 |
| SSD NVRAM 空間 |N/A |邏輯 |共用 |N/A |SSD 儲存體集區中 NVRAM 邏輯專用的儲存空間。 |
| HDD 儲存體集區 |N/A |邏輯 |共用 |N/A |顯示從裝置 HDD 建立之邏輯儲存體集區的狀態。 |
| SSD 儲存體集區 |N/A |邏輯 |共用 |N/A |顯示從裝置 SSD 建立之邏輯儲存體集區的狀態。 |
| 控制器 [0-1] [狀態] |I/O |實體 |Controller |是 |顯示控制器的狀態，以及它在底座內是作用中或待命模式。 |
| 控制器中的溫度感應器 |I/O |實體 |Controller |否 |I/O 模組、CPU 溫度、DIMM 和 PCIe 感應器等許多溫度感應器都會顯示其狀態，指出溫度是否在容許範圍內。 |
| SAS 擴展器 |I/O |實體 |Controller |否 |指出序列連接 SCSI (SAS) 擴展器的狀態，此擴展器用來連接到控制器的整合式儲存體。 |
| SAS 連接器 [0-1] |I/O |實體 |Controller |否 |指出每個 SAS 連接器的狀態，用來將整合式儲存體連接到 SAS 擴展器。 |
| SBB 中間板相互連接 |I/O |實體 |Controller |否 |指出中間板連接器的狀態，用來連接到中間板的每個控制器。 |
| 處理器核心 |I/O |實體 |Controller |否 |指出每個控制器內的處理器核心狀態。 |
| 機箱電子裝置電源 |I/O |實體 |Controller |否 |指出機箱所使用的電源系統狀態。 |
| 機箱電子裝置診斷 |I/O |實體 |Controller |否 |指出控制器所提供的診斷子系統狀態。 |
| 基礎板管理控制器 (BMC) |I/O |實體 |Controller |否 |指出基礎板管理控制器 (BMC) 的狀態，也就是特殊服務處理器，可透過感應器監視硬體裝置並與系統管理員透過獨立連接進行通訊。 |
| 乙太網路 |I/O |實體 |Controller |否 |指出每個網路介面的狀態，也就是控制器上提供的管理和資料連接埠。 |
| NVRAM |I/O |實體 |Controller |否 |指出 NVRAM 的狀態，也就是由電池備份的競態隨機存取記憶體，用來在 	電源中斷時保留關鍵應用程式資訊。 |

## <a name="component-list-for-ebod-enclosure-of-storsimple-device"></a>StorSimple 裝置之 EBOD 機箱的元件清單
下表摘要列出您內部部署 StorSimple 裝置之 EBOD 機箱 (只在 8600 機型中為贈品) 中包含的實體和邏輯元件。

| 元件 | 模組 | 類型 | 位置 | FRU？ | 說明 |
| --- | --- | --- | --- | --- | --- |
| 插槽 [0-11] 中的磁碟機 |磁碟機 |實體 |共用 |是 |EBOD 機箱前方的每個 HDD 磁碟機都表示為一行。 |
| 周圍溫度感應器 |機箱 |實體 |共用 |否 |測量底座內溫度。 |
| 中間板溫度感應器 |機箱 |實體 |共用 |否 |測量中間板的溫度。 |
| 有聲警報器 |機箱 |實體 |共用 |否 |指出底座內的有聲警報器子系統是否正常運作。 |
| 機箱 |機箱 |實體 |共用 |是 |指出底座的目前狀態。 |
| 機箱設定 |機箱 |實體 |共用 |否 |指 OPS 或底座的前板。 |
| 電源線電壓感應器 |PCM |實體 |共用 |否 |許多電源線電壓感應器都會顯示其狀態，指出測量的電壓是否在容許範圍內。 |
| 電源線電流感應器 |PCM |實體 |共用 |否 |許多電源線電流感應器都會顯示其狀態，指出測量的電流是否在容許範圍內。 |
| PCM 中的溫度感應器 |PCM |實體 |共用 |否 |許多溫度感應器 (例如入口和熱點感應器) 都會顯示其狀態，其指出測量的溫度是否在容許範圍內。 |
| 電源供應器 [0-1] |PCM |實體 |共用 |是 |位於裝置背面的兩個 PCM 中的每個電源供應器都表示為一行。 |
| 冷卻 [0-1] |PCM |實體 |共用 |是 |位於兩個 PCM 中的四部冷卻風扇都表示為一行。 |
| 本機儲存體 [HDD] |N/A |邏輯 |共用 |N/A |顯示從裝置 HDD 建立之邏輯儲存體集區的狀態。 |
| 控制器 [0-1] [狀態] |I/O |實體 |Controller |是 |EBOD 模組中會顯示控制器的狀態。 |
| EBOD 中的溫度感應器 |I/O |實體 |Controller |否 |來自每部控制器的許多溫度感應器都會顯示其狀態，其指出溫度是否在容許範圍內。 |
| SAS 擴展器 |I/O |實體 |Controller |否 |指出 SAS 擴展器的狀態，此擴展器用來連接到控制器的整合式儲存體。 |
| SAS 連接器 [0-2] |I/O |實體 |Controller |否 |指出每個 SAS 連接器的狀態，用來將整合式儲存體連接到 SAS 擴展器。 |
| SBB 中間板相互連接 |I/O |實體 |Controller |否 |指出中間板連接器的狀態，用來連接到中間板的每個控制器。 |
| 機箱電子裝置電源 |I/O |實體 |Controller |否 |指出機箱所使用的電源系統狀態。 |
| 機箱電子裝置診斷 |I/O |實體 |Controller |否 |指出控制器所提供的診斷子系統狀態。 |
| 裝置控制器的連接 |I/O |實體 |Controller |否 |指出 EBOD I/O 模組和裝置控制器之間的連接狀態。 |

## <a name="next-steps"></a>後續步驟
* 若要使用 StorSimple 裝置管理員服務管理裝置，請參閱[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。
* 如果您需要疑難排解降級或失敗狀態的裝置元件，請參閱 [StorSimple 監視指示器](storsimple-monitoring-indicators.md)。
* 若要更換故障的硬體元件，請參閱 [StorSimple 硬體元件更換](storsimple-hardware-component-replacement.md)。
* 如果持續發生裝置問題，請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。

