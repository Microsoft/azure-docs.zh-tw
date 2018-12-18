---
title: 教學課程：Azure Active Directory 與 HR2day by Merces 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 HR2day by Merces 之間的單一登入
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 853d08c9-27b1-48d4-b8e7-3705140eb67f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/24/2017
ms.author: jeedes
ms.openlocfilehash: 984e2e9999a2aba7a595034f1fec8bafb976f310
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39441606"
---
# <a name="tutorial-azure-active-directory-integration-with-hr2day-by-merces"></a>教學課程：Azure Active Directory 與 HR2day by Merces 整合

在本教學課程中，您將了解如何整合 HR2day by Merces 與 Azure Active Directory (Azure AD)。

HR2day by Merces 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 HR2day by Merces 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 HR2day by Merces。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 HR2day by Merces 整合，您需要下列項目：

- Azure AD 訂用帳戶。
- 已啟用 HR2day by Merces 單一登入的訂用帳戶。

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD，可取得 [Azure AD 一個月免費試用版](https://azure.microsoft.com/pricing/free-trial/)。  

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本大綱中說明的案例由二個主要建構元素組成：

1. 從資源庫新增 HR2day by Merces。
1. 設定並測試 Azure AD 單一登入。

## <a name="add-hr2day-by-merces-from-the-gallery"></a>從資源庫新增 HR2day by Merces
若要設定將 HR2day by Merces 整合到 Azure AD 中，請從資源庫將 HR2day by Merces 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 HR2day by Merces，請採取下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com) 的左方瀏覽窗格中，選取 **[Azure Active Directory]** 圖示。 

    ![Active Directory][1]

1. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **HR2day by Merces**。

    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/tutorial_hr2daybymerces_search.png)

1. 在結果窗格中，選取 [HR2day by Merces]，然後選取 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/tutorial_hr2daybymerces_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 HR2day by Merces 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 HR2day by Merces 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 HR2day by Merce 中的相關使用者之間建立連結關聯性。

在 HR2day by Merces 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要搭配 HR2day by Merces 來設定及測試 Azure AD 單一登入，您需要完成下列構成要素：

1. [設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)：讓您的使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)：使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 HR2day by Merces 測試使用者](#creating-an-hr2day-by-merces-test-user)：在 HR2day by Merces 中建立對應的 Britta Simon，以連結到該使用者在 Azure AD 中的代表項目。
1. [指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)：讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#testing-single-sign-on)：驗證設定是否能夠運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 HR2day by Merces 應用程式中設定單一登入。

**若要使用 HR2day by Merces 來設定 Azure AD 單一登入，請採取下列步驟：**

1. 在 Azure 入口網站的 [HR2day by Merces] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入][4]

1. 若要啟用單一登入，請在 [單一登入] 對話方塊中，於 [模式] 選取 [SAML 登入]。
 
    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2daybymerces_samlbase.png)

1. 在 [HR2day by Merces 網域與 URL] 區段中，採取下列步驟：

    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2daybymerces_url.png)

    a. 在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://<tenantname>.force.com/<instancename>`。

    b. 在 [識別碼] 方塊中，以下列模式輸入 URL：`https://hr2day.force.com/<companyname>`。

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [HR2day by Merces 用戶端支援小組](mailto:servicedesk@merces.nl)以取得這些值。 
 


1. 在 [SAML 簽署憑證] 區段上，選取 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2daybymerces_certificate.png) 

