---
title: 教學課程：Azure Active Directory 與 TOPdesk - Secure 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 TOPdesk - Secure 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 8e06ee33-18f9-4c05-9168-e6b162079d88
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2018
ms.author: jeedes
ms.openlocfilehash: 8529dfda5ee4a7fc3360f91163b7f5f5bbf6c6ff
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42140617"
---
# <a name="tutorial-azure-active-directory-integration-with-topdesk---secure"></a>教學課程：Azure Active Directory 與 TOPdesk - Secure 整合

在本教學課程中，您將了解如何整合 TOPdesk - Secure 與 Azure Active Directory (Azure AD)。

將 TOPdesk - Secure 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 TOPdesk - Secure 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 TOPdesk - Secure (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 TOPdesk - Secure 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 TOPdesk - Secure 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 TOPdesk - Secure
1. 設定並測試 Azure AD 單一登入

## <a name="adding-topdesk---secure-from-the-gallery"></a>從資源庫新增 TOPdesk - Secure

若要設定將 TOPdesk - Secure 整合到 Azure AD 中，您需要將 TOPdesk - Secure 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 TOPdesk - Secure，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **TOPdesk - Secure**，從結果面板中選取 [TOPdesk - Secure]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 TOPdesk - Secure](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 TOPdesk - Secure 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 TOPdesk - Secure 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 TOPdesk - Secure 中的相關使用者之間建立連結關聯性。

在 TOPdesk - Secure 中，指派 Azure AD 中的**使用者名稱**值作為 [Username] \(使用者名稱\) 的值，以建立連結關聯性。

若要設定及測試與 TOPdesk - Secure 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 TOPdesk - Secure 測試使用者](#create-a-topdesk---secure-test-user)** - 在 TOPdesk - Secure 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的對應項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 TOPdesk - Secure 應用程式中設定單一登入。

**若要設定與 TOPdesk - Secure 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [TOPdesk - Secure] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_samlbase.png)

