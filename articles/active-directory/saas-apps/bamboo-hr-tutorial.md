---
title: 教學課程：Azure Active Directory 與 BambooHR 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 BambooHR 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: f826b5d2-9c64-47df-bbbf-0adf9eb0fa71
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/18/2018
ms.author: jeedes
ms.openlocfilehash: dc6664321588d383b4656199c3e8ea79159ca850
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39437670"
---
# <a name="tutorial-azure-active-directory-integration-with-bamboohr"></a>教學課程：Azure Active Directory 與 BambooHR 整合

在本教學課程中，您會了解如何將 BambooHR 與 Azure Active Directory (Azure AD) 整合。

將 BambooHR 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 BambooHR 的人員。
- 您可以讓使用者搭配他們的 Azure AD 帳戶使用單一登入 (SSO) 自動登入 BambooHR。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 BambooHR 的整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 BambooHR SSO 的訂用帳戶

> [!NOTE]
> 當您測試本教學課程中的步驟時，建議您不要使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[申請一個月免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二個主要構成要素組成：

1. 從資源庫新增 BambooHR
1. 設定並測試 Azure AD 單一登入

## <a name="add-bamboohr-from-the-gallery"></a>從資源庫新增 BambooHR
若要設定將 BambooHR 整合到 Azure AD 中，請執行下列操作，將 BambooHR 從資源庫新增到受控 SaaS 應用程式清單：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory]。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式] > [所有應用程式]。

    ![[企業應用程式] 窗格][2]
    
1. 若要新增應用程式，請選取 [新增應用程式]。

    ![[新增應用程式] 按鈕][3]

