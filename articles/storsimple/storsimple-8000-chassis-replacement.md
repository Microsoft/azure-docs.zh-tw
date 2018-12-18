---
title: 更換 StorSimple 8000 系列裝置上的底座 | Microsoft Docs
description: 描述如何移除並更換 StorSimple 主要機箱或 EBOD 機箱的底座。
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
ms.date: 06/05/2017
ms.author: alkohli
ms.openlocfilehash: 073fcf0064f1d1482f4683d733f00cf918ff2f38
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108156"
---
# <a name="replace-the-chassis-on-your-storsimple-device"></a>更換 StorSimple 裝置上的底座
## <a name="overview"></a>概觀
本教學課程說明如何取下並更換 StorSimple 8000 系列裝置中的底座。 StorSimple 8100 模型是單一機箱裝置 (一個底座)，而 8600 是雙重機箱裝置 (兩個底座)。 若為 8600 型號，裝置中可能有兩個會發生故障的底座：主要機箱的底座或 EBOD 機箱的底座。

在任一情況下，Microsoft 隨附的更換底座是空的。 將不會包含電源和冷卻模組 (PCM)、控制器模組、固態硬碟 (SSD)、硬碟機 (HDD) 或 EBOD 模組。

> [!IMPORTANT]
> 取下並更換底座之前，請閱讀 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)中的安全資訊。


## <a name="remove-the-chassis"></a>取下底座
請執行下列步驟來取下 StorSimple 裝置上的底座。

#### <a name="to-remove-a-chassis"></a>若要取下底座
1. 確認 StorSimple 裝置已關閉，並與所有電源中斷連接。
2. 取下所有網路和 SAS 纜線 (如果適當的話)。
3. 取下機架中的裝置。
4. 取下每個磁碟機，並記下從哪些插槽中取下它們。 如需詳細資訊，請參閱 [取下磁碟機](storsimple-8000-disk-drive-replacement.md#remove-the-disk-drive)。
5. 在 EBOD 機箱 (如果這是故障的底座) 上，取下 EBOD 控制器模組。 如需詳細資訊，請參閱 [取下 EBOD 控制器](storsimple-8000-ebod-controller-replacement.md#remove-an-ebod-controller)。
   
    在主要機箱 (如果這是故障的底座) 上，取下控制器，並記下從哪些插槽中取下它們。 如需詳細資訊，請參閱 [取下控制器](storsimple-8000-controller-replacement.md#remove-a-controller)。

## <a name="install-the-chassis"></a>安裝底座
請執行下列步驟來安裝 StorSimple 裝置上的底座。

#### <a name="to-install-a-chassis"></a>若要安裝底座
1. 以機架掛接底座。 如需詳細資訊，請參閱[以機架掛接 StorSimple 8100 裝置](storsimple-8100-hardware-installation.md#rack-mount-your-storsimple-8100-device)或[以機架掛接 StorSimple 8600 裝置](storsimple-8600-hardware-installation.md#rack-mount-your-storsimple-8600-device)。
2. 在以機架掛接了底座之後，請將控制器模組安裝在先前安裝它們的同一位置中。
3. 將磁碟機安裝在先前安裝它們的同一位置和插槽中。
   
   > [!NOTE]
   > 我們建議您先將 SSD 安裝於插槽中，然後再安裝 HDD。
  
4. 在以機架掛接裝置，並安裝元件之後，請將裝置連接至適當的電源，並開啟裝置。 如需詳細資料，請參閱[將您的 StorSimple 8100 裝置接上纜線](storsimple-8100-hardware-installation.md#cable-your-storsimple-8100-device)或[將您的 StorSimple 8600 裝置接上纜線](storsimple-8600-hardware-installation.md#cable-your-storsimple-8600-device)。

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

