---
title: 教學課程：Azure Active Directory 與 Box 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Box 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 3b565c8d-35e2-482a-b2f4-bf8fd7d8731f
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: f5aa724e9848c9794eef093aef15b0aaed9cae97
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39435755"
---
# <a name="integrate-azure-active-directory-with-box"></a>整合 Azure Active Directory 與 Box

在本教學課程中，您將了解如何整合 Azure Active Directory (Azure AD) 與 Box。

整合 Azure AD 與 Box 可提供下列優點：

- 您可以在 Azure AD 中控制可存取 Box 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Box (單一登入，又稱為 SSO)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

若要了解 SaaS 應用程式與 Azure AD 的整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Box 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Box SSO 的訂用帳戶

> [!NOTE]
> 當您測試本教學課程中的步驟時，「不」建議您使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 Box
1. 設定並測試 Azure AD 單一登入

## <a name="add-box-from-the-gallery"></a>從資源庫新增 Box
若要設定 Azure AD 與 Box 的整合，請執行下列步驟，從資源庫將 Slack 新增到受控 SaaS 應用程式清單：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory]。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式] > [所有應用程式]。

    ![企業應用程式視窗][2]
    
1. 若要新增新的應用程式，請選取視窗頂端的 [新增應用程式] 按鈕。

    ![[新增應用程式] 按鈕][3]

1. 在搜尋方塊中輸入 **Box**，從結果清單中選取 [Box]，然後選取 [新增]。

    ![結果清單中的 Box](./media/box-tutorial/tutorial_box_search.png)
### <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Box 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須識別 Box 使用者及其在 Azure AD 中對應的使用者。 換句話說，必須在 Azure AD 使用者和 Box 中的相同使用者之間建立連結關聯性。

若要建立連結關聯性，請將 Azure AD 中的*使用者名稱*值指派為 Box *使用者名稱*。

若要設定及測試與 Box 搭配運作的 Azure AD 單一登入，請完成接下來五個小節中的建置組塊。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Box 應用程式中設定單一登入，做法如下：

1. 在 Azure 入口網站的 [Box] 應用程式整合視窗中，選取 [單一登入]。

    ![單一登入連結][4]

1. 在 [單一登入] 視窗的 [單一登入模式] 方塊中，選取 [SAML 登入]。
 
    ![單一登入視窗](./media/box-tutorial/tutorial_box_samlbase.png)

1. 在 [Box 網域及 URL] 下，執行下列操作：

    ![「Box 網域及 URL」單一登入資訊](./media/box-tutorial/url3.png)

    a. 在 [登入 URL] 方塊中，以下列格式輸入 URL︰*https://\<subdomain>.box.com*。

    b. 在 [識別碼] 文字方塊中，輸入 **box.net**。
     
    > [!NOTE] 
    > 上述值並非真正的值。 請使用實際的登入 URL 及識別碼來更新這些值。 若要取得這些值，請連絡 [Box 用戶端支援小組](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire)。 

1. 在 [SAML 簽署憑證] 下，選取 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/box-tutorial/tutorial_box_certificate.png) 

1. 選取 [ **儲存**]。

    ![設定單一登入儲存按鈕](./media/box-tutorial/tutorial_general_400.png)
    
1. 若要設定應用程式的 SSO，請遵循[自行設定 SSO](https://community.box.com/t5/How-to-Guides-for-Admins/Setting-Up-Single-Sign-On-SSO-for-your-Enterprise/ta-p/1263#ssoonyourown) 中的程序。

> [!NOTE] 
> 如果您無法為 Box 帳戶啟用 SSO 設定，您可能需要連絡 [Box 用戶端支援小組](https://community.box.com/t5/custom/page/page-id/submit_sso_questionaire)，並提供下載的 XML 檔案。

> [!TIP]
> 當您設定此應用程式時，可以在 [Azure 入口網站](https://portal.azure.com)中閱讀先前那些指示的簡要版本。 在 [Active Directory]  >  [企業應用程式] 區段中新增應用程式之後，請選取 [單一登入] 索引標籤，然後在底部的 [組態] 區段中存取內嵌的文件。 如需內嵌文件功能的詳細資訊，請參閱 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。
>

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您會執行下列步驟，在 Azure 入口網站中建立測試使用者 Britta Simon：

![建立 Azure AD 測試使用者][100]

1. 在 Azure 入口網站的左側窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 連結](./media/box-tutorial/create_aaduser_01.png)

1. 若要顯示目前使用者清單，請選取 [使用者和群組]  >  [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/box-tutorial/create_aaduser_02.png)

1. 在 [所有使用者] 視窗頂端，選取 [新增]。

    ![[新增] 按鈕](./media/box-tutorial/create_aaduser_03.png)

    [使用者] 視窗隨即開啟。

1. 在 [使用者] 視窗中，執行下列操作：

    ![[使用者] 視窗](./media/box-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-box-test-user"></a>建立 Box 測試使用者

在本節中，您會在 Box 中建立測試使用者 Britta Simon。 Box 支援預設啟用的 Just-In-Time 佈建。 如果還沒有任何使用者存在，當您嘗試存取 Box 時，就會建立新的使用者。 您不需要手動建立使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Box 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。 若要這樣做，請執行下列動作：

![指派使用者角色][200]

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至 [目錄] 檢視，然後選取 [企業應用程式]  >  [所有應用程式]。

    ![[企業應用程式] 和 [所有應用程式] 連結][201] 

1. 在 [應用程式] 清單中，選取 [Box]。

    ![Box 連結](./media/box-tutorial/tutorial_box_app.png)  

1. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 視窗的 [使用者] 清單中，選取 [Britta Simon]。

1. 選取 [選取] 按鈕。

1. 在 [新增指派] 視窗中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 **Box** 圖格時，您會開啟可讓您登入 Box 應用程式的登入頁面。

## <a name="additional-resources"></a>其他資源

* [整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](box-userprovisioning-tutorial.md)



<!--Image references-->

[1]: ./media/box-tutorial/tutorial_general_01.png
[2]: ./media/box-tutorial/tutorial_general_02.png
[3]: ./media/box-tutorial/tutorial_general_03.png
[4]: ./media/box-tutorial/tutorial_general_04.png

[100]: ./media/box-tutorial/tutorial_general_100.png

[200]: ./media/box-tutorial/tutorial_general_200.png
[201]: ./media/box-tutorial/tutorial_general_201.png
[202]: ./media/box-tutorial/tutorial_general_202.png
[203]: ./media/box-tutorial/tutorial_general_203.png

