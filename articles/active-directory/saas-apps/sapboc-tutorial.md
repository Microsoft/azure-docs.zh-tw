---
title: 教學課程：Azure Active Directory 與 SAP Business Object Cloud 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAP Business Object Cloud 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 6c5e44f0-4e52-463f-b879-834d80a55cdf
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/14/2017
ms.author: jeedes
ms.openlocfilehash: ffd4480a13549caba17becff27a43f51fcaa1988
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39041733"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-business-object-cloud"></a>教學課程：Azure Active Directory 與 SAP Business Object Cloud 整合

在本教學課程中，您將了解如何整合 SAP Business Object Cloud 與 Azure Active Directory (Azure AD)。

將 SAP Business Object Cloud 與 Azure AD 整合時，您會獲得下列優點：

- 您可以在 Azure AD 中控制可存取 SAP Business Object Cloud 的人員。
- 您可以使用單一登入和使用者的 Azure AD 帳戶，將您的使用者自動登入 SAP Business Object Cloud。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

若要深入了解軟體即服務 (SaaS) 應用程式與 Azure AD 的整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要進行 Azure AD 與 SAP Business Object Cloud 整合的設定，您需要下列項目：

- Azure AD 訂用帳戶
- 已開啟單一登入的 SAP Business Object Cloud

> [!NOTE]
> 若您要測試本教學課程中的步驟，不建議您在生產環境中進行測試。

