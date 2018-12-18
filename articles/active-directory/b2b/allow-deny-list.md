---
title: 允許或封鎖對特定組織的 B2B 使用者的邀請 - Azure Active Directory |Microsoft Docs
description: 說明系統管理員如何使用 Azure 入口網站或 PowerShell 設定存取或拒絕清單，以允許或封鎖來自特定網域的 B2B 使用者。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 04/19/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 563b2d6393533a86305213b6cdec6ca901e53257
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45985374"
---
# <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>允許或封鎖對特定組織的 B2B 使用者的邀請

您可以使用允許清單或拒絕清單，以允許或封鎖對特定組織的 B2B 使用者的邀請。 例如，如果您想要封鎖個人電子郵件地址網域，您可以設定包含 Gmail.com 和 Outlook.com 等網域的拒絕清單。 或者，如果您的企業與其他企業 (如 Contoso.com、Fabrikam.com 和 Litware.com) 有合作關係，而您想要將邀請僅限定於這些組織，則您可以將 Contoso.com、Fabrikam.com 和 Litware.com 新增至您的允許清單中。
  
## <a name="important-considerations"></a>重要考量︰

- 您可以建立允許清單或拒絕清單。 您無法同時設定這兩種清單。 根據預設，未列入允許清單的網域即會列於拒絕清單中，反之亦然。 
- 您對每個組織只能建立一個原則。 您可以更新原則以包含多個網域，或者，您可以刪除原則以建立新的原則。 
- 此清單會在商務用 OneDrive 和 SharePoint Online 允許/封鎖清單以外獨立運作。 如果您想要在 SharePoint Online 中限制個別的檔案共用，您必須設定商務用 OneDrive 和 SharePoint Online 的允許或拒絕清單。 如需詳細資訊，請參閱[在 SharePoint Online 和商務用 OneDrive 中限制網域共用](https://support.office.com/article/restricted-domains-sharing-in-sharepoint-online-and-onedrive-for-business-5d7589cd-0997-4a00-a2ba-2320ec49c4e9)。
- 此清單不適用於已兌換邀請的外部使用者。 清單在設定後將會強制執行。 如果使用者邀請處於擱置狀態，而且您設定了封鎖其網域的原則，則使用者兌換邀請的嘗試將會失敗。

## <a name="set-the-allow-or-deny-list-policy-in-the-portal"></a>在入口網站中設定允許或拒絕清單原則

依預設會啟用**允許邀請傳送至任何網域 (涵蓋性最高)** 設定。 在此情況下，您可以邀請任何組織的 B2B 使用者。

### <a name="add-a-deny-list"></a>新增拒絕清單

這是最常見的情況，也就是，您的組織想要與絕大多數的組織合作，但需要防止來自特定網域的使用者受邀為 B2B 使用者。

若要新增拒絕清單：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [Azure Active Directory] > [使用者] > [使用者設定]。
3. 在 [外部使用者] 下，選取 [管理外部共同作業設定]。
4. 在 [共同作業限制] 下，選取 [拒絕對指定網域的邀請]。
5. 在 [目標網域] 下，輸入您要封鎖的網域之一的名稱。 若有多個網域，請將每個網域輸入於不同行。 例如︰

   ![顯示已新增網域的拒絕選項](./media/allow-deny-list/DenyListSettings.png)
 
6. 完成後，按一下 [儲存] 。

設定原則後，如果您嘗試邀請來自封鎖網域的使用者，則會顯示一則訊息，指出您的邀請原則目前已封鎖該使用者的網域。
 
### <a name="add-an-allow-list"></a>新增允許清單

這是限制性較高的組態，可讓您在允許清單中設定特定網域，並限制對未提及的任何其他組織或網域發出邀請。 

如果您想要使用允許清單，請務必審慎而完整地評估您的業務需求為何。 如果您讓此原則的限制過多，使用者有可能會選擇透過電子郵件傳送文件，或尋求以其他非 IT 認可的方法進行共同作業。


若要新增允許清單：

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選取 [Azure Active Directory] > [使用者] > [使用者設定]。
3. 在 [外部使用者] 下，選取 [管理外部共同作業設定]。
4. 在 [共同作業限制] 下，選取 [僅允許對指定網域的邀請 (限制性最高)]。
5. 在 [目標網域] 下，輸入您要允許的網域之一的名稱。 若有多個網域，請將每個網域輸入於不同行。 例如︰

   ![顯示已新增網域的允許選項](./media/allow-deny-list/AllowListSettings.png)
 
6. 完成後，按一下 [儲存] 。

設定原則後，如果您嘗試使用者，且使用者來自不在允許清單上的網域，則會顯示一則訊息，指出您的邀請原則目前已封鎖該使用者的網域。

### <a name="switch-from-allow-to-deny-list-and-vice-versa"></a>從允許清單切換為拒絕清單，或反向作業 

如果您從某個原則切換為其他原則，將會捨棄現有的原則組態。 執行切換之前，請務必先備份組態的詳細資料。 

## <a name="set-the-allow-or-deny-list-policy-using-powershell"></a>使用 PowerShell 設定允許或拒絕清單原則

### <a name="prerequisite"></a>必要條件

若要使用 PowerShell 設定允許或拒絕清單，您必須安裝適用於 Windows PowerShell 的 Azure Active Directory 模組預覽版。 明確而言，請安裝 AzureADPreview 模組 2.0.0.98 版或更新版本。

若要檢查模組的版本 (並確認是否已安裝)：
 
1. 以提高權限使用者的身分開啟 Windows PowerShell (以系統管理員的身分執行)。 
2. 執行下列命令，以確認您是否已在電腦上安裝任何版本的「適用於 Windows PowerShell 的 Azure Active Directory 模組」：

   ````powershell  
   Get-Module -ListAvailable AzureAD*
   ````

如果未安裝此模組，或您沒有必要的版本，請執行下列其中一個動作：

- 若未傳回任何結果，請執行下列命令，以安裝最新版的 AzureADPreview 模組：
  
   ````powershell  
   Install-Module AzureADPreview
   ````
- 如果結果中僅顯示 AzureAD 模組，請執行下列命令，以安裝 AzureADPreview 模組： 

   ````powershell 
   Uninstall-Module AzureAD 
   Install-Module AzureADPreview 
   ````
- 如果結果中僅顯示 AzureADPreview 模組，但版本低於 2.0.0.98，請執行下列命令加以更新： 

   ````powershell 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
   ````

- 如果結果中顯示了 AzureAD 和 AzureADPreview 模組，但 AzureADPreview 模組的版本低於 2.0.0.98，請執行下列命令加以更新： 

   ````powershell 
   Uninstall-Module AzureAD 
   Uninstall-Module AzureADPreview 
   Install-Module AzureADPreview 
    ````

### <a name="use-the-azureadpolicy-cmdlets-to-configure-the-policy"></a>使用 AzureADPolicy Cmdlet 設定原則

若要建立允許或拒絕清單，請使用 [New-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview) Cmdlet。 下列範例說明如何設定會封鎖 "live.com" 網域的拒絕清單。

````powershell 
$policyValue = @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}")

