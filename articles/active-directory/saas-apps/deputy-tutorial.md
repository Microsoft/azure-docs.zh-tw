---
title: 教學課程：Azure Active Directory 與 Deputy 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Deputy 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 5665c3ac-5689-4201-80fe-fcc677d4430d
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/22/2017
ms.author: jeedes
ms.openlocfilehash: d4cf47dba0501694c5ed000c087f16d36b0dcdde
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39427141"
---
# <a name="tutorial-azure-active-directory-integration-with-deputy"></a>教學課程：Azure Active Directory 與 Deputy 整合

在本教學課程中，您將了解如何整合 Degreed 與 Azure Active Directory (Azure AD)。

Deputy 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制誰有 Deputy 的存取權
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Deputy (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

如要設定 Azure AD 與 Deputy 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Deputy 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Deputy
1. 設定並測試 Azure AD 單一登入

## <a name="adding-deputy-from-the-gallery"></a>從資源庫新增 Deputy
若要設定將 Deputy 整合到 Azure AD 中，您需要從資源庫將 Deputy 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Deputy，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Deputy**。

    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/tutorial_deputy_search.png)

1. 在結果窗格中，選取 [Deputy]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/tutorial_deputy_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Deputy 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Deputy 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 Deputy 中相關使用者之間的連結關聯性。

在 Deputy 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

如要設定及測試搭配 Deputy 的 Azure AD 單一登入，您需要完成下列構成元素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Deputy 測試使用者](#creating-a-deputy-test-user)** - 在 Deputy 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Deputy 應用程式中設定單一登入。

**若要使用 Deputy 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Deputy] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_samlbase.png)

1. 如果您想要以「IDP 起始模式」設定應用程式，請在 [Deputy 網域和 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_url1.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL：
    |  |
    | ----|
    | `https://<subdomain>.<region>.au.deputy.com` |
    | `https://<subdomain>.<region>.ent-au.deputy.com` |
    | `https://<subdomain>.<region>.na.deputy.com`|
    | `https://<subdomain>.<region>.ent-na.deputy.com`|
    | `https://<subdomain>.<region>.eu.deputy.com` |
    | `https://<subdomain>.<region>.ent-eu.deputy.com` |
    | `https://<subdomain>.<region>.as.deputy.com` |
    | `https://<subdomain>.<region>.ent-as.deputy.com` |
    | `https://<subdomain>.<region>.la.deputy.com` |
    | `https://<subdomain>.<region>.ent-la.deputy.com` |
    | `https://<subdomain>.<region>.af.deputy.com` |
    | `https://<subdomain>.<region>.ent-af.deputy.com` |
    | `https://<subdomain>.<region>.an.deputy.com` |
    | `https://<subdomain>.<region>.ent-an.deputy.com` |
    | `https://<subdomain>.<region>.deputy.com` |

    b. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：
    | |
    |----|
    | `https://<subdomain>.<region>.au.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-au.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.na.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-na.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.eu.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-eu.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.as.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-as.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.la.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-la.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.af.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-af.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.an.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.ent-an.deputy.com/exec/devapp/samlacs.` |
    | `https://<subdomain>.<region>.deputy.com/exec/devapp/samlacs.` |

1. 按一下 [顯示進階 URL 設定]。 如果您想要以 **SP** 起始模式設定應用程式：

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_url2.png)

    在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<your-subdomain>.<region>.deputy.com`
    
    >[!NOTE]
    > Deputy 區域尾碼是選擇性的，或者應該使用下列其中一個︰au | na | eu |as |la |af |an |ent-au |ent-na |ent-eu |ent-as | ent-la | ent-af | ent-an

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 請連絡 [Deputy 支援小組](https://www.deputy.com/call-centers-customer-support-scheduling-software)以取得這些值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/deputy-tutorial/tutorial_general_400.png)
    
1. 在 [Deputy 組態] 區段上，按一下 [設定 Deputy] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_configure.png) 

1. 瀏覽至下列 URL：[https://(your-subdomain).deputy.com/exec/config/system_config]( https://(your-subdomain).deputy.com/exec/config/system_config)。 移至 [安全性設定]，然後按一下 [編輯]。
   
    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_004.png)

1. 在此 [安全性設定]  頁面上，執行下列步驟。

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_005.png)
    
    a. 啟用**社交登入**。
   
    b. 在記事本中開啟您從 Azure 入口網站下載的 Base64 編碼憑證，將其內容複製到剪貼簿，然後貼到 [OpenSSL 憑證] 文字方塊。
   
    c. 在 [SAML SSO URL] 文字方塊中，輸入︰`https://<your subdomain>.deputy.com/exec/devapp/samlacs?dpLoginTo=<saml sso url>`
    
    d. 在 [SAML SSO URL] 文字方塊中，以您的子網域取代 `<your subdomain>`。
   
    e. 在 [SAML SSO URL] 文字方塊中，以您從 Azure 入口網站複製的 **SAML 單一登入服務 URL** 取代 `<saml sso url>`。
   
    f. 按一下 [儲存設定] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/deputy-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-deputy-test-user"></a>建立 Deputy 測試使用者

若要讓 Azure AD 使用者可以登入 Deputy，就必須將他們佈建到 Deputy。 Deputy 需以手動的方式佈建。

#### <a name="to-provision-a-user-account-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：
1. 以系統管理員身分登入您的 Deputy 公司網站。

1. 在導覽窗格的頂端，按一下 [人員] 。
   
   ![People](./media/deputy-tutorial/tutorial_deputy_001.png "People")

1. 按一下 [新增人員] 按鈕，然後按一下 [新增一個人]。
   
   ![新增人員](./media/deputy-tutorial/tutorial_deputy_002.png "新增人員")

1. 執行下列步驟，然後按一下 [儲存並邀請]。
   
   ![新增使用者](./media/deputy-tutorial/tutorial_deputy_003.png "新增使用者")

   a. 在 [名稱] 文字方塊中，輸入像是 **BrittaSimon** 的使用者全名。
   
   b. 在 [電子郵件] 文字方塊中，輸入您要佈建的 Azure AD 帳戶的電子郵件地址。
   
   c. 在 [公司] 文字方塊中，輸入企業名稱。
   
   d. 按一下 [儲存並邀請] 按鈕。

1. AAD 帳戶的持有者會收到一封電子郵件，並依照連結在啟用其帳戶前進行確認。 您可以使用任何其他的 Deputy 使用者帳戶建立工具或 Deputy 提供的 API 來佈建 AAD 使用者帳戶。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Deputy 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Deputy，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Deputy] 。

    ![設定單一登入](./media/deputy-tutorial/tutorial_deputy_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。

當您在存取面板中按一下 Deputy 圖格時，應該會自動登入 Deputy 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/deputy-tutorial/tutorial_general_01.png
[2]: ./media/deputy-tutorial/tutorial_general_02.png
[3]: ./media/deputy-tutorial/tutorial_general_03.png
[4]: ./media/deputy-tutorial/tutorial_general_04.png

[100]: ./media/deputy-tutorial/tutorial_general_100.png

[200]: ./media/deputy-tutorial/tutorial_general_200.png
[201]: ./media/deputy-tutorial/tutorial_general_201.png
[202]: ./media/deputy-tutorial/tutorial_general_202.png
[203]: ./media/deputy-tutorial/tutorial_general_203.png

