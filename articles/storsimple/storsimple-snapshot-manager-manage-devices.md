---
title: 使用 StorSimple Snapshot Manager 來管理裝置 | Microsoft Docs
description: 描述如何使用 StorSimple Snapshot Manager MMC 嵌入式管理單元，來連接和管理 StorSimple 裝置。
services: storsimple
documentationcenter: ''
author: SharS
manager: timlt
editor: ''
ms.assetid: 966ecbe3-a7fa-4752-825f-6694dd949946
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 06/05/2017
ms.author: v-sharos
ms.openlocfilehash: f5e3186a4271e0be781f367fa75ada195c58c960
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23111436"
---
# <a name="use-storsimple-snapshot-manager-to-connect-and-manage-storsimple-devices"></a>使用 StorSimple Snapshot Manager 來連接和管理 StorSimple 裝置
## <a name="overview"></a>概觀
您可以使用 StorSimple Snapshot Manager [ **範圍** ] 窗格中的節點，來確認已匯入的 StorSimple 裝置資料，並重新整理已連接的儲存體裝置。 此外，當您按一下 [裝置] 節點時，也可以在 [結果] 窗格中看到已連接的裝置清單和對應的狀態資訊。

![連接的裝置](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_connect_devices.png)

**圖 1：StorSimple Snapshot Manager 連接的裝置** 

