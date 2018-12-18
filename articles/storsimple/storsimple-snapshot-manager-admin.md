---
title: StorSimple Snapshot Manager 管理 | Microsoft Docs
description: 提供有關 StorSimple Snapshot Manager 解決方案系統管理工作和工作流程的概觀與詳細資訊連結。
services: storsimple
documentationcenter: NA
author: SharS
manager: timlt
editor: ''
ms.assetid: 1cdbb61d-bd16-4be4-ade2-ceab11508acb
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/05/2016
ms.author: v-sharos
ms.openlocfilehash: a99b3d7336c3dc1a1f249915d6971a49f4b69be6
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109216"
---
# <a name="use-storsimple-snapshot-manager-to-administer-your-storsimple-solution"></a>使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案

## <a name="overview"></a>概觀
StorSimple Snapshot Manager 是 Microsoft Management Console (MMC) 嵌入式管理單元，可簡化資料保護和備份管理 Microsoft Azure StorSimple 環境中。 使用 StorSimple Snapshot Manager 時，您可以將資料中心和雲端中的 Microsoft Azure StorSimple 資料當作單一整合式儲存體解決方案來管理，因而簡化備份程序並降低成本。

StorSimple Snapshot Manager 中央管理主控台可讓您建立一致、 時間點備份複本的本機和雲端資料。 例如，您可以使用主控台：

* 設定、備份及刪除磁碟區。
* 設定磁碟區群組，以確保是應用程式一致的備份資料。
* 管理備份原則，以便在預先決定的排程上備份資料。
* 建立可以儲存在雲端，並用於災害復原的資料複本。

本文章提供教學課程的連結，描述 StorSimple Snapshot Manager 以及如何使用它來完成系統管理工作和工作流程。

* 如需有關 StorSimple Snapshot Manager 元件和架構的詳細資訊，請參閱 [什麼是 StorSimple Snapshot Manager？](storsimple-what-is-snapshot-manager.md) 
* 若要下載 StorSimple Snapshot Manager，請移至 [StorSimple Snapshot Manager 下載頁面](https://www.microsoft.com/download/details.aspx?id=44220)。
* 如需 StorSimple Snapshot Manager 部署程序，請移至 [部署 StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md)。

> [!NOTE]
> 您無法使用 StorSimple Snapshot Manager 來管理 Microsoft Azure StorSimple Virtual Arrays (也稱為 StorSimple 內部部署虛擬裝置)。


## <a name="storsimple-snapshot-manager-tasks-and-workflows"></a>StorSimple Snapshot Manager 工作和工作流程
您可以使用 StorSimple Snapshot Manager 來監視和管理目前的、已排程的和已完成的備份作業。 此外，StorSimple Snapshot Manager 提供最多 64 個已完成備份的目錄。 您可以使用目錄來尋找及還原磁碟區或個別檔案。 

| 如果您想要執行此動作... | 使用本教學課程... |
|:--- |:--- |
| 深入了解 StorSimple Snapshot Manager |[什麼是 StorSimple Snapshot Manager？ ](storsimple-what-is-snapshot-manager.md) |
| 安裝 StorSimple Snapshot Manager<br>重新安裝 StorSimple Snapshot Manager<br>移除 StorSimple Snapshot Manager |[部署 StorSimple Snapshot Manager](storsimple-snapshot-manager-deployment.md) |
| 使用 StorSimple Snapshot Manager 功能表和功能：<ul><li>功能表列</li><li>工具列</li><li>範圍窗格</li><li>結果窗格</li><li>動作窗格</li><li>鍵盤瀏覽和快速鍵</li></ul> |[StorSimple Snapshot Manager 使用者介面](storsimple-use-snapshot-manager.md) |
| 使用包含在 StorSimple Snapshot Manager 中的常見 MMC 功能：<ul><li>檢視</li><li>從這裡開啟新視窗</li><li>重新整理</li><li>匯出清單</li><li>說明</li></ul> |[使用 StorSimple Snapshot Manager 中的 MMC 功能表動作](storsimple-snapshot-manager-mmc-menu.md) |
| 新增或更換裝置<br>連接裝置<br>確認匯入的磁碟區群組<br>重新整理已連接的裝置<br>驗證裝置<br>檢視裝置詳細資料<br>刪除裝置組態<br>變更裝置密碼<br>更換故障的裝置<br> |[使用 StorSimple Snapshot Manager 連接並管理 StorSimple 裝置](storsimple-snapshot-manager-manage-devices.md) |
| 掛接磁碟區<br>檢視磁碟區的相關資訊<br>刪除磁碟區<br>重新掃描磁碟區<br>設定和備份基本磁碟區<br>設定及備份動態鏡像磁碟區 |[使用 StorSimple Snapshot Manager 檢視和管理磁碟區](storsimple-snapshot-manager-manage-volumes.md) |
| 檢視磁碟區群組<br>建立磁碟區群組<br>備份磁碟區群組<br>編輯磁碟區群組<br>刪除磁碟區群組 |[使用 StorSimple Snapshot Manager 建立及管理磁碟區群組](storsimple-snapshot-manager-manage-volume-groups.md) |
| 建立備份原則 <br>編輯備份原則<br>刪除備份原則 |[使用 StorSimple Snapshot Manager 建立和管理備份原則](storsimple-snapshot-manager-manage-backup-policies.md) |
| 檢視和管理排程的備份工作<br>檢視和管理最近的備份作業<br>檢視和管理目前正在執行的備份工作 |[使用 StorSimple Snapshot Manager 檢視和管理備份工作](storsimple-snapshot-manager-manage-backup-jobs.md) |
| 還原磁碟區<br>複製磁碟區或磁碟區群組<br>刪除備份<br>復原檔案<br>還原 Storsimple Snapshot Manager 資料庫 |[使用 StorSimple Snapshot Manager 管理備份目錄](storsimple-snapshot-manager-manage-backup-catalog.md) |

## <a name="next-steps"></a>後續步驟
[下載 StorSimple Snapshot Manager](https://www.microsoft.com/download/details.aspx?id=44220)。

