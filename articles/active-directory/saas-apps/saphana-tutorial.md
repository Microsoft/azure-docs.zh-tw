---
title: 教學課程：Azure Active Directory 與 SAP HANA 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 SAP HANA 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: cef4a146-f4b0-4e94-82de-f5227a4b462c
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2017
ms.author: jeedes
ms.openlocfilehash: e498b0ca4b9efe09c2fe2f2bfcdcb3cc68b9c2c4
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39430248"
---
# <a name="tutorial-azure-active-directory-integration-with-sap-hana"></a>教學課程：Azure Active Directory 與 SAP HANA 整合

在本教學課程中，您會了解如何整合 SAP HANA 與 Azure Active Directory (Azure AD)。

當您將 SAP Hana 與 Azure AD 整合時，會獲得下列優點：

- 您可以在 Azure AD 中控制可存取 SAP Hana 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 SAP Hana。
- 您可以在 Azure 入口網站集中管理您的帳戶。

如需 SaaS 應用程式與 Azure AD 整合的詳細資訊，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 SAP HANA 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用單一登入 (SSO) 的 SAP Hana 訂用帳戶
- 執行於任何公用 IaaS、內部部署、Azure VM 或 Azure 中 SAP 大型執行個體的 HANA 執行個體
- XSA 管理 Web 介面，以及安裝在 HANA 執行個體上的 HANA Studio

> [!NOTE]
> 我們不建議使用 SAP Hana 的生產環境以測試本教學課程中的步驟。 先在應用程式的開發或預備環境中測試整合，然後使用生產環境。

