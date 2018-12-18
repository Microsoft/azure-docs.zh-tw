---
title: 教學課程：Azure Active Directory 與 SilkRoad Life Suite 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SilkRoad Life Suite 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 3cd92319-7964-41eb-8712-444f5c8b4d15
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/12/2017
ms.author: jeedes
ms.openlocfilehash: 4d8be22a6b700d5ea9d95ee19d6ad3fa7bf5910a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39440827"
---
# <a name="tutorial-azure-active-directory-integration-with-silkroad-life-suite"></a>教學課程：Azure Active Directory 與 SilkRoad Life Suite 整合

在本教學課程中，您將了解如何整合 SilkRoad Life Suite 與 Azure Active Directory (Azure AD)。

SilkRoad Life Suite 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 SilkRoad Life Suite 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 SilkRoad Life Suite (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 SilkRoad Life Suite 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 SilkRoad Life Suite 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫加入 SilkRoad Life Suite
1. 設定並測試 Azure AD 單一登入

## <a name="adding-silkroad-life-suite-from-the-gallery"></a>從資源庫加入 SilkRoad Life Suite
若要設定 SilkRoad Life Suite 與 Azure AD 整合，您需要從資源庫將 SilkRoad Life Suite 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 SilkRoad Life Suite，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **SilkRoad Life Suite**，從結果面板中選取 [SilkRoad Life Suite]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 SilkRoad Life Suite](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 SilkRoad Life Suite 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 SilkRoad Life Suite 與 Azure AD 中互相對應的使用者。 換句話說，Azure AD 使用者和 SilkRoad Life Suite 中的相關使用者之間必須建立連結關聯性。

在 SilkRoad Life Suite 中，指派 Azure AD 中的**使用者名稱**值作為 [Username] \(使用者名稱\) 的值，以建立連結關聯性。

若要設定及測試對 SilkRoad Life Suite 的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 SilkRoad Life Suite 測試使用者](#create-a-silkroad-life-suite-test-user)** - 在 SilkRoad Life Suite 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的對應項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 SilkRoad Life Suite 應用程式中設定單一登入。

**若要使用 SilkRoad Life Suite 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SilkRoad Life Suite] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_samlbase.png)

1. 在 [SilkRoad Life Suite 網域及 URL] 區段上，執行下列步驟：

    ![SilkRoad Life Suite 網域及 URL 單一登入資訊](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_url1.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<subdomain>.silkroad-eng.com/Authentication/`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： 
    | |
    |--|
    | `https://<subdomain>.silkroad-eng.com/Authentication/SP` |
    | `https://<subdomain>.silkroad.com/Authentication/SP` |

    c. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL： 
    | |
    |--|
    | `https://<subdomain>.silkroad-eng.com/Authentication/` |
    | `https://<subdomain>.silkroad.com/Authentication/` |
     
    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的識別碼、回覆 URL 和登入 URL 來更新這些值。 請連絡 [SilkRoad Life Suite 用戶端支援小組](https://www.silkroad.com/locations/) \(英文\) 以取得這些值。 

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/silkroad-life-suite-tutorial/tutorial_general_400.png)
    
1. 在 [SilkRoad Life Suite 設定] 區段上，按一下 [設定 SilkRoad Life Suite] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![SilkRoad Life Suite 設定](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_configure.png) 

1. 以系統管理員身分登入您的 SilkRoad 公司網站。 
 
    >[!NOTE] 
    > 若要取得用於設定與 Microsoft Azure AD 的同盟存取的 SilkRoad Life Suite 驗證應用程式，請連絡 SilkRoad 支援人員或您的 SilkRoad 服務代表。

1. 移至 [服務提供者]，然後按一下 [同盟詳細資料]。 
   
    ![Azure AD 單一登入][10]

1. 按一下 [下載同盟中繼資料] ，然後將資料檔儲存在您的電腦中。
   
    ![Azure AD 單一登入][11] 

1. 在您的 **SilkRoad** 應用程式中，按一下 [驗證來源]。
   
    ![Azure AD 單一登入][12] 

1. 按一下 [加入驗證來源] 。 
   
    ![Azure AD 單一登入][13] 

1. 在 [加入驗證來源]  區段中，執行下列步驟： 
   
    ![Azure AD 單一登入][14]
  
    a. 在 [Option 2 - Metadata File] \(選項 2 - 中繼資料檔案\) 底下，按一下 [Option 2 - Metadata File] \(瀏覽\) 來上傳從 Azure 入口網站下載的中繼資料檔案。
  
    b. 按一下 [使用檔案資料建立身分識別提供者]。

1. 在 [驗證來源] 區段中，按一下 [編輯]。 
    
     ![Azure AD 單一登入][15] 

1. 在 [編輯驗證來源]  對話方塊中，執行下列步驟： 
    
     ![Azure AD 單一登入][16] 

    a. 對 [已啟用] 選取 [是]。

    b. 在 [EntityId] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。
   
    c. 在 [IdP Description] \(IdP 描述\) 文字方塊中，輸入您的設定描述 (例如：*Azure AD SSO*)。

    d. 在 [Metadata File] \(中繼資料檔案\) 文字方塊中，上傳您從 Azure 入口網站下載的**中繼資料**檔案。
  
    e. 在 [IdP Name] \(IdP 名稱\) 文字方塊中，輸入設定特定的名稱 (例如：*Azure SP*)。
  
    f. 在 [Logout Service URL] \(登出服務 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    g. 在 [Sign-on Service URL] \(登入服務 URL\) 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    h. 按一下 [檔案] 。

1. 停用所有其他驗證來源。 
    
     ![Azure AD 單一登入][17]

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/silkroad-life-suite-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/silkroad-life-suite-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/silkroad-life-suite-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/silkroad-life-suite-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-silkroad-life-suite-test-user"></a>建立 SilkRoad Life Suite 測試使用者

在本節中，您會在 SilkRoad Life Suite 中建立名為 Britta Simon 的使用者。 請與 [SilkRoad Life Suite 用戶端支援小組](https://www.silkroad.com/locations/)合作，在 SilkRoad Life Suite 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SilkRoad Life Suite 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派到 SilkRoad Life Suite，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [SilkRoad Life Suite] 。

    ![應用程式清單中的 [SilkRoad Life Suite] 連結](./media/silkroad-life-suite-tutorial/tutorial_silkroadlifesuite_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在 [存取面板] 中按一下 [SilkRoad Life Suite] 磚時，您應該會自動登入您的 SilkRoad Life Suite 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/silkroad-life-suite-tutorial/tutorial_general_01.png
[2]: ./media/silkroad-life-suite-tutorial/tutorial_general_02.png
[3]: ./media/silkroad-life-suite-tutorial/tutorial_general_03.png
[4]: ./media/silkroad-life-suite-tutorial/tutorial_general_04.png

[100]: ./media/silkroad-life-suite-tutorial/tutorial_general_100.png

[200]: ./media/silkroad-life-suite-tutorial/tutorial_general_200.png
[201]: ./media/silkroad-life-suite-tutorial/tutorial_general_201.png
[202]: ./media/silkroad-life-suite-tutorial/tutorial_general_202.png
[203]: ./media/silkroad-life-suite-tutorial/tutorial_general_203.png
[10]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_06.png
[11]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_07.png
[12]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_08.png
[13]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_09.png
[14]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_10.png
[15]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_11.png
[16]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_12.png
[17]: ./media/silkroad-life-suite-tutorial/tutorial_silkroad_13.png
