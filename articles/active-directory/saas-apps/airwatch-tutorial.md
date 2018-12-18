---
title: 教學課程：Azure Active Directory 與 AirWatch 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 AirWatch 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.assetid: 96a3bb1c-96c6-40dc-8ea0-060b0c2a62e5
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/20/2017
ms.author: jeedes
ms.reviewer: jeedes
ms.openlocfilehash: f3bbcbb70759e7a995797cf89ad75a2a39314927
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39048475"
---
# <a name="tutorial-azure-active-directory-integration-with-airwatch"></a>教學課程：Azure Active Directory 與 AirWatch 整合

在本教學課程中，您將了解如何整合 AirWatch 與 Azure Active Directory (Azure AD)。

AirWatch 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 AirWatch 的人員
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 AirWatch (單一登入)
- 您可以在 Azure 入口網站中集中管理您的帳戶

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 AirWatch 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 AirWatch 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在 [這裡](https://azure.microsoft.com/pricing/free-trial/)取得一個月試用。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 AirWatch
2. 設定並測試 Azure AD 單一登入

## <a name="adding-airwatch-from-the-gallery"></a>從資源庫新增 AirWatch
若要設定 AirWatch 與 Azure AD 整合，您需要從資源庫將 AirWatch 新增至受控 SaaS 應用程式清單。

**若要從資源庫新增 AirWatch，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Active Directory][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![[應用程式]][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![[應用程式]][3]

4. 在搜尋方塊中，鍵入 **AirWatch**。

    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/tutorial_airwatch_search.png)

5. 在結果窗格中，選取 [AirWatch]，然後按一下 [新增] 按鈕以新增應用程式。

    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/tutorial_airwatch_addfromgallery.png)

##  <a name="configuring-and-testing-azure-ad-single-sign-on"></a>設定並測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 AirWatch 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 AirWatch 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 AirWatch 中的相關使用者之間建立連結關聯性。

建立此連結關聯性的方法，就是將 Azure AD 中**使用者名稱**的值指派為 AirWatch 中 **Username** 的值。

