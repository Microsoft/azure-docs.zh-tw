---
title: Office 365 群組在 Azure Active Directory 中的群組名稱原則設定 (預覽) | Microsoft Docs
description: 如何設定 Azure Active Directory 中 Office 365 群組的到期日 (預覽)
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 05/21/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 111be7d3ee00f2b40ace3bfe4efdacc5029ccf77
ms.sourcegitcommit: 194789f8a678be2ddca5397137005c53b666e51e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39239129"
---
# <a name="enforce-a-naming-policy-for-office-365-groups-in-azure-active-directory-preview"></a>在 Azure Active Directory 中對 Office 365 群組強制執行命名原則 (預覽)

若要對使用者所建立或編輯的 Office 365 群組強制執行一致的命名慣例，請在 Azure Active Directory (Azure AD) 中為租用戶設定群組命名原則。 例如，您可以使用命名原則來傳達群組的功能、成員資格、地理區域或群組的建立者。 您也可以使用命名原則將通訊錄中的群組分類。 您可以使用原則來防止群組名稱和別名中使用特定字組。

> [!IMPORTANT]
> 若要使用「Office 365 群組命名原則」預覽版，每個獨特使用者只要是一或多個 Office 365 群組的成員，都必須具備 Azure Active Directory Premium P1 授權或 Azure AD Basic EDU 授權。

命名原則會套用至跨工作負載 (例如 Microsoft Teams、SharePoint、Exchange 或 Planner) 所建立的建立或編輯群組。 群組名稱和群組別名皆為套用對象。 如果您在 Azure AD 中設定命名原則，而您又有現存的 Exchange 群組命名原則，所套用的會是 Azure AD 命名原則。

## <a name="naming-policy-features"></a>命名原則功能
您可以透過兩種不同方式對 Office 365 群組強制執行命名原則：

-   **前置詞後置詞命名原則** 您可以定義會在之後自動新增的前置詞或後置詞，以對您的群組強制執行命名慣例 (例如，在「GRP\_JAPAN\_My Group\_Engineering」這個群組名稱中，GRP\_JAPAN\_ 是前置詞，\_Engineering 是後置詞)。 

-   **自訂封鎖字組** 您可以上傳一組要在使用者所建立之群組中封鎖的組織專屬字組 (例如，「CEO, Payroll, HR」)。

### <a name="prefix-suffix-naming-policy"></a>前置詞後置詞命名原則

命名慣例結構一般是「前置詞[GroupName]後置詞」。 雖然您可以定義多個前置詞和後置詞，但設定中的 [GroupName] 只能有一個。 前置詞或後置詞可以是固定字串，也可以是使用者屬性，例如 \[Department\] 就會根據建立群組的使用者來替換。 前置詞和後置詞字串合在一起所允許的字元總數是 53 個字元。 

前置詞和後置詞可以包含群組名稱和群組別名所支援的特殊字元。 前置詞或後置詞中不受群組別名支援的字元仍會套用到群組名稱，但會從群組別名中移除。 由於有此限制，群組名稱和群組別名所套用的前置詞和後置詞可能會不同。 

#### <a name="fixed-strings"></a>固定字串

您可以使用字串來方便掃描和區分全域通訊清單中與群組工作負載左側導覽連結中的群組。 某些常見的前置詞是「Grp\_Name」、「\#Name」、「\_Name」之類的關鍵字

#### <a name="user-attributes"></a>使用者屬性

您可以使用屬性來協助您和您的使用者識別群組在建立時所適用的對象是哪個部門、辦公室或地理區域。 例如，如果您將命名原則定義為 `PrefixSuffixNamingRequirement = “GRP [GroupName] [Department]”` 和 `User’s department = Engineering`，則強制執行的群組名稱可能會是「GRP My Group Engineering」。 支援的 Azure AD 屬性是 \[Department\]、\[Company\]、\[Office\]、\[StateOrProvince\]、\[CountryOrRegion\]、\[Title\]。 不支援的使用者屬性則會以固定字串視之；例如，「\[postalCode\]」。 擴充屬性和自訂屬性則不受支援。

我們對您的建議是，使用已為組織中的所有使用者填入值的屬性，而不要使用具有長值的屬性。

