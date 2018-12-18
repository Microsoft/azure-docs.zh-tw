---
title: StorSimple 技術規格 | Microsoft Docs
description: 說明適用於 StorSimple 硬體元件的技術規格與法規標準符合性資訊。
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/02/2017
ms.author: alkohli
ms.openlocfilehash: 855ea6c34082b859bb5b5b6e69b3e3f2fa54eb4a
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39056458"
---
# <a name="technical-specifications-and-compliance-for-the-storsimple-device"></a>適用於 StorSimple 裝置的技術規格和相容性

## <a name="overview"></a>概觀

Microsoft Azure StorSimple 裝置的硬體元件會遵循本文中概述的技術規格與法規標準。 技術規格描述電源與冷卻模組 (PCM)、磁碟機、儲存體容量及機箱。 相容性資訊則涵蓋國際標準、安全性和排放量，以及連接纜線等資訊。

## <a name="power-and-cooling-module-specifications"></a>電源和冷卻模組規格

StorSimple 裝置配置兩個 100-240 V 雙風扇、符合 SBB 規範的電源和冷卻模組 (PCM)。 這提供了備援電源設定。 如果 PCM 故障，裝置會繼續在其他 PCM 上正常運作，直到更換故障的模組為止。

EBOD 機箱使用 580 W PCM，而主要機箱會使用 764 W PCM。 下表列出與 PCM 相關聯的技術規格。

| 規格 | 580 W PCM (EBOD) | 764 W PCM (主要) |
| --- | --- | --- |
| 最大輸出電力 |580 W |764 |
| 頻率 |50/60 Hz |50/60 Hz |
| 選取電壓範圍 |自動設定範圍：90 - 264 V AC、47/63 Hz |自動設定範圍：90 - 264 V AC、47/63 Hz |
| 最大瞬間電流 |20 A |20 A |
| 功率因素校正 |> 95% 額定輸入電壓 |> 95% 額定輸入電壓 |
| 諧波 |符合 EN61000-3-2 |符合 EN61000-3-2 |
| 輸出 |5V 待機電壓 \@ 2.0 A |5V 待機電壓 \@ 2.7 A |
| +5V \@ 42 A |+5V \@ 40 A | |
| +12V \@ 38 A |+12V \@ 38 A | |
| 隨插即用 |是 |是 |
| 開關和 LED |AC ON/OFF 開關和四個狀態指示器 LED |AC ON/OFF 開關和六個狀態指示器 LED |
| 機箱冷卻 |具有變動風扇速度控制的軸流散熱風扇 |具有變動風扇速度控制的軸流散熱風扇 |

## <a name="power-consumption-statistics"></a>耗電量統計資料

下表列出各種 StorSimple 裝置機型的一般耗電量資料 (實際值與發佈資料可能不盡相同)。

| 條件 | 240 V AC | 240 V AC | 240 V AC | 110 V AC | 110 V AC | 110 V AC |
| --- | --- | --- | --- | --- | --- | --- |
|  風扇慢速，磁碟機閒置 |1.45 A |0.31 kW |1057.76 BTU/時 |3.19 A |0.34 kW |1160.13 BTU/時 |
|  風扇慢速，磁碟機存取中 |1.54 A |0.33 kW |1126.01 BTU/時 |3.27 A |0.36 kW |1228.37 BTU/時 |
|  風扇快速，磁碟機閒置，兩個 PSU 已運轉 |2.14 A |0.49 kW |1671.95 BTU/時 |4.99 A |0.54 kW |1842.56 BTU/時 |
|  風扇快速，磁碟機閒置，一個 PSU 已運轉、一個 PSU 閒置 |2.05 A |0.48 kW |1637.83 BTU/時 |4.58 A |0.50 kW |1706.07 BTU/時 |
|  風扇快速，磁碟機存取中，兩個 PSU 已運轉 |2.26 A |0.51 kW |1740.19 BTU/時 |4.95 A |0.54 kW |1842.56 BTU/時 |
|  風扇快速，磁碟機存取中，一個 PSU 已運轉、一個 PSU 閒置 |2.14 A |0.49 kW |1671.95 BTU/時 |4.81 A |0.53 kW |1808.44 BTU/時 |

