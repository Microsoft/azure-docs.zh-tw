---
title: 檢視和管理 StorSimple Virtual Array 作業 | Microsoft Docs
description: 說明 StorSimple 裝置管理員服務的「作業」頁面，以及如何使用該頁面來追蹤 StorSimple Virtual Array 的最近和目前作業。
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 31879821-b599-4609-a7f4-d4b0f658a933
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: na
ms.date: 11/11/2016
ms.author: alkohli
ms.openlocfilehash: 3fd1c262a8ce94d8e98f2b066a8028d974b15b1d
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/05/2018
ms.locfileid: "23109676"
---
# <a name="use-the-storsimple-device-manager-service-to-view-jobs-for-the-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員服務來檢視 StorSimple Virtual Array 的作業
## <a name="overview"></a>概觀
[作業] 刀鋒視窗提供單一中央入口網站，可檢視和管理已連接至 StorSimple 裝置管理員服務的虛擬陣列上啟動的作業。 您可以針對多個虛擬裝置，檢視執行中、完成和失敗的作業。 結果會以表格式格式呈現。

![[作業] 刀鋒視窗](./media/storsimple-virtual-array-manage-jobs/ova-jobs-blade.png)

您可以透過篩選欄位，快速找到您所感興趣的工作，例如：

* **時間範圍** – 您可以根據日期和時間範圍來篩選作業。
* **裝置** – 作業會在連線到服務的特定裝置上進行初始化。 接著會根據下列屬性，將篩選的作業列表顯示：
  
  * **類型** – 作業名稱可以是 [全部]、[備份]、[還原]、[容錯移轉]、[下載更新]，或 [安裝更新]。
  * **狀態** – 作業可以是 [全部]、[進行中]、[成功]、[失敗]或 [已取消]。
  * **實體** – 作業可以與磁碟區、共用或裝置相關聯。
  * **裝置** – 用來啟動工作之裝置的名稱。
  * **啟動於** – 啟動工作的時間。
  * **持續時間** – 作業執行的持續時間。
* **狀態** – 您可以搜尋全部、執行中、完成或已取消的作業。
* **作業類型** – 作業類型可以是全部、備份、還原、容錯移轉、下載更新或安裝更新。

工作清單每隔 30 秒會重新整理。

## <a name="view-job-details"></a>檢視工作詳細資料
執行下列步驟來檢視任何工作的詳細資料。

#### <a name="to-view-job-details"></a>若要檢視工作詳細資料
1. 在 [作業] 刀鋒視窗上，搭配適當的篩選執行查詢，以顯示您感興趣的作業。 您可以搜尋完成或執行中作業。
2. 從作業表格式清單中選取作業。
   
    ![[作業] 刀鋒視窗](./media/storsimple-virtual-array-manage-jobs/ova-jobs-blade.png)
3. 按一下頁面底部的 [詳細資料] 。
4. 在 [詳細資料]  對話方塊中，您可以檢視狀態、詳細資料和時間統計資料。 下圖顯示 [備份作業詳細資料]  對話方塊的範例。
   
    ![工作詳細資料](./media/storsimple-virtual-array-manage-jobs/ova-jobs-details.png)

#### <a name="job-failures-when-the-virtual-machine-is-paused-in-the-hypervisor"></a>虛擬機器在 hypervisor 中暫停時，作業失敗
當 StorSimple Virtual Array 上正在執行作業且裝置 (Hypervisor 中佈建的虛擬機器) 暫停超過 15 分鐘時，作業會失敗。 這是因為您的 StorSimple Virtual Array 時間未與 Microsoft Azure 時間同步處理。 

您會看到下列錯誤：「您的裝置時間與 Microsoft Azure 不同步超過 15 分鐘。 請確定 Hypervisor 和裝置時間與 NTP 伺服器同步。 請確定沒有連線問題。 若要針對連線問題進行疑難排解，請從虛擬裝置的本機 Web UI 執行診斷測試。」

備份、還原、更新和容錯移轉作業都可能發生這些失敗。 如果您的虛擬機器佈建到 Hyper-V，電腦與您的 Hypervisor 時間最終會同步。 一旦發生這種情況，您可以重新啟動您的作業。

## <a name="next-steps"></a>後續步驟
了解如何[使用本機 Web UI 來管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