### <a name="custom-blocked-words"></a>自訂封鎖字組

封鎖字組清單是要在群組名稱和別名中封鎖之片語的逗號分隔清單。 系統不會執行子字串搜尋。 群組名稱必須和一或多個自訂封鎖字組完全相符，才會觸發失敗。 由於不會執行子字串搜尋，所以即便「lass」是封鎖字組，使用者仍可使用「Class」之類的一般字組。

封鎖字組清單規則：
- 封鎖字組不區分大小寫。
- 使用者在輸入封鎖字組來作為群組名稱的一部分時，會看到附上封鎖字組的錯誤訊息。
- 封鎖字組沒有字元限制。
- 封鎖字組清單中可以設定的片語上限為 5000 個。 

### <a name="administrator-override"></a>系統管理員覆寫

選定的系統管理員可以不受這些原則限制 (範圍橫跨所有群組工作負載和端點)，因此可以使用封鎖字組和自己的命名慣例來建立群組。 以下是不受群組命名原則限制的系統管理員角色清單。

- 全域管理員
- 合作夥伴第 1 層支援
- 合作夥伴第 2 層支援
- 使用者帳戶管理員
- 目錄寫入器

## <a name="install-powershell-cmdlets-to-configure-a-naming-policy"></a>安裝 PowerShell Cmdlet 來設定命名原則

請務必要將 Windows PowerShell 的任何舊版 Azure Active Directory PowerShell for Graph 模組解除安裝，並安裝 [Azure Active Directory PowerShell for Graph - 公開預覽版本 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.137)，然後才執行 PowerShell 命令。 

1. 以系統管理理員身分開啟 Windows PowerShell 應用程式。
2. 將任何舊版的 AzureADPreview 解除安裝。
  
  ````
  Uninstall-Module AzureADPreview
  ````
3. 安裝最新版的 AzureADPreview。
  
  ````
  Install-Module AzureADPreview
  ````
如果系統發出有關所存取之存放庫不受信任的提示，請輸入 **Y**。新模組可能需要幾分鐘的時間才能安裝完成。

## <a name="configure-the-group-naming-policy-for-a-tenant-using-azure-ad-powershell"></a>使用 Azure AD PowerShell 設定租用戶的群組命名原則

1. 在電腦上開啟 Windows PowerShell 視窗。 不需較高的權限即可將它開啟。

2. 執行下列命令，以準備執行 Cmdlet。
  
  ````
  Import-Module AzureADPreview
  Connect-AzureAD
  ````
  在所開啟的 [登入帳戶] 畫面中，輸入系統管理員帳戶和密碼以連線到服務，然後選取 [登入]。

3. 請遵循[用於進行群組設定的 Azure Active Directory Cmdlet](groups-settings-cmdlets.md) 中的步驟，建立此租用戶的群組設定。

### <a name="view-the-current-settings"></a>檢視目前的設定

1. 擷取目前的命名原則，以檢視目前的設定。
  
  ````
  $Setting = Get-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id
  ````
  
2. 顯示目前的群組設定。
  
  ````
  $Setting.Values
  ````
  
### <a name="set-the-naming-policy-and-custom-blocked-words"></a>設定命名原則和自訂封鎖字組

1. 在 Azure AD PowerShell 中設定群組名稱前置詞和後置詞。
  
  ````
  $Setting["PrefixSuffixNamingRequirement"] =“GRP_[GroupName]_[Department]"
  ````
  
2. 設定想要限制的自訂封鎖字組。 下列範例說明如何新增自己的自訂字組。
  
  ````
  $Setting["CustomBlockedWordsList"]=“Payroll,CEO,HR"
  ````
  
3. 儲存設定以使新原則生效，如下列範例所示。
  
  ````
  Set-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id -DirectorySetting $Setting
  ````
  
就這麼簡單。 您已設定命名原則並新增封鎖字組。

## <a name="export-or-import-the-list-of-custom-blocked-words"></a>匯出或匯入自訂封鎖字組清單

如需詳細資訊，請參閱[用於進行群組設定的 Azure Active Directory Cmdlet](groups-settings-cmdlets.md) 一文。

