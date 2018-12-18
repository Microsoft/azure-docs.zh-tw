---
title: 教學課程：Azure Active Directory 與 Salesforce 沙箱整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Salesforce Sandbox 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: ee54c39e-ce20-42a4-8531-da7b5f40f57c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/07/2018
ms.author: jeedes
ms.openlocfilehash: 6feafba41cf65a752dd5bf0819b0b93bacff0aff
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42141666"
---
# <a name="tutorial-azure-active-directory-integration-with-salesforce-sandbox"></a>教學課程：Azure Active Directory 與 Salesforce 沙箱整合

在本教學課程中，您會了解如何整合 Salesforce Sandbox 與 Azure Active Directory (Azure AD)。

沙箱讓您能夠針對不同用途 (例如開發、測試和訓練) 在個別環境中建立貴組織的多個複本，而不會危害 Salesforce 生產環境組織中的資料和應用程式。
如需詳細資訊，請參閱[沙箱概觀](https://help.salesforce.com/articleView?id=create_test_instance.htm&language=en_us&type=5)。

Salesforce Sandbox 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Salesforce Sandbox 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Salesforce Sandbox (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Salesforce Sandbox 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 啟用 Salesforce Sandbox 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Salesforce Sandbox
2. 設定並測試 Azure AD 單一登入

## <a name="adding-salesforce-sandbox-from-the-gallery"></a>從資源庫新增 Salesforce Sandbox

若要設定將 Salesforce Sandbox 整合到 Azure AD 中，您需要從資源庫將 Salesforce Sandbox 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Salesforce Sandbox，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Salesforce Sandbox**，從結果面板中選取 [Salesforce Sandbox]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Salesforce Sandbox](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，設定及測試與 Salesforce Sandbox 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Salesforce Sandbox 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Salesforce Sandbox 中的相關使用者之間，建立連結關聯性。

在 Salesforce Sandbox 中，將 Azure AD 中**使用者名稱**的值，指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Salesforce Sandbox 搭配運作的 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Salesforce Sandbox 測試使用者](#create-a-salesforce-sandbox-test-user)** - 使 Salesforce Sandbox 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 Salesforce Sandbox 應用程式中設定單一登入。

**若要與 Salesforce Sandbox 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Salesforce Sandbox] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_samlbase.png)

3. 如果您想要以 **IDP** 起始模式設定應用程式，請在 [Salesforce 沙箱網域和 URL] 區段上執行下列步驟：

   ![Salesforce Sandbox 網域和 URL 單一登入資訊](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_url1.png)

   在 [回覆 URL] 文字方塊中輸入您組織專屬的**回覆 URL**。

   > [!NOTE]
   > 使用實際的「回覆 URL」來更新 [回覆 URL] 值，本教學課程稍後會說明。

4. 在 [SAML 簽署憑證] 區段上，按一下 [憑證\(原始\)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_certificate.png)

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/salesforce-sandbox-tutorial/tutorial_general_400.png)

6. 在 [Salesforce Sandbox 組態] 區段上，按一下 [設定 Salesforce Sandbox] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_configure.png)

7. 在瀏覽器中開啟新索引標籤，登入您的 Salesforce Sandbox 系統管理員帳戶。

8. 按一下分頁右上角**設定圖示**底下的 [設定]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure1.png)

9. 在左側瀏覽窗格中向下捲動至 [設定]，按一下 [識別] 以展開相關的區段。 然後按一下 [單一登入設定]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-admin-sso.png)

10. 在 [單一登入設定] 頁面上，按一下 [編輯] 按鈕。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure3.png)

11. 選取 [啟用 SAML]，然後按一下 [儲存]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-enable-saml.png)

12. 若要設定 SAML 單一登入設定，請按一下 [新增] 。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-admin-sso-new.png)