3. 在 [TOPdesk - Secure 網域及 URL] 區段上，執行下列步驟：

    ![TOPdesk - Secure 網域及 URL 單一登入資訊](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.topdesk.net`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<companyname>.topdesk.net/tas/secure/login/verify`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<companyname>.topdesk.net/tas/public/login/saml`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 「回覆 URL」在本教學課程中稍後會提供說明。 請連絡 [TOPdesk - Secure 用戶端支援小組](http://www.topdesk.com/us/support) \(英文\) 以取得這些值。 

4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/topdesk-secure-tutorial/tutorial_general_400.png)

6. 在 [TOPdesk - Secure 設定] 區段上，按一下 [設定 TOPdesk - Secure] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![TOPdesk - Secure 設定](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_configure.png)

7. 以系統管理員身分登入您的 **TOPdesk - Secure** 公司網站。

8. 在 [TOPdesk] 功能表中按一下 [設定]。

    ![設定](./media/topdesk-secure-tutorial/ic790598.png "設定")

9. 按一下 [登入設定] 。

    ![登入設定](./media/topdesk-secure-tutorial/ic790599.png "登入設定")

10. 展開 [登入設定] 功能表，然後按一下 [一般]。

    ![一般](./media/topdesk-secure-tutorial/ic790600.png "一般")

11. 在 [SAML 登入] 組態區段的 **Secure** 區段中，執行下列步驟：

    ![技術設定](./media/topdesk-secure-tutorial/ic790855.png "技術設定")

    a. 按 [下載]  來下載公用中繼資料檔案，然後再將它儲存在本機電腦上。

    b. 開啟此中繼資料檔案，然後找到 **AssertionConsumerService** 節點。

    ![判斷提示取用者服務](./media/topdesk-secure-tutorial/ic790856.png "判斷提示取用者服務")

    c. 複製 **AssertionConsumerService** 值，在 [TOPdesk - Secure 網域及 URL] 區段的 [回覆 URL] 文字方塊中，貼上此值。

12. 若要建立憑證檔案，請執行下列步驟：

    ![憑證](./media/topdesk-secure-tutorial/ic790606.png "憑證")

    a. 從 Azure 入口網站開啟下載的中繼資料檔案。

    b. 展開 **RoleDescriptor** 節點，其具有 **fed:ApplicationServiceType** 的 **xsi:type**。

    c. 複製 **X509Certificate** 節點的值。

    d. 儲存複製的 **X509Certificate** 值到本機電腦的檔案中。

13. 在 **Public** 區段中，按一下 [加入]。

    ![新增](./media/topdesk-secure-tutorial/ic790607.png "新增")

14. 在 [SAML 組態輔助程式]  對話頁面上，執行下列步驟：

    ![SAML 組態輔助程式](./media/topdesk-secure-tutorial/ic790608.png "SAML 組態輔助程式")

    a. 若要上傳您從 Azure 入口網站下載的中繼資料檔案，請在 [同盟中繼資料] 下按一下 [瀏覽]。

    b. 若要上傳您的憑證檔案，請在 [憑證 (RSA)] 下按一下 [瀏覽]。

    c. 針對 **私密金鑰 (RSA、PKCS8、DER)**，您可以上傳您自己的私密金鑰，也可以連絡 [TOPdesk - Secure 用戶端支援小組](http://www.topdesk.com/us/support)以取得私密金鑰。

    d. 若要上傳您從 TOPdesk 支援小組取得的標誌檔案，請在 [標誌圖示] 下按一下 [瀏覽]。

    e. 在 [使用者名稱屬性] 文字方塊中，輸入 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`。

    f. 在 [顯示名稱]  文字方塊中，輸入您的組態名稱。

    g. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/topdesk-secure-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/topdesk-secure-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/topdesk-secure-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/topdesk-secure-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-topdesk---secure-test-user"></a>建立 TOPdesk - Secure 測試使用者

若要讓 Azure AD 使用者可以登入 TOPdesk - Secure，則必須將他們佈建到 TOPdesk - Secure。  
TOPdesk - Secure 需以手動的方式佈建。

### <a name="to-configure-user-provisioning-perform-the-following-steps"></a>若要設定使用者佈建，請執行下列步驟：

1. 以系統管理員身分登入您的 **TOPdesk - Secure** 公司網站。

2. 在上方功能表中按一下 [TOPdesk] \> [新增] \> [支援檔案] \> [操作員]。

    ![操作員](./media/topdesk-secure-tutorial/ic790610.png "操作員")

3. 在 [新增操作員]  對話方塊上，執行下列步驟：

    ![新增操作員](./media/topdesk-secure-tutorial/ic790611.png "新增操作員")

    a. 按一下 [General] \(一般\) 索引標籤。

    b. 在 [Surname] \(姓氏\) 文字方塊中，輸入使用者的姓氏，例如 **Simon**。

    c. 在 [位置] 區段中選取該帳戶的 [網站]。

    d. 在 [TOPdesk 登入] 區段的 [登入名稱] 文字方塊中，輸入使用者的登入名稱。

    e. 按一下 [檔案] 。

> [!NOTE]
> 您可以使用任何其他的 TOPdesk - Secure 使用者帳戶建立工具或 TOPdesk - Secure 提供的 API 來佈建 AAD 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 TOPdesk - Secure 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 TOPdesk - Secure，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [TOPdesk - Secure]。

    ![應用程式清單中的 [TOPdesk - Secure] 連結](./media/topdesk-secure-tutorial/tutorial_topdesk-secure_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [TOPdesk - Secure] 圖格時，應該會自動登入您的 TOPdesk - Secure 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/topdesk-secure-tutorial/tutorial_general_01.png
[2]: ./media/topdesk-secure-tutorial/tutorial_general_02.png
[3]: ./media/topdesk-secure-tutorial/tutorial_general_03.png
[4]: ./media/topdesk-secure-tutorial/tutorial_general_04.png

[100]: ./media/topdesk-secure-tutorial/tutorial_general_100.png

[200]: ./media/topdesk-secure-tutorial/tutorial_general_200.png
[201]: ./media/topdesk-secure-tutorial/tutorial_general_201.png
[202]: ./media/topdesk-secure-tutorial/tutorial_general_202.png
[203]: ./media/topdesk-secure-tutorial/tutorial_general_203.png