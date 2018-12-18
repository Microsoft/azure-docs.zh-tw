---
title: 教學課程：Azure Active Directory 與 Adobe Experience Manager 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Adobe Experience Manager 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 88a95bb5-c17c-474f-bb92-1f80f5344b5a
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/28/2017
ms.author: jeedes
ms.openlocfilehash: 56b392e57809cea0ae93800df39bb9dacd164ce2
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39054177"
---
# <a name="tutorial-azure-active-directory-integration-with-adobe-experience-manager"></a>教學課程：Azure Active Directory 與 Adobe Experience Manager 整合

在本教學課程中，您將了解如何整合 Adobe Experience Manager 與 Azure Active Directory (Azure AD)。

Adobe Experience Manager 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Adobe Experience Manager 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Adobe Experience Manager。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Adobe Experience Manager 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Adobe Experience Manager 單一登入的訂用帳戶

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 Adobe Experience Manager
2. 設定並測試 Azure AD 單一登入

## <a name="add-adobe-experience-manager-from-the-gallery"></a>從資源庫新增 Adobe Experience Manager
若要設定 Adobe Experience Manager 與 Azure AD 整合，您需要從資源庫將 Adobe Experience Manager 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Adobe Experience Manager，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

2. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
3. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Adobe Experience Manager**。 從結果面板中選取 [Adobe Experience Manager]，然後選取 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Adobe Experience Manager](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Adobe Experience Manager 搭配運作的 Azure AD 單一登入。

若要讓單一登入能夠運作，Azure AD 必須知道 Adobe Experience Manager 與 Azure AD 中互相對應的使用者。 換句話說，您必須在 Azure AD 使用者與 Adobe Experience Manager 中的相關使用者之間建立連結。

在 Adobe Experience Manager 中，為 [Username] \(使用者名稱\) 值提供與 Azure AD 中**使用者名稱**相同的值。 現在您已經建立兩名使用者之間的連結。 

若要設定及測試與 Adobe Experience Manager 搭配運作的 Azure AD 單一登入，請完成下列構成要素：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
2. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
3. [建立 Adobe Experience Manager 測試使用者](#create-an-adobe-experience-manager-test-user)，以在 Adobe Experience Manager 中建立一個與 Azure AD 中代表 Britta Simon 之使用者連結的 Britta Simon 對應項目。
4. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. [測試單一登入](#test-single-sign-on)，驗證設定是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Adobe Experience Manager 應用程式中設定單一登入。

**若要設定與 Adobe Experience Manager 搭配運作的 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Adobe Experience Manager] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入連結][4]

2. 若要啟用單一登入，請在 [單一登入] 對話方塊的 [模式] 下拉式功能表中，選取 [SAML 登入]。
 
    ![單一登入對話方塊](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_samlbase.png)

3. 如果您想要以 **IdP** 模式設定應用程式，請在 [Adobe Experience Manager 網域及 URL] 區段中執行下列步驟：

    ![Adobe Experience Manager 網域及 URL 單一登入資訊](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_url1.png)

    a. 在 [識別碼] 方塊中，輸入您也會在 AEM 伺服器上定義的唯一值。 

    b. 在 [回覆 URL] 方塊中，以下列模式輸入 URL：`https://<AEM Server Url>/saml_login`。

    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 來更新這些值。 若要取得這些值，請連絡 [Adobe Experience Manager 支援小組](https://helpx.adobe.com/support/experience-manager.html) \(英文\)。
 
4. 按一下 [顯示進階 URL 設定]。 接著，如果您想要以 **SP**起始模式設定應用程式，請執行下列步驟：

    ![Adobe Experience Manager 網域及 URL 單一登入資訊](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_spconfigure.png)

    在 [登入 URL] 方塊中，輸入您的 Adobe Experience Manager 伺服器 URL。 

5. 在 [SAML 簽署憑證] 區段中，選取 [憑證 (Base64)]。 然後將憑證檔案儲存在您的電腦上。

    ![憑證下載連結](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_certificate.png) 

6. 若要開啟設定登入視窗，請在 [Adobe Experience Manager 設定] 區段中，選取 [設定 Adobe Experience Manager]。 從 [快速參考] 區段中複製 [SAML 登入服務 URL]、[SAML 實體識別碼] 及 [登出識別碼]。

    ![設定區段連結](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_configure.png) 

7. 選取 [ **儲存**]。

    ![設定單一登入 [儲存] 按鈕](./media/adobeexperiencemanager-tutorial/tutorial_general_400.png)

8. 在另一個瀏覽器視窗中，開啟 **Adobe Experience Manager** 管理入口網站。

9. 選取 [Settings] \(設定\) > [Security] \(安全性\) > [Users] \(使用者\)。

    ![設定單一登入 [儲存] 按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user.png)

10. 選取 [Administrator] \(系統管理員\) 或任何其他相關的使用者。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin6.png)

11. 選取 [Account Settings] \(帳戶設定\) > [Manage TrustStore] \(管理 TrustStore\)。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_managetrust.png)

