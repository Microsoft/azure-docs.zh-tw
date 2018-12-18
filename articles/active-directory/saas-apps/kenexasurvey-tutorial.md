---
title: 教學課程：Azure Active Directory 與 IBM Kenexa Survey Enterprise 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 IBM Kenexa Survey Enterprise 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: c7aac6da-f4bf-419e-9e1a-16b460641a52
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/30/2017
ms.author: jeedes
ms.openlocfilehash: 6828617e0ae61a3784e4db3d1c2ecf4ce9862ce2
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39449470"
---
# <a name="tutorial-azure-active-directory-integration-with-ibm-kenexa-survey-enterprise"></a>教學課程：Azure Active Directory 與 IBM Kenexa Survey Enterprise 整合

在本教學課程中，您會了解如何整合 IBM Kenexa Survey Enterprise 與 Azure Active Directory (Azure AD)。

IBM Kenexa Survey Enterprise 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 IBM Kenexa Survey Enterprise 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶，透過單一登入 (SSO) 自動登入 IBM Kenexa Survey Enterprise。
- 您可以集中管理您的帳戶：Azure 入口網站。

若您想深入了解軟體即服務 (SaaS) 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 IBM Kenexa Survey Enterprise 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 IBM Kenexa Survey Enterprise SSO 的訂用帳戶

> [!NOTE]
> 當您測試本教學課程中的步驟時，不建議您使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD SSO。 教學課程中說明的情節由二個主要建置組塊組成：

* 從資源庫新增 IBM Kenexa Survey Enterprise
* 設定並測試 Azure AD SSO

## <a name="add-ibm-kenexa-survey-enterprise-from-the-gallery"></a>從資源庫新增 IBM Kenexa Survey Enterprise
若要設定將 IBM Kenexa Survey Enterprise 整合到 Azure AD 中，您需要從資源庫將 IBM Kenexa Survey Enterprise 新增到受控 SaaS 應用程式清單。

若要從資源庫新增 IBM Kenexa Survey Enterprise，請執行下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)的左窗格中，按一下 [Azure Active Directory] 按鈕。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增應用程式，請按一下 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中輸入 **IBM Kenexa Survey Enterprise**。

    ![建立 Azure AD 測試使用者](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_search.png)

1. 在結果清單中，選取 [IBM Kenexa Survey Enterprise]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 IBM Kenexa Survey Enterprise](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_addfromgallery.png)

##  <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 IBM Kenexa Survey Enterprise 搭配運作的 Azure AD SSO。

為了讓 SSO 運作，Azure AD 需要識別 Azure AD 中對應的 IBM Kenexa Survey Enterprise 使用者。 換句話說，必須在 Azure AD 使用者和 IBM Kenexa Survey Enterprise 中的相關使用者之間，建立連結關聯性。

若要建立連結關聯性，請將 IBM Kenexa Survey Enterprise 中**使用者名稱**的值，指派為 Azure AD 中**使用者名稱**的值。

若要設定及測試與 IBM Kenexa Survey Enterprise 搭配運作的 Azure AD SSO，請完成下兩節中的建置組塊。

### <a name="configure-azure-ad-sso"></a>設定 Azure AD SSO

在本節中，您會執行下列步驟，在 Azure 入口網站中啟用 Azure AD SSO，並在 IBM Kenexa Survey Enterprise 中設定 SSO：

1. 在 Azure 入口網站的 [IBM Kenexa Survey Enterprise] 應用程式整合頁面上，按一下 [單一登入]。

    ![IBM Kenexa Survey Enterprise 設定單一登入連結][4]

1. 在 [單一登入] 對話方塊的 [模式] 方塊中，選取 [SAML 登入] 來啟用 SSO。
 
    ![單一登入對話方塊](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_samlbase.png)

