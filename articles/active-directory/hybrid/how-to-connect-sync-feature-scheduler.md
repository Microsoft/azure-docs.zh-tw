---
title: Azure AD Connect 同步：排程器 | Microsoft Docs
description: 本主題說明 Azure AD Connect 同步處理中內建的排程器功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 6b1a598f-89c0-4244-9b20-f4aaad5233cf
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/12/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: d8deb03d03446c1452d73a7c08df4cf14ffcd5b5
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46304557"
---
# <a name="azure-ad-connect-sync-scheduler"></a>Azure AD Connect 同步處理：排程器
本主題說明 Azure AD Connect 同步處理 (又稱為 同步處理引擎) 中內建的排程器。

此功能是隨組建 1.1.105.0 (於 2016 年 2 月發行) 一起導入。

## <a name="overview"></a>概觀
Azure AD Connect 同步處理會使用排程器來同步處理您內部部署目錄中發生的變更。 有兩個排程器程序，一個用於密碼同步處理，另一個用於物件/屬性同步處理以及維護工作。 本主題包含後者。

在舊版中，物件和屬性的排程器是在同步處理引擎的外部。 它使用 Windows 工作排程器或個別的 Windows 服務來觸發同步處理程序。 排程器已隨 1.1 版內建於同步處理引擎中，並且允許進行一些自訂。 新的預設同步處理頻率為 30 分鐘。

排程器負責兩項工作：

* **同步處理循環**。 將變更匯入、同步處理及匯出的程序。
* **維護工作**。 為密碼重設和「裝置註冊服務」(DRS) 更新金鑰和憑證。 清除作業記錄檔中的舊項目。

排程器本身永遠處於執行狀態，但是您可以將它設定為只執行這些工作其中之一或完全不執行這些工作。 例如，如果您需要使用自己的同步處理循環程序，您可以將此工作在排程器中停用，但仍執行維護工作。

## <a name="scheduler-configuration"></a>排程器組態
若要查看目前的組態設定，請移至 PowerShell 並執行 `Get-ADSyncScheduler`。 它會向您顯示類似以下圖片︰

![GetSyncScheduler](./media/how-to-connect-sync-feature-scheduler/getsynccyclesettings2016.png)

如果您在執行這個 Cmdlet 時看到 **無法使用同步命令或 Cmdlet** ，則表示 PowerShell 模組並未載入。 如果您在網域控制站或 PowerShell 限制層級高於預設設定的伺服器上執行 Azure AD Connect，即會發生此問題。 如果您看到此錯誤，則請執行 `Import-Module ADSync` ，以使 Cmdlet 可供使用。

