---
title: 如何在 Azure Active Directory 中將最近刪除的使用者還原或永久移除 | Microsoft Docs
description: 了解如何使用 Azure Active Directory 檢視可還原的使用者、還原已刪除的使用者，或是永久刪除使用者。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: lizross
ms.reviewer: jeffsta
ms.custom: it-pro
ms.openlocfilehash: 88d3c672cd072cd4b252f7ce4ede3a4c7b13a7db
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45736167"
---
# <a name="how-to-restore-or-permanently-remove-a-recently-deleted-user-with-azure-active-directory"></a>做法：如何使用 Azure Active Directory 將最近刪除的使用者還原或永久移除
刪除使用者之後，其帳戶會維持在暫時停權狀態 30 天。 在這 30 天期間，可以還原該使用者帳戶及其所有屬性。 30 天期限過後，系統會自動將該使用者永久刪除。

您可以在 Azure 入口網站中使用 Azure Active Directory (Azure AD)，檢視可還原的使用者、還原已刪除的使用者，或永久刪除使用者。

>[!Important]
>您和 Microsoft 客戶支援服務都無法還原已永久刪除的使用者。

## <a name="required-permissions"></a>所需的權限
您必須具備下列其中一個角色，才能還原及永久刪除使用者。

- 公司系統管理員

- 合作夥伴第 1 層支援

- 合作夥伴第 2 層支援

- 使用者帳戶管理員

## <a name="view-your-restorable-users"></a>檢視可還原的使用者
您可以查看刪除 30 天以內的所有使用者。 這些使用者均可還原。

### <a name="to-view-your-restorable-users"></a>若要檢視可還原的使用者
1. 使用目錄的全域系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [Azure Active Directory] 並選取 [使用者]，然後選取 [已刪除的使用者]。

    檢閱可還原的使用者清單。

    ![[使用者 - 已刪除的使用者] 頁面上顯示仍可還原的使用者](media/active-directory-users-restore/users-deleted-users-view-restorable.png)

## <a name="restore-a-recently-deleted-user"></a>還原最近刪除的使用者
雖然使用者的帳戶暫時停權，但所有相關的目錄資訊仍保留。 還原使用者時，同時也會還原這些目錄資訊。

### <a name="to-restore-a-user"></a>若要還原使用者
1. 在 [使用者 - 已刪除的使用者] 頁面上，搜尋並選取其中一個可用的使用者。 例如 _Mary Parker_。

2. 選取 [還原使用者]。

    ![[使用者 - 已刪除的使用者] 頁面上顯示反白的 [還原使用者] 選項](media/active-directory-users-restore/users-deleted-users-restore-user.png)

## <a name="permanently-delete-a-user"></a>永久刪除使用者
您可以從目錄中永久刪除使用者，不需等到 30 天後自動刪除。 無論是您、其他系統管理員還是 Microsoft 客戶支援服務，均無法還原已永久刪除的使用者。

>[!Note]
>如果您意外永久刪除使用者，則必須建立新的使用者，並以手動方式輸入所有先前的資訊。 如需如何建立使用者的詳細資訊，請參閱[新增或刪除使用者](add-users-azure-active-directory.md)。

### <a name="to-permanently-delete-a-user"></a>若要永久刪除使用者

1. 在 [使用者 - 已刪除的使用者] 頁面上，搜尋並選取其中一個可用的使用者。 例如 _Rae Huff_。

2. 選取 [永久刪除]。

    ![[使用者 - 已刪除的使用者] 頁面上顯示反白的 [還原使用者] 選項](media/active-directory-users-restore/users-deleted-users-permanent-delete-user.png)

## <a name="next-steps"></a>後續步驟
還原或刪除使用者之後，您可以執行下列基本程序：

- [新增或刪除使用者](add-users-azure-active-directory.md)

- [將角色指派給使用者](active-directory-users-assign-role-azure-portal.md)

- [新增或變更設定檔資訊](active-directory-users-profile-azure-portal.md)

- [從另一個目錄中新增來賓使用者](../b2b/what-is-b2b.md) 

如需其他可用使用者管理工作的詳細資訊，請參閱 [Azure Active Directory 使用者管理文件](../users-groups-roles/index.yml)。
