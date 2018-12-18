---
title: 管理 StorSimple 8000 系列備份原則 | Microsoft Docs
description: 說明如何使用 StorSimple 裝置管理員服務在 StorSimple 8000 裝置上建立並管理手動備份、備份排程與備份保留。
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
ms.date: 07/05/2017
ms.author: alkohli
ms.openlocfilehash: 569dbfdeb7dcd526cb5a54b487ea1bfb59b13cc6
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38681575"
---
# <a name="use-the-storsimple-device-manager-service-in-azure-portal-to-manage-backup-policies"></a>在 Azure 入口網站中，使用 StorSimple 裝置管理員服務來管理備份原則


## <a name="overview"></a>概觀

本教學課程說明如何使用 StorSimple 裝置管理員服務的 [備份原則] 刀鋒視窗來控制 StorSimple 磁碟區的備份程序和備份保留。 它也會說明如何完成手動備份。

在您備份磁碟區時，您可以選擇建立本機快照或雲端快照。 如果您正在備份固定在本機的磁碟區，我們建議您指定雲端快照。 拍下大量的固定在本機的磁碟區的本機快照，再加上具大量變換的資料集，將會導致本機空間很快就不足的情況。 如果您選擇建立本機快照，我們建議您拍下較少量的每日快照來備份最新的狀態，加以保留一天後再予以刪除。

針對固定在本機的磁碟區，在拍下雲端快照時，您僅會將變更的資料複製到雲端，並在其中將重複項目刪除及壓縮快照。

## <a name="the-backup-policy-blade"></a>[備份原則] 刀鋒視窗

您的 StorSimple 裝置的 [備份原則] 刀鋒視窗可讓您管理備份原則並排程本機和雲端快照。 備份原則用來設定磁碟區集合的備份排程和備份保留。 備份原則可讓您同時建立多個磁碟區的快照。 這表示備份原則所建立的備份將會是與當機時一致的複本。

備份原則表格式清單也可讓您依下列一個或多個欄位，篩選現有的備份原則：

* **原則名稱** – 與原則相關聯的名稱。 不同類型的原則包括：

  * 已排程的原則 (由使用者明確建立)。
  * 匯入的原則 (原先是在 StorSimple Snapshot Manager 中所建立)。 這些包含說明從中匯入原則之 StorSimple Snapshot Manager 主機的標記。

  > [!NOTE]
  > 建立磁碟區時，不會再啟用自動或預設備份原則。

* **上一次成功的備份** – 使用此原則所進行的上一次成功備份的日期和時間。

* **下一次備份** – 此原則將起始的下一次排定的備份的日期和時間。

* **磁碟區** – 與原則相關聯的磁碟區。 建立備份時，會將與備份原則相關聯的所有磁碟區群組在一起。

* **排程** – 與備份原則相關聯的排程數目。

您可以執行的備份原則常用作業包括：

* 新增備份原則
* 新增或修改排程
* 新增或移除磁碟區
* 刪除備份原則
* 進行手動備份

## <a name="add-a-backup-policy"></a>新增備份原則

新增備份原則，以自動排程備份。 當您第一次建立磁碟區時，沒有與您的磁碟區相關聯的預設備份原則。 您必須新增並指派備份原則以保護磁碟區資料。

在 Azure 入口網站中執行下列步驟，以便為 StorSimple 裝置新增備份原則。 新增原則之後，您可以定義排程 (請參閱 [新增或修改排程](#add-or-modify-a-schedule))。

[!INCLUDE [storsimple-8000-add-backup-policy-u2](../../includes/storsimple-8000-add-backup-policy-u2.md)]

## <a name="add-or-modify-a-schedule"></a>新增或修改排程

您可以在 StorSimple 裝置上新增或修改附加到現有備份原則的排程。 在 Azure 入口網站中執行下列步驟，以新增或修改排程。

[!INCLUDE [storsimple-8000-add-modify-backup-schedule](../../includes/storsimple-8000-add-modify-backup-schedule-u2.md)]


## <a name="add-or-remove-a-volume"></a>新增或移除磁碟區

您可以在 StorSimple 裝置上新增或移除指派至備份原則的磁碟區。 在 Azure 入口網站中執行下列步驟，以新增或移除磁碟區。

[!INCLUDE [storsimple-8000-add-volume-backup-policy-u2](../../includes/storsimple-8000-add-remove-volume-backup-policy-u2.md)]


## <a name="delete-a-backup-policy"></a>刪除備份原則

在 Azure 入口網站中執行下列步驟，以便刪除 StorSimple 裝置上的備份原則。

[!INCLUDE [storsimple-8000-delete-backup-policy](../../includes/storsimple-8000-delete-backup-policy.md)]

## <a name="take-a-manual-backup"></a>進行手動備份

在 Azure 入口網站中執行下列步驟，以針對單一磁碟區建立隨選 (手動) 備份。

[!INCLUDE [storsimple-8000-create-manual-backup](../../includes/storsimple-8000-create-manual-backup.md)]

## <a name="next-steps"></a>後續步驟

深入了解[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。

