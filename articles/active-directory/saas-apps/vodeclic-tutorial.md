---
title: 教學課程：Azure Active Directory 與 Vodeclic 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Vodeclic 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: d77a0f53-e3a3-445e-ab3e-119cef6e2e1d
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/06/2017
ms.author: jeedes
ms.openlocfilehash: fb985b389139bfd8d54e6c54d101bbfa8a68a6d4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39444618"
---
# <a name="tutorial-azure-active-directory-integration-with-vodeclic"></a>教學課程：Azure Active Directory 與 Vodeclic 整合

在本教學課程中，您將了解如何整合 Vodeclic 與 Azure Active Directory (Azure AD)。

將 Vodeclic 與 Azure AD 整合可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Vodeclic 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Vodeclic (單一登入，或稱 SSO)。
- 您可以在 Azure 入口網站集中管理您的帳戶。

若您想了解 SaaS 應用程式與 Azure AD 整合的更多詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Vodeclic 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Vodeclic SSO 的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，[取得一個月免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Vodeclic
1. 設定並測試 Azure AD 單一登入

## <a name="add-vodeclic-from-the-gallery"></a>從資源庫新增 Vodeclic
若要設定將 Vodeclic 整合到 Azure AD 中，您需要將 Vodeclic 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Vodeclic，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Vodeclic**。 從結果面板中選取 [Vodeclic]，然後選取 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Vodeclic](./media/vodeclic-tutorial/tutorial_vodeclic_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Vodeclic 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Vodeclic 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Vodeclic 中的相關使用者之間建立連結。

在 Vodeclic 中，為 [Username] \(使用者名稱\) 值提供與 Azure AD 中**使用者名稱**相同的值。 現在您已經建立兩名使用者之間的連結。

若要設定及測試與 Vodeclic 搭配運作的 Azure AD 單一登入，請完成下列構成要素：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，以讓您的使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 Vodeclic 測試使用者](#create-a-vodeclic-test-user)，以在 Vodeclic 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
1. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#test-single-sign-on)，驗證設定是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Vodeclic 應用程式中設定單一登入。

**若要設定與 Vodeclic 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Vodeclic] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊中的 [單一登入模式] 底下，選取 [SAML 登入] 以啟用單一登入。
 
    ![單一登入對話方塊](./media/vodeclic-tutorial/tutorial_vodeclic_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Vodeclic 網域及 URL] 區段中執行下列步驟：

    ![Vodeclic 網域及 URL 單一登入資訊](./media/vodeclic-tutorial/tutorial_vodeclic_url.png)

    a. 在 [識別碼] 方塊中，以下列模式輸入 URL：`https://<companyname>.lms.vodeclic.net/auth/saml`

    b. 在 [回覆 URL] 方塊中，使用下列模式輸入 URL：`https://<companyname>.lms.vodeclic.net/auth/saml/callback`

1. 如果您想要以 **SP** 起始模式設定應用程式，請選取 [顯示進階 URL 設定] 核取方塊，然後執行下列步驟︰

    ![Vodeclic 網域及 URL 單一登入資訊](./media/vodeclic-tutorial/tutorial_vodeclic_url1.png)

    在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://<companyname>.lms.vodeclic.net/auth/saml`
     
    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的識別碼、回覆 URL 及登入 URL 來更新這些值。 請連絡 [Vodeclic 用戶端支援小組](mailto:hotline@vodeclic.com)以取得這些值。

1. 在 [SAML 簽章憑證] 區段中，選取 [中繼資料 XML]。 然後，將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/vodeclic-tutorial/tutorial_vodeclic_certificate.png) 

1. 選取 [ **儲存**]。

    ![設定單一登入儲存按鈕](./media/vodeclic-tutorial/tutorial_general_400.png)
    
1. 若要在 **Vodeclic** 端設定單一登入，請將已下載的**中繼資料 XML** 傳送給 [Vodeclic 支援小組](mailto:hotline@vodeclic.com)。 他們會進行此設定，讓兩端的 SAML SSO 連線都設定正確。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤，即可透過底部的 [設定] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/vodeclic-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]。 然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/vodeclic-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/vodeclic-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/vodeclic-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-vodeclic-test-user"></a>建立 Vodeclic 測試使用者

在本節中，您會在 Vodeclic 中建立名為 Britta Simon 的使用者。 請與 [Vodeclic 支援小組](mailto:hotline@vodeclic.com)合作，以在 Vodeclic 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

> [!NOTE]
> 根據應用程式需求，您可能需要將您的電腦列入允許清單中。 若要這麼做，您必須將公用 IP 位址分享給 [Vodeclic 支援小組](mailto:hotline@vodeclic.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Vodeclic 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Vodeclic，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，然後移至目錄檢視。 接下來，移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Vodeclic]。

    ![應用程式清單中的 Vodeclic 連結](./media/vodeclic-tutorial/tutorial_vodeclic_app.png)  

1. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，選取 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [Vodeclic] 圖格時，應該會自動登入您的 Vodeclic 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/vodeclic-tutorial/tutorial_general_01.png
[2]: ./media/vodeclic-tutorial/tutorial_general_02.png
[3]: ./media/vodeclic-tutorial/tutorial_general_03.png
[4]: ./media/vodeclic-tutorial/tutorial_general_04.png

[100]: ./media/vodeclic-tutorial/tutorial_general_100.png

[200]: ./media/vodeclic-tutorial/tutorial_general_200.png
[201]: ./media/vodeclic-tutorial/tutorial_general_201.png
[202]: ./media/vodeclic-tutorial/tutorial_general_202.png
[203]: ./media/vodeclic-tutorial/tutorial_general_203.png

