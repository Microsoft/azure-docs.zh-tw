---
title: 教學課程：Azure Active Directory 與 TINFOIL SECURITY 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 TINFOIL SECURITY 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: da02da92-e3b0-4c09-ad6c-180882b0f9f8
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/20/2017
ms.author: jeedes
ms.openlocfilehash: 4f25768cc7e4f5865c6cfa96ebfe3b0df97deeb6
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421367"
---
# <a name="tutorial-azure-active-directory-integration-with-tinfoil-security"></a>教學課程：Azure Active Directory 與 TINFOIL SECURITY 整合

在本教學課程中，您會了解如何將 TINFOIL SECURITY 與 Azure Active Directory (Azure AD) 進行整合。

將 TINFOIL SECURITY 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 TINFOIL SECURITY 的人員
- 您可以讓使用者使用其 Azure AD 帳戶自動登入 TINFOIL SECURITY (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 TINFOIL SECURITY 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 TINFOIL SECURITY 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 TINFOIL SECURITY
1. 設定和測試 Azure AD 單一登入

## <a name="add-tinfoil-security-from-the-gallery"></a>從資源庫新增 TINFOIL SECURITY
如要設定將 TINFOIL SECURITY 整合到 Azure AD 中，您需要從資源庫將 TINFOIL SECURITY 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 TINFOIL SECURITY，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **TINFOIL SECURITY**，從結果面板中選取 [TINFOIL SECURITY]，然後按一下 [新增] 按鈕以新增應用程式。

    ![來自資源庫的 TINFOIL SECURITY](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 TINFOIL SECURITY 設定及測試 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 TINFOIL SECURITY 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 TINFOIL SECURITY 中相關使用者之間的連結關聯性。

在 TINFOIL SECURITY 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 TINFOIL SECURITY 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 TINFOIL SECURITY 測試使用者](#create-a-tinfoil-security-test-user)** - 使 TINFOIL SECURITY 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 TINFOIL SECURITY 應用程式中設定單一登入。

**若要使用 TINFOIL SECURITY 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [TINFOIL SECURITY] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![SAML 型登入](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_samlbase.png)

1. 在 [TINFOIL SECURITY 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已預先與 Azure 整合。

    ![設定單一登入](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_url.png)


1. 在 [SAML 簽署憑證] 區段上，複製 [指紋] 值。

    ![[SAML 簽署憑證] 區段](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_certificate.png) 

1. 若要加入必要的屬性對應，請執行下列步驟：
    
    ![屬性](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_attribute1.png "屬性")
    
    | 屬性名稱    |   屬性值 |
    | ------------------- | -------------------- |
    | accountid | UXXXXXXXXXXXXX |
    
    a. 按一下 [加入使用者屬性] 。
    
    ![新增屬性](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_attribute.png "屬性")
    
    ![新增屬性](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_addatt.png "屬性")
    
    b. 在 [屬性名稱] 文字方塊中，輸入 **accountid**。
    
    c. 在 [屬性值] 文字方塊中，貼上您會在本教學課程稍後取得的帳戶識別碼值。
    
    d. 按一下 [確定] 。    

1. 按一下 [儲存]  按鈕。

    ![[儲存] 按鈕](./media/tinfoil-security-tutorial/tutorial_general_400.png)

1. 在 [TINFOIL SECURITY 組態] 區段上，按一下 [設定 TINFOIL SECURITY] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![TINFOIL SECURITY 組態](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 TINFOIL SECURITY 公司網站。

1. 在頂端工具列中，按一下 [我的帳戶] 。
   
    ![儀表板](./media/tinfoil-security-tutorial/ic798971.png "儀表板")

1. 按一下 [安全性] 。
   
    ![安全性](./media/tinfoil-security-tutorial/ic798972.png "安全性")

1. 在 [單一登入]  組態頁面上，執行下列步驟：
   
    ![單一登入](./media/tinfoil-security-tutorial/ic798973.png "單一登入")
   
    a. 選取 [啟用 SAML] 。
   
    b. 按一下 [手動設定] 。
   
    c. 在 [SAML Post URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值
   
    d. 在 [SAML 憑證指紋] 文字方塊中，貼上您從 [SAML 簽署憑證] 區段複製的 [指紋] 值。
  
    e. 複製 [您的帳戶識別碼] 值，並將該值貼到 Azure 入口網站中 [新增屬性] 區段底下的 [屬性值] 文字方塊中。
   
    f. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/tinfoil-security-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![[使用者和群組] -> [所有使用者] ](./media/tinfoil-security-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![使用者](./media/tinfoil-security-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/tinfoil-security-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-tinfoil-security-test-user"></a>建立 TINFOIL SECURITY 測試使用者

若要讓 Azure AD 使用者可以登入 TINFOIL SECURITY，則必須將他們佈建到 TINFOIL SECURITY。 TINFOIL SECURITY 需以手動的方式佈建。

**若要佈建使用者，請執行下列步驟：**

1. 如果該使用者屬於企業帳戶，您需要[連絡 TINFOIL SECURITY 支援小組](https://www.tinfoilsecurity.com/contact)以建立使用者帳戶。

1. 如果該使用者是一般的 TINFOIL SECURITY SaaS 使用者，則可以將合作者加入該使用者的任何網站。 這樣會觸發傳送邀請到指定電子郵件的程序，以建立新的 TINFOIL SECURITY 使用者帳戶。

> [!NOTE]
> 您可以使用任何其他的 TINFOIL SECURITY 使用者帳戶建立工具或 TINFOIL SECURITY 提供的 API 來佈建 Azure AD 使用者帳戶。
> 
> 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 TINFOIL SECURITY 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 TINFOIL SECURITY，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [TINFOIL SECURITY]。

    ![選取 TINFOIL SECURITY](./media/tinfoil-security-tutorial/tutorial_tinfoil-security_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [TINFOIL SECURITY] 圖格時，應該會自動登入 TINFOIL SECURITY 應用程式。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/tinfoil-security-tutorial/tutorial_general_01.png
[2]: ./media/tinfoil-security-tutorial/tutorial_general_02.png
[3]: ./media/tinfoil-security-tutorial/tutorial_general_03.png
[4]: ./media/tinfoil-security-tutorial/tutorial_general_04.png

[100]: ./media/tinfoil-security-tutorial/tutorial_general_100.png

[200]: ./media/tinfoil-security-tutorial/tutorial_general_200.png
[201]: ./media/tinfoil-security-tutorial/tutorial_general_201.png
[202]: ./media/tinfoil-security-tutorial/tutorial_general_202.png
[203]: ./media/tinfoil-security-tutorial/tutorial_general_203.png

