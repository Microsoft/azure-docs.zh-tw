---
title: 在 Azure 中使用 RBAC 管理外部使用者的存取權 | Microsoft Docs
description: 了解如何在 Azure 中使用角色型存取控制 (RBAC) 來管理組織外部使用者的存取權。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 03/20/2018
ms.author: rolyon
ms.reviewer: skwan
ms.custom: it-pro
ms.openlocfilehash: bd75ecde75d0f22dc66f047cd063dd85807f6f33
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44304436"
---
# <a name="manage-access-for-external-users-using-rbac"></a>使用 RBAC 管理外部使用者的存取權

不論是大型組織，還是與需要存取您環境中特定資源，但不一定需要存取整個基礎結構或任何計費相關範圍的外部共同作業者、廠商或自由工作者合作的 SMB，角色型存取控制 (RBAC) 都可為其提供更好的安全性管理。 RBAC 能提供彈性，可擁有一個系統管理員帳戶 (訂用帳戶等級中的服務系統管理員角色) 所管理的 Azure 訂用帳戶，並邀請多個使用者在相同的訂用帳戶下運作，而無需任何系統管理權限。

> [!NOTE]
> 從「Office 365 系統管理中心」佈建的 Office 365 訂用帳戶或 Azure Active Directory 授權 (例如︰存取 Azure Active Directory) 沒有資格使用 RBAC。

## <a name="assign-rbac-roles-at-the-subscription-scope"></a>指派訂用帳戶範圍內的 RBAC 角色

使用 RBAC 時，有兩個常見的範例 (但是不限於)︰

* 邀請來自組織 (並非管理使用者 Azure Active Directory 租用戶的一部分) 外部的使用者來管理特定的資源或是整個訂用帳戶
* 與組織內部但屬於不同小組或群組的使用者 (使用者之 Azure Active Directory 租用戶的成員) 合作，而這些使用者需要整個訂用帳戶或環境中特定資源群組或資源範圍的細微存取權

## <a name="grant-access-at-a-subscription-level-for-a-user-outside-of-azure-active-directory"></a>為 Azure Active Directory 外部的使用者授與訂用帳戶等級的存取權

只有訂用帳戶的**擁有者**可以授與 RBAC 角色。 因此，管理員必須以已預先指派此角色或已建立 Azure 訂用帳戶的使用者身分登入。

從 Azure 入口網站中，在您以管理員身分登入之後，請選取「訂用帳戶」，並選擇所需的訂用帳戶。
![Azure 入口網站中的訂用帳戶刀鋒視窗](./media/role-assignments-external-users/0.png) 根據預設，如果管理使用者已購買 Azure 訂用帳戶，使用者就會顯示為**帳戶管理員**，這是訂用帳戶角色。 如需 Azure 訂用帳戶角色的詳細資訊，請參閱[新增或變更 Azure 訂用帳戶管理員](../billing/billing-add-change-azure-subscription-administrator.md)。

在此範例中，使用者 "alflanigan@outlook.com" 是「預設租用戶 Azure」AAD 租用戶中的「免費試用版」訂用帳戶之**擁有者**。 由於此使用者是具有初始 Microsoft 帳戶 “Outlook” (Microsoft 帳戶 = Outlook、Live 等) 的 Azure 訂用帳戶建立者，因此這個租用戶中所新增其他所有使用者的預設網域名稱會是 **"\@alflaniganuoutlook.onmicrosoft.com"**。 根據設計，新網域的語法構成方式是，將建立租用戶的使用者之使用者名稱和網域名稱加以組合，並新增 **".onmicrosoft.com"** 延伸。
此外，使用者在新增及驗證新租用戶的自訂網域名稱之後，即可使用租用戶中的自訂網域名稱進行登入。 如需有關如何驗證 Azure Active Directory 租用戶中自訂網域名稱的詳細資訊，請參閱[將自訂網域名稱新增至您的目錄](/active-directory/active-directory-add-domain)。

在此範例中，「預設租用戶 Azure」目錄僅包含具有 "\@alflanigan.onmicrosoft.com" 網域名稱的使用者。

