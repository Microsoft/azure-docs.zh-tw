---
title: 教學課程：Azure Active Directory 與 PagerDuty 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 PagerDuty 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 0410456a-76f7-42a7-9bb5-f767de75a0e0
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2018
ms.author: jeedes
ms.openlocfilehash: 0e571880d9893c0027c200c6f49dc704fea09ead
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39443693"
---
# <a name="tutorial-azure-active-directory-integration-with-pagerduty"></a>教學課程：Azure Active Directory 與 PagerDuty 整合

在本教學課程中，您會了解如何整合 PagerDuty 與 Azure Active Directory (Azure AD)。

將 PagerDuty 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中管控可存取 PagerDuty 的人員
- 您可以讓使用者使用其 Azure AD 帳戶自動登入 PagerDuty (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 PagerDuty 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 PagerDuty 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 PagerDuty
1. 設定並測試 Azure AD 單一登入

## <a name="adding-pagerduty-from-the-gallery"></a>從資源庫新增 PagerDuty
若要設定將 PagerDuty 整合到 Azure AD 中，您需要從資源庫將 PagerDuty 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 PagerDuty，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **PagerDuty**，從結果面板中選取 [PagerDuty]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/pagerduty-tutorial/tutorial_pagerduty_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 PagerDuty 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 PagerDuty 與 Azure AD 中互相對應的使用者。 換句話說，必須要建立某位 Azure AD 使用者與 PagerDuty 中相關使用者之間的連結關聯性。

在 PagerDuty 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 PagerDuty 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 PagerDuty 測試使用者](#create-a-pagerduty-test-user)** - 使 PagerDuty 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)** - 驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 PagerDuty 應用程式中設定單一登入。

**若要使用 PagerDuty 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [PagerDuty] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/pagerduty-tutorial/tutorial_pagerduty_samlbase.png)

1. 在 [PagerDuty 網域與 URL] 區段上，執行下列步驟：

    ![PagerDuty 網域及 URL 單一登入資訊](./media/pagerduty-tutorial/tutorial_pagerduty_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<tenant-name>.pagerduty.com`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<tenant-name>.pagerduty.com`

    > [!NOTE]
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [PagerDuty 用戶端支援小組](https://www.pagerduty.com/support/)以取得這些值。

1. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/pagerduty-tutorial/tutorial_pagerduty_certificate.png)

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/pagerduty-tutorial/tutorial_general_400.png)

1. 在 [PagerDuty 設定] 區段上，按一下 [設定 PagerDuty] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL 和 SAML 單一登入服務 URL]。

    ![PagerDuty 設定](./media/pagerduty-tutorial/tutorial_pagerduty_configure.png)

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 Pagerduty 公司網站。

1. 在頂端的功能表中，按一下 [帳戶設定] 。

    ![帳戶設定](./media/pagerduty-tutorial/ic778535.png "帳戶設定")

1. 按一下 [單一登入] 。

    ![單一登入](./media/pagerduty-tutorial/ic778536.png "單一登入")

1. 在 [啟用單一登入 (SSO)] 頁面上執行下列步驟：

    ![啟用單一登入](./media/pagerduty-tutorial/ic778537.png "啟用單一登入")

    a. 在記事本中開啟從 Azure 入口網站下載的 Base-64 編碼憑證，將其內容複製到您的剪貼簿，然後貼到 [X.509 憑證] 文字方塊中
  
    b. 在 [登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 **SAML 單一登入服務 URL**。
  
    c. 在 [登出 URL] 文字方塊中，貼上您從 Azure 入口網站複製的**登出 URL**。

    d. 選取 [Allow username/password login] \(允許使用者名稱/密碼登入\)。

    e. 選取 [Require EXACT authentication context comparison] \(要求完全相符的驗證內容比較\) 核取方塊。

    f. 按一下 [儲存變更] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕](./media/pagerduty-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![[使用者和群組] 與 [所有使用者] 連結](./media/pagerduty-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![[新增] 按鈕](./media/pagerduty-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![[使用者] 對話方塊](./media/pagerduty-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-pagerduty-test-user"></a>建立 PagerDuty 測試使用者

若要讓 Azure AD 使用者能夠登入 PagerDuty，必須將他們佈建到 PagerDuty 中。  
PagerDuty 需以手動方式佈建。

>[!NOTE]
>您可以使用任何其他的 Pagerduty 使用者帳戶建立工具或 Pagerduty 提供的 API 來佈建 Azure Active Directory 使用者帳戶。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 登入您的 **Pagerduty** 租用戶。

1. 在頂端的功能表中，按一下 [使用者] 。

1. 按一下 [加入使用者] 。
   
    ![新增使用者](./media/pagerduty-tutorial/ic778539.png "新增使用者")

1.  在 [邀請小組] 對話方塊中，執行下列步驟：
   
    ![邀請團隊成員](./media/pagerduty-tutorial/ic778540.png "邀請團隊成員")

    a. 輸入使用者的**名字及姓氏**，如 **Britta Simon**。 
   
    b. 輸入使用者的**電子郵件**地址，如 **brittasimon@contoso.com**。
   
    c. 按一下 [新增]，然後按一下 [傳送邀請]。
   
    >[!NOTE]
    >所有加入的使用者將會收到建立 PagerDuty 帳戶的邀請。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 PagerDuty 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派到 PagerDuty，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [PagerDuty]。

    ![應用程式清單中的 PagerDuty 連結](./media/pagerduty-tutorial/tutorial_pagerduty_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

在存取面板中按一下 PagerDuty 圖格時，應該會自動登入 PagerDuty 應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/pagerduty-tutorial/tutorial_general_01.png
[2]: ./media/pagerduty-tutorial/tutorial_general_02.png
[3]: ./media/pagerduty-tutorial/tutorial_general_03.png
[4]: ./media/pagerduty-tutorial/tutorial_general_04.png

[100]: ./media/pagerduty-tutorial/tutorial_general_100.png

[200]: ./media/pagerduty-tutorial/tutorial_general_200.png
[201]: ./media/pagerduty-tutorial/tutorial_general_201.png
[202]: ./media/pagerduty-tutorial/tutorial_general_202.png
[203]: ./media/pagerduty-tutorial/tutorial_general_203.png
