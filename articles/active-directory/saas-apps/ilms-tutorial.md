---
title: 教學課程：Azure Active Directory 與 iLMS 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 iLMS 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: d6e11639-6cea-48c9-b008-246cf686e726
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/13/2017
ms.author: jeedes
ms.openlocfilehash: 0e67e97a68ca333dff366dd5e0222c96a1022557
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39425267"
---
# <a name="tutorial-azure-active-directory-integration-with-ilms"></a>教學課程：Azure Active Directory 與 iLMS 整合

在本教學課程中，您會了解如何將 iLMS 與 Azure Active Directory (Azure AD) 進行整合。

iLMS 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制誰有 iLMS 的存取權
- 您可以讓使用者使用其 Azure AD 帳戶自動登入 iLMS (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 iLMS 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 iLMS 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則您不應使用生產環境，。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 iLMS
1. 設定並測試 Azure AD 單一登入

## <a name="adding-ilms-from-the-gallery"></a>從資源庫新增 iLMS
若要設定將 iLMS 整合到 Azure AD 中，您需要從資源庫將 iLMS 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 iLMS，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
1. 若要新增新的應用程式，按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

1. 在搜尋方塊中，輸入 **iLMS**。

    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/tutorial_ilms_search.png)

1. 在結果窗格中，選取 [iLMS]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/tutorial_ilms_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 iLMS 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 iLMS 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 iLMS 中的相關使用者之間建立連結關聯性。

建立此連結關聯性的方法，是將 Azure AD 中**使用者名稱**的值，指派為 iLMS 中 **Username** 的值。

若要設定及測試與 iLMS 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 iLMS 測試使用者](#creating-an-ilms-test-user)** - 使 iLMS 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 iLMS 應用程式中設定單一登入。

**若要設定與 iLMS 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [iLMS] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_samlbase.png)

1. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [iLMS 網域和 URL] 區段上執行下列步驟：

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_url.png)

    a. 在 [識別碼] 文字方塊中，將您從 iLMS 系統管理入口網站中 SAML 設定的 [服務提供者] 區段複製的 [識別碼] 值貼上。

    b. 在 [回覆 URL] 文字方塊中，將您從 iLMS 系統管理入口網站中 SAML 設定的 [服務提供者] 區段複製的 [端點 (URL)] 值貼上，並包含下列模式：`https://www.inspiredlms.com/Login/<instanceName>/consumer.aspx`

    >[!Note]
    >這個 '123456' 是識別項的範例值。

1. 如果您想要以 **SP** 起始模式設定應用程式，請勾選 [顯示進階 URL 設定]：

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_url1.png)

    在 [登入 URL] 文字方塊中，將您從 iLMS 系統管理入口網站中 SAML 設定的 [服務提供者] 區段複製的 [端點 (URL)] 值貼上，如同：`https://www.inspiredlms.com/Login/<instanceName>/consumer.aspx`     

1. 若要啟用 JIT 佈建，iLMS 應用程式會預期要有特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定單一登入](./media/ilms-tutorial/4.png)
    
    建立 **Department, Region** 和 **Division** 屬性，並在 iLMS 中新增這些屬性的名稱。 上述的所有屬性都是必要屬性。  

    > [!NOTE] 
    > 您必須啟用 iLMS 中的 [建立無法辨識的使用者帳戶] 來對應這些屬性。 請依照[這裡](http://support.inspiredelearning.com/customer/portal/articles/2204526)的指示，了解屬性的設定。

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | ---------------| --------------- |    
    | division | user.department |
    | region | user.state |
    | department | user.jobtitle |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_04.png)

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定]。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/ilms-tutorial/tutorial_general_400.png)

1. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 **iLMS 系統管理入口網站** 。