選取訂用帳戶之後，管理使用者必須依序按一下 [存取控制 (IAM)] 以及 [新增角色]。

![Azure 入口網站中的存取控制 IAM 功能](./media/role-assignments-external-users/1.png)

![在 Azure 入口網站的存取控制 IAM 功能中新增使用者](./media/role-assignments-external-users/2.png)

下一個步驟是選取要指派的角色，以及要指派 RBAC 角色的使用者。 在 [角色] 下拉式功能表中，管理使用者只會看到 Azure 中提供的內建 RBAC 角色。 如需每個角色及其可指派範圍的詳細說明，請參閱[內建角色](built-in-roles.md)。

然後，管理使用者必須新增外部使用者的電子郵件地址。 預期的行為是要使外部使用者不顯示在現有的租用戶中。 外部使用者受邀請之後，將會與目前在「訂用帳戶」範圍獲指派 RBAC 角色的所有目前使用者一起顯示在 [訂用帳戶] > [存取控制 (IAM)] 下。

![將權限新增至新的 RBAC 角色](./media/role-assignments-external-users/3.png)

![訂用帳戶等級的 RBAC 角色清單](./media/role-assignments-external-users/4.png)

已邀請使用者 "chessercarlton@gmail.com" 成為「免費試用」訂用帳戶的**擁有者**。 傳送邀請之後，外部使用者會收到包含啟用連結的電子郵件確認。
![RBAC 角色的電子郵件邀請](./media/role-assignments-external-users/5.png)

在組織外部的新使用者，沒有任何「預設租用戶 Azure」目錄中的現有屬性。 在外部使用者同意被記錄在與他獲指派角色之訂用帳戶關聯的目錄中之後，便會建立這些屬性。

![RBAC 角色的電子郵件邀請訊息](./media/role-assignments-external-users/6.png)

從現在起，外部使用者在 Azure Active Directory 租用戶中就會顯示為外部使用者，而在 Azure 入口網站中即可檢視此資訊。

![使用者刀鋒視窗 Azure Active Directory Azure 入口網站](./media/role-assignments-external-users/7.png)

在 Azure 入口網站的 [使用者] 檢視中，可由不同的圖示類型來辨識外部使用者。

不過，除非**全域管理員**加以允許，否則將**擁有者**或**參與者**存取權授與給**訂用帳戶**範圍內的外部使用者，並不允許存取管理使用者的目錄。 在使用者屬性中，可以識別 [使用者類型]，其中包含兩個常見的參數 [成員] 和 [來賓]。 成員是已在目錄中註冊的使用者，來賓則是從外部來源邀請到目錄的使用者。 如需詳細資訊，請參閱 [Azure Active Directory 系統管理員如何新增 B2B 共同作業使用者](../active-directory/active-directory-b2b-admin-add-users.md)。

> [!NOTE]
> 請確定外部使用者在入口網站中輸入認證之後，選取正確的目錄進行登入。 相同使用者可以存取多個目錄，並且可以選擇其中一個目錄，方法是按一下 Azure 入口網站右上角的使用者名稱，然後從下拉式清單中選取適當的目錄。

在目錄中使用來賓身分時，外部使用者可以管理 Azure 訂用帳戶的所有資源，但無法存取目錄。

![存取限於 Azure Active Directory Azure 入口網站](./media/role-assignments-external-users/9.png)

Azure Active Directory 和 Azure 訂用帳戶沒有父子式關聯性，如同其他 Azure 資源 (例如︰虛擬機器、虛擬網路、Web Apps、儲存體等) 與 Azure 訂用帳戶。 所有後者都是在 Azure 訂用帳戶底下建立、管理及計費，而 Azure 訂用帳戶則是用來管理對 Azure 目錄的存取權。 如需詳細資訊，請參閱 [Azure 訂用帳戶如何與 Azure AD 相關](/active-directory/active-directory-how-subscriptions-associated-directory)。

所有內建 RBAC 角色的**擁有者**和**參與者**會提供對環境中所有資源的完整管理存取權，差異在於參與者無法建立新的 RBAC 角色並加以刪除。 其他內建角色 (例如**虛擬機器參與者**) 僅對該名稱所表示的資源提供完整管理存取權，不論它們要建立在哪一個**資源群組**。