13. 在 [單一登入設定] 區段中，執行下列步驟：

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-saml-config1.png)

    a. 選取 [已啟用 SAML]  核取方塊。

    b. 在 [簽發者] 欄位中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c. 若要上傳 [識別提供者憑證]，按一下 [瀏覽] 以瀏覽及選取您從 Azure 入口網站下載的憑證檔案。

    d. 在 [識別提供者登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值。

    e. 作為 **SAML 識別類型**，選擇下列其中一個選項：

      * 如果使用者的 Salesforce 使用者名稱傳入 SAML 判斷提示，選取 [判斷提示包含使用者的 Salesforce 使用者名稱]

      * 如果使用者物件的同盟識別碼傳入 SAML 判斷提示，選取 [判斷提示包含使用者物件的同盟識別碼]
  
    f. 針對 [SAML 身分識別位置]，選取 [身分識別為屬性項目]。

    g. SFDC 不支援 SAML 登出。  但可將 `https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0` 貼到 [自訂登出 URL] 文字方塊中，以解決此問題。

    h. 按一下 [檔案] 。

14. 在 [單一登入設定] 頁面上，按一下 [下載中繼資料] 按鈕。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure4.png)

15. 在不同的瀏覽器視窗中開啟下載的中繼資料，然後複製 [位置] 值，並在 Azure 入口網站中，將此值貼到 [Salesforce 沙箱網域和 URL] 區段上的 [回覆 URL] 文字方塊中。  

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure5.png)

16. 如果您想要以 **SP** 起始模式設定應用程式，下列是此動作的必要條件：

    a. 應具有已驗證的網域。

    b. 您需要設定在 Salesforce 沙箱中設定並啟用網域，稍後會在本教學課程中說明此動作的步驟。

    c. 在 Azure 入口網站的 [Salesforce 沙箱網域和 URL] 區段中，按一下 [顯示進階 URL 設定]，然後執行下列步驟：
  
    ![Salesforce Sandbox 網域和 URL 單一登入資訊](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_url.png)

    在 [登入 URL] 文字方塊中，以下列模式輸入值：`https://<instancename>--Sandbox.<entityid>.my.salesforce.com`

    > [!NOTE]
    > 啟用網域之後，應從 Salesforce 沙箱入口網站複製此值。

17. 在 [SAML 簽署憑證] 區段上，按一下 [憑證\(原始\)]，然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_certificate.png)

18. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/salesforce-sandbox-tutorial/tutorial_general_400.png)

19. 在 [Salesforce Sandbox 組態] 區段上，按一下 [設定 Salesforce Sandbox] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [SAML 實體 ID 和 SAML 單一登入服務 URL]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_configure.png)

20. 在瀏覽器中開啟新索引標籤，登入您的 Salesforce Sandbox 系統管理員帳戶。

21. 按一下分頁右上角**設定圖示**底下的 [設定]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure1.png)

22. 在左側瀏覽窗格中向下捲動至 [設定]，按一下 [識別] 以展開相關的區段。 然後按一下 [單一登入設定]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-admin-sso.png)

23. 在 [單一登入設定] 頁面上，按一下 [編輯] 按鈕。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/configure3.png)

24. 選取 [啟用 SAML]，然後按一下 [儲存]。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-enable-saml.png)

25. 若要設定 SAML 單一登入設定，請按一下 [新增] 。

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-admin-sso-new.png)

