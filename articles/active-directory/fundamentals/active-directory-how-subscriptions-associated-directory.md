---
title: 如何將現有的 Azure 訂用帳戶新增到您的 Azure Active Directory 租用戶 | Microsoft Docs
description: 了解如何將現有的 Azure 訂用帳戶新增到您的 Azure Active Directory 租用戶。
services: active-directory
author: eross-msft
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.component: fundamentals
ms.topic: conceptual
ms.date: 09/13/2018
ms.author: lizross
ms.reviewer: jeffsta
ms.custom: it-pro
ms.openlocfilehash: dd62b22eca40a214c5b08a9bc48815e40fe90e47
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984069"
---
# <a name="how-to-associate-or-add-an-azure-subscription-to-azure-active-directory"></a>如何：關聯或新增 Azure 訂用帳戶到 Azure Active Directory
Azure 訂用帳戶有與 Azure Active Directory (Azure AD) 之間的信任關係，這表示訂用帳戶信任 Azure AD 來驗證其使用者、服務與裝置。 多個訂用帳戶可以信任相同的 Azure AD 目錄，但每個訂用帳戶都只能信任單一目錄。

若您的訂用帳戶已過期，您將無法再存取與訂用帳戶關聯的所有其他資源。 不過，Azure AD Directory 依然存在於 Azure 中，可讓您使用不同的 Azure 訂用帳戶來關聯及管理目錄。

針對驗證目的，您的所有使用者都有單一主目錄。 不過，您的使用者也可以是其他目錄中的來賓。 您可以同時為 Azure AD 中的每個使用者使用主目錄與來賓目錄。

>[!Important]
>所有具備指派存取權的[角色型存取控制 (RBAC)](../../role-based-access-control/role-assignments-portal.md) 使用者以及所有訂用帳戶系統管理員都會在訂用帳戶目錄變更之後喪失存取權。 此外，若您有任何金鑰保存庫，它們也會受訂用帳戶移動影響。 若要修正該問題，您必須[變更金鑰保存庫租用戶識別碼](../../key-vault/key-vault-subscription-move-fix.md)再繼續作業。


## <a name="before-you-begin"></a>開始之前
您必須先執行下列工作，才能關聯或新增您的訂用帳戶：

- 使用符合下列條件的帳戶登入：
    - 具有訂用帳戶的 **RBAC 擁有者**存取權。

    - 同時存在於與訂用帳戶關聯的目前目錄中，以及此後您要關聯訂用帳戶的新目錄中。 如需有關如何取得其他目錄存取權的詳細資訊，請參閱 [Azure Active Directory 系統管理員如何新增 B2B 共同作業使用者](../b2b/add-users-administrator.md)。

- 確定您不是使用 Azure 雲端服務提供者 (CSP) 訂用帳戶 (MS-AZR-0145P、MS-AZR-0146P、MS-AZR-159P)、Microsoft 內部訂用帳戶 (MS-AZR-0015P) 或 Microsoft Imagine 訂用帳戶 (MS-AZR-0144P)。
    
## <a name="to-associate-an-existing-subscription-to-your-azure-ad-directory"></a>將現有的訂用帳戶關聯至您的 Azure AD 目錄
1. 在 [Azure 入口網站的 [訂用帳戶] 頁面](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade)登入並選取您要使用的訂用帳戶。

2. 按一下 [變更目錄]。

    ![已反白顯示 [變更目錄] 選項的 [訂用帳戶] 頁面](media/active-directory-how-subscriptions-associated-directory/change-directory-button.png)

3. 檢閱出現的任何警告，然後選取 [變更]。

    ![顯示要變更之目錄的 [變更目錄] 頁面](media/active-directory-how-subscriptions-associated-directory/edit-directory-ui.png)

    訂用帳戶的目錄會變更， 而且您會看到成功訊息。

    ![成功訊息](media/active-directory-how-subscriptions-associated-directory/edit-directory-success.png)    

4. 使用目錄切換器來移至您的新目錄。 最多可能需要 10 分鐘的時間才能正確顯示所有項目。

    ![目錄切換器頁面](media/active-directory-how-subscriptions-associated-directory/directory-switcher.png)

變更訂用帳戶目錄是服務層級作業，因此不會影響訂用帳戶帳單擁有權。 帳戶系統管理員仍然可以從[帳戶中心](https://account.azure.com/subscriptions)變更服務系統管理員。 若要刪除原始目錄，您必須將訂用帳戶訂單擁有權傳輸到新的帳戶系統管理員。若要深入了解如何移轉帳單擁有權，請參閱[將 Azure 訂用帳戶的擁有權轉移轉至另一個帳戶](../../billing/billing-subscription-transfer.md)。 

## <a name="next-steps"></a>後續步驟

- 若要建立新的 Azure AD 租用戶，請參閱[存取 Azure Active Directory 以建立新租用戶](active-directory-access-create-new-tenant.md)

- 若要深入了解如何在 Microsoft Azure 中控制資源存取，請參閱 [了解 Azure 中的資源存取](../../role-based-access-control/rbac-and-directory-admin-roles.md)

- 若要深入了解如何在 Azure AD 中指派角色，請參閱[如何使用 Azure Active Directory 指派目錄角色給使用者](active-directory-users-assign-role-azure-portal.md)
