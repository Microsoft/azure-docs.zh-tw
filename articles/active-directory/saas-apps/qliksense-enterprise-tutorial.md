---
title: 教學課程：Azure Active Directory 與 Qlik Sense Enterprise 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Qlik Sense Enterprise 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 8c27e340-2b25-47b6-bf1f-438be4c14f93
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/06/2018
ms.author: jeedes
ms.openlocfilehash: a8816451b45171e0ba8cbd7acc937201c587c481
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39627945"
---
# <a name="tutorial-azure-active-directory-integration-with-qlik-sense-enterprise"></a>教學課程：Azure Active Directory 與 Qlik Sense Enterprise 整合

在本教學課程中，您會了解如何整合 Qlik Sense Enterprise 與 Azure Active Directory (Azure AD)。

Qlik Sense Enterprise 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Qlik Sense Enterprise 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Qlik Sense Enterprise (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Qlik Sense Enterprise 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Qlik Sense Enterprise 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以在這裡取得一個月試用：[試用供應項目](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Qlik Sense Enterprise
2. 設定並測試 Azure AD 單一登入

## <a name="adding-qlik-sense-enterprise-from-the-gallery"></a>從資源庫新增 Qlik Sense Enterprise
若要設定將 Qlik Sense Enterprise 整合到 Azure AD 中，您需要從資源庫將 Qlik Sense Enterprise 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 Qlik Sense Enterprise，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，輸入 **Qlik Sense Enterprise**，從結果面板中選取 [Qlik Sense Enterprise]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Qlik Sense Enterprise](./media/qliksense-enterprise-tutorial/tutorial_qliksense-enterprise_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者身分，使用 Qlik Sense Enterprise 設定及測試 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Qlik Sense Enterprise 與 Azure AD 中互相對應的使用者。 換句話說，必須建立 Azure AD 使用者和 Qlik Sense Enterprise 中相關使用者之間的連結關聯性。

在 Qlik Sense Enterprise 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要設定及測試與 Qlik Sense Enterprise 搭配運作的 Azure AD 單一登入，您需要完成下列構成要素：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Qlik Sense Enterprise 測試使用者](#create-a-qlik-sense-enterprise-test-user)** - 使 Qlik Sense Enterprise 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會於 Azure 入口網站啟用 Azure AD 單一登入，然後在您的 Qlik Sense Enterprise 應用程式中設定單一登入。

**若要使用 Qlik Sense Enterprise 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Qlik Sense Enterprise] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/qliksense-enterprise-tutorial/tutorial_qliksense-enterprise_samlbase.png)

3. 在 [Qlik Sense Enterprise 網域與 URL] 區段中，執行下列步驟：

    ![Qlik Sense Enterprise 網域與 URL 單一登入資訊](./media/qliksense-enterprise-tutorial/tutorial_qliksense-enterprise_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<Qlik Sense Fully Qualifed Hostname>:4443/azure/hub`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL：
    | |
    |--|
    | `https://<Qlik Sense Fully Qualifed Hostname>.qlikpoc.com`|
    | `https://<Qlik Sense Fully Qualifed Hostname>.qliksense.com`|
    | |

    c. 在 [回覆 URL] 文字方塊中，以下列模式輸入 URL：

    `https://<Qlik Sense Fully Qualifed Hostname>:4443/samlauthn/`

    > [!NOTE]
    > 這些都不是真正的值。 請使用實際的「登入 URL」、「識別碼」和「回覆 URL」來更新這些值 (本教學課程稍後會說明)，或連絡 [Qlik Sense Enterprise 用戶端支援小組](https://www.qlik.com/us/services/support)以取得這些值。

4. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/qliksense-enterprise-tutorial/tutorial_qliksense-enterprise_certificate.png)

5. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/qliksense-enterprise-tutorial/tutorial_general_400.png)

6. 準備同盟中繼資料 XML 檔案，以便將該檔案上傳至 Qlik Sense 伺服器。

    > [!NOTE]
    > 將 IdP 中繼資料上傳至 Qlik Sense 伺服器之前，必須先編輯此檔案才能移除資訊，以確保 Azure AD 與 Qlik Sense 伺服器之間的運作正常。

    ![QlikSense][qs24]

    a. 在文字編輯器中開啟從 Azure 入口網站下載的 FederationMetaData.xml 檔案。

    b. 搜尋 **RoleDescriptor** 值。  共會有四個項目 (兩對開頭和結尾元素標籤)。

    c. 從檔案中刪除 RoleDescriptor 標籤與其間的所有資訊。

    d. 將此檔案存放在附近，以便稍後用於本文件。

