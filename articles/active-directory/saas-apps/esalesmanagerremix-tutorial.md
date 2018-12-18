---
title: 教學課程：Azure Active Directory 與 E Sales Manager Remix 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 E Sales Manager Remix 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 89b5022c-0d5b-4103-9877-ddd32b6e1c02
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: d96fd1eacc98e88dc8578b259781cc661cf85933
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39442824"
---
# <a name="integrate-azure-active-directory-with-e-sales-manager-remix"></a>整合 Azure Active Directory 與 E Sales Manager Remix

在本教學課程中，您將了解如何整合 Azure Active Directory (Azure AD) 與 E Sales Manager Remix。

整合 Azure AD 與 E Sales Manager Remix 可提供下列優點：

- 您可以在 Azure AD 中控制可存取 E Sales Manager Remix 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 E Sales Manager Remix (單一登入，又稱為 SSO)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

若要深入了解 SaaS 應用程式與 Azure AD 整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 E Sales Manager Remix 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 E Sales Manager Remix SSO 的訂用帳戶

> [!NOTE]
> 當您測試本教學課程中的步驟時，「不」建議您使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二項主要的基本工作組成：

* 從資源庫新增 E Sales Manager Remix
* 設定並測試 Azure AD 單一登入

## <a name="add-e-sales-manager-remix-from-the-gallery"></a>從資源庫新增 E Sales Manager Remix
若要設定 Azure AD 與 E Sales Manager Remix 的整合，請從資源庫將 E Sales Manager Remix 新增到受控 SaaS 應用程式清單中，做法如下：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory]。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式] > [所有應用程式]。

    ![企業應用程式視窗][2]
    
