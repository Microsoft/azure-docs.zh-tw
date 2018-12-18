---
title: 教學課程：Azure Active Directory 與 BenSelect 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 BenSelect 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: ffa17478-3ea1-4356-a289-545b5b9a4494
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/23/2017
ms.author: jeedes
ms.openlocfilehash: 9af2de667085ef736c3ae37b75e3c67981161267
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39434273"
---
# <a name="tutorial-azure-active-directory-integration-with-benselect"></a>教學課程：Azure Active Directory 與 BenSelect 整合

在本教學課程中，您將了解如何整合 BenSelect 與 Azure Active Directory (Azure AD)。

將 BenSelect 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中管控可存取 BenSelect 的人員
- 您可以讓使用者透過其 Azure AD 帳戶自動登入 BenSelect (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 BenSelect 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 BenSelect 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 BenSelect
1. 設定並測試 Azure AD 單一登入

## <a name="adding-benselect-from-the-gallery"></a>從資源庫新增 BenSelect
若要設定將 BenSelect 整合到 Azure AD 中，您需要從資源庫將 BenSelect 新增到受控 SaaS 應用程式清單。

**若要從資源庫加入 BenSelect，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **BenSelect**。

    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/tutorial_benselect_search.png)

1. 在結果面板中，選取 [BenSelect]，然後按一下 [新增] 按鈕以新增該應用程式。

    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/tutorial_benselect_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 BenSelect 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 BenSelect 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 BenSelect 中的相關使用者之間建立連結關聯性。

在 BenSelect 中，將 [Username] 的值指派為 Azure AD 中 [使用者名稱] 的值，以建立連結關聯性。

若要設定及測試與 BenSelect 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 BenSelect 測試使用者](#creating-a-benselect-test-user)** - 在 BenSelect 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的 Britta Simon 對應項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 BenSelect 應用程式中設定單一登入。

**若要設定與 BenSelect 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [BenSelect] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_samlbase.png)

1. 在 [BenSelect 網域及 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_url.png)

    在 **[回覆 URL]** 文字方塊中，以下列模式輸入 URL：`https://www.benselect.com/enroll/login.aspx?Path=<tenant name>`

    > [!NOTE] 
    > 這不是真實的值。 請使用實際的「回覆 URL」來更新此值。 請連絡 [BenSelect 支援小組](mailto:support@selerix.com)以取得此值。
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證]\(原始\)，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_certificate.png) 

1. BenSelect 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_06.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中：

    a. 在 [使用者識別碼] 下拉式清單中，選取 [ExtractMailPrefix]。

    b. 在 [郵件] 下拉式清單中，選取 [user.userprincipalname]。

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/benselect-tutorial/tutorial_general_400.png)

1. 在 [BenSelect 組態] 區段上，按一下 [設定 BenSelect] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_configure.png) 

1. 若要在 **BenSelect** 端設定單一登入，您必須將已下載的「憑證 (原始)」、「登出 URL」、「SAML 實體識別碼」及「SAML 單一登入服務 URL」傳送給 [BenSelect 支援小組](mailto:support@selerix.com)。

   >[!NOTE]
   >您必須提到這項整合需要 SHA256 演算法 (不支援 SHA1)，以便在適當的伺服器 (如 app2101 等) 上設定 SSO。 
   
> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/benselect-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-benselect-test-user"></a>建立 BenSelect 測試使用者

本節的目標是要在 BenSelect 中建立一個名為 Britta Simon 的使用者。 請與 [BenSelect 支援小組](mailto:support@selerix.com)合作，在 BenSelect 帳戶中新增使用者。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 BenSelect 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 BenSelect，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [BenSelect]。

    ![設定單一登入](./media/benselect-tutorial/tutorial_benselect_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD SSO 組態。

當您在「存取面板」中按一下 [BenSelect] 圖格時，應該會自動登入您的 BenSelect 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/benselect-tutorial/tutorial_general_01.png
[2]: ./media/benselect-tutorial/tutorial_general_02.png
[3]: ./media/benselect-tutorial/tutorial_general_03.png
[4]: ./media/benselect-tutorial/tutorial_general_04.png

[100]: ./media/benselect-tutorial/tutorial_general_100.png

[200]: ./media/benselect-tutorial/tutorial_general_200.png
[201]: ./media/benselect-tutorial/tutorial_general_201.png
[202]: ./media/benselect-tutorial/tutorial_general_202.png
[203]: ./media/benselect-tutorial/tutorial_general_203.png

