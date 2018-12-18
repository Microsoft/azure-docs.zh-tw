---
title: 教學課程：Azure Active Directory 與 Boomi 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Boomi 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 40d034ff-7394-4713-923d-1f8f2ed8bf36
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/03/2018
ms.author: jeedes
ms.openlocfilehash: cf925e0e0e7b6b4c10b6b21d17214f91473a9026
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432944"
---
# <a name="tutorial-azure-active-directory-integration-with-boomi"></a>教學課程：Azure Active Directory 與 Boomi 整合

在本教學課程中，您會了解如何整合 Boomi 與 Azure Active Directory (Azure AD)。

Boomi 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Boomi 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Boomi (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Boomi 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Boomi 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Boomi
1. 設定並測試 Azure AD 單一登入

## <a name="adding-boomi-from-the-gallery"></a>從資源庫新增 Boomi
若要設定將 Boomi 整合至 Azure AD 中，您需要從資源庫將 Boomi 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 Boomi，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Boomi**，從結果面板中選取 [Boomi]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Boomi](./media/boomi-tutorial/tutorial_boomi_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Boomi 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Boomi 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Boomi 中的相關使用者之間建立連結關聯性。

在 BeeLine 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Boomi 搭配運作的 Azure AD 單一登入，您需要完成下列基本工作：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Boomi 測試使用者](#create-a-boomi-test-user)** - 在 Boomi 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Boomi 應用程式中設定單一登入。

**若要設定與 Boomi 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Boomi] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/boomi-tutorial/tutorial_boomi_samlbase.png)

1. 在 [Boomi 網域與 URL] 區段中，執行下列步驟：

    ![Boomi 網域及 URL 單一登入資訊](./media/boomi-tutorial/tutorial_boomi_url.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://platform.boomi.com/`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://platform.boomi.com/sso/<boomi-tenant>/saml`

    > [!NOTE] 
    > [回覆 URL] 不是真實的值。 請使用實際的「回覆 URL」來更新此值。 請連絡 [Boomi 支援小組](https://boomi.com/company/contact/) \(英文\) 以取得此值。
 
1. Boomi 應用程式需要特定格式的 SAML 判斷提示。 請設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定單一登入](./media/boomi-tutorial/tutorial_attribute.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，針對下表顯示的每一列執行下列步驟：

    | 屬性名稱 | 屬性值 |
    | -------------- | --------------- |
    | FEDERATION_ID | user.mail |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。
    
    ![設定單一登入](./media/boomi-tutorial/tutorial_officespace_04.png)
    
    ![設定單一登入](./media/boomi-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定] 。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/boomi-tutorial/tutorial_boomi_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/boomi-tutorial/tutorial_general_400.png)

1. 在 [Boomi 組態] 區段上，按一下 [設定 Boomi] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![Boomi 設定](./media/boomi-tutorial/tutorial_boomi_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Boomi 公司網站。 

1. 瀏覽至 [公司名稱]，並移至 [設定]。

1. 按一下 [SSO 選項] 索引標籤並執行下列步驟。

    ![在應用程式端設定單一登入](./media/boomi-tutorial/tutorial_boomi_11.png)

    a. 選取 [啟用 SAML 單一登入] 核取方塊。

    b. 按一下 [匯入]，將已從 Azure AD 下載的憑證上傳至**識別提供者憑證**。
    
    c. 在 [識別提供者登入 URL] 文字方塊中，放入來自 Azure AD 應用程式組態視窗的 [SAML 單一登入服務 URL] 值。

    d. 針對 [同盟識別碼位置]，選取 [同盟識別碼位於 FEDERATION_ID 屬性元素] 選項按鈕。 

    e. 按一下 [儲存]  按鈕。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/boomi-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/boomi-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/boomi-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/boomi-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
  
### <a name="create-a-boomi-test-user"></a>建立 Boomi 測試使用者

若要讓 Azure AD 使用者可以登入 Boomi，必須將他們佈建到 Boomi。 Boomi 需以手動方式佈建。

### <a name="to-provision-a-user-account-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：

1. 以系統管理員身分登入您的 Boomi 公司網站。

1. 登入之後，瀏覽至 [使用者管理]，然後移至 [使用者]。

    ![使用者](./media/boomi-tutorial/tutorial_boomi_001.png "使用者")

1. 按一下 **+** 圖示，[新增/維護使用者角色] 對話方塊隨即開啟。

    ![使用者](./media/boomi-tutorial/tutorial_boomi_002.png "使用者")

    ![使用者](./media/boomi-tutorial/tutorial_boomi_003.png "使用者")

    a. 在 [使用者電子郵件地址] 文字方塊中，輸入像是 BrittaSimon@contoso.com 的使用者電子郵件。
    
    b. 在 [名字] 文字方塊中，輸入使用者的名字，例如 Britta。

    c. 在 [姓氏] 文字方塊中，輸入使用者的姓氏，例如 Simon。
    
    d. 輸入使用者的 [同盟識別碼]。 每位使用者必須有同盟識別碼，以唯一地識別帳戶內的使用者。
    
    e. 指派 [標準使用者] 角色給使用者。 請勿指派系統管理員角色，因為這樣他會取得正常 Atmosphere 存取權和單一登入存取權。
    
    f. 按一下 [確定]。
    
    > [!NOTE]
    > 使用者的密碼是透過識別提供者來管理，他不會收到附上 AtomSphere 帳戶登入密碼的歡迎通知電子郵件。 您可以使用任何其他的 Boomi 使用者帳戶建立工具或 Boomi 提供的 API 來佈建 AAD 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Boomi 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派至 Boomi，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Boomi]。

    ![應用程式清單中的 Boomi 連結](./media/boomi-tutorial/tutorial_boomi_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 [Boomi] 圖格時，應該會自動登入您的 Boomi 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/boomi-tutorial/tutorial_general_01.png
[2]: ./media/boomi-tutorial/tutorial_general_02.png
[3]: ./media/boomi-tutorial/tutorial_general_03.png
[4]: ./media/boomi-tutorial/tutorial_general_04.png

[100]: ./media/boomi-tutorial/tutorial_general_100.png

[200]: ./media/boomi-tutorial/tutorial_general_200.png
[201]: ./media/boomi-tutorial/tutorial_general_201.png
[202]: ./media/boomi-tutorial/tutorial_general_202.png
[203]: ./media/boomi-tutorial/tutorial_general_203.png

