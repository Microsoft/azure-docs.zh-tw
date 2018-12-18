---
title: Azure Active Directory 中的 Office 365 群組到期日 | Microsoft Docs
description: 如何為 Azure Active Directory 中的 Office 365 群組設定到期日
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 03/09/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 29a53101bff8c384d01f952c4498e09d9d970ee3
ms.sourcegitcommit: 3d0295a939c07bf9f0b38ebd37ac8461af8d461f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43841729"
---
# <a name="configure-the-expiration-policy-for-office-365-groups"></a>為 Office 365 群組設定到期原則

您現在可以為 Office 365 群組設定到期原則，來管理這些群組的生命週期。 您只能為 Azure Active Directory (Azure AD) 中的 Office 365 群組設定到期日。 

一旦您為群組設定到期日：
-   群組擁有者會在到期日快到時收到更新群組的通知
-   未更新的群組會遭到刪除
-   群組擁有者或系統管理員可在 30 天內還原已刪除的任何 Office 365 群組

> [!NOTE]
> 若要針對 Office 365 群組設定及使用到期原則，您必須具有套用到期原則之所有群組成員的現成 Azure AD Premium 授權。

如需有關如何下載及安裝 Azure AD PowerShell Cmdlet 的資訊，請參閱 [Azure Active Directory PowerShell for Graph 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.137)。

## <a name="roles-and-permissions"></a>角色和權限
以下是可以針對 Azure AD 中的 Office 365 群組設定及使用到期日的角色。

角色 | 權限
-------- | --------
全域管理員或使用者帳戶管理員 | 可以建立、讀取、更新或刪除 Office 365 群組到期日原則設定<br>可以更新任何 Office 365 群組
使用者 | 可以更新它們所擁有的 Office 365 群組<br>可以還原它們所擁有的 Office 365 群組<br>可以讀取到期原則設定

如需有關將已刪除群組還原之權限的詳細資訊，請參閱[在 Azure Active Directory 中還原已刪除的 Office 365 群組](groups-restore-deleted.md)。

## <a name="set-group-expiration"></a>設定群組到期日

