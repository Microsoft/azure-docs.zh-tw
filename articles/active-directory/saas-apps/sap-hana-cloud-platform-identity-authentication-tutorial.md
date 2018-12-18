---
title: 教學課程：Azure Active Directory 與 SAP Cloud Platform Identity Authentication 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAP Cloud Platform Identity Authentication 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 1c1320d1-7ba4-4b5f-926f-4996b44d9b5e
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/03/2018
ms.author: jeedes
ms.openlocfilehash: 266c9523f45294899e3cddbe782cbc54846eb119
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39422302"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-cloud-platform-identity-authentication"></a>教學課程：Azure Active Directory 與 SAP Cloud Platform Identity Authentication 整合

在本教學課程中，您會了解如何整合 SAP Cloud Platform Identity Authentication 與 Azure Active Directory (Azure AD)。 SAP Cloud Platform Identity Authentication 可作為 Proxy IdP，來存取以 Azure AD 作為主要 IdP 的 SAP 應用程式。

當您整合 SAP Cloud Platform Identity Authentication 與 Azure AD 時，會得到下列優點：

- 您可以在 Azure AD 中控制可存取 SAP 應用程式的人員。
- 您可以讓使用者使用其 Azure AD 帳戶自動登入 SAP 應用程式。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)一文。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SAP Cloud Platform Identity Authentication 的整合，您需要下列項目：

- Azure AD 訂用帳戶。
- SAP Cloud Platform Identity Authentication 的已啟用單一登入訂用帳戶。

