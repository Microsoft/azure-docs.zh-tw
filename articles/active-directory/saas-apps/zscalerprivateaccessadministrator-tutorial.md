---
title: 教學課程：Azure Active Directory 與 Zscaler Private Access Administrator 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Zscaler Private Access Administrator 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: c87392a7-e7fe-4cdc-a8e6-afe1ed975172
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/08/2018
ms.author: jeedes
ms.openlocfilehash: 61b469ba5f64a52b87843432dfe60fe1d83ffec2
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39428378"
---
# <a name="tutorial-azure-active-directory-integration-with-zscaler-private-access-administrator"></a>教學課程：Azure Active Directory 與 Zscaler Private Access Administrator 整合

在本教學課程中，您將了解如何整合 Zscaler Private Access Administrator 與 Azure Active Directory (Azure AD)。

將 Zscaler Private Access Administrator 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Zscaler Private Access Administrator 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Zscaler Private Access Administrator (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Zscaler Private Access Administrator 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Zscaler Private Access Administrator 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Zscaler Private Access Administrator
1. 設定並測試 Azure AD 單一登入

## <a name="adding-zscaler-private-access-administrator-from-the-gallery"></a>從資源庫新增 Zscaler Private Access Administrator
若要設定將 Zscaler Private Access Administrator 整合到 Azure AD 中，您需要將 Zscaler Private Access Administrator 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Zscaler Private Access Administrator，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Zscaler Private Access Administrator**，從結果面板中選取 [Zscaler Private Access Administrator]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Zscaler Private Access Administrator](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Zscaler Private Access Administrator 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Zscaler Private Access Administrator 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Zscaler Private Access Administrator 中的相關使用者之間建立連結關聯性。

若要設定及測試與 Zscaler Private Access Administrator 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Zscaler Private Access Administrator 測試使用者](#create-a-zscaler-private-access-administrator-test-user)** - 在 Zscaler Private Access Administrator 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 管理入口網站中啟用 Azure AD 單一登入，然後在您的 Zscaler Private Access Administrator 應用程式中設定單一登入。

**若要設定與 Zscaler Private Access Administrator 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Zscaler Private Access Administrator] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Zscaler Private Access Administrator 網域及 URL] 區段上執行下列步驟：

    ![Zscaler Private Access Administrator 網域及 URL 單一登入資訊](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<subdomain>.private.zscaler.com/auth/metadata`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://<subdomain>.private.zscaler.com/auth/sso`

    c. 按一下 [顯示進階 URL 設定]

    d. 在 [轉送狀態] 文字方塊中，輸入一個值：`idpadminsso`

1.  如果您想要以 **SP** 起始模式設定應用程式，請執行下列步驟：

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<subdomain>.private.zscaler.com/auth/sso`

    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的「識別碼」、「回覆 URL」及「登入 URL」來更新這些值。 請連絡 [Zscaler Private Access Administrator 支援小組](https://help.zscaler.com/zpa-submit-ticket) \(英文\) 以取得這些值。
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_400.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入 Zscaler Private Access Administrator。

1. 按一下頂端的 [Administration] \(管理\)，然後瀏覽至 [AUTHENTICATION] \(驗證\) 區段並按一下 [IdP Configuration] \(IdP 設定\)。

    ![Zscaler Private Access Administrator 管理](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

1. 按一下右上角的 [Add IdP Configuration] \(新增 IdP 設定\)。 

    ![Zscaler Private Access Administrator 新增 IdP](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addpidp.png)

1. 在 [Add IdP Configuration] \(新增 IdP 設定\) 頁面上，執行下列步驟：
 
    ![Zscaler Private Access Administrator IdP 選取](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_idpselect.png)

    a. 按一下 [IdP Metadata File Upload] \(IdP 中繼資料檔案上傳\) 欄位中的 [Select File] \(選取檔案)，以上傳從 Azure AD 下載的中繼資料檔案。

    b. 它會從 Azure AD 讀取 **IdP 中繼資料**，並填入所有欄位資訊，如下所示。

    ![Zscaler Private Access Administrator IdP 設定](./media/zscalerprivateaccessadministrator-tutorial/idpconfig.png)

    c. 在 [Single Sign On] \(單一登入\) 選取 [Administrator] \(系統管理員\)。

    d. 從 [Domains] \(網域\) 欄位中選取您的網域。
    
    e. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/zscalerprivateaccessadministrator-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/zscalerprivateaccessadministrator-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/zscalerprivateaccessadministrator-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/zscalerprivateaccessadministrator-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
  
### <a name="create-a-zscaler-private-access-administrator-test-user"></a>建立 Zscaler Private Access Administrator 測試使用者

若要讓 Azure AD 使用者能夠登入 Zscaler Private Access Administrator，必須將他們佈建到 Zscaler Private Access Administrator 中。 就 Zscaler Private Access Administrator 而言，必須以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 Zscaler Private Access Administrator 公司網站。

1. 按一下頂端的 [Administration] \(管理\)，然後瀏覽至 [AUTHENTICATION] \(驗證\) 區段並按一下 [IdP Configuration] \(IdP 設定\)。

    ![Zscaler Private Access Administrator 管理](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_admin.png)

1. 從功能表左側按一下 [Administrators] \(系統管理員\)。

    ![Zscaler Private Access Administrator 系統管理員](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_adminstrator.png)

1. 按一下右上角的 [Add Administrator] \(新增系統管理員\)：

    ![Zscaler Private Access Administrator 新增管理員](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_addadmin.png)

1. 在 [Add Administrator] \(新增系統管理員\) 頁面上，執行下列步驟：

    ![Zscaler Private Access Administrator 管理員使用者](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_useradmin.png)

    a. 在 [Username] \(使用者名稱\) 文字方塊中，輸入使用者的電子郵件，例如 **BrittaSimon@contoso.com**。

    b. 在 [Password] \(密碼\) 文字方塊中，輸入密碼。

    c. 在 [Confirm Password] \(確認密碼\) 文字方塊中，輸入密碼。

    d. 在 [Role] \(角色\) 選取 [Zscaler Private Access Administrator]。

    e. 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件，例如 **BrittaSimon@contoso.com**。

    f. 在 [Phone] \(電話\) 文字方塊中，輸入電話號碼。

    g. 在 [Timezone] \(時區\) 文字方塊中，選取時區。

    h. 按一下 [檔案] 。  

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Zscaler Private Access Administrator 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Zscaler Private Access Administrator，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Zscaler Private Access Administrator]。

    ![應用程式清單中的 Zscaler Private Access Administrator 連結](./media/zscalerprivateaccessadministrator-tutorial/tutorial_zscalerprivateaccessadministrator_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Zscaler Private Access Administrator] 圖格時，應該會自動登入您的 Zscaler Private Access Administrator 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_01.png
[2]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_02.png
[3]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_03.png
[4]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_04.png

[100]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_100.png

[200]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_200.png
[201]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_201.png
[202]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_202.png
[203]: ./media/zscalerprivateaccessadministrator-tutorial/tutorial_general_203.png

