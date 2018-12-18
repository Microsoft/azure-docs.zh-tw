---
title: 教學課程：Azure Active Directory 與 Workday 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Workday 之間的單一登入。
services: active-directory
documentationCenter: na
author: cmmdesai
manager: mtillman
ms.reviewer: jeedes
ms.assetid: e9da692e-4a65-4231-8ab3-bc9a87b10bca
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/11/2018
ms.author: chmutali
ms.openlocfilehash: 78b9fe704c5c8a1f81da480787f1791e88bf4f72
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44714713"
---
# <a name="tutorial-azure-active-directory-integration-with-workday"></a>教學課程：Azure Active Directory 與 Workday 整合

在本教學課程中，您將了解如何整合 Workday 與 Azure Active Directory (Azure AD)。

Workday 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Workday 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Workday (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Workday 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Workday 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Workday
2. 設定並測試 Azure AD 單一登入

## <a name="adding-workday-from-the-gallery"></a>從資源庫新增 Workday

若要設定將 Workday 整合到 Azure AD 中，您需要從資源庫將 Workday 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Workday，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Workday**，從結果面板中選取 [Workday]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Workday](./media/workday-tutorial/tutorial_workday_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Workday 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Workday 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Workday 中的相關使用者之間建立連結關聯性。

在 Workday 中，指派 Azure AD 中**使用者名稱**的值作為 [Username] \(使用者名稱\) 的值，以建立連結關聯性。

若要使用 Workday 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Workday 測試使用者](#create-a-workday-test-user)** - 在 Workday 中建立一個與 Azure AD 中代表使用者之項目連結的 Britta Simon 對應項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Workday 應用程式中設定單一登入。

**若要使用 Workday 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Workday] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/workday-tutorial/tutorial_workday_samlbase.png)

3. 在 [Workday 網域與 URL] 區段上，執行下列步驟：

    ![Workday 網域及 URL 單一登入資訊](./media/workday-tutorial/tutorial_workday_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://impl.workday.com/<tenant>/login-saml2.htmld`

    b. 在 [識別碼] 文字方塊中，輸入 URL：`http://www.workday.com`

4. 勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Workday 網域及 URL 單一登入資訊](./media/workday-tutorial/tutorial_workday_url1.png)

    在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://impl.workday.com/<tenant>/login-saml.htmld`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的「登入 URL」及「回覆 URL」來更新這些值。 您的回覆 URL 必須有子網域 (例如：www、wd2、wd3、wd3-impl、wd5、wd5-impl)。
    > 使用 "http://www.myworkday.com" 等形式可以運作，但 "http://myworkday.com" 不行。 請連絡 [Workday 客戶支援小組](https://www.workday.com/en-us/partners-services/services/support.html)以取得這些值。

5. Workday 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示此設定的範例。

    ![設定單一登入](./media/Workday-tutorial/tutorial_workday_attributes.png)

    > [!NOTE]
    > 依預設，這裡已使名稱識別碼和 UPN (user.userprincipalname) 相對應。 您必須將名稱識別碼對應至 Workday 帳戶中的實際使用者識別碼 (您的電子郵件或 UPN 等)，才能讓 SSO 順利運作。

6. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/workday-tutorial/tutorial_workday_certificate.png)

7. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/workday-tutorial/tutorial_general_400.png)

8. 在 [Workday 組態] 區段上，按一下 [設定 Workday] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![Workday 設定](./media/workday-tutorial/tutorial_workday_configure.png)

9. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Workday 公司網站。

10. 在**搜尋方塊**中，使用首頁左上方的名稱 [Edit Tenant Setup – Security] \(編輯租用戶設定 – 安全性\) 來進行搜尋。

    ![編輯租用戶安全性](./media/workday-tutorial/IC782925.png "編輯租用戶安全性")

