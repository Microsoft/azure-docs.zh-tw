---
title: 如何指派或移除 Azure Active Directory 授權 | Microsoft Docs
description: 使用 Azure Active Directory 指派或移除使用者或群組的 Azure Active Directory 授權。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: f8b932bc-8b4f-42b5-a2d3-f2c076234a78
ms.service: active-directory
ms.component: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 09/05/2018
ms.author: lizross
ms.reviewer: jeffsta
custom: it-pro
ms.openlocfilehash: e1b0b2f84c67e30c3bb998554dc662b002744003
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45603874"
---
# <a name="how-to-assign-or-remove-azure-active-directory-licenses"></a>做法：指派或移除 Azure Active Directory 授權
許多 Azure Active Directory (Azure AD) 服務會要求您啟用 Azure AD 產品，以及授權該產品的每個使用者或群組 (和相關聯的成員)。 只有擁有有效授權的使用者才能存取和使用授權 Azure AD 服務。

## <a name="available-product-editions"></a>可用的產品版本
Azure AD 產品有數個可用版本。

- Azure AD Free

- Azure AD Basic

- Azure AD Premium 1 (Azure AD P1)

- Azure AD Premium 2 (Azure AD P2)

如需每個產品版本和相關授權詳細資訊的具體資訊，請參閱[我需要什麼授權？](../authentication/concept-sspr-licensing.md)。

## <a name="view-your-product-edition-and-license-details"></a>檢視您的產品版本和授權詳細資訊
您可以檢視可用的產品 (包括各自的授權)，檢查任何將至的到期日和可用的指派數目。

### <a name="to-find-your-product-and-license-details"></a>若要尋找您的產品和授權詳細資訊
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [Azure Active Directory]，然後選取 [授權]。

    [授權] 頁面隨即顯示。

    ![授權頁面上顯示已購買的產品及已指派的授權數目](media/license-users-groups/license-details-blade.png)
    
3. 選取 [已購買產品] 連結，即可檢視 [產品] 頁面，並查看每個特定產品版本的 [已指派]、[可用] 和 [即將過期] 詳細資訊。

    ![[產品] 頁面顯示產品版本與相關聯的授權資訊](media/license-users-groups/license-products-blade-with-products.png)

4. 選取產品版本名稱以查看其經授權的使用者和群組。

## <a name="assign-licenses-to-users-or-groups"></a>指派授權給使用者或群組
請確定需要授權 Azure AD 服務的所有人都擁有適當的授權。 要將授權的權限提供給個別使用者還是整個群組取決於您。

>![Note] 群組型授權是 Azure AD 的公用預覽版功能，可與任何付費 Azure AD 授權方案搭配使用。 如需有關預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。<br><br>如需如何新增使用者的詳細資訊，請參閱[如何新增或刪除 Azure Active Directory 中的使用者](add-users-azure-active-directory.md)。 如需如何建立群組並新增成員的詳細資訊，請參閱[建立基本群組並新增成員](active-directory-groups-create-azure-portal.md)。

### <a name="to-assign-a-license-to-a-specific-user"></a>若要將授權指派給特定使用者
1. 在 [產品] 頁面上，選取您想要指派給使用者的版本名稱。 例如 _Azure Active Directory Premium Plan 2_。

    ![[產品] 頁面顯示反白的產品版本](media/license-users-groups/license-products-blade-with-product-highlight.png)

2. 在 [Azure Active Directory Premium Plan 2] 頁面上，選取 [指派]。

    ![[產品] 頁面顯示反白的 [指派] 選項](media/license-users-groups/license-products-blade-with-assign-option-highlight.png)

3. 在 [指派] 頁面上，選取 [使用者和群組]，然後搜尋並選取您想指派授權的使用者。 例如 _Mary Parker_。

    ![指派授權頁面顯示反白的搜尋和 [選取] 選項](media/license-users-groups/assign-license-blade-with-highlight.png)

