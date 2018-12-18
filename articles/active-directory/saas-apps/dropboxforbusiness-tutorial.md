---
title: 教學課程：Azure Active Directory 與 Dropbox for Business 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Dropbox for Business 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 63502412-758b-4b46-a580-0e8e130791a1
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/20/2018
ms.author: jeedes
ms.openlocfilehash: eadf6724891d348c2ea3654bcf19ef0d74078049
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42145561"
---
# <a name="tutorial-azure-active-directory-integration-with-dropbox-for-business"></a>教學課程：Azure Active Directory 與 Dropbox for Business 整合

在本教學課程中，您會了解如何整合 Dropbox for Business 與 Azure Active Directory (Azure AD)。

Dropbox for Business 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取商務用 Dropbox 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Dropbox for Business (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Dropbox for Business 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Dropbox for Business 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Dropbox for Business
2. 設定並測試 Azure AD 單一登入

## <a name="adding-dropbox-for-business-from-the-gallery"></a>從資源庫新增 Dropbox for Business

若要設定 Dropbox for Business 與 Azure AD 整合，您需要從資源庫將 SAP Dropbox for Business 新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 Dropbox for Business，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 [商務用 Dropbox]，從結果面板中選取 [商務用 Dropbox]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的商務用 Dropbox](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Dropbox for Business 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Dropbox for Business 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者與 Dropbox for Business 中的相關使用者之間建立連結關聯性。

在商務用 Dropbox 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Dropbox for Business 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立商務用 Dropbox 測試使用者](#create-a-dropbox-for-business-test-user)** - 在商務用 Dropbox 中建立 Britta Simon 的對應項目，且該項目與 Azure AD 中代表使用者的項目連結。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您將在 Azure 入口網站中啟用 Azure AD 單一登入，並在 Dropbox for Business 應用程式中設定單一登入。

**若要使用 Dropbox for Business 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Dropbox for Business] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_samlbase.png)

3. 在 [Dropbox for Business 網域與 URL] 區段上，執行下列步驟：

    ![商務用 Dropbox 網域及 URL 單一登入資訊](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_url1.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://www.dropbox.com/sso/<id>`

    b. 在 [識別碼] 文字方塊中，輸入一個值：`Dropbox`

    > [!NOTE]
    > 上述登入 URL 值並非真正的值。 您將會使用實際的登入 URL 來更新值，稍後會在本教學課程中說明。

4. 在 [SAML 簽署憑證] 區段上，按一下 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_certificate.png) 

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/dropboxforbusiness-tutorial/tutorial_general_400.png)

6. 在 [Dropbox for Business 設定] 區段中，按一下 [設定 Dropbox for Business] 可開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 單一登入服務 URL]。

    ![商務用 Dropbox 組態](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_configure.png) 

7. 若要在 [商務用 Dropbox] 端上設定單一登入，請移至商務用 Dropbox 租用戶，並登入您的商務用 Dropbox 租用戶。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/ic769509.png "設定單一登入")

8. 按一下 [使用者圖示]，然後選取 [設定] 索引標籤。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure1.png "設定單一登入")

9. 在左側的導覽窗格中，按一下 [管理主控台]。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure2.png "設定單一登入")

10. 在 [管理主控台] 上，按一下左側導覽窗格中的 [設定]。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure3.png "設定單一登入")

11. 選取 [驗證]區段下的 [單一登入] 選項。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure4.png "設定單一登入")

12. 在 [單一登入]  區段中，執行下列步驟：  

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure5.png "設定單一登入")

    a. 在 [單一登入]的下拉式清單中選取 [必要] 選項。

    b. 在 [識別提供者登入 URL] 文字方塊中，按一下 [新增登入 URL]，並貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值，然後選取 [完成]。

    ![設定單一登入](./media/dropboxforbusiness-tutorial/configure6.png "設定單一登入")

    c. 按一下 [上傳憑證]，然後瀏覽至您從 Azure 入口網站下載的 **Base64 編碼憑證檔案**。

    d. 按一下 [複製連結]，並在 Azure 入口網站中，將複製的值貼到 [商務用 Dropbox 網域和 URL] 區段的 [登入 URL] 文字方塊。

    e. 按一下 [檔案] 。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/dropboxforbusiness-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/dropboxforbusiness-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/dropboxforbusiness-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/dropboxforbusiness-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-dropbox-for-business-test-user"></a>建立商務用 Dropbox 測試使用者

本節會在 Dropbox for Business 中建立名為 Britta Simon 的使用者。 Dropbox for Business 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作項目。 如果 Dropbox for Business 中還沒有該使用者，當您嘗試存取 Dropbox for Business 時，就會建立新的使用者。

>[!Note]
>如果您需要手動建立使用者，請連絡 [Dropbox for Business 用戶端支援小組](https://www.dropbox.com/business/contact) 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Dropbox for Business 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Dropbox for Business，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Dropbox for Business]。

    ![應用程式清單中的商務用 Dropbox](./media/dropboxforbusiness-tutorial/tutorial_dropboxforbusiness_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您按一下「存取面板」中的 [Dropbox for Business] 圖格時，應該會取得您 Dropbox for Business 應用程式的登入頁面。
 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/dropboxforbusiness-tutorial/tutorial_general_01.png
[2]: ./media/dropboxforbusiness-tutorial/tutorial_general_02.png
[3]: ./media/dropboxforbusiness-tutorial/tutorial_general_03.png
[4]: ./media/dropboxforbusiness-tutorial/tutorial_general_04.png

[100]: ./media/dropboxforbusiness-tutorial/tutorial_general_100.png

[200]: ./media/dropboxforbusiness-tutorial/tutorial_general_200.png
[201]: ./media/dropboxforbusiness-tutorial/tutorial_general_201.png
[202]: ./media/dropboxforbusiness-tutorial/tutorial_general_202.png
[203]: ./media/dropboxforbusiness-tutorial/tutorial_general_203.png

