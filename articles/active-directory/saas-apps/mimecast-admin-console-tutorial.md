---
title: 教學課程：Azure Active Directory 與 Mimecast Admin Console 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Mimecast Admin Console 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 81c50614-f49b-4bbc-97d5-3cf77154305f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/08/2017
ms.author: jeedes
ms.openlocfilehash: ce4142c5b4a20886a94c87699f262f7238fc2cb4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39438564"
---
# <a name="tutorial-azure-active-directory-integration-with-mimecast-admin-console"></a>教學課程：Azure Active Directory 與 Mimecast Admin Console 整合

在本教學課程中，您會了解如何整合 Mimecast Admin Console 與 Azure Active Directory (Azure AD)。

Mimecast Admin Console 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Mimecast Admin Console 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Mimecast Admin Console (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Mimecast Admin Console 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Mimecast Admin Console 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Mimecast Admin Console
1. 設定並測試 Azure AD 單一登入

## <a name="adding-mimecast-admin-console-from-the-gallery"></a>從資源庫新增 Mimecast Admin Console
若要設定將 Mimecast Admin Console 整合到 Azure AD 中，您需要從資源庫將 Mimecast Admin Console 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Mimecast Admin Console，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Mimecast Admin Console**，從結果面板中選取 **Mimecast Admin Console**，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Mimecast Admin Console](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Mimecast Admin Console 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Mimecast Admin Console 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Mimecast Admin Console 中的相關使用者之間，建立連結關聯性。

在 Mimecast Admin Console 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Mimecast Admin Console 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Mimecast Admin Console 測試使用者](#create-a-mimecast-admin-console-test-user)** - 使 Mimecast Admin Console 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Mimecast Admin Console 應用程式中設定單一登入。

**若要設定與 Mimecast Admin Console 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Mimecast Admin Console] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_samlbase.png)

1. 在 [Mimecast Admin Console 網域及 URL] 區段中，執行下列步驟：

    ![Mimecast Admin Console 網域與 URL 單一登入資訊](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_url.png)

    在 [登入 URL] 文字方塊中，輸入 URL：
    | |
    | -- |
    | `https://webmail-uk.mimecast.com`|
    | `https://webmail-us.mimecast.com`|

    > [!NOTE] 
    > 登入 URL 是地區特定的 URL。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/mimecast-admin-console-tutorial/tutorial_general_400.png)

1. 在 [Mimecast Admin Console 設定] 區段上，按一下 [設定 Mimecast Admin Console 入口網站] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體識別碼] 和 [SAML 單一登入服務 URL]。

    ![Mimecast Admin Console 設定](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_configure.png) 

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Mimecast Admin Console 公司網站。

1. 移至 [服務 \> 應用程式]。

    ![服務](./media/mimecast-admin-console-tutorial/ic794998.png "服務")

1. 按一下 [驗證設定檔] 。

    ![驗證設定檔](./media/mimecast-admin-console-tutorial/ic794999.png "驗證設定檔")
    
1. 按一下 [新驗證設定檔] 。

    ![新驗證設定檔](./media/mimecast-admin-console-tutorial/ic795000.png "新驗證設定檔")

1. 在 [驗證設定檔]  區段中，執行下列步驟：

    ![驗證設定檔](./media/mimecast-admin-console-tutorial/ic795015.png "驗證設定檔")
    
    a. 在 [描述]  文字方塊中，輸入您的組態名稱。
    
    b. 選取 [強制執行 Mimecast Admin Console 的 SAML 驗證] 。
    
    c. 在 [提供者] 中選取 [Azure Active Directory]。
    
    d. 將您從 Azure 入口網站複製的 **SAML 實體識別碼**貼到 [簽發者 URL] 文字方塊中。
    
    e. 將您從 Azure 入口網站複製的 **SAML 單一登入服務 URL** 貼到 [登入 URL] 文字方塊中。

    f. 將您從 Azure 入口網站複製的 **SAML 單一登入服務 URL** 貼到 [登出 URL] 文字方塊中。
    
    >[!NOTE]
    >Mimecast Admin Console 的登入 URL 值與登出 URL 值是相同的。
    
    g. 在記事本中開啟您從 Azure 入口網站下載的 base-64 憑證，移除第一行 (“*--*“) 與最後一行 (“*--*“)，將它的其餘內容複製到您的剪貼簿，然後貼到 [識別提供者憑證 (中繼資料)] 文字方塊中。
    
    h. 選取 [允許單一登入] 。
    
    i. 按一下 [檔案] 。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985) 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/mimecast-admin-console-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/mimecast-admin-console-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/mimecast-admin-console-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/mimecast-admin-console-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-mimecast-admin-console-test-user"></a>建立 Mimecast Admin Console 測試使用者

若要讓 Azure AD 使用者能夠登入 Mimecast Admin Console，必須將它們佈建到 Mimecast Admin Console。 Mimecast Admin Consol 需以手動的方式佈建。

* 您需要先註冊網域才能建立使用者。

**若要設定使用者佈建，請執行下列步驟：**

1. 以系統管理員身份登入您的 **Mimecast Admin Console** 。
1. 移至 [目錄 \> 內部]。
   
   ![目錄](./media/mimecast-admin-console-tutorial/ic795003.png "目錄")
1. 按一下 [註冊新網域] 。
   
   ![註冊新網域](./media/mimecast-admin-console-tutorial/ic795004.png "註冊新網域")
1. 在您建立好新網域之後，，按一下 [新位址] 。
   
   ![新位址](./media/mimecast-admin-console-tutorial/ic795005.png "新位址")
1. 在 [新位址] 對話方塊中，執行下列步驟：
   
   ![儲存](./media/mimecast-admin-console-tutorial/ic795006.png "儲存")
   
   a. 在相關的文字方塊中，輸入您要佈建之有效 Azure AD 帳戶的 [電子郵件地址]、[全域名稱]、[密碼]、[確認密碼] 屬性。

   b. 按一下 [檔案] 。

>[!NOTE]
>您可以使用任何其他的 Mimecast Admin Console 使用者帳戶建立工具或 Mimecast Admin Console 提供的 API，佈建 Azure AD 使用者帳戶。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Mimecast Admin Console 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Mimecast Admin Console，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Mimecast Admin Console]。

    ![應用程式清單中的 Mimecast Admin Console 連結](./media/mimecast-admin-console-tutorial/tutorial_mimecastadminconsole_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 Mimecast Admin Console 圖格時，應該會自動登入您的 Mimecast Admin Console 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/mimecast-admin-console-tutorial/tutorial_general_01.png
[2]: ./media/mimecast-admin-console-tutorial/tutorial_general_02.png
[3]: ./media/mimecast-admin-console-tutorial/tutorial_general_03.png
[4]: ./media/mimecast-admin-console-tutorial/tutorial_general_04.png

[100]: ./media/mimecast-admin-console-tutorial/tutorial_general_100.png

[200]: ./media/mimecast-admin-console-tutorial/tutorial_general_200.png
[201]: ./media/mimecast-admin-console-tutorial/tutorial_general_201.png
[202]: ./media/mimecast-admin-console-tutorial/tutorial_general_202.png
[203]: ./media/mimecast-admin-console-tutorial/tutorial_general_203.png