1. 在 [IBM Kenexa Survey Enterprise 網域及 URL] 區段中，執行下列步驟：

    ![IBM Kenexa Survey Enterprise 網域及 URL 單一登入資訊](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_url.png)

    a. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL：`https://surveys.kenexa.com/<companycode>`

    b. 在 [回覆 URL] 文字方塊中，使用下列模式輸入 URL：`https://surveys.kenexa.com/<companycode>/tools/sso.asp`

    > [!NOTE] 
    > 上述值並非真正的值。 請使用實際的識別碼和回覆 URL 來更新這些值。 若要取得實際的值，請連絡 [IBM Kenexa Survey Enterprise 支援小組](https://www.ibm.com/support/home/?lnk=fcw)。

1. 在 [SAML 簽署憑證] 下，按一下 [憑證 (Base64/)]，然後將憑證檔案儲存到您的電腦。

    ![憑證 (Base64) 下載連結](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_certificate.png) 

    IBM Kenexa Survey Enterprise 應用程式需要特定格式的「安全性聲明標記語言 (SAML)」判斷提示，所以您必須將自訂屬性對應新增至 SAML 權杖屬性設定。 回應中的使用者識別碼宣告的值，必須符合 Kenexa 系統中設定的 SSO 識別碼。 若要將組織中適當的使用者識別碼對應為 SSO 網際網路資料包通訊協定 (IDP)，請與 [IBM Kenexa Survey Enterprise 支援小組](https://www.ibm.com/support/home/?lnk=fcw)合作。 

    根據預設，Azure AD 會將使用者識別碼設為使用者主體名稱 (UPN) 值。 您可以在 [屬性] 索引標籤中變更此值，如以下螢幕擷取畫面所示。 只有在您正確完成對應之後，整合才有作用。
    
    ![[使用者屬性] 對話方塊](./media/kenexasurvey-tutorial/tutorial_attribute.png) 

1. 按一下 [檔案] 。

    ![設定單一登入儲存按鈕](./media/kenexasurvey-tutorial/tutorial_general_400.png)

1. 若要開啟 [設定登入] 視窗，請在 [IBM Kenexa Survey Enterprise 組態] 下，按一下 [設定 IBM Kenexa Survey Enterprise]。 
 
    ![設定 IBM Kenexa Survey Enterprise 連結](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_configure.png)

1. 從 [快速參考] 區段中，複製 [登出 URL]、[SAML 實體識別碼] 和 [SAML 單一登入服務 URL] 值。

1. 在 [設定登入] 視窗的 [快速參考] 下，複製 [登出 URL]、[SAML 實體識別碼] 和 [SAML 單一登入服務 URL] 值。

1. 若要在 **IBM Kenexa Survey Enterprise** 端設定 SSO，請將已下載的**憑證 (Base64)**、**登出 URL**、**SAML 實體識別碼**和 **SAML 單一登入服務 URL** 值，傳送給 [IBM Kenexa Survey Enterprise 支援小組](https://www.ibm.com/support/home/?lnk=fcw)。

> [!TIP]
> 當您設定應用程式時，您可以在 [Azure 入口網站](https://portal.azure.com)中參考這些指示的簡要版本。 當您從 [Active Directory] > [企業應用程式] 區段新增應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過最後的 [設定] 區段存取內嵌的文件。 若要深入了解內嵌文件功能，請參閱 [Azure AD 內嵌文件](https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
在本節中，您會執行下列步驟，在 Azure 入口網站中建立測試使用者 Britta Simon：

![建立 Azure AD 測試使用者][100]

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/kenexasurvey-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![[使用者和群組] 與 [所有使用者] 連結](./media/kenexasurvey-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。
 
    ![[新增] 按鈕](./media/kenexasurvey-tutorial/create_aaduser_03.png) 

1. 在 [使用者] 對話方塊中，執行下列步驟：
 
    ![[使用者] 對話方塊](./media/kenexasurvey-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-an-ibm-kenexa-survey-enterprise-test-user"></a>建立 IBM Kenexa Survey Enterprise 測試使用者

在本節中，您要在 IBM Kenexa Survey Enterprise 中建立名為 Britta Simon 的使用者。 

若要在 IBM Kenexa Survey Enterprise 中建立使用者，並對應其 SSO 識別碼，您可以與 [IBM Kenexa Survey Enterprise 支援小組](https://www.ibm.com/support/home/?lnk=fcw)合作。 此 SSO 識別碼值也應該至 Azure AD 中的使用者識別碼值。 您可以在 [屬性] 索引標籤上變更此預設設定。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 IBM Kenexa Survey Enterprise 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure SSO。

![指派使用者角色][200] 

若要將使用者 Britta Simon 指派給 IBM Kenexa Survey Enterprise，請執行下列步驟：

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視，移至 [目錄] 檢視，選取 [企業應用程式]，然後按一下 [所有應用程式]。

    ![[企業應用程式] 和 [所有應用程式] 連結][201] 

1. 在 [應用程式] 清單中，選取 [IBM Kenexa Survey Enterprise]。

    ![[應用程式] 清單中的 IBM Kenexa Survey Enterprise 連結](./media/kenexasurvey-tutorial/tutorial_kenexasurvey_app.png) 

1. 在左窗格中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202] 

1. 按一下 [新增] 按鈕，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊的 [使用者] 清單中，選取 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，按一下 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD SSO 組態。

當您在存取面板中按一下 [IBM Kenexa Survey Enterprise] 圖格時，應該會自動登入您的 IBM Kenexa Survey Enterprise 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/kenexasurvey-tutorial/tutorial_general_01.png
[2]: ./media/kenexasurvey-tutorial/tutorial_general_02.png
[3]: ./media/kenexasurvey-tutorial/tutorial_general_03.png
[4]: ./media/kenexasurvey-tutorial/tutorial_general_04.png

[100]: ./media/kenexasurvey-tutorial/tutorial_general_100.png

[200]: ./media/kenexasurvey-tutorial/tutorial_general_200.png
[201]: ./media/kenexasurvey-tutorial/tutorial_general_201.png
[202]: ./media/kenexasurvey-tutorial/tutorial_general_202.png
[203]: ./media/kenexasurvey-tutorial/tutorial_general_203.png

 
