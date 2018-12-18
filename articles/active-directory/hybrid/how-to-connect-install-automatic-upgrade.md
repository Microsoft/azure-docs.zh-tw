---
title: Azure AD Connect：自動升級 | Microsoft Docs
description: 本主題說明 Azure AD Connect 同步處理中內建的自動升級功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 6b395e8f-fa3c-4e55-be54-392dd303c472
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/26/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 72503c9f073a046b81001322cdbb4fb8a1af76cf
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47224115"
---
# <a name="azure-ad-connect-automatic-upgrade"></a>Azure AD Connect：自動升級
此功能已隨組建 [1.1.105.0 (於 2016 年 2 月發行)](reference-connect-version-history.md#111050) 一起推出。  這項功能已在[組建 1.1.561](reference-connect-version-history.md#115610) 中更新，且現在支援先前未支援的其他案例。

## <a name="overview"></a>概觀
使用「自動升級」  功能是可確保您 Azure AD Connect 安裝永遠維持在最新狀態的最簡單方式。 此功能預設為啟用，以供進行快速安裝及 DirSync 升級。 新版本發行時，您的安裝會自動升級。
預設會針對下列情況啟用自動升級：

* 快速設定安裝和 DirSync 升級。
* 使用 SQL Express LocalDB (這是「快速設定」一律使用的)。 使用 SQL Express 的 DirSync 也會使用 LocalDB。
* AD 帳戶是「快速設定」和 DirSync 所建立的預設 MSOL_ 帳戶。
* Metaverse 中的物件少於 100,000 個

您可以使用 PowerShell Cmdlet `Get-ADSyncAutoUpgrade`來檢視目前的自動升級狀態。 其狀態如下：

| State | 註解 |
| --- | --- |
| 已啟用 |已啟用自動升級。 |
| 暫止 |只有系統才能設定。 系統**目前沒有**資格再接收自動升級。 |
| 已停用 |已停用自動升級。 |

您可以使用 `Set-ADSyncAutoUpgrade` 在 [已啟用] 與 [已停用] 之間進行變更。 應該只有系統才能設定 [已暫止] 狀態。

自動升級使用 Azure AD Connect Health 做為升級基礎結構。 為了讓自動升級能夠運作，請確定您已依照 **Office 365 URL 與 IP 位址範圍** 中的記載，在您 Proxy 伺服器中開啟 [Azure AD Connect Health](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2)的 URL。


如果伺服器上正在執行 **Synchronization Service Manager** UI，則會暫止升級，直到 UI 關閉為止。

## <a name="troubleshooting"></a>疑難排解
如果您的 Connect 安裝未如預期般自動升級，請遵循下列步驟來找出可能的錯誤。

首先，不建議您在新版本發行的第一天就自動升級。 由於升級前有刻意設計的隨機性，因此不用擔心您的安裝沒有立即升級。

如果您認為有問題，請先執行 `Get-ADSyncAutoUpgrade` 確保已啟用自動升級。

然後，確定您已在您的 Proxy 或防火牆中開啟所需的 URL。 自動更新會如 [概觀](#overview)所述使用 Azure AD Connect Health。 如果您使用 Proxy，請確定 Health 已設定為使用 [roxy 伺服器](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy)。 而且測試對 Azure AD 的 [Health 連線](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service) 。

透過對已驗證 Azure AD 的連線，即可查看事件記錄檔。 啟動事件檢視器，並查看 **應用程式** 事件記錄。 新增來源 **Azure AD Connect 升級**的事件記錄篩選以及事件識別碼範圍 **300-399**。  
![自動升級的事件記錄篩選](./media/how-to-connect-install-automatic-upgrade/eventlogfilter.png)  

您現在可以看到與自動升級狀態關聯的事件記錄。  
![自動升級的事件記錄篩選](./media/how-to-connect-install-automatic-upgrade/eventlogresult.png)  

結果碼前面會有包含狀態概觀的前置詞。

| 結果碼前置詞 | 說明 |
| --- | --- |
| 成功 |安裝已順利升級。 |
| UpgradeAborted |發生暫時狀況導致升級停止。 它將會重試一次，而且預期稍後成功。 |
| UpgradeNotSupported |系統具有封鎖自動升級系統的組態。 它將會重試以查看狀態是否已變更，但預期情況是系統必須手動升級。 |

以下是最常見的訊息清單。 清單不完整，但結果訊息應該清楚顯示問題所在。

| 結果訊息 | 說明 |
| --- | --- |
| **UpgradeAborted** | |
| UpgradeAbortedCouldNotSetUpgradeMarker |無法寫入登錄。 |
| UpgradeAbortedInsufficientDatabasePermissions |內建的系統管理員群組沒有資料庫的權限。 請手動升級至最新版的 Azure AD Connect 來解決此問題。 |
| UpgradeAbortedInsufficientDiskSpace |沒有足夠的光碟空間，以支援升級。 |
| UpgradeAbortedSecurityGroupsNotPresent |找不到且無法解析同步處理引擎使用的所有安全性群組。 |
| UpgradeAbortedServiceCanNotBeStarted |NT 服務 **Microsoft Azure AD Sync** 無法啟動。 |
| UpgradeAbortedServiceCanNotBeStopped |NT 服務 **Microsoft Azure AD Sync** 無法停止。 |
| UpgradeAbortedServiceIsNotRunning |NT 服務 **Microsoft Azure AD Sync** 並未執行。 |
| UpgradeAbortedSyncCycleDisabled |[排程器](how-to-connect-sync-feature-scheduler.md) 中的 SyncCycle 選項已停用。 |
| UpgradeAbortedSyncExeInUse |伺服器上的 [Synchronization Service Manager UI](how-to-connect-sync-service-manager-ui.md) 為開啟。 |
| UpgradeAbortedSyncOrConfigurationInProgress |安裝精靈正在執行或排程器外部已排定同步處理。 |
| **UpgradeNotSupported** | |
| UpgradeNotSupportedAdfsSignInMethod | 您已選取 ADFS 作為登入方法。 | 
| UpgradeNotSupportedCustomizedSyncRules |您已將自己的自訂規則加入組態。 |
| UpgradeNotSupportedDeviceWritebackEnabled |您已啟用 [裝置回寫](how-to-connect-device-writeback.md) 功能。 |
| UpgradeNotSupportedGroupWritebackEnabled |您已啟用 [群組回寫](how-to-connect-preview.md#group-writeback) 功能。 |
| UpgradeNotSupportedInvalidPersistedState |安裝不是快速設定或 DirSync 升級。 |
| UpgradeNotSupportedMetaverseSizeExceeeded |Metaverse 中的物件超過 100,000 個。 |
| UpgradeNotSupportedMultiForestSetup |您正連接到多個樹系。 快速安裝只會連接到一個樹系。 |
| UpgradeNotSupportedNonLocalDbInstall |您不是使用 SQL Server Express LocalDB 資料庫。 |d
| UpgradeNotSupportedNonMsolAccount |[AD DS 連接器帳戶](reference-connect-accounts-permissions.md#ad-ds-connector-account)已不再是預設的 MSOL_ 帳戶。 |
| UpgradeNotSupportedNotConfiguredSignInMethod | 若要設定 AAD Connect，請在選取登入方法時，選擇 [不設定]。 | 
| UpgradeNotSupportedPtaSignInMethod | 您已選取 [傳遞驗證] 作為登入方法。 |
| UpgradeNotSupportedStagingModeEnabled |伺服器設定為 [預備模式](how-to-connect-sync-operations.md#staging-mode)。 |
| UpgradeNotSupportedUserWritebackEnabled |您已啟用 [使用者回寫](how-to-connect-preview.md#user-writeback) 功能。 |

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
