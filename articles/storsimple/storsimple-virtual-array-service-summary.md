---
title: StorSimple Virtual Array 服務摘要刀鋒視窗 | Microsoft Docs
description: 描述 StorSimple 裝置管理員的服務摘要刀鋒視窗，並說明如何使用它來監視 StorSimple Virtual Array 的健康狀態。
services: storsimple
documentationcenter: ''
author: manuaery
manager: syadav
editor: ''
ms.assetid: 8a2b9a84-b0e6-48b9-b366-d16f004241a5
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/21/2016
ms.author: manuaery
ms.openlocfilehash: 284e404c44505a98d9e0ed5abe87cd945415af56
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109736"
---
# <a name="use-the-service-summary-blade-for-storsimple-device-manager-connected-to-storsimple-virtual-array"></a>在連接至 Microsoft Azure StorSimple Virtual Array 的 StorSimple 裝置管理員中使用服務摘要刀鋒視窗
## <a name="overview"></a>概觀
針對連接至服務的 StorSimple Virtual Array (也稱為 StorSimple 內部部署虛擬裝置，或簡稱虛擬裝置)，StorSimple 裝置管理員的服務摘要刀鋒視窗提供摘要檢視，並醒目提示需要系統管理員注意的裝置。 本教學課程介紹服務摘要刀鋒視窗、說明內容和功能，並描述您可以從這個刀鋒視窗執行的工作。

![服務儀表板](./media/storsimple-virtual-array-service-summary/service-blade.png)

## <a name="management-commands-and-essentials"></a>管理命令和基本資訊
在 StorSimple 摘要刀鋒視窗中，您會看到選項可用來管理 StorSimple 裝置管理員服務，以及已向此服務註冊的虛擬陣列。 您在刀鋒視窗的上方橫幅和左邊會看到管理命令。

請使用這些選項來執行各種作業，例如新增共用或磁碟區，或監視虛擬陣列上執行的各種作業。

基本資訊區域擷取一些重要屬性，例如原本建立 StorSimple 裝置管理員的資源群組、位置和訂用帳戶。

## <a name="storsimple-device-manager-service-summary"></a>StorSimple 裝置管理員服務摘要
* [警示] 圖格提供所有虛擬裝置上的所有作用中警示的快照集，依警示嚴重性分組。 按一下此圖格來開啟 [警示] 刀鋒視窗，您可以在其中按一下個別警示來檢視該警示的其他詳細資料，包括任何建議的動作。 如果已解決問題，您也可以清除警示。
* [容量] 圖格顯示所有虛擬裝置上相對於可用的總儲存空間而言，已佈建和剩餘的主要儲存空間。 [已佈建] 是指已備妥和配置供使用的儲存空間數量，[剩餘] 是指所有虛擬裝置上可佈建的剩餘容量。 [剩餘階層式] 容量是可供佈建的容量，包括雲端，而 [剩餘本機] 是連接至虛擬陣列的磁碟上剩餘的容量。
* 在 [使用量] 圖表中，您可以查看虛擬裝置的相關度量。 您可以檢視所有虛擬裝置已使用的主要儲存空間，以及虛擬裝置在過去 7 天 (預設期間) 耗用的雲端儲存空間。 使用圖表右上角的 [編輯] 選項來選擇不同的時間範圍。
* [裝置] 圖格提供 StorSimple 裝置管理員中的虛擬陣列數目摘要 (依裝置狀態分組)。 按一下此圖格以開啟 [裝置] 清單刀鋒視窗，然後按一下個別裝置來深入該裝置的裝置摘要。 您也可以從指定的裝置摘要刀鋒視窗執行裝置的特定動作。 如需裝置摘要刀鋒視窗的詳細資訊，請參閱[裝置摘要刀鋒視窗](storsimple-virtual-array-device-summary.md)。

## <a name="view-the-activity-logs"></a>檢視活動記錄檔
若要檢視 StorSimple 裝置管理員中執行的各種作業，請按一下 StorSimple 服務摘要刀鋒視窗左側的 [活動記錄檔] 連結。 這會帶您前往 [活動記錄檔] 刀鋒視窗，您可以在其中看到最近執行的作業摘要。

![活動記錄檔](./media/storsimple-virtual-array-service-summary/activity-log.png)

## <a name="next-steps"></a>後續步驟
了解如何 [使用本機 Web UI 來管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

