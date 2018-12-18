---
title: 教學課程：Azure Active Directory 與 N2F - Expense reports 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 N2F - Expense reports 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: f56d53d7-5a08-490a-bfb9-78fefc2751ec
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/08/2018
ms.author: jeedes
ms.openlocfilehash: 27fb299bc3bbbbf75bdf40ae02eac627763ce6d4
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40007110"
---
# <a name="tutorial-azure-active-directory-integration-with-n2f---expense-reports"></a>教學課程：Azure Active Directory 與 N2F - Expense reports 整合

在本教學課程中，您將了解如何整合 N2F - Expense reports 與 Azure Active Directory (Azure AD)。

N2F - Expense reports 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 N2F - Expense reports 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 N2F - Expense reports (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)

## <a name="prerequisites"></a>必要條件

若要設定與 N2F - Expense reports 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 N2F - Expense reports 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 N2F - Expense reports
2. 設定並測試 Azure AD 單一登入

## <a name="adding-n2f---expense-reports-from-the-gallery"></a>從資源庫新增 N2F - Expense reports

若要設定將 N2F - Expense reports 整合到 Azure AD 中，您需要將 N2F - Expense reports 從資源庫新增到受控 SaaS app 清單。

**若要從資源庫新增 N2F - Expense reports，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **N2F - Expense reports**，從結果面板中選取 [N2F - Expense reports]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 N2F - Expense reports](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 N2F - Expense reports 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 N2F - Expense reports 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 N2F - Expense reports 中相關使用者之間的連結關聯性。

若要設定及測試與 N2F - Expense reports 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 N2F - Expense reports 測試使用者](#create-a-n2f---expense-reports-test-use)** - 在 N2F - Expense reports 中建立 Britta Simon 的對應項目，且該項目須與 Azure AD 中代表使用者的項目連結。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在 N2F - Expense reports 應用程式中設定單一登入。

**若要設定使用 N2F - Expense reports 的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [N2F - Expense reports] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_samlbase.png)

3. 如果您想要以 **IDP** 起始模式設定應用程式，則不需要在 [N2F - Expense reports 網域及 URL] 區段中執行任何步驟，因為應用程式已預先整合到 Azure。

    ![N2F - Expense reports 網域及 URL 單一登入資訊](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_url1.png)

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![N2F - Expense reports 網域及 URL 單一登入資訊](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_url2.png)

    在 [登入 URL] 文字方塊中，輸入 URL：`https://www.n2f.com/app/`

5. 在 [SAML 簽署憑證] 區段上，按一下 [複製] 按鈕以複製**應用程式同盟中繼資料 URL**，並將它貼到 [記事本]。

    ![憑證下載連結](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_certificate.png)

6. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/n2f-expensereports-tutorial/tutorial_general_400.png)

7. 在 [N2F - Expense reports 組態] 區段上，按一下 [設定N2F - Expense reports] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼]。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_configure.png)

8. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 N2F - Expense reports 公司網站。

9. 按一下 [設定]，然後選取下拉式清單中的 [進階設定]。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/configure1.png)

10. 選取 [帳戶設定] 索引標籤。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/configure2.png)

11. 選取 [驗證]，然後選取 [+ 新增驗證方法] 索引標籤。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/configure3.png)

12. 選取 [SAML Microsoft Office 365] 作為驗證方法。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/configure4.png)

13. 在 [驗證方法]  區段上，執行下列步驟：

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/configure5.png)

    a. 在 [實體識別碼] 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 實體識別碼**值。

    b. 在 [中繼資料 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [應用程式同盟中繼資料 Url] 值。

    c. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/n2f-expensereports-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/n2f-expensereports-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/n2f-expensereports-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/n2f-expensereports-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-n2f---expense-reports-test-user"></a>建立 N2F - Expense reports 測試使用者

為了讓 Azure AD 使用者能夠登入 N2F - Expense reports，必須將他們佈建到 N2F - Expense reports 中。 在 N2F - Expense reports 的情況下，需手動佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 N2F - Expense reports 公司網站。

2. 按一下 [設定]，然後選取下拉式清單中的 [進階設定]。

   ![N2F-費用的新增使用者](./media/n2f-expensereports-tutorial/configure1.png)

3. 從左側導覽面板中，選取 [使用者] 索引標籤。

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/user1.png)

4. 選取 [+ 新增使用者] 索引標籤。

   ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/user2.png)

5. 在 [使用者] 區段中，執行下列步驟：

    ![N2F - Expense reports 組態](./media/n2f-expensereports-tutorial/user3.png)

    a. 在 [電子郵件地址] 文字方塊中，輸入像是 **brittasimon@contoso.com** 的使用者電子郵件地址。

    b. 在 [名字] 文字方塊中，輸入使用者的名字，例如 **Britta**。

    c. 在 [名稱] 文字方塊中，輸入使用者的名稱，例如 **BrittaSimon**。

    d. 根據您的組織需求，選擇 [角色，直屬經理 (N+1)] 及 [部門]。

    e. 按一下 [驗證並傳送邀請]。

    > [!NOTE]
    > 如果您在新增使用者時遇到任何問題，請連絡 [N2F - Expense reports 支援小組](mailto:support@n2f.com)

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 N2F - Expense reports 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 N2F - Expense reports，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [N2F - Expense reports]。

    ![應用程式清單中的 N2F - Expense reports 連結](./media/n2f-expensereports-tutorial/tutorial_n2f-expensereports_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 N2F - Expense reports 圖格時，應該會自動登入您的 N2F - Expense reports 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/n2f-expensereports-tutorial/tutorial_general_01.png
[2]: ./media/n2f-expensereports-tutorial/tutorial_general_02.png
[3]: ./media/n2f-expensereports-tutorial/tutorial_general_03.png
[4]: ./media/n2f-expensereports-tutorial/tutorial_general_04.png

[100]: ./media/n2f-expensereports-tutorial/tutorial_general_100.png

[200]: ./media/n2f-expensereports-tutorial/tutorial_general_200.png
[201]: ./media/n2f-expensereports-tutorial/tutorial_general_201.png
[202]: ./media/n2f-expensereports-tutorial/tutorial_general_202.png
[203]: ./media/n2f-expensereports-tutorial/tutorial_general_203.png

