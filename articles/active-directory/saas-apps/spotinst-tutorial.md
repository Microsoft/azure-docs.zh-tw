---
title: 教學課程：Azure Active Directory 與 Spotinst 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Spotinst 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 2f6dbd70-c2db-4ae9-99ee-976c3090d214
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/25/2018
ms.author: jeedes
ms.openlocfilehash: 7edf1cbc5cc351e25a9ae7b319768376ea9968a3
ms.sourcegitcommit: a5eb246d79a462519775a9705ebf562f0444e4ec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39267047"
---
# <a name="tutorial-azure-active-directory-integration-with-spotinst"></a>教學課程：Azure Active Directory 與 Spotinst 整合

在本教學課程中，您將了解如何整合 Spotinst 與 Azure Active Directory (Azure AD)。

Spotinst 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Spotinst 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Spotinst (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定與 Spotinst 的 Azure AD 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Spotinst 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Spotinst
2. 設定並測試 Azure AD 單一登入

## <a name="adding-spotinst-from-the-gallery"></a>從資源庫新增 Spotinst
若要設定 Spotinst 與 Azure AD 整合，您需要從資源庫將 Spotinst 加入到受控 SaaS 應用程式清單。

**若要從資源庫新增 Spotinst，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Spotinst**，從結果面板中選取 [Spotinst]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Spotinst](./media/spotinst-tutorial/tutorial_spotinst_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Spotinst 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Spotinst 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Spotinst 中的相關使用者之間建立連結關聯性。

若要使用 Spotinst 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Spotinst 測試使用者](#create-a-spotinst-test-user)** - 在 Spotinst 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Spotinst 應用程式中設定單一登入。

**若要使用 Spotinst 設定 Azure AD 單一登入功能，請執行下列步驟：**

1. 在 Azure 入口網站的 [Spotinst] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/spotinst-tutorial/tutorial_spotinst_samlbase.png)

3. 如果您想要以 IDP 起始模式設定應用程式，請在 [Spotinst 網域和 URL] 區段上執行下列步驟：

    ![Spotinst 網域與 URL 單一登入資訊](./media/spotinst-tutorial/tutorial_spotinst_url1.png)

    a. 按一下 [顯示進階 URL 設定]。

    b. 在 [轉送狀態] 文字方塊中輸入下列值：`<ID>`

    c. 如果您想要以 **SP** 起始模式設定應用程式，請在 [登入 URL] 文字方塊中輸入下列 URL：`https://console.spotinst.com`

    > [!NOTE]
    > 轉送狀態的值不是真正的值。 您將會使用實際的「回覆狀態」值來更新 [回覆狀態] 值，本教學課程稍後會說明。

4. Spotinst 應用程式需要特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中，管理這些屬性的值。 以下螢幕擷取畫面顯示上述情況的範例。

    ![設定單一登入](./media/spotinst-tutorial/tutorial_Spotinst_attribute.png)

5. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱 | 屬性值 |
    | ---------------| --------------- |
    | 電子郵件 | user.mail |
    | 名字 | user.givenname |
    | 姓氏 | user.surname |
    
    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入](./media/spotinst-tutorial/tutorial_attribute_04.png)

    ![設定單一登入](./media/spotinst-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 讓 [命名空間] 保持空白。

    e. 按一下 [確定]。

6. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/spotinst-tutorial/tutorial_spotinst_certificate.png) 

7. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/spotinst-tutorial/tutorial_general_400.png)

8. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Spotinst。

9. 按一下畫面右上方的 [使用者圖示]，然後按一下 [設定]。

    ![Spotinst 設定](./media/spotinst-tutorial/tutorial_spotinst_settings.png)

10. 按一下頂端的 [安全性] 索引標籤，然後選取 [識別提供者] 並執行下列步驟：

    ![Spotinst 安全性](./media/spotinst-tutorial/tutorial_spotinst_security.png)

    a. 複製執行個體的 [轉送狀態] 值，並將其貼到 Azure 入口網站 [Spotinst 網域和 URL] 區段的 [轉送狀態] 文字方塊中。

    b. 按一下 [瀏覽]，上傳您從 Azure 入口網站下載的中繼資料 xml 檔案

    c. 按一下 [儲存] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/spotinst-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/spotinst-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/spotinst-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/spotinst-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-spotinst-test-user"></a>建立 Spotinst 測試使用者

本節目標是在 Spotinst 中建立名為 Britta Simon 的使用者。

1. 如果您已經以 **SP** 起始模式設定應用程式，請執行下列步驟：

   a. 在不同的網頁瀏覽器視窗中，以安全性系統管理員身分登入 Spotinst。

   b. 按一下畫面右上方的 [使用者圖示]，然後按一下 [設定]。

    ![Spotinst 設定](./media/spotinst-tutorial/tutorial_spotinst_settings.png)

    c. 按一下 [使用者]，然後選取 [新增使用者]。

    ![Spotinst 設定](./media/spotinst-tutorial/adduser1.png)

    d. 在 [新增使用者] 區段中，執行下列步驟：

    ![Spotinst 設定](./media/spotinst-tutorial/adduser2.png)

    * 在 [全名] 文字方塊中，輸入使用者的全名，例如 **BrittaSimon**。

    * 在 [電子郵件] 文字方塊中，輸入使用者的電子郵件地址，例如 **brittasimon@contoso.com**。

    * 針對 [組織角色、帳戶角色和帳戶] 選取組織特定詳細資料。

2. 如果您已經以 **IDP** 起始模式設定應用程式，則這個區段沒有適用的動作項目。 Spotinst 支援預設啟用的 Just-In-Time 佈建。 當您嘗試存取 Spotinst 時，如果 Spotinst 還沒有使用者，它將會建立新的使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Spotinst 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Spotinst，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Spotinst]。

    ![應用程式清單中的 Spotinst 連結](./media/spotinst-tutorial/tutorial_spotinst_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Spotinst] 圖格時，應該會自動登入您的 Spotinst 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/spotinst-tutorial/tutorial_general_01.png
[2]: ./media/spotinst-tutorial/tutorial_general_02.png
[3]: ./media/spotinst-tutorial/tutorial_general_03.png
[4]: ./media/spotinst-tutorial/tutorial_general_04.png

[100]: ./media/spotinst-tutorial/tutorial_general_100.png

[200]: ./media/spotinst-tutorial/tutorial_general_200.png
[201]: ./media/spotinst-tutorial/tutorial_general_201.png
[202]: ./media/spotinst-tutorial/tutorial_general_202.png
[203]: ./media/spotinst-tutorial/tutorial_general_203.png

