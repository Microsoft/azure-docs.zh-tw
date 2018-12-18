---
title: 在 Azure 入口網站中新增 B2B 共同作業使用者 - Azure Active Directory | Microsoft Docs
description: 說明系統管理員如何使用 Azure Active Directory (Azure AD) B2B 共同作業，將來自夥伴組織的來賓使用者新增到其目錄。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 07/10/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: fef4615517da08262cc5845aaa076472c3874b34
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45984283"
---
# <a name="add-azure-active-directory-b2b-collaboration-users-in-the-azure-portal"></a>在 Azure 入口網站中新增 Azure Active Directory B2B 共同作業使用者

身為全域管理員或是獲派任何受限管理員目錄角色的使用者，您可以使用 Azure 入口網站來邀請 B2B 共同作業使用者。 您可以將來賓使用者邀請到目錄、群組或應用程式。 透過上述任一方法邀請使用者之後，受邀使用者的帳戶就會新增至 Azure Active Directory (Azure AD)，且使用者類型為「來賓」。 來賓使用者接著必須兌換其邀請才能存取資源。

將來賓使用者新增至目錄之後，您就可以將共用應用程式的直接連結傳送給來賓使用者，或者來賓使用者可以按一下邀請電子郵件中的兌換 URL。 如需關於兌換程序的詳細資訊，請參閱 [B2B 共同作業邀請兌換](redemption-experience.md)。

> [!IMPORTANT]
> 您應該遵循[做法：在 Azure Active Directory 中新增貴組織的隱私權資訊](https://aka.ms/adprivacystatement)中的步驟，以新增貴組織隱私權聲明的 URL。 在第一次邀請兌換程序中，受邀的使用者必須同意您的隱私權條款才能繼續進行。 

## <a name="add-guest-users-to-the-directory"></a>將來賓使用者新增到目錄

若要將 B2B 共同作業使用者新增至目錄，請遵循下列步驟：

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導覽窗格中，選取 [Azure Active Directory]。
3. 在 [管理] 底下選取 [使用者]。
4. 選取 [新增來賓使用者]。

   ![顯示 [新增來賓使用者] 在 UI 中的位置](./media/add-users-administrator/NewGuestUser-Directory.png) 
 
5. 在 [使用者名稱] 下，輸入外部使用者的電子郵件地址。 您也可以選擇納入歡迎訊息。 例如︰

   ![顯示 [新增來賓使用者] 在 UI 中的位置](./media/add-users-administrator/InviteGuest.png) 

    > [!NOTE]
    > 有些電子郵件提供者允許使用者將加號 (+) 和額外文字新增到其電子郵件地址，以協助收件匣篩選等事項。 不過，Azure AD 目前不支援電子郵件地址中有加號。 若要避免傳遞問題，請省略 @ 符號之前的加號及其後任何字元。

6. 選取 [邀請] 即可自動對來賓使用者傳送邀請。 
 
在傳送邀請之後，系統就會自動將使用者帳戶以來賓身分新增至目錄。


![顯示 B2B 使用者與「來賓」使用者類型](./media/add-users-administrator/GuestUserType.png)  

## <a name="add-guest-users-to-a-group"></a>將來賓使用者新增到群組
如果您需要透過手動方式，以 Azure AD 系統管理員身分將 B2B 共同作業使用者新增到群組，請遵循下列步驟：

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導覽窗格中，選取 [Azure Active Directory]。
3. 在 [管理] 底下選取 [群組]。
4. 選取群組 (或按一下 [新增群組] 以建立新的群組)。 您不妨納入群組描述，說明該群組包含 B2B 來賓使用者。
5. 選取 [成員]。 
6. 執行下列其中一項：
   - 如果目錄中已有來賓使用者，請搜尋 B2B 使用者。 選取使用者，然後按一下 [選取] 將使用者新增至群組中。
   - 如果目錄中尚未有來賓使用者存在，請藉由在搜尋方塊中鍵入使用者的電子郵件地址、選擇性鍵入個人訊息，然後按一下 [選取]，來邀請他們加入群組。 邀請會自動傳送給受邀的使用者。
     
     ![用來新增來賓成員的新增邀請按鈕](./media/add-users-administrator/GroupInvite.png)
   
您也可以搭配使用動態群組與 Azure AD B2B 共同作業。 如需詳細資訊，請參閱[動態群組與 Azure Active Directory B2B 共同作業](use-dynamic-groups.md)。

## <a name="add-guest-users-to-an-application"></a>將來賓使用者新增到應用程式

若要以 Azure AD 系統管理員身分將 B2B 共同作業使用者新增到應用程式，請遵循下列步驟：

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導覽窗格中，選取 [Azure Active Directory]。
3. 在 [管理] 下選取 [企業應用程式] > [所有應用程式]。
4. 選取要對其新增來賓使用者的應用程式。
5. 在應用程式的儀表板上，選取 [使用者總數] 以開啟 [使用者和群組] 窗格。

    ![用來開啟使用者和群組的使用者總數按鈕](./media/add-users-administrator/AppUsersAndGroups.png)

6. 選取 [新增使用者]。
7. 在 [新增指派] 下，選取 [使用者和群組]。
8. 執行下列其中一項：
   - 如果目錄中已有來賓使用者，請搜尋 B2B 使用者。 選取使用者，按一下 [選取]，然後按一下 [指派] 將使用者新增至應用程式中。
   - 如果目錄中還沒有來賓使用者，請選取 [邀請]。
           
       ![用來新增來賓成員的新增邀請按鈕](./media/add-users-administrator/AppInviteUsers.png)
   
      在 [邀請來賓] 下輸入電子郵件地址，選擇性鍵入輸入個人訊息，然後選取 [邀請]。 按一下 [選取]，然後按一下 [指派] 將使用者新增至應用程式中。 邀請會自動傳送給受邀的使用者。

9. 來賓使用者會出現在應用程式的 [使用者和群組] 清單中，具有 [預設存取] 的指派角色。 如果您想要變更角色，請執行下列作業：
   - 選取來賓使用者，然後選取 [編輯]。 
   - 在 [編輯指派] 底下按一下 [選取角色]，然後選取您想要指派給所選使用者的角色。
   - 按一下 [選取] 。
   - 按一下 [指派]。
 
## <a name="resend-invitations-to-guest-users"></a>將邀請重新傳送給來賓使用者

如果來賓使用者尚未兌換其邀請，您可以重新傳送邀請電子郵件。

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 在導覽窗格中，選取 [Azure Active Directory]。
3. 在 [管理] 底下選取 [使用者]。
5. 選取使用者帳戶。
6. 在 [管理] 底下選取 [設定檔]。
7. 如果使用者尚未接受邀請，則 [重新傳送邀請] 選項會是可供使用的狀態。 選取此按鈕即可重新傳送。

   ![使用者設定檔中的 [重新傳送邀請] 選項](./media/add-users-administrator/Resend-Invitation.png)

> [!NOTE]
> 如果您重新傳送的邀請原本會將使用者導向特定應用程式，請務必了解，新邀請中的連結會改為將使用者導向最上層的存取面板。

## <a name="next-steps"></a>後續步驟

- 若要了解非 Azure AD 系統管理員要如何新增 B2B 來賓使用者，請參閱[資訊工作者如何新增 B2B 共同作業使用者？](add-users-information-worker.md)
- 如需關於邀請電子郵件的資訊，請參閱[B2B 共同作業邀請電子郵件的元素](invitation-email-elements.md)。

