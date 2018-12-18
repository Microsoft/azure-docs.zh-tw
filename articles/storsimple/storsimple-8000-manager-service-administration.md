---
title: StorSimple 裝置管理員服務管理 | Microsoft Docs
description: 了解如何使用 Azure 入口網站中的 StorSimple 裝置管理員服務管理 StorSimple 裝置。
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/12/2017
ms.author: alkohli
ms.openlocfilehash: 0e7d7f44a70278a7777ba6c32c8e546074953fdc
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23108566"
---
# <a name="use-the-storsimple-device-manager-service-to-administer-your-storsimple-device"></a>使用 StorSimple 裝置管理員服務管理 StorSimple 裝置

## <a name="overview"></a>概觀

本文描述 StorSimple 裝置管理員服務介面，包括如何與其連接、各種可用選項，以及可以透過此 UI 執行的特定工作流程連結。 本指南適用於以下兩者；StorSimple 實體裝置和雲端設備。

閱讀本文知後，您將了解：

* 連接至 StorSimple 裝置管理員服務
* 透過 StorSimple 裝置管理員服務管理 StorSimple 裝置

## <a name="connect-to-storsimple-device-manager-service"></a>連接至 StorSimple 裝置管理員服務

StorSimple 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple 裝置。 您可以使用在瀏覽器中執行的中央 Microsoft Azure 入口網站來管理這些裝置。 若要連接至 StorSimple 裝置管理員服務，請執行下列動作。

#### <a name="to-connect-to-the-service"></a>連接至此服務
1. 瀏覽至 [https://portal.azure.com/](https://portal.azure.com/)。
2. 使用您的 Microsoft 帳戶認證，登入 Microsoft Azure 入口網站 (位於窗格右上角)。
3. 向下捲動左方瀏覽窗格以存取 StorSimple 裝置管理員服務。


## <a name="administer-storsimple-device-using-storsimple-device-manager-service"></a>使用 StorSimple 裝置管理員服務管理 StorSimple 裝置

下表顯示所有一般管理工作和複雜工作流程 (可在 StorSimple 裝置管理員服務 UI 內執行) 的摘要。 這些工作會根據在其中啟動它們的 UI 刀鋒視窗加以組織。

如需每個工作流程的詳細資訊，請按一下資料表中的適當程序。

#### <a name="storsimple-device-manager-workflows"></a>StorSimple 裝置管理員工作流程

| 如果您想要執行此動作... | 使用此程序。 |
| --- | --- |
| 建立服務</br>刪除服務</br>取得服務註冊金鑰</br>重新產生服務註冊金鑰 |[部署 StorSimple 裝置管理員服務](storsimple-8000-manage-service.md) |
| 檢視活動記錄檔 |[使用 StorSimple 裝置管理員服務摘要](storsimple-8000-service-dashboard.md) |
| 變更服務資料加密金鑰</br>檢視作業記錄檔 |[使用 StorSimple 裝置管理員服務儀表板](storsimple-8000-service-dashboard.md) |
| 停用裝置</br>刪除裝置 |[停用或刪除裝置](storsimple-8000-deactivate-and-delete-device.md) |
| 了解災害復原和裝置容錯移轉</br>容錯移轉至實體裝置</br>容錯移轉至虛擬裝置</br>業務持續性災害復原 (BCDR) |[StorSimple 裝置的容錯移轉與災害復原](storsimple-8000-device-failover-disaster-recovery.md) |
| 列出磁碟區備份</br>選取備份組</br>刪除備份組 |[管理備份](storsimple-8000-manage-backup-catalog.md) |
| 複製磁碟區 |[複製磁碟區](storsimple-8000-clone-volume-u2.md) |
| 還原備份組 |[還原備份組](storsimple-8000-restore-from-backup-set-u2.md) |
| 有關儲存體帳戶</br>新增儲存體帳戶</br>編輯儲存體帳戶</br>刪除儲存體帳戶</br>替換儲存體帳戶的金鑰 |[管理儲存體帳戶](storsimple-8000-manage-storage-accounts.md) |
| 關於頻寬範本</br>新增頻寬範本</br>編輯頻寬範本</br>刪除頻寬範本</br>使用預設頻寬範本</br>建立在指定時間啟動的全天頻寬範本 |[管理頻寬範本](storsimple-8000-manage-bandwidth-templates.md) |
| 關於存取控制記錄</br>建立存取控制記錄</br>編輯存取控制記錄</br>刪除存取控制記錄 |[管理存取控制記錄](storsimple-8000-manage-acrs.md) |
| 檢視工作詳細資料</br>取消工作 |[管理工作](storsimple-8000-manage-jobs-u2.md) |
| 接收警示通知</br>管理警示</br>檢閱警示 |[檢視和管理 StorSimple 警示](storsimple-8000-manage-alerts.md) |
| 建立監視圖表 |[監視您的 StorSimple 裝置](storsimple-monitor-device.md) |
| 新增磁碟區容器</br>修改磁碟區容器</br>刪除磁碟區容器 |[管理磁碟區容器](storsimple-8000-manage-volume-containers.md) |
| 新增磁碟區</br>修改磁碟區</br>使磁碟區離線</br>刪除磁碟區</br>監視磁碟區 |[管理磁碟區](storsimple-8000-manage-volumes-u2.md) |
| 修改裝置設定</br>修改時間設定</br>修改 DNS.md 設定</br>設定網路介面 |[修改 StorSimple 裝置的裝置組態](storsimple-8000-modify-device-config.md) |
| 檢視 Web Proxy 設定 |[設定裝置的 Web Proxy](storsimple-8000-configure-web-proxy.md) |
| 修改裝置系統管理員密碼</br>修改 StorSimple Snapshot Manager 密碼 |[變更 StorSimple 密碼](storsimple-8000-change-passwords.md) |
| 設定遠端管理 |[遠端連接至 StorSimple 裝置](storsimple-8000-remote-connect.md) |
| 設定警示設定 |[檢視和管理 StorSimple 警示](storsimple-8000-manage-alerts.md) |
| 為 StorSimple 裝置設定 CHAP |[為 StorSimple 裝置設定 CHAP](storsimple-configure-chap.md) |
| 新增備份原則</br>新增或修改排程</br>刪除備份原則</br>進行手動備份</br>建立具有多個磁碟區和排程的自訂備份原則 |[管理備份原則](storsimple-8000-manage-backup-policies-u2.md) |
| 停止裝置控制器</br>重新啟動裝置控制器</br>關閉裝置控制器</br>將裝置重設為原廠預設值</br>(上述項目僅適用於內部部署裝置) |[管理 StorSimple 裝置控制器](storsimple-8000-manage-device-controller.md) |
| 了解 StorSimple 硬體元件</br>監視硬體狀態</br>(上述項目僅適用於內部部署裝置) |[監視硬體元件](storsimple-8000-monitor-hardware-status.md) |
| 建立支援封裝 |[建立及管理支援封裝](storsimple-8000-contact-microsoft-support.md#start-a-support-session-in-windows-powershell-for-storsimple) |
| 安裝軟體更新 |[更新您的裝置](storsimple-update-device.md) |

## <a name="next-steps"></a>後續步驟

如果您遇到與 StorSimple 裝置的日常作業或其中任何硬體元件的任何問題，請參閱：

* [使用診斷工具進行疑難排解](storsimple-8000-diagnostics.md)
* [使用 StorSimple 監視 LED 指示燈](storsimple-monitoring-indicators.md)

如果您不能解決問題，且您必須建立服務要求，請參閱： [連絡 Microsoft 支援服務](storsimple-8000-contact-microsoft-support.md)。

