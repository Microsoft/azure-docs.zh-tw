---
title: 教學課程：Azure Active Directory 與 Symantec Web Security Service (WSS) 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Symantec Web Security Service (WSS) 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: d6e4d893-1f14-4522-ac20-0c73b18c72a5
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2017
ms.author: jeedes
ms.openlocfilehash: dbf21c7c22a9b3273a65f7e186a2ac02ccae6ba2
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39436202"
---
# <a name="tutorial-azure-active-directory-integration-with-symantec-web-security-service-wss"></a>教學課程：Azure Active Directory 與 Symantec Web Security Service (WSS) 整合

在本教學課程中，您將學習如何整合 Symantec Web Security Service (WSS) 帳戶與 Azure Active Directory (Azure AD) 帳戶，讓 WSS 可使用 SAML 驗證來驗證 Azure AD 中所佈建的使用者，並強制執行使用者或群組層級的原則規則。

Symantec Web Security Service (WSS) 與 Azure AD 整合提供下列優點：

- 從 Azure AD 入口網站管理您的 WSS 帳戶使用的所有使用者和群組。 

- 讓使用者可以使用其 Azure AD 認證在 WSS 中驗證其自身。

- 強制執行 WSS 帳戶中所定義的使用者和群組層級原則規則。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Symantec Web Security Service (WSS) 整合，您需要下列項目：

- Azure AD 訂用帳戶
- Symantec Web Security Service (WSS) 帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用目前正用於生產用途的 WSS 帳戶。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用目前正用於生產用途的 WSS 帳戶來進行這項測試。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會將 Azure AD 設定為能夠使用 Azure AD 帳戶中所定義的使用者認證來對 WSS 進行單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Symantec Web Security Service (WSS) 應用程式
1. 設定並測試 Azure AD 單一登入

## <a name="adding-symantec-web-security-service-wss-from-the-gallery"></a>從資源庫新增 Symantec Web Security Service (WSS)
若要設定 Symantec Web Security Service (WSS) 與 Azure AD 整合，您需要從資源庫將 Symantec Web Security Service (WSS) 新增到受控 SaaS App 清單。

**若要從資源庫新增 Symantec Web Security Service (WSS)，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **Symantec Web Security Service (WSS)**，從結果面板選取 [Symantec Web Security Service (WSS)]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 Symantec Web Security Service (WSS)](./media/symantec-tutorial/tutorial_symantecwebsecurityservicewss_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Symantec Web Security Service (WSS) 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Symantec Web Security Service (WSS) 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Symantec Web Security Service (WSS) 中的相關使用者之間建立連結關聯性。

在 Symantec Web Security Service (WSS) 中，將**使用者名稱**的值指派為 Azure AD 中**使用者名稱**的值，以建立連結關聯性。

若要使用 Symantec Web Security Service (WSS) 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 Symantec Web Security Service (WSS) 測試使用者](#create-a-symantec-web-security-service-wss-test-user)** - 使 Symantec Web Security Service (WSS) 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Symantec Web Security Service (WSS) 應用程式中設定單一登入。

**若要使用 Symantec Web Security Service (WSS) 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Symantec Web Security Service]\(WSS\) 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/symantec-tutorial/tutorial_symantecwebsecurityservicewss_samlbase.png)

1. 在 [Symantec Web Security Service (WSS) 網域與 URL] 區段上，執行下列步驟：

    ![Symantec Web Security Service (WSS) 的網域和 URL 單一登入資訊](./media/symantec-tutorial/tutorial_symantecwebsecurityservicewss_url.png)

    a. 在 [識別碼] 文字方塊中，輸入 URL：`https://saml.threatpulse.net:8443/saml/saml_realm`

    b. 在 [回覆 URL] 文字方塊中，輸入 URL：`https://saml.threatpulse.net:8443/saml/saml_realm/bcsamlpost`

    > [!NOTE]
    > 如果 [識別碼] 和 [回覆 URL] 的值因為某些原因而沒有作用，請連絡 [Symantec Web Security Service (WSS) 客戶支援小組](https://www.symantec.com/contact-us)。

1. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/symantec-tutorial/tutorial_symantecwebsecurityservicewss_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/symantec-tutorial/tutorial_general_400.png)
    
1. 若要在 Symantec Web Security Service (WSS) 端設定單一登入，請參閱 WSS 線上文件。 所下載的**中繼資料 XML** 檔案必須匯入到 WSS 入口網站。 如果您需要協助進行 WSS 入口網站設定，請連絡 [Symantec Web Security Service (WSS) 支援小組](https://www.symantec.com/contact-us)。

> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/symantec-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/symantec-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/symantec-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/symantec-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-symantec-web-security-service-wss-test-user"></a>建立 Symantec Web Security Service (WSS) 測試使用者

在本節中，您要在 Symantec Web Security Service (WSS) 中建立名為 Britta Simon 的使用者。 您可以在 WSS 入口網站中手動建立對應的使用者名稱，您也可以等候幾分鐘 (~ 15 分鐘)，讓 Azure AD 中佈建的使用者/群組同步處理至 WSS 入口網站。 您必須先建立和啟動使用者，然後才能使用單一登入。 將用來瀏覽網站的使用者電腦公用 IP 位址也必須佈建到 Symantec Web Security Service (WSS) 入口網站中。

> [!NOTE]
> 請[按一下這裡](http://www.bing.com/search?q=my+ip+address&qs=AS&pq=my+ip+a&sc=8-7&cvid=29A720C95C78488CA3F9A6BA0B3F98C5&FORM=QBLH&sp=1)以取得您機器的公用 IP 位址。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Symantec Web Security Service (WSS) 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要將 Britta Simon 指派給 Symantec Web Security Service (WSS)，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [Symantec Web Security Service (WSS)]。

    ![應用程式清單中的 [Symantec Web Security Service (WSS)] 連結](./media/symantec-tutorial/tutorial_symantecwebsecurityservicewss_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，因為您已將 WSS 帳戶設定為要使用 Azure AD 來進行 SAML 驗證，您將會測試單一登入功能。

在將網頁瀏覽器設定為將流量 Proxy 處理至 WSS 後，如果您開啟網頁瀏覽器並嘗試瀏覽至網站，則系統會將您重新導向至 Azure 的登入頁面。 請輸入已佈建到 Azure AD 之測試使用者 (也就是 BrittaSimon) 的認證以及相關聯的密碼。 通過驗證之後，您就可以瀏覽至您選擇的網站。 如果您在 WSS 端建立原則規則以阻止 BrittaSimon 瀏覽至特定網站，則當您嘗試以 BrittaSimon 這位使用者的身分瀏覽至該網站時，您應該會看見 WSS 封鎖頁面。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/symantec-tutorial/tutorial_general_01.png
[2]: ./media/symantec-tutorial/tutorial_general_02.png
[3]: ./media/symantec-tutorial/tutorial_general_03.png
[4]: ./media/symantec-tutorial/tutorial_general_04.png

[100]: ./media/symantec-tutorial/tutorial_general_100.png

[200]: ./media/symantec-tutorial/tutorial_general_200.png
[201]: ./media/symantec-tutorial/tutorial_general_201.png
[202]: ./media/symantec-tutorial/tutorial_general_202.png
[203]: ./media/symantec-tutorial/tutorial_general_203.png