12. 在 [Add Certificate from CER file] \(從 CER 檔案新增憑證\) 底下，按一下 [Select Certificate File] \(選取憑證檔案\)。 瀏覽並選取您已經從 Azure 入口網站下載的憑證檔案。

    ![設定單一登入 [儲存] 按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_user2.png)

13. 憑證即會新增至 TrustStore。 請記下憑證的別名。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin7.png)

14. 在 [Users] \(使用者\) 頁面上，選取 [authentication-service]。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin8.png)

15. 選取 [Account settings] \(帳戶設定\) > [Create/Manage KeyStore] \(建立/管理 KeyStore\)。 提供密碼以建立 KeyStore。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin9.png)

16. 返回管理畫面。 然後選取 [Settings] \(設定\) > [Operations] \(作業\) > [Web Console] \(Web 主控台\)。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin1.png)

    這會開啟設定頁面。

    ![設定單一登入 [儲存] 按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin2.png)

17. 找出 **Adobe Granite SAML 2.0 Authentication Handler**。 然後選取 [Add] \(新增\) 圖示。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin3.png)

19. 在此頁面上執行下列動作。

    ![設定單一登入儲存按鈕](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_admin4.png)

    a. 在 [Path] \(路徑\) 方塊中，輸入 **/**。

    b. 在 [IDP URL] 方塊中，輸入您從 Azure 入口網站複製的 **SAML 單一登入服務 URL**。

    c. 在 [IDP Certificate Alias] \(IDP 憑證別名\) 方塊中，輸入您已在 TrustStore 中新增的**憑證別名**值。

    d. 在 [Service Provider Entity ID] \(服務提供者實體識別碼\) 方塊中，輸入您已在 Azure 入口網站中設定的唯一 **SAML 實體識別碼**值。

    e. 在 [Assertion Consumer Service URL] \(判斷提示取用者服務 URL\) 方塊中，輸入您已在 Azure 入口網站中設定的**回覆 URL** 值。

    f. 在 [Password of Key Store] \(金鑰存放區的密碼\) 方塊中，輸入您已在 KeyStore 中設定的**密碼**。

    g. 在 [User Attribute ID] \(使用者屬性識別碼\) 方塊中，輸入**名稱識別碼**或您案例中相關的其他使用者識別碼。

    h. 選取 [Autocreate CRX Users] \(自動建立 CRX 使用者\)。

    i. 在 [Logout URL] \(登出 URL\) 方塊中，貼上您從 Azure 入口網站取得的**登出 URL**值。

    j. 選取 [ **儲存**]。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本。 從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤。然後透過底部的 [設定] 區段來存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/adobeexperiencemanager-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/adobeexperiencemanager-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，請在 [所有使用者] 對話方塊的頂端，選取 [新增]。

    ![[新增] 按鈕](./media/adobeexperiencemanager-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/adobeexperiencemanager-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊。 然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
  
### <a name="create-an-adobe-experience-manager-test-user"></a>建立 Adobe Experience Manager 測試使用者

在本節中，您會在 Adobe Experience Manager 中建立名為 Britta Simon 的使用者。 如果您已選取 [Autocreate CRX Users] \(自動建立 CRX 使用者\) 選項，系統就會在成功驗證之後，自動建立使用者。 

如果您想要手動建立使用者，請與 [Adobe Experience Manager 支援小組](https://helpx.adobe.com/support/experience-manager.html) \(英文\) 合作，以在 Adobe Experience Manager 平台中新增使用者。 

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Adobe Experience Manager 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Adobe Experience Manager，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視。 接著，移至目錄檢視，選取 [企業應用程式]，然後選取 [所有應用程式]。

    ![指派使用者][201] 

2. 在應用程式清單中，選取 [Adobe Experience Manager]。

    ![應用程式清單中的 Adobe Experience Manager 連結](./media/adobeexperiencemanager-tutorial/tutorial_adobeexperiencemanager_app.png)  

3. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

6. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

7. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [Adobe Experience Manager] 圖格時，應該會自動登入您的 Adobe Experience Manager 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/adobeexperiencemanager-tutorial/tutorial_general_01.png
[2]: ./media/adobeexperiencemanager-tutorial/tutorial_general_02.png
[3]: ./media/adobeexperiencemanager-tutorial/tutorial_general_03.png
[4]: ./media/adobeexperiencemanager-tutorial/tutorial_general_04.png

[100]: ./media/adobeexperiencemanager-tutorial/tutorial_general_100.png

[200]: ./media/adobeexperiencemanager-tutorial/tutorial_general_200.png
[201]: ./media/adobeexperiencemanager-tutorial/tutorial_general_201.png
[202]: ./media/adobeexperiencemanager-tutorial/tutorial_general_202.png
[203]: ./media/adobeexperiencemanager-tutorial/tutorial_general_203.png

