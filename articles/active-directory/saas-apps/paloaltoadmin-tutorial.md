---
title: 教學課程：Azure Active Directory 與 Palo Alto 網路 - 系統管理 UI 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Palo Alto 網路 - 系統管理 UI 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: a826eaec-15af-4c85-8855-8a3374d1efb9
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/12/2018
ms.author: jeedes
ms.openlocfilehash: fed368c0df265495d9fee764f86825957fae8bab
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39447419"
---
# <a name="integrate-azure-active-directory-with-palo-alto-networks---admin-ui"></a>整合 Azure Active Directory 與 Palo Alto 網路 - 系統管理 UI

在本教學課程中，您將了解如何整合 Azure Active Directory (Azure AD) 與 Palo Alto 網路 - 系統管理 UI。

將 Azure AD 與 Palo Alto 網路 - 系統管理 UI 整合，您將獲得下列好處：

- 您可以在 Azure AD 中控制可存取 Palo Alto 網路 - 系統管理 UI 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Palo Alto 網路 - 系統管理 UI (單一登入，又稱為 SSO)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

若要了解 SaaS 應用程式與 Azure AD 的整合，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Palo Alto 網路 - 系統管理 UI 整合，您需要下列項目：

- Azure AD 訂用帳戶
- Palo Alto Networks 新一代防火牆或全景 (防火牆的集中式管理系統)

> [!NOTE]
> 當您測試本教學課程中的步驟時，「不」建議您使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 

本教學課程中說明的案例由二個主要建置組塊組成：

* 從資源庫新增 Palo Alto 網路 - 系統管理 UI
* 設定並測試 Azure AD 單一登入

## <a name="add-palo-alto-networks---admin-ui-from-the-gallery"></a>從資源庫新增 Palo Alto 網路 - 系統管理 UI
若要設定 Azure AD 與 Palo Alto 網路 - 系統管理 UI 的整合，請執行下列動作，從資源庫將 Palo Alto 網路 - 系統管理 UI 新增到受控 SaaS 應用程式清單：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory]。 

    ![Azure Active Directory 按鈕][1]

1. 選取 [企業應用程式] > [所有應用程式]。

    ![企業應用程式視窗][2]
    
1. 若要新增新的應用程式，請選取視窗頂端的 [新增應用程式] 按鈕。

    ![[新增應用程式] 按鈕][3]