在訂用帳戶等級指派**虛擬機器參與者**的內建 RBAC 角色，表示使用者指派角色︰

* 可以檢視所有虛擬機器，不論其部署日期和所屬的資源群組
* 具有訂用帳戶中虛擬機器的完整管理存取權
* 無法檢視訂用帳戶中任何其他資源類型
* 無法從計費觀點來操作任何變更

## <a name="assign-a-built-in-rbac-role-to-an-external-user"></a>將內建的 RBAC 角色指派給外部使用者

這項測試的不同案例中，會將外部使用者 "alflanigan@gmail.com" 新增為**虛擬機器參與者**。

![虛擬機器參與者內建角色](./media/role-assignments-external-users/11.png)

這個具有此內建角色之外部使用者的正常行為，就是僅查看和管理虛擬機器，以及其相鄰 Resource Manager 僅在部署時所需的資源。 就設計而言，這些受限角色所提供的存取權僅限在 Azure 入口網站中所建立與它們對應的資源。

![Azure 入口網站中的虛擬機器參與者角色概觀](./media/role-assignments-external-users/12.png)

## <a name="grant-access-at-a-subscription-level-for-a-user-in-the-same-directory"></a>為相同目錄中的使用者授與訂用帳戶等級的存取權

程序流程與新增外部使用者的相同，從授與 RBAC 角色的系統管理員觀點以及取得角色存取權的使用者皆然。 此處的差異在於，受邀的使用者無法接收任何電子郵件邀請，因為訂用帳戶內的所有資源範圍在登入之後都會在儀表板中提供使用。

## <a name="assign-rbac-roles-at-the-resource-group-scope"></a>指派資源群組範圍內的 RBAC 角色

對於兩種類型的使用者 - 外部或內部 (相同目錄的一部分)，指派**資源群組**範圍內的 RBAC 角色與指派訂用帳戶等級的角色程序相同。 獲指派 RBAC 角色的使用者在其環境中所看到的資源群組，僅限於已從 Azure 入口網站中的 [資源群組] 圖示指派存取權給他們的資源群組。

## <a name="assign-rbac-roles-at-the-resource-scope"></a>指派資源範圍內的 RBAC 角色

在 Azure 中指派資源範圍內的角色與指派訂用帳戶等級或資源群組等級的角色程序相同，這兩個案例皆遵循相同的工作流程。 同樣地，獲指派 RBAC 角色的使用者所能看到的項目，也僅限於在 [所有資源] 索引標籤或直接在其儀表板中指派存取權給他們的項目。

不論是在資源群組範圍還是資源範圍，RBAC 都有一個重要層面，就是讓使用者確定登入正確的目錄。

![Azure 入口網站中的目錄登入](./media/role-assignments-external-users/13.png)

## <a name="assign-rbac-roles-for-an-azure-active-directory-group"></a>為 Azure Active Directory 群組指派 RBAC 角色

在於 Azure 的三種不同範圍使用 RBAC 的所有案例中，都提供以指派的使用者身分進行各種資源之管理、部署及系統管理的權限，而無須管理個人訂用帳戶。 不論 RBAC 角色是指派給訂用帳戶、資源群組還是資源範圍，所有由所指派使用者進一步建立的資源，都會在使用者能夠存取的一個 Azure 訂用帳戶下計費。 如此一來，擁有整個 Azure 訂用帳戶計費系統管理員權限的使用者，可以完整概觀耗用量，不論資源管理者是誰。

對於較大型組織，考慮到管理使用者想要將細微存取授與小組或整個部門的方面，而非個別授與每位使用者，可透過相同的方式來套用 Azure Active Directory 群組的 RBAC 角色，因此考慮將它作為非常有效率的時間和管理選項。 為了說明此範例中，**參與者**角色已新增至訂用帳戶等級之租用戶的其中一個群組。

![新增 AAD 群組的 RBAC 角色](./media/role-assignments-external-users/14.png)

這些群組是安全性群組，只在 Azure Active Directory 內進行佈建和管理。

