---
title: 在 Azure Active Directory 中將靜態群組成員資格類型變更為動態 | Microsoft Docs
description: 如何建立成員資格規則，以自動填入群組和規則參考。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 08/01/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: 4cc9c446c1a8ff7c82b08ba9787a40598a8b4cd4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39450728"
---
# <a name="change-static-group-membership-to-dynamic-in-azure-active-directory"></a>在 Azure Active Directory 中將靜態群組成員資格變更為動態

在 Azure Active Directory (Azure AD) 中，您可以將群組的成員資格從靜態變更為動態 (反之亦然)。 Azure AD 會在系統中保留相同的群組名稱和識別碼，因此群組的所有現有參考仍然有效。 如果您改為建立新群組，則必須更新這些參考。 動態群組成員資格可免除新增及移除使用者的額外管理負擔。 本文告訴您如何使用 Azure AD 系統管理中心或 PowerShell Cmdlet，將現有的群組從靜態轉換為動態成員資格。

> [!WARNING]
> 將現有的靜態群組變更為動態群組時，系統將從群組中移除所有現有的成員，然後再處理成員資格規則，以新增成員。 如果群組用來控制對應用程式或資源的存取，則在完全處理成員資格規則之前，原始成員可能會喪失存取權。
>
> 建議您先測試新的成員資格規則，以確保群組中的新成員資格如預期般運作。

## <a name="change-the-membership-type-for-a-group"></a>變更群組的成員資格類型

1. 使用具備您租用戶中全域管理員或使用者帳戶管理員身分的帳戶來登入 [Azure AD 系統管理中心](https://aad.portal.azure.com)。
2. 選取 [群組]。
3. 從 [所有群組] 清單中，開啟您想要變更的群組。
4. 選取 [屬性] 。
5. 在群組的 [屬性] 頁面上，依據您所需的成員資格類型，選取 [已指派] (靜態)、[動態使用者] 或 [動態裝置] 作為 [成員資格類型]。 針對動態成員資格，您可以使用規則建立器來選取簡單規則的選項，或自行撰寫成員資格規則。 

下列步驟是針對某個使用者群組將群組從靜態變更為動態成員資格的範例。

1. 在所選群組的 [屬性] 上，選取 [動態使用者] 作為 [成員資格類型]，然後在說明群組成員資格變更的對話方塊上，選取 [是] 以繼續。 
  
   ![選取動態使用者作為成員資格類型](./media/groups-change-type/select-group-to-convert.png)
  
2. 選取 [新增動態查詢]，然後提供規則。
  
   ![輸入規則](./media/groups-change-type/enter-rule.png)
  
3. 建立規則之後，在頁面底部選取 [新增查詢]。
4. 在群組的 [屬性] 頁面上，選取 [儲存] 以儲存變更。 群組的 [成員資格類型] 會立即在群組清單中更新。

> [!TIP]
> 如果您輸入的成員資格規則不正確，群組轉換可能會失敗。 入口網站右上角將會顯示通知，當中包含系統為什麼無法接受該規則的解釋。 請仔細閱讀，以了解您可以如何調整規則來讓規則變成有效。 如需規則語法的範例以及成員資格規則支援的完整屬性、運算子和值清單，請參閱 [Azure Active Directory 中群組的動態成員資格規則](groups-dynamic-membership.md)。


## <a name="change-membership-type-for-a-group-powershell"></a>變更群組的成員資格類型 (PowerShell)

> [!NOTE]
> 若要變更動態群組屬性，您必須使用**預覽版本** [Azure AD PowerShell 第 2 版](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0)中的 Cmdlet。 您可以從 [PowerShell 資源庫](https://www.powershellgallery.com/packages/AzureADPreview)安裝預覽版。

以下是在現有群組上切換成員資格管理的函式範例。 在此範例中，需小心以正確操作 GroupTypes 屬性，並保留任何與動態成員資格無關的值。

```
#The moniker for dynamic groups as used in the GroupTypes property of a group object
$dynamicGroupTypeString = "DynamicMembership"

function ConvertDynamicGroupToStatic
{
    Param([string]$groupId)

    #existing group types
    [System.Collections.ArrayList]$groupTypes = (Get-AzureAdMsGroup -Id $groupId).GroupTypes

    if($groupTypes -eq $null -or !$groupTypes.Contains($dynamicGroupTypeString))
    {
        throw "This group is already a static group. Aborting conversion.";
    }


    #remove the type for dynamic groups, but keep the other type values
    $groupTypes.Remove($dynamicGroupTypeString)

    #modify the group properties to make it a static group: i) change GroupTypes to remove the dynamic type, ii) pause execution of the current rule
    Set-AzureAdMsGroup -Id $groupId -GroupTypes $groupTypes.ToArray() -MembershipRuleProcessingState "Paused"
}

function ConvertStaticGroupToDynamic
{
    Param([string]$groupId, [string]$dynamicMembershipRule)

    #existing group types
    [System.Collections.ArrayList]$groupTypes = (Get-AzureAdMsGroup -Id $groupId).GroupTypes

    if($groupTypes -ne $null -and $groupTypes.Contains($dynamicGroupTypeString))
    {
        throw "This group is already a dynamic group. Aborting conversion.";
    }
    #add the dynamic group type to existing types
    $groupTypes.Add($dynamicGroupTypeString)

    #modify the group properties to make it a static group: i) change GroupTypes to add the dynamic type, ii) start execution of the rule, iii) set the rule
    Set-AzureAdMsGroup -Id $groupId -GroupTypes $groupTypes.ToArray() -MembershipRuleProcessingState "On" -MembershipRule $dynamicMembershipRule
}
```
若要讓群組變成靜態的：

```
ConvertDynamicGroupToStatic "a58913b2-eee4-44f9-beb2-e381c375058f"
```

若要讓群組變成動態的：

```
ConvertStaticGroupToDynamic "a58913b2-eee4-44f9-beb2-e381c375058f" "user.displayName -startsWith ""Peter"""
```

## <a name="next-steps"></a>後續步驟

這些文章提供有關 Azure Active Directory 中群組的其他資訊。

* [查看現有的群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [建立新群組並新增成員](../fundamentals/active-directory-groups-create-azure-portal.md)
* [管理群組的設定](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理群組的成員資格](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理群組中使用者的動態規則](groups-dynamic-membership.md)
