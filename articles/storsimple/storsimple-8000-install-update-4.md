---
title: 在 StorSimple 8000 系列裝置安裝 Update 4 | Microsoft Docs
description: 說明如何在 StorSimple 8000 系列裝置上安裝 StorSimple 8000 系列 Update 4。
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
ms.date: 08/02/2017
ms.author: alkohli
ms.openlocfilehash: 57d6d63c55f8ad4da5d1905a1e209da454b0491c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38630189"
---
# <a name="install-update-4-on-your-storsimple-device"></a>在 StorSimple 裝置上安裝 Update 4

## <a name="overview"></a>概觀

本教學課程說明如何透過 Azure 入口網站及使用 Hotfix 方法，在執行舊軟體版本的 StorSimple 裝置上安裝 Update 4。 當閘道器是設定於 StorSimple 裝置之 DATA 0 以外的網路介面上，且您正嘗試從 Update 1 以前的軟體版本更新時，就會使用 Hotfix 方法。

Update 4 包含裝置軟體、USM 韌體、LSI 驅動程式與韌體、Storport 和 Spaceport、OS 安全性更新和許多其他作業系統更新。  裝置的軟體、USM 韌體、Spaceport、Storport 和其他 OS 更新為非干擾性更新。 您可以透過 Azure 入口網站或透過 Hotfix 方法，套用非干擾性或定期更新。 磁碟韌體更新為干擾性更新，只能使用裝置的 Windows PowerShell 介面透過 Hotfix 方法套用。

> [!IMPORTANT]
> * 安裝前會執行一組手動和自動預先檢查，以根據硬體狀態和網路連線來判斷裝置健全狀況。 這些預先檢查只會在您從 Azure 入口網站套用更新時執行。
> * 建議您透過 Azure 入口網站安裝軟體和其他定期更新。 如果入口網站中的更新前閘道器檢查失敗，請移至裝置的 Windows PowerShell 介面安裝更新 (勿透過其他方式)。 視您從哪一個版本更新而定，可能需要 4 小時 (或以上) 來安裝更新。 維護模式更新也必須透過裝置的 Windows PowerShell 介面安裝。 由於維護模式更新是干擾性更新，它們將會導致裝置的停機時間。
> * 如果執行選擇性的 StorSimple Snapshot Manager，更新裝置之前，請先將您的 Snapshot Manager 版本升級至 Update 4。


[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-4-via-the-azure-portal"></a>透過 Azure 入口網站安裝 Update 4
請執行下列步驟來將您的裝置更新至 [Update 4](storsimple-update4-release-notes.md)。

> [!NOTE]
> Microsoft 會從裝置提取其他診斷資訊。 因此，當我們的作業小組識別有問題的裝置時，我們更有能力從裝置收集資訊並診斷問題。 

[!INCLUDE [storsimple-8000-install-update4-via-portal](../../includes/storsimple-8000-install-update4-via-portal.md)]

確認您的裝置是否正在執行 **StorSimple 8000 系列 Update 4 (6.3.9600.17820)**。 [ **上次更新日期** ] 應該也已修改。

* 您會看到有可用的維護模式更新 (此訊息可能會在您安裝更新之後繼續顯示長達 24 小時)。 維護模式更新為干擾性更新，會導致裝置產生停機時間，且只能透過您裝置的 Windows PowerShell 介面加以套用。

* 下載維護模式更新，方法是使用[下載 Hotfix](#to-download-hotfixes) 中列出的步驟來搜尋和下載 KB4011837，它會安裝磁碟韌體更新 (此時應該已經安裝其他更新)。 請遵循 [安裝及驗證維護模式 Hotfix](#to-install-and-verify-maintenance-mode-hotfixes) 中列出的步驟安裝維護模式更新。

## <a name="install-update-4-as-a-hotfix"></a>以 Hotfix 方式安裝 Update 4
建議的方法是透過 Azure 入口網站安裝 Update 4。

請在嘗試透過 Azure 入口網站安裝更新，而無法通過閘道器檢查時執行此程序。 當您指派閘道器給非 DATA 0 網路介面且您的裝置正在執行早於 Update 1 的軟體版本時，檢查才會失敗。

可以使用 Hotfix 方法升級的軟體版本為：

* Update 0.1、0.2、0.3
* Update 1、1.1、1.2
* Update 2、2.1、2.2
* Update 3、3.1


Hotfix 方法涉及下列三個步驟：

1. 從 Microsoft Update Catalog 下載 Hotfix。
2. 安裝及驗證定期模式 Hotfix。
3. 安裝及驗證維護模式 Hotfix。

#### <a name="download-updates-for-your-device"></a>下載適用於您裝置的更新

您必須以指定的順序和建議的資料夾下載並安裝下列 Hotfix：

| 順序 | KB | 說明 | 更新類型 | 安裝時間 |安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4011839 |軟體更新 |Regular <br></br>非干擾性 |~ 25 分鐘 |FirstOrderUpdate|
| 2A. |KB4011841 <br> KB4011842 |LSI 驅動程式與韌體更新 <br> USM 韌體更新 (3.38 版) |Regular <br></br>非干擾性 |~ 3 小時 <br> (包括 2A. 2B. + 2C.)|SecondOrderUpdate|
| 2B. |KB3139398、KB3108381 <br> KB3205400、KB3142030 <br> KB3197873、KB3197873 <br> KB3192392、KB3153704 <br> KB3174644、KB3139914  |OS 安全性更新套件 <br> 下載 Windows Server 2012 R2 |Regular <br></br>非干擾性 |- |SecondOrderUpdate|
| 2C. |KB3210083、KB3103616 <br> KB3146621、KB3121261 <br> KB3123538 |OS 更新套件 <br> 下載 Windows Server 2012 R2 |Regular <br></br>非干擾性 |- |SecondOrderUpdate|

除了前面表格所示的所有更新之外，您可能還需要安裝磁碟韌體更新。 您可以執行 `Get-HcsFirmwareVersion` Cmdlet 來確認是否需要進行磁碟韌體更新。 如果您執行的是這些韌體版本：`XMGJ`、`XGEG`、`KZ50`、`F6C2`、`VR08`、`N002`、`0106`，您就不需要安裝這些更新。

| 順序 | KB | 說明 | 更新類型 | 安裝時間 | 安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 3. |KB3121899 |磁碟韌體 |維護  <br></br>干擾性 |~ 30 分鐘 | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * 此程序僅需要執行一次，即可套用 Update 4。 您可以使用 Azure 入口網站套用後續的更新。
> * 如果您是從 Update 3 或 3.1 進行更新，則總安裝時間會接近 4 小時。
> * 使用此程序套用更新之前，請確定兩個裝置控制器都在線上，而且所有硬體元件的狀況良好。

請執行以下步驟來下載及安裝 Hotfix。

[!INCLUDE [storsimple-install-update4-hotfix](../../includes/storsimple-install-update4-hotfix.md)]

[!INCLUDE [storsimple-install-troubleshooting](../../includes/storsimple-install-troubleshooting.md)]

## <a name="next-steps"></a>後續步驟
深入了解 [Update 4 版本](storsimple-update4-release-notes.md)。