若要測試本教學課程中的步驟，請遵循下列建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，可取得 [Azure AD 一個月免費試用版](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二個主要建置組塊組成：

1. 從資源庫新增 SAP HANA
1. 設定並測試 Azure AD 單一登入

## <a name="add-sap-hana-from-the-gallery"></a>從資源庫新增 SAP Hana
若要設定將 SAP Hana 整合到 Azure AD 中，請從資源庫將 SAP Hana 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 SAP Hana，請執行下列步驟：**

1. 在 [Azure 入口網站](https://portal.azure.com)的左側窗格中，選取 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 移至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請選取對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **SAP Hana**。 然後從結果面板中選取 [SAP Hana]。 最後，選取 [新增] 按鈕以新增應用程式。 

    ![新的應用程式](./media/saphana-tutorial/tutorial_saphana_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入
在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 SAP HANA 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 SAP Hana 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 SAP Hana 中的相關使用者之間建立連結關聯性。

在 SAP Hana 中，請將與 Azure AD 中**使用者名稱**相同的值提供給 **Username** 值。 此步驟會建立兩名使用者之間的連結。

若要使用 SAP Hana 設定及測試 Azure AD 單一登入，請完成下列建構元素：

1. [設定 Azure AD 單一登入](#configuring-azure-ad-single-sign-on)，讓您的使用者能夠使用此功能。
1. [建立 Azure AD 測試使用者](#creating-an-azure-ad-test-user)，以使用 Britta Simon 測試 Azure AD 單一登入。
1. [建立 SAP Hana 測試使用者](#creating-a-sap-hana-test-user)使 SAP Hana 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. [指派 Azure AD 測試使用者](#assigning-the-azure-ad-test-user)，讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. [測試單一登入](#testing-single-sign-on)，驗證設定是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，並在您的 SAP HANA 應用程式中設定單一登入。

**若要使用 SAP Hana 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [SAP Hana] 應用程式整合頁面上，選取 [單一登入]。

    ![設定單一登入][4]

1. 在 [單一登入] 對話方塊中，於 [SAML 型登入] 下，選取 [模式]。
 
    ![單一登入對話方塊](./media/saphana-tutorial/tutorial_saphana_samlbase.png)

1. 在 [SAP Hana 網域與 URL] 區段上，執行下列步驟：

    ![網域與 URL 單一登入資訊](./media/saphana-tutorial/tutorial_saphana_url.png)

    a. 在 [識別碼] 方塊中，輸入下列值：`HA100` 

    b. 在 [回覆 URL] 方塊中，使用下列模式輸入 URL：`https://<Customer-SAP-instance-url>/sap/hana/xs/saml/login.xscfunc`

    > [!NOTE] 
    > 這些都不是真正的值。 請使用實際的識別碼和回覆 URL 更新這些值。 請連絡 [SAP Hana 客戶支援小組](https://cloudplatform.sap.com/contact.html)以取得這些值。 

1. 在 [SAML 簽章憑證] 區段中，選取 [中繼資料 XML]。 然後，將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/saphana-tutorial/tutorial_saphana_certificate.png) 

    >[!Note]
    >如果憑證沒有作用，請選取 Azure AD 中的 [啟用新憑證] 核取方塊來啟用該憑證。 

1. SAP Hana 應用程式需要特定格式的 SAML 判斷提示。 以下螢幕擷取畫面顯示了此格式的範例。 

    我們已將**使用者識別碼**對應至 **user.mail** 的 **ExtractMailPrefix()** 函數。 如此一來，使用者電子郵件的首碼值便會是唯一的使用者識別碼。 在每次成功的回應中，此使用者識別碼都會傳送至 SAP Hana 應用程式。

    ![設定單一登入](./media/saphana-tutorial/attribute.png)

1. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，採取下列步驟：

    a. 在 [使用者識別碼] 下拉式清單中，選取 [ExtractMailPrefix]。
    
    b. 在 [郵件] 下拉式清單中，選取 [user.mail]。

1. 選取 [儲存] 按鈕。

    ![設定單一登入儲存按鈕](./media/saphana-tutorial/tutorial_general_400.png)
    
1. 若要在 SAP Hana 端設定單一登入，請瀏覽至個別的 HTTPS 端點以登入您的 **HANA XSA Web 主控台**。

    > [!NOTE]
    > 在預設組態中，URL 會將要求重新導向至登入畫面，這需要已通過驗證之 SAP Hana 資料庫使用者的認證。 登入的使用者必須擁有執行 SAML 管理工作的權限。

1. 前往 XSA Web 介面中的 [SAML 識別提供者]。 在畫面底部選取 **+** 按鈕，以顯示 [新增識別提供者資訊] 窗格。 然後採取下列步驟：

    ![新增識別提供者](./media/saphana-tutorial/sap1.png)

    a. 在 [新增識別提供者資訊] 窗格中，將中繼資料 XML 內容 (從 Azure 入口網站下載) 貼到 [中繼資料] 方塊中。

    ![[新增識別提供者] 設定](./media/saphana-tutorial/sap2.png)

    b. 如果 XML 文件的內容有效，剖析程序會擷取 [一般資料] 畫面區域中的 [主旨]、[實體識別碼] 和 [簽發者] 欄位所需的資訊。 該程序也會擷取 [目的地] 畫面區域中 URL 欄位的所需資訊，例如 [基底 URL] 和 [SingleSignOn URL] (*) 欄位。

    ![[新增識別提供者] 設定](./media/saphana-tutorial/sap3.png)

    c. 在 [一般資料] 畫面區域的 [名稱] 方塊中，輸入新 SAML SSO 識別提供者的名稱。

    > [!NOTE]
    > SAML IDP 名稱為必填，且不可重複。 當您選取 SAML 作為 SAP Hana XS 應用程式要使用的驗證方法時，該名稱會出現在可用的 SAML IDP 清單中。 例如，您可以在 XS Artifact Administration 工具的 [驗證] 畫面區域中執行此操作。

1. 選取 [儲存]，以儲存 SAML 識別提供者的詳細資料，並將新的 SAML IDP 新增至已知 SAML IDP 的清單。

    ![[儲存] 按鈕](./media/saphana-tutorial/sap4.png)

1. 在 HANA Studio 之 [設定] 索引標籤的系統屬性中，依 **saml** 篩選設定。 然後將 **assertion_timeout** 從 **10 秒**調整為 **120 秒**。

    ![assertion_timeout 設定](./media/saphana-tutorial/sap7.png)

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！ 從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，選取 [單一登入] 索引標籤，即可透過底部的 [設定] 區段存取內嵌的文件。 您可以在 [Azure AD 內嵌文件](https://go.microsoft.com/fwlink/?linkid=845985)中閱讀更多有關內嵌文件功能的資訊。
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者
本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 使用者][100]

**若要在 Azure AD 中建立測試使用者，請採取下列步驟：**

1. 在 **Azure 入口網站**的左側窗格中，選取 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕](./media/saphana-tutorial/create_aaduser_01.png) 

1. 若要顯示使用者清單，請移至 [使用者和群組]。 然後選取 [所有使用者]。
    
    ![[使用者和群組] 與 [所有使用者] 連結](./media/saphana-tutorial/create_aaduser_02.png) 

1. 若要開啟 [使用者] 對話方塊，請在對話方塊的頂端選取 [新增]。
 
    ![[新增] 按鈕](./media/saphana-tutorial/create_aaduser_03.png) 

1. 在 [使用者] 對話方塊中，採取下列步驟：
 
    ![[使用者] 對話方塊](./media/saphana-tutorial/create_aaduser_04.png) 

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入 BrittaSimon 的**電子郵件地址**。

    c. 選取 [顯示密碼]，並記下密碼。

    d. 選取 [建立] 。
 
### <a name="create-a-sap-hana-test-user"></a>建立 SAP Hana 測試使用者

若要讓 Azure AD 使用者登入 SAP Hana，您必須在 SAP HANA 中佈建這些使用者。
SAP Hana 支援預設啟用的 Just-In-Time 佈建。

如果您需要手動建立使用者，請採取下列步驟：

>[!NOTE]
>您可以變更使用者所使用的外部驗證。 他們可以使用外部系統 (例如 Kerberos) 進行驗證。 如需外部身分識別的詳細資訊，請連絡[網域系統管理員](https://cloudplatform.sap.com/contact.html)。

1. 以系統管理員身分開啟 [SAP Hana Studio](https://help.sap.com/viewer/a2a49126a5c546a9864aae22c05c3d0e/2.0.01/en-us)，然後為 SAML SSO 啟用 DB-User。

    ![建立使用者](./media/saphana-tutorial/sap5.png)

1. 選取 [SAML] 左邊的不可見核取方塊，並選取 [設定] 連結。

1. 選取 [新增] 以新增 SAML IDP。  選取適當的 SAML IDP，然後選取 [確定]。

1. 新增 [外部識別] \(在此情況下為 BrittaSimon)，或選擇 [任何]。 然後選取 [確定]。

    >[!Note]
    >如果未選取 [任何] 核取方塊，則 HANA 中的使用者名稱必須完全符合 UPN 中網域尾碼前面的使用者名稱。 (舉例而言，BrittaSimon@contoso.com 會在 HANA 中變成 BrittaSimon。)

1. 為了進行測試，請對使用者指派所有 **XS** 角色。

    ![指派角色](./media/saphana-tutorial/sap6.png)

    > [!TIP]
    > 您只應該提供適用於您使用案例的權限。

1. 儲存使用者。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 SAP HANA 的存取權授與 ，讓 Britta Simon 能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派到 SAP HANA，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視。 移至目錄檢視中，並移至**企業應用程式**。 選取 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [SAP HANA]。

    ![指派使用者](./media/saphana-tutorial/tutorial_saphana_app.png) 

1. 在左側功能表中，選取 [使用者和群組]。

    ![[使用者和群組] 連結][202] 

1. 選取 [新增] 按鈕。 在 [新增指派] 對話方塊中，選取 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊中，選取 [使用者] 清單中的 [Britta Simon]。

1. 在 [使用者和群組] 對話方塊中，按一下 [選取] 按鈕。

1. 在 [新增指派] 對話方塊中，選取 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入組態。

當您在存取面板中選取 [SAP Hana] 圖格時，應該會自動登入您的 SAP Hana 應用程式。
如需存取面板的詳細資訊，請參閱[存取面板簡介](../user-help/active-directory-saas-access-panel-introduction.md)。

## <a name="additional-resources"></a>其他資源

* [如何整合 SaaS 應用程式與 Azure Active Directory 的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/saphana-tutorial/tutorial_general_01.png
[2]: ./media/saphana-tutorial/tutorial_general_02.png
[3]: ./media/saphana-tutorial/tutorial_general_03.png
[4]: ./media/saphana-tutorial/tutorial_general_04.png

[100]: ./media/saphana-tutorial/tutorial_general_100.png

[200]: ./media/saphana-tutorial/tutorial_general_200.png
[201]: ./media/saphana-tutorial/tutorial_general_201.png
[202]: ./media/saphana-tutorial/tutorial_general_202.png
[203]: ./media/saphana-tutorial/tutorial_general_203.png

