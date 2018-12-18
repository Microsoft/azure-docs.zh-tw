---
title: StorSimple 8000 系列 Update 5 版本資訊 | Microsoft Docs
description: 說明 StorSimple 8000 系列 Update 5 的新功能、問題及因應措施。
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 11/13/2017
ms.author: alkohli
ms.openlocfilehash: 672757e82bcf645b705f46a9975e09c9dc5eef92
ms.sourcegitcommit: 659cc0ace5d3b996e7e8608cfa4991dcac3ea129
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/13/2017
ms.locfileid: "24518914"
---
# <a name="storsimple-8000-series-update-5-release-notes"></a>StorSimple 8000 系列 Update 5 版本資訊

## <a name="overview"></a>概觀

下列版本資訊說明 StorSimple 8000 系列 Update 5 的新功能，並識別未決的重要問題。 當中也包含此版本中隨附之 StorSimple 軟體更新的清單。

Update 5 可以套用至任何執行 Update 0.1 到 Update 4 的 StorSimple 裝置。 與 Update 5 關聯的裝置版本為 6.3.9600.17845。

在 StorSimple 方案中部署更新之前，請檢閱版本資訊中所包含的資訊。

> [!IMPORTANT]
> * Update 5 是強制性更新，且必須立即安裝。 如需詳細資訊，請參閱如何[套用 Update 5](storsimple-8000-install-update-5.md)。
> * Update 5 有裝置軟體、磁碟韌體、作業系統安全性和其他作業系統更新。 安裝此更新，大約需要花費 4 小時。 磁碟韌體更新是干擾性更新，會導致您的裝置停機。 建議您套用 Update 5，讓您的裝置維持在最新狀態。
> * 如果是新版本，您可能不會立即看到更新，因為我們會分階段推出更新。 請等候數天然後再次掃描更新，因為很快就會提供這些更新。

## <a name="whats-new-in-update-5"></a>Update 5 的新功能

Update 5 包含以下重要的改良功能和錯誤修正。

* **使用 Azure Active Directory (AAD) 來向 StorSimple 裝置管理員服務驗證** – 從 Update 5 起，系統會使用 Azure Active Directory 來向 StorSimple 裝置管理員服務驗證。 舊的驗證機制會在 2017 年 12 月後淘汰。 所有使用者都必須在其防火牆規則中包含新的驗證 URL。 如需詳細資訊，請移至 [StorSimple 裝置網路需求中所列出的驗證 URL](storsimple-8000-system-requirements.md#url-patterns-for-azure-portal)。

    如果未將驗證 URL 包含在防火牆規則內，使用者會看到重大警示，其內容指出他們的 StorSimple 裝置無法向服務驗證。 如果使用者看到此警示，就必須包含新的驗證 URL。 如需詳細資訊，請移至 [StorSimple 網路警示](storsimple-8000-manage-alerts.md#networking-alerts)。

* **新版的 StorSimple Snapshot Manager** - 新版的 StorSimple Snapshot Manager 發行隨附於 Update 5，且與所有執行 Update 4 或更新版本的 StorSimple 裝置相容。 我們建議您更新成這個版本。 舊版的 StorSimple Snapshot Manager 是用於執行 Update 3 或更舊版本的 StorSimple 裝置。 [下載適當版本的 StorSimple Snapshot Manager](https://www.microsoft.com/en-us/download/details.aspx?id=44220)，並參考[部署 StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md)。


## <a name="issues-fixed-in-update-5"></a>Update 5 中修正的問題

下表提供 Update 5 中已修正之問題的摘要。

| 否 | 功能 | 問題 | 適用於實體裝置 | 適用於虛擬裝置 |
| --- | --- | --- | --- | --- |
| 1 |Windows PowerShell 遠端處理 |在舊版中，使用者會在嘗試透過 Windows PowerShell 對 StorSimple 雲端設備建立遠端連線時收到錯誤。 此版本已找出此問題的根本原因並加以修正。 |否 |yes |
| 2 |頻寬範本 |舊版中的頻寬範本有問題，從而導致頻寬比裝置所設定的值還低。 本版已經修正這個問題。 |yes |yes |
| 3 |容錯移轉 |在舊版中，當擁有大量磁碟區的裝置容錯移轉至另一個執行 Update 4 的裝置時，該程序會在嘗試套用存取控制記錄時失敗。 此版本已經修正這個問題。 |yes |yes |



## <a name="known-issues-in-update-5-from-previous-releases"></a>Update 5 中舊版的已知問題

Update 5 中沒有新的已知問題。 如需 Update 5 中舊版遺留的問題清單，請移至 [Update 3 版本資訊](storsimple-update3-release-notes.md#known-issues-in-update-3)。

## <a name="serial-attached-scsi-sas-controller-and-firmware-updates-in-update-5"></a>Update 5 中的序列連結 SCSI (SAS) 控制器和韌體更新

此版本包含 SAS 控制器和 LSI 驅動程式與韌體的更新。 如需有關如何安裝這些更新的詳細資訊，請參閱[在 StorSimple 裝置上安裝 Update 5](storsimple-8000-install-update-5.md) 。

## <a name="storsimple-cloud-appliance-updates-in-update-5"></a>Update 5 中的 StorSimple 雲端設備更新 | Microsoft Docs

這項更新無法套用至 StorSimple Cloud Appliance (也稱為虛擬裝置)。 必須使用 Update 5 映像建立新的雲端設備。 如需如何建立 StorSimple 雲端設備的資訊，請移至[部署和管理 StorSimple 雲端設備](storsimple-8000-cloud-appliance-u2.md)。

## <a name="next-step"></a>後續步驟

了解如何[在 StorSimple 裝置上安裝 Update 5](storsimple-8000-install-update-5.md)。