1. 按一下 [設定] 索引標籤下的 [SSO:SAML]，開啟 SAML 設定並執行下列步驟︰
    
    ![設定單一登入](./media/ilms-tutorial/1.png) 

    a. 展開 [服務提供者] 區段，然後複製**識別碼**和**端點 (URL)** 值。

    ![設定單一登入](./media/ilms-tutorial/2.png) 

    b. 在 [識別提供者] 區段中，按一下 [匯入中繼資料]。
    
    c. 在 [SAML 簽章憑證] 區段中，選取從 Azure 入口網站下載的 [中繼資料] 檔案。

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_ssoconfig1.png) 

    d. 如果您想要啟用 JIT 佈建來建立無法辨識之使用者的 iLMS 帳戶，請遵循以下步驟︰
        
       - 請勾選 [建立無法辨識的使用者帳戶]。
       
       ![Configure Single Sign-On](./media/ilms-tutorial/tutorial_ilms_ssoconfig2.png)

       -  將 Azure AD 中的屬性與 iLMS 中的屬性進行對應。 在屬性資料行中，指定屬性名稱或預設值。

    e. 移至 [商務規則] 索引標籤，然後執行下列步驟︰ 
        
       ![Configure Single Sign-On](./media/ilms-tutorial/5.png)

       - 請勾選 [建立無法辨識的區域、事業處和部門]，來建立單一登入時尚未存在的區域、事業處和部門。
        
       - 請勾選 [在登入期間更新使用者設定檔]，來指定使用者的設定檔是否隨著每個單一登入而更新。 
        
       - 如果勾選 [更新使用者設定檔中非必要欄位的空白值] 選項，則登入時為空白的選擇性設定檔欄位也將會造成使用者的 iLMS 設定檔包含這些欄位的空白值。
        
       - 請勾選 [傳送錯誤通知電子郵件]，並輸入您要接收錯誤通知電子郵件之使用者的電子郵件。

1. 按一下 [儲存] 按鈕以儲存設定。

    ![設定單一登入](./media/ilms-tutorial/save.png)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
    
### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/create_aaduser_01.png) 

1. 移至 [使用者和群組]，然後按一下 [所有使用者] 以顯示使用者清單。
    
    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/create_aaduser_02.png) 

1. 在對話方塊的頂端，按一下 [新增] 以開啟 [使用者] 對話方塊。
 
    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/create_aaduser_03.png) 

1. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/ilms-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-an-ilms-test-user"></a>建立 iLMS 測試使用者

應用程式支援及時 (Just In Time) 使用者佈建，而在驗證之後，則會在應用程式中自動建立使用者。 如果在 iLMS 系統管理入口網站設定 SAML 組態期間，您已按下 [建立無法辨識的使用者帳戶] 核取方塊，JIT 就會正常運作。

如果您需要以手動方式建立使用者，請依照下列步驟進行︰

1. 以系統管理員身分登入您的 iLMS 公司網站。

1. 按一下[使用者] 索引標籤下的 [註冊使用者]開啟 [註冊使用者] 頁面。 
   
   ![新增員工](./media/ilms-tutorial/3.png)

1. 在 [註冊使用者] 頁面上，執行下列步驟。

    ![新增員工](./media/ilms-tutorial/create_testuser_add.png)

    a. 在 [名字] 文字方塊中輸入 Britta 這個名字。
   
    b. 在 [姓氏] 文字方塊中輸入 Simon 這個姓氏。

    c. 在 [電子郵件識別碼] 文字方塊中，輸入 Britta Simon 帳戶的電子郵件地址。

    d. 在 [區域] 下拉式清單中，選取區域的值。

    e. 在 [事業處] 下拉式清單中，選取事業處的值。

    f. 在 [部門] 下拉式清單中，選取部門的值。

    g. 按一下 [檔案] 。

    > [!NOTE] 
    > 您也可以選取 [傳送註冊郵件] 核取方塊，將註冊郵件傳送給使用者。

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 iLMS 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 iLMS，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [iLMS]。

    ![設定單一登入](./media/ilms-tutorial/tutorial_ilms_app.png) 

1. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [iLMS] 圖格時，系統應該會將您自動登入 iLMS 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/ilms-tutorial/tutorial_general_01.png
[2]: ./media/ilms-tutorial/tutorial_general_02.png
[3]: ./media/ilms-tutorial/tutorial_general_03.png
[4]: ./media/ilms-tutorial/tutorial_general_04.png

[100]: ./media/ilms-tutorial/tutorial_general_100.png

[200]: ./media/ilms-tutorial/tutorial_general_200.png
[201]: ./media/ilms-tutorial/tutorial_general_201.png
[202]: ./media/ilms-tutorial/tutorial_general_202.png
[203]: ./media/ilms-tutorial/tutorial_general_203.png

