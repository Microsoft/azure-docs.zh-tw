---
title: StorSimple 裝置的安全性 | Microsoft Docs
description: 描述安全性慣例、方針和考量，並說明如何安全地安裝和操作您的 StorSimple 裝置。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: dae6d535-1ca2-4d2b-b221-6819043aa068
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/16/2016
ms.author: alkohli
ms.openlocfilehash: a178e8880bcbcada9d66eaacf5ccbdb7c55957cb
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109926"
---
# <a name="safely-install-and-operate-your-storsimple-device"></a>安全地安裝和操作您的 StorSimple 裝置
![警告圖示](./media/storsimple-safety/IC740879.png)
 ![閱讀安全性注意事項圖示](./media/storsimple-safety/IC740885.png) **讀取的安全性和健全狀況資訊**

請閱讀本文中適用於 Microsoft Azure StorSimple 裝置的所有安全性和健全狀況資訊。 保留隨附於 StorSimple 裝置的所有列印指南供日後參考。 無法遵循下列指示並適當地設定、使用和照顧這項產品可能會提高嚴重傷害或人員死亡的風險，或造成裝置的損毀。 [本指南的可下載版本](http://www.microsoft.com/download/details.aspx?id=44233) 也可供使用。

## <a name="safety-icon-conventions"></a>安全性圖示慣例
以下是您在檢閱安全性預防措施時會發現的圖示，這些措施可在設定與執行 Microsoft Azure StorSimple 裝置時觀查到。

| 圖示 | 說明 |
|:--- |:--- |
| ![危險圖示](./media/storsimple-safety/IC740879.png) **危險！** |指出危險的情況，如果無法避免，將會導致死亡或嚴重傷害。 此訊號文字僅限用於最極端的情況。 |
| ![警告圖示](./media/storsimple-safety/IC740879.png) **警告！** |指出危險的情況，如果無法避免，可能會導致死亡或嚴重傷害。 |
| ![警告圖示](./media/storsimple-safety/IC740879.png) **小心！** |指出危險的情況，如果無法避免，可能會導致次要或中度的傷害。 |
| ![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：** |表示重要資訊，但與危險無關。 |
| ![電擊圖示](./media/storsimple-safety/IC740882.png) **電擊危險** |高電壓 |
| ![超重圖示](./media/storsimple-safety/IC740883.png) **重量** | |
| ![沒有使用者可自行維修零件圖示](./media/storsimple-safety/IC740879.png) **沒有使用者可自行維修的零件** |除非受過適當訓練，否則請勿觸碰。 |
| ![閱讀安全性注意事項圖示](./media/storsimple-safety/IC740885.png)**請先閱讀所有指示** | |
| ![傾倒危險圖示](./media/storsimple-safety/IC740886.png) **傾倒危險** | |

## <a name="handling-precautions"></a>處理預防措施
![警告圖示](./media/storsimple-safety/IC740879.png) ![超重圖示](./media/storsimple-safety/IC740883.png) **警告！** 

減少受傷的風險：

* 完整設定的機箱重達 32 公斤 (70 磅)；請勿嘗試自己將其抬起。
* 移動機箱之前，請務必確定由兩人承受重量。 請注意，一個人嘗試抬起這個重量可能造成受傷。
* 請勿利用單元後方電源和冷卻模組 (PCM) 上的把手抬起機箱。 這些把手不是設計來承受重量的。

## <a name="connection-precautions"></a>連接預防措施
![Warning Icon](./media/storsimple-safety/IC740879.png) ![Electrical Shock Icon](./media/storsimple-safety/IC740882.png) **警告！**

減少傷害、電擊或死亡的可能性：

* 由多個 AC 來源提供電源時，請中斷所有供應電源以達完全隔離。
* 在您移動單元，或者認為單元受到任何損毀時，請一律拔除電源。
* 提供電源供應線安全的接地線。 請確認機箱接地符合國家和當地需求，然後再供應電源。
* 從機箱移除 PCM 之前請務必中斷電源連接。
* 假設電源供應線上的插頭是主要的中斷裝置，請確保插座位於設備附近且可輕鬆觸及。

![Warning Icon](./media/storsimple-safety/IC740879.png) ![Electrical Shock Icon](./media/storsimple-safety/IC740882.png) **警告！**

降低電力連接過熱或火災的可能性：

* 提供適當的電源及電力多載保護，以符合技術規格中詳細列出的需求。
* 請勿使用分叉的電源線 ("Y" 引線)。
* 若要符合適用的安全性、放射性與熱需求，您不能移除任何封蓋且所有機架必須填入外掛程式模組或磁碟機空白。
* 請確定以製造商所指定的方式使用設備。 如果未以製造商所指定的方式使用設備，設備所提供的保護可能會減損。

![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：**

讓您的設備正確作業並避免產品損害：

* 在裝置背面的 RJ45 連接埠僅適用於乙太網路連線。 這些不必連接到電信網路。
* 請務必在可容納前後冷卻設計的機架上安裝該裝置。
* 所有外掛程式模組和空白面板都是系統機箱的一部分。 立即加入替代項目時必須只能將其移除。 系統不能在所有模組或空白都備妥之前執行。

## <a name="rack-system-precautions"></a>機架系統預防措施
在機櫃中掛接裝置時，必須考量下列安全性需求。

![警告圖示](./media/storsimple-safety/IC740879.png) ![傾倒危險圖示](./media/storsimple-safety/IC740886.png) **警告！**

減少因為傾倒而受傷的可能性：

* 機架設計應該支援已安裝機箱的總重量，而且應該納入穩定功能，適用於防止機架在安裝或正常使用期間傾倒或推倒。
* 載入時機架，從下到上填滿機架並從上到下清空。
* 不要一次從機架滑動超過一個機箱，以避免機架倒塌的危險。

![警告圖示](./media/storsimple-safety/IC740879.png) ![電擊圖示](./media/storsimple-safety/IC740882.png) **警告！**

減少傷害、電擊或死亡的可能性：

* 機架應該有一個安全的電子散發系統。 它必須提供機箱的過電流保護，而且安裝的機箱總數不能超載。 應該觀察顯示在名牌上的電力消耗評等。
* 電力發散系統必須提供可靠的地面給機架中的每個機箱。
* 電力發散系統的設計必須考量所有機箱中所有電源供應器的總漏地電流。 請注意，每個機箱中的每個電源供應器都有最大值 1.0 mA，60 Hz、264 伏特的漏地電流。 機架可能需要加上下列標籤「高漏地電流。 在連接供應電源之前，接地 (接地) 是不可或缺的。
* 利用機箱設定機架時，機架上必須符合 UL 60950-1 和 IEC 60950-1/EN 60950-1 的安全性需求。

![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：**

關於機架系統的適當冷卻：

* 請確定機架設計考量到機箱作業周圍溫度最大值為攝氏 35 度 (華氏 95 度)。
* 系統在低壓、後方排氣安裝的條件下運作 (機架門和障礙建立的背部壓力不能超過 5 Pascal [0.5 mm-1.0 水位表])。

## <a name="power-cooling-module-pcm-precautions"></a>電源冷卻模組 (PCM) 預防措施
裝置設計為利用兩個 PCM 運作。 每個 PCM 都具有電源供應器和雙軸風扇。 在嚴苛條件期間，系統會允許其中一個電源供應器失敗，同時繼續正常作業。 兩個 PCM (因此和電源供應器) 必須一律安裝。 單一 PCM 不會提供備援電源。 因此，即使一個 PCM 失敗也可能會導致停機或可能會遺失資料。

![警告圖示](./media/storsimple-safety/IC740879.png) ![電擊圖示](./media/storsimple-safety/IC740882.png) **警告！**

減少傷害、電擊或死亡的可能性：

* 請勿從 PCM 移除封蓋。 沒有內部觸電的危險。 若要傳回 PCM 並取得替代項目，請 [連絡 Microsoft 支援服務](storsimple-contact-microsoft-support.md)。

![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：**

讓您的設備正確作業並避免產品損害：

* 您必須在 24 小時內取代失敗的 PCM。 移除 PCM 以進行取代之後，必須在移除後 10 分鐘內完成取代。
* 請勿移除 PCM，除非可以立即安裝替代項目。 機箱不得在所有模組尚未備妥之前運作。

## <a name="electrostatic-discharge-esd-precautions"></a>靜電放電 (ESD) 預防措施
![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：**

請遵循下列 ESD 相關的預防措施。

* 請確定您已安裝並檢查適合的抗靜電腕或踝帶。
* 處理模組和元件時，請觀察所有傳統的 ESD 預防措施。
* 避免接觸後擋板元件和模組連接器。
* 瑕疵擔保未涵蓋 ESD 損害。

## <a name="battery-disposal-precautions"></a>電池處置預防措施
電源供應器會使用特殊的電池，以在暫時的短期電源中斷期間保護記憶體的內容。 電池置於 PCM 中。 記住下列電池相關資訊。

![警告圖示](./media/storsimple-safety/IC740879.png) **警告！**

減少短路、火災、爆炸、傷害或死亡的風險：

* 根據國家/地區規定處置使用過的電池。
* 請勿拆解、損毀或加熱至高於攝氏 60 度 (華氏 140 度) 或焚毀。 只能以提供的電池取代 PCM 電池。 使用另一個電池可能有火災或爆炸的風險。
* 如果從電源供應器移除電池，請使用電池上的保護端帽。

![注意事項圖示](./media/storsimple-safety/IC740881.png) **注意事項：**

以空運傳送或傳輸電池時，請遵循 [http://www.iata.org/whatwedo/cargo/dgr/Pages/lithium-batteries.aspx](http://www.iata.org/whatwedo/cargo/dgr/Pages/lithium-batteries.aspx)

在您檢閱這些安全性注意事項之後，接下來的步驟是為裝置解除封裝、安裝機架和纜線。

## <a name="next-steps"></a>後續步驟
* 針對 8100 裝置，請至 [安裝您的 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md)。
* 針對 8600 裝置，請至 [安裝您的 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md)。