1. 本節說明如何讓使用者透過他們的 Azure AD 帳戶，在 HR2day by Merces 中進行驗證。 使用者藉由以 SAML 通訊協定為基礎的同盟執行此動作。

    HR2day by Merces 應用程式需要採用特定格式的 SAML 判斷提示，因此您必須將自訂屬性對應新增到 SAML 權杖。 以下螢幕擷取畫面顯示上述的範例。 

    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2day_00.png)
    
    > [!NOTE] 
    您必須先連絡 [HR2day by Merces 用戶端支援小組](mailto:servicedesk@merces.nl)，向其要求取得您租用戶的唯一識別碼屬性值，才能設定 SAML 判斷提示。 您需要這個值來完成下一節中的步驟： 

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，設定 SAML 權杖屬性，如下圖所示。 然後採取下列步驟。
    
      | 屬性名稱    |   屬性值 |  
    | ------------------- | -------------------- |    
    | ATTR_LOGINCLAIM | `join([mail],"102938475Z","@"` |
    
      a. 若要開啟 [新增屬性] 對話方塊，請選取 [新增屬性]。

    ![設定單一登入](./media/hr2day-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/hr2day-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 方塊中，輸入 **ATTR_LOGINCLAIM**。

    c. 從 [值] 清單中，選取 [Join()]。

    d. 從 [String1] 清單中，選取 [user.mail]。

    e. 對於 **String2**，輸入 HR2day 小組所提供的唯一識別碼。

    f. 在 [分隔符號] 方塊中，輸入 **\@**。
    
    g. 選取 [確定]。

1. 選取 [儲存] 按鈕。

    ![設定單一登入](./media/hr2day-tutorial/tutorial_general_400.png)

1. 在 [HR2day by Merces 設定] 區段中，選取 [設定 HR2day by Merces] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL]、[SAML 實體識別碼]，以及 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2daybymerces_configure.png) 

1. 若要為您的應用程式設定 SSO，請連絡 [HR2day by Merces 用戶端支援小組](mailTo:servicedesk@merces.nl)。 將下載的**憑證 (Base64)** 檔案附加至您的電子郵件。 同時提供 [登出 URL]、[SAML 實體識別碼]，以及 [SAML 單一登入服務 URL]，讓這些項目能針對 SSO 整合來進行設定。

    > [!NOTE]
    >請向 Merces 小組表明這項整合需要以下列模式設定「實體識別碼」：**https://hr2day.force.com/INSTANCENAME**。

    > [!TIP]
    >現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤。然後透過底部的 [設定] 區段來存取內嵌的文件。 您可以在 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)中閱讀更多有關內嵌文件功能的資訊。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站** 的左方瀏覽窗格中，選取 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，請在對話方塊的頂端選取 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/create_aaduser_03.png) 

1. 在 [使用者] 對話方塊中，採取下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/hr2day-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼]，並記下密碼。

    d. 選取 [建立] 。
 
### <a name="create-an-hr2day-by-merces-test-user"></a>建立 HR2day by Merces 測試使用者

本節的目標是要在 HR2day by Merces 中建立名為 Britta Simon 的使用者。 若要新增 HR2day 帳戶中的使用者，請與 [HR2day by Merces 用戶端支援小組](mailto:servicedesk@merces.nl)合作。 

> [!NOTE]
> 如果您需要手動建立使用者，請連絡 [HR2day by Merces 用戶端支援小組](mailto:servicedesk@merces.nl)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 HR2day by Merces 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 HR2day by Merces，請採取下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，移至目錄檢視，然後移至 [企業應用程式]。 接著，選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [HR2day by Merces] 。

    ![設定單一登入](./media/hr2day-tutorial/tutorial_hr2daybymerces_app.png) 

1. 在左側功能表中，選取 [使用者和群組]。

    ![指派使用者][202] 

1. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中，選取 [Britta Simon]。

1. 按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD 單一登入設定。  

當您在「存取面板」中選取 [HR2day by Merces] 圖格時，會自動登入您的 HR2day by Merces 應用程式。

## <a name="additional-resources"></a>其他資源

* [有關如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/hr2day-tutorial/tutorial_general_01.png
[2]: ./media/hr2day-tutorial/tutorial_general_02.png
[3]: ./media/hr2day-tutorial/tutorial_general_03.png
[4]: ./media/hr2day-tutorial/tutorial_general_04.png

[100]: ./media/hr2day-tutorial/tutorial_general_100.png

[200]: ./media/hr2day-tutorial/tutorial_general_200.png
[201]: ./media/hr2day-tutorial/tutorial_general_201.png
[202]: ./media/hr2day-tutorial/tutorial_general_202.png
[203]: ./media/hr2day-tutorial/tutorial_general_203.png

