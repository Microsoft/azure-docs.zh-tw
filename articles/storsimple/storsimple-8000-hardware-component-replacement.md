---
title: StorSimple 8000 系列硬體元件更換 | Microsoft Docs
description: 說明如何安全地更換 PCM、電池、控制器模組、EBOD 控制器、磁碟機，以及 StorSimple 裝置底座。
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
ms.custom: ''
ms.openlocfilehash: 6de50c5031db59176bdf17ecc69b934559220f6a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108446"
---
# <a name="replace-a-hardware-component-on-your-storsimple-8000-series-device"></a>更換 StorSimple 8000 系列裝置上的硬體元件

## <a name="overview"></a>概觀
硬體元件更換教學課程將說明 Microsoft Azure StorSimple 8000 系列裝置的硬體元件，以及取下並更換這些元件所需的步驟。 本文說明安全圖示、提供詳細教學課程的重點，並列出可替換的元件。

> [!IMPORTANT]
> 在嘗試取下或更換任何 StorSimple 元件之前，請確定先閱讀[安全圖示慣例](#safety-icon-conventions)和其他[安全性預防措施](storsimple-safety.md)。


### <a name="safety-icon-conventions"></a>安全性圖示慣例
下表說明本教學課程中使用的安全性圖示。 當您瀏覽取下並更換裝置元件的步驟時，請密切注意這些安全性圖示。

| 圖示 | 文字 | 其他資訊 |
|:--- |:--- |:--- |
| ![警告圖示](./media/storsimple-hardware-component-replacement/Warning.png) |**危險！** |指出危險的情況，如果無法避免，將會導致死亡或嚴重傷害。 此訊號文字僅限用於最極端的情況。 |
| ![警告圖示](./media/storsimple-hardware-component-replacement/Warning.png) |**警告！** |指出危險的情況，如果無法避免，可能會導致死亡或嚴重傷害。 |
| ![注意圖示](./media/storsimple-hardware-component-replacement/Caution.png) |**小心！** |指出危險的情況，如果無法避免，可能會導致次要或中度的傷害。 |
| ![注意事項圖示](./media/storsimple-hardware-component-replacement/NoticeIcon.png) |**注意事項：** |表示重要資訊，但與危險無關。 |
| ![電擊圖示](./media/storsimple-hardware-component-replacement/Electric.png) |**電擊危險** |表示高電壓。 |
| ![超重圖示](./media/storsimple-hardware-component-replacement/Weight.png) |**超重** | |
| ![沒有使用者可自行維修的零件圖示](./media/storsimple-hardware-component-replacement/NoUserServiceableParts.png) |**沒有使用者可自行維修的零件** |除非受過適當訓練，否則請勿觸碰。 |
| ![閱讀指示圖示](./media/storsimple-hardware-component-replacement/ReadInstructions.png) |**請先閱讀所有指示** | |
| ![傾倒危險圖示](./media/storsimple-hardware-component-replacement/TipHazard.png) |**傾倒危險** | |

### <a name="before-you-begin"></a>開始之前
讓您自己熟悉有關裝置的安全性資訊，以及本教學課程中使用的安全性圖示。 如需完整資訊，請移至 [安全地安裝和操作您的 StorSimple 裝置](storsimple-safety.md) 。 請務必閱讀 [安全性預防措施](storsimple-safety.md#handling-precautions) ，然後再處理 StorSimple 裝置。

在嘗試更換元件之前，請考量下列資訊。

![Warning Icon](./media/storsimple-hardware-component-replacement/Warning.png) ![Electrical Shock Icon](./media/storsimple-hardware-component-replacement/Electric.png) **警告！**

* 處理 StorSimple 裝置的模組和元件時，請使用靜電防護或防靜電墊，讓您自己適當地接地。
* 請勿觸及任何電路。 在處理可能曝露電路的元件時，請使用提供的把手和導路。

![Warning Icon](./media/storsimple-hardware-component-replacement/Warning.png) ![Notice Icon](./media/storsimple-hardware-component-replacement/NoticeIcon.png) **注意事項：**

當更換模組時， **決不在機箱背面留下空白機架**。 在取下問題組件之前，請取得更換或空白模組。

## <a name="hardware-component-replacement-procedures"></a>硬體元件更換程序
StoreSimple 8000 系列裝置由主要和/或 EBOD 機箱的數個外掛程式模組所組成。 8100 具有單一主要機箱，而 8600 是具有主要機箱與 EBOD 機箱的雙重機箱裝置。

下表彙總裝置中的主要硬體元件。 按一下 [更換程序]  資料行中的連結，即可移到相關聯的教學課程。

| 元件 | # Present | 外掛程式模組？ | 更換程序 |
|:--- |:--- |:--- |:--- |
| 底座 |1 |否 |[更換 StorSimple 裝置上的底座](storsimple-8000-chassis-replacement.md) |
| 主要控制器 |2 |yes |[更換 StorSimple 裝置上的控制器模組](storsimple-8000-controller-replacement.md) |
| 764 瓦電源和冷卻模組 (PCM) |2 |yes |[更換 StorSimple 裝置上的電源和冷卻模組](storsimple-8000-power-cooling-module-replacement.md) |
| 備用電池 |2 |yes |[更換 StorSimple 裝置上的備用電池模組](storsimple-8000-battery-replacement.md) |
| 磁碟機 |12 |yes |[更換 StorSimple 裝置上的磁碟機](storsimple-8000-disk-drive-replacement.md) |

**表 1** 主要機箱中的硬體元件

主要機箱和 EBOD 機箱在其 I/O 模組中各有不同。 此外，PCM 有具不同的瓦數。 主要機箱中的 PCM 為 764 瓦，而 EBOD 機箱中的 PCM 則為 580 瓦。主要機箱中的 PCM 也包含備用電池模組。

| 元件 | # Present | 外掛程式模組？ | 更換程序 |
|:--- |:--- |:--- |:--- |
| 底座 |1 |否 |[更換 StorSimple 裝置上的底座](storsimple-8000-chassis-replacement.md) |
| EBOD 控制器 |2 |yes |[更換 StorSimple 裝置上的 EBOD 控制器](storsimple-8000-ebod-controller-replacement.md) |
| 580 瓦電源和冷卻模組 (PCM) |2 |yes |[更換 StorSimple 裝置上的電源和冷卻模組](storsimple-8000-power-cooling-module-replacement.md) |
| 磁碟機 |12 |yes |[更換 StorSimple 裝置上的磁碟機](storsimple-8000-disk-drive-replacement.md) |

**表 2** EBOD 機箱中的硬體元件

裝置上的外掛程式模組會在下列前端和後端圖表中反白顯示。 如果需要更換，您可以使用這些圖表，來判斷各種外掛程式模組的位置。 前端圖表顯示磁碟機，而 EBOD 機箱和主要機箱的後端圖表則顯示外掛程式模組。

![具有磁碟機的裝置前擋板](./media/storsimple-hardware-component-replacement/IC741028.png)

**圖 1** 裝置正面

| 標籤 | 說明 |
|:--- |:--- |
| 0 - 11 |磁碟機 (總共 12 部) |

主要機箱和 EBOD 機箱都具有磁碟機載具模組。 底座裝載十二部 3.5"磁碟機，依 3 x 4 格式排列。

![裝置主要機箱模組的後擋板](./media/storsimple-hardware-component-replacement/IC740994.png)

**圖 2** 主要機箱背面

| 標籤 | 說明 |
|:--- |:--- |
| 1 |PCM 0 |
| 2 |PCM 1 |
| 3 |控制器 0 |
| 4 |控制器 1 |

![裝置 EBOD 機箱外掛程式模組的後擋板](./media/storsimple-hardware-component-replacement/IC769599.png)

**圖 3** EBOD 機箱背面

| 標籤 | 說明 |
|:--- |:--- |
| 1 |PCM 0 |
| 2 |PCM 1 |
| 3 |EBOD 控制器 0 |
| 4 |EBOD 控制器 1 |

## <a name="field-replaceable-units"></a>現場可更換裝置
下列現場可更換裝置 (FRU) 可供您的 StorSimple 裝置使用：

* 底座 (包括整合的操作面板)
* 764 W AC PCM
* 580 W AC PCM
* 具有磁碟機載具模組的硬碟機
* 控制器模組
* EBOD 控制器模組
* 備用電池模組
* 機架掛接滑軌套件

請 [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md) ，來訂購任何上述替換單位。

## <a name="next-steps"></a>後續步驟
請先閱讀所有 [安全資訊](storsimple-safety.md) ，再嘗試更換 StorSimple 硬體元件。

