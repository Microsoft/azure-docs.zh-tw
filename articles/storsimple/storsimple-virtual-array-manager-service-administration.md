---
title: Microsoft Azure StorSimple Manager Virtual Array 管理 | Microsoft Docs
description: 了解如何使用 Azure 入口網站中的 StorSimple 裝置管理員服務管理 StorSimple 內部部署 Virtual Array。
services: storsimple
documentationcenter: ''
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 958244a5-f9f5-455e-b7ef-71a65558872e
ms.service: storsimple
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/1/2016
ms.author: alkohli
ms.openlocfilehash: a74a160eae88a2d03460a1346479c333d8f9d524
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23109896"
---
# <a name="use-the-storsimple-device-manager-service-to-administer-your-storsimple-virtual-array"></a>使用 StorSimple 裝置管理員服務管理 StorSimple Virtual Array
![安裝程序流程](./media/storsimple-virtual-array-manager-service-administration/manage4.png)

## <a name="overview"></a>概觀
本文描述 StorSimple 裝置管理員服務介面，包括如何連接它和各種可用的選項，並提供可以透過此 UI 執行的特定工作流程的連結。

閱讀本文之後，您將了解如何：

* 連接至 StorSimple 裝置管理員服務
* 瀏覽 StorSimple 裝置管理員 UI
* 透過 StorSimple 裝置管理員服務管理 StorSimple Virtual Array

> [!NOTE]
> 若要檢視 StorSimple 8000 系列裝置可用的管理選項，請移至 [使用 StorSimple Manager 服務管理 StorSimple 裝置](storsimple-manager-service-administration.md)。
> 
> 

## <a name="connect-to-the-storsimple-device-manager-service"></a>連接至 StorSimple 裝置管理員服務
StorSimple Manager 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple Virtual Array。 您可以使用在瀏覽器中執行的中央 Microsoft Azure 入口網站來管理這些裝置。 若要連接至 StorSimple 裝置管理員服務，請執行下列動作。

#### <a name="to-connect-to-the-service"></a>連接至此服務
1. 移至 [https://ms.portal.azure.com](https://ms.portal.azure.com)。
2. 使用您的 Microsoft 帳戶認證，登入 Microsoft Azure 入口網站 (位於窗格右上角)。
3. 在 StorSimple 裝置管理員上瀏覽至 [瀏覽] --> [篩選]，以檢視指定訂用帳戶中所有的裝置管理員。

## <a name="use-the-storsimple-device-manager-service-to-perform-management-tasks"></a>使用 StorSimple 裝置管理員服務來執行管理工作
下表顯示所有一般管理工作和複雜工作流程 (可在 StorSimple 裝置管理員服務摘要刀鋒視窗內執行) 的摘要。 這些工作會根據在其中啟動它們的刀鋒視窗加以組織。

如需每個工作流程的詳細資訊，請按一下資料表中的適當程序。

#### <a name="storsimple-device-manager-workflows"></a>StorSimple 裝置管理員工作流程
| 如果您想要執行此動作... | 使用此程序 |
| --- | --- | --- |
| 建立服務</br>刪除服務</br>取得服務註冊金鑰。</br>重新產生服務註冊金鑰 |[部署 StorSimple 裝置管理員服務](storsimple-virtual-array-manage-service.md) |
| 檢視活動記錄檔 |[使用 StorSimple 服務摘要](storsimple-virtual-array-service-summary.md) |
| 停用 Virtual Array</br>刪除 Virtual Array |[停用或刪除虛擬陣列](storsimple-virtual-array-deactivate-and-delete-device.md) |
| 災害復原和裝置容錯移轉</br>容錯移轉必要條件</br>業務持續性災害復原 (BCDR)</br>災害復原時發生錯誤 |[StorSimple Virtual Array 的災害復原和裝置容錯移轉](storsimple-virtual-array-failover-dr.md) |
| 備份共用和磁碟區</br>進行手動備份</br>變更備份排程</br>檢視現有的備份 |[備份 StorSimple Virtual Array](storsimple-virtual-array-backup.md) |
| 從備份組複製共用</br>從備份組複製磁碟區</br>項目層級復原 (僅限檔案伺服器) |[從 StorSimple Virtual Array 的備份複製](storsimple-virtual-array-clone.md) |
| 有關儲存體帳戶</br>新增儲存體帳戶</br>編輯儲存體帳戶</br>刪除儲存體帳戶 |[管理 StorSimple Virtual Array 的儲存體帳戶](storsimple-virtual-array-manage-storage-accounts.md) |
| 關於存取控制記錄</br>加入或修改存取控制記錄 </br>刪除存取控制記錄 |[管理 StorSimple Virtual Array 的存取控制記錄](storsimple-virtual-array-manage-acrs.md) |
| 檢視工作詳細資料 |[管理 StorSimple Virtual Array 作業](storsimple-virtual-array-manage-jobs.md) |
| 設定警示設定</br>接收警示通知</br>管理警示</br>檢閱警示 |[檢視和管理 StorSimple Virtual Array 的警示](storsimple-virtual-array-manage-alerts.md) |
| 修改裝置系統管理員密碼 |[變更 StorSimple Virtual Array 裝置系統管理員密碼](storsimple-virtual-array-change-device-admin-password.md) |
| 安裝軟體更新 |[更新您的 Virtual Array](storsimple-virtual-array-install-update.md) |

> [!NOTE]
> 您必須使用 [本機 Web UI](storsimple-ova-web-ui-admin.md) 以執行下列工作：
> 
> * [擷取服務資料加密金鑰](storsimple-ova-web-ui-admin.md#get-the-service-data-encryption-key)
> * [建立支援封裝](storsimple-ova-web-ui-admin.md#generate-a-log-package)
> * [停止和重新啟動 Virtual Array](storsimple-ova-web-ui-admin.md#shut-down-and-restart-your-device)
> 
> 

## <a name="next-steps"></a>後續步驟
如需 Web UI 及如何使用的詳細資訊，請移至 [使用 StorSimple Web UI 管理 StorSimple Virtual Array](storsimple-ova-web-ui-admin.md)。