4. 選取 [指派] 選項，確認已開啟適當的授權選項，然後選取 [確定]。

    ![授權選項頁面顯示適用於此版本的所有選項](media/license-users-groups/license-option-blade-assignments.png)

    [指派授權] 頁面隨即更新，顯示已選取一名使用者且已設定指派。

    >[!NOTE]
    >某些位置無法使用部分 Microsoft 服務。 必須先指定 [使用位置]，才可以將授權指派給使用者。 您可以在 Azure AD 中的 [Azure Active Directory]&gt;[使用者]&gt;[設定檔]&gt;[設定] 區域中設定此值。

5. 選取 [指派]。

    系統會將使用者新增至授權使用者清單中，且該使用者擁有隨附 Azure AD 服務的存取權。

### <a name="to-assign-a-license-to-an-entire-group"></a>若要指派授權給整個群組
1. 在 [產品] 頁面上，選取您想要指派給使用者的版本名稱。 例如 _Azure Active Directory Premium Plan 2_。

    ![[產品] 刀鋒視窗顯示反白的產品版本](media/license-users-groups/license-products-blade-with-product-highlight.png)

2. 在 [Azure Active Directory Premium Plan 2] 頁面上，選取 [指派]。

    ![[產品] 頁面顯示反白的 [指派] 選項](media/license-users-groups/license-products-blade-with-assign-option-highlight.png)

3. 在 [指派] 頁面上，選取 [使用者和群組]，然後搜尋並選取您想指派授權的群組。 例如 [MDM 原則 - 西部]。

    ![指派授權頁面顯示反白的搜尋和 [選取] 選項](media/license-users-groups/assign-group-license-blade-with-highlight.png)

4. 選取 [指派] 選項，確認已開啟適當的授權選項，然後選取 [確定]。

    ![授權選項頁面顯示適用於此版本的所有選項](media/license-users-groups/license-option-blade-group-assignments.png)

    [指派授權] 頁面隨即更新，顯示已選取一名使用者且已設定指派。

    >[!NOTE]
    >某些位置無法使用部分 Microsoft 服務。 必須先為所有成員指定 [使用位置]，才可以將授權指派給群組。 您可以在 Azure AD 中的 [Azure Active Directory]&gt;[使用者]&gt;[設定檔]&gt;[設定] 區域中設定此值。 未指定使用位置的所有使用者皆會繼承租用戶的位置。

5. 選取 [指派]。

    系統會將群組新增至授權群組清單中，且所有成員均擁有隨附 Azure AD 服務的存取權。


## <a name="remove-a-license"></a>移除授權
您可以從 [授權] 頁面移除使用者或群組的授權。

### <a name="to-remove-a-license-from-a-specific-user"></a>若要移除特定使用者的授權
1. 在產品版本的 [授權使用者] 頁面上，選取應失去授權的使用者。 例如 _Alain Charon_。

2. 選取 [移除授權]。

    ![[授權使用者] 頁面顯示反白的 [移除] 授權選項](media/license-users-groups/license-products-user-blade-with-remove-option-highlight.png)

### <a name="to-remove-a-license-from-a-group"></a>若要移除群組的授權
1. 在產品版本的 [授權群組] 頁面上，選取應失去授權的群組。 例如 [MDM 原則 - 西部]。

2. 選取 [移除授權]。

    ![[授權群組] 頁面顯示反白的 [移除] 授權選項](media/license-users-groups/license-products-group-blade-with-remove-option-highlight.png)

>[!Important]
>您無法直接移除使用者從群組那裡繼承的授權。 而是需將使用者從繼承授權的群組中移除。

## <a name="next-steps"></a>後續步驟
指派授權後，您可以執行下列程序：

- [識別並解決授權指派問題](../users-groups-roles/licensing-groups-resolve-problems.md)

- [將經授權的使用者新增至群組以進行授權](../users-groups-roles/licensing-groups-migrate-users.md)

- [使用群組來管理 Azure Active Directory 授權的案例、限制及已知問題](../users-groups-roles/licensing-group-advanced.md)

- [新增或變更設定檔資訊](active-directory-users-profile-azure-portal.md)