---
title: 教學課程：Azure Active Directory 與 Absorb LMS 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Absorb LMS 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: ba9f1b3d-a4a0-4ff7-b0e7-428e0ed92142
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2017
ms.author: jeedes
ms.openlocfilehash: 066ae92056e4b80b6627b371d6ebeb3235b2781d
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39043773"
---
# <a name="tutorial-azure-active-directory-integration-with-absorb-lms"></a>教學課程：Azure Active Directory 與 Absorb LMS

在本教學課程中，您將了解如何整合 Absorb LMS 與 Azure Active Directory (Azure AD)。

將 Absorb LMS 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Absorb LMS 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Absorb LMS (透過單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

若您想深入了解軟體即服務 (SaaS) 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Absorb LMS 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Absorb LMS 單一登入功能的訂用帳戶

> [!NOTE]
> 不建議使用此教學課程的實際執行環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

* 從資源庫新增 Absorb LMS
* 設定並測試 Azure AD 單一登入

## <a name="add-absorb-lms-from-the-gallery"></a>從資源庫新增 Absorb LMS
若要設定將 Absorb LMS 整合到 Azure AD 中，請從資源庫將 Absorb LMS 新增到受控 SaaS 應用程式清單。

若要從資源庫新增 Absorb LMS，請執行下列操作：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 移至 [企業應用程式] > [所有應用程式]。

    ![[企業應用程式] 窗格][2]
    
3. 若要新增應用程式，請選取 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Absorb LMS**、選取結果面板中的 [Absorb LMS]，然後選取 [新增] 按鈕。

    ![結果清單中的 Absorb LMS](./media/absorblms-tutorial/tutorial_absorblms_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 Britta Simon 的測試使用者為基礎，設定及測試與 Absorb LMS 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Absorb LMS 對應使用者位於 Azure AD 中。 換句話說，您必須建立 Azure AD 使用者和 Absorb LMS 中對應使用者之間的連結關聯性。

建立此連結關聯性的方法是將 Azure AD 中的*使用者名稱*值指定為 Absorb LMS 中的 *Username* 值。

若要設定及測試與 Absorb LMS 搭配運作的 Azure AD 單一登入，請完成接下來五個區段中的建置區塊。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Absorb LMS 應用程式中設定單一登入。

若要設定與 Absorb LMS 搭配運作的 Azure AD 單一登入，請執行下列操作：

1. 在 Azure 入口網站的 [Absorb LMS] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊的 [模式] 方塊中，選取 [SAML 登入] 來啟用單一登入。
 
    ![單一登入對話方塊](./media/absorblms-tutorial/tutorial_absorblms_samlbase.png)

3. 在 [Absorb LMS 網域與 URL] 區段中，執行下列操作：

    ![Absorb LMS 網域與 URL 單一登入資訊](./media/absorblms-tutorial/tutorial_absorblms_url.png)

    a. 在 [識別碼] 方塊中，輸入使用下列語法的 URL：`https://<subdomain>.myabsorb.com/Account/SAML`。

    b. 在 [回覆 URL] 方塊中，輸入使用下列語法的 URL：`https://<subdomain>.myabsorb.com/Account/SAML`。
     
    > [!NOTE] 
    > 這些 URL 不是真正的值。 請使用實際的識別碼和回覆 URL 來更新這些值。 若要取得這些值，請連絡 [Absorb LMS 用戶端支援小組](https://www.absorblms.com/support)。 

4. 在 [SAML 簽署憑證] 區段的 [下載] 欄中，選取 [中繼資料 XML]，然後將中繼資料檔案儲存到您的電腦中。

    ![簽署憑證下載連結](./media/absorblms-tutorial/tutorial_absorblms_certificate.png) 

5. 選取 [ **儲存**]。

    ![設定單一登入儲存按鈕](./media/absorblms-tutorial/tutorial_general_400.png)
    
6. 在 [Absorb LMS 設定] 區段中，選取 [設定 Absorb LMS] 來開啟 [設定登入] 視中，然後再複製 [快速參考] 區段中的 [登出 URL]。

    ![[Absorb LMS 設定] 窗格](./media/absorblms-tutorial/tutorial_absorblms_configure.png) 

7. 在新的網頁瀏覽器視窗中，以系統管理員身分登入您的 Absorb LMS 公司網站。

8. 選取右上方的 [帳戶] 按鈕。 

    ![[帳戶] 按鈕](./media/absorblms-tutorial/1.png)

9. 在 [帳戶] 窗格中，選取 [入口網站設定]。

    ![[入口網站設定] 連結](./media/absorblms-tutorial/2.png)
    
10. 選取 [使用者]  索引標籤。

    ![[使用者] 索引標籤](./media/absorblms-tutorial/3.png)

11. 在 [單一登入設定] 頁面上，執行下列操作：

    ![[單一登入設定] 頁面](./media/absorblms-tutorial/4.png)

    a. 在 [模式] 方塊中，選取 [識別提供者起始]。

    b. 在 [記事本] 中，開啟您從 Azure 入口網站下載的憑證。 移除 **---BEGIN CERTIFICATE---** 和 **---END CERTIFICATE---** 標記。 接著，在 [金鑰] 方塊中，貼上其餘的內容。
    
    c. 在 [ID 屬性] 方塊中，選取您在 Azure AD 中設定為使用者識別碼的屬性。 例如，如果已在 Azure AD 中選取 *userPrincipalName*，請選取 [使用者名稱]。

    d. 在 [登入 URL] 方塊中，從應用程式的 Azure 入口網站 [屬性] 頁面貼上**使用者存取 URL**。

    e. 在 [登出 URL] 中，貼上您從 Azure 入口網站的 [設定登入] 視窗所複製的**登出 URL** 值。

12. 將 [只允許 SSO 登入] 切換為 [開啟]。

    ![[只允許 SSO 登入] 切換](./media/absorblms-tutorial/5.png)

13. 選取 [儲存]。

> [!TIP]
> 當您設定應用程式時，可以在 [Azure 入口網站](https://portal.azure.com)中閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增應用程式之後，選取 [單一登入] 索引標籤，即可透過底部的 [設定] 區段存取內嵌的文件。 如需詳細資訊，請參閱 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您會在 Azure 入口網站中建立測試使用者 Britta Simon。

![建立 Azure AD 測試使用者][100]

若要在 Azure AD 中建立測試使用者，請執行下列動作：

1. 在 Azure 入口網站的左側窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 按鈕](./media/absorblms-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請選取 [使用者和群組] > [所有使用者]。
    
    ![[使用者和群組] 與 [所有使用者] 連結](./media/absorblms-tutorial/create_aaduser_02.png) 

3. 在對話方塊頂端，選取 [新增]。
 
    ![[新增] 按鈕](./media/absorblms-tutorial/create_aaduser_03.png) 

4. 在 [使用者] 對話方塊中，執行下列動作：
 
    ![[使用者] 對話方塊](./media/absorblms-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中的值。

    d. 選取 [建立] 。

### <a name="create-an-absorb-lms-test-user"></a>建立 Absorb LMS 測試使用者

若要將 Azure AD 使用者登入 Absorb LMS，必須在 Absorb LMS 中設定這些使用者。  

就 Absorb LMS 而言，設定是手動工作。

若要設定使用者帳戶，請執行下列操作：

1. 以系統管理員身分登入您的 Absorb LMS 公司網站。

2. 在左側窗格中，選取 [使用者]。

    ![[Absorb LMS 使用者] 連結](./media/absorblms-tutorial/absorblms_users.png)

3. 在 [使用者] 窗格中，選取 [使用者]。

    ![[使用者] 連結](./media/absorblms-tutorial/absorblms_userssub.png)

4. 在 [新增] 下拉式清單中，選取 [使用者]。

    ![[新增] 下拉式清單](./media/absorblms-tutorial/absorblms_createuser.png)

5. 在 [新增使用者] 頁面上，執行下列操作：

    ![[新增使用者] 頁面](./media/absorblms-tutorial/user.png)

    a. 在 [名字] 方塊中，輸入名字，例如 **Britta**。

    b. 在 [姓氏] 方塊中，輸入姓氏，例如 **Simon**。
    
    c. 在 [使用者名稱] 方塊中，輸入全名，例如 **Britta Simon**。

    d. 在 [密碼] 方塊中，輸入 Britta Simon 的密碼。

    e. 在 [確認密碼] 方塊中，重新輸入密碼。
    
    f. 將 [為使用中] 切換設定為 [使用中]。  

6. 選取 [儲存]。
 
### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Absorb LMS 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

若要將使用者 Britta Simon 指派給 Absorb LMS，請執行下列操作：

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至 [目錄] 檢視，然後選取 [企業應用程式] > [所有應用程式]。

    ![[所有應用程式] 連結][201] 

2. 在 [應用程式] 清單中，選取 [Absorb LMS]。

    ![應用程式清單中的 Absorb LMS 連結](./media/absorblms-tutorial/tutorial_absorblms_app.png) 

3. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202] 

4. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊的 [使用者] 清單中，選取 [Britta Simon]。

6. 在 [使用者和群組] 對話方塊中，選取 [選取] 按鈕。

7. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

在 [存取面板] 中，選取 [Absorb LMS] 磚，就會自動將您登入 Absorb LMS 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/absorblms-tutorial/tutorial_general_01.png
[2]: ./media/absorblms-tutorial/tutorial_general_02.png
[3]: ./media/absorblms-tutorial/tutorial_general_03.png
[4]: ./media/absorblms-tutorial/tutorial_general_04.png

[100]: ./media/absorblms-tutorial/tutorial_general_100.png

[200]: ./media/absorblms-tutorial/tutorial_general_200.png
[201]: ./media/absorblms-tutorial/tutorial_general_201.png
[202]: ./media/absorblms-tutorial/tutorial_general_202.png
[203]: ./media/absorblms-tutorial/tutorial_general_203.png
