---
title: 教學課程：Azure Active Directory 與 SAP Cloud for Customer 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAP Cloud for Customer 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 90154dab-eba2-4563-bcf0-f2acc797ea97
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/14/2017
ms.author: jeedes
ms.openlocfilehash: 59bbcbf9aaef17394151e7db1471b63b87a46288
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39445712"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-cloud-for-customer"></a>教學課程：Azure Active Directory 與 SAP Cloud for Customer 整合

在本教學課程中，您會了解如何整合 SAP Cloud for Customer 與 Azure Active Directory (Azure AD)。

SAP Cloud for Customer 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 SAP Cloud for Customer 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 SAP Cloud for Customer (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SAP Cloud for Customer 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 SAP Cloud for Customer 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在這裡取得一個月試用：[試用供應項目](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 SAP Cloud for Customer
1. 設定並測試 Azure AD 單一登入

## <a name="adding-sap-cloud-for-customer-from-the-gallery"></a>從資源庫新增 SAP Cloud for Customer
若要設定 SAP Cloud for Customer 與 Azure AD 整合，您需要從資源庫將 SAP Cloud for Customer 加入到受控 SaaS 應用程式清單中。

**若要從資源庫加入 SAP Cloud for Customer，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中輸入 **SAP Cloud for Customer**。

    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_search.png)

1. 在結果面板中，選取 [SAP Cloud for Customer]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 SAP Cloud for Customer 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 SAP Cloud for Customer 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 SAP Cloud for Customer 中的相關使用者之間，建立連結關聯性。

在 SAP Cloud for Customer 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 SAP Cloud for Customer 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 SAP Cloud for Customer 測試使用者](#creating-a-sap-cloud-for-customer-test-user)** - 使 SAP Cloud for Customer 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 SAP Cloud for Customer 應用程式中設定單一登入。

**若要使用 SAP Cloud for Customer 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SAP Cloud for Customer] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_samlbase.png)

1. 在 [SAP Cloud for Customer 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<server name>.crm.ondemand.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<server name>.crm.ondemand.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [SAP Cloud for Customer 支援小組](https://www.sap.com/about/agreements.sap-cloud-services-customers.html)以取得這些值。 

1. 在 [使用者屬性] 區段中，執行下列步驟：

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_attribute.png)

    a. 在 [使用者識別碼] 清單中，選取 **ExtractMailPrefix()** 函式。

    b. 從 [郵件] 清單中，選取您想要用於實作的使用者屬性。
    例如，如果您想要使用 EmployeeID 為唯一的使用者識別碼，而且已在 ExtensionAttribute2 中儲存屬性值，則選取 [user.extensionattribute2]。  

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_general_400.png)

1. 在 [SAP Cloud for Customer 組態] 區段上，按一下 [設定 SAP Cloud for Customer] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_configure.png) 

1. 為了設定 SSO，請執行下列步驟：
   
    a. 使用系統管理員權限登入 SAP Cloud for Customer 入口網站。
   
    b. 瀏覽至 [應用程式和使用者管理常見工作]，然後按一下 [識別提供者] 索引標籤。
   
    c. 按一下 [新增識別提供者]，然後選取您從 Azure 入口網站下載的中繼資料 XML 檔案。 藉由匯入中繼資料，系統會自動上傳所需的簽章憑證和加密憑證。
   
    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_54.png)
   
    d. Azure Active Directory 需要 SAML 要求中的「判斷提示取用者服務 URL」元素，因此請選取 [包括判斷提示取用者服務 URL] 核取方塊。
   
    e. 按一下 [啟用單一登入]。
   
    f. 儲存您的變更。
   
    g. 按一下 [我的系統] 索引標籤。
   
    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_52.png)
   
    h. 在 [Azure AD 登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 單一登入服務 URL**。
   
    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_53.png)
   
    i. 透過選取 [手動選取識別提供者]，指定員工是否可以在以使用者識別碼和密碼登入或 SSO 之間進行手動選擇。
   
    j. 在 [SSO URL] 區段中，指定您的員工應用來登入系統的 URL。 
    在 [傳送給員工的 URL] 清單中，您可以在下列選項之間做選擇︰
   
    **非 SSO URL**
   
    系統只會傳送一般系統 URL 給員工。 員工無法使用 SSO 來登入，而是必須改為使用密碼或憑證。
   
    **SSO URL** 
   
    系統只會傳送 SSO URL 給員工。 員工可以使用 SSO 來登入。 驗證要求會透過 IdP 重新導向。
   
    **自動選取**
   
    如果未啟用 SSO，系統會傳送一般系統 URL 給員工。 如果已啟用 SSO，系統會檢查員工是否有密碼。 如果有密碼，便會將 SSO URL 和非 SSO URL 傳送給員工。 但如果員工沒有密碼，則只會傳送 SSO URL 給員工。
   
    k. 儲存您的變更。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/sap-customer-cloud-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-sap-cloud-for-customer-test-user"></a>建立 SAP Cloud for Customer 測試使用者

在本節中，您要在 SAP Cloud for Customer 中建立名為 Britta Simon 的使用者。 請與 [SAP Cloud for Customer 支援小組](https://www.sap.com/about/agreements.sap-cloud-services-customers.html)合作，在 SAP Cloud for Customer 平台中新增使用者。 

> [!NOTE]
> 請確定 NameID 值符合 SAP Cloud for Customer 平台中的使用者名稱欄位。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SAP Cloud for Customer 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派到 SAP Cloud for Customer，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [SAP Cloud for Customer] 。

    ![設定單一登入](./media/sap-customer-cloud-tutorial/tutorial_sapcloudforcustomer_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

在存取面板中按一下 [SAP Cloud for Customer] 圖格，應該會自動登入您的 SAP Cloud for Customer 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/sap-customer-cloud-tutorial/tutorial_general_01.png
[2]: ./media/sap-customer-cloud-tutorial/tutorial_general_02.png
[3]: ./media/sap-customer-cloud-tutorial/tutorial_general_03.png
[4]: ./media/sap-customer-cloud-tutorial/tutorial_general_04.png

[100]: ./media/sap-customer-cloud-tutorial/tutorial_general_100.png

[200]: ./media/sap-customer-cloud-tutorial/tutorial_general_200.png
[201]: ./media/sap-customer-cloud-tutorial/tutorial_general_201.png
[202]: ./media/sap-customer-cloud-tutorial/tutorial_general_202.png
[203]: ./media/sap-customer-cloud-tutorial/tutorial_general_203.png

