---
title: 如何使用 Azure Active Directory 來編輯群組資訊 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 來編輯群組的資訊。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: lizross
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: a02987fdce3a15cd5d416234e3717df6d33622ec
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45731337"
---
# <a name="how-to-edit-your-group-information-using-azure-active-directory"></a>做法：使用 Azure Active Directory 來編輯群組資訊

您可以使用 Azure Active Directory 來編輯群組的設定，包括更新其名稱、描述或成員資格類型。

## <a name="to-edit-your-group-settings"></a>若要編輯群組設定
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com)。

2. 選取 [Azure Active Directory]，然後選取 [群組]。

    [群組 - 所有群組] 頁面隨即出現，其中顯示所有作用中的群組。

3. 在 [群組 - 所有群組] 頁面上，於 [搜尋] 方塊中盡可能輸入大量群組名稱。 本文的目的是要搜尋 [MDM 原則 - 西部] 群組。

    搜尋結果會出現在 [搜尋] 方塊底下，並隨著您輸入更多字元而更新結果。

    ![[所有群組] 頁面上的 [搜尋] 方塊中顯示搜尋文字](media/active-directory-groups-settings-azure-portal/search-for-specific-group.png)

4. 選取 [MDM 原則 - 西部] 群組，然後從 [管理] 區域選取 [屬性] 。

    ![[群組概觀] 頁面顯示數目和成員，以及反白的 [成員] 選項](media/active-directory-groups-settings-azure-portal/group-overview-blade.png)

5. 視需要更新 [一般設定] 資訊，包括：

    ![群組的屬性設定](media/active-directory-groups-settings-azure-portal/group-properties-settings.png)

    - **群組名稱。** 編輯現有的群組名稱。
    
    - **群組描述。** 編輯現有的群組描述。

    - **群組類型。** 群組類型一旦建立便無法變更。 若要變更 [群組類型]，您必須先刪除群組再建立新群組。
    
    - **成員資格類型。** 變更成員資格類型。 如需各種可用成員資格類型的詳細資訊，請參閱[做法：使用 Azure Active Directory 入口網站建立基本群組並新增成員](active-directory-groups-create-azure-portal.md)
    
    - **物件識別碼。** 物件識別碼無法變更，但可以將它複製以便在 PowerShell 命令中用於群組。 如需使用 PowerShell Cmdlet 的詳細資訊，請參閱[用於進行群組設定的 Azure Active Directory Cmdlet](../users-groups-roles/groups-settings-v2-cmdlets.md)。

## <a name="next-steps"></a>後續步驟
這些文章提供有關 Azure Active Directory 的其他資訊。

- [檢視群組和成員](active-directory-groups-view-azure-portal.md)

- [建立基本群組並新增成員](active-directory-groups-create-azure-portal.md)

- [如何從群組中新增或移除成員](active-directory-groups-members-azure-portal.md)

- [管理群組中使用者的動態規則](../users-groups-roles/groups-create-rule.md)

- [管理群組的成員資格](active-directory-groups-membership-azure-portal.md)

- [使用群組管理對資源的存取](active-directory-manage-groups.md)

- [將 Azure 訂用帳戶關聯或新增至 Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)