視您的 [檢視] 選項而定，[結果] 窗格會顯示每個裝置的下列相關資訊。 (如需設定檢視的詳細資訊，請移至 [檢視功能表](storsimple-use-snapshot-manager.md#view-menu)。

| 結果資料行 | 說明 |
|:--- |:--- |
| Name |Azure 傳統入口網站中設定的裝置名稱 |
| 模型 |裝置的型號 |
| 版本 |裝置上安裝的軟體版本 |
| 狀態 |裝置是否可用 |
| 上次同步處理 |上次同步處理裝置的日期和時間時 |
| 序號 |裝置的序號 |

如果您以滑鼠右鍵按一下 [範圍] 窗格中的 [裝置] 節點，則可以選取下列動作：

* 新增或更換裝置
* 連接裝置並確認匯入
* 重新整理已連接的裝置

如果您按一下 [裝置] 節點，然後再以滑鼠右鍵按一下 [結果] 窗格中的裝置名稱，則可以選取下列動作：

* 驗證裝置
* 檢視裝置詳細資料
* 重新整理裝置
* 刪除裝置組態
* 變更裝置密碼

> [!NOTE]
> 所有這些動作也可在 [ **動作** ] 窗格中取得。


本教學課程說明如何使用 StorSimple Snapshot Manager，來連接和管理裝置，並執行下列工作：

* 新增或更換裝置
* 連接裝置並確認匯入
* 重新整理已連接的裝置
* 驗證裝置
* 檢視裝置詳細資料
* 重新整理個別裝置
* 刪除裝置組態
* 變更過期的裝置密碼
* 更換故障的裝置

> [!NOTE]
> 如需使用 StorSimple Snapshot Manager 介面的一般資訊，請至 [StorSimple Snapshot Manager 使用者介面](storsimple-use-snapshot-manager.md)。


## <a name="add-or-replace-a-device"></a>新增或更換裝置
請使用下列程序來新增或更換 StorSimple 裝置。

#### <a name="to-add-or-replace-a-device"></a>若要新增或更換裝置
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [裝置] 節點，然後按一下 [設定裝置]。 [ **設定裝置** ] 對話方塊隨即出現。
   
    ![設定 StorSimple 裝置](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_config_device.png) 
3. 在 [ **裝置** ] 下拉式清單方塊中，選取裝置或虛擬裝置的 IP 位址。 
4. 在 [密碼]  文字方塊中，輸入您在 Azure 傳統入口網站中為裝置建立的 StorSimple Snapshot Manager 密碼。 按一下 [SERVICEPRINCIPAL] 。 StorSimple Snapshot Manager 會搜尋您所識別的裝置。 
   
   * 如果裝置可供使用，StorSimple Snapshot Manager 會新增連接。
   * 如果由於任何原因而無法使用裝置，StorSimple Snapshot Manager 會傳回錯誤訊息。 按一下 [確定] 以關閉錯誤訊息，然後按一下 [取消] 以關閉 [設定裝置] 對話方塊。

## <a name="connect-a-device-and-verify-imports"></a>連接裝置並確認匯入
請使用下列程序來連接 StorSimple 裝置，並確認任何已匯入的現有磁碟區群組具有相關聯的備份，。

#### <a name="to-connect-a-device-and-verify-imports"></a>若要連接裝置並確認匯入
1. 若要將裝置連接至 StorSimple Snapshot Manager，請遵循新增或更換裝置中的指示。 當它連接至裝置時，StorSimple Snapshot Manager 的回應如下：
   
   * 如果由於任何原因而無法使用裝置，StorSimple Snapshot Manager 會傳回錯誤訊息。 
   
   * 如果裝置可供使用，StorSimple Snapshot Manager 會新增連接。 當您選取裝置時，它會出現在 [結果] 窗格中，而且狀態欄位會指出該裝置 [可用]。 StorSimple Snapshot Manager 會匯入任何針對裝置設定的磁碟區群組，前提是磁碟區群組具有相關聯的備份。 不會匯入備份原則。 不會匯入沒有相關聯之備份的磁碟區群組。
2. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
3. 以滑鼠右鍵按一下 [範圍] 窗格中的最上層節點，然後按一下 [切換匯入顯示]。
   
    ![選取 [切換匯入顯示]](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Toggle_Imports_Display.png) 
4. [ **切換匯入顯示** ] 對話方塊隨即出現，顯示已匯入之磁碟區群組和備份的狀態。 按一下 [SERVICEPRINCIPAL] 。

在順利匯入磁碟區群組和備份之後，您可以使用 StorSimple Snapshot Manager 來管理它們，就如同您管理您使用 StorSimple Snapshot Manager 所建立並設定的磁碟區群組和備份一般。 

## <a name="refresh-connected-devices"></a>重新整理已連接的裝置
請使用下列程序，同步處理已連接的 StorSimple 裝置與 StorSimple Snapshot Manager。

#### <a name="to-refresh-connected-devices"></a>若要重新整理已連接的裝置
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，以滑鼠右鍵按一下 [裝置]，然後按一下 [重新整理裝置]。 這會同步處理已連接的裝置與 StorSimple Snapshot Manager，讓您可以檢視磁碟區群組和備份，包括任何最近新增的項目。 
   
    ![重新整理 StorSimple 裝置](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Refresh_devices.png)

[ **重新整理裝置** ] 動作會從已連接的裝置擷取任何新的磁碟區群組和任何相關聯的備份。 不同於適用於 [磁碟區] 節點的 [重新掃描磁碟區] 動作，[重新整理裝置] 不會還原備份登錄。

## <a name="authenticate-a-device"></a>驗證裝置
請使用下列程序，利用 StorSimple Snapshot Manager 來驗證 StorSimple 裝置。

#### <a name="to-authenticate-a-device"></a>若要驗證裝置
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，按一下 [裝置]。
3. 在 [結果] 窗格中，以滑鼠右鍵按一下裝置的名稱，然後按一下 [驗證]。
4. [ **驗證** ] 對話方塊隨即出現。 輸入裝置密碼，然後按一下 [ **確定**]。
   
    ![[驗證] 對話方塊](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Authenticate.png) 

## <a name="view-device-details"></a>檢視裝置詳細資料
請使用下列程序來檢視 StorSimple 裝置的詳細資料，如有必要，請重新同步處理裝置與 StorSimple Snapshot Manager。

#### <a name="to-view-and-resynchronize-device-details"></a>若要檢視和重新同步處理裝置詳細資料
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，按一下 [裝置]。
3. 在 [結果] 窗格中，以滑鼠右鍵按一下裝置的名稱，然後按一下 [詳細資料]。

4. [裝置詳細資料] 對話方塊隨即出現。 此方塊會顯示名稱、型號、版本、序號、狀態、目標 iSCSI 合格名稱 (IQN)，以及上次同步處理日期和時間。

* 按一下 [ **重新同步處理** ] 以同步處理裝置。
* 按一下 [確定] 或 [取消] 以關閉對話方塊。
  
  ![裝置詳細資料](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_Device_details.png) 

## <a name="refresh-an-individual-device"></a>重新整理個別裝置
請使用下列程序，來重新同步處理個別的 StorSimple 裝置與 StorSimple Snapshot Manager 。

#### <a name="to-refresh-a-device"></a>若要重新整理裝置
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。 
2. 在 [範圍] 窗格中，按一下 [裝置]。 
3. 在 [結果] 窗格中，以滑鼠右鍵按一下裝置的名稱，然後按一下 [重新整理裝置]。 這會同步處理裝置與 StorSimple Snapshot Manager。

## <a name="delete-a-device-configuration"></a>刪除裝置組態
請使用下列程序，從 StorSimple Snapshot Manager 刪除個別的 StorSimple 裝置組態。

#### <a name="to-delete-a-device-configuration"></a>若要刪除裝置組態
1. 按一下桌面圖示，以啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，按一下 [裝置]。 
3. 在 [結果] 窗格中，以滑鼠右鍵按一下裝置的名稱，然後按一下 [刪除]。 
4. 下列訊息隨即出現。 按一下 [是] 以刪除組態，或按一下 [否] 以取消刪除。
   
    ![刪除裝置組態](./media/storsimple-snapshot-manager-manage-devices/HCS_SSM_DeleteDevice.png)

## <a name="change-an-expired-device-password"></a>變更過期的裝置密碼
您必須輸入密碼，才能使用 StorSimple Snapshot Manager 驗證 StorSimple 裝置。 當使用 Windows PowerShell 介面來設定裝置時，您可以設定此密碼。 不過，密碼會過期。 如果發生這種情況，您可以使用 Azure 傳統入口網站變更密碼。 然後，因為在密碼過期之前，已在 StorSimple Snapshot Manager 設定裝置，所以您必須在 StorSimple Snapshot Manager 重新驗證裝置。

#### <a name="to-change-the-expired-password"></a>若要變更過期的密碼
1. 在 Azure 傳統入口網站中，啟動 StorSimple Manager 服務。
2. 按一下 確定 **裝置** > **設定** 。
3. 向下捲動到 StorSimple Snapshot Manager 區段。 輸入 14 或 15 個字元的密碼。 請確定密碼混有大寫、小寫、數字和特殊字元。
4. 請重新輸入密碼加以確認。
5. 按一下頁面底部的 [儲存]  。

#### <a name="to-re-authenticate-the-device"></a>若要重新驗證裝置
1. 啟動 StorSimple Snapshot Manager。
2. 在 [範圍] 窗格中，按一下 [裝置]。 已設定的裝置清單會出現在 [ **結果** ] 窗格中。
3. 選取裝置、按一下滑鼠右鍵，然後按一下 [ **驗證**]。
4. 在 [ **驗證** ] 視窗中，輸入新密碼。
5. 選取裝置、按一下滑鼠右鍵，然後選取 [ **重新整理裝置**]。 這會同步處理裝置與 StorSimple Snapshot Manager。

## <a name="replace-a-failed-device"></a>更換故障的裝置
如果 StorSimple 裝置故障，並更換為備用 (容錯移轉) 裝置，請使用下列步驟，連接至新裝置並檢視相關聯的備份。

#### <a name="to-connect-to-a-new-device-after-failover"></a>若要在容錯移轉之後連接至新裝置
1. 重新設定新裝置的 iSCSI 連接。 如需相關指示，請移至 [部署您的內部部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)中的「步驟 7：掛載、初始化和格式磁碟區」。

> [!NOTE]
> 如果新的 StorSimple 裝置具有與舊裝置相同的 IP 位址，則您也許能夠連接舊組態。


1. 停止 Microsoft StorSimple 管理服務：
   
   1. 啟動伺服器管理員。
   2. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   3. 在 [服務] 視窗中，選取 [Microsoft StorSimple 管理服務]。
   4. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [停止服務]。
2. 移除與舊裝置相關的組態資訊：
   
   1. 在 [檔案總管] 中，瀏覽至 C:\ProgramData\Microsoft\StorSimple\BACatalog.
   2. 刪除 BACatalog 資料夾中的檔案。
3. 重新啟動 Microsoft StorSimple 管理服務：
   
   1. 在伺服器管理員儀表板的 [工具] 功能表上，選取 [服務]。
   2. 在 [服務] 視窗中，選取 [Microsoft StorSimple 管理服務]。
   3. 在右窗格的 [Microsoft StorSimple 管理服務] 之下，按一下 [重新啟動服務]。
4. 啟動 StorSimple Snapshot Manager。
5. 若要設定新的 StorSimple 裝置，請完成＜部署 StorSimple Snapshot Manager＞中的 [步驟 2：連接 StorSimple 裝置](storsimple-snapshot-manager-deployment.md)中的步驟。
6. 以滑鼠右鍵按一下 [範圍] 窗格中的最上層節點 (範例中的 StorSimple Snapshot Manager)，然後按一下 [切換匯入顯示]。 
7. 當可在 StorSimple Snapshot Manager 中看到匯入的磁碟區群組和備份時，即會出現一則訊息。 按一下 [SERVICEPRINCIPAL] 。

## <a name="next-steps"></a>後續步驟
* 了解如何 [使用 StorSimple Snapshot Manager 來管理您的 StorSimple 解決方案](storsimple-snapshot-manager-admin.md)。
* 了解如何 [使用 StorSimple Snapshot Manager 來檢視和管理磁碟區](storsimple-snapshot-manager-manage-volumes.md)。

