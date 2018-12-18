---
title: 教學課程：Azure Active Directory 與 Versal 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Versal 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 5b2e53c0-61a3-4954-ae46-8c28c6368bfd
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2017
ms.author: jeedes
ms.openlocfilehash: a6e1f73218efb11da475f3e67188863c3b99de97
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421333"
---
# <a name="tutorial-azure-active-directory-integration-with-versal"></a>教學課程：Azure Active Directory 與 Versal 整合

在本教學課程中，您會了解如何將 Versal 與 Azure Active Directory (Azure AD) 進行整合。

Versal 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Versal 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Versal (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Versal 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Versal 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Versal
1. 設定並測試 Azure AD 單一登入

## <a name="adding-versal-from-the-gallery"></a>從資源庫新增 Versal
若要設定將 Versal 整合到 Azure AD 中，您需要從資源庫將 Versal 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Versal，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Versal**，從結果面板中選取 **Versal**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Versal](./media/versal-tutorial/tutorial_versal_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Versal 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Versal 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Versal 中相關使用者之間的連結關聯性。

在 Versal 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Versal 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Versal 測試使用者](#create-a-versal-test-user)** - 讓 Versal 中對應的 Britta Simon 連結到使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Versal 應用程式中設定單一登入。

**若要使用 Versal 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Versal] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/versal-tutorial/tutorial_versal_samlbase.png)

1. 在 [Versal 網域與 URL] 區段中，執行下列步驟：

    ![Versal 網域與 URL 單一登入資訊](./media/versal-tutorial/tutorial_versal_url.png)

    a. 在 [識別碼] 文字方塊中，輸入值：`VERSAL`

    b. 在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://versal.com/sso/saml/orgs/<organization_id>`

    > [!NOTE] 
    > [回覆 URL] 不是真實的值。 請使用實際的「回覆 URL」來更新此值。 請連絡 [Versal 支援小組](https://support.versal.com/hc/)以取得此值。
    
1. 應用程式需要特定格式的 SAML 判斷提示，需要您將自訂屬性對應新增到您的 SAML 權杖屬性設定。 以下螢幕擷取畫面顯示上述的範例。 [使用者識別碼] 的預設值是 **user.userprincipalname**，但是 **Versal** 預期它是與使用者電子郵件地址對應的值。 對此您可以使用清單中的 **user.mail** 屬性，或者根據組織組態使用適當的屬性值。
    
    ![使用者識別碼下拉式功能表](./media/versal-tutorial/tutorial_versal_attribute.png)

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/versal-tutorial/tutorial_versal_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/versal-tutorial/tutorial_general_400.png)
    
1. 若要在 **Versal** 端設定單一登入，您必須將所下載的「中繼資料 XML」和「SAML 簽署憑證」傳送給 [ Versal 支援小組](https://support.versal.com/hc/)。 他們將設定 Versal 組織，讓兩端的 SAML SSO 連線都設定正確。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/versal-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/versal-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/versal-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/versal-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
  
### <a name="create-a-versal-test-user"></a>建立 Versal 測試使用者

在本節中，您要在 Versal 中建立名為 Britta Simon 的使用者。 請遵循[建立 SAML 測試使用者](https://support.versal.com/hc/en-us/articles/115011672887-Creating-a-SAML-test-user)支援指南，以在組織內建立使用者 Britta Simon。 您必須先在 Versal 中建立和啟動使用者，然後才能使用單一登入。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Versal 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Versal，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Versal]。

    ![應用程式清單中的 Versal 連結](./media/versal-tutorial/tutorial_versal_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用內嵌在網站內的 Versal 課程來測試 Azure AD 單一登入設定。
如需有關如何使用 Azure AD 單一登入的支援來內嵌 Versal 課程的指示，請參閱[內嵌組織課程](https://support.versal.com/hc/en-us/articles/203271866-Embedding-organizational-courses) **SAML 單一登入**支援指南。 

您必須建立課程、與貴組織分享並加以發行，才能測試課程內嵌。 如需詳細資訊，請參閱[建立課程](https://support.versal.com/hc/en-us/articles/203722528-Create-a-course)、[發佈課程](https://support.versal.com/hc/en-us/articles/203753398-Publishing-a-course)和[課程和學習者管理](https://support.versal.com/hc/en-us/articles/206029467-Course-and-learner-management)。  
                     

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/versal-tutorial/tutorial_general_01.png
[2]: ./media/versal-tutorial/tutorial_general_02.png
[3]: ./media/versal-tutorial/tutorial_general_03.png
[4]: ./media/versal-tutorial/tutorial_general_04.png

[100]: ./media/versal-tutorial/tutorial_general_100.png

[200]: ./media/versal-tutorial/tutorial_general_200.png
[201]: ./media/versal-tutorial/tutorial_general_201.png
[202]: ./media/versal-tutorial/tutorial_general_202.png
[203]: ./media/versal-tutorial/tutorial_general_203.png

