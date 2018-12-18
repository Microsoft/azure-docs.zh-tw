---
title: 在 StorSimple 8000 系列裝置安裝 Update 5 | Microsoft Docs
description: 說明如何在 StorSimple 8000 系列裝置上安裝 StorSimple 8000 系列 Update 5。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/13/2017
ms.author: alkohli
ms.openlocfilehash: d6e17c7609fd41b8f4457edda373f6882a1a9d2b
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38698705"
---
# <a name="install-update-5-on-your-storsimple-device"></a>在 StorSimple 裝置上安裝 Update 5

## <a name="overview"></a>概觀

本教學課程說明如何透過 Azure 入口網站及使用 Hotfix 方法，在執行舊軟體版本的 StorSimple 裝置上安裝 Update 5。 當您嘗試在執行 Update 3 之前版本的裝置上安裝 Update 5 時，會使用 Hotfix 方法。 當閘道器是在 StorSimple 裝置之 DATA 0 以外的網路介面上設定的，且您要嘗試從 Update 1 之前的軟體版本更新時，也會使用 Hotfix 方法。

Update 5 包含裝置軟體、Storport 和 Spaceport、OS 安全性更新和 OS 更新，以及磁碟韌體更新。  裝置軟體、Spaceport、Storport、安全性和其他 OS 更新皆為非干擾性更新。 您可以透過 Azure 入口網站或透過 Hotfix 方法，套用非干擾性或定期更新。 磁碟韌體更新為干擾性更新，會在裝置進入維護模式時使用裝置的 Windows PowerShell 介面透過 Hotfix 方法來套用。

> [!IMPORTANT]
> * Update 5 是強制性更新，應立即安裝。 如需詳細資訊，請參閱 [Update 5 版本資訊](storsimple-update5-release-notes.md)。
> * 安裝前會執行一組手動和自動預先檢查，以根據硬體狀態和網路連線來判斷裝置健全狀況。 這些預先檢查只會在您從 Azure 入口網站套用更新時執行。
> * 強烈建議您在更新執行 Update 3 之前版本的裝置時，使用 Hotfix 方法安裝更新。 如果您遇到任何問題，請[記錄支援票證](storsimple-8000-contact-microsoft-support.md)。
> * 建議您透過 Azure 入口網站安裝軟體和其他定期更新。 如果入口網站中的更新前閘道器檢查失敗，請移至裝置的 Windows PowerShell 介面安裝更新 (勿透過其他方式)。 視您從哪一個版本更新而定，可能需要 4 小時 (或以上) 來安裝更新。 維護模式更新必須透過裝置的 Windows PowerShell 介面安裝。 由於維護模式更新是干擾性更新，它們會導致裝置的停機時間。
> * 如果執行選擇性的 StorSimple Snapshot Manager，更新裝置之前，請先將您的 Snapshot Manager 版本升級至 Update 5。


[!INCLUDE [storsimple-preparing-for-update](../../includes/storsimple-preparing-for-updates.md)]

## <a name="install-update-5-via-the-azure-portal"></a>透過 Azure 入口網站安裝 Update 5
請執行下列步驟來將您的裝置更新至 [Update 5](storsimple-update5-release-notes.md)。

> [!NOTE]
> Microsoft 會從裝置提取其他診斷資訊。 因此，當我們的作業小組識別有問題的裝置時，我們更有能力從裝置收集資訊並診斷問題。

[!INCLUDE [storsimple-8000-install-update4-via-portal](../../includes/storsimple-8000-install-update5-via-portal.md)]

確認您的裝置是否正在執行 **StorSimple 8000 系列 Update 5 (6.3.9600.17845)**。 [上次更新日期] 應該已修改。

您會看到有可用的維護模式更新 (此訊息可能會在您安裝更新之後繼續顯示長達 24 小時)。 下一節會詳細說明安裝維護模式更新的步驟。

[!INCLUDE [storsimple-8000-install-maintenance-mode-updates](../../includes/storsimple-8000-install-maintenance-mode-updates.md)]

## <a name="install-update-5-as-a-hotfix"></a>以 Hotfix 方式安裝 Update 5

可以使用 Hotfix 方法升級的軟體版本為：