以下 PowerShell 指令碼範例可匯出多個封鎖字組：

````
$Words = (Get-AzureADDirectorySetting).Values | Where-Object -Property Name -Value CustomBlockedWordsList -EQ 
Add-Content "c:\work\currentblockedwordslist.txt" -Value $words.value.Split(",").Replace("`"","")  
````

以下 PowerShell 指令碼範例可匯入多個封鎖字組：

````
$BadWords = Get-Content "C:\work\currentblockedwordslist.txt"
$BadWords = [string]::join(",", $BadWords)
$Settings = Get-AzureADDirectorySetting | Where-Object {$_.DisplayName -eq "Group.Unified"}
if ($Settings.Count -eq 0)
    {$Template = Get-AzureADDirectorySettingTemplate | Where-Object {$_.DisplayName -eq "Group.Unified"}
    $Settings = $Template.CreateDirectorySetting()
    New-AzureADDirectorySetting -DirectorySetting $Settings
    $Settings = Get-AzureADDirectorySetting | Where-Object {$_.DisplayName -eq "Group.Unified"}}
$Settings["CustomBlockedWordsList"] = $BadWords
$Settings["EnableMSStandardBlockedWords"] = $True
Set-AzureADDirectorySetting -Id $Settings.Id -DirectorySetting $Settings 
````

## <a name="naming-policy-experiences-across-office-365-apps"></a>跨 Office 365 應用程式的命名原則體驗

在 Azure AD 中設定群組命名原則後，使用者若於 Office 365 應用程式建立群組，將會看見： 

* 根據命名原則所形成之名稱的預覽 (含前置詞和後置詞)，且在使用者輸入群組名稱時就會立刻看見
* 使用者在輸入封鎖字組時會看到錯誤訊息，從而能夠將封鎖字組移除。

工作負載 | 法規遵循
----------- | -------------------------------
Azure Active Directory 入口網站 | 當使用者為了建立或編輯群組而輸入群組名稱時，Azure AD 入口網站和存取面板入口網站會顯示命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，系統會顯示附上封鎖字組的錯誤訊息，讓使用者可將其移除。
Outlook Web Access (OWA) | 當使用者輸入群組名稱或群組別名時，Outlook Web Access 會顯示命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，UI 中會顯示錯誤訊息以及封鎖字組，讓使用者可將其移除。
Outlook 桌面 | 在 Outlook 桌面中所建立的群組會遵守命名原則設定。 Outlook 桌面應用程式還不會顯示強制使用之群組名稱的預覽，也不會在使用者輸入群組名稱時傳回自訂封鎖字組錯誤。 不過，建立或編輯群組時會自動套用命名原則，而且使用者會在群組名稱或別名中有自訂封鎖字組時看到錯誤訊息。
Microsoft Teams | 當使用者輸入小組名稱時，Microsoft Teams 會顯示群組命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，系統會顯示錯誤訊息以及封鎖字組，讓使用者可將其移除。
SharePoint  |  當使用者輸入網站名稱或群組電子郵件地址時，SharePoint 會顯示命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，系統會顯示錯誤訊息以及封鎖字組，讓使用者可將其移除。
Microsoft Stream | 當使用者輸入群組名稱或群組電子郵件別名時，Microsoft Stream 會顯示群組命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，系統會顯示附上封鎖字組的錯誤訊息，讓使用者可將其移除。
Outlook iOS 和 Android 應用程式 | 在 Outlook 應用程式中所建立的群組會遵守所設定的命名原則。 Outlook 行動應用程式還不會顯示命名原則所強制使用之名稱的預覽，也不會在使用者輸入群組名稱時傳回自訂封鎖字組錯誤。 不過，在按一下建立/編輯時會自動套用命名原則，而且使用者會在群組名稱或別名中有自訂封鎖字組時看到錯誤訊息。
Groups 行動應用程式 | 在 Groups 行動應用程式中所建立的群組會遵守命名原則。 Groups 行動應用程式不會顯示命名原則的預覽，也不會在使用者輸入群組名稱時傳回自訂封鎖字組錯誤。 不過，建立或編輯群組時會自動套用命名原則，而且使用者會在群組名稱或別名中有自訂封鎖字組時看到適當的錯誤。
Planner | Planner 會遵守命名原則。 輸入計劃名稱時，Planner 會顯示命名原則預覽。 當使用者輸入自訂封鎖字組時，系統會在建立計劃時顯示錯誤訊息。
Dynamics 365 for Customer Engagement | Dynamics 365 for Customer Engagement 會遵守命名原則。 當使用者輸入群組名稱或群組電子郵件別名時，Dynamics 365 會顯示命名原則所強制使用的名稱。 當使用者輸入自訂封鎖字組時，系統會顯示附上封鎖字組的錯誤訊息，讓使用者可將其移除。
學校資料同步 (SDS) | 透過 SDS 所建立的群組會遵守命名原則，但不會自動套用命名原則。 對於需要建立群組並上傳到 SDS 的類別名稱，SDS 系統管理員必須對此名稱附加前置詞和後置詞。 否則，群組的建立或編輯會失敗。
Outlook Customer Manager (OCM) | Outlook Customer Manager 會遵守命名原則，而此原則會自動套用到 Outlook Customer Manager 中所建立的群組。 如果偵測到自訂封鎖字組，系統會封鎖 OCM 中的群組建立，並讓使用者無法使用 OCM 應用程式。
Classroom 應用程式 | 在 Classroom 應用程式中所建立的群組會遵守命名原則，但不會自動套用命名原則，而且不會在使用者輸入教室群組名稱時顯示命名原則預覽。 使用者必須輸入具有前置詞和後置詞的強制教室群組名稱。 如果沒有這麼做，教室群組的建立或編輯作業會失敗，並顯示錯誤。
Power BI | Power BI 工作區會遵守命名原則。    
Yammer | 當使用者以其 Azure Active Directory 帳戶登入 Yammer 來建立群組或編輯群組名稱時，群組名稱會符合命名原則。 這適用於 Office 365 連線群組及所有其他 Yammer 群組。<br>如果在具有命名原則之前建立 Office 365 連線群組，群組名稱不會自動遵循命名原則。 當使用者編輯群組名稱時，系統會提示他們新增前置詞和後置詞。
StaffHub  | StaffHub 小組不會遵守命名原則，但基礎 Office 365 群組會。 StaffHub 小組名稱不會套用前置詞和後置詞，也不會檢查自訂封鎖字組。 但 StaffHub 會套用前置詞和後置詞，並且會從基礎 Office 365 群組中移除封鎖字組。
Exchange PowerShell | Exchange PowerShell Cmdlet 會遵守命名原則。 使用者若未在群組名稱和群組別名 (mailNickname) 中遵守命名原則，將會收到適當的錯誤訊息以及針對自訂封鎖字組所建議的前置詞和後置詞。
Azure Active Directory PowerShell Cmdlet | Azure Active Directory PowerShell Cmdlet 會遵守命名原則。 使用者若未在群組名稱和群組別名中遵守命名慣例，將會收到適當的錯誤訊息以及針對自訂封鎖字組所建議的前置詞和後置詞。
Exchange 系統管理中心 | Exchange 系統管理中心會遵守命名原則。 使用者若未在群組名稱和群組別名中遵守命名慣例，將會收到適當的錯誤訊息以及針對自訂封鎖字組所建議的前置詞和後置詞。
Office 365 系統管理中心 | Office 365 系統管理中心會遵守命名原則。 當使用者建立或編輯群組名稱時，系統會自動套用命名原則，而且使用者會在輸入自訂封鎖字組時收到適當的錯誤。 Office 365 系統管理中心還不會顯示命名原則的預覽，也不會在使用者輸入群組名稱時傳回自訂封鎖字組錯誤。

## <a name="next-steps"></a>後續步驟
這些文章提供有關 Azure AD 群組的其他資訊。

* [查看現有的群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Office 365 群組的到期原則](groups-lifecycle.md)
* [管理群組的設定](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理群組的成員](../fundamentals/active-directory-groups-members-azure-portal.md)
* [管理群組的成員資格](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理群組中使用者的動態規則](groups-dynamic-membership.md)
