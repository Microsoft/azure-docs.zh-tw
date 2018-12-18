---
title: 更換 Microsoft Azure StorSimple 8000 系列裝置上的電池 | Microsoft Docs
description: 描述如何取下、更換和維護 StorSimple 裝置上的備份電池模組。
services: storsimple
documentationcenter: ''
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: f8071cde67017ff031418f0d97da15a618c4969b
ms.sourcegitcommit: 9292e15fc80cc9df3e62731bafdcb0bb98c256e1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/10/2018
ms.locfileid: "27742808"
---
# <a name="replace-the-backup-battery-module-on-your-storsimple-device"></a>更換 StorSimple 裝置上的備份電池模組

## <a name="overview"></a>概觀
Microsoft Azure StorSimple 裝置上的主要機箱電源和冷卻模組 (PCM) 具有額外的電池組。 這個電池組會提供電源，以便如果主要機箱失去 AC 電源，StorSimple 裝置可以儲存資料。 這個電池組稱為 *備份電池模組*。 備份電池模組僅針對 StorSimple 裝置中的主要機箱而存在 (EBOD 機箱未包含備份電池模組) 。

本教學課程說明如何：

* 取下備份電池模組
* 安裝新的備份電池模組
* 維護備份電池模組

> [!IMPORTANT]
> 取下及更換備份電池模組之前，請閱讀 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)中的安全資訊。


## <a name="remove-the-backup-battery-module"></a>取下備份電池模組
StorSimple 裝置的備份電池模組是現場可置換裝置。 在 PCM 中安裝它之前，電池模組應該儲存在其原始包裝中。 執行下列步驟以取下備份電池。

#### <a name="to-remove-the-backup-battery-module"></a>若要取下備份電池模組
1. 在 Azure 入口網站中，移至您的 StorSimple 裝置管理員服務刀鋒視窗。 移至 [裝置]，然後從裝置清單選取您的裝置。 瀏覽至 [監視器] > [硬體健康情況]。 在 [共用元件] 下，查看電池狀態。
2. 識別電池故障的 PCM。 圖 1 顯示 StorSimple 裝置的背面。
   
    ![裝置主要機箱模組的後擋板](./media/storsimple-battery-replacement/IC740994.png)
   
    **圖 1** 顯示 PCM 和控制器模組的主要裝置背面
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |PCM 0 |
   | 2 |PCM 1 |
   | 3 |控制器 0 |
   | 4 |控制器 1 |
   
    如圖 2 中的號碼 3 所示，PCM 0 上對應到 **電池故障** 的監視指示器 LED 應該亮起。
   
    ![裝置後擋板 PCM 監視 LED 指示燈](./media/storsimple-battery-replacement/IC740992.png)
   
    **圖 2** 顯示監視指示器 LED 的 PCM 背面
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |AC 電源故障 |
   | 2 |風扇故障 |
   | 3 |電池故障 |
   | 4 |PCM 正常 |
   | 5 |DC 電源故障 |
   | 6 |電池狀況良好 |
3. 若要取下電池故障的 PCM，請遵循 [取下 PCM](storsimple-8000-power-cooling-module-replacement.md#remove-a-pcm)中的步驟。
4. 一旦取下 PCM，請提起並向上旋轉電池模組把手 (如下圖中所示)，並將它拔出以取下電池。
   
    ![從 pcm 取出電池](./media/storsimple-battery-replacement/IC741019.png)
   
    **圖 3** 從 PCM 取下電池
5. 將模組放置在現場可更換裝置包裝中。
6. 將故障裝置退回給 Microsoft，以取得適當的服務和處理。

## <a name="install-a-new-backup-battery-module"></a>安裝新的備份電池模組
執行下列步驟，以在 StorSimple 裝置的主要機箱中的 PCM 安裝更換電池模組。

#### <a name="to-install-the-battery-module"></a>若要安裝電池模組
1. 以適當的方向將備份電池模組放入 PCM 中。
2. 全力按壓電池模組把手以固定連接器。
3. 依照 [更換 StorSimple 裝置上的電源和冷卻模組](storsimple-8000-power-cooling-module-replacement.md)中的指導方針，更換主要機箱的 PCM。
4. 在更換完成之後，請移至您的裝置，然後在 Azure 入口網站移至 [監視器] > [硬體健康情況]。 確認電池的狀態，以確定安裝成功。 綠色狀態表示電池狀況良好。

## <a name="maintain-the-backup-battery-module"></a>維護備份電池模組
在 StorSimple 裝置中，備份電池模組會在停電期間提供電源給控制器。 它可讓 StorSimple 裝置在以控制方式關閉之前儲存重要資料。 由於 PCM 中有兩個完全充電的電池，系統可以處理兩個連續停電事件。

在 Azure 入口網站中，[監視器] 刀鋒視窗下的 [硬體健康情況] 指出電池是否故障，或電力何時即將用光。 在 [共用元件] 下，電池狀態是以 [PCM 1 中的電池] 或 [PCM 0 中的電池] 指出。 此刀鋒視窗會顯示 [降級] 狀態，表示電力即將用光，或顯示 [故障]，表示電力已用光。

> [!NOTE]
> 當電池只需充電時，可以報告 [ **故障** ]。


如果 [ **降級** ] 狀態出現，我們建議採取下列動作：

* 系統最近可能遭遇停電，或電池可能正在進行定期維護。 觀察系統 12 小時，再繼續進行。
  
  * 在連續 12 小時連接至 AC 電源，而且控制器與 PCM 正在執行之後，如果狀態仍為 [ **降級** ]，則需要更換電池。 請 [連絡 Microsoft 支援](storsimple-8000-contact-microsoft-support.md) ，以取得更換備份電池模組。
  * 如果狀態在 12 小時之後變成良好，則電池可操作，而且只需維護費用。
* 如果沒有相關聯的 AC 電源喪失，而且 PCM 已開啟並連接至 AC 電源，則電池需要更換。 [Contact Microsoft Support](storsimple-8000-contact-microsoft-support.md) ，以訂購更換備份電池模組。

> [!IMPORTANT]
> 依據國家及地區法規處置故障電池。

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