1. 開啟其帳戶在 Azure AD 租用戶中為全域系統管理員的 [Azure AD 系統管理中心](https://aad.portal.azure.com)。

2. 選取 [群組]，然後選取 [到期]，以開啟到期設定。
  
  ![[到期] 刀鋒視窗](./media/groups-lifecycle/expiration-settings.png)

4. 在 [到期] 刀鋒視窗中，您可以：

  * 設定群組的存留期 (以天為單位)。 您可以選取其中一個預設值或自訂值 (應為 31 天或更多)。 
  * 指定當群組沒有擁有者時應該傳送續訂和到期通知的電子郵件地址。 
  * 選取到期的 Office 365 群組。 您可以啟用**所有** Office 365 群組的到期日，也可以選擇僅啟用**已選取**的 Office 365 群組，或選取 [無] 以停用所有群組的到期日。
  * 當您完成時，選取 [儲存] 會儲存您的設定。


這種電子郵件通知會在群組到期前 30 天、前 15 天和前 1 天傳送給 Office 365 群組擁有者。

![到期電子郵件通知](./media/groups-lifecycle/expiration-notification.png)

群組擁有者可以從**續約群組**通知電子郵件，直接存取「存取面板」中的群組詳細資料頁面。 使用者可以從該處取得群組的相關詳細資訊，例如其描述、其最後更新時間、其到期時間，還能夠更新群組。 群組詳細資料頁面現在也包含 Office 365 群組資源的連結，可方便群組擁有者檢視其群組中的內容和活動。

當群組到期時，會在到期日隔天刪除群組。 這種電子郵件通知會傳送給 Office 365 群組擁有者，通知他們 Office 365 群組的到期日和後續刪除。

![群組刪除電子郵件通知](./media/groups-lifecycle/deletion-notification.png)

在刪除群組後的 30 天內，您可以選取 [還原群組] 或使用 PowerShell Cmdlet 來還原群組，如[在 Azure Active Directory 中還原已刪除的 Office 365 群組](groups-restore-deleted.md)所述。
    
如果您要還原的群組包含文件、SharePoint 網站或其他持續物件，則可能需要 24 小時，才能完全還原群組及其內容。

> [!NOTE]
> * 當您第一次設定到期日時，任何早於到期間隔的群組會設定為 30 後到期。 第一封續訂通知電子郵件會在第一天發出。 
>   例如，400 天前已建立群組 A，而且到期間隔設定為 180 天。 您套用了到期設定，除非擁有者續訂群組 A，不然該群組就會在 30 天後刪除。
> * 目前在一個租用戶上，只能為 Office 365 群組設定一個到期原則。
> * 刪除並還原動態群組時，會將其視為新群組，並根據規則重新填入。 此流程可能需要多達 24 小時的時間。

## <a name="how-office-365-group-expiration-works-with-a-mailbox-on-legal-hold"></a>Office 365 群組到期如何與法務保留中的信箱搭配運作
當群組到期而被刪除時，在刪除後的 30 天之後，系統就會將來自應用程式 (例如「行事曆」、「網站」或「小組」) 的群組資料永久刪除，但是會保留法務保留中的群組信箱，而不會永久刪除。 系統管理員可以使用 Exchange Cmdlet 來還原信箱以擷取資料。 

## <a name="how-office-365-group-expiration-works-with-retention-policy"></a>Office 365 群組到期如何與保留原則搭配運作
保留原則是透過「安全性與合規性中心」來設定的。 如果您已經為 Office 365 群組設定保留原則，當群組到期而被刪除時，系統會根據保留原則中所定義的特定天數，將群組信箱中的群組對話及群組網站中的檔案會保留在保留容器中。 在群組到期後，使用者將看不到該群組及其內容，但可以透過電子文件探索來復原網站和信箱資料。

## <a name="powershell-examples"></a>PowerShell 範例
以下是一些範例，說明如何使用 PowerShell Cmdlet 來為您租用戶中的 Office 365 群組設定到期設定：

1. 安裝 PowerShell v2.0 預覽模組 (2.0.0.137) 並在 PowerShell 提示字元登入：
  ````
  Install-Module -Name AzureADPreview
  connect-azuread 
  ````
2. 設定到期設定 New-AzureADMSGroupLifecyclePolicy：此 Cmdlet 會將租用戶中所有 Office 365 群組的存留期設定為 365 天。 無擁有者之 Office 365 群組的續約通知會傳送給 ‘emailaddress@contoso.com’
  
  ````
  New-AzureADMSGroupLifecyclePolicy -GroupLifetimeInDays 365 -ManagedGroupTypes All -AlternateNotificationEmails emailaddress@contoso.com
  ````
3. 擷取現有的原則 Get-AzureADMSGroupLifecyclePolicy：此 Cmdlet 會擷取目前已設定的 Office 365 群組到期設定。 在此範例中，您可以看見：
  * 原則識別碼 
  * 租用戶中所有 Office 365 群組的存留期已設定為 365 天
  * 無擁有者之 Office 365 群組的續約通知會傳送給 ‘emailaddress@contoso.com’。
  
  ````
  Get-AzureADMSGroupLifecyclePolicy
  
  ID                                    GroupLifetimeInDays ManagedGroupTypes AlternateNotificationEmails
  --                                    ------------------- ----------------- ---------------------------
  26fcc232-d1c3-4375-b68d-15c296f1f077  365                 All               emailaddress@contoso.com
  ```` 
   
4. 更新現有的原則 Set-AzureADMSGroupLifecyclePolicy：此 Cmdlet 可用來更新現有的原則。 在以下範例中，現有原則中的群組存留期會從 365 天變更為 180 天。 
  
  ````
  Set-AzureADMSGroupLifecyclePolicy -Id “26fcc232-d1c3-4375-b68d-15c296f1f077”   -GroupLifetimeInDays 180 -AlternateNotificationEmails "emailaddress@contoso.com"
  ````
  
5. 將特定群組新增至原則 Add-AzureADMSLifecyclePolicyGroup：此 Cmdlet 會將群組新增至生命週期原則。 例如： 
  
  ````
  Add-AzureADMSLifecyclePolicyGroup -Id “26fcc232-d1c3-4375-b68d-15c296f1f077” -groupId "cffd97bd-6b91-4c4e-b553-6918a320211c"
  ````
  
6. 移除現有的原則 Remove-AzureADMSGroupLifecyclePolicy：此 Cmdlet 會刪除 Office 365 群組到期設定，但需要原則識別碼。 這會停用 Office 365 群組的到期日。 
  
  ````
  Remove-AzureADMSGroupLifecyclePolicy -Id “26fcc232-d1c3-4375-b68d-15c296f1f077”
  ````
  
下列 Cmdlet 可用來更詳細地設定原則。 如需詳細資訊，請參閱 [PowerShell 文件](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview&branch=master#groups) \(英文\)。

* Get-AzureADMSGroupLifecyclePolicy
* New-AzureADMSGroupLifecyclePolicy
* Get-AzureADMSGroupLifecyclePolicy
* Set-AzureADMSGroupLifecyclePolicy
* Remove-AzureADMSGroupLifecyclePolicy
* Add-AzureADMSLifecyclePolicyGroup
* Remove-AzureADMSLifecyclePolicyGroup
* Reset-AzureADMSLifeCycleGroup   
* Get-AzureADMSLifecyclePolicyGroup

## <a name="next-steps"></a>後續步驟
這些文章提供有關 Azure AD 群組的其他資訊。

* [查看現有的群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [管理群組的設定](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理群組的成員](../fundamentals/active-directory-groups-members-azure-portal.md)
* [管理群組的成員資格](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理群組中使用者的動態規則](groups-dynamic-membership.md)