1. 在搜尋方塊中，輸入 **BambooHR**。 在結果清單中，選取 [BambooHR]，然後選取 [新增]。

    ![結果清單中的 BambooHR](./media/bamboo-hr-tutorial/tutorial_bamboohr_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會使用測試使用者 "Britta Simon" 來設定及測試與 BambooHR 搭配運作的 Azure AD SSO。

若要讓 SSO 能夠運作，Azure AD 必須知道它在 BambooHR 中互相對應的使用者。 換句話說，您必須在 Azure AD 使用者與 BambooHR 中的相關使用者之間建立連結關聯性。

若要在 BambooHR 中建立此連結關聯性，請指派 Azure AD **使用者名稱**值作為 BambooHR [Username] \(使用者名稱\) 值。

若要設定及測試與 BambooHR 搭配運作的 Azure AD SSO，請完成接下來五個小節中的構成要素。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會執行下列操作，在 Azure 入口網站中啟用 Azure AD SSO，並在 BambooHR 應用程式中設定 SSO：

1. 在 Azure 入口網站的 [BambooHR] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 視窗的 [模式] 下拉式清單中，選取 [SAML 登入]。
 
    ![[單一登入] 視窗](./media/bamboo-hr-tutorial/tutorial_bamboohr_samlbase.png)

1. 在 [BambooHR 網域及 URL] 底下，執行下列操作：

    ![[BambooHR 網域及 URL] 區段](./media/bamboo-hr-tutorial/tutorial_bamboohr_url.png)

    a. 在 [登入 URL] 方塊中，以下列格式輸入 URL︰`https://<company>.bamboohr.com`。

    b. 在 [識別碼] 方塊中，輸入值：`BambooHR-SAML`。

    > [!NOTE] 
    > [登入 URL] 的值不是真正的值。 請使用您的實際登入 URL 來更新此值。 若要取得此值，請連絡 [BambooHR 用戶端支援小組](https://www.bamboohr.com/contact.php) \(英文\)。 
 
1. 在 [SAML 簽署憑證] 下，選取 [憑證 (Base64)]，然後將憑證檔案儲存到您的電腦。

    ![憑證下載連結](./media/bamboo-hr-tutorial/tutorial_bamboohr_certificate.png) 

1. 選取 [ **儲存**]。

    ![[儲存] 按鈕](./media/bamboo-hr-tutorial/tutorial_general_400.png)

1. 在 [BambooHR 設定] 區段底下，選取 [設定 BambooHR] 以開啟 [設定登入] 視窗。 在 [快速參考] 區段中，複製 [SAML 單一登入服務 URL] 供稍後使用。

    ![BambooHR 設定](./media/bamboo-hr-tutorial/tutorial_bamboohr_configure.png) 

1. 在新視窗中，以系統管理員身分登入您的 BambooHR 公司網站。

1. 在首頁上，執行下列操作：
   
    ![BambooHR 單一登入頁面](./media/bamboo-hr-tutorial/ic796691.png "單一登入")   

    a. 選取 [Apps] \(應用程式\)。
   
    b. 在 [Apps] \(應用程式\) 窗格中，選取 [Single Sign-On] \(單一登入\)。
   
    c. 選取 [SAML Single Sign-On] \(SAML 單一登入\)。

1. 在 [SAML Single Sign-On] \(SAML 單一登入\) 窗格中，執行下列操作：
   
    ![[SAML Single Sign-On] \(SAML 單一登入\) 窗格](./media/bamboo-hr-tutorial/IC796692.png "SAML 單一登入")
   
    a. 在 [SSO Login Url] \(SSO 登入 URL\) 方塊中，貼上您在步驟 6 中從 Azure 入口網站複製的 **SAML 單一登入服務 URL**。
      
    b. 在「記事本」中開啟從 Azure 入口網站下載的 Base-64 編碼憑證，複製其內容，然後貼到 [X.509 Certificate] \(X.509 憑證\) 方塊中。
   
    c. 選取 [ **儲存**]。

> [!TIP]
> 當您設定此應用程式時，可以在 [Azure 入口網站](https://portal.azure.com)中閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增應用程式之後，只要選取 [單一登入] 索引標籤，即可透過底部的 [設定] 區段存取內嵌的文件。 如需相關資訊，請參閱 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 "Britta Simon" 的測試使用者。

   ![建立 Azure AD 測試使用者 Britta Simon][100]

若要在 Azure AD 中建立測試使用者，請執行下列動作：

1. 在 Azure 入口網站的左側窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 按鈕](./media/bamboo-hr-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/bamboo-hr-tutorial/create_aaduser_02.png)

1. 在 [所有使用者] 窗格頂端，選取 [新增]。

    ![[新增] 按鈕](./media/bamboo-hr-tutorial/create_aaduser_03.png)

1. 在 [使用者] 視窗中，執行下列操作：

    ![[使用者] 視窗](./media/bamboo-hr-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-bamboohr-test-user"></a>建立 BambooHR 測試使用者

若要讓 Azure AD 使用者能夠登入 BambooHR，請執行下列操作，在 BambooHR 中手動設定這些使用者：

1. 以系統管理員身分登入您的 **BambooHR** 網站。

1. 在頂端的工具列中，選取 [Settings] \(設定\)。
   
    ![[Settings] \(設定\) 按鈕](./media/bamboo-hr-tutorial/IC796694.png "設定")

1. 選取 [概觀]。

1. 在左側窗格中，選取 [Security] \(安全性\) > [Users] \(使用者\)。

1. 輸入您想要設定之有效 Azure AD 帳戶的使用者名稱、密碼及電子郵件地址。

1. 選取 [ **儲存**]。
        
>[!NOTE]
>若要設定 Azure AD 使用者帳戶，您也可以使用 BambooHR 使用者帳戶建立工具或 API。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

請將 BambooHR 的存取權授與 Britta Simon 來讓她能夠使用 Azure SSO。

![指派使用者角色][200] 

若要將使用者 Britta Simon 指派給 BambooHR，請執行下列操作：

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至 [目錄] 檢視，然後選取 [企業應用程式] > [所有應用程式]。

    ![指派使用者][201] 

1. 在 [企業應用程式] 清單中，選取 [BambooHR]。

    ![[企業應用程式] 清單中的 [BambooHR] 連結](./media/bamboo-hr-tutorial/tutorial_bamboohr_app.png)  

1. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增] 按鈕，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 視窗的 [使用者] 清單中，選取 [Britta Simon]。

1. 選取 [選取] 按鈕。

1. 在 [新增指派] 視窗中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

請使用「存取面板」來測試您的 Azure AD SSO 設定。

當您在「存取面板」中選取 [BambooHR] 圖格時，應該會自動登入您的 BambooHR 應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/bamboo-hr-tutorial/tutorial_general_01.png
[2]: ./media/bamboo-hr-tutorial/tutorial_general_02.png
[3]: ./media/bamboo-hr-tutorial/tutorial_general_03.png
[4]: ./media/bamboo-hr-tutorial/tutorial_general_04.png

[100]: ./media/bamboo-hr-tutorial/tutorial_general_100.png

[200]: ./media/bamboo-hr-tutorial/tutorial_general_200.png
[201]: ./media/bamboo-hr-tutorial/tutorial_general_201.png
[202]: ./media/bamboo-hr-tutorial/tutorial_general_202.png
[203]: ./media/bamboo-hr-tutorial/tutorial_general_203.png

