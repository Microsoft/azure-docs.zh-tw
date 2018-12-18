---
title: 在 Azure Active Directory 中檢視貴組織的群組和成員的快速入門 | Microsoft Docs
description: 快速入門包含有關如何使用 Azure 入口網站來搜尋和檢視貴組織群組及其指派成員的步驟。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: lizross
ms.custom: it-pro
ms.reviewer: krbain
ms.openlocfilehash: 9da9f055163f5df4ea064bd46caa811e39fba20c
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47056398"
---
<!--As a brand-new Azure AD administrator, I need to view my organization’s groups along with the assigned members, so I can manage permissions to apps and services for people in my organization-->

# <a name="quickstart-view-your-organizations-groups-and-members-in-azure-active-directory"></a>快速入門：在 Azure Active Directory 中檢視貴組織的群組和成員
您可以使用 Azure 入口網站，檢視貴組織現有的群組和群組成員。 群組用來管理對於可能受限制的應用程式和服務，需要相同存取權和權限的所有使用者 (成員)。

在本快速入門中，您將檢視貴組織的所有現有群組並檢視指派的成員。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。 

## <a name="prerequisites"></a>必要條件
在開始之前，您必須：

- 建立 Azure Active Directory 租用戶。 如需詳細資訊，請參閱[存取 Azure Active Directory 入口網站並建立新的租用戶](active-directory-access-create-new-tenant.md)。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站
您必須使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com/)。

## <a name="create-a-new-group"></a>建立新群組 
建立新的群組，名為 _MDM 原則 - 西部_。 如需建立群組的詳細資訊，請參閱[如何建立基本的群組及新增成員](active-directory-groups-create-azure-portal.md)。

1. 選取 [Azure Active Directory]、[群組]，然後選取 [新增群組]。

2. 完成 [群組] 頁面：
    
    - **群組類型：** 選取 [安全性]
    
    - **群組名稱：** 輸入 _MDM 原則 - 西部_
    
    - **成員資格類型：** 選取 [已指派]。

3. 選取 [建立] 。

## <a name="create-a-new-user"></a>建立新的使用者
建立新的使用者，名為 _Alain Charon_。 使用者必須存在才能新增為群組成員。 如需如何建立使用者的詳細資訊，請參閱[如何新增或刪除使用者](add-users-azure-active-directory.md)。

1. 選取 [Azure Active Directory]、[使用者]，然後選取 [新增使用者]。

2. 完成 [使用者] 頁面：

    - **名稱：** 輸入 _Alain Charon_。

    - **使用者名稱：** 輸入 *alain@contoso.com*。

3. 複製在 [密碼] 方塊中提供的自動產生密碼，然後選取 [建立]。

## <a name="add-a-group-member"></a>新增群組成員
現有您有一個群組和一個使用者，您可以將 _Alain Charon_ 新增為 [MDM 原則 - 西部] 群組的成員。 如需新增群組成員的詳細資訊，請參閱[ 如何新增或移除群組成員](active-directory-groups-members-azure-portal.md)。

1. 選取 [Azure Active Directory] > [群組]。

2. 從 [群組 - 所有群組] 頁面，搜尋並選取 [MDM 原則 - 西部] 群組。

3. 從 [MDM 原則 - 西部概觀] 頁面的 [管理] 區域中，選取 [成員]。

4. 選取 [新增成員]，然後搜尋並選取 **Alain Charon**。

5. 選擇 [選取]。

## <a name="view-all-groups"></a>檢視所有群組
您可以在 Azure 入口網站的 [群組 - 所有群組] 頁面中，看到貴組織的所有群組。

- 選取 [Azure Active Directory] > [群組]。

    [群組 - 所有群組] 頁面隨即出現，其中顯示所有作用中的群組。

    ![[群組 - 所有群組] 頁面，其中顯示所有現有的群組](media/active-directory-groups-view-azure-portal/groups-all-groups-blade-with-all-groups.png)

## <a name="search-for-the-group"></a>搜尋群組
搜尋 [群組-所有群組] 頁面以尋找 [MDM 原則 - 西部] 群組。

1. 從 [群組-所有群組] 頁面，在 [搜尋] 方塊中輸入 _MDM_。

    搜尋結果會出現在 [搜尋] 方塊之下，包括 [MDM 原則 - 西部] 群組。

    ![已填妥搜尋方塊的 [群組 – 所有群組] 頁面](media/active-directory-groups-view-azure-portal/search-for-specific-group.png)

3. 選取 [MDM 原則 - 西部] 群組。

4. 檢視 [MDM 原則 - 西部概觀] 頁面上的群組資訊，包括該群組的成員數目。

    ![具有成員資訊的 [MDM 原則 - 西部概觀] 頁面](media/active-directory-groups-view-azure-portal/group-overview-blade.png)

## <a name="view-group-members"></a>檢視群組成員
既然您已經找到群組，您可以檢視所有已指派的成員。

- 從 [管理] 區域中選取 [成員]，然後檢閱指派給該特定群組的完整成員名稱清單，包括 _Alain Charon_。

    ![已指派給 [MDM 原則 – 西部] 群組的成員清單](media/active-directory-groups-view-azure-portal/groups-all-members.png)

## <a name="clean-up-resources"></a>清除資源
此群組使用於此文件的**使用說明指南**一節中可用的數個使用說明程序中。 不過，如果您不想使用此群組，您可以使用下列步驟來刪除它及其指派的成員：

1. 在 [群組-所有群組] 頁面上，搜尋 [MDM 原則 - 西部] 群組。

2.  選取 [MDM 原則 - 西部] 群組。

    [MDM 原則 - 西部概觀] 頁面隨即出現。

3. 選取 [刪除] 。

    群組及其相關聯的成員都已刪除。

    ![已醒目提示刪除連結的 [MDM 原則 - 西部概觀] 頁面](media/active-directory-groups-view-azure-portal/group-overview-blade-delete.png)

    >[!Important]
    >這不會刪除使用者 Alain Charon，只會刪除他在已刪除群組中的成員資格。

## <a name="next-steps"></a>後續步驟
前進到下一篇文章，了解如何將訂用帳戶關聯至您的 Azure AD 目錄。

> [!div class="nextstepaction"]
> [與 Azure 訂用帳戶建立關聯](active-directory-how-subscriptions-associated-directory.md)