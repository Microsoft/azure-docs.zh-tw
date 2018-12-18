---
title: 教學課程：Azure Active Directory 與 Cisco Webex 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Cisco Webex 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 26704ca7-13ed-4261-bf24-fd6252e2072b
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/08/2018
ms.author: jeedes
ms.openlocfilehash: 73e20afdcacec76482f8ebf01bf2cef2105912a6
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40005509"
---
# <a name="tutorial-azure-active-directory-integration-with-cisco-webex"></a>教學課程：Azure Active Directory 與 Cisco Webex 整合

在本教學課程中，您將了解如何整合 Cisco Webex 與 Azure Active Directory (Azure AD)。

將 Cisco Webex 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Cisco Webex 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Cisco Webex。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需有關 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Cisco Webex 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Cisco Webex 單一登入的訂用帳戶

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，可以[取得一個月的免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Cisco Webex
2. 設定並測試 Azure AD 單一登入

## <a name="add-cisco-webex-from-the-gallery"></a>從資源庫新增 Cisco Webex
若要設定將 Cisco Webex 整合到 Azure AD 中，您需要將 Cisco Webex 從資源庫新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Cisco Webex，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Cisco Webex**。 

5. 從結果面板中選取 [Cisco Webex]。 然後選取 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Cisco Webex](./media/cisco-webex-tutorial/tutorial_ciscowebex_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Cisco Webex 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Cisco Webex 與 Azure AD 中互相對應的使用者。 換句話說，您必須在 Azure AD 使用者與 Cisco Webex 中的相關使用者之間建立連結。

在 Cisco Webex 中，為 [Username] \(使用者名稱\) 值提供與 Azure AD 中**使用者名稱**相同的值。 現在您已經建立兩名使用者之間的連結。 

若要設定及測試與 Cisco Webex 搭配運作的 Azure AD 單一登入，請完成下列構成要素：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
2. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
3. [建立 Cisco Webex 測試使用者](#create-a-cisco-webex-test-user)，以在 Cisco Webex 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的對應項目。
4. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. [測試單一登入](#test-single-sign-on)，以驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Cisco Webex 應用程式中設定單一登入。

**若要設定與 Cisco Webex 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Cisco Webex] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

2. 若要啟用單一登入，請在 [單一登入] 對話方塊的 [模式] 下拉式清單中，選取 [SAML 登入]。
 
    ![單一登入對話方塊](./media/cisco-webex-tutorial/tutorial_ciscowebex_samlbase.png)

3. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Cisco Webex 公司網站。

4. 按一下功能表左側的 [設定]。

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_cisco_webex_10.png)

5. 在 [設定] 頁面上，向下捲動到 [驗證] 區段之下，按一下 [修改]。

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_cisco_webex_14.png)

6. 選取 [整合協力廠商識別提供者 (進階)]，然後移至下一個畫面。

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_cisco_webex_15.png)

7. 在 [匯出目錄中繼資料] 頁面上，按一下 [下載中繼資料檔案] 以下載中繼資料檔案。

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_cisco_webex_16.png)

8. 在 Azure 入口網站的 [Cisco Webex 網域及 URL] 區段之下，執行下列步驟，以上傳已下載的**服務提供者中繼資料檔案**並設定應用程式：

    a. 按一下 [上傳中繼資料檔案]。

    ![Cisco Webex 網域及 URL 單一登入資訊](./media/cisco-webex-tutorial/tutorial_ciscowebex_upload.png)

    b. 按一下**資料夾圖示**以選取中繼資料檔案，然後按一下 [上傳]。

    ![Cisco Webex 網域及 URL 單一登入資訊](./media/cisco-webex-tutorial/tutorial_ciscowebex_uploadconfig.png)

    c. 成功完成上傳**服務提供者中繼資料檔案**之後，[Cisco Webex 網域及 URL] 區段文字方塊中會自動填入 [識別碼] 和 [回覆 URL] 的值，如下所示：

    ![Cisco Webex 網域及 URL 單一登入資訊](./media/cisco-webex-tutorial/tutorial_ciscowebex_url.png)

    d. 在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://<SUBDOMAIN>.webex.com/`
     
    > [!NOTE] 
    > 這些都不是真正的值。 以實際的登入 URL 更新這些值。 請連絡 [Cisco Webex 用戶端支援小組](https://www.webex.co.in/support/support-overview.html) \(英文\) 以取得這些值。

9. Cisco Webex 應用程式預期 SAML 判斷提示會包含特定屬性。 設定此應用程式的下列屬性。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_ciscowebex_07.png) 

10. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    |  屬性名稱  | 屬性值 |
    | --------------- | -------------------- |    
    |   firstname    | user.givenname |
    |   lastname    | user.surname |
    |   UID    | user.mail |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定] 。

11. 在 [SAML 簽署憑證] 區段上，選取 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/cisco-webex-tutorial/tutorial_ciscowebex_certificate.png) 

12. 選取 [ **儲存**]。

    ![設定單一登入 [儲存] 按鈕](./media/cisco-webex-tutorial/tutorial_general_400.png)
    
13. 在 Cisco Webex 公司網站系統管理員頁面上，使用檔案瀏覽器選項來找出並上傳 Azure AD 中繼資料檔案。 然後，選取 [需要中繼資料中憑證授權單位所簽署的憑證 (較安全)]，然後前往下一個畫面。 

    ![設定單一登入](./media/cisco-webex-tutorial/tutorial_cisco_webex_11.png)

14. 選取 [測試 SSO 連接]，並在新的瀏覽器索引標籤開啟時，透過登入向 Azure AD 進行驗證。

15. 返回 [Cisco Cloud Collaboration Management] 瀏覽器索引標籤。如果測試成功，請選取 [這項測試成功。啟用單一登入] 選項，然後按 [儲存]。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/cisco-webex-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/cisco-webex-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/cisco-webex-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/cisco-webex-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-a-cisco-webex-test-user"></a>建立 Cisco Webex 測試使用者

本節的目標是要在 Cisco Webex 中建立名為 Britta Simon 的使用者。 Cisco Webex 支援預設啟用的 Just-In-Time 佈建和自動使用者佈建。 您可以在[這裡](https://docs.microsoft.com/azure/active-directory/saas-apps/cisco-webex-provisioning-tutorial)找到關於如何設定自動使用者佈建的更多詳細資料。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Cisco Webex 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Cisco Webex，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視。 接著，移至目錄檢視，然後移至 [企業應用程式]。  

2. 選取 [所有應用程式]。

    ![指派使用者][201] 

3. 在應用程式清單中，選取 [Cisco Webex]。

    ![應用程式清單中的 [Cisco Webex] 連結](./media/cisco-webex-tutorial/tutorial_ciscowebex_app.png)  

3. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

6. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

7. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [Cisco Webex] 圖格時，會自動登入您的 Cisco Webex 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/cisco-webex-tutorial/tutorial_general_01.png
[2]: ./media/cisco-webex-tutorial/tutorial_general_02.png
[3]: ./media/cisco-webex-tutorial/tutorial_general_03.png
[4]: ./media/cisco-webex-tutorial/tutorial_general_04.png

[100]: ./media/cisco-webex-tutorial/tutorial_general_100.png

[200]: ./media/cisco-webex-tutorial/tutorial_general_200.png
[201]: ./media/cisco-webex-tutorial/tutorial_general_201.png
[202]: ./media/cisco-webex-tutorial/tutorial_general_202.png
[203]: ./media/cisco-webex-tutorial/tutorial_general_203.png

