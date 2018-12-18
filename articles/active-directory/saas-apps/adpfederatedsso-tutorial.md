---
title: 教學課程：Azure Active Directory 與 ADP 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 ADP 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 7be5331b-0481-48f7-9d6b-619dfec657e1
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/29/2018
ms.author: jeedes
ms.openlocfilehash: 75b84c2856373126ceba0fc536e41d270f4d2d05
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39048774"
---
# <a name="tutorial-azure-active-directory-integration-with-adp"></a>教學課程：Azure Active Directory 與 ADP

在本教學課程中，您會了解如何將 ADP 與 Azure Active Directory (Azure AD) 整合。

整合 ADP 與 Azure AD 提供下列優點：

- 您可以在 Azure AD 中控制可存取 ADP 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 ADP (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 ADP 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 ADP 的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 ADP
2. 設定並測試 Azure AD 單一登入

## <a name="adding-adp-from-the-gallery"></a>從資源庫新增 ADP
若要設定將 ADP 整合到 Azure AD 中，您需要從資源庫將 ADP 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 ADP，請執行下列步驟：**

1.  以系統管理員身分登入您的 Microsoft Azure 身分識別提供者環境。

2. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

3. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
4. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

5. 在搜尋方塊中，輸入 **ADP**，從結果面板中選取 [ADP]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 ADP](./media/adpfederatedsso-tutorial/tutorial_adp_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 ADP 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 ADP 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 ADP 中相關使用者之間的連結關聯性。

在 ADP 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 ADP 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 ADP 測試使用者](#create-an-adp-test-user)** - 在 ADP 中建立一個與 Azure AD 中代表 Britta Simon 之項目連結的 Britta Simon 對應項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 ADP 應用程式中設定單一登入。

**若要設定與 ADP 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 **ADP** 應用程式整合頁面上，按一下 [屬性] 索引標籤，然後執行下列步驟： 

    ![單一登入屬性](./media/adpfederatedsso-tutorial/tutorial_adp_prop.png)

    a. 將 [為使用者啟用登入] 欄位值設定為 [是]。

    b. 複製 [使用者存取 URL]，而且您必須將其貼到 [設定登入 URL] 區段中，本教學課程稍後會說明。

    c. 將 [需要使用者指派] 欄位值設定為 [是]。

    d. 將 [是否要向使用者顯示] 欄位值設定為 [否]。

2. 按一下 **ADP** 應用程式整合頁面上的 [單一登入]。

    ![設定單一登入連結][4]

3. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/adpfederatedsso-tutorial/tutorial_adp_samlbase.png)

4. 在 [ADP 網域及 URL] 區段上，執行下列步驟：

    ![ADP 網域與 URL 單一登入資訊](./media/adpfederatedsso-tutorial/tutorial_adp_url.png)

    在 [識別碼] 文字方塊中，輸入 URL：`https://fed.adp.com` 
    
5. ADP 應用程式需要特定格式的 SAML 判斷提示，其要求您將自訂屬性對應新增至您的 SAML 權杖屬性組態。 以下螢幕擷取畫面顯示上述的範例。 宣告名稱將一律為 **"PersonImmutableID"**，且其值已對應至 **employeeid**。 

    在這裡，會透過 **employeeid** 完成從 Azure AD 到 ADP 的使用者對應，但您也可以根據應用程式設定，將其對應到不同的值。 因此，請先與 [ADP 支援小組](https://www.adp.com/contact-us/overview.aspx)合作以使用正確的使用者識別碼，然後將該值與 **"PersonImmutableID"** 宣告對應。

    ![設定單一登入](./media/adpfederatedsso-tutorial/tutorial_adp_attribute.png)

6. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | ------------------- | -------------------- |    
    | PersonImmutableID | user.employeeid |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/adpfederatedsso-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/adpfederatedsso-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定] 。

    > [!NOTE] 
    > 您必須先連絡 [ADP 支援小組](https://www.adp.com/contact-us/overview.aspx)，向其要求租用戶的唯一識別碼屬性值，才能設定 SAML 判斷提示。 您需要此值來設定應用程式的自訂宣告。 

7. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/adpfederatedsso-tutorial/tutorial_adp_certificate.png) 