在本教學課程中測試步驟的建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，可以[取得一個月的免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 SAP Business Object Cloud。
2. 設定和測試 Azure AD 單一登入。

## <a name="add-sap-business-object-cloud-from-the-gallery"></a>從資源庫新增 SAP Business Object Cloud
若要進行 SAP Business Object Cloud 與 Azure AD 整合的設定，請在資源庫中將 SAP Business Object Cloud 新增至受控 SaaS 應用程式清單中。

若要從資源庫新增 SAP Business Object Cloud：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側功能表中，選取 [Azure Active Directory]。 

    ![Azure Active Directory 按鈕][1]

2. 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![[企業應用程式] 頁面][2]
    
3. 若要新增新的應用程式，請選取 [新增應用程式]。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **SAP Business Object Cloud**。

    ![搜尋方塊](./media/sapboc-tutorial/tutorial_sapboc_search.png)

5. 在結果面板中，選取 [SAP Business Object Cloud]，然後選取 [新增]。

    ![結果清單中的 SAP Business Object Cloud](./media/sapboc-tutorial/tutorial_sapboc_addfromgallery.png)

##  <a name="set-up-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您將以名為 Britta Simon 的測試使用者身分，設定及測試與 SAP Business Object Cloud 搭配運作的 Azure AD 單一登入。

為使單一登入運作，Azure AD 必須知道 SAP Business Object Cloud 中互相對應的 Azure AD 使用者。 必須在 Azure AD 使用者與 SAP Business Object Cloud 的相關使用者之間建立連結關聯性。

若要建立連結關聯性，在 SAP Business Object Cloud 中，將 Azure AD 中 [使用者名稱] 的值指派為 [使用者名稱] 的值。

若要設定及測試與 SAP Business Object Cloud 搭配運作的 Azure AD 單一登入，請完成下列工作：

1. [設定 Azure AD 單一登入](#set-up-azure-ad-single-sign-on)。 設定要使用這項功能的使用者。
2. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)。 使用 Britta Simon 使用者來測試 Azure AD 單一登入。
3. [建立 SAP Business Object Cloud 測試使用者](#create-an-sap-business-object-cloud-test-user)。 在 SAP Business Object Cloud 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
4. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)。 將 Britta Simon 設定為使用 Azure AD 單一登入。
5. [測試單一登入](#test-single-sign-on)。 驗證組態是否能運作。

### <a name="set-up-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站開啟 Azure AD 單一登入。 然後，您要在 SAP Business Object Cloud 應用程式中設定單一登入。

若要使用 SAP Business Object Cloud 設定 Azure AD 單一登入：

1. 在 Azure 入口網站的 [SAP Business Object Cloud] 應用程式整合頁面上，選取 [單一登入]。

    ![選取 [單一登入]][4]

2. 在 [單一登入] 頁面上，選取 [SAML 登入] 作為 [模式]。
 
    ![選取 SAML 登入](./media/sapboc-tutorial/tutorial_sapboc_samlbase.png)

3. 在 [SAP Business Object Cloud 網域及 URL] 下，完成下列步驟：

    1. 在 [登入 URL] 方塊中，輸入具有下列模式的 URL： 
    | |
    |-|-|
    | `https://<sub-domain>.sapanalytics.cloud/` |
    | `https://<sub-domain>.sapbusinessobjects.cloud/` |

    2. 在 [識別碼] 方塊中，輸入具有下列模式的 URL：
    | |
    |-|-|
    | `<sub-domain>.sapbusinessobjects.cloud` |
    | `<sub-domain>.sapanalytics.cloud` |

    ![SAP Business Object Cloud 網域和 URL 頁面的 URL](./media/sapboc-tutorial/tutorial_sapboc_url.png)
 
    > [!NOTE] 
    > 這些 URL 中的值僅供示範。 使用實際的「登入 URL」及「識別碼 URL」來更新這些值。 若要取得登入 URL，請連絡 [SAP Business Object Cloud 用戶端支援小組](https://help.sap.com/viewer/product/SAP_BusinessObjects_Cloud/release/en-US)。 您可以從系統管理員主控台下載 SAP Business Object Cloud 中繼資料，以取得識別項 URL。 稍後在本教學課程中會加以說明。 

4. 在 [SAML 簽章憑證] 下，選取 [中繼資料 XML]。 然後，將中繼資料檔案儲存在您的電腦上。

    ![選取 [中繼資料 XML]](./media/sapboc-tutorial/tutorial_sapboc_certificate.png) 

5. 選取 [ **儲存**]。

    ![選取 [儲存]](./media/sapboc-tutorial/tutorial_general_400.png)

6. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 SAP Business Object Cloud 公司網站。

7. 選取 [功能表] > [系統] > [管理]。
    
    ![依序選取 [功能表]、[系統] 和 [管理]](./media/sapboc-tutorial/config1.png)

8. 在 [安全性] 索引標籤上，選取 [編輯] \(畫筆) 圖示。
    
    ![在 [安全性] 索引標籤上，選取 [編輯] 圖示](./media/sapboc-tutorial/config2.png)  

9. 選取 [SAML 單一登入 (SSO)] 作為 [驗證方法]。

    ![選取 [SAML 單一登入] 作為驗證方法](./media/sapboc-tutorial/config3.png)  

10. 選取 [下載]，以下載服務提供者中繼資料 (步驟 1)。 在中繼資料檔案中，尋找並複製 **entityID** 值。 在 Azure 入口網站中，在 [SAP Business Object Cloud 網域和 URL] 下，將值貼入 [識別碼] 方塊。

    ![複製並貼上 entityID 值](./media/sapboc-tutorial/config4.png)  

11. 若要在您從 Azure 入口網站下載的檔案中上傳服務提供者中繼資料 (步驟 2)**，請在 [上傳識別提供者中繼資料]** 下，選取 [上傳]。  

    ![在上傳識別提供者中繼資料下，選取 [上傳]](./media/sapboc-tutorial/config5.png)

12. 在 [使用者屬性值] 清單中，選取您需要用於實作的使用者屬性 (步驟 3)。 此使用者屬性會對應到識別提供者。 若要在使用者的頁面上輸入自訂屬性，請使用 [自訂 SAML 對應] 選項。 或者，您可以選取 [電子郵件] 或 [使用者識別碼]作為使用者屬性。 在本範例中，我們選取 [電子郵件]，因為我們在 Azure 入口網站的 [使用者屬性] 區段中，使用 **userprincipalname** 屬性來對應使用者識別項宣告。 這會提供唯一的使用者電子郵件，並傳送給每個成功 SAML 回應中的 SAP Business Object Cloud 應用程式。

    ![選取使用者屬性](./media/sapboc-tutorial/config6.png)

13. 若要使用識別提供者 (步驟 4) 來驗證帳戶，請在 [登入認證 (電子郵件)] 方塊中，輸入使用者的電子郵件地址。 然後，選取 [驗證帳戶]。 系統會將登入認證新增至使用者帳戶。

    ![輸入電子郵件，並選取 [驗證帳戶]](./media/sapboc-tutorial/config7.png)

14. 選取 [儲存] 圖示。

    ![[儲存] 圖示](./media/sapboc-tutorial/save.png)

> [!TIP]
> 當您設定應用程式時，在 [Azure 入口網站](https://portal.azure.com)中即可閱讀這些指示的簡要版本！ 選取 [Active Directory] > [企業應用程式] 新增此應用程式之後，選取 [單一登入] 索引標籤。您可以透過頁面底部的 [設定] 區段來存取內嵌的文件。 如需詳細資訊，請參閱 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
在本節中，您會在 Azure 入口網站中建立名為 Britta Simon 的測試使用者。

若要在 Azure AD 中建立測試使用者：

1. 在 Azure 入口網站的左側功能表中，選取 [Azure Active Directory]。

    ![建立 Azure AD 測試使用者](./media/sapboc-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請選取 [使用者和群組]，然後選取 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/sapboc-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，請選取 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/sapboc-tutorial/create_aaduser_03.png) 

4. 在 [使用者] 對話方塊中，完成下列步驟：
 
    1. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    2. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    3. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    4. 選取 [建立] 。

        ![[使用者] 對話方塊](./media/sapboc-tutorial/create_aaduser_04.png) 

    ![建立 Azure AD 使用者][100]

### <a name="create-an-sap-business-object-cloud-test-user"></a>建立 SAP Business Object Cloud 測試使用者

必須在 SAP Business Object Cloud 中佈建 Azure AD 使用者之後，他們才能登入 SAP Business Object Cloud。 在 SAP Business Object Cloud 中，必須以手動方式進行佈建。

若要佈建使用者帳戶：

1. 以系統管理員身分登入您的 SAP Business Object Cloud 公司網站。

2. 選取 [功能表] > [安全性] > [使用者]。

    ![新增員工](./media/sapboc-tutorial/user1.png)

3. 若要在 [使用者] 頁面上新增新的使用者詳細資料，請選取 [+]。 

    ![新增使用者頁面](./media/sapboc-tutorial/user4.png)

    然後完成下列步驟：

    1. 在 [使用者識別碼] 方塊中，輸入使用者的使用者識別碼，例如 **Britta**。

    2. 在 [名字] 方塊中，輸入使用者的名字，例如 **Britta**。

    3. 在 [姓氏] 方塊中，輸入使用者的姓氏，例如 **Simon**。

    4. 在 [顯示名稱] 方塊中，輸入使用者的全名，例如 **Britta Simon**。

    5. 在 [電子郵件] 方塊中，輸入使用者的電子郵件地址，例如 **brittasimon@contoso.com**。

    6. 在 [選取角色] 頁面上，選取適當的使用者角色，然後選取 [確定]。

      ![選取角色](./media/sapboc-tutorial/user3.png)

    7. 選取 [儲存] 圖示。    


### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SAP Business Object Cloud 的使用者帳戶存取權授與使用者 Britta Simon，讓她能夠使用 Azure AD 單一登入。

若要將 Britta Simon 指派到 SAP Business Object Cloud：

1. 在 Azure 入口網站中，開啟應用程式檢視，然後移至目錄檢視。 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [SAP Business Object Cloud]。

    ![設定單一登入](./media/sapboc-tutorial/tutorial_sapboc_app.png) 

3. 在左側功能表中，選取 [使用者和群組]。

    ![選取 [使用者和群組]][202] 

4. 選取 [新增] 。 然後在 [新增指派] 頁面上，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 頁面上，選取使用者清單中的 [Britta Simon]。

6. 在 [使用者和群組] 頁面上，選取 [選取]。

7. 在 [新增指派] 頁面上，選取 [指派]。

![指派使用者角色][200] 
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

選取存取面板中的 [SAP Business Object Cloud] 圖格時，應會自動登入您的 SAP Business Object Cloud 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)


<!--Image references-->

[1]: ./media/sapboc-tutorial/tutorial_general_01.png
[2]: ./media/sapboc-tutorial/tutorial_general_02.png
[3]: ./media/sapboc-tutorial/tutorial_general_03.png
[4]: ./media/sapboc-tutorial/tutorial_general_04.png

[100]: ./media/sapboc-tutorial/tutorial_general_100.png

[200]: ./media/sapboc-tutorial/tutorial_general_200.png
[201]: ./media/sapboc-tutorial/tutorial_general_201.png
[202]: ./media/sapboc-tutorial/tutorial_general_202.png
[203]: ./media/sapboc-tutorial/tutorial_general_203.png

