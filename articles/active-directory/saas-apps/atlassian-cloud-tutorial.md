---
title: 教學課程：Azure Active Directory 與 Atlassian Cloud 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Atlassian Cloud 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 729b8eb6-efc4-47fb-9f34-8998ca2c9545
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/16/2018
ms.author: jeedes
ms.openlocfilehash: 68613b8613a2e5a9139b83eb23e66884659efc47
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39114929"
---
# <a name="tutorial-azure-active-directory-integration-with-atlassian-cloud"></a>教學課程：Azure Active Directory 與 Atlassian Cloud 整合

在本教學課程中，您會了解如何整合 Atlassian Cloud 與 Azure Active Directory (Azure AD)。

Atlassian Cloud 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Atlassian Cloud 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Atlassian Cloud (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如需有關軟體即服務 (SaaS) 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Atlassian Cloud 整合，您需要下列項目：

- Azure AD 訂用帳戶。
- 若要針對 Atlassian Cloud 產品啟用安全性聲明標記語言 (SAML) 單一登入，您必須設定 Atlassian Access。 深入了解 [Atlassian Access]( https://www.atlassian.com/enterprise/cloud/identity-manager) \(英文\)。

> [!NOTE]
> 當您測試本教學課程中的步驟時，建議您不要使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
教學課程中說明的情節由二個主要建置組塊組成：

* 從資源庫新增 Atlassian Cloud
* 設定並測試 Azure AD 單一登入

## <a name="add-atlassian-cloud-from-the-gallery"></a>從資源庫新增 Atlassian Cloud
若要設定 Atlassian Cloud 與 Azure AD 的整合，請執行下列操作，將 Atlassian Cloud 從資源庫新增到受控 SaaS 應用程式清單：

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕][1]

2. 選取 [企業應用程式] > [所有應用程式]。

    ![[企業應用程式] 窗格][2]
    
3. 若要新增應用程式，請選取 [新增應用程式]。

    ![[新增應用程式] 按鈕][3]

4. 在搜尋方塊中，輸入 **Atlassian Cloud**，從結果清單中選取 [Atlassian Cloud]，然後選取 [新增]。

    ![結果清單中的 [Atlassian Cloud]](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 *Britta Simon* 的測試使用者為基礎，設定及測試與 Atlassian Cloud 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須識別 Atlassian Cloud 使用者及其在 Azure AD 中對應的使用者。 換句話說，您必須在 Azure AD 使用者與 Atlassian Cloud 中的相關使用者之間建立連結關聯性。

若要建立此連結關聯性，請將指派給 Azure AD「使用者名稱」的相同值，指派為 Atlassian Cloud [Username] \(使用者名稱\) 值。

若要設定及測試與 Atlassian Cloud 搭配運作的 Azure AD 單一登入，您需要完成下列各節中的構成要素。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Atlassian Cloud 應用程式中設定單一登入。

若要設定與 Atlassian Cloud 搭配運作的 Azure AD 單一登入，請執行下列操作：

1. 在 Azure 入口網站的 [Atlassian Cloud] 應用程式整合窗格中，選取 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 視窗的 [單一登入模式] 方塊中，選取 [SAML 登入]。

    ![[單一登入] 視窗](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_samlbase.png)

3. 若要以 **IDP 起始**模式設定應用程式，請在 [Atlassian Cloud 網域及 URL] 底下，執行下列操作：

    ![Atlassian Cloud 網域及 URL 單一登入資訊](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_url.png)
    
    a. 在 [識別碼] 方塊中，以下列模式輸入 URL：`https://auth.atlassian.com/saml/<unique ID>`。
    
    b. 在 [回覆 URL] 方塊中，以下列模式輸入 URL：`https://auth.atlassian.com/login/callback?connection=saml-<unique ID>`。

    c. 按一下 [顯示進階 URL 設定]。

    d. 在 [轉送狀態] 方塊中，以下列模式輸入 URL︰`https://<instancename>.atlassian.net`。

    > [!NOTE]
    > 上述值並非真正的值。 請使用實際的識別碼和回覆 URL 來更新這些值。 您將可從本教學課程稍後說明的 [Atlassian Cloud SAML 設定] 畫面取得這些真正的值。

4. 若要以 SP 起始模式設定應用程式，請選取 [顯示進階 URL 設定]，然後在 [登入 URL] 方塊中，以下列模式輸入 URL：`https://<instancename>.atlassian.net`。

    ![Atlassian Cloud 網域及 URL 單一登入資訊](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_url1.png)

    > [!NOTE]
    > 上述登入 URL 值並非真正的值。 使用實際的登入 URL 來更新此值。 請連絡 [Atlassian Cloud 用戶端支援小組](https://support.atlassian.com/)以取得此值。

5. 在 [SAML 簽署憑證] 底下，選取 [憑證 (Base64)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_certificate.png)

6. 您的 Atlassian Cloud 應用程式會預期要有特定格式的 SAML 判斷提示，這需要您將自訂屬性對應新增到您的「SAML 權杖屬性」設定。 

    [使用者識別碼] 值預設會與 user.userprincipalname 對應。 請將此值變更為與 user.mail 對應。 您也可以根據組織的設定，選擇任何其他適當的值，但在大多數情況下，電子郵件應該能夠運作。

    ![憑證下載連結](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_attribute.png)

7. 選取 [ **儲存**]。

    ![[設定單一登入] 的 [儲存] 按鈕](./media/atlassian-cloud-tutorial/tutorial_general_400.png)

8. 若要開啟 [設定單一登入] 視窗，請在 [Atlassian Cloud 設定] 區段中，選取 [設定 Atlassian Cloud]。

9. 在 [快速參考] 區段中，複製 [SAML 實體識別碼] 和 [SAML 單一登入服務 URL]。

    ![Atlassian Cloud 設定](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_configure.png)

10. 若要針對您的應用程式設定 SSO，請使用系統管理員認證來登入 Atlassian 入口網站。

11. 您必須先驗證您的網域再繼續設定單一登入。 如需詳細資訊，請參閱 [Atlassian 網域驗證](https://confluence.atlassian.com/cloud/domain-verification-873871234.html)文件。

12. 在左側窗格中，選取 [SAML single sign-on] \(SAML 單一登入\) 。 如果您尚未這樣做，請訂閱 Atlassian Identity Manager。

    ![設定單一登入](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_11.png)

13. 在 [Add SAML configuration] \(新增 SAML 設定\) 視窗中，執行下列操作：

    ![設定單一登入](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_12.png)

    a. 在 [Identity provider Entity ID] \(身分識別提供者實體識別碼\) 方塊中，貼上您從 Azure 入口網站複製的 SAML 實體識別碼。

    b. 在 [Identity provider SSO URL] \(身分識別提供者 SSO URL\) 方塊中，貼上您從 Azure 入口網站複製的 SAML 單一登入服務 URL。

    c. 開啟從 Azure 入口網站下載的憑證 (.txt 檔案)，複製值 (不含 *Begin Certificate* 和 *End Certificate* 行)，然後將它貼到 [Public X509 certificate] \(公用 X509 憑證\) 方塊中。
    
    d. 選取 [Save Configuration] \(儲存設定\)。
     
14. 若要確定您已設定正確的 URL，請執行下列操作來更新 Azure AD 設定：

    ![設定單一登入](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_13.png)

    a. 在 SAML 視窗中，複製 [SP Identity ID] \(SP 身分識別識別碼\)，然後貼到 Azure 入口網站中 [Atlassian Cloud 網域及 URL] 底下的 [識別碼] 方塊中。
    
    b. 在 SAML 視窗中，複製 [SP Assertion Consumer Service URL] \(SP 判斷提示取用者服務 URL\)，然後貼到 Azure 入口網站中 [Atlassian Cloud 網域及 URL] 底下的 [回覆 URL] 方塊中。 登入 URL 是您 Atlassian Cloud 的租用戶 URL。

    > [!NOTE]
    > 如果您是現有的客戶，在更新 Azure 入口網站中的 [SP Identity ID] \(SP 身分識別識別碼\) 和 [SP Assertion Consumer Service URL] \(SP 判斷提示取用者服務 URL\) 值之後，請選取 [Yes, update configuration] \(是，更新設定\)。 如果您是新客戶，則可以略過這個步驟。
    
15. 在 Azure 入口網站中，選取 [儲存]。

    ![設定單一登入](./media/atlassian-cloud-tutorial/tutorial_general_400.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

在本節中，您會執行下列步驟，在 Azure 入口網站中建立測試使用者 Britta Simon：

   ![建立 Azure AD 測試使用者][100]

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/atlassian-cloud-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請選取 [使用者和群組] > [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/atlassian-cloud-tutorial/create_aaduser_02.png)

3. 在 [所有使用者] 視窗中，選取 [新增]。

    ![[新增] 按鈕](./media/atlassian-cloud-tutorial/create_aaduser_03.png)

4. 在 [使用者] 視窗中，執行下列操作：

    ![[使用者] 視窗](./media/atlassian-cloud-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。

### <a name="create-an-atlassian-cloud-test-user"></a>建立 Atlassian Cloud 測試使用者

若要讓 Azure AD 使用者能夠登入 Atlassian Cloud，請執行下列操作，在 Atlassian Cloud 中手動佈建使用者帳戶：

1. 在 [Administration] \(管理\) 窗格中，選取 [Users] \(使用者\)。

    ![Atlassian Cloud [Users] \(使用者\) 連結](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_14.png)

2. 若要在 Atlassian Cloud 中建立使用者，請選取 [Invite user] \(邀請使用者\)。

    ![建立 Atlassian Cloud 使用者](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_15.png)

3. 在 [Email address] \(電子郵件地址\) 方塊中，輸入使用者的電子郵件地址，然後指派應用程式存取權。

    ![建立 Atlassian Cloud 使用者](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_16.png)

4. 若要傳送電子郵件邀請給使用者，請選取 [Invite users] \(邀請使用者\)。 電子郵件邀請會傳送給使用者，使用者在接受邀請之後，就會在系統中變成作用中使用者。

>[!NOTE]
>您也可以在 [Users] \(使用者\) 區段中選取 [Bulk Create] \(大量建立\) 按鈕來大量建立使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Atlassian Cloud 的存取權授與使用者 Britta Simon，讓她能夠使用 Azure 單一登入。 若要這樣做，請執行下列動作：

![指派使用者角色][200]

1. 在 Azure 入口網站中，開啟 [應用程式] 檢視、移至目錄檢視，然後選取 [企業應用程式] > [所有應用程式]。

    ![指派使用者][201]

2. 在 [應用程式] 清單中，選取 [Atlassian Cloud]。

    ![應用程式清單中的 [Atlassian Cloud] 連結](./media/atlassian-cloud-tutorial/tutorial_atlassiancloud_app.png)

3. 在左側窗格中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 選取 [新增]，然後在 [新增指派] 窗格中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 視窗的 [使用者] 清單中，選取 [Britta Simon]。

6. 在 [使用者和群組] 視窗中，選取 [選取]。

7. 在 [新增指派] 視窗中，選取 [指派]。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在「存取面板」中選取 [Atlassian Cloud] 圖格時，應該會自動登入您的 Atlassian Cloud 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/atlassian-cloud-tutorial/tutorial_general_01.png
[2]: ./media/atlassian-cloud-tutorial/tutorial_general_02.png
[3]: ./media/atlassian-cloud-tutorial/tutorial_general_03.png
[4]: ./media/atlassian-cloud-tutorial/tutorial_general_04.png

[100]: ./media/atlassian-cloud-tutorial/tutorial_general_100.png

[200]: ./media/atlassian-cloud-tutorial/tutorial_general_200.png
[201]: ./media/atlassian-cloud-tutorial/tutorial_general_201.png
[202]: ./media/atlassian-cloud-tutorial/tutorial_general_202.png
[203]: ./media/atlassian-cloud-tutorial/tutorial_general_203.png
