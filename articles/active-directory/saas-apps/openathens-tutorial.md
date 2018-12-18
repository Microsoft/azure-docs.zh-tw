---
title: 教學課程：Azure Active Directory 與 OpenAthens 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 OpenAthens 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: dd4adfc7-e238-41d5-8b25-1811f08078b6
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/05/2017
ms.author: jeedes
ms.openlocfilehash: 269b216a94b1233c5f9f9a634fda3c05e46cac90
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39435897"
---
# <a name="tutorial-azure-active-directory-integration-with-openathens"></a>教學課程：Azure Active Directory 與 OpenAthens 整合

在本教學課程中，您會了解如何將 OpenAthens 與 Azure Active Directory (Azure AD) 進行整合。

OpenAthens 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中管控可存取 OpenAthens 的人員。
- 您可以讓使用者使用其 Azure AD 帳戶自動登入 OpenAthens (單一登入)。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 OpenAthens 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 OpenAthens 單一登入的訂用帳戶

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，可以[取得一個月的免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 OpenAthens
1. 設定並測試 Azure AD 單一登入

## <a name="adding-openathens-from-the-gallery"></a>從資源庫新增 OpenAthens
若要設定將 OpenAthens 整合到 Azure AD 中，您需要從資源庫將 OpenAthens 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 OpenAthens**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]，然後移至 [所有應用程式]。

    ![企業應用程式窗格][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **OpenAthens**，從結果面板中選取 [OpenAthens]，然後選取 [新增] 按鈕。

    ![結果清單中的 OpenAthens](./media/openathens-tutorial/tutorial_openathens_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 OpenAthens 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 OpenAthens 與 Azure AD 中互相對應的使用者。 換句話說，您必須建立 Azure AD 使用者和 OpenAthens 中相關使用者之間的連結關聯性。

在 OpenAthens 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要使用 OpenAthens 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 OpenAthens 測試使用者](#create-a-openathens-test-user)，在 OpenAthens 中建立 Britta Simon 的對應項目，且該項目須與 Azure AD 中代表使用者的項目連結。
1. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#test-single-sign-on)，驗證設定是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 OpenAthens 應用程式中設定單一登入。

**若要設定 Azure AD 單一登入與 OpenAthens**

1. 在 Azure 入口網站的 [OpenAthens] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

1. 若要啟用單一登入，在 [單一登入] 對話方塊中，於 [模式] 選取 [SAML 登入]。
 
    ![單一登入對話方塊](./media/openathens-tutorial/tutorial_openathens_samlbase.png)

1. 在 [OpenAthens 網域及 URL] 區段中，於 [識別碼] 文字方塊中輸入值 `https://login.openathens.net/saml/2/metadata-sp`。

    ![OpenAthens 網域及 URL 單一登入資訊](./media/openathens-tutorial/tutorial_openathens_url.png)

1. 在 [SAML 簽署憑證] 區段上，選取 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![AMSL 簽署憑證下載連結](./media/openathens-tutorial/tutorial_openathens_certificate.png) 

1. 選取 [儲存] 按鈕。

    ![單一登入儲存按鈕](./media/openathens-tutorial/tutorial_general_400.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 OpenAthens 公司網站。

1. 在 [管理] 索引標籤下方的清單選取 [連線]。 

    ![設定單一登入](./media/openathens-tutorial/tutorial_openathens_application1.png)

1. 選取 [SAML 1.1/2.0]，然後選取 [設定] 按鈕。

    ![設定單一登入](./media/openathens-tutorial/tutorial_openathens_application2.png)
    
1. 若要新增設定，請選取 [瀏覽] 按鈕以上傳您從 Azure 入口網站下載的中繼資料 .xml 檔案，然後選取 [新增]。

    ![設定單一登入](./media/openathens-tutorial/tutorial_openathens_application3.png)

1. 在 [詳細資料] 索引標籤下方執行以下步驟。

    ![設定單一登入](./media/openathens-tutorial/tutorial_openathens_application4.png)

    a. 在 [顯示名稱對應] 中，選取 [使用屬性]。

    b. 在 [顯示名稱屬性] 文字方塊中，輸入 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` 值。
    
    c. 在 [唯一使用者對應] 中，選取 [使用屬性]。

    d. 在 [唯一使用者屬性] 文字方塊中，輸入 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` 值。

    e. 在 [狀態] 中，勾選三個核取方塊。

    f. 在 [建立本機帳戶] 中，選取 [自動]。

    g. 選取 [儲存變更]。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤，即可透過底部的 [設定] 區段存取內嵌的文件。 若要深入了解內嵌文件功能，請參閱 [Azure AD 內嵌文件](https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 "Britta Simon" 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者**

1. 在 Azure 入口網站的左側窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 按鈕](./media/openathens-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/openathens-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/openathens-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/openathens-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 文字方塊中顯示的值。

    d. 選取 [建立] 。
  
### <a name="create-an-openathens-test-user"></a>建立 OpenAthens 測試使用者

OpenAthens 支援 Just-In-Time 佈建，使用者在成功通過驗證後會自動建立。 在本節中，您不需要執行任何動作。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 OpenAthens 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 OpenAthens**

1. 在 Azure 入口網站中，開啟應用程式檢視，瀏覽至目錄檢視，並移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在 [應用程式] 清單中，選取 [OpenAthens]。

    ![應用程式清單中的 OpenAthens 連結](./media/openathens-tutorial/tutorial_openathens_app.png)  

1. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增] 按鈕。 然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 清單中，選取 [Britta Simon]。

1. 在 [使用者和群組] 清單中，選取 [選取] 按鈕。

1. 在 [新增指派] 窗格中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

在存取面板中選取 [OpenAthens] 圖格時，應該會自動登入 OpenAthens 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* 有關如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單，請參閱[搭配 Azure AD 的 SaaS 應用程式整合教學課程](tutorial-list.md)。
* 有關搭配 Azure Active Directory 的應用程式存取和單一登入的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

<!--Image references-->

[1]: ./media/openathens-tutorial/tutorial_general_01.png
[2]: ./media/openathens-tutorial/tutorial_general_02.png
[3]: ./media/openathens-tutorial/tutorial_general_03.png
[4]: ./media/openathens-tutorial/tutorial_general_04.png

[100]: ./media/openathens-tutorial/tutorial_general_100.png

[200]: ./media/openathens-tutorial/tutorial_general_200.png
[201]: ./media/openathens-tutorial/tutorial_general_201.png
[202]: ./media/openathens-tutorial/tutorial_general_202.png
[203]: ./media/openathens-tutorial/tutorial_general_203.png

