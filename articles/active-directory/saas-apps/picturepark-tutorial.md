---
title: 教學課程：Azure Active Directory 與 Picturepark 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Picturepark 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 31c21cd4-9c00-4cad-9538-a13996dc872f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/06/2017
ms.author: jeedes
ms.openlocfilehash: 2414240d3ab4b5cedce734579f0d39a3df59c3cf
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39422190"
---
# <a name="tutorial-azure-active-directory-integration-with-picturepark"></a>教學課程：Azure Active Directory 與 Picturepark 整合

在本教學課程中，您會了解如何整合 Picturepark 與 Azure Active Directory (Azure AD)。

Picturepark 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Picturepark 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Picturepark (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Picturepark 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Picturepark 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Picturepark
1. 設定並測試 Azure AD 單一登入

## <a name="adding-picturepark-from-the-gallery"></a>從資源庫新增 Picturepark
若要設定將 Picturepark 整合到 Azure AD 中，您需要從資源庫將 Picturepark 新增到受控的 SaaS 應用程式清單。

**若要從資源庫新增 Picturepark，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **Picturepark**。

    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/tutorial_picturepark_search.png)

1. 在結果面板中，選取 [Picturepark]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/tutorial_picturepark_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Picturepark 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Picturepark 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Picturepark 中的相關使用者之間，建立連結關聯性。

在 Picturepark 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Picturepark 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Picturepark 測試使用者](#creating-a-picturepark-test-user)** - 使 Picturepark 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Picturepark 應用程式中設定單一登入。

**若要設定與 Picturepark 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Picturepark] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/picturepark-tutorial/tutorial_picturepark_samlbase.png)

1. 在 [Picturepark 網域及 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/picturepark-tutorial/tutorial_picturepark_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<companyname>.picturepark.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： 
    
    |  |
    |--|
    | `https://<companyname>.current-picturepark.com`|
    | `https://<companyname>.picturepark.com`|
    | `https://<companyname>.next-picturepark.com`|
    | |

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [Picturepark 客戶支援小組](https://picturepark.com/about/contact/)以取得這些值。 
 
1. 在 [SAML 簽署憑證] 區段上，複製憑證的 [指紋] 值。

    ![設定單一登入](./media/picturepark-tutorial/tutorial_picturepark_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/picturepark-tutorial/tutorial_general_400.png)

1. 在 [Picturepark 組態] 區段上，按一下 [設定 Picturepark] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/picturepark-tutorial/tutorial_picturepark_configure.png) 

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Picturepark 公司網站。

1. 在最上面的工具列中，按一下 [系統管理工具]，然後按一下 [管理主控台]。
   
    ![管理主控台](./media/picturepark-tutorial/ic795062.png "管理主控台")

1. 按一下 [驗證]，然後按一下 [識別提供者]。
   
    ![驗證](./media/picturepark-tutorial/ic795063.png "驗證")

1. 在 [識別提供者組態]  區段中，執行下列步驟：
   
    ![識別提供者組態](./media/picturepark-tutorial/ic795064.png "識別提供者組態")
   
    a. 按一下 [新增] 。
  
    b. 輸入組態的名稱。
   
    c. 選取 [設定為預設值] 。
   
    d. 在 [簽發者 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。
   
    e. 在 [信任的簽發者指紋] 文字方塊中，貼上您從 [SAML 簽署憑證] 區段複製的 [指紋] 值。 

1. 按一下 [JoinDefaultUsersGroup] 。

1. 若要在 [宣告] 文字方塊中設定 [Emailaddress] 屬性，請輸入 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress`，然後按一下 [儲存]。

      ![組態](./media/picturepark-tutorial/ic795065.png "組態")

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/picturepark-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-picturepark-test-user"></a>建立 Picturepark 測試使用者

若要讓 Azure AD 使用者能夠登入 Picturepark，必須將他們佈建到 Mindflash。 Picturepark 需以手動的方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 登入您的 **Picturepark** 租用戶。

1. 在最上面的工具列中，按一下 [系統管理工具]，然後按一下 [使用者]。
   
    ![使用者](./media/picturepark-tutorial/ic795067.png "使用者")

1. 在 [使用者總覽] 索引標籤中，按一下 [新增]。
   
    ![使用者管理](./media/picturepark-tutorial/ic795068.png "使用者管理")

1. 在 [建立使用者] 對話方塊中，針對您想要佈建的有效 Azure Active Directory 使用者，執行下列步驟：
   
    ![建立使用者](./media/picturepark-tutorial/ic795069.png "建立使用者")
   
    a. 在 [電子郵件地址] 文字方塊中，輸入使用者的**電子郵件地址**：**BrittaSimon@contoso.com**。  
   
    b. 在 [密碼] 和 [確認密碼] 文字方塊中，輸入 BrittaSimon 的**密碼**。 
   
    c. 在 [名字] 文字方塊中，輸入使用者的**名字**：**Britta**。 
   
    d. 在 [姓氏] 文字方塊中，輸入使用者的**姓氏**：**Simon**。
   
    e. 在 [公司] 文字方塊中，輸入使用者的**公司名稱**。 
   
    f. 在 [國家/地區] 文字方塊中，選取使用者的**國家/地區**。
  
    g. 在 [ZIP] 文字方塊中，輸入縣/市的**郵遞區號**。
   
    h. 在 [縣/市] 文字方塊中，輸入使用者的**縣/市名稱**。

    i. 選取 [語言] 。
   
    j. 按一下頁面底部的 [新增] 。

>[!NOTE]
>您可以使用任何其他的 Picturepark 使用者帳戶建立工具，或 Picturepark 提供的 API，以佈建 Azure AD 使用者帳戶。
> 

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Picturepark 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 Picturepark，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Picturepark]。

    ![設定單一登入](./media/picturepark-tutorial/tutorial_picturepark_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Picturepark 圖格時，應該會自動登入您的 Picturepark 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/picturepark-tutorial/tutorial_general_01.png
[2]: ./media/picturepark-tutorial/tutorial_general_02.png
[3]: ./media/picturepark-tutorial/tutorial_general_03.png
[4]: ./media/picturepark-tutorial/tutorial_general_04.png

[100]: ./media/picturepark-tutorial/tutorial_general_100.png

[200]: ./media/picturepark-tutorial/tutorial_general_200.png
[201]: ./media/picturepark-tutorial/tutorial_general_201.png
[202]: ./media/picturepark-tutorial/tutorial_general_202.png
[203]: ./media/picturepark-tutorial/tutorial_general_203.png

