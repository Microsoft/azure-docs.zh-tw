---
title: 教學課程：針對 Azure AD B2B 來賓使用者強制執行多重要素驗證
description: 了解當您使用 Azure AD B2B 來與外部使用者和合作夥伴組織共同作業時，如何要求使用多重要素驗證 (MFA)。
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: tutorial
ms.date: 06/25/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 34ca5667b5bf47bec8c48a7e5836b7d51ad08724
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45986812"
---
# <a name="tutorial-enforce-multi-factor-authentication-for-b2b-guest-users"></a>教學課程：針對 B2B 來賓使用者強制執行多重要素驗證

當與外部 B2B 來賓使用者共同作業時，使用多重要素驗證 (MFA) 原則來保護您的應用程式是個不錯的做法。 如此一來，外部使用者在存取您的資源時，將不僅只是需要使用者名稱和密碼。 在 Azure Active Directory (Azure AD) 中，您可以使用要求使用 MFA 進行存取的條件式存取原則來達成此目標。 MFA 原則可以在租用戶、應用程式或個別來賓使用者層級強制執行，方式就像針對您自己組織的成員啟用這些原則一樣。

範例：

![要求使用 MFA 的 B2B 應用程式](media/tutorial-mfa/aad-b2b-mfa-example.png)

1.  A 公司的系統管理員或員工邀請某位來賓使用者使用已設定為要求使用 MFA 進行存取的雲端或內部部署應用程式。
2.  來賓使用者以自己的工作、學校或社交身分識別登入。 
3.  系統要求使用者完成 MFA 挑戰。 
4.  使用者設定與 A 公司搭配使用的 MFA，然後選擇其 MFA 選項。 系統允許使用者存取應用程式。

在此教學課程中，您將：

> [!div class="checklist"]
> * 在設定 MFA 之前先測試登入體驗。
> * 建立要求使用 MFA 來存取您環境中雲端應用程式的條件式存取原則。 在此教學課程中，我們將使用「Microsoft Azure 管理」應用程式來說明程序。
> * 使用 What If 工具來模擬 MFA 登入。
> * 測試您的條件式存取原則。
> * 清除測試使用者和原則。

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="prerequisites"></a>先決條件

若要完成此教學課程中的案例，您需要：

 - **Azure AD Premium 版的存取權**，這包括條件式存取原則功能。 若要強制執行 MFA，您必須建立 Azure AD 條件式存取原則。 請注意，不論合作夥伴是否具備 MFA 功能，MFA 原則一律會在您個組織強制執行。 如果您已為組織設定 MFA，將必須確定您有足夠的 Azure AD Premium 授權可供來賓使用者使用。 
 - 一個可供您新增至租用戶目錄作為來賓使用者並用來登入的**有效外部電子郵件帳戶**。 如果您不知道如何建立來賓帳戶，請參閱[在 Azure 入口網站中新增 B2B 來賓使用者](add-users-administrator.md)。

## <a name="create-a-test-guest-user-in-azure-ad"></a>在 Azure AD 中建立測試來賓使用者

1. 以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在左窗格中，選取 [Azure Active Directory]。
3.  在 [管理] 底下選取 [使用者]。
4.  選取 [新增來賓使用者]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-user-3.png)

5.  在 [使用者名稱] 下，輸入外部使用者的電子郵件地址。 您也可以選擇納入歡迎訊息。 

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-user-4.png)

6.  選取 [邀請] 即可自動對來賓使用者傳送邀請。 將會顯示「已成功邀請使用者」訊息。 
7.  在傳送邀請之後，系統就會自動將使用者帳戶以來賓身分新增至目錄。

