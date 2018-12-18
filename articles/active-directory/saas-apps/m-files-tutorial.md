---
title: 教學課程：Azure Active Directory 與 M-Files 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 M-Files 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 4536fd49-3a65-4cff-9620-860904f726d0
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/28/2017
ms.author: jeedes
ms.openlocfilehash: 41b53cb785679dec47ead99188e5cefbb132d87a
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39424947"
---
# <a name="tutorial-azure-active-directory-integration-with-m-files"></a>教學課程：Azure Active Directory 與 M-Files 整合

在本教學課程中，您將了解如何整合 M-Files 與 Azure Active Directory (Azure AD)。

M-Files 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 M-Files 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 M-Files (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 M-Files 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 M-Files 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 M-Files
1. 設定並測試 Azure AD 單一登入

## <a name="adding-m-files-from-the-gallery"></a>從資源庫新增 M-Files
若要設定 M-Files 與 Azure AD 整合，您需要從資源庫將 M-Files 加入到受控 SaaS 應用程式清單。

**若要從資源庫新增 M-Files，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **M-Files**。

    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/tutorial_m-files_search.png)

1. 在結果窗格中，選取 [M-Files]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/tutorial_m-files_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 M-Files 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 M-Files 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 M-Files 中的相關使用者之間建立連結關聯性。

在 M-Files 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 M-Files 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 M-Files 測試使用者](#creating-a-m-files-test-user)** - 使 M-Files 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 M-Files 應用程式中設定單一登入。

**若要使用 M-Files 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [M-Files] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/m-files-tutorial/tutorial_m-files_samlbase.png)

1. 在 [M-Files 網域與 URL] 區段中，執行下列步驟：

    ![設定單一登入](./media/m-files-tutorial/tutorial_m-files_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<tenantname>.cloudvault.m-files.com/authentication/MFiles.AuthenticationProviders.Core/sso`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `https://<tenantname>.cloudvault.m-files.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [M-Files 客戶支援小組](mailto:support@m-files.com)以取得這些值。 
 
1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![設定單一登入](./media/m-files-tutorial/tutorial_m-files_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/m-files-tutorial/tutorial_general_400.png)

1. 若要取得為您的應用程式設定的 SSO，請連絡 [M-Files 支援小組](mailto:support@m-files.com)並提供下載的中繼資料。
   
    >[!NOTE]
    >如果您想為您的 M-File 傳統型應用程式設定 SSO，請依照下列步驟執行。 如果您只想為 M-Files 的 Web 版本設定 SSO，則不需要任何額外步驟。  

1. 請依照下列步驟設定 M-File 傳統型應用程式，以搭配 Azure AD 啟用 SSO。 若要下載 M-Files，請移至 [M-Files 下載](https://www.m-files.com/en/download-latest-version)頁面。

1. 開啟 [M-Files 桌面設定] 視窗。 然後按一下 [新增]。
   
    ![設定單一登入](./media/m-files-tutorial/tutorial_m_files_10.png)

1. 在 [文件保存庫連線內容] 視窗中，執行下列步驟︰
   
    ![設定單一登入](./media/m-files-tutorial/tutorial_m_files_11.png)  

    在 [伺服器] 區段下，輸入以下的值︰  

    a. 在 [名稱] 中，輸入 `<tenant-name>.cloudvault.m-files.com`。 
 
    b. 在 [連接埠號碼]中，輸入 **4466**。 

    c. 在 [通訊協定]中，選取 **HTTPS**。 

    d. 在 [驗證] 欄位中，選取 [特定的 Windows 使用者]。 然後，系統會向您提示簽署頁面。 請插入您的 Azure AD 認證。 

    e. 在 [伺服器上的保存庫]，選取伺服器上對應的保存庫。
 
    f. 按一下 [確定]。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/m-files-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-m-files-test-user"></a>建立 M-Files 測試使用者

本節的目標是要在 M-Files 中建立名為 Britta Simon 的使用者。 請與 [M-Files 支援小組](mailto:support@m-files.com)合作，在 M-Files 中新增使用者。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 M-Files 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 M-Files，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [M-Files]。 

    ![設定單一登入](./media/m-files-tutorial/tutorial_m-files_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

本節的目標是要使用「存取面板」來測試您的 Azure AD SSO 組態。

當您在存取面板中按一下 [M-Files] 磚時，應該會自動登入您的 M-Files 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/m-files-tutorial/tutorial_general_01.png
[2]: ./media/m-files-tutorial/tutorial_general_02.png
[3]: ./media/m-files-tutorial/tutorial_general_03.png
[4]: ./media/m-files-tutorial/tutorial_general_04.png

[100]: ./media/m-files-tutorial/tutorial_general_100.png

[200]: ./media/m-files-tutorial/tutorial_general_200.png
[201]: ./media/m-files-tutorial/tutorial_general_201.png
[202]: ./media/m-files-tutorial/tutorial_general_202.png
[203]: ./media/m-files-tutorial/tutorial_general_203.png