* Update 0.1、0.2、0.3
* Update 1、1.1、1.2
* Update 2、2.1、2.2
* Update 3、3.1
* Update 4

> [!NOTE] 
> 嘗試從 Update 3 及更新版本更新時，安裝 Update 5 的建議方法是透過 Azure 入口網站進行。 更新執行 Update 3 之前版本的裝置時，請使用此程序。 您也可以在嘗試透過 Azure 入口網站安裝更新，而無法通過閘道器檢查時使用此程序。 當您指派閘道器給非 DATA 0 網路介面且您的裝置正在執行早於 Update 1 的軟體版本時，檢查會失敗。

Hotfix 方法涉及下列三個步驟：

1. 從 Microsoft Update Catalog 下載 Hotfix。
2. 安裝及驗證定期模式 Hotfix。
3. 安裝及驗證維護模式 Hotfix。

#### <a name="download-updates-for-your-device"></a>下載適用於您裝置的更新

您必須以指定的順序和建議的資料夾下載並安裝下列 Hotfix：

| 順序 | KB | 說明 | 更新類型 | 安裝時間 |安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 1. |KB4037264 |軟體更新<br> 下載 _HcsSoftwareUpdate.exe_ 和 _CisMSDAgent.exe_ |Regular <br></br>非干擾性 |~ 25 分鐘 |FirstOrderUpdate|

如果從執行 Update 4 的裝置更新，您只需要將作業系統累積更新當作第二順位的更新來進行安裝。

| 順序 | KB | 說明 | 更新類型 | 安裝時間 |安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 2A. |KB4025336 |OS 累積更新套件 <br> 下載 Windows Server 2012 R2 版本 |Regular <br></br>非干擾性 |- |SecondOrderUpdate|

如果從執行 Update 3 或更早版本的裝置進行安裝，除了累積更新，也請安裝下列項目。

| 順序 | KB | 說明 | 更新類型 | 安裝時間 |安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 2B. |KB4011841 <br> KB4011842 |LSI 驅動程式與韌體更新 <br> USM 韌體更新 (3.38 版) |Regular <br></br>非干擾性 |~ 3 小時 <br> (包括 2A. 2B. + 2C.)|SecondOrderUpdate|
| 2C. |KB3139398 <br> KB3142030 <br> KB3108381 <br> KB3153704 <br> KB3174644 <br> KB3139914   |OS 安全性更新套件 <br> 下載 Windows Server 2012 R2 版本 |Regular <br></br>非干擾性 |- |SecondOrderUpdate|
| 2D. |KB3146621 <br> KB3103616 <br> KB3121261 <br> KB3123538 |OS 更新套件 <br> 下載 Windows Server 2012 R2 版本 |Regular <br></br>非干擾性 |- |SecondOrderUpdate|


除了前面表格所示的所有更新之外，您可能還需要安裝磁碟韌體更新。 您可以執行 `Get-HcsFirmwareVersion` Cmdlet 來確認是否需要進行磁碟韌體更新。 如果您執行的是這些韌體版本：`XMGJ`、`XGEG`、`KZ50`、`F6C2`、`VR08`、`N003`、`0107`，您就不需要安裝這些更新。

| 順序 | KB | 說明 | 更新類型 | 安裝時間 | 安裝在資料夾|
| --- | --- | --- | --- | --- | --- |
| 3. |KB4037263 |磁碟韌體 |維護  <br></br>干擾性 |~ 30 分鐘 | ThirdOrderUpdate |

<br></br>

> [!IMPORTANT]
> * 如果您是從 Update 4 進行更新，則總安裝時間會接近 4 小時。
> * 使用此程序套用更新之前，請確定兩個裝置控制器都在線上，而且所有硬體元件的狀況良好。

請執行以下步驟來下載及安裝 Hotfix。

[!INCLUDE [storsimple-install-update5-hotfix](../../includes/storsimple-install-update5-hotfix.md)]

[!INCLUDE [storsimple-8000-install-troubleshooting](../../includes/storsimple-8000-install-troubleshooting.md)]

## <a name="next-steps"></a>後續步驟
深入了解 [Update 5 版本](storsimple-update5-release-notes.md)。