7. 以可建立虛擬 Proxy 組態的使用者身分，瀏覽至 Qlik Sense Qlik 管理主控台 (QMC)。

8. 在 QMC 中，按一下 [虛擬Proxy] 功能表項目。

    ![QlikSense][qs6]

9. 按一下畫面底部的 [新建] 按鈕。

    ![QlikSense][qs7]

10. 虛擬 Proxy 編輯畫面隨即出現。  畫面右邊的功能表會顯示組態選項。

    ![QlikSense][qs9]

11. 勾選 [識別] 功能表選項後，輸入 Azure 虛擬 Proxy 組態的識別資訊。

    ![QlikSense][qs8]  

    a. [說明] 欄位是虛擬 Proxy 組態的易記名稱。  輸入說明的值。

    b. [前置詞] 欄位可識別虛擬 Proxy 端點，以便連接到採用 Azure AD 單一登入的 Qlik Sense。  輸入此虛擬 Proxy 的獨特前置詞名稱。

    c. [工作階段閒置逾時 (分鐘)] 是指透過此虛擬 Proxy 連接的逾時值。

    d. [工作階段 cookie 標頭名稱] 是 cookie 名稱，用於儲存使用者在成功驗證後收到的 Qlik Sense 工作階段識別碼。  此名稱必須是唯一的。

12. 按一下 [驗證] 功能表選項，讓它立即可見。  [驗證] 畫面隨即出現。

    ![QlikSense][qs10]

    a. [匿名存取模式] 下拉式清單可讓您決定匿名使用者是否可透過虛擬 Proxy 存取 Qlik Sense。  預設選項是 [沒有匿名使用者]。

    b. [驗證方法] 下拉式清單可讓您決定虛擬 Proxy 將使用的驗證配置。  從下拉式清單選取 [SAML]。  此時會顯示更多選項。

    c. 在 [SAML 主機 URI] 欄位中，輸入使用者必須輸入才可透過此 SAML 虛擬 Proxy 存取 Qlik Sense 的主機名稱。  主機名稱是 Qlik Sense 伺服器的 URI。

    d. 在 [SAML 實體識別碼] 中，輸入與 [SAML 主機 URI] 欄位相同的值。

    e. [SAML IdP 中繼資料] 是稍早在 [從 Azure AD 組態編輯同盟中繼資料] 區段中編輯的檔案。  **上傳 IdP 中繼資料之前，必須先編輯此檔案**才能移除資訊，以確保 Azure AD 與 Qlik Sense 伺服器之間的運作正常。  **如果檔案尚未進行編輯，請參閱上述指示。**  如果檔案已經過編輯，請按一下 [瀏覽] 按鈕，然後選取已編輯的中繼資料檔案，將它上傳至虛擬 Proxy 組態。

    f. 輸入 SAML 屬性的屬性名稱或結構描述參考，代表 Azure AD 將傳送至 Qlik Sense 伺服器的 **UserID**。  在設定後的 Azure 應用程式畫面中可取得結構描述參考資訊。  若要使用名稱屬性，請輸入 `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`。

    g. 輸入**使用者目錄**的值，此值會在使用者透過 Azure AD 向 Qlik Sense 伺服器進行驗證時附加至使用者。  硬式編碼值必須以**方括號 []** 括住。  若要使用 Azure AD SAML 判斷提示中傳送的屬性，請在此文字方塊中輸入屬性名稱 (**不需**方括號)。

    h. [SAML 簽章演算法] 可設定虛擬 Proxy 組態的服務提供者 (在此例中是 Qlik Sense 伺服器) 憑證簽署。  如果 Qlik Sense 伺服器使用透過 Microsoft Enhanced RSA 和 AES 密碼編譯提供者產生的受信任憑證，請將 SAML 簽署演算法變更為 **SHA-256**。

    i. [SAML 屬性對應] 區段可讓其他屬性 (如群組) 傳送到 Qlik Sense，以便用於安全性規則。

13. 按一下 [負載平衡] 功能表選項，讓它立即可見。  [負載平衡] 畫面隨即出現。

    ![QlikSense][qs11]

14. 按一下 [新增伺服器節點] 按鈕，選取 Qlik Sense 為了達到負載平衡而會將工作階段傳送至的引擎節點，然後按一下 [新增] 按鈕。

    ![QlikSense][qs12]