## <a name="test-the-sign-in-experience-before-mfa-setup"></a>在設定 MFA 之前先測試登入體驗
1.  使用您的測試使用者名稱和密碼來登入 [Azure 入口網站](https://portal.azure.com/)。
2.  請注意，您可以只使用您的登入認證來存取 Azure 入口網站。 不需要任何其他驗證。
3.  登出。

## <a name="create-a-conditional-access-policy-that-requires-mfa"></a>建立要求使用 MFA 的條件式存取原則
1.  以全域管理員、安全性系統管理員或條件式存取系統管理員的身分，登入 [Azure 入口網站](https://portal.azure.com/)。
2.  在 Azure 入口網站中，選取 [Azure Active Directory]。 
3.  在 [Azure Active Directory] 頁面的 [安全性] 區段中，選取 [條件式存取]。
4.  在 [條件式存取] 頁面頂端的工具列中，選取 [新增原則]。
5.  在 [新增] 頁面的 [名稱] 文字方塊中，輸入「必須使用 MFA 才能存取 B2B 入口網站」。
6.  在 [指派] 區段中，選取 [使用者和群組]。
7.  在 [使用者和群組] 頁面上，選擇 [選取使用者與群組]，然後選取 [所有來賓使用者 (預覽)]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-policy-6.png)
9.  選取 [完成] 。
10. 在 [新增] 頁面的 [指派] 區段中，選取 [雲端應用程式]。
11. 在 [雲端應用程式] 頁面上，選擇 [選取應用程式]，然後選擇 [選取]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-policy-10.png)

12. 在 [選取] 頁面上，選擇 [Microsoft Azure 管理]，然後選擇 [選取]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-policy-11.png)

13. 在 [雲端應用程式] 頁面上，選取 [完成]。
14. 在 [新增] 頁面的 [存取控制] 區段中，選取 [授與]。
15. 在 [授與] 頁面上，選擇 [授與存取權]，選取 [需要多重要素驗證] 核取方塊，然後選擇 [選取]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-policy-13.png)

16. 在 [啟用原則] 底下，選取 [開啟]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-policy-14.png)

17. 選取 [建立] 。

## <a name="use-the-what-if-option-to-simulate-sign-in"></a>使用 What If 選項來模擬登入

1.  在 [條件式存取 - 原則] 頁面上，選取 [What If]。 

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-whatif-1.png)

2.  選取 [使用者]，選擇您的測試來賓使用者，然後選擇 [選取]。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-whatif-2.png)

3.  選取 [雲端應用程式]。
4.  在 [雲端應用程式] 頁面上，選擇 [選取應用程式]，然後按一下 [選取]。 在應用程式清單中，選取 [Microsoft Azure 管理]，然後按一下 [選取]。 

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-whatif-3.png)

5.  在 [雲端應用程式] 頁面上，選取 [完成]。
6.  選取 [What If]，然後確認您的新原則出現在 [評估結果] 底下的 [會套用的原則] 索引標籤上。

    ![選取 Azure Active Directory](media/tutorial-mfa/tutorial-mfa-whatif-4.png)

## <a name="test-your-conditional-access-policy"></a>測試條件式存取原則
1.  使用您的測試使用者名稱和密碼來登入 [Azure 入口網站](https://portal.azure.com/)。
2.  您應該會看到需要其他驗證方法的要求。 請注意，可能需要一些時間，原則才會生效。

    ![選取 Azure Active Directory](media/tutorial-mfa/mfa-required.png)
 
3.  登出。

## <a name="clean-up-resources"></a>清除資源
當您不再需要測試使用者和測試條件式存取原則時，請將其移除。
1.  以 Azure AD 系統管理員身分登入 [Azure 入口網站](https://portal.azure.com/)。
2.  在左窗格中，選取 [Azure Active Directory]。
3.  在 [管理] 底下選取 [使用者]。
4.  選取測試使用者，然後選取 [刪除使用者]。
5.  在左窗格中，選取 [Azure Active Directory]。
6.  在 [安全性] 底下，選取 [條件式存取]。
7.  在 [原則名稱] 清單中，選取您測試原則的操作功能表 (...)，然後選取 [刪除]。 選取 [是] 以進行確認。
## <a name="next-steps"></a>後續步驟
在此教學課程中，您已建立會要求來賓使用者在登入您其中一個雲端應用程式時使用 MFA 的條件式存取原則。 若要深入了解如何新增共同作業的來賓使用者，請參閱[在 Azure 入口網站中新增 Azure Active Directory B2B 共同作業使用者](add-users-administrator.md)。