## <a name="disk-drive-specifications"></a>磁碟機規格

您的 StorSimple 裝置最大可支援 12 3.5" 規格的序列連接 SCSI (SAS) 磁碟機。 視產品設定而定，實際的磁碟機可以是固態磁碟 (SSD) 或硬碟 (HDD)。 機箱正面有 12 個磁碟機插槽 (採用 3 x 4 設定)。 EBOD 機箱允許另外 12 個磁碟機提供額外的儲存空間。 這些磁碟機一律為 HDD。

## <a name="storage-specifications"></a>儲存體規格

StorSimple 裝置混合搭載 8100 及 8600 的硬碟與固態磁碟機。 8100 與 8600 的可用容量總計各約 15 TB 及 38 TB。 下表列出 StorSimple 解決方案的容量內容中，HDD、SSD 及雲端的容量詳細資料。

| 裝置型號/容量 | 8100 | 8600 |
| --- | --- | --- |
| 硬碟 (HDD) 數 |8 |19 |
| 固態硬碟 (SSD) 數 |4 |5 |
| 單一 HDD 容量 |4 TB |4 TB |
| 單一 SSD 容量 |400 GB |800 GB |
| 備用容量 |4 TB |4 TB |
| 可用的 HDD 容量 |14 TB |36 TB |
| 可用的 SSD 容量 |800 GB |2 TB |
| 可用容量總計* |~ 15 TB |~ 38 TB |
| 解決方案的最大容量 (含雲端) |200 TB |500 TB |

<sup>* </sup>- *可用的容量總計包括可供資料、中繼資料及緩衝區使用的容量。您可以在 8100 裝置上佈建最多 8.5 TB 的本機固定磁碟區，或者在較大的 8600 裝置上佈建最多 22.5 TB 的磁碟區。如需詳細資訊，請移至 [StorSimple 本機固定磁碟區](storsimple-8000-local-volume-faq.md)。*

## <a name="enclosure-dimensions-and-weight-specifications"></a>機箱尺寸和重量規格

下表列出各種機箱的尺寸和重量規格。

### <a name="enclosure-dimensions"></a>機箱尺寸

下表列出機箱尺寸 (以公釐和英吋為單位)。

| 機箱 | 公釐 | 英吋 |
| --- | --- | --- |
| 高度 |87.9 |3.46 |
| 法蘭座的寬度 |483 |19.02 |
| 機身的寬度 |443 |17.44 |
| 從法蘭座至機身末端的深度 |577 |22.72 |
| 從操作面板至機箱最末端的深度 |630.5 |24.82 |
| 從法蘭座至機箱最末端的深度 |603 |23.74 |

### <a name="enclosure-weight"></a>機箱重量

視設定而定，完全裝載的主要機箱重量從 21 至 33 公斤，需要兩個人才能搬動。

| 機箱 | 重量 |
| --- | --- |
| 最大重量 (取決於設定) |30 kg - 33 kg |
| 空的 (未裝入磁碟機) |21 - 23 kg |

## <a name="enclosure-environment-specifications"></a>機箱環境規格

本節將列出機箱環境的相關規格。 溫度、溼度、海拔高度、震盪、振動、方向、安全和電磁相容性 (EMC) 均納入此類別中。

### <a name="temperature-and-humidity"></a>溫度和溼度

| 機箱 | 周圍溫度範圍 | 周圍相對溼度 | 最大濕球 |
| --- | --- | --- | --- |
| 運作 |5°C - 35°C(41°F - 95°F) |20% - 80% 無凝結- |28°C (82°F) |
| 無法運作 |-40°C - 70°C(40°F - 158°F) |5% - 100% 無凝結 |29°C (84°F) |

### <a name="airflow-altitude-shock-vibration-orientation-safety-and-emc"></a>氣流、海拔高度、震盪、振動、方向、安全和 EMC