11. 在 [重新導向 URL]  區段中，執行下列步驟：

    ![重新導向 URL](./media/workday-tutorial/IC7829581.png "重新導向 URL")

    a. 按一下 [加入資料列]。

    b. 在 [登入重新導向 URL] 文字方塊和 [行動裝置重新導向 URL] 文字方塊中，輸入您在 Azure 入口網站的 [Workday 網域與 URL] 區段上輸入的 [登入 URL]。

    c. 在 Azure 入口網站的 [設定登入] 視窗上，複製 [登出 URL]，然後將它貼至 [登出重新導向 URL] 文字方塊中。

    d. 在 [Used for Environments] \(用於環境\) 文字方塊中，選取環境名稱。  

    >[!NOTE]
    > [環境] 屬性的值會繫結至租用戶 URL 的值：  
    >-如果 Workday 租用戶 URL 的網域名稱開頭為 impl (例如：*https://impl.workday.com/\<tenant\>/login-saml2.htmld*)，則 **Environment** 屬性必須設為 Implementation。  
    >-如果網域名稱的開頭是其他字元，您需要連絡 [Workday 客戶支援小組](https://www.workday.com/en-us/partners-services/services/support.html)以取得相符的 [環境] 值。

12. 在 [SAML 設定]  區段中，執行下列步驟：

    ![SAML 設定](./media/workday-tutorial/IC782926.png "SAML 設定")

    a.  按一下 [啟用 SAML 驗證] 。

    b.  按一下 [加入資料列]。

13. 在 [SAML Identity Providers] \(SAML 身分識別提供者\) 區段中，執行下列步驟：

    ![SAML 身分識別提供者](./media/workday-tutorial/IC7829271.png "SAML 身分識別提供者")

    a. 在 [Identity Provider Name] \(身分識別提供者名稱\) 文字方塊中，輸入提供者名稱 (例如：*SPInitiatedSSO*)。

    b. 在 Azure 入口網站的 [設定登入] 視窗上，複製 [SAML 實體識別碼] 值，然後將它貼至 [簽發者] 文字方塊中。

    ![SAML 身分識別提供者](./media/workday-tutorial/IC7829272.png "SAML 身分識別提供者")

    c. 在 Azure 入口網站的 [設定登入] 視窗上，複製 [登出 URL] 值，然後將它貼至 [登出回應 URL] 文字方塊中。

    d. 在 Azure 入口網站的 [設定登入] 視窗上，複製 [SAML 單一登入服務 URL] 值，然後將它貼至 [IdP SSO 服務 URL] 文字方塊中。

    e. 在 [Used for Environments] \(用於環境\) 文字方塊中，選取環境名稱。

    f. 按一下 [識別提供者公開金鑰憑證]，然後按一下 [建立]。

    ![建立](./media/workday-tutorial/IC782928.png "建立")

    g. 按一下 [建立 x509 公開金鑰]。

    ![建立](./media/workday-tutorial/IC782929.png "建立")

14. 在 [檢視 x509 公開金鑰]  區段中，執行下列步驟：

    ![檢視 x509 公開金鑰](./media/workday-tutorial/IC782930.png "檢視 x509 公開金鑰")

    a. 在 [名稱] 文字方塊中，輸入您的憑證名稱 (例如：PPE\_SP)。

    b. 在 [有效開始日期] 文字方塊中輸入憑證屬性值的有效開始日期。

    c.  在 [有效結束日期] 文字方塊中輸入憑證屬性值的有效結束日期。

    > [!NOTE]
    > 按兩下所下載的憑證，即可取得有效開始日期和有效結束日期。  這些日期會列在 [詳細資料]  索引標籤之下。
    >
    >

    d.  在記事本中開啟 base-64 編碼的憑證，然後複製其內容。

    e.  在 [憑證] 文字方塊中貼上剪貼簿的內容。

    f.  按一下 [確定]。

15. 執行下列步驟：

    ![SSO 組態](./media/workday-tutorial/WorkdaySSOConfiguratio.png "SSO 組態")

    a.  在 [服務提供者識別碼] 文字方塊中，輸入 **http://www.workday.com**。

    b. 選取 [不要壓縮 SP 起始的驗證要求]。

    c. 選取 **SHA256** 做為 [驗證要求簽章方法]。

    ![驗證要求簽章方法](./media/workday-tutorial/WorkdaySSOConfiguration.png "驗證要求簽章方法") 

    d. 按一下 [確定]。

    ![確定](./media/workday-tutorial/IC782933.png "確定")

    > [!NOTE]
    > 請確定您已正確設定單一登入。 如果您以不正確的設定啟用單一登入，您可能無法使用認證進入應用程式，並且會遭到鎖定。在此情況下，Workday 提供備份的登入 URL，使用者可以透過此格式：[Your Workday URL]/login.flex?redirect=n，使用他們的一般使用者名稱和密碼來登入

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/workday-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/workday-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/workday-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/workday-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-workday-test-user"></a>建立 Workday 測試使用者

在本節中，您會在 Workday 中建立名為 Britta Simon 的使用者。 請與 [ 客戶支援小組](https://www.workday.com/en-us/partners-services/services/support.html)合作，在 Workday 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Workday 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Workday，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Workday]。

    ![應用程式清單中的 Workday 連結](./media/workday-tutorial/tutorial_workday_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Workday] 圖格時，應該會自動登入您的 Workday 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)


<!--Image references-->

[1]: ./media/workday-tutorial/tutorial_general_01.png
[2]: ./media/workday-tutorial/tutorial_general_02.png
[3]: ./media/workday-tutorial/tutorial_general_03.png
[4]: ./media/workday-tutorial/tutorial_general_04.png

[100]: ./media/workday-tutorial/tutorial_general_100.png

[200]: ./media/workday-tutorial/tutorial_general_200.png
[201]: ./media/workday-tutorial/tutorial_general_201.png
[202]: ./media/workday-tutorial/tutorial_general_202.png
[203]: ./media/workday-tutorial/tutorial_general_203.png