1. 在搜尋方塊中輸入 **Palo Alto 網路 - 系統管理 UI**，從結果清單中選取 [Palo Alto 網路 - 系統管理 UI]，然後選取 [新增]。

    ![結果清單中的 [Palo Alto 網路 - 系統管理 UI]](./media/paloaltoadmin-tutorial/tutorial_step4-add-from-the-gallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Palo Alto 網路 - 系統管理 UI 搭配運作的 Azure AD 單一登入。

為了單一登入能夠運作，Azure AD 必須識別 Palo Alto 網路 - 系統管理 UI 使用者及其在 Azure AD 中對應的使用者。 換句話說，必須在 Azure AD 使用者與 Palo Alto 網路 - 系統管理 UI 中的相同使用者之間建立連結關聯性。

若要建立連結關聯性，請將 Azure AD 中的*使用者名稱*值指派為 Palo Alto 網路 - 系統管理 UI *使用者名稱*。

若要設定及測試與 Palo Alto 網路 - 系統管理 UI 搭配運作的 Azure AD 單一登入，請完成接下來五個小節中的建置組塊。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

執行下列動作，在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Palo Alto 網路 - 系統管理 UI 應用程式中設定單一登入：

1. 在 Azure 入口網站中的 [Palo Alto 網路 - 系統管理 UI] 應用程式整合頁面上，選取 [單一登入]。

    ![單一登入連結][4]

1. 在 [單一登入] 視窗的 [單一登入模式] 方塊中，選取 [SAML 登入]。
 
    ![單一登入視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_samlbase.png)

1. 在 [Palo Alto 網路 - 系統管理 UI 網域和 URL] 下，執行下列動作：

    ![[Palo Alto 網路 - 系統管理 UI 網域和 URL] 單一登入資訊](./media/paloaltoadmin-tutorial/tutorial_general_show_advanced_url.png)
    
    a. 在 [登入 URL] 方塊中，以下列格式輸入 URL︰*https://\<Customer Firewall FQDN>/php/login.php*。

    b. 在 [識別碼] 方塊中，以下列格式輸入 URL︰*https://\<Customer Firewall FQDN>:443/SAML20/SP*。
    
    c. 在 [回覆 URL] 方塊中，以下列格式輸入判斷提示取用者服務 (ACS) URL︰*https://\<Customer Firewall FQDN>:443/SAML20/SP/ACS*。
    
    > [!NOTE] 
    > 上述值並非真正的值。 請使用實際的登入 URL 及識別碼來更新這些值。 若要取得這些值，請連絡 [Palo Alto 網路 - 系統管理 UI 用戶端支援小組](https://support.paloaltonetworks.com/support)。 
 
1. 由於 Palo Alto 網路 - 系統管理 UI 應用程式需要特定格式的 SAML 判斷提示，請依照下圖中的說明設定宣告。 請執行下列動作，管理 [應用程式整合] 頁面的 [使用者屬性] 區段中的屬性值：
    
    ![SAML Token 屬性清單](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_attribute.png)
    
   > [!NOTE]
   > 這些屬性值只是範例，因此您必須對應 *username* 和 *adminrole* 的適當值。 此外還有另一個選用屬性 *accessdomain*，可用來限制管理員對防火牆上特定虛擬系統的存取。
   >
        
    | 屬性名稱 | 屬性值 |
    | --- | --- |    
    | username | user.userprincipalname |
    | adminrole | customadmin |

    a. 選取 [新增屬性]。  
    
    ![[新增屬性] 按鈕](./media/paloaltoadmin-tutorial/tutorial_attribute_04.png)

    [新增屬性] 視窗隨即開啟。

    ![[新增屬性] 視窗](./media/paloaltoadmin-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 方塊中，輸入針對該資料列顯示的屬性值。
    
    d. 選取 [確定] 。

    > [!NOTE]
    > 如需關於這些屬性的詳細資訊，請參閱下列文章：
    > * [系統管理 UI (adminrole) 的系統管理角色設定檔](https://www.paloaltonetworks.com/documentation/80/pan-os/pan-os/firewall-administration/manage-firewall-administrators/configure-an-admin-role-profile)
    > * [系統管理 UI (accessdomain) 的裝置存取網域](https://www.paloaltonetworks.com/documentation/80/pan-os/web-interface-help/device/device-access-domain)
    >

1. 在 [SAML 簽章憑證] 下選取 [中繼資料 XML]，然後選取 [儲存]。

    ![中繼資料 XML 下載連結](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_certificate.png) 

    ![[儲存] 按鈕](./media/paloaltoadmin-tutorial/tutorial_general_400.png)

1. 以系統管理員的身分，在新視窗中開啟 Palo Alto 網路防火牆的系統管理 UI。

1. 選取 [裝置] 索引標籤。

    ![[裝置] 索引標籤](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_admin1.png)

1. 在左窗格中選取 [SAML 身分識別提供者]，然後選取 [匯入] 以匯入中繼資料檔案。

    ![匯入中繼資料檔案按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_admin2.png)

1. 在 [SAML 身分識別提供者伺服器設定檔匯入] 視窗中，執行下列動作：

    ![[SAML 身分識別提供者伺服器設定檔匯入] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_idp.png)

    a. 在 [設定檔名稱] 方塊中提供名稱，例如 **AzureAD 系統管理 UI**。
    
    b. 在 [身分識別提供者中繼資料] 下選取 [瀏覽]，然後選取您先前從 Azure 入口網站下載的 metadata.xml 檔案。
    
    c. 清除 [驗證身分識別提供者憑證] 核取方塊。
    
    d. 選取 [確定] 。
    
    e. 若要認可防火牆的組態，請選取 [認可]。

1. 在左窗格中選取 [SAML 身分識別提供者]，然後選取您在先前的步驟中建立的 SAML 身分識別提供者設定檔 (例如 **AzureAD 系統管理 UI**)。 

    ![SAML 身分識別提供者設定檔](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_idp_select.png)

1. 在 [SAML 身分識別提供者伺服器設定檔] 視窗中，執行下列動作：

    ![[SAML 身分識別提供者伺服器設定檔] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_slo.png)
  
    a. 在 [身分識別提供者 SLO URL] 方塊中，將先前匯入的 SLO URL 取代為下列 URL：**https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0**。
  
    b. 選取 [確定] 。

1. 在 Palo Alto 網路防火牆的系統管理 UI 中選取 [裝置]，然後選取 [管理員角色]

1. 選取 [新增] 按鈕。 

1. 在 [管理員角色設定檔] 視窗的 [名稱] 方塊中，提供管理員角色的名稱 (例如 **fwadmin**)。  
    管理員角色名稱應符合身分識別提供者所傳送的 SAML 管理員角色屬性名稱。 在步驟 4 中已建立管理員角色的名稱和值。

    ![設定 Palo Alto 網路管理員角色](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_adminrole.png)
  
1. 在防火牆的系統管理 UI 上選取 [裝置]，然後選取 [驗證設定檔]。

1. 選取 [新增] 按鈕。 

1. 在 [驗證設定檔] 視窗中，執行下列動作： 

    ![[驗證設定檔] 視窗](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authentication_profile.png)

    a. 在 [名稱] 方塊中提供名稱，例如 **AzureSAML_Admin_AuthProfile**。
    
    b. 選取 [類型] 下拉式清單中，選取 [SAML]。 
   
    c. 在 [IdP 伺服器設定檔] 下拉式清單中，選取適當的 SAML 身分識別提供者伺服器設定檔 (例如 **AzureAD 系統管理 UI**)。
   
    c. 選取 [啟用單一登出] 核取方塊。
    
    d. 在 [管理員角色屬性] 方塊中輸入屬性名稱，例如 **adminrole**。 
    
    e. 選取 [進階] 索引標籤，然後在 [允許清單] 下選取 [新增]。 
    
    ![[進階] 索引標籤上的 [新增] 按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_allowlist.png)
    
    f. 選取 [全部] 核取方塊，或選取可以使用此設定檔進行驗證的特定使用者和群組。  
    當使用者進行驗證時，防火牆會針對此清單中的項目比對相關聯的使用者名稱或群組。 如果您未新增項目，則無法驗證任何使用者。

    g. 選取 [確定] 。

1. 若要使用 Azure 讓管理員能夠使用 SAML SSO，請選取 [裝置] > [設定]。 在 [設定] 窗格中選取 [管理] 索引標籤，然後選取 [驗證設定] 下的 [設定] (「齒輪」) 按鈕。 

 ![[設定] 按鈕](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authsetup.png)

1. 選取您在步驟 17 中建立的 SAML 驗證設定檔，例如 **AzureSAML_Admin_AuthProfile**。

 ![[驗證設定檔] 欄位](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_authsettings.png)

1. 選取 [確定] 。

1. 若要認可組態，請選取 [認可]。


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
 
### <a name="create-a-palo-alto-networks---admin-ui-test-user"></a>建立 Palo Alto 網路 - 系統管理 UI 測試使用者

Palo Alto 網路 - 系統管理 UI 支援 Just-In-Time 使用者佈建。 如果使用者尚不存在，在成功驗證之後即會在系統中自動建立。 您不需要手動建立使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Palo Alto 網路 - 系統管理 UI 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。 若要這樣做，請執行下列動作：

![指派使用者角色][200] 

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至 [目錄] 檢視，然後選取 [企業應用程式]  >  [所有應用程式]。

    ![[企業應用程式] 和 [所有應用程式] 連結][201] 

1. 在 [應用程式] 清單中，選取 [Palo Alto 網路 - 系統管理 UI]。

    ![Palo Alto 網路 - 系統管理 UI](./media/paloaltoadmin-tutorial/tutorial_paloaltoadmin_app.png)  

1. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 視窗的 [使用者] 清單中，選取 [Britta Simon]。

1. 選取 [選取] 按鈕。

1. 在 [新增指派] 視窗中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [Palo Alto 網路 - 系統管理 UI ] 圖格時，您應會自動登入「Palo Alto 網路 - 系統管理 UI」應用程式。

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/paloaltoadmin-tutorial/tutorial_general_01.png
[2]: ./media/paloaltoadmin-tutorial/tutorial_general_02.png
[3]: ./media/paloaltoadmin-tutorial/tutorial_general_03.png
[4]: ./media/paloaltoadmin-tutorial/tutorial_general_04.png

[100]: ./media/paloaltoadmin-tutorial/tutorial_general_100.png

[200]: ./media/paloaltoadmin-tutorial/tutorial_general_200.png
[201]: ./media/paloaltoadmin-tutorial/tutorial_general_201.png
[202]: ./media/paloaltoadmin-tutorial/tutorial_general_202.png
[203]: ./media/paloaltoadmin-tutorial/tutorial_general_203.png

