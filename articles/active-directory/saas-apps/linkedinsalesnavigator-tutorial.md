---
title: 教學課程：Azure Active Directory 與 LinkedIn Sales Navigator 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 LinkedIn Sales Navigator 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 7a9fa8f3-d611-4ffe-8d50-04e9586b24da
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: f0e34a614251cf11c9547d749fef58dfa8ca623a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425192"
---
# <a name="tutorial-azure-active-directory-integration-with-linkedin-sales-navigator"></a>教學課程：Azure Active Directory 與 LinkedIn Sales Navigator 整合

在本教學課程中，您將了解如何整合 LinkedIn Sales Navigator 與 Azure Active Directory (Azure AD)。

LinkedIn Sales Navigator 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 LinkedIn Sales Navigator 的人員
- 您可以讓使用者利用自己的 Azure AD 帳戶，來自動登入 LinkedIn Sales Navigator (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請瀏覽[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

如要設定 Azure AD 與 LinkedIn Sales Navigator 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 LinkedIn Sales Navigator 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 LinkedIn Sales Navigator
1. 設定並測試 Azure AD 單一登入

## <a name="adding-linkedin-sales-navigator-from-the-gallery"></a>從資源庫新增 LinkedIn Sales Navigator
如要設定將 LinkedIn Sales Navigator 整合到 Azure AD 中，您需要從資源庫中將 LinkedIn Sales Navigator 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 LinkedIn Sales Navigator，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **LinkedIn Sales Navigator**。

    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_search.png)

1. 在結果窗格中，選取 [LinkedIn Sales Navigator]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 LinkedIn Sales Navigator 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 LinkedIn Sales Navigator 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 LinkedIn Sales Navigator 中相關使用者之間的連結關聯性。

建立此連結關聯性的方法是將 Azure AD 中**使用者名稱**的值，指派為 LinkedIn Sales Navigator 中 **Username** 的值。

若要設定及測試與 LinkedIn Sales Navigator 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 LinkedIn Sales Navigator 測試使用者](#creating-a-linkedin-sales-navigator-test-user)** - 使 LinkedIn Sales Navigator 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 LinkedIn Sales Navigator 應用程式中設定單一登入。

**如要設定搭配 LinkedIn Sales Navigator 的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [LinkedIn Sales Navigator] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊的 [模式] 中選取 [SAML 型登入]，以啟用單一登入。
 
    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_samlbase.png)

1. 在不同的網頁瀏覽器視窗中，以管理員身分登入您的 **LinkedIn Sales Navigator** 網站。

1. 在 [帳戶中心] 中，按一下 [設定] 底下的 [全域設定]。 此外，請從下拉式清單中選取 [Sales Navigator]。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_01.png)

1. 按一下 [或按一下這裡以從表單載入和複製個別欄位]，並複製 [實體識別碼] 和 [判斷提示取用者存取 (ACS) URL]。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_031.png)

1. 如果您想要在 [IDP 起始] 模式下設定應用程式，請在 Azure 入口網站的 [LinkedIn Sales Navigator 網域和 URL] 區段下方執行下列步驟。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_url1.png)

    a. 在 [識別碼] 文字方塊中，輸入從 LinkedIn 入口網站複製的 [實體 ID] 

    b. 在 [回覆 URL] 文字方塊中，輸入從 LinkedIn 入口網站複製的 [判斷提示取用者存取 (ACS) URL]

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_url2.png)

    在 [登入 URL] 文字方塊中，以下列模式輸入值：`https://www.linkedin.com/checkpoint/enterprise/login/<account id>?application=salesNavigator`

1. **LinkedIn Sales Navigator** 應用程式需要特定格式的 SAML 判斷提示，要求您將自訂屬性對應新增到您的 SAML 權杖屬性組態。 下列螢幕擷取畫面顯示了一個範例。 [使用者識別碼] 的預設值是 **user.userprincipalname**，但是 LinkedIn Sales Navigator 預期它是與使用者電子郵件地址對應的值。 您可以使用清單中的 **user.mail** 屬性，或者根據組織組態使用適當的屬性值。 

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/updateusermail.png)
    
1. 在 [使用者屬性] 區段中，按一下 [檢視及編輯所有其他使用者屬性]，然後設定屬性。 使用者需要新增名稱為 **email**、**department**、**firstname** 和 **lastname** 的四個宣告，而且值必須分別與 **user.mail**、**user.department**、**user.givenname** 和 **user.surname** 對應

    | 屬性名稱 | 屬性值 |
    | --- | --- |    
    | 電子郵件| user.mail |
    | department| user.department |
    | firstname| user.givenname |
    | lastname| user.surname |
    
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/userattribute.png)
    
    a. 按一下 [新增屬性] 以開啟 [屬性] 對話方塊。
    
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/tutorial_attribute_04.png)
    
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/tutorial_attribute_05.png)
   
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定]。

1. 針對 **name** 屬性執行下列步驟：

    a. 按一下該屬性，以開啟 [編輯屬性] 視窗。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/url_update.png)

    b. 刪除**命名空間**中的 URL 值。
    
    c. 按一下 [確定] 以儲存設定。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_general_400.png)

1. 移至 [LinkedIn 系統管理員設定] 區段。 按一下 [上傳 XML 檔案] 以上傳您從 Azure 入口網站下載的中繼資料 XML 檔案。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_metadata_03.png)

1. 按一下 [開啟] 以啟用 SSO。 SSO 狀態會從 [未連線] 變更為 [已連線]

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedin_admin_05.png)


> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-linkedin-sales-navigator-test-user"></a>建立 LinkedIn Sales Navigator 測試使用者

LinkedIn Sales Navigator 應用程式支援及時 (JIT) 使用者佈建，而在驗證之後，應用程式會自動建立使用者。 啟用 [自動指派授權] 以指派授權給使用者。
   
   ![建立 Azure AD 測試使用者](./media/linkedinsalesnavigator-tutorial/LinkedinUserprovswitch.png)

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 LinkedIn Sales Navigator 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 LinkedIn Sales Navigator，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [LinkedIn Sales Navigator]。

    ![設定單一登入](./media/linkedinsalesnavigator-tutorial/tutorial_linkedinsalesnavigator_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您按一下存取面板中的 [LinkedIn Sales Navigator] 圖格時，系統應該會將您重新導向至組織的頁面，您必須在其中提供您個人的 LinkedIn 帳戶詳細資料。 它會連結您的個人帳戶與您的 LinkedIn 商務帳戶。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_01.png
[2]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_02.png
[3]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_03.png
[4]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_04.png

[100]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_100.png

[200]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_200.png
[201]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_201.png
[202]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_202.png
[203]: ./media/linkedinsalesnavigator-tutorial/tutorial_general_203.png