> [!NOTE]
> 我們不建議使用生產環境來測試本教學課程中的步驟。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，[取得一個月免費試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 SAP Cloud Platform Identity Authentication
1. 設定並測試 Azure AD 單一登入

深入探討技術細節之前，一定要了解您即將查看的概念。 SAP Cloud Platform Identity Authentication 和 Active Directory 同盟服務可讓您利用 SAP Cloud Platform Identity Authentication 所保護的 SAP 應用程式和服務，跨越 Azure AD (如 IdP) 所保護的應用程式或服務實作 SSO。

SAP Cloud Platform Identity Authentication 目前作為 SAP 應用程式的領導 Proxy 識別提供者。 而 Azure Active Directory 做為此設定中的識別提供者。 

下圖說明此關聯性：

![建立 Azure AD 測試使用者](./media/sap-hana-cloud-platform-identity-authentication-tutorial/architecture-01.png)

透過此設定，您的 SAP Cloud Platform Identity Authentication 租用戶會設定為 Azure Active Directory 中信任的應用程式。 

接著會在 SAP Cloud Platform Identity Authentication 管理主控台中設定您想以此方式保護的所有 SAP 應用程式和服務。

因此，必須在 SAP Cloud Platform Identity Authentication 中進行授與 SAP 應用程式和服務存取權的授權作業 (相對於在 Azure Active Directory 中設定授權)。

藉由透過 Azure Active Directory Marketplace 將 SAP Cloud Platform Identity Authentication 設定為應用程式，您不需要設定個別宣告或 SAML 判斷提示。

>[!NOTE] 
>目前只有 Web SSO 經過這兩個合作夥伴的測試。 應用程式對 API 或 API 對 API 通訊所需的流程應能運作，但尚未經過測試。 它們會在後續活動期間進行測試。
>

## <a name="add-sap-cloud-platform-identity-authentication-from-the-gallery"></a>從資源庫新增 SAP Cloud Platform Identity Authentication
若要設定將 SAP Cloud Platform Identity Authentication 整合到 Azure AD 中，您需要將 SAP Cloud Platform Identity Authentication 從資源庫新增到受控 SaaS 應用程式清單中。

**若要從資源庫新增 SAP Cloud Platform Identity Authentication，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左方瀏覽窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **SAP Cloud Platform Identity Authentication**。 

1. 從結果面板選取 [SAP Cloud Platform Identity Authentication]，然後選取 [新增] 按鈕。

    ![結果清單中的 SAP Cloud Platform Identity Authentication](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會使用 SAP Cloud Platform Identity Authentication 設定及測試 Azure AD 單一登入。 您以名為 "Britta Simon" 的測試使用者來設定及測試。

若要讓單一登入運作，Azure AD 必須知道 SAP Cloud Platform Identity Authentication 中互相對應的使用者。 換句話說，您必須建立 Azure AD 使用者與 SAP Cloud Platform Identity Authentication 中相關使用者之間的連結。

在 SAP Cloud Platform Identity Authentication 中，提供值 **Username**，該值與 Azure AD 中的**使用者名稱**值相同。 現在您已經建立兩名使用者之間的連結。

若要使用 SAP Cloud Platform Identity Authentication 來設定並測試 Azure AD 單一登入，請完成下列建置組塊：

1. [設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 SAP Cloud Platform Identity Authentication 測試使用者](#create-an-sap-cloud-platform-identity-authentication-test-user) 使 SAP Cloud Platform Identity Authentication 中對應的 Britta Simon 連結到她在 Azure AD 中的代表項目。
1. [指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#test-single-sign-on)，以驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 SAP Cloud Platform Identity Authentication 應用程式中設定單一登入。

**若要使用 SAP Cloud Platform Identity Authentication 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SAP Cloud Platform Identity Authentication] 應用程式整合分頁上，選取 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊中，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_samlbase.png)

1. 如果您想要在 **IDP** 起始模式中設定應用程式，請在 [SAP Cloud Platform Identity Authentication 網域及 URL] 區段中，執行下列步驟：  

    ![SAP Cloud Platform Identity Authentication 網域及 URL 單一登入資訊](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_url.png)

    a. 在 [識別碼] 方塊中，以下列模式輸入 URL：`<IAS-tenant-id>.accounts.ondemand.com`

    b. 在 [回覆 URL] 方塊中，使用下列模式輸入 URL：`https://<IAS-tenant-id>.accounts.ondemand.com/saml2/idp/acs/<IAS-tenant-id>.accounts.ondemand.com`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [SAP Cloud Platform Identity Authentication 客戶支援小組](https://cloudplatform.sap.com/capabilities/security/trustcenter.html)以取得這些值。 如果您不知道此識別碼值，請參閱關於[租用戶 SAML 2.0 設定](https://help.hana.ondemand.com/cloud_identity/frameset.htm?e81a19b0067f4646982d7200a8dab3ca.html)的 SAP Cloud Platform Identity Authentication 文件。

1. 如果您想要以 **SP** 起始模式設定應用程式，請選取 [顯示進階 URL 設定]。

    ![SAP Cloud Platform Identity Authentication 網域及 URL 單一登入資訊](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_url1.png)

    在 [登入 URL] 方塊中，以下列模式輸入 URL︰`{YOUR BUSINESS APPLICATION URL}`。

    > [!NOTE]
    > 這不是真實的值。 請使用實際的登入 URL 來更新此值。 請使用您特有的商務應用程式登入 URL。 如有任何疑問，請連絡 [SAP Cloud Platform Identity Authentication 客戶支援小組](https://cloudplatform.sap.com/capabilities/security/trustcenter.html)。

1. 在 [SAML 簽章憑證] 區段中，選取 [中繼資料 XML]。 然後，將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_certificate.png)

1. SAP Cloud Platform Identity Authentication 應用程式需要特定格式的 SAML 判斷提示。 在應用程式整合分頁的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示格式的範例。 

    ![設定單一登入](./media/sap-hana-cloud-platform-identity-authentication-tutorial/attribute.png)

1. 在預期例如 **firstName** 之屬性的 SAP 應用程式中，在 [使用者屬性] 區段中新增 **firstName** 屬性。 此選項可用於 [SAML 權杖屬性] 對話方塊的 [單一登入] 對話方塊。

    a. 若要開啟 [新增屬性] 對話方塊，請選取 [新增屬性]。 
    
    ![設定單一登入](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_attribute_04.png)
    
    ![設定單一登入](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 方塊中，輸入屬性名稱 **firstName**。
    
    c. 從 [值] 清單選取 **user.givenname** 屬性值。
    
    d. 選取 [確定]。

1. 選取 [儲存] 按鈕。

    ![設定單一登入儲存按鈕](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_general_400.png)

1. 在 [SAP Cloud Platform Identity Authentication 設定] 區段中，按一下 [設定 SAP Cloud Platform Identity Authentication] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登出 URL、SAML 實體識別碼和 SAML 單一登入服務 URL]。

    ![SAP Cloud Platform Identity Authentication 組態](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_configure.png) 

1. 若要為您的應用程式設定 SSO，請移至 SAP Cloud Platform Identity Authentication 管理主控台。 URL 具有下列模式：`https://<tenant-id>.accounts.ondemand.com/admin`。 然後參閱位於[與 Microsoft Azure AD 整合](https://help.hana.ondemand.com/cloud_identity/frameset.htm?626b17331b4d4014b8790d3aea70b240.html)的 SAP Cloud Platform Identity Authentication 相關文件。 

1. 在 Azure 入口網站中，選取 [儲存] 按鈕。

1. 只有當您想要為另一個 SAP 應用程式新增和啟用 SSO 時，才繼續以下步驟。 重複**從資源庫新增 SAP Cloud Platform Identity Authentication**一節底下的步驟。

1. 在 Azure 入口網站的 [SAP Cloud Platform Identity Authentication] 應用程式整合分頁上，選取 [連結的登入]。

    ![設定連結的登入](./media/sap-hana-cloud-platform-identity-authentication-tutorial/linked_sign_on.png)

1. 儲存組態。

>[!NOTE] 
>新的應用程式會利用先前 SAP 應用程式的單一登入設定。 請確定您在 SAP Cloud Platform Identity Authentication 管理主控台中使用相同的公司識別提供者。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤，透過底部的 [設定] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/sap-hana-cloud-platform-identity-authentication-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後選取 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/sap-hana-cloud-platform-identity-authentication-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，請選取 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/sap-hana-cloud-platform-identity-authentication-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，採取下列步驟：

    ![[使用者] 對話方塊](./media/sap-hana-cloud-platform-identity-authentication-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 選取 [建立] 。
 
### <a name="create-an-sap-cloud-platform-identity-authentication-test-user"></a>建立 SAP Cloud Platform Identity Authentication 測試使用者

您不需要在 SAP Cloud Platform Identity Authentication 中建立使用者。 Azure AD 使用者存放區中的使用者可以使用 SSO 功能。

SAP Cloud Platform Identity Authentication 支援 [識別身分同盟] 選項。 此選項可讓應用程式檢查公司識別提供者所驗證的使用者是否存在於 SAP Cloud Platform Identity Authentication 的使用者存放區中。 

[識別身分同盟] 選項預設為停用。 如果已啟用 [識別身分同盟]，則只有匯入 SAP Cloud Platform Identity Authentication 的使用者可以存取應用程式。 

如需如何啟用或停用與 SAP Cloud Platform Identity Authentication 之識別身分同盟的詳細資訊，請參閱[設定與 SAP Cloud Platform Identity Authentication 之使用者存放區的識別身分同盟](https://help.hana.ondemand.com/cloud_identity/frameset.htm?c029bbbaefbf4350af15115396ba14e2.html)中的＜啟用與 SAP Cloud Platform Identity Authentication 的識別身分同盟＞。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您可將 SAP Cloud Platform Identity Authentication 的存取權授予 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 SAP Cloud Platform Identity Authentication，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，然後移至目錄檢視。 接下來，移至 [企業應用程式]，然後選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 **SAP Cloud Platform Identity Authentication**。

    ![應用程式清單中的 SAP Cloud Platform Identity Authentication 連結](./media/sap-hana-cloud-platform-identity-authentication-tutorial/tutorial_sapcpia_app.png)  

1. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 選取 [新增] 按鈕。 然後，在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

選取存取面板中的 SAP Cloud Platform Identity Authentication 圖格時，您會自動登入 SAP Cloud Platform Identity Authentication 應用程式。

如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/sapcloudauth-tutorial/tutorial_general_01.png
[2]: ./media/sapcloudauth-tutorial/tutorial_general_02.png
[3]: ./media/sapcloudauth-tutorial/tutorial_general_03.png
[4]: ./media/sapcloudauth-tutorial/tutorial_general_04.png

[100]: ./media/sapcloudauth-tutorial/tutorial_general_100.png

[200]: ./media/sapcloudauth-tutorial/tutorial_general_200.png
[201]: ./media/sapcloudauth-tutorial/tutorial_general_201.png
[202]: ./media/sapcloudauth-tutorial/tutorial_general_202.png
[203]: ./media/sapcloudauth-tutorial/tutorial_general_203.png
