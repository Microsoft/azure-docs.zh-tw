---
title: 於 Azure 部署 StorSimple 裝置管理員服務 | Microsoft Docs
description: 說明如何在 Azure 入口網站中建立和刪除 StorSimple 裝置管理員服務，並說明如何管理服務註冊金鑰。
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
ms.date: 05/09/2018
ms.author: alkohli
ms.openlocfilehash: d6010b7ff03689588251a9649eecb412bf9f3a8d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38701915"
---
# <a name="deploy-the-storsimple-device-manager-service-for-storsimple-8000-series-devices"></a>為 StorSimple 8000 系列裝置部署 StorSimple 裝置管理員服務

## <a name="overview"></a>概觀

StorSimple 裝置管理員服務在 Microsoft Azure 中執行，並連接至多個 StorSimple 裝置。 建立服務之後，可使用該服務，從單一集中位置來管理所有連接至 StorSimple 裝置管理員服務的裝置，因而將管理負擔降到最低。

本教學課程說明建立、刪除、移轉服務和管理服務註冊金鑰所需的步驟。 本文所含資訊僅適用於 StorSimple 8000 系列裝置。 如需有關 StorSimple Virtual Array 的詳細資訊，請移至[為您的 StorSimple Virtual Array 部署 StorSimple 裝置管理員服務](storsimple-virtual-array-manage-service.md)。

