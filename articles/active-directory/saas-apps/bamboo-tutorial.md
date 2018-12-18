---
title: 教學課程：Azure Active Directory 與 SAML SSO for Bamboo by resolution GmbH 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAML SSO for Bamboo by resolution GmbH 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: f00160c7-f4cc-43bf-af18-f04168d3767c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/04/2017
ms.author: jeedes
ms.openlocfilehash: 95aada1303a807034d22689f71cea37696df4154
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432451"
---
# <a name="tutorial-azure-active-directory-integration-with-saml-sso-for-bamboo-by-resolution-gmbh"></a>教學課程：Azure Active Directory 與 SAML SSO for Bamboo by resolution GmbH 整合

在本教學課程中，您將了解如何整合 SAML SSO for Bamboo by resolution GmbH 與 Azure Active Directory (Azure AD)。

SAML SSO for Bamboo by resolution GmbH 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 SAML SSO for Bamboo by resolution GmbH 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶，自動登入 SAML SSO for Bamboo by resolution GmbH (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SAML SSO for Bamboo by resolution GmbH 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 SAML SSO for Bamboo by resolution GmbH 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 SAML SSO for Bamboo by resolution GmbH
1. 設定並測試 Azure AD 單一登入

## <a name="adding-saml-sso-for-bamboo-by-resolution-gmbh-from-the-gallery"></a>從資源庫新增 SAML SSO for Bamboo by resolution GmbH
若要設定 SAML SSO for Bamboo by resolution GmbH 與 Azure AD 的整合，您必須從資源庫將 SAML SSO for Bamboo by resolution GmbH 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 SAML SSO for Bamboo by resolution GmbH，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **SAML SSO for Bamboo by resolution GmbH**、從結果面板中選取 [SAML SSO for Bamboo by resolution GmbH]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 SAML SSO for Bamboo by resolution GmbH](./media/bamboo-tutorial/tutorial_bamboo_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 SAML SSO for Bamboo by resolution GmbH 設定及測試 Azure AD 單一登入。

為了讓單一登入運作，Azure AD 必須知道 SAML SSO for Bamboo by resolution GmbH 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 SAML SSO for Bamboo by resolution GmbH 中的相關使用者之間建立連結關聯性。

在 SAML SSO for Bamboo by resolution GmbH 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 SAML SSO for Bamboo by resolution GmbH 設定及測試 Azure AD 單一登入，您需要完成下列基本要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 SAML SSO for Bamboo by resolution GmbH 測試使用者](#create-a-saml-sso-for-bamboo-by-resolution-gmbh-test-user)**：讓 SAML SSO for Bamboo by resolution GmbH 中有對應 Britta Simon 的使用者，並連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 SAML SSO for Bamboo by resolution GmbH 應用程式中設定單一登入。

**若要使用 SAML SSO for Bamboo by resolution GmbH 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SAML SSO for Bamboo by resolution GmbH] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/bamboo-tutorial/tutorial_bamboo_samlbase.png)