若要設定及測試與 AirWatch 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 AirWatch 測試使用者](#creating-a-airwatch-test-user)** - 在 AirWatch 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
4. **[指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[Testing Single Sign-On](#testing-single-sign-on)** - 驗證組態是否能運作。

### <a name="configuring-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 AirWatch 應用程式中設定單一登入。

**若要設定與 AirWatch 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [AirWatch] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![設定單一登入](./media/airwatch-tutorial/tutorial_airwatch_samlbase.png)

3. 在 [AirWatch 網域與 URL] 區段上，執行下列步驟：

    ![設定單一登入](./media/airwatch-tutorial/tutorial_airwatch_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<subdomain>.awmdm.com/AirWatch/Login?gid=companycode`

    b. 在 [識別碼] 文字方塊中，以 `AirWatch` 形式輸入值

    > [!NOTE] 
    > 這不是真正的值。 使用實際的「登入 URL」來更新此值。 請連絡 [AirWatch 用戶端支援小組](http://www.air-watch.com/company/contact-us/)以取得這些值。 
 
4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將 XML 檔案儲存在您的電腦上。

    ![設定單一登入](./media/airwatch-tutorial/tutorial_airwatch_certificate.png) 

5. 在 [AirWatch 設定] 區段上，按一下 [設定 AirWatch] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![設定單一登入](./media/airwatch-tutorial/tutorial_airwatch_configure.png) 

6. 按一下 [儲存]  按鈕。

    ![設定單一登入](./media/airwatch-tutorial/tutorial_general_400.png)
<CS>
7. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 AirWatch 公司網站。

8. 在左側導覽窗格中按一下 [帳戶]，然後按一下 [系統管理員]。
   
   ![系統管理員](./media/airwatch-tutorial/ic791920.png "系統管理員")

9. 展開 [設定] 功能表，然後按一下 [目錄服務]。
   
   ![設定](./media/airwatch-tutorial/ic791921.png "設定")

10. 按一下 [使用者] 索引標籤，在 [基準 DN] 文字方塊中輸入您的網域名稱，然後按一下 [儲存]。
   
   ![使用者](./media/airwatch-tutorial/ic791922.png "使用者")

11. 按一下 [伺服器]  索引標籤。
   
   ![伺服器](./media/airwatch-tutorial/ic791923.png "伺服器")

12. 執行下列步驟：
    
    ![上傳](./media/airwatch-tutorial/ic791924.png "上傳")   
    
    a. 針對 [目錄類型]，選取 [無]。

    b. 選取 [使用 SAML 進行驗證] 。

    c. 若要上傳已下載的憑證，請按一下 [上傳] 。

13. 在 [要求]  區段中，執行下列步驟：
    
    ![要求](./media/airwatch-tutorial/ic791925.png "要求")  

    a. 針對 [要求繫結類型]，選取 [POST]。

    b. 在 Azure 入口網站的 [設定在 Airwatch 單一登入] 對話頁面上，複製 [SAML 單一登入服務 URL] 值，然後將它貼到 [識別提供者單一登入 URL] 文字方塊中。

    c. 針對 [NameID 格式]，選取 [電子郵件地址]。

    d. 按一下 [檔案] 。

14. 再按一次 [使用者]  索引標籤。
    
    ![使用者](./media/airwatch-tutorial/ic791926.png "使用者")

15. 在 [屬性]  區段中，執行下列步驟：
    
    ![屬性](./media/airwatch-tutorial/ic791927.png "屬性")

    a. 在 [物件識別碼] 文字方塊中，輸入 **http://schemas.microsoft.com/identity/claims/objectidentifier**。

    b. 在 [使用者名稱] 文字方塊中，輸入 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress**。

    c. 在 [顯示名稱] 文字方塊中，輸入 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname**。

    d. 在 [名字] 文字方塊中，輸入 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname**。

    e. 在 [姓氏] 文字方塊中，輸入 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname**。

    f. 在 [電子郵件] 文字方塊中，輸入 **http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress**。

    g. 按一下 [檔案] 。

<CE>

### <a name="creating-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 **Azure 入口網站**的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/create_aaduser_01.png) 

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。
    
    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/create_aaduser_02.png) 

3. 若要開啟 [使用者] 對話方塊，按一下對話方塊頂端的 [新增]。
 
    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/create_aaduser_03.png) 

4. 在 [使用者]  對話頁面上，執行下列步驟：
 
    ![建立 Azure AD 測試使用者](./media/airwatch-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 文字方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 文字方塊中，輸入 Britta Simon 的「電子郵件地址」。

    c. 選取 [顯示密碼] 並記下 [密碼] 的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="creating-a-airwatch-test-user"></a>建立 AirWatch 測試使用者

若要讓 Azure AD 使用者能夠登入 AirWatch，必須將他們佈建到 AirWatch。

* AirWatch 需以手動方式佈建。

**若要佈建使用者帳戶，請執行下列步驟：**

1. 以系統管理員身分登入您的 **AirWatch** 公司網站。
2. 在左側導覽窗格中按一下 [帳戶]，然後按一下 [使用者]。
   
   ![使用者](./media/airwatch-tutorial/ic791929.png "使用者")
3. 在 [使用者] 功能表中，按一下 [清單檢視]，然後按一下 [新增] \> [新增使用者]。
   
   ![新增使用者](./media/airwatch-tutorial/ic791930.png "新增使用者")
4. 在 [新增/編輯使用者]  對話方塊中，執行下列步驟：

   ![新增使用者](./media/airwatch-tutorial/ic791931.png "新增使用者")   
   1. 在相關的文字方塊中，輸入您想要佈建之有效 Azure Active Directory 帳戶的 [使用者名稱]、[密碼]、[確認密碼]、[名字]、[姓氏]、[電子郵件地址]。
   2. 按一下 [檔案] 。

>[!NOTE]
>您可以使用任何其他的 AirWatch 使用者帳戶建立工具或 AirWatch 提供的 API 來佈建 AAD 使用者帳戶。
>  

### <a name="assigning-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 AirWatch 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者][200] 

**若要將 Britta Simon 指派給 AirWatch，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [AirWatch]。

    ![設定單一登入](./media/airwatch-tutorial/tutorial_airwatch_app.png) 

3. 在左側功能表中，按一下 [使用者和群組]。

    ![指派使用者][202] 

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![指派使用者][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="testing-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

如果要測試您的單一登入設定，請開啟存取面板。 如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。


## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/airwatch-tutorial/tutorial_general_01.png
[2]: ./media/airwatch-tutorial/tutorial_general_02.png
[3]: ./media/airwatch-tutorial/tutorial_general_03.png
[4]: ./media/airwatch-tutorial/tutorial_general_04.png

[100]: ./media/airwatch-tutorial/tutorial_general_100.png

[200]: ./media/airwatch-tutorial/tutorial_general_200.png
[201]: ./media/airwatch-tutorial/tutorial_general_201.png
[202]: ./media/airwatch-tutorial/tutorial_general_202.png
[203]: ./media/airwatch-tutorial/tutorial_general_203.png

