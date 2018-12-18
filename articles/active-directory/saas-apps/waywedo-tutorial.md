---
title: 教學課程：Azure Active Directory 與 Way We Do 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Way We Do 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 84fc4f36-ecd1-42c6-8a70-cb0f3dc15655
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/25/2018
ms.author: jeedes
ms.openlocfilehash: bc415ec7c577e221a1ab5af585dff5b4fc9ab7dc
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39259492"
---
# <a name="tutorial-azure-active-directory-integration-with-way-we-do"></a>教學課程：Azure Active Directory 與 Way We Do 整合

在本教學課程中，您將了解如何整合 Way We Do 與 Azure Active Directory (Azure AD)。

Way We Do 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Way We Do 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Way We Do (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Way We Do 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用單一登入的 Way We Do 訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Way We Do
2. 設定並測試 Azure AD 單一登入

## <a name="adding-way-we-do-from-the-gallery"></a>從資源庫新增 Way We Do
若要設定 Way We Do 與 Azure AD 整合，您需要從資源庫將 Way We Do 新增到受控的 SaaS 應用程式清單。

**若要從資源庫新增 Way We Do，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Way We Do**，從結果面板中選取 [Way We Do]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 [Way We Do]](./media/waywedo-tutorial/tutorial_waywedo_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Way We Do 設定和測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Way We Do 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Way We Do 中的相關使用者之間建立連結關聯性。

若要使用 Way We Do 來設定和測試 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Way We Do 測試使用者](#create-a-way-we-do-test-user)** - 使 Way We Do 中 Britta Simon 的對應使用者連結到使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Way We Do 應用程式中設定單一登入。

**若要使用 Way We Do 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Way We Do] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/waywedo-tutorial/tutorial_waywedo_samlbase.png)

3. 在 [Way We Do 網域及 URL] 區段上，執行下列步驟：

    ![[Way We Do 網域及 URL] 單一登入資訊](./media/waywedo-tutorial/tutorial_waywedo_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<SUBDOMAIN>.waywedo.com/Authentication/ExternalSignIn`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<SUBDOMAIN>.waywedo.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Way We Do 用戶端支援小組](mailto:support@waywedo.com)以取得這些值。 
 
4. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (原始)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/waywedo-tutorial/tutorial_waywedo_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/waywedo-tutorial/tutorial_general_400.png)

6. 在 [Way We Do 設定] 區段上，按一下 [設定 Way We Do] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![[Way We Do 設定]](./media/waywedo-tutorial/tutorial_waywedo_configure.png) 

7. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Way We Do。

8. 在 Way We Do 中的任何頁面上，按一下右上角的**人員圖示**，然後在下拉式清單中按一下 [Account] \(帳戶\)。

    ![Way We Do [Account] \(帳戶\)](./media/waywedo-tutorial/tutorial_waywedo_account.png) 

9. 按一下**功能表圖示**以開啟推送導覽功能表，然後按一下 [Single Sign On] \(單一登入\)。

    ![Way We Do 單一登入](./media/waywedo-tutorial/tutorial_waywedo_single.png)

10. 在 [Single sign-on setup] \(單一登入設定\) 頁面上，執行下列步驟：

    ![Way We Do [Save] \(儲存\)](./media/waywedo-tutorial/tutorial_waywedo_save.png)

    a. 按一下 [Turn on single sign-on] \(開啟單一登入\)，切換為 [Yes] \(是\) 以啟用單一登入。

    b. 在 [Single sign-on name] \(單一登入名稱\) 文字方塊中，輸入您的名稱。

    c. 在 [Entity ID] \(實體識別碼\) 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 實體識別碼**值。

    d. 在 [SAML SSO URL] 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 單一登入服務 URL** 值。

    e. 按一下 [Certificate] \(憑證\) 旁的 [select] \(選取\) 按鈕來上傳憑證。

    f. **選擇性設定** -
    
    * 啟用密碼：停用此選項時，會針對 Way We Do 使用一般密碼，讓使用者只能使用單一登入。

    * 啟用自動佈建：啟用此選項時，用於登入的電子郵件地址將自動與 Way We Do 中的使用者清單進行比較。 如果電子郵件地址與 Way We Do 中的作用中使用者不符，就會為登入的人員自動新增新的使用者帳戶，並要求該人員提供所有遺失的資訊。

      > [!NOTE]
      > 透過單一登入新增的使用者會被新增為一般使用者，且不會被指派系統中的角色。 系統管理員後續將能夠將這些人員的安全性角色修改為編輯者或系統管理員，同時也可以指派一或多個組織圖角色。 

    g. 按一下 [Save] \(儲存\) 以保存您的設定。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/waywedo-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/waywedo-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/waywedo-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/waywedo-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-way-we-do-test-user"></a>建立 Way We Do 測試使用者

本節目標是在 Way We Do 中建立名為 Britta Simon 的使用者。 Way We Do 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 嘗試存取 Way We Do 時，如果使用者還不存在，就會建立新使用者。

> [!Note]
> 如果您需要手動建立使用者，請連絡 [Way We Do 用戶端支援小組](mailto:support@waywedo.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Way We Do 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Way We Do，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Way We Do]。

    ![應用程式清單中的 [Way We Do] 連結](./media/waywedo-tutorial/tutorial_waywedo_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Way We Do] 磚時，應該會自動登入您的 Way We Do 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/waywedo-tutorial/tutorial_general_01.png
[2]: ./media/waywedo-tutorial/tutorial_general_02.png
[3]: ./media/waywedo-tutorial/tutorial_general_03.png
[4]: ./media/waywedo-tutorial/tutorial_general_04.png

[100]: ./media/waywedo-tutorial/tutorial_general_100.png

[200]: ./media/waywedo-tutorial/tutorial_general_200.png
[201]: ./media/waywedo-tutorial/tutorial_general_201.png
[202]: ./media/waywedo-tutorial/tutorial_general_202.png
[203]: ./media/waywedo-tutorial/tutorial_general_203.png