> [!NOTE]
> -  Azure 入口網站支援執行 Update 5.0 或更新版本的裝置。 如果您的裝置不是最新的，請立即安裝 Update 5。 如需詳細資訊，請移至[安裝 Update 5](storsimple-8000-install-update-5.md)。 
> - 如果您使用 StorSimple 雲端設備 (8010/8020)，則無法更新雲端設備。 使用最新版的軟體搭配 Update 5.0，建立新的雲端設備，然後容錯移轉至所建立的新雲端設備。 
> - 執行 Update 4.0 或更舊版本的所有裝置將會遇到[縮減的管理功能](storsimple-8000-manage-service.md#supported-operations-on-devices-running-versions-prior-to-update-5.0)。 

## <a name="create-a-service"></a>建立服務
若要建立 StorSimple 裝置管理員服務，必須擁有：

* 具有企業合約的訂用帳戶
* 使用中的 Microsoft Azure 儲存體帳戶
* 用於存取管理的計費資訊

僅限 Enterprise 合約的訂用帳戶。 您也可以選擇在建立服務時產生預設儲存體帳戶。

單一服務可以管理多個裝置。 不過，裝置不能跨越多個服務。 大型企業可以擁有多個服務執行個體使用不同的訂用帳戶、組織或甚至是部署位置。 

> [!NOTE]
> 您需要個別的 StorSimple 裝置管理員服務執行個體來管理 StorSimple 8000 系列裝置和 StorSimple Virtual Array。

執行下列步驟來建立服務。

[!INCLUDE [storsimple-create-new-service](../../includes/storsimple-8000-create-new-service.md)]


各項 StorSimple 裝置管理員服務具有以下屬性：

* **名稱** – 在建立時指派給您的 StorSimple 裝置管理員服務的名稱。 **建立服務之後，無法變更服務名稱。這也適用於其他實體，例如裝置、磁碟區、磁碟區容器和備份原則，在 Azure 入口網站中無法重新命名。**
* **狀態** – 服務的狀態，可以是 [作用中]、[建立] 或是 [線上]。
* **位置** – 部署 StorSimple 裝置所在的地理位置。
* **訂用帳戶** – 與您的服務相關聯的計費訂用帳戶。

## <a name="delete-a-service"></a>刪除服務

刪除服務之前，請確定沒有任何連接的裝置正在使用它。 如果服務正在使用中，請停用連接的裝置。 停用作業將會斷絕裝置與服務之間的連接，但是會保留雲端中的裝置資料。

> [!IMPORTANT]
> 刪除服務之後，就無法回復此作業。 使用服務的任何裝置需要先重設為原廠預設值，然後才可搭配其他服務使用。 在此情況下，裝置上的本機資料以及設定將會遺失。

執行下列步驟來刪除服務。

### <a name="to-delete-a-service"></a>刪除服務

1. 搜尋您想要刪除的服務。 按一下 [資源] 圖示，然後輸入適當的字詞以進行搜尋。 在搜尋結果中，按一下您要刪除的服務。

    ![搜尋要刪除的服務](./media/storsimple-8000-manage-service/deletessdevman1.png)

2. 這會帶您前往 StorSimple 裝置管理員服務刀鋒視窗。 按一下 [刪除] 。

    ![刪除服務](./media/storsimple-8000-manage-service/deletessdevman2.png)

3. 在確認通知處按一下 [ **是** ]。 刪除服務可能需要數分鐘的時間。

    ![確認刪除](./media/storsimple-8000-manage-service/deletessdevman3.png)

## <a name="get-the-service-registration-key"></a>取得服務註冊金鑰。

在您成功建立服務之後，您必須在 StorSimple 裝置註冊服務。 若要註冊您的第一個 StorSimple 裝置，您必須使用服務註冊金鑰。 若要使用現有的 StorSimple 服務註冊其他裝置，您需要註冊金鑰和服務資料加密金鑰 (在註冊期間於第一個裝置上產生)。 如需服務資料加密金鑰的詳細資訊，請參閱 [StorSimple 安全性](storsimple-8000-security.md)。 可進入 [StorSimple 裝置管理員] 刀鋒視窗上的 [金鑰] 取得註冊金鑰。

執行下列步驟以取得服務註冊金鑰。

[!INCLUDE [storsimple-8000-get-service-registration-key](../../includes/storsimple-8000-get-service-registration-key.md)]

將服務註冊金鑰保存在安全的位置。 您需要這個金鑰，以及服務資料加密金鑰，才能對額外裝置註冊此服務。 取得服務註冊金鑰之後，您必須透過適用於 StorSimple 的 Windows PowerShell 介面設定裝置。

如需如何使用此註冊金鑰的詳細資訊，請參閱[步驟 3：透過 Windows PowerShell for StorSimple 設定和註冊裝置](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple)。

## <a name="regenerate-the-service-registration-key"></a>重新產生服務註冊金鑰
如果您必須執行金鑰替換或服務管理員的清單已變更，您必須重新產生服務註冊金鑰。 當您重新產生金鑰時，新的金鑰僅用於註冊後續裝置。 已註冊的裝置不會受到此程序影響。

執行下列步驟以重新產生服務註冊金鑰。

### <a name="to-regenerate-the-service-registration-key"></a>重新產生服務註冊金鑰
1. 在 [StorSimple 裝置管理員]中，移至 [管理]**&gt;**[金鑰]。
    
    ![[金鑰] 刀鋒視窗](./media/storsimple-8000-manage-service/regenregkey2.png)

2. 在 [金鑰] 刀鋒視窗中，按一下 [重新產生]。

    ![按一下重新產生](./media/storsimple-8000-manage-service/regenregkey3.png)
3. 在 [重新產生服務註冊金鑰] 刀鋒視窗中，檢閱重新產生金鑰時所需的動作。 向此服務註冊的所有後續裝置會使用新的註冊金鑰。 按一下 [重新產生] 以確認。 重新產生完成之後會通知您。

    ![確認重新產生](./media/storsimple-8000-manage-service/regenregkey4.png)

4. 新的服務註冊金鑰隨即顯示。

5. 複製這個金鑰並儲存，以對任何新的裝置註冊此服務。



## <a name="change-the-service-data-encryption-key"></a>變更服務資料加密金鑰
服務資料加密金鑰用來加密從 StorSimple Manager 服務傳送至 StorSimple 裝置的機密客戶資料，例如儲存體帳戶認證。 如果您的 IT 組織在存放裝置上有金鑰輪替原則，則您必須定期變更這些金鑰。 根據 StorSimple Manager 服務是管理單一裝置或多個裝置而定，金鑰變更程序可能稍有不同。 如需詳細資訊，請移至 [StorSimple 安全性和資料保護](storsimple-8000-security.md)。

變更服務資料加密金鑰分成 3 個步驟：

1. 使用 Azure Resource Manager 的 Windows PowerShell 指令碼，授權裝置以變更服務資料加密金鑰。
2. 使用 Windows PowerShell for StorSimple，起始服務資料加密金鑰變更。
3. 如果您有一個以上的 StorSimple 裝置，請在其他裝置上更新服務資料加密金鑰。

### <a name="step-1-use-windows-powershell-script-to-authorize-a-device-to-change-the-service-data-encryption-key"></a>步驟 1：使用 Windows PowerShell 指令碼，授權裝置以變更服務資料加密金鑰
一般而言，裝置系統管理員將會要求服務管理員授權裝置，以變更服務資料加密金鑰。 然後服務管理員就會授權裝置來變更金鑰。

使用以 Azure Resource Manager 為基礎的指令碼來執行此步驟。 服務管理員可以選擇具備授權資格的裝置。 然後裝置就會獲得授權，以啟動服務資料加密金鑰變更程序。 

如需有關使用指令碼的詳細資訊，請移至 [Authorize-ServiceEncryptionRollover.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Authorize-ServiceEncryptionRollover.ps1)

#### <a name="which-devices-can-be-authorized-to-change-service-data-encryption-keys"></a>哪些裝置可獲得授權以變更服務資料加密金鑰？
裝置必須符合下列準則，才能獲得授權以起始服務資料加密金鑰變更：

* 裝置必須在線上，才能獲得授權以變更服務資料加密金鑰。
* 如果尚未起始金鑰變更，您可以在 30 分鐘後再次授權同一個裝置。
* 如果先前獲得授權的裝置尚未起始金鑰變更，則您可以授權另一個裝置。 新裝置已獲授權之後，舊裝置就無法起始變更。
* 正在變換服務資料加密金鑰時，無法授權裝置。
* 如果有些已向服務註冊的裝置已變換加密，而某些裝置還沒有，則您可以授權裝置。 

### <a name="step-2-use-windows-powershell-for-storsimple-to-initiate-the-service-data-encryption-key-change"></a>步驟 2：使用 Windows PowerShell for StorSimple 起始服務資料加密金鑰變更
這個步驟是在 Windows PowerShell for StorSimple 介面中，對已獲授權的 StorSimple 裝置執行。

> [!NOTE]
> 除非金鑰變換完成，否則無法在 Azure 入口網站對 StorSimple Manager 服務執行任何作業。


如果您使用裝置序列主控台連接到 Windows PowerShell 介面，請執行下列步驟。

#### <a name="to-initiate-the-service-data-encryption-key-change"></a>起始服務資料加密金鑰變更
1. 選取選項 1 以使用完整存取權登入。
2. 在命令提示字元中，輸入：
   
     `Invoke-HcsmServiceDataEncryptionKeyChange`
3. Cmdlet 順利完成之後，您就會得到新的服務資料加密金鑰。 複製並儲存此金鑰，以供此程序的步驟 3 使用。 此金鑰將用來更新已向 StorSimple Manager 服務註冊的其餘所有裝置。
   
   > [!NOTE]
   > 此程序必須在授權 StorSimple 裝置後的四個小時內起始。
   > 
   > 
   
   然後，新金鑰就會傳送至服務並推播至所有已向服務註冊的裝置。 之後，服務儀表板上將出現警示。 此服務會停用已註冊的裝置上的所有作業，然後裝置管理員就必須在其他裝置上更新服務資料加密金鑰。 不過，不會中斷 I/O (將資料傳送至雲端的主機)。
   
   如果只有一個裝置向您的服務註冊，則變換程序現在已完成，您可以略過下一個步驟。 如果有多個裝置向您的服務註冊，請繼續步驟 3。

### <a name="step-3-update-the-service-data-encryption-key-on-other-storsimple-devices"></a>步驟 3：在其他 StorSimple 裝置上更新服務資料加密金鑰
如果有多個裝置向您的 StorSimple Manager 服務註冊，您就必須在 StorSimple 裝置的 Windows PowerShell 介面中執行這些步驟。 您在步驟 2 取得的金鑰必須用於更新所有註冊 StorSimple Manager 服務的其他 StorSimple 裝置。

請執行下列步驟，在您的裝置上更新服務資料加密。

#### <a name="to-update-the-service-data-encryption-key-on-physical-devices"></a>若要在實體裝置上更新服務資料加密金鑰
1. 使用 Windows PowerShell for StorSimple 連線到主控台。 選取選項 1 以使用完整存取權登入。
2. 在命令提示字元中，輸入：`Invoke-HcsmServiceDataEncryptionKeyChange – ServiceDataEncryptionKey`
3. 提供您在 [步驟 2：使用 Windows PowerShell for StorSimple 起始服務資料加密金鑰變更](#to-initiate-the-service-data-encryption-key-change)中取得的服務資料加密金鑰。

#### <a name="to-update-the-service-data-encryption-key-on-all-the-80108020-cloud-appliances"></a>若要更新所有 8010/8020 雲端設備上的服務資料加密金鑰
1. 下載並安裝 [Update-CloudApplianceServiceEncryptionKey.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Update-CloudApplianceServiceEncryptionKey.ps1) PowerShell 指令碼。 
2. 開啟 PowerShell，並在命令提示字元中，輸入：`Update-CloudApplianceServiceEncryptionKey.ps1 -SubscriptionId [subscription] -TenantId [tenantid] -ResourceGroupName [resource group] -ManagerName [device manager]`

此指令碼將確保服務資料加密金鑰已在裝置管理員下的所有 8010/8020 雲端設備上設定。

## <a name="supported-operations-on-devices-running-versions-prior-to-update-50"></a>執行 Update 5.0 之前版本的裝置上支援的作業
Azure 入口網站僅支援執行 Update 5.0 和更新版本的 StorSimple 裝置。 對執行舊版本的裝置的支援有限。 移轉至 Azure 入口網站之後，請使用下表了解執行 Update 5.0 之前版本的裝置可支援哪些作業。

| 作業                                                                                                                       | 支援      |
|---------------------------------------------------------------------------------------------------------------------------------|----------------|
| 註冊裝置                                                                                                               | yes            |
| 設定裝置設定，例如一般設定、網路設定和安全性設定                                                                | yes            |
| 掃描、下載，及安裝更新                                                                                             | yes            |
| 停用裝置                                                                                                               | yes            |
| 刪除裝置                                                                                                                   | yes            |
| 建立、修改及刪除磁碟區容器                                                                                   | 否             |
| 建立、修改及刪除磁碟區                                                                                             | 否             |
| 建立、修改及刪除備份原則                                                                                      | 否             |
| 進行手動備份                                                                                                            | 否             |
| 進行排程備份                                                                                                         | 不適用 |
| 從備份組還原                                                                                                        | 否             |
| 複製至執行 Update 3.0 和更新版本的裝置 <br> 來源裝置執行的是 Update 3.0 之前的版本。                                | yes            |
| 複製到執行 Update 3.0 之前版本的裝置                                                                          | 否             |
| 作為容錯移轉的來源裝置 <br> (從執行 Update 3.0 之前版本的裝置到執行 Update 3.0 和更新版本的裝置)                                                               | yes            |
| 作為容錯移轉的目標裝置 <br> (到執行 Update 3.0 之前軟體版本的裝置)                                                                                   | 否             |
| 清除警示                                                                                                                  | yes            |
| 檢視備份原則、備份類別目錄、磁碟區、磁碟區容器、監視圖表、作業，以及傳統入口網站中建立的警示 | yes            |
| 開啟和關閉裝置控制器                                                                                              | yes            |


## <a name="next-steps"></a>後續步驟
* 深入了解 [StorSimple 部署程序](storsimple-8000-deployment-walkthrough-u2.md)。
* 深入了解 [管理 StorSimple 儲存體帳戶](storsimple-8000-manage-storage-accounts.md)。
* 深入了解如何[使用 StorSimple 裝置管理員服務管理 StorSimple 裝置](storsimple-8000-manager-service-administration.md)。