15. 按一下 [進階] 功能表選項，讓它立即可見。 [進階] 畫面隨即出現。

    ![QlikSense][qs13]

    [主機允許清單] 會識別在連接到 Qlik Sense 伺服器時所接受的主機名稱。  **輸入連接到 Qlik Sense 伺服器時使用者會指定的主機名稱。** 主機名稱是與 SAML 主機 uri 相同的值 (不含 https://)。

16. 按一下 [套用] 按鈕。

    ![QlikSense][qs14]

17. 按一下 [確定] 以接受警告訊息，該訊息指出將重新啟動連結至虛擬 Proxy 的 Proxy。

    ![QlikSense][qs15]

18. 畫面右側會出現 [相關聯的項目] 功能表。  按一下 [Proxy] 功能表選項。

    ![QlikSense][qs16]

19. Proxy 畫面隨即出現。  按一下底部的 [連結] 按鈕，將 Proxy 連結到虛擬 Proxy。

    ![QlikSense][qs17]

20. 選取將會支援此虛擬 Proxy 連線的 Proxy 節點，然後按一下 [連結] 按鈕。  連結之後，Proxy 會列在相關聯的 Proxy 之下。

    ![QlikSense][qs18]
  
    ![QlikSense][qs19]

21. 大約五到十秒後，[重新整理 QMC] 訊息會出現。  按一下 [重新整理 QMC] 按鈕。

    ![QlikSense][qs20]

22. 當 QMC 重新整理時，按一下 [虛擬 Proxy]  功能表項目。 新的 SAML 虛擬 Proxy 項目會列在螢幕上的表格中。  按一下此虛擬 Proxy 項目。

    ![QlikSense][qs51]

23. 畫面底部的 [下載 SP 中繼資料] 按鈕將會啟用。  按一下 [下載 SP 中繼資料] 按鈕，將中繼資料儲存至檔案。

    ![QlikSense][qs52]

24. 開啟 SP 中繼資料檔案。  觀察 **entityID** 項目和 **AssertionConsumerService** 項目。  這些值相當於 Azure AD 應用程式組態中的 [識別碼]、[登入 URL] 和 [回覆 URL]。 如果這些值不相符，請將其貼在 Azure AD 應用程式組態中的 [Qlik Sense Enterprise 網域與 URL] 區段，然後您應該在 Azure AD App 組態精靈中取代這些值。

    ![QlikSense][qs53]

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

   ![Azure Active Directory 按鈕](./media/qliksense-enterprise-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

   ![[使用者和群組] 與 [所有使用者] 連結](./media/qliksense-enterprise-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

   ![[新增] 按鈕](./media/qliksense-enterprise-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

   ![[使用者] 對話方塊](./media/qliksense-enterprise-tutorial/create_aaduser_04.png)

   a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

   b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

   c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

   d. 按一下頁面底部的 [新增] 。

### <a name="create-a-qlik-sense-enterprise-test-user"></a>建立 Qlik Sense Enterprise 測試使用者

在本節中，您要在 Qlik Sense Enterprise 中建立名為 Britta Simon 的使用者。 請與 [Qlik Sense Enterprise 用戶端支援小組](https://www.qlik.com/us/services/support)合作，以在 Qlik Sense Enterprise 平台中新增使用者。 您必須先建立和啟動使用者，然後才能使用單一登入。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Qlik Sense Enterprise 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Qlik Sense Enterprise，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [Qlik Sense Enterprise] 。

    ![應用程式清單中的 Qlik Sense Enterprise 連結](./media/qliksense-enterprise-tutorial/tutorial_qliksense-enterprise_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Qlik Sense Enterprise] 圖格時，應該會自動登入您的 Qlik Sense Enterprise 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/qliksense-enterprise-tutorial/tutorial_general_01.png
[2]: ./media/qliksense-enterprise-tutorial/tutorial_general_02.png
[3]: ./media/qliksense-enterprise-tutorial/tutorial_general_03.png
[4]: ./media/qliksense-enterprise-tutorial/tutorial_general_04.png

[100]: ./media/qliksense-enterprise-tutorial/tutorial_general_100.png

[200]: ./media/qliksense-enterprise-tutorial/tutorial_general_200.png
[201]: ./media/qliksense-enterprise-tutorial/tutorial_general_201.png
[202]: ./media/qliksense-enterprise-tutorial/tutorial_general_202.png
[203]: ./media/qliksense-enterprise-tutorial/tutorial_general_203.png

[qs6]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_06.png
[qs7]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_07.png
[qs8]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_08.png
[qs9]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_09.png
[qs10]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_10.png
[qs11]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_11.png
[qs12]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_12.png
[qs13]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_13.png
[qs14]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_14.png
[qs15]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_15.png
[qs16]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_16.png
[qs17]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_17.png
[qs18]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_18.png
[qs19]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_19.png
[qs20]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_20.png
[qs24]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_24.png
[qs51]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_51.png
[qs52]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_52.png
[qs53]: ./media/qliksense-enterprise-tutorial/tutorial_qliksenseenterprise_53.png