1. 若要新增應用程式，請選取視窗頂端的 [新增應用程式]。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **E Sales Manager Remix**，從結果清單中選取 [E Sales Manager Remix]，然後選取 [新增]。

    ![結果清單中的 E Sales Manager Remix](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 E Sales Manager Remix 設定及測試 Azure AD 單一登入。

為了單一登入能夠運作，Azure AD 必須識別 E Sales Manager Remix 使用者及其在 Azure AD 中對應的使用者。 換句話說，必須在 Azure AD 使用者和 E Sales Manager Remix 的相同使用者之間建立連結關聯性。

若要設定及測試與 E Sales Manager Remix 搭配運作的 Azure AD 單一登入，請完成接下來五個小節中的建置區塊：

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 E Sales Manager Remix 應用程式中設定單一登入，做法如下：

1. 在 Azure 入口網站的 [E Sales Manager Remix] 應用程式整合網頁上，選取 [單一登入]。

    ![單一登入連結][4]

1. 在 [單一登入] 視窗的 [單一登入模式] 方塊中，選取 [SAML 登入]。
 
    ![單一登入視窗](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_samlbase.png)

1. 在 [E Sales Manager Remix 網域與 URL] 區段下，執行下列步驟：

    ![E Sales Manager Remix 網域及 URL 單一登入資訊](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_url.png)

    a. 在 [登入 URL] 方塊中，以下列格式輸入 URL︰*https://\<Server-Based-URL>/\<sub-domain>/esales-pc*。

    b. 在 [識別碼] 方塊中，以下列格式輸入 URL︰*https://\<Server-Based-URL>/\<sub-domain>/*。

    c. 記下**識別碼**的值，您稍後會在本教學課程中使用。
    
    > [!NOTE] 
    > 上述值並非真正的值。 使用實際的登入 URL 及識別碼來更新這些值。 請連絡 [E Sales Manager Remix 用戶端支援小組](mailto:esupport@softbrain.co.jp)以取得這些值。

1. 在 [SAML 簽署憑證] 下，選取 [憑證 (Base64)]，然後將憑證檔案儲存到您的電腦。

    ![憑證 (Base64) 下載連結](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_certificate.png) 

1. 選取 [檢視及編輯所有其他使用者屬性] 核取方塊，然後選取 **emailaddress** 屬性。
    
    ![使用者屬性視窗](./media/esalesmanagerremix-tutorial/configure1.png)

    [編輯屬性] 視窗隨即開啟。

1. 複製 [命名空間] 和 [名稱] 的值。 產生\<命名空間>/\<名稱> 樣式的值，儲存起來供本教學課程稍後使用。

    ![編輯屬性視窗](./media/esalesmanagerremix-tutorial/configure2.png)

1. 在 [E Sales Manager Remix 設定] 下，選取 [設定 E Sales Manager Remix]。

    ![E Sales Manager Remix 設定](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_configure.png) 

    [設定登入] 視窗隨即開啟。

1. 在 [快速參考] 區段中，複製登出 URL 和 SAML 單一登入服務 URL。

1. 選取 [ **儲存**]。

    ![[儲存] 按鈕](./media/esalesmanagerremix-tutorial/tutorial_general_400.png)

1. 以系統管理員身分登入您的 E Sales Manager Remix 應用程式。

1. 選取右上方的 [到系統管理功能表]。

    ![到系統管理功能表命令](./media/esalesmanagerremix-tutorial/configure4.png)

1. 在左窗格中，選取 [系統設定]  >  [與外部系統合作]。

    ![系統設定以及與外部系統合作連結](./media/esalesmanagerremix-tutorial/configure5.png)
    
1. 在 [與外部系統合作] 視窗中，選取 [SAML]。

    ![與外部系統合作視窗](./media/esalesmanagerremix-tutorial/configure6.png)

1. 在 [SAML 驗證設定] 下，執行下列步驟：

    ![SAML 驗證設定區段](./media/esalesmanagerremix-tutorial/configure3.png)
    
    a. 選取 [PC 版本] 核取方塊。
    
    b. 在 [共同作業項目] 區段中，從下拉式清單中選取 [電子郵件]。

    c. 在 [共同作業項目] 方塊中，貼上您從 Azure 入口網站複製的宣告值，也就是 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress**。

    d. 在 [簽發者 (實體 ID)] 方塊中，貼上您之前從 Azure 入口網站的 [E Sales Manager Remix 網域和 URL] 區段中複製的識別碼值。

    e. 若要從 Azure 入口網站上傳已下載的憑證，選取 [檔案選取]。

    f. 在 [識別提供者登入 URL] 方塊中，貼上您之前從 Azure 入口網站複製的 SAML 單一登入服務 URL。

    g. 在 [識別提供者登出 URL] 方塊中，貼上您之前從 Azure 入口網站複製的登出 URL 值。

    h. 選取 [設定完成]。

> [!TIP]
> 當您設定此應用程式時，可以在 [Azure 入口網站](https://portal.azure.com)中閱讀先前那些指示的簡要版本。 在 [Active Directory]  >  [企業應用程式] 區段中新增應用程式之後，請選取 [單一登入] 索引標籤，然後在底部的 [組態] 區段中存取內嵌的文件。 如需內嵌文件功能的詳細資訊，請參閱 [Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您會執行下列步驟，在 Azure 入口網站中建立測試使用者 Britta Simon：

![建立 Azure AD 測試使用者][100]

1. 在 Azure 入口網站的左側窗格中，選取 [Azure Active Directory]。

    ![Azure Active Directory 連結](./media/paloaltoadmin-tutorial/create_aaduser_01.png)

1. 若要顯示目前使用者清單，請選取 [使用者和群組]  >  [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/paloaltoadmin-tutorial/create_aaduser_02.png)

1. 在 [所有使用者] 視窗頂端，選取 [新增]。

    ![[新增] 按鈕](./media/paloaltoadmin-tutorial/create_aaduser_03.png)
    
    [使用者] 視窗隨即開啟。

1. 在 [使用者] 視窗中，執行下列操作：

    ![[使用者] 視窗](./media/paloaltoadmin-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-an-e-sales-manager-remix-test-user"></a>建立 E Sales Manager Remix 測試使用者

1. 以系統管理員身分登入您的 E Sales Manager Remix 應用程式。

1. 從右上方的功能表選取 [到系統管理功能表]。

    ![E Sales Manager Remix 設定](./media/esalesmanagerremix-tutorial/configure4.png)

1. 選取 [您的公司設定]  >  [維護部門和員工]，然後選取 [已註冊員工]。

    ![已註冊員工索引標籤](./media/esalesmanagerremix-tutorial/user1.png)

1. 在 [新增員工註冊] 區段中，執行下列步驟：
    
    ![新增員工註冊區段](./media/esalesmanagerremix-tutorial/user2.png)

    a. 在 [員工名稱] 方塊中，輸入使用者的名稱 (例如 **Britta**)。

    b. 完成其餘必要欄位。
    
    c. 如果您啟用 SAML，系統管理員將無法從登入頁面登入。 選取 [系統管理員登入] 核取方塊，將系統管理員登入權限授與給使用者。

    d. 選取 [註冊]。

1. 日後若要以系統管理員身分登入，請以具有系統管理員權限的使用者身分登入，然後在右上角選取 [到系統管理功能表]。

    ![到系統管理功能表命令](./media/esalesmanagerremix-tutorial/configure4.png)

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將使用者 E Sales Manager Remix 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。 若要這樣做，請執行下列動作： 

![指派使用者角色][200] 

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至 [目錄] 檢視，然後選取 [企業應用程式]  >  [所有應用程式]。

    ![[企業應用程式] 和 [所有應用程式] 連結][201] 

1. 在 [應用程式] 清單中，選取 [E Sales Manager Remix]。

    ![E Sales Manager Remix 連結](./media/esalesmanagerremix-tutorial/tutorial_esalesmanagerremix_app.png)  

1. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 視窗的 [使用者] 清單中，選取 [Britta Simon]。

1. 選取 [選取] 按鈕。

1. 在 [新增指派] 視窗中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您選取存取面板中的 [E Sales Manager Remix] 圖格時，應該會自動登入您的 E Sales Manager Remix 應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/esalesmanagerremix-tutorial/tutorial_general_01.png
[2]: ./media/esalesmanagerremix-tutorial/tutorial_general_02.png
[3]: ./media/esalesmanagerremix-tutorial/tutorial_general_03.png
[4]: ./media/esalesmanagerremix-tutorial/tutorial_general_04.png

[100]: ./media/esalesmanagerremix-tutorial/tutorial_general_100.png

[200]: ./media/esalesmanagerremix-tutorial/tutorial_general_200.png
[201]: ./media/esalesmanagerremix-tutorial/tutorial_general_201.png
[202]: ./media/esalesmanagerremix-tutorial/tutorial_general_202.png
[203]: ./media/esalesmanagerremix-tutorial/tutorial_general_203.png