26. 如果您要新增第二個執行個體，必須啟用網域，如我們先前所述 (起始 SP 的案例)。 在 [SAML 單一登入設定] 區段中，執行下列步驟：

    ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-saml-config.png)

    a. 在 [名稱] 文字方塊中，輸入設定的名稱 (例如：*SPSSOWAAD_Test*)。

    b. 在 [簽發者] 欄位中，貼上您從 Azure 入口網站複製的 [SAML 實體識別碼] 值。

    c. 在 [實體識別碼] 文字方塊中，使用 `https://test.salesforce.com` 值作為第一個執行個體，然後從應用程式的第二個執行個體開始，您可以使用租用戶專屬的識別碼值。

    d. 若要上傳 [識別提供者憑證]，按一下 [選擇檔案] 以瀏覽及選取您從 Azure 入口網站下載的憑證檔案。

    e. 作為 **SAML 識別類型**，選擇下列其中一個選項：

      * 如果使用者的 Salesforce 使用者名稱傳入 SAML 判斷提示，選取 [判斷提示包含使用者的 Salesforce 使用者名稱]

      * 如果使用者物件的同盟識別碼傳入 SAML 判斷提示，選取 [判斷提示包含使用者物件的同盟識別碼]

      * 如果使用者物件的使用者識別碼傳入 SAML 判斷提示，選取 [判斷提示包含使用者物件的使用者識別碼]

    f. 對於 [SAML 身分識別位置]，選取 [身分識別位於 Subject 陳述式的 NameIdentifier 元素中]。

    g. 在 [服務提供者起始的要求繫結]，選取 [HTTP POST]。

    h. 在 [識別提供者登入 URL] 文字方塊中，貼上您從 Azure 入口網站複製的 [單一登入服務 URL] 值。

    i. SFDC 不支援 SAML 登出。  但可將 `https://login.microsoftonline.com/common/wsfederation?wa=wsignout1.0` 貼到 [自訂登出 URL] 文字方塊中，以解決此問題。

    j. 按一下 [檔案] 。

27. 若要在 Salesforce 沙箱上啟用網域，請執行下列步驟：

    > [!NOTE]
    > 啟用網域之前，您必須在 Salesforce 沙箱上建立相同網域。 如需詳細資訊，請參閱[定義您的網域名稱](https://help.salesforce.com/HTViewHelpDoc?id=domain_name_define.htm&language=en_US)。 建好網域之後，請確定已正確設定網域。

    * 在 Salesforce 沙箱的左方導覽窗格中，按一下 [公司設定] 以展開相關的區段，然後按一下 [我的網域]。

         ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-my-domain.png)

    * 在 [驗證組態] 區段中，按一下 [編輯]。

        ![設定單一登入](./media/salesforce-sandbox-tutorial/sf-edit-auth-config.png)

    * 在 [驗證設定] 區段中，針對 [驗證服務]，選取在 Salesforce 沙箱中設定 SSO 期間所設定的 SAML 單一登入設定名稱，然後按一下 [儲存]。

        ![設定單一登入](./media/salesforce-sandbox-tutorial/configure2.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/salesforce-sandbox-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/salesforce-sandbox-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/salesforce-sandbox-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/salesforce-sandbox-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-a-salesforce-sandbox-test-user"></a>建立 Salesforce Sandbox 測試使用者

本節會在 Salesforce Sandbox 中建立名為 Britta Simon 的使用者。 Salesforce Sandbox 支援預設啟用的 Just-In-Time 佈建。 在這一節沒有您需要進行的動作項目。 如果 Salesforce Sandbox 中還沒有使用者，當您嘗試存取 Salesforce Sandbox 時，就會建立新的使用者。 Salesforce Sandbox 也支援自動使用者佈建，您可以在[這裡](salesforce-sandbox-provisioning-tutorial.md)找到關於如何設定自動使用者佈建的更多詳細資料。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Salesforce Sandbox 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Salesforce Sandbox，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Salesforce Sandbox]。

    ![應用程式清單中的 Salesforce Sandbox 連結](./media/salesforce-sandbox-tutorial/tutorial_salesforcesandbox_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Salesforce Sandbox] 圖格時，應該會自動登入您的 Salesforce Sandbox 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)
* [設定使用者佈建](salesforce-sandbox-provisioning-tutorial.md)


<!--Image references-->

[1]: ./media/salesforce-sandbox-tutorial/tutorial_general_01.png
[2]: ./media/salesforce-sandbox-tutorial/tutorial_general_02.png
[3]: ./media/salesforce-sandbox-tutorial/tutorial_general_03.png
[4]: ./media/salesforce-sandbox-tutorial/tutorial_general_04.png

[100]: ./media/salesforce-sandbox-tutorial/tutorial_general_100.png

[200]: ./media/salesforce-sandbox-tutorial/tutorial_general_200.png
[201]: ./media/salesforce-sandbox-tutorial/tutorial_general_201.png
[202]: ./media/salesforce-sandbox-tutorial/tutorial_general_202.png
[203]: ./media/salesforce-sandbox-tutorial/tutorial_general_203.png