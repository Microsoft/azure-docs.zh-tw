---
title: 更換 StorSimple 8600 EBOD 控制器 | Microsoft Docs
description: 說明如何取下並更換 StorSimple 8600 裝置上的一個或兩個 EBOD 控制器。
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
ms.openlocfilehash: 45699c267d1009c4884dd164fd3f2950d6d5f555
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108216"
---
# <a name="replace-an-ebod-controller-on-your-storsimple-device"></a>更換 StorSimple 裝置上的 EBOD 控制器

## <a name="overview"></a>概觀
本教學課程說明如何更換 Microsoft Azure StorSimple 裝置上故障的 EBOD 控制器模組。 若要更換 EBOD 控制器模組，您必須：

* 取下故障的 EBOD 控制器
* 安裝新的 EBOD 控制器

開始之前，請考量下列資訊：

* 空白的 EBOD 模組必須插入至所有未使用的插槽。 如果插槽處於開啟狀態，機箱將無法適當地冷卻。
* EBOD 控制器是可熱交換，而且可以取下或更換。 請勿取下故障的模組，除非您有更換模組。 當起始更換程序時，您必須在 10 分鐘內完成。

> [!IMPORTANT]
> 在嘗試取下或更換任何 StorSimple 元件之前，請確定先閱讀[安全圖示慣例](storsimple-safety.md#safety-icon-conventions)和其他[安全性預防措施](storsimple-safety.md)。

## <a name="remove-an-ebod-controller"></a>取下 EBOD 控制器
在取下 StorSimple 裝置中故障的 EBOD 控制器模組之前，請確定另一個 EBOD 控制器模組作用中且執行中。 下列程序和資料表說明如何取下 EBOD 控制器模組。

#### <a name="to-remove-an-ebod-module"></a>若要取下 EBOD 模組
1. 開啟 Azure 入口網站。
2. 移至您的裝置，並瀏覽至 [設定] > [硬體健康狀態]，並確認作用中 EBOD 控制器模組的 LED 狀態為綠色，而故障的 EBOD 控制器模組的 LED 為紅色。
3. 在裝置背面找出故障的 EBOD 控制器模組。
4. 先取下將 EBOD 控制器模組連接到控制器的纜線，再從系統取出 EBOD 模組。
5. 記下已連接至控制器之 EBOD 控制器模組的確切 SAS 連接埠。 在更換 EBOD 模組之後，您必須將系統還原至這個組態。
   
   > [!NOTE]
   > 通常，這將是連接埠 A，在下圖標示為 [主機輸入]。
   
    ![EBOD 控制器的後擋板](./media/storsimple-ebod-controller-replacement/IC741049.png)
   
     **圖 1** EBOD 模組的背面
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |故障 LED |
   | 2 |電源 LED |
   | 3 |SAS 連接器 |
   | 4 |SAS LED |
   | 5 |僅供原廠使用的序列埠 |
   | 6 |連接埠 A (主機輸入) |
   | 7 |連接埠 B (主機輸出) |
   | 8 |連接埠 C (僅限原廠使用) |

## <a name="install-a-new-ebod-controller"></a>安裝新的 EBOD 控制器
下列程序和資料表說明如何在 StorSimple 中安裝 EBOD 控制器模組。

#### <a name="to-install-an-ebod-controller"></a>若要安裝新的 EBOD 控制器
1. 請檢查 EBOD 是否損毀，尤其是介面連接器。 如果有任何接腳彎曲，請勿安裝新的 EBOD 控制器。
2. 當閂鎖在開啟的位置時，將模組滑入機箱，直到閂鎖扣上。
   
    ![安裝 EBOD 控制器](./media/storsimple-ebod-controller-replacement/IC741050.png)
   
    **圖 2** 安裝 EBOD 控制器模組
3. 關閉閂鎖。 您應該會聽到喀嚓一聲，表示閂鎖已扣上。
   
    ![鬆開 EBOD 閂鎖](./media/storsimple-ebod-controller-replacement/IC741047.png)
   
    **圖 3** 關閉 EBOD 模組閂鎖
4. 重新連接纜線。 使用更換之前存在的確切組態。 請參閱下圖和資料表，以取得有關如何連接纜線的詳細資料。
   
    ![將您的 4U 裝置接上纜線，以取得電源](./media/storsimple-ebod-controller-replacement/IC770723.png)
   
    **圖 4**。 重新連接纜線
   
   | 標籤 | 說明 |
   |:--- |:--- |
   | 1 |主要機箱 |
   | 2 |PCM 0 |
   | 3 |PCM 1 |
   | 4 |控制器 0 |
   | 5 |控制器 1 |
   | 6 |EBOD 控制器 0 |
   | 7 |EBOD 控制器 1 |
   | 8 |EBOD 機箱 |
   | 9 |電源分配單元 |

## <a name="next-steps"></a>後續步驟
深入了解 [StorSimple 硬體元件更換](storsimple-8000-hardware-component-replacement.md)。

