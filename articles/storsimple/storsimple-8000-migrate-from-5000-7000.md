---
title: 將 StorSimple 5000-7000 系列的資料移轉到 8000 系列裝置| Microsoft Docs
description: 提供移轉功能的概觀和必要條件。
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/23/2018
ms.author: alkohli
ms.openlocfilehash: 23fde5fab8f223ff1d958a8fac6a31818f1fdd7c
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818911"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-8000-series-device"></a>將 StorSimple 5000-7000 系列的資料移轉到 8000 系列裝置

> [!IMPORTANT]
> - StorSimple 5000/7000 系列將於 2019 年 7 月 31 日到達結束支援 (EOS) 狀態。 建議 StorSimple 5000/7000 系列的客戶遷移至文件中所述的其中一個替代方案。
> - 移轉目前是協助進行的作業。 如果您要將 StorSimple 5000-7000 系列裝置的資料移轉到 8000 系列裝置，您需要向 Microsoft 支援服務排定移轉的時間。 Microsoft 支援服務接著會啟用移轉用的訂用帳戶。 如需詳細資訊，請參閱如何[開啟支援票證](storsimple-8000-contact-microsoft-support.md)。
> - 您提出服務要求之後，可能需要幾週的時間來執行移轉計劃並實際開始移轉。
> - 連絡 Microsoft 支援服務之前，務必詳閱和完成本文提及的[移轉必要條件](#migration-prerequisites)。

## <a name="overview"></a>概觀

本文介紹的移轉功能可讓 StorSimple 5000-7000 系列客戶將資料移轉到 StorSimple 8000 系列的實體裝置或 8010/8020 雲端設備上。 文內也提供逐步解說的下載連結，有助於瞭解將 5000-7000 系列舊版裝置的資料移轉到 8000 系列實體裝置或雲端設備時所需的步驟。

本文適用於內部部署 8000 系列裝置和 StorSimple 雲端設備。


## <a name="migration-feature-versus-host-side-migration"></a>相較於主機端移轉的移轉功能

您可以使用移轉功能或執行主機端移轉來移轉資料。 本節會具體說明每個方法，包括各自的優缺點。 請運用這些資訊，找出移轉資料時希望使用的方法。

移轉功能會模擬從 7000/5000 系列至 8000 系列的災害復原 (DR) 程序。 這項功能可讓您將 5000/7000 系列格式的資料移轉至 Azure 的 8000 系列格式。 使用 StorSimple 移轉工具即可起始移轉程序。 此工具會先下載和轉換 8000 系列裝置上的備份中繼資料，然後使用最新的備份將磁碟區公開在裝置上。

|      | 優點                                                                                                                                     |缺點                                                                                                                                                              |
|------|-------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.   | 移轉流程會保留 5000/7000 系列的備份記錄。                                               | 使用者嘗試存取資料時，此種移轉會從 Azure 下載資料，因此會產生資料下載費用。                                     |
| 2.   | 主機端不會移轉任何資料。                                                                                                     | 在 8000 系列開始備份，直到最新備份出現在 8000 系列上之間的這段時間，此流程需要停機 (可以使用移轉工具進行估計)。 |
| 3.   | 此流程會保留 8000 系列裝置上的所有原則、頻寬範本、加密和其他設定。                      | 使用者的存取動作只會取回使用者所存取的資料，而不會將整個資料集解除凍結。                                                  |
| 4.   | 此流程需要額外的時間來轉換在 Azure 中以非同步方式完成的所有舊版備份，不過不會影響實際執行 | 移轉只能在雲端設定的層級進行。  雲端設定中的個別磁碟區無法個別移轉                       |

主機端移轉能夠單獨設定 8000 系列，並將 5000/7000 系列裝置的資料複製到 8000 系列裝置。 這相當於將一個儲存體裝置的資料移轉到另一個儲存體裝置。 有各式各樣的工具可用來複製資料，例如 Diskboss、robocopy 等。

|      | 優點                                                                                                                      |缺點                                                                                                                                                                                                      |
|------|---------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1.   | 移轉可以分階段或逐磁碟區進行。                                               | 先前的備份 (5000/7000 系列) 無法在 8000 系列裝置上使用。                                                                                                       |
| 2.   | 可將資料彙總到 Azure 上的一個儲存體帳戶。                                                       | 8000 系列第一次備份到雲端需要較長的時間，因為 8000 系列的所有資料都要備份到 Azure 上。                                                                     |
| 3.   | 成功移轉之後，所有資料都會在設備本機上。 存取資料時不會有任何延遲。 | Azure 儲存體耗用量會增加，直到從 5000/7000 裝置刪除資料為止。                                                                                                        |
| 4.   |                                                                                                                           | 如果 7000/5000 系列裝置有大量的資料，在移轉期間必須將資料從 Azure 下載，因此會產生與 Azure 下載資料有關的費用，並會出現延遲情況 |

本文僅著重在從 5000/7000 到 8000 系列裝置的移轉功能。 如需有關主機端移轉的詳細資訊，請移至[從其他儲存體裝置移轉](http://download.microsoft.com/download/9/4/A/94AB8165-CCC4-430B-801B-9FD40C8DA340/Migrating%20Data%20to%20StorSimple%20Volumes_09-02-15.pdf)。

## <a name="migration-prerequisites"></a>移轉必要條件

以下是舊版 5000 或 7000 系列裝置和 8000 系列 StorSimple 裝置的移轉必要條件。

> [!IMPORTANT]
> 請先詳閱並達成移轉的必要條件，再向 Microsoft 支援服務提出服務要求。

### <a name="for-the-50007000-series-device-source"></a>5000/7000 系列裝置 (來源)

開始移轉之前，請確定︰

* 您有 5000 或 7000 系列來源裝置；該裝置可以啟動或關閉。

    > [!IMPORTANT]
    > 建議您在整個移轉流程中，對此裝置進行序列存取。 若出現任何裝置問題，序列存取有助於進行疑難排解。

* 5000 或 7000 系列來源裝置目前執行的軟體版本為 v2.1.1.518 或更新版本。 不支援舊版。
* 若要確認 5000 或 7000 系列目前執行的版本，請看 Web UI 的右上角。 該處應會顯示您的裝置目前執行的軟體版本。 若要移轉，5000 或 7000 系列應該執行 v2.1.1.518。

    ![請檢查繼承裝置上的軟體版本](media/storsimple-8000-migrate-from-5000-7000/check-version-legacy-device1.png)

    * 如果您使用中的裝置並非執行 v2.1.1.518 或更新版本，請將系統升級到所要求的最低版本。 如需詳細說明，請參閱[將系統升級至 v2.1.1.518](http://onlinehelp.storsimple.com/111_Appliance/6_System_Upgrade_Guides/Current_(v2.1.1)/000_Software_Patch_Upgrade_Guide_v2.1.1.518)。
    * 如果您執行的是 v2.1.1.518，請移至 Web UI，查看是否有任何登錄還原失敗的通知。 如果登錄還原失敗，請執行登錄還原。 如需詳細資訊，請移至[執行登錄還原](http://onlinehelp.storsimple.com/111_Appliance/2_User_Guides/1_Current_(v2.1.1)/1_Web_UI_User_Guide_WIP/2_Configuration/4_Cloud_Accounts/1_Cloud_Credentials#Restoring_Backup_Registry)。
    * 如果關閉的裝置並非執行 v2.1.1.518，請容錯移轉至執行 v2.1.1.518 的備用裝置。 如需詳細說明，請參閱 5000/7000 系列 StorSimple 裝置的 DR。
    * 製作雲端快照集來備份裝置的資料。
    * 檢查來源裝置上是否在執行其他任何進行中的備份作業。 StorSimple 資料保護主控台主機上的作業也包括在內。 請等候目前的工作完成。


### <a name="for-the-8000-series-physical-device-target"></a>對於 8000 系列實體裝置 (目標)

開始移轉之前，請確定︰

* 已登錄並執行目標的 8000 系列裝置。 如需詳細資訊，請參閱如何[使用 StorSimple Manager 服務部署 StorSimple 裝置](storsimple-8000-deployment-walkthrough-u2.md)。
* 8000 系列裝置已安裝最新的 StorSimple 8000 系列 Update 5，並且正在執行 6.3.9600.17845 或更新的版本。 如果您的裝置未安裝最新的更新，您需要安裝最新的更新，才能繼續移轉。 如需詳細資訊，請參閱如何[在 8000 系列裝置上安裝最新的更新](storsimple-8000-install-update-5.md)。
* 您的訂用帳戶已啟用，可供移轉使用。 如果訂用帳戶未啟用，請連絡 Microsoft 支援服務啟用訂用帳戶，以便進行移轉。

### <a name="for-the-80108020-cloud-appliance-target"></a>8010/8020 雲端設備 (目標)

開始移轉之前，請確定︰

* 已登錄並執行目標雲端設備。 如需詳細資訊，請參閱如何[部署和管理 StorSimple 雲端設備](storsimple-8000-cloud-appliance-u2.md)。
* 您的雲端設備正在執行最新的 StorSimple 8000 系列 Update 5 軟體版本 6.3.9600.17845。 如果您的雲端設備並未執行 Update 5，請先建立新的 Update 5 雲端設備，再繼續進行移轉。 如需詳細資訊，請參閱如何[建立 8010/8020 雲端設備](storsimple-8000-cloud-appliance-u2.md)。

### <a name="for-the-computer-running-storsimple-migration-tool"></a>對於執行 StorSimple 移轉工具的電腦

StorSimple 移轉工具是 UI 型工具，可讓您將 StorSimple 5000-7000 系列的資料移轉到 8000 系列裝置上。 若要安裝 StorSimple 移轉工具，請使用符合下列需求的電腦。

電腦具有網際網路連線，而且：

* 執行下列作業系統
    * Windows 10。
    * Windows Server 2012 R2 (或更新版本)，才能安裝 StorSimple 移轉工具。
* 已安裝.NET 4.5.2。
* 至少有 5 GB 的可用空間可用於安裝和使用該工具。

> [!TIP]
> 如果您的 StorSimple 裝置連接到 Windows Server 主機，您可以在 Windows Server 主機電腦上安裝移轉工具。

#### <a name="to-install-storsimple-migration-tool"></a>若要安裝 StorSimple 移轉工具

執行下列步驟，在電腦上安裝 StorSimple 移轉工具。

1. 將資料夾 _StorSimple8000SeriesMigrationTool_ 複製到 Windows 電腦。 確定複製軟體的目的地磁碟機具有足夠的空間。

    開啟資料夾中的工具設定檔 _StorSimple8000SeriesMigrationTool.exe.config_。 以下是檔案的程式碼片段。
    
    ```
        <add key="UserName" value="username@xyz.com" />
        <add key="SubscriptionName" value="YourSubscriptionName" />
        <add key="SubscriptionId" value="YourSubscriptionId" />
        <add key="TenantId" value="YourTenantId" />
        <add key="ResourceName" value="YourResourceName" />
        <add key="ResourceGroupName" value="YourResourceGroupName" />

    ```
2. 編輯與索引鍵對應的值，並取代為：

    * `UserName` – 登入 Azure 入口網站的使用者名稱。
    * `SubscriptionName and SubscriptionId` – Azure 訂用帳戶的名稱和識別碼。 在 StorSimple 裝置管理員服務登陸頁面的 [一般] 下，按一下 [屬性]。 複製與您的服務相關聯的訂用帳戶名稱訂用帳戶識別碼。
    * `ResourceName` – Azure 入口網站中的 StorSimple 裝置管理員服務名稱。 也會顯示在服務屬性之下。
    * `ResourceGroup` – 與 Azure 入口網站的 StorSimple 裝置管理員服務相關聯的資源群組名稱。 也會顯示在服務屬性之下。
    ![檢查目標裝置的服務屬性](media/storsimple-8000-migrate-from-5000-7000/check-service-properties1.png)
    * `TenantId` – Azure 入口網站中的 Azure Active Directory 租用戶識別碼。 以系統管理員身分登入 Microsoft Azure。 在 Microsoft Azure 入口網站中，按一下 **Azure Active Directory**。 在 [管理] 下，按一下 [屬性]。 租用戶識別碼會顯示於 [目錄識別碼]。
    ![檢查 Azure Active Directory 的租用戶識別碼](media/storsimple-8000-migrate-from-5000-7000/check-tenantid-aad.png)

3.  儲存對設定檔所做的變更。
4.  執行 _StorSimple8000SeriesMigrationTool.exe_ 啟動工具。 出現輸入認證的提示時，請提供 和 Azure 入口網站訂用帳戶相關聯的認證。 
5.  StorSimple 移轉工具 UI 隨即顯示。
  

## <a name="next-steps"></a>後續步驟
下載如何[將 StorSimple 5000-7000 系列的資料移轉到 8000 系列裝置](https://gallery.technet.microsoft.com/Azure-StorSimple-50007000-c1a0460b)的逐步指示。