| 機箱 | 運作規格 |
| --- | --- |
| 氣流 |系統氣流是從前面到後面。 系統必須以低壓、尾段排氣安裝操作。 機架門和障礙物所產生的背壓不得超出 5 帕斯卡 (0.5 mm 水位標尺)。 |
| 可運作的海拔高度 |-30 公尺至 3045 公尺 (-100 英呎至 10,000 英呎)，其最大運作溫度的降額在 7000 英呎以上為 5°C。 |
| 無法運作的海拔高度 |-305 公尺至 12,192 公尺 (-1,000 英呎至 40,000 英呎) |
| 可運作的震盪 |5g 10 ms ½ sine |
| 無法運作的震盪 |30g 10 ms ½ sine |
| 可運作的振動 |0.21g RMS 5-500 Hz 隨機 |
| 無法運作的振動 |1.04g RMS 2-200 Hz 隨機 |
| 振動 (重新放置) |3g 2-200 Hz sine |
| 定位和掛接 |19" 機架掛接 (2 個 EIA 單位) |
| 機架滑軌 |符合與 IEC 297 相容的最小 700 mm (31.50 英吋) 深度機架 |
| 安全與核可 |CE 和 UL EN 61000-3、IEC 61000-3、UL 61000-3 |
| EMC |EN55022 (CISPR - A)，FCC A |

## <a name="international-standards-compliance"></a>國際標準相容性

您的 Microsoft Azure StorSimple 裝置符合下列國際標準：  

* CE - EN 60950 - 1
* IEC 60950 - 1 的 CB 報告
* UL 60950 - 1 的 UL 和 cUL

## <a name="safety-compliance"></a>安全法規遵循

您的 Microsoft Azure StorSimple 裝置符合下列安全分級：

* 系統產品類型核可：UL、cUL、CE
* 安全法規遵循：UL 60950、IEC 60950、EN 60950

## <a name="emc-compliance"></a>EMC 法規遵循

您的 Microsoft Azure StorSimple 裝置符合下列 EMC 分級：

### <a name="emissions"></a>放射性規格

此裝置符合傳導和輻射放射性層級的 EMC 標準。

* 傳導放射性限制層級：CFR 47 Part 15B Class A EN55022 Class A CISPR Class A
* 輻射放射性限制層級：CFR 47 Part 15B Class A EN55022 Class A CISPR Class A

### <a name="harmonics-and-flicker"></a>諧波和變動

此裝置符合 EN61000-3-2/3。

### <a name="immunity-limit-levels"></a>免疫限制層級

此裝置符合 EN55024。

## <a name="ac-power-cord-compliance"></a>AC 電源線相容性

插頭和完整電源線組件必須符合適用於使用裝置之國家或地區的標準，而且必須具有該國家或地區可接受的安全核可。 下表列出適用於美國和歐洲的標準。

### <a name="ac-power-cords---usa-must-be-nrtl-listed"></a>AC 電源線 - 美國 (必須由 NRTL 列名)

| 元件 | 規格 |
| --- | --- |
| 電源線類型 |SV 或 SVT、最少 18 AWG、3 導體、最大長度為 2.0 公尺 |
| 插頭 |NEMA 5-15P 接地式插塞插頭，額定電壓 120 V、10 A 或 IEC 320 C14、250 V、10 A |
| 插座 |IEC 320 C-13、250 V、10 A |

### <a name="ac-power-cords---europe"></a>AC 電源線 - 歐洲

| 元件 | 規格 |
| --- | --- |
| 電源線類型 |諧波，H05-VVF-3G1.0 |
| 插座 |IEC 320 C-13、250 V、10 A |

## <a name="supported-network-cables"></a>支援的網路纜線

針對 10 GbE 的網路介面 DATA 2 和 DATA 3，請參閱 [支援的網路纜線和模組清單](storsimple-supported-hardware-for-10-gbe-network-interfaces.md)。

## <a name="next-steps"></a>後續步驟

您現在已可在您的資料中心內部署 StorSimple 裝置。 如需詳細資訊，請參閱 [部署您的內部部署裝置](storsimple-8000-deployment-walkthrough-u2.md)。

