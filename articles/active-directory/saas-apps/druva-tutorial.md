---
title: 教學課程：Azure Active Directory 與 Druva 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Druva 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: ab92b600-1fea-4905-b1c7-ef8e4d8c495c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/23/2017
ms.author: jeedes
ms.openlocfilehash: 985304244acdfafa4fa99dbbe876f35b3e6c58b2
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39439356"
---
# <a name="tutorial-azure-active-directory-integration-with-druva"></a>教學課程：Azure Active Directory 與 Druva 整合

在本教學課程中，您會了解如何將 Druva 與 Azure Active Directory (Azure AD) 進行整合。

Druva 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Druva 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Druva (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Druva 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Druva 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Druva
1. 設定並測試 Azure AD 單一登入

## <a name="adding-druva-from-the-gallery"></a>從資源庫新增 Druva
若要設定將 Druva 整合到 Azure AD 中，您需要從資源庫將 Druva 新增到受控 SaaS 應用程式清單。

**若要從資源庫加入 Druva，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Druva**，從結果面板中選取 [Druva]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Druva](./media/druva-tutorial/tutorial_druva_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Druva 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Druva 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Druva 中相關使用者之間的連結關聯性。

在 Druva 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Druva 搭配運作的 Azure AD 單一登入，您需要完成下列建構元素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Druva 測試使用者](#create-a-druva-test-user)** - 使 Druva 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會於 Azure 入口網站啟用 Azure AD 單一登入，然後在您的 Druva 應用程式中設定單一登入。

**若要使用 Druva 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Druva] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/druva-tutorial/tutorial_druva_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Druva 網域及 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/druva-tutorial/tutorial_druva_url.png)

    在 [識別碼] 文字方塊中，輸入字串值：`druva-cloud`
    
1. 按一下 [顯示進階 URL 設定]。 如果您想要以 **SP** 起始模式設定應用程式：

    ![設定單一登入](./media/druva-tutorial/tutorial_druva_url1.png)
    
    在 [登入 URL] 文字方塊中，輸入 URL：`https://cloud.druva.com/home`

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/druva-tutorial/tutorial_druva_certificate.png) 

1. Druva 應用程式需要特定格式的 SAML 判斷提示，您需要將自訂屬性對應加入您的 **SAML 權杖屬性**組態。 

    ![設定單一登入](./media/druva-tutorial/tutorial_druva_attribute.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱      | 屬性值      |
    | ------------------- | -------------------- |
    | insync\_auth\_token |輸入權杖產生的值 |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。
    
    ![設定單一登入](./media/druva-tutorial/tutorial_attribute_04.png)
    
    ![設定單一登入](./media/druva-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。 教學課程稍後將說明權杖產生的值。
    
    d. 按一下 [確定] 。    

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/druva-tutorial/tutorial_general_400.png)

1. 在 [Druva 組態] 區段上，按一下 [設定 Druva] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/druva-tutorial/tutorial_druva_configure.png) 

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Druva 公司網站。

1. 移至 [管理] \> [設定]。

    ![設定](./media/druva-tutorial/ic795091.png "設定")

1. 在 [單一登入設定] 對話方塊中，執行下列步驟：

    ![單一登入設定](./media/druva-tutorial/ic795092.png "單一登入設定")
    
    a. 在 [識別碼提供者登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值。
        
    b. 在 [識別碼提供者登出 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值
        
    c. 在記事本中開啟您的 base-64 編碼的憑證，將其內容複製到您的剪貼簿，然後貼到 [識別提供者憑證]  文字方塊中。
     
    d. 按一下 [儲存] 來開啟 [設定] 頁面。

1. 在 [設定] 頁面上，按一下 [產生 SSO Token]。

    ![設定](./media/druva-tutorial/ic795093.png "設定")

1. 在 [單一登入驗證 Token]  對話方塊中，執行下列步驟：

    ![SSO 權杖](./media/druva-tutorial/ic795094.png "SSO 權杖")
    
    a. 按一下 [複製]，在 Azure 入口網站之 [新增屬性] 區段的 [值] 文字方塊中貼上複製的值。
    
    b. 按一下 [關閉] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/druva-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/druva-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/druva-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/druva-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-druva-test-user"></a>建立 Druva 測試使用者

若要讓 Azure AD 使用者可以登入 Druva，則必須將他們佈建到 Druva。 Druva 需以手動的方式佈建。

**若要設定使用者佈建，請執行下列步驟：**

1. 以系統管理員身分登入您的 **Druva** 公司網站。

1. 移至 [管理] \> [使用者]。
   
   ![管理使用者](./media/druva-tutorial/ic795097.png "管理使用者")

1. 按一下 [新建] 。
   
   ![管理使用者](./media/druva-tutorial/ic795098.png "管理使用者")

1. 在 [建立新的使用者] 對話方塊中，執行下列步驟：
   
   ![建立新的使用者](./media/druva-tutorial/ic795099.png "建立新的使用者")
   
   a. 在 [電子郵件地址] 文字方塊中，輸入使用者的電子郵件，例如 **brittasimon@contoso.com**。
   
   b. 在 [名稱] 文字方塊中，輸入使用者的名稱，例如 **BrittaSimon**。
   
   c. 按一下 [建立使用者] 。

>[!NOTE]
>您可以使用任何其他的 Druva 使用者帳戶建立工具，或是使用 Druva 提供的 API 來佈建 Azure AD 使用者帳戶。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Druva 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Druva，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Druva]。

    ![應用程式清單中的 Druva 連結](./media/druva-tutorial/tutorial_druva_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在「存取面板」中按一下 Druva 圖格時，應該會自動登入您的 Druva 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/druva-tutorial/tutorial_general_01.png
[2]: ./media/druva-tutorial/tutorial_general_02.png
[3]: ./media/druva-tutorial/tutorial_general_03.png
[4]: ./media/druva-tutorial/tutorial_general_04.png

[100]: ./media/druva-tutorial/tutorial_general_100.png

[200]: ./media/druva-tutorial/tutorial_general_200.png
[201]: ./media/druva-tutorial/tutorial_general_201.png
[202]: ./media/druva-tutorial/tutorial_general_202.png
[203]: ./media/druva-tutorial/tutorial_general_203.png