New-AzureADPolicy -Definition $policyValue -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
````

以下顯示相同的範例，但內含原則定義。

````powershell  
New-AzureADPolicy -Definition @("{`"B2BManagementPolicy`":{`"InvitationsAllowedAndBlockedDomainsPolicy`":{`"AllowedDomains`": [],`"BlockedDomains`": [`"live.com`"]}}}") -DisplayName B2BManagementPolicy -Type B2BManagementPolicy -IsOrganizationDefault $true 
````

若要設定允許或拒絕清單原則，請使用 [Set-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/set-azureadpolicy?view=azureadps-2.0-preview) Cmdlet。 例如︰

````powershell   
Set-AzureADPolicy -Definition $policyValue -Id $currentpolicy.Id 
````

若要取得原則，請使用 [Get-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview) Cmdlet。 例如︰

````powershell
$currentpolicy = Get-AzureADPolicy | ?{$_.Type -eq 'B2BManagementPolicy'} | select -First 1 
````

若要移除原則，請使用 [Remove-AzureADPolicy](https://docs.microsoft.com/powershell/module/azuread/remove-azureadpolicy?view=azureadps-2.0-preview) Cmdlet。 例如︰

````powershell
Remove-AzureADPolicy -Id $currentpolicy.Id 
````

## <a name="next-steps"></a>後續步驟

- 如需 Azure AD B2B 的概觀，請參閱[何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- 如需條件式存取和 B2B 共同作業的相關資訊，請參閱 [B2B 共同作業使用者的條件式存取](conditional-access.md)。