* **AllowedSyncCycleInterval**。 Azure AD 所允許的同步處理週期最短時間間隔。 同步處理頻率一旦超過此設定，即不受支援。
* **CurrentlyEffectiveSyncCycleInterval**。 目前作用中的排程。 如果頻率未高於 AllowedSyncInterval，其值就會與 CustomizedSyncInterval (如果已設定) 相同。 如果您使用 1.1.281 之前的組件且變更 CustomizedSyncCycleInterval，將會在下一個同步處理循環後才生效。 從組建 1.1.281 起，變更立即生效。
* **CustomizedSyncCycleInterval**。 如果您想要讓排程器以預設值 30 分鐘以外的任何其他頻率執行，那麼您要設定此設定。 在上圖中，已將排程器改為設定成每小時執行一次。 如果您將此設定設定成低於 AllowedSyncInterval 的值，則會使用後者。
* **NextSyncCyclePolicyType**。 Delta (差異) 或 Initial (初始)。 定義下一次執行應該只處理差異變更，還是應該進行完整匯入並同步處理。後者會一併重新處理任何新的或已變更的規則。
* **NextSyncCycleStartTimeInUTC**。 排程器下一次啟動下一個同步處理循環的時間。
* **PurgeRunHistoryInterval**。 應該保留作業記錄的時間。 您可以在同步處理服務管理員中檢閱這些記錄。 這些記錄預設會保留 7 天。
* **SyncCycleEnabled**。 指出排程器是否要在其作業中一併執行匯入、同步處理及匯出程序。
* **MaintenanceEnabled**。 顯示是否已啟用維護程序。 它會更新憑證/金鑰，並清除作業記錄。
* **StagingModeEnabled**。 顯示是否已啟用 [預備模式](how-to-connect-sync-operations.md#staging-mode) 。 如果已啟用此設定，則它會停用執行匯出，但仍執行匯入和同步處理。
* **SchedulerSuspended**。 由 Connect 在升級時設定，用來防止排程器執行。

您可以使用 `Set-ADSyncScheduler`變更這些設定的其中一部分。 您可以修改下列參數：

* CustomizedSyncCycleInterval
* NextSyncCyclePolicyType
* PurgeRunHistoryInterval
* SyncCycleEnabled
* MaintenanceEnabled

在先前的 Azure AD Connect 組建中，**isStagingModeEnabled** 已在 Set-ADSyncScheduler 中公開。 **不支援**設定此屬性。 **SchedulerSuspended** 屬性應該僅供 Connect 修改。 **不支援**直接使用 PowerShell 來修改此屬性。

排程器組態儲存在 Azure AD 中。 如果您有預備伺服器，主要伺服器上的任何變更也會影響預備伺服器 (IsStagingModeEnabled 除外)。

### <a name="customizedsynccycleinterval"></a>CustomizedSyncCycleInterval
語法： `Set-ADSyncScheduler -CustomizedSyncCycleInterval d.HH:mm:ss`  
d - 天、HH - 小時、mm - 分鐘、ss - 秒

範例： `Set-ADSyncScheduler -CustomizedSyncCycleInterval 03:00:00`  
將排程器變更為每隔 3 小時執行一次。

範例： `Set-ADSyncScheduler -CustomizedSyncCycleInterval 1.0:0:0`  
將排程器變更為每天執行。

### <a name="disable-the-scheduler"></a>停用排程器  
如果您需要變更組態，那麼您需要停用排程器。 例如，當您[設定篩選](how-to-connect-sync-configure-filtering.md)或[變更同步處理規則](how-to-connect-sync-change-the-configuration.md)時。

若要停用排程器，請執行 `Set-ADSyncScheduler -SyncCycleEnabled $false`。

![停用排程器](./media/how-to-connect-sync-feature-scheduler/schedulerdisable.png)

當您完成變更之後，不要忘記使用 `Set-ADSyncScheduler -SyncCycleEnabled $true` 再次啟用排程器。

## <a name="start-the-scheduler"></a>啟動排程器
排程器預設為每隔 30 分鐘執行一次。 在某些情況下，您可能會想要在排定的循環之間執行同步處理循環，或是需要執行不同類型的同步處理循環。

**差異同步處理循環**  
：差異同步處理循環包含下列步驟：

* 在所有的連接器上執行差異匯入
* 在所有的連接器上執行差異同步處理
* 在所有的連接器上執行匯出

您可能因為有一個必須立即同步處理的緊急變更，而需要手動執行循環。 如果您需要手動執行循環，則請從 PowerShell 執行 `Start-ADSyncSyncCycle -PolicyType Delta`。

**完整同步處理循環**  
如果您已進行下列其中一項組態變更，就需要執行完整同步處理循環 (也稱為 Initial (初始))：

* 新增更多要從來源目錄匯入的物件或屬性
* 對同步處理規則進行變更
* 變更 [篩選](how-to-connect-sync-configure-filtering.md) 以納入不同數目的物件

如果您已進行上述其中一項變更，您就需要執行完整同步處理循環，以便讓同步處理引擎有機會重新合併連接器空間。 完整同步處理循環包含下列步驟：

* 在所有的連接器上執行完整匯入
* 在所有的連接器上執行完整同步處理
* 在所有的連接器上執行匯出

若要起始完整同步處理循環，請從 PowerShell 提示字元執行 `Start-ADSyncSyncCycle -PolicyType Initial` 。 此命令會啟動完整同步處理循環。

## <a name="stop-the-scheduler"></a>停止排程器
如果排程器目前正在執行同步處理循環，您可能必須將它停止。 例如，如果您啟動安裝精靈並收到以下錯誤：

![SyncCycleRunningError](./media/how-to-connect-sync-feature-scheduler/synccyclerunningerror.png)

當同步處理循環正在執行時，您會無法進行組態變更。 您可以等到排程器完成程序，也可以停止它，以便立即進行變更。 停止目前的循環並無任何損害，而且擱置的變更會在下一次執行時處理。

1. 首先，使用 PowerShell Cmdlet `Stop-ADSyncSyncCycle`來通知排程器停止其目前的循環。
2. 如果您使用 1.1.281 之前的組件，停止排程器並不會阻止目前的連接器進行其目前的工作。 若要強制停止連接器，請採取下列動作：![StopAConnector](./media/how-to-connect-sync-feature-scheduler/stopaconnector.png)
   * 從 [開始] 功能表啟動 [同步處理服務]  。 移至 [連接器]，反白選取狀態為 [正在執行] 的連接器，然後從 [動作] 中選取 [停止]。

排程器仍在作用中，並且會在下次有機會時重新啟動。

## <a name="custom-scheduler"></a>自訂排程器
本節記載的 Cmdlet 只有在組建 [1.1.130.0](reference-connect-version-history.md#111300) 和更新版本中才有提供。

如果內建的排程器無法滿足您的需求，您可以使用 PowerShell 來排程連接器。

### <a name="invoke-adsyncrunprofile"></a>Invoke-ADSyncRunProfile
您可使用下列方式，啟動連接器的設定檔：

```
Invoke-ADSyncRunProfile -ConnectorName "name of connector" -RunProfileName "name of profile"
```

您可以在 [Synchronization Service Manager UI](how-to-connect-sync-service-manager-ui.md) 中找到可用來做為[連接器名稱](how-to-connect-sync-service-manager-ui-connectors.md)和[執行設定檔名稱](how-to-connect-sync-service-manager-ui-connectors.md#configure-run-profiles)的名稱。

![叫用執行設定檔](./media/how-to-connect-sync-feature-scheduler/invokerunprofile.png)  

`Invoke-ADSyncRunProfile` Cmdlet 是同步的，亦即只有當連接器完成作業 (不論成功或發生錯誤) 時，它才會交回控制權。

排程連接器時，建議您遵循下列順序進行排程︰

1. (完整/差異) 從內部部署目錄匯入，例如 Active Directory
2. (完整/差異) 從 Azure AD 匯入
3. (完整/差異) 從內部部署目錄同步處理，例如 Active Directory
4. (完整/差異) 從 Azure AD 同步處理
5. 匯出至 Azure AD
6. 匯出至內部部署目錄，例如 Active Directory

此順序是內建的排程器執行連接器的方式。

### <a name="get-adsyncconnectorrunstatus"></a>Get-ADSyncConnectorRunStatus
您也可以監視同步處理引擎，以查看其為忙碌或閒置。 如果同步處理引擎處於閒置狀態且沒有執行連接器，這個 Cmdlet 會傳回空的結果。 如果連接器正在執行，它會傳回連接器的名稱。

```
Get-ADSyncConnectorRunStatus
```

![連接器執行狀態](./media/how-to-connect-sync-feature-scheduler/getconnectorrunstatus.png)  
在上圖中，第一行是來自同步處理引擎處於閒置狀態時。 第二行則是從 Azure AD 連接器執行時。

## <a name="scheduler-and-installation-wizard"></a>排程器和安裝精靈
如果您啟動安裝精靈，則排程器將會暫時停用。 此行為是因為系統會假設您進行組態變更，而如果同步處理引擎正在執行中，將會無法套用這些設定。 基於這個理由，請勿讓安裝精靈保持開啟，因為這會阻止同步處理引擎執行任何同步處理動作。

## <a name="next-steps"></a>後續步驟
深入了解 [Azure AD Connect 同步](how-to-connect-sync-whatis.md) 組態。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