1. 如果您想要以 IDP 起始模式設定應用程式，請在 [SAML SSO for Bamboo by resolution GmbH 網域和 URL] 區段上執行下列步驟：

    ![SAML SSO for Bamboo by resolution GmbH 網域和 URL 單一登入資訊](./media/bamboo-tutorial/tutorial_bamboo_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<server-base-url>/plugins/servlet/samlsso`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<server-base-url>/plugins/servlet/samlsso`

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]，然後執行下列步驟：

    ![SAML SSO for Bamboo by resolution GmbH 網域和 URL 單一登入資訊](./media/bamboo-tutorial/tutorial_bamboo_url1.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<server-base-url>/plugins/servlet/samlsso`
     
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 請連絡 [SAML SSO for Bamboo by resolution GmbH 用戶端支援小組](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bamboo/server/support) \(英文\) 以取得這些值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/bamboo-tutorial/tutorial_bamboo_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/bamboo-tutorial/tutorial_general_400.png)

1. 以系統管理員身分登入 SAML SSO for Bamboo by resolution GmbH 公司網站。

1. 在主要工具列右邊，按一下 [Settings] \(設定\) > [Add-ons] \(附加元件\)。

    ![設定](./media/bamboo-tutorial/tutorial_bamboo_setings.png)

1. 移至 [SECURITY] \(安全性\) 區段，按一下功能表列上的 [SAML SingleSignOn] \(SAML 單一登入\)。

    ![Samlsingle](./media/bamboo-tutorial/tutorial_bamboo_samlsingle.png)

1. 在 [SAML 單一登入外掛程式設定] 頁面上，按一下 [新增 idp]。 

    ![新增 idp](./media/bamboo-tutorial/tutorial_bamboo_addidp.png)

1. 在 [選擇您的 SAML 識別提供者] 頁面上，執行下列步驟：

    ![識別提供者](./media/bamboo-tutorial/tutorial_bamboo_identityprovider.png)

    a. 針對 [Idp 類型] 選取 [AZURE AD]。

    b. 在 [名稱] 文字方塊中，輸入名稱。

    c. 在 [描述] 文字方塊中，輸入描述。

    d. 按 [下一步] 。

1. 在 [Identity provider configuration] \(識別提供者設定\) 頁面上，按一下 [Next] \(下一步\) 按鈕。

    ![識別設定](./media/bamboo-tutorial/tutorial_bamboo_identityconfig.png)

1.  在 [Import SAML Idp Metadata] \(匯入 SAML Idp 中繼資料\) 頁面上，按一下 [Load File] \(載入檔案\)，以上傳您已從 Azure 入口網站下載的**中繼資料 XML** 檔案。

    ![Idpmetadata](./media/bamboo-tutorial/tutorial_bamboo_idpmetadata.png)

1. 按 [下一步] 。

1. 按一下 [儲存設定]。

    ![儲存](./media/bamboo-tutorial/tutorial_bamboo_save.png)
    
> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/bamboo-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/bamboo-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/bamboo-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/bamboo-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-saml-sso-for-bamboo-by-resolution-gmbh-test-user"></a>建立 SAML SSO for Bamboo by resolution GmbH 測試使用者

本節目標是在 SAML SSO for Bamboo by resolution GmbH 中建立名為 Britta Simon 的使用者。 SAML SSO for Bamboo by resolution GmbH 支援 Just-In-Time 佈建，而且也可以手動建立使用者。請根據您的需求來連絡 [SAML SSO for Bamboo by resolution GmbH 用戶端支援小組](https://marketplace.atlassian.com/plugins/com.resolution.atlasplugins.samlsso-bamboo/server/support) \(英文\)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SAML SSO for Bamboo by resolution GmbH 的存取權授與 Britta Simon，使其能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派至 SAML SSO for Bamboo by resolution GmbH，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [SAML SSO for Bamboo by resolution GmbH]。

    ![應用程式清單中的 SAML SSO for Bamboo by resolution GmbH](./media/bamboo-tutorial/tutorial_bamboo_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您按一下 [存取面板] 中的 [SAML SSO for Bamboo by resolution GmbH] 磚時，您應該會自動登入 SAML SSO for Bamboo by resolution GmbH 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/bamboo-tutorial/tutorial_general_01.png
[2]: ./media/bamboo-tutorial/tutorial_general_02.png
[3]: ./media/bamboo-tutorial/tutorial_general_03.png
[4]: ./media/bamboo-tutorial/tutorial_general_04.png

[100]: ./media/bamboo-tutorial/tutorial_general_100.png

[200]: ./media/bamboo-tutorial/tutorial_general_200.png
[201]: ./media/bamboo-tutorial/tutorial_general_201.png
[202]: ./media/bamboo-tutorial/tutorial_general_202.png
[203]: ./media/bamboo-tutorial/tutorial_general_203.png

