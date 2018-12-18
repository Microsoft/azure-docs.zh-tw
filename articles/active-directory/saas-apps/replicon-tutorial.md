---
title: 教學課程：Azure Active Directory 與 Replicon 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Replicon 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 02a62f15-917c-417c-8d80-fe685e3fd601
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/15/2018
ms.author: jeedes
ms.openlocfilehash: 10f4749524c4d4b35b83e5637d35f4ef9a8f9ab6
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39041325"
---
# <a name="tutorial-azure-active-directory-integration-with-replicon"></a>教學課程：Azure Active Directory 與 Replicon 整合

在本教學課程中，您將了解如何整合 Replicon 與 Azure Active Directory (Azure AD)。

將 Replicon 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Replicon 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Replicon (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Replicon 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Replicon 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Replicon
2. 設定並測試 Azure AD 單一登入

## <a name="adding-replicon-from-the-gallery"></a>從資源庫新增 Replicon
若要設定將 Replicon 整合到 Azure AD 中，您需要將 Replicon 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Replicon，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Replicon**，從結果面板中選取 [Replicon]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Replicon](./media/replicon-tutorial/tutorial_replicon_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Replicon 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Replicon 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Replicon 中的相關使用者之間建立連結關聯性。

在 Replicon 中，指派 Azure AD 中的**使用者名稱**值作為 [Username] \(使用者名稱\) 的值，以建立連結關聯性。

若要設定及測試與 Replicon 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Replicon 測試使用者](#create-a-replicon-test-user)** - 在 Replicon 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的對應項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Replicon 應用程式中設定單一登入。

**若要設定與 Replicon 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Replicon] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/replicon-tutorial/tutorial_replicon_samlbase.png)

3. 在 [Replicon 網域及 URL] 區段上，執行下列步驟：

    ![Replicon 網域及 URL 單一登入資訊](./media/replicon-tutorial/tutorial_replicon_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://na2.replicon.com/<companyname>/saml2/sp-sso/post`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://global.replicon.com/<companyname>`

    c. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://global.replicon.com/!/saml2/<companyname>/sso/post`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「單一登入 URL」、「識別碼」及「回覆 URL」來更新這些值。 請連絡 [Replicon 用戶端支援小組](https://www.replicon.com/customerzone/contact-support) \(英文\) 以取得這些值。 

4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/replicon-tutorial/tutorial_replicon_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/replicon-tutorial/tutorial_general_400.png)

6. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Replicon 公司網站。

7. 若要設定 SAML 2.0，請執行下列步驟：

    ![啟用 SAML 驗證](./media/replicon-tutorial/ic777805.png "啟用 SAML 驗證")

    a. 若要顯示 [EnableSAML Authentication2] 對話方塊，請將下列內容附加至您 URL 中公司機碼之後︰`/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

    * 下面會顯示完整 URL 的結構描述︰`https://na2.replicon.com/\<YourCompanyKey\>/services/SecurityService1.svc/help/test/EnableSAMLAuthentication2`

   b. 按一下 [+] 來展開 [v20Configuration] 區段。

   c. 按一下 [+] 來展開 [metaDataConfiguration] 區段。

   d. 按一下 [選擇檔案]，選取您的識別提供者中繼資料 XML 檔案，然後按一下 [提交]。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/replicon-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/replicon-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/replicon-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/replicon-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-replicon-test-user"></a>建立 Replicon 測試使用者

本節的目標是在 Replicon 中建立名為 Britta Simon 的使用者。 Replicon 支援自動使用者佈建，該功能預設為啟用。 您可以在[這裡](replicon-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

**如果您需要手動建立使用者，請執行下列步驟：**

1. 在 Web 瀏覽器視窗中，以系統管理員身分登入您的 Replicon 公司網站。

2. 移至 [管理] \> [使用者]。

    ![使用者](./media/replicon-tutorial/ic777806.png "使用者")

3. 按一下 [新增使用者] 。

    ![新增使用者](./media/replicon-tutorial/ic777807.png "新增使用者")

4. 在 [使用者設定檔]  區段中，執行下列步驟：

    ![使用者設定檔](./media/replicon-tutorial/ic777808.png "使用者設定檔")

    a. 在 [Login Name] \(登入名稱\) 文字方塊中，輸入您要佈建之 Azure AD 使用者的 Azure AD 電子郵件地址，例如 **BrittaSimon@contoso.com**。

    b. 針對 [驗證類型] 選取 [SSO]。

    c. 在 [部門]  文字方塊中，輸入使用者的部門。

    d. 針對 [員工類型] 選取 [系統管理員]。

    e. 按一下 [儲存使用者設定檔] 。

>[!NOTE]
>您可以使用任何其他 Replicon 使用者帳戶建立工具或 Replicon 所提供的 API 來佈建 Azure AD 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Replicon 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Replicon，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [Replicon]。

    ![應用程式清單中的 [Replicon] 連結](./media/replicon-tutorial/tutorial_replicon_app.png)

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Replicon] 圖格時，應該會自動登入您的 Replicon 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](replicon-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/replicon-tutorial/tutorial_general_01.png
[2]: ./media/replicon-tutorial/tutorial_general_02.png
[3]: ./media/replicon-tutorial/tutorial_general_03.png
[4]: ./media/replicon-tutorial/tutorial_general_04.png

[100]: ./media/replicon-tutorial/tutorial_general_100.png

[200]: ./media/replicon-tutorial/tutorial_general_200.png
[201]: ./media/replicon-tutorial/tutorial_general_201.png
[202]: ./media/replicon-tutorial/tutorial_general_202.png
[203]: ./media/replicon-tutorial/tutorial_general_203.png
