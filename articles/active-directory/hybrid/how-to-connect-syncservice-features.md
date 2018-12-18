---
title: Azure AD Connect 同步服務功能與組態 | Microsoft Docs
description: 描述 Azure AD Connect 同步處理服務的服務端功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 213aab20-0a61-434a-9545-c4637628da81
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/25/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: fa7d0f98f8119ad34e0764423e99af4564f20387
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46305847"
---
# <a name="azure-ad-connect-sync-service-features"></a>Azure AD Connect 同步處理服務功能
Azure AD connect 同步處理功能有兩個元件：

* 名為 **Azure AD Connect 同步處理**的內部部署元件，也稱為**同步處理引擎**。
* 位於 Azure AD 的服務，也稱為 **Azure AD Connect 同步處理服務**

本主題說明 **Azure AD Connect 同步處理服務** 的下列功能如何運作，以及如何使用 Windows PowerShell 進行設定。

這些設定是由 [適用於 Windows PowerShell 的 Azure Active Directory 模組](https://aka.ms/aadposh)所設定。 從 Azure AD Connect 個別進行下載和安裝。 本主題說明的 Cmdlet 已導入 [2016 年 3 月版本 (組建 9031.1)](http://social.technet.microsoft.com/wiki/contents/articles/28552.microsoft-azure-active-directory-powershell-module-version-release-history.aspx#Version_9031_1)。 如果您沒有本主題說明的 Cmdlet，或它們未產生相同的結果，請確認您執行的是最新版本。

若要查看 Azure AD 目錄中的組態，請執行 `Get-MsolDirSyncFeatures`。  
![Get-MsolDirSyncFeatures 結果](./media/how-to-connect-syncservice-features/getmsoldirsyncfeatures.png)

其中許多設定只能由 Azure AD Connect 進行變更。

以下是可在 `Set-MsolDirSyncFeature`中配置的設定：

| DirSyncFeature | 註解 |
| --- | --- |
| [EnableSoftMatchOnUpn](#userprincipalname-soft-match) |不只主要 SMTP 位址，還允許物件聯結 userPrincipalName。 |
| [SynchronizeUpnForManagedUsers](#synchronize-userprincipalname-updates) |可讓同步處理引擎更新受控/授權 (非同盟) 使用者的 userPrincipalName 屬性。 |

啟用功能後，即無法將其停用。

> [!NOTE]
> 從 2016 年 8 月 24 日起，預設會對新的 Azure AD 目錄啟用*重複屬性恢復*功能。 這項功能也會在此日期前建立的目錄上推出和啟用。 當您的目錄即將啟用此功能時，您會收到電子郵件通知。
> 
> 

下列設定是由 Azure AD Connect 所設定，而且無法由 `Set-MsolDirSyncFeature`修改：

| DirSyncFeature | 註解 |
| --- | --- |
| DeviceWriteback |[Azure AD Connect：啟用裝置回寫](how-to-connect-device-writeback.md) |
| DirectoryExtensions |[Azure AD Connect 同步處理：目錄擴充](how-to-connect-sync-feature-directory-extensions.md) |
| [DuplicateProxyAddressResiliency<br/>DuplicateUPNResiliency](#duplicate-attribute-resiliency) |如果屬性是另一個物件的複本，即會將該屬性隔離，而不會在匯出期間導致整個物件失敗。 |
| 密碼雜湊同步處理 |[使用 Azure AD Connect 同步來實作密碼雜湊同步處理](how-to-connect-password-hash-synchronization.md) \(英文\) |
|傳遞驗證|[使用 Azure Active Directory 傳遞驗證來進行使用者登入](how-to-connect-pta.md)|
| UnifiedGroupWriteback |[預覽：群組回寫](how-to-connect-preview.md#group-writeback) |
| UserWriteback |目前不支援。 |

## <a name="duplicate-attribute-resiliency"></a>重複屬性恢復功能
含重複 UPNs / proxyAddresses 的物件，會將該屬性「隔離」，並指派暫時的值，而不會讓佈建物件失敗。 解決衝突時，會自動將暫時 UPN 變更為適當的值。 如需詳細資訊，請參閱 [身分識別同步處理和重複屬性恢復功能](how-to-connect-syncservice-duplicate-attribute-resiliency.md)。

## <a name="userprincipalname-soft-match"></a>UserPrincipalName 大致比對
啟用這項功能後，除了一律啟用的 [主要 SMTP 位址](https://support.microsoft.com/kb/2641663)以外，還會對 UPN 套用大致比對。 大致比對功能是用來針對 Azure AD 中現有的雲端使用者與內部部署使用者進行比對。

如果您需要比對內部部署 AD 帳戶與雲端中建立的現有帳戶，卻不能使用 Exchange Online 時，啟用此功能很有用。 在此案例中，您通常不需要在雲端中設定 SMTP 屬性。

在新建立的 Azure AD 目錄中，預設會開啟這項功能。 您可以執行下列項目，查看是否已啟用此功能︰  

```
Get-MsolDirSyncFeatures -Feature EnableSoftMatchOnUpn
```

如果您的 Azure AD 目錄未啟用這項功能，您可以執行下列項目加以啟用︰  

```
Set-MsolDirSyncFeature -Feature EnableSoftMatchOnUpn -Enable $true
```

## <a name="synchronize-userprincipalname-updates"></a>同步處理 userPrincipalName 更新
在過去，除非下列兩項條件都成立，否則皆會將透過內部部署使用同步處理服務的 UserPrincipalName 屬性更新封鎖：

* 使用者受到管理 (非同盟)。
* 使用者尚未指派授權。

如需詳細資訊，請參閱 [Office 365、Azure 或 Intune 中的使用者名稱不符合內部部署的 UPN 或替代登入識別碼](https://support.microsoft.com/kb/2523192)。

當內部部署中的 userPrincipalName 有所變更，且您使用密碼雜湊同步處理時，啟用此功能可讓同步處理引擎更新 userPrincipalName。如果您使用同盟，則不支援這項功能。

在新建立的 Azure AD 目錄中，預設會開啟這項功能。 您可以執行下列項目，查看是否已啟用此功能︰  

```
Get-MsolDirSyncFeatures -Feature SynchronizeUpnForManagedUsers
```

如果您的 Azure AD 目錄未啟用這項功能，您可以執行下列項目加以啟用︰  

```
Set-MsolDirSyncFeature -Feature SynchronizeUpnForManagedUsers -Enable $true
```

啟用這項功能之後，現有的 userPrincipalName 值會保持不變。 下次 userPrincipalName 屬性的內部部署變更時，使用者的一般差異同步處理即會更新 UPN。  

## <a name="see-also"></a>另請參閱
* [Azure AD Connect 同步處理](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。

