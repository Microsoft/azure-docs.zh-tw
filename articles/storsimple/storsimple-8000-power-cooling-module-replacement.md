---
title: 更換 StorSimple 8000 系列裝置上的 PCM | Microsoft Docs
description: 說明如何取下及更換 StorSimple 裝置上的電源和冷卻模組
services: storsimple
documentationcenter: ''
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
ms.openlocfilehash: 7d181e6e434c998573dbea4b541cfacf7a28ee66
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108596"
---
# <a name="replace-a-power-and-cooling-module-on-your-storsimple-device"></a>更換 StorSimple 裝置上的電源和冷卻模組
## <a name="overview"></a>概觀
Microsoft Azure StorSimple 裝置的電源和冷卻模組 (PCM) 包含電源供應器和冷卻風扇，是透過主要及 EBOD 機箱控制。 每個機箱只有一個認證的 PCM 模型。 764 W PCM 是認證的主要機箱， 580 W PCM 是認證的 EBOD 機箱。 雖然主要機箱和 EBOD 機箱的 PCM 不同，更換程序完全相同。

本教學課程說明如何：

* 取下 PCM
* 安裝替代的 PCM

> [!IMPORTANT]
> 取下及更換 PCM 之前，請閱讀 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)中的安全資訊。


## <a name="before-you-replace-a-pcm"></a>更換 PCM 之前
更換 PCM 前，請注意下列重要問題：

* 如果 PCM 的電源供應器故障，故障模組請保留不動，但拔掉電源線。 風扇會繼續從機箱獲得電力，並持續提供適當的冷卻。 如果風扇故障，必須立刻更換 PCM。
* 取下 PCM 前，請先中斷 PCM 電力，可關閉主開關 (如果有的話) 或是直接拔掉電源線。 這是對您的系統發出即將關閉電源的警告。
* 取代故障的 PCM 前，請確定其他 PCM 功能正常可供系統持續作業。 必須盡快以可完全正常運作的 PCM 取代故障的 PCM。
* PCM 模組的更換只需要幾分鐘即可完成，但必須在取下故障 PCM 的 10 分鐘內完成以防止過熱。
* 請注意，從工廠出貨的替代 764 W PCM 模組將不會包含備用電池模組。 您在執行替代作業之前，必須先取出故障 PCM 的電池，並將它插入替代模組。 如需詳細資訊，請參閱如何 [移除並插入備用電池模組](storsimple-8000-battery-replacement.md)。

## <a name="remove-a-pcm"></a>取下 PCM
當您準備好要取下 Microsoft Azure StorSimple 裝置的電源和冷卻模組 (PCM)，請遵循這些指示。

> [!NOTE]
> 取下 PCM 之前，請確認您有正確的替代元件 (主要機箱使用764 W，EBOD 機箱使用 580 W)。

#### <a name="to-remove-a-pcm"></a>取下 PCM
1. 在 Azure 傳統入口網站中，按一下 [設定] > [監視] > [硬體健康狀態]。 檢查 [共用元件] 下 PCM 元件的狀態，找出故障的 PCM：
   
   * 如果 PCM 0 的電源供應器故障，[PCM 0 中的電源供應器]  的狀態會變成紅色。
   * 如果 PCM 1 的電源供應器故障，[PCM 1 中的電源供應器]  的狀態會變成紅色。
   * 如果 PCM 1 的風扇故障，[PCM 0 的冷卻 0]或 [PCM 0 的冷卻 1] 其中之一的狀態會變成紅色。
2. 在主要機箱背面找到故障的 PCM。 如果您使用的是 8600 型號，請查看前端面板 LED 顯示器上顯示的「系統單元識別碼」來識別主要機箱。 主要機箱的預設單位識別碼是 **00**，EBOD 機箱的預設單位識別碼是 **01**。 下圖和下表說明 LED 顯示器的前端面板。
   
    ![前置 OPS 面板上的系統識別碼](./media/storsimple-power-cooling-module-replacement/IC740991.png)
   
     **圖 1** 裝置的正面面板  
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |靜音按鈕 |
   | 2 |系統電源 |
   | 3 |模組錯誤 |
   | 4 |邏輯錯誤 |
   | 5 |單元識別碼顯示 |