8. 若要在 **ADP** 端設定單一登入，您必須將已下載的**中繼資料 XML** 上傳至 [ADP 網站](https://adpfedsso.adp.com/public/login/index.fcc)。

> [!NOTE]  
> 此流程可能需要幾天的時間。 

### <a name="configure-your-adp-services-for-federated-access"></a>設定 ADP 服務以進行同盟存取

>[!Important]
> 您的員工如果需要對 ADP 服務進行同盟存取，您就必須將其指派給 ADP 服務應用程式，接著再將其重新指派給特定的 ADP 服務。
從您的 ADP 代表收到確認之後，請設定您的 ADP 服務並指派/管理使用者，以控制使用者對特定 ADP 服務的存取權。

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **ADP**，從結果面板中選取 [ADP]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 ADP](./media/adpfederatedsso-tutorial/tutorial_adp_addservicegallery.png)

5. 在 Azure 入口網站的 **ADP** 應用程式整合頁面上，按一下 [屬性] 索引標籤，然後執行下列步驟：  

    ![單一登入已連結的屬性](./media/adpfederatedsso-tutorial/tutorial_adp_linkedproperties.png)

    a.  將 [為使用者啟用登入] 欄位值設定為 [是]。

    b.  將 [需要使用者指派] 欄位值設定為 [是]。

    c.  將 [是否要向使用者顯示] 欄位值設定為 [是]。

6. 按一下 **ADP** 應用程式整合頁面上的 [單一登入]。

    ![設定單一登入連結][4]

7. 在 [單一登入] 對話方塊中，選取 [連結的登入] 作為 [模式]， 以將您的應用程式連結至 **ADP**。

    ![已連結的單一登入](./media/adpfederatedsso-tutorial/tutorial_adp_linked.png)

8. 瀏覽至 [設定登入 URL] 區段，執行下列步驟︰

    ![單一登入屬性](./media/adpfederatedsso-tutorial/tutorial_adp_linkedsignon.png)
                                                              
    a. 貼上您從上面 [屬性] 索引標籤 (來自主要 ADP 應用程式) 複製的 [使用者存取 URL]。
                                                             
    b. 以下是 5 個支援不同**轉送狀態 URL** 的應用程式。 您必須手動將特定應用程式的適當**轉送狀態 URL** 值附加至 [使用者存取 URL]。
    
    * **ADP Workforce Now**
        
        `<User access URL>?relaystate=https://fed.adp.com/saml/fedlanding.html?WFN`

    * **ADP Workforce Now Enhanced Time**
        
        `<User access URL>?relaystate=https://fed.adp.com/saml/fedlanding.html?EETDC2`
    
    * **ADP Vantage HCM**
        
        `<User access URL>?relaystate=https://fed.adp.com/saml/fedlanding.html?ADPVANTAGE`

    * **ADP Enterprise HR**

        `<User access URL>?relaystate=https://fed.adp.com/saml/fedlanding.html?PORTAL`

    * **MyADP**

        `<User access URL>?relaystate=https://fed.adp.com/saml/fedlanding.html?REDBOX`

9. **儲存** 您的變更。

10. 從您的 ADP 代表收到確認之後，使用一或兩個使用者開始進行測試。

    a. 將幾個使用者指派給 ADP 服務應用程式，以測試同盟存取。

    b. 當使用者存取資源庫上的 ADP 服務應用程式，且能夠存取其 ADP 服務時，即表示測試成功。
 
11. 確認測試成功時，將同盟 ADP 服務指派給個別的使用者或使用者群組 (本教學課程稍後會說明)，然後向您的員工首度發行。 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/adpfederatedsso-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/adpfederatedsso-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/adpfederatedsso-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/adpfederatedsso-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-an-adp-test-user"></a>建立 ADP 測試使用者

本節目標是在 ADP 中建立名為 Britta Simon 的使用者。 請與 [ADP 支援小組](https://www.adp.com/contact-us/overview.aspx)合作，在 ADP 帳戶中新增使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 ADP 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 ADP，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [ADP]。

    ![應用程式清單中的 ADP 連結](./media/adpfederatedsso-tutorial/tutorial_adp_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 ADP 圖格時，應該會自動登入您的 ADP 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/adpfederatedsso-tutorial/tutorial_general_01.png
[2]: ./media/adpfederatedsso-tutorial/tutorial_general_02.png
[3]: ./media/adpfederatedsso-tutorial/tutorial_general_03.png
[4]: ./media/adpfederatedsso-tutorial/tutorial_general_04.png

[100]: ./media/adpfederatedsso-tutorial/tutorial_general_100.png

[200]: ./media/adpfederatedsso-tutorial/tutorial_general_200.png
[201]: ./media/adpfederatedsso-tutorial/tutorial_general_201.png
[202]: ./media/adpfederatedsso-tutorial/tutorial_general_202.png
[203]: ./media/adpfederatedsso-tutorial/tutorial_general_203.png

