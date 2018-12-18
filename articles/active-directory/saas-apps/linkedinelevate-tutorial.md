---
title: 教學課程：Azure Active Directory 與 LinkedIn Elevate 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 LinkedIn Elevate 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 2ad9941b-c574-42c3-bd0f-5d6ec68537ef
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2018
ms.author: jeedes
ms.openlocfilehash: 8b11b5e3e420577590e95c6839673f54c52d078b
ms.sourcegitcommit: 4eddd89f8f2406f9605d1a46796caf188c458f64
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49116171"
---
# <a name="tutorial-azure-active-directory-integration-with-linkedin-elevate"></a>教學課程：Azure Active Directory 與 LinkedIn Elevate 整合

在本教學課程中，您將了解如何整合 LinkedIn Elevate 與 Azure Active Directory (Azure AD)。

LinkedIn Elevate 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 LinkedIn Elevate 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 LinkedIn Elevate (單一登入)
- 您可以在 Azure 管理入口網站中集中管理您的帳戶

若您想了解 SaaS app 與 Azure AD 整合的更多詳細資訊，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 LinkedIn Elevate 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 LinkedIn Elevate 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 LinkedIn Elevate
1. 設定並測試 Azure AD 單一登入

## <a name="adding-linkedin-elevate-from-the-gallery"></a>從資源庫新增 LinkedIn Elevate
若要設定將 LinkedIn Elevate 整合到 Azure AD 中，您需要從資源庫將 LinkedIn Elevate 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 LinkedIn Elevate，請執行下列步驟：**

1. 在 **[Azure 管理入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]

1. 按一下對話方塊頂端的 [新增] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **LinkedIn Elevate**。 從結果面板中，按一下 [LinkedIn Elevate] 以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/tutorial-linkedinElevate_000.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 LinkedIn Elevate 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 LinkedIn Elevate 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 LinkedIn Elevate 中的相關使用者之間建立連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中**使用者名稱**的值指派為 LinkedIn Elevate 中 **Username** 的值。

若要設定及測試與 LinkedIn Elevate 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 LinkedIn Elevate 測試使用者](#creating-a-linkedin-elevate-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 管理入口網站中啟用 Azure AD 單一登入，並在您的 LinkedIn Elevate 應用程式中設定單一登入。

**若要設定與 LinkedIn Elevate 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 管理入口網站的 [LinkedIn Elevate] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，選取 [SAML 型登入] 做為 [模式]，以啟用單一登入。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial-linkedin_01.png)

1. 在不同的網頁瀏覽器視窗中，以管理員身分登入您的 LinkedIn Elevate 租用戶。

1. 在 [帳戶中心] 中，按一下 [設定] 底下的 [全域設定]。 此外，從下拉式清單選取 [提升 - 提升 AAD 測試]。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_admin_01.png)

1. 按一下 **，或按一下「這裡」以從表單**  載入和複製個別欄位，並且複製 [實體 ID] 和 [判斷提示取用者存取 (ACS) URL]

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_admin_03.png)

1. 如果您想要在 **IDP 起始**模式中設定 SSO，請在 Azure 入口網站的 [LinkedIn Elevate 網域和 URL] 底下執行下列步驟

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_signon_01.png)

    a. 在 [識別碼] 文字方塊中，輸入從 LinkedIn 入口網站複製的 [實體 ID] 

    b. 在 [回覆 URL] 文字方塊中，輸入從 LinkedIn 入口網站複製的 [判斷提示取用者存取 (ACS) URL]

1. 如果您想要在 **SP 起始**中設定 SSO，則在組態區段中按一下 [顯示進階 URL] 設定，然後使用下列模式設定登入 URL：

    `https://www.linkedin.com/checkpoint/enterprise/login/<AccountId>?application=elevate&applicationInstanceId=<InstanceId>` 

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_signon_02.png) 

1. LinkedIn Elevate 應用程式需要特定格式的 SAML 判斷提示，要求您將自訂屬性對應新增到您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示上述的範例。 [使用者識別碼] 的預設值是 **user.userprincipalname**，但是 LinkedIn Elevate 預期它是與使用者電子郵件地址對應的值。 對此您可以使用清單中的 **user.mail** 屬性，或者根據組織組態使用適當的屬性值。

    ![設定單一登入](./media/linkedinelevate-tutorial/updateusermail.png)

1. 在 [使用者屬性] 區段中，按一下 [檢視及編輯所有其他使用者屬性]，然後設定屬性。 您必須新增名為**部門**的另一個宣告，且值必須對應至 **user.department**。

    | 屬性名稱 | 屬性值 |
    | --- | --- |
    | department| user.department |

      ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/userattribute.png)

      a. 按一下 [新增屬性]，以開啟屬性的詳細資料頁面，如下所示新增部門屬性

      ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/adduserattribute.png)

      b. 按一下 [確定] 儲存屬性。

      c. 將 **emailaddress** 屬性的名稱變更為 **email**。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial-linkedinElevate_certificate.png) 

1. 按一下 [檔案] 。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_general_400.png)

1. 移至 [LinkedIn 系統管理員設定] 區段。 按一下 [上傳 XML 檔案] 選項，將您剛剛從 Azure 入口網站下載的 XML 檔案上傳。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_metadata_03.png)

1. 按一下 [開啟] 以啟用 SSO。 SSO 狀態將會從 [未連線] 變更為 [已連線]

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial_linkedin_admin_05.png)

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節目標是在 Azure 管理入口網站中建立名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 管理入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。

    ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。

    ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：

    ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="creating-a-linkedin-elevate-test-user"></a>建立 LinkedIn Elevate 測試使用者

LinkedIn Elevate 應用程式支援即時使用者佈建，且在驗證後會在應用程式中自動建立使用者。 在 LinkedIn Elevate 入口網站的系統管理設定頁面上，將 [自動指派授權] 切換成啟用即時佈建，這個動作也會將授權指派給使用者。 LinkedIn Elevate 也支援自動使用者佈建，您可以在[這裡](linkedinelevate-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

   ![建立 Azure AD 測試使用者](./media/linkedinelevate-tutorial/LinkedinUserprovswitch.png)

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 LinkedIn Elevate 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 LinkedIn Elevate，請執行下列步驟：**

1. 在 Azure 管理入口網站中，開啟應用程式檢視，然後瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

1. 在應用程式清單中，選取 [LinkedIn Elevate]。

    ![設定單一登入](./media/linkedinelevate-tutorial/tutorial-linkedinElevate_0001.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 LinkedIn Elevate 圖格時，您應該會進入 Azure 登入頁面；成功登入之後，您應該會進入 LinkedIn Elevate 應用程式。

## <a name="additional-resources"></a>其他資源

* [教學課程︰以 Azure Active Directory 設定自動使用者佈建的 LinkedIn Elevate](linkedinelevate-provisioning-tutorial.md)
* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](linkedinelevate-provisioning-tutorial.md)

<!--Image references-->

[1]: ./media/linkedinElevate-tutorial/tutorial_general_01.png
[2]: ./media/linkedinElevate-tutorial/tutorial_general_02.png
[3]: ./media/linkedinElevate-tutorial/tutorial_general_03.png
[4]: ./media/linkedinElevate-tutorial/tutorial_general_04.png

[100]: ./media/linkedinElevate-tutorial/tutorial_general_100.png

[200]: ./media/linkedinElevate-tutorial/tutorial_general_200.png
[201]: ./media/linkedinElevate-tutorial/tutorial_general_201.png
[202]: ./media/linkedinElevate-tutorial/tutorial_general_202.png
[203]: ./media/linkedinElevate-tutorial/tutorial_general_203.png