3. 主要機箱背面的監視 LED 指示燈也可用來識別故障的 PCM。 請看下列圖表以了解如何使用 LED 找出故障的 PCM。 例如，如果對應至 [風扇故障]  的 LED 燈亮了，表示風扇故障。 同樣的，如果對應至 [AC 故障]  的 LED 燈亮了，表示電源供應器故障。 
   
    ![裝置後擋板 PCM 監視 LED 指示燈](./media/storsimple-power-cooling-module-replacement/IC740992.png)
   
     **圖 2** PCM 背面和 LED 指示燈
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |AC 電源故障 |
   | 2 |風扇故障 |
   | 3 |電池故障 |
   | 4 |PCM 正常 |
   | 5 |DC 電源故障 |
   | 6 |電池狀況良好 |
4. 請看以下的 StorSimple 裝置背面圖，找出故障的 PCM 模組。 左邊是 PCM 0，右邊是 PCM 1。 下表說明模組。
   
     ![裝置主要機箱模組的後擋板](./media/storsimple-power-cooling-module-replacement/IC740994.png)
   
     **圖 3** 裝置背面和外掛程式模組 
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |PCM 0 |
   | 2 |PCM 1 |
   | 3 |控制器 0 |
   | 4 |控制器 1 |
5. 關閉故障的 PCM，拔下電源供應器電線。 您現在可以取下 PCM。
6. 用姆指與食指抓住閂鎖和 PCM 把手的一端，擠壓在一起以開啟把手。
   
    ![開啟 PCM 把手](./media/storsimple-power-cooling-module-replacement/IC740995.png)
   
    **圖 4** 打開 PCM 把手
7. 抓住把手，取下 PCM。
   
    ![取下裝置 PCM](./media/storsimple-power-cooling-module-replacement/IC740996.png)
   
    **圖 5** 取下 PCM

## <a name="install-a-replacement-pcm"></a>安裝替代的 PCM
請遵循這些指示安裝 StorSimple 裝置的 PCM。 請確定您在安裝替代 PCM 之前已經插入備用電池模組 (僅適用 764 W PCM)。 如需詳細資訊，請參閱如何 [移除並插入備用電池模組](storsimple-8000-battery-replacement.md)。

#### <a name="to-install-a-pcm"></a>安裝 PCM
1. 請確認您有這個機箱的正確替代 PCM。 主要機箱需使用 764 W PCM，EBOD 機箱需使用 580 W PCM。 您不應該嘗試在主要機箱中使用 580 W PCM，或在 EBOD 機箱中使用 764 W PCM。 下圖顯示貼在 PCM 上的標籤中何處可找到這項資訊。
   
    ![裝置 PCM 標籤](./media/storsimple-power-cooling-module-replacement/IC740973.png)
   
    **圖 6** PCM 標籤
2. 檢查機箱有無損毀，並特別注意接頭。 
   
   > [!NOTE]
   > **如果接頭的任何接腳彎曲，請勿安裝該模組。**
   > 
   > 
3. PCM 把手在開啟的位置，將模組滑入機箱。
   
    ![安裝裝置 PCM](./media/storsimple-power-cooling-module-replacement/IC740975.png)
   
    **圖 7** 安裝 PCM
4. 手動關閉 PCM 把手。 您應該會聽到喀嚓一聲，表示把手閂鎖已扣上。
   
   > [!NOTE]
   > 若要確保接頭的接腳確實連接，您可以用不會鬆開閂鎖的力道輕拉把手。 如果 PCM 滑出來，表示接頭連接之前閂鎖就關閉了。
   
5. 將電源線插到電力來源和 PCM。
6. 收妥防拉束。
7. 開啟 PCM。
8. 確認更換成功：在 StorSimple 裝置管理員服務的 Azure 入口網站中，導覽至裝置，然後至 [設定] > [監視] > [硬體健康狀態]。 在 [共用元件] 下，PCM 的狀態應該是綠色。
   
   > [!NOTE]
   > 可能需要幾分鐘的時間讓替代的 PCM 完全初始化。

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

