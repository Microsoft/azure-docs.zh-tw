---
title: 教學課程：Azure Active Directory 與 Adobe Creative Cloud 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Adobe Creative Cloud 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: c199073f-02ce-45c2-b515-8285d4bbbca2
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/10/2018
ms.author: jeedes
ms.openlocfilehash: 506f52faf916aa0d5ca2e8587bdbcc16ab88e130
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39054279"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-creative-cloud"></a>教學課程：Azure Active Directory 與 Adobe Creative Cloud 整合

在本教學課程中，您會了解如何整合 Adobe Creative Cloud 與 Azure Active Directory (Azure AD)。

Adobe Creative Cloud 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Adobe Creative Cloud 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Adobe Creative Cloud (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Adobe Creative Cloud 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Adobe Creative Cloud 單一登入的訂閱
- 需要 Adobe Creative Cloud Enterprise 版

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Adobe Creative Cloud
2. 設定並測試 Azure AD 單一登入

## <a name="adding-adobe-creative-cloud-from-the-gallery"></a>從資源庫新增 Adobe Creative Cloud

若要設定將 Adobe Creative Cloud 整合至 Azure AD 中，您需要從資源庫將 Adobe Creative Cloud 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 Adobe Creative Cloud，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Adobe Creative Cloud**，從結果面板選取 [Adobe Creative Cloud]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Adobe Creative Cloud](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Adobe Creative Cloud 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Adobe Creative Cloud 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Adobe Creative Cloud 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Adobe Creative Cloud 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Adobe Creative Cloud 測試使用者](#create-an-adobe-creative-cloud-test-user)** - 在 Adobe Creative Cloud 中，建立一個與 Azure AD 中代表使用者的項目連結的 Britta Simon 對應項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Adobe Creative Cloud 應用程式中設定單一登入。

**若要設定與 Adobe Creative Cloud 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Adobe Creative Cloud] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_samlbase.png)

3. 如果您想要以 IDP 起始模式設定應用程式，請在 [Adobe Creative Cloud 網域和 URL] 區段上執行下列步驟：

    ![Adobe Creative Cloud 網域和 URL 單一登入資訊](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://www.okta.com/saml2/service-provider/<token>`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<company name>.okta.com/auth/saml20/accauthlinktest`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 若要取得這些值，請連絡 [Adobe Creative Cloud Enterprise](https://www.adobe.com/au/creativecloud/business/teams/plans.html)。

4. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![Adobe Creative Cloud 網域和 URL 單一登入資訊](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_url2.png)

    在 [登入 URL] 文字方塊中，輸入下列值：`https://adobe.com`

5. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_certificate.png)
     
6. Adobe Creative Cloud 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以從應用程式的 [使用者屬性] 索引標籤管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入](./media/adobe-creative-cloud-tutorial/tutorial_attribute.png)

7. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱 | 屬性值 |
    | ---------------| ----------------|
    | 名字 |user.givenname |
    | 姓氏 |user.surname |
    | 電子郵件 |user.mail |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/adobe-creative-cloud-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/adobe-creative-cloud-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 按一下 [確定] 。

    > [!NOTE]
    > 使用者需要具備有效的 Office 365 ExO 授權，電子郵件宣告值才會填入 SAML 回應中。

8. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/adobe-creative-cloud-tutorial/tutorial_general_400.png)

9. 在 [Adobe Creative Cloud 組態] 區段中，按一下 [設定 Adobe Creative Cloud] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![Adobe Creative Cloud 設定](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_configure.png)

10. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 [Adobe Admin Console](https://adminconsole.adobe.com)。

11. 移至上方導覽列中的 [設定]，然後選擇 [身分識別]。 網域清單隨即開啟。 按一下您網域的 [設定] 連結。 接著執行 [需要單一登入組態] 區段中的下列步驟。 如需詳細資訊，請參閱[設定網域](https://helpx.adobe.com/enterprise/using/set-up-domain.html)。

    ![設定](https://helpx.adobe.com/content/dam/help/en/enterprise/using/configure-microsoft-azure-with-adobe-sso/_jcr_content/main-pars/procedure_719391630/proc_par/step_3/step_par/image/edit-sso-configuration.png "設定")

    a. 按一下 [瀏覽]，將已從 Azure AD 下載的憑證上傳至 **IDP 憑證**。

    b. 在 [IDP 簽發者] 文字方塊中，放入 [SAML 實體 ID] 的值，該值是您從 Azure 入口網站的 [設定登入] 區段複製而來。

    c. 在 [IDP 登入 URL] 文字方塊中，放入 [SAML SSO 服務 URL] 的值，該值是您從 Azure 入口網站的 [設定登入] 區段複製而來。

    d. 選取 [HTTP - 重新導向] 作為 **IDP 繫結**。

    e. 選取 [電子郵件地址] 作為**使用者登入設定**。

    f. 按一下 [儲存]  按鈕。

12. 儀表板現在會顯示 XML「下載中繼資料」檔案。 它包含 Adobe 的 EntityDescriptor URL 和 AssertionConsumerService URL。 請開啟檔案，然後在 Azure AD 應用程式中加以設定。

    ![在應用程式端設定單一登入](./media/adobe-creative-cloud-tutorial/tutorial_adobe-creative-cloud_003.png)

    a. 在 [設定應用程式設定] 對話方塊上，使用 Adobe 提供給您的 EntityDescriptor 值作為**識別碼**。

    b. 在 [設定應用程式設定] 對話方塊上，使用 Adobe 提供給您的 AssertionConsumerService 值作為**回覆 URL**。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/adobe-creative-cloud-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/adobe-creative-cloud-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/adobe-creative-cloud-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/adobe-creative-cloud-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-an-adobe-creative-cloud-test-user"></a>建立 Adobe Creative Cloud 測試使用者

若要讓 Azure AD 使用者能夠登入 Adobe Creative Cloud，必須將他們佈建到 Adobe Creative Cloud。 Adobe Creative Cloud 需以手動方式佈建。

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：

1. 以系統管理員身分登入 [Adobe Admin Console](https://adminconsole.adobe.com)。

2. 在 Adobe 主控台內將使用者新增為同盟識別碼，並將它們指派給產品設定檔。 如需新增使用者的詳細資訊，請參閱[在 Adobe Admin Console 中新增使用者](https://helpx.adobe.com/enterprise/using/users.html#Addusers) 

3. 此時，在 Adobe 登入表單中輸入您的電子郵件地址/upn，按下 tab 鍵，隨後應備份至 Azure AD 同盟：
    * Web 存取：www.adobe.com > 登入
    * 在桌面應用程式公用程式內 > 登入
    * 在應用程式內 > 說明 > 登入

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Adobe Creative Cloud 的存取權授與 Britta Simon，讓他能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Adobe Creative Cloud，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Adobe Creative Cloud]。

    ![應用程式清單中的 Adobe Creative Cloud 連結](./media/adobe-creative-cloud-tutorial/tutorial_adobecreativecloud_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Adobe Creative Cloud] 圖格時，應該會自動登入您的 Adobe Creative Cloud 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定網域 (adobe.com)](https://helpx.adobe.com/enterprise/using/set-up-domain.html)
* [設定 Azure 以便搭配 Adobe SSO (adobe.com) 使用](https://helpx.adobe.com/enterprise/kb/configure-microsoft-azure-with-adobe-sso.html)

<!--Image references-->

[1]: ./media/adobe-creative-cloud-tutorial/tutorial_general_01.png
[2]: ./media/adobe-creative-cloud-tutorial/tutorial_general_02.png
[3]: ./media/adobe-creative-cloud-tutorial/tutorial_general_03.png
[4]: ./media/adobe-creative-cloud-tutorial/tutorial_general_04.png

[100]: ./media/adobe-creative-cloud-tutorial/tutorial_general_100.png

[200]: ./media/adobe-creative-cloud-tutorial/tutorial_general_200.png
[201]: ./media/adobe-creative-cloud-tutorial/tutorial_general_201.png
[202]: ./media/adobe-creative-cloud-tutorial/tutorial_general_202.png
[203]: ./media/adobe-creative-cloud-tutorial/tutorial_general_203.png
