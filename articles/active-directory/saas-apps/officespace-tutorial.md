---
title: 教學課程：Azure Active Directory 與 OfficeSpace Software 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 OfficeSpace Software 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 95d8413f-db98-4e2c-8097-9142ef1af823
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2017
ms.author: jeedes
ms.openlocfilehash: 7162645f41d26f1496bdec9c4d694e749bfec6a1
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39449524"
---
# <a name="tutorial-azure-active-directory-integration-with-officespace-software"></a>教學課程：Azure Active Directory 與 OfficeSpace Software 整合

在本教學課程中，您將了解如何整合 OfficeSpace Software 與 Azure Active Directory (Azure AD)。

OfficeSpace Software 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 OfficeSpace Software 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 OfficeSpace Software (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 OfficeSpace Software 整合，您需要以下項目：

- Azure AD 訂用帳戶
- 已啟用 OfficeSpace Software 單一登入功能的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫加入 OfficeSpace Software
1. 設定並測試 Azure AD 單一登入

## <a name="adding-officespace-software-from-the-gallery"></a>從資源庫加入 OfficeSpace Software
若要設定 OfficeSpace Software 與 Azure AD 整合，您需要從資源庫將 OfficeSpace Software 新增到受控 SaaS 應用程式清單。

**若要從資源庫新增 OfficeSpace Software，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。 

    ![Azure Active Directory 按鈕][1]

1. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]
    
1. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

1. 在搜尋方塊中，輸入 **OfficeSpace Software**，從結果面板中選取 [OfficeSpace Software]，然後按一下 [新增] 按鈕以新增應用程式。

    ![結果清單中的 OfficeSpace Software](./media/officespace-tutorial/tutorial_officespace_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 OfficeSpace Software 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 OfficeSpace Software 與 Azure AD 中互相對應的使用者。 換句話說，Azure AD 使用者和 OfficeSpace Software 中的相關使用者之間必須建立連結關聯性。

在 OfficeSpace Software 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 OfficeSpace Software 設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
1. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
1. **[建立 OfficeSpace Software 測試使用者](#create-a-officespace-software-test-user)** - 使 OfficeSpace Software 中對應的 Britta Simon 連結到該使用者在 Azure AD 中的代表項目。
1. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
1. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會於 Azure 入口網站啟用 Azure AD 單一登入，然後在您的 OfficeSpace Software 應用程式中設定單一登入。

**若要使用 OfficeSpace Software 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [OfficeSpace Software] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

1. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/officespace-tutorial/tutorial_officespace_samlbase.png)

1. 在 [OfficeSpace Software 網域及 URL] 區段上，執行下列步驟：

    ![OfficeSpace Software 網域與 URL 單一登入資訊](./media/officespace-tutorial/tutorial_officespace_url.png)

    a. 在 [登入 URL] 文字方塊中，使用下列模式輸入 URL︰ `https://<company name>.officespacesoftware.com/users/sign_in/saml`

    b. 在 [識別碼] 文字方塊中，使用下列模式輸入 URL： `<company name>.officespacesoftware.com`

    > [!NOTE] 
    > 這些都不是真正的值。 使用實際的「登入 URL」及「識別碼」來更新這些值。 請連絡 [OfficeSpace Software 用戶端支援小組](mailto:support@officespacesoftware.com)以取得這些值。 

1. OfficeSpace Software 應用程式會預期要有特定格式的 SAML 判斷提示。 請設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。
    
    ![設定屬性](./media/officespace-tutorial/tutorial_officespace_attribute.png)

1. 在 [單一登入] 對話方塊內的 [使用者屬性] 區段中，選取 [user.mail] 當作 [使用者識別碼]，並在下表顯示的每個列上執行下列步驟：
    
    | 屬性名稱 | 屬性值 |
    | --- | --- |    
    | 電子郵件 | user.mail |
    | name | user.displayname |
    | first_name | user.givenname |
    | last_name | user.surname |

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定新增 ](./media/officespace-tutorial/tutorial_attribute_04.png)

    ![設定屬性](./media/officespace-tutorial/tutorial_attribute_05.png)
    
    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。
    
    c. 在 [值] 清單中，選取該列所顯示的值。
    
    d. 按一下 [確定]。
 
1. 在 [SAML 簽署憑證] 區段上，複製憑證的 [指紋] 值。

    ![憑證下載連結](./media/officespace-tutorial/tutorial_officespace_certificate.png) 

1. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/officespace-tutorial/tutorial_general_400.png)

1. 在 [OfficeSpace Software 組態] 區段上，按一下 [設定 OfficeSpace Software] 以開啟 [設定登入] 視窗。 從 [快速參考] 區段中複製 [登入 URL] 和 [SAML 單一登入服務 URL]。

    ![OfficeSpace Software 設定](./media/officespace-tutorial/tutorial_officespace_configure.png) 

1. 在不同的 Web 瀏覽器視窗中，以系統管理員身分登入您的 OfficeSpace Software 租用戶。

1. 移至 [設定]，然後按一下 [連接器]。

    ![在應用程式端設定單一登入](./media/officespace-tutorial/tutorial_officespace_002.png)

1. 按一下 [SAML 驗證]。

    ![在應用程式端設定單一登入](./media/officespace-tutorial/tutorial_officespace_003.png)

1. 在 [SAML 驗證]  區段中，執行下列步驟：

    ![在應用程式端設定單一登入](./media/officespace-tutorial/tutorial_officespace_004.png)

    a. 在 [登出提供者 url] 文字方塊中，貼上您從 Azure 入口網站複製的 [登出 URL] 值。

    b. 在 [用戶端 idp 目標 url] 文字方塊中，貼上您從 Azure 入口網站複製的 [SAML 單一登入服務 URL] 值。

    c. 將您從 Azure 入口網站複製的 [指紋] 值，貼到 [用戶端 IDP 憑證指紋] 文字方塊。 

    d. 按一下 [儲存設定] 。


> [!TIP]
> 現在，當您設定此應用程式時，在 [Azure 入口網站](https://portal.azure.com)內即可閱讀這些指示的簡要版本！  從 [Active Directory] > [企業應用程式] 區段新增此應用程式之後，只要按一下 [單一登入] 索引標籤，即可透過底部的 [組態] 區段存取內嵌的文件。 您可以從以下連結閱讀更多有關內嵌文件功能的資訊：[Azure AD 內嵌文件]( https://go.microsoft.com/fwlink/?linkid=845985)
> 

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/officespace-tutorial/create_aaduser_01.png)

1. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/officespace-tutorial/create_aaduser_02.png)

1. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/officespace-tutorial/create_aaduser_03.png)

1. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/officespace-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。
 
### <a name="create-a-officespace-software-test-user"></a>建立 OfficeSpace Software 測試使用者

本節目標是在 OfficeSpace Software 中建立名為 Britta Simon 的使用者。 OfficeSpace Software 支援預設啟用的 Just-In-Time 佈建。

在這一節沒有您需要進行的動作項目。 嘗試存取 OfficeSpace Software 時，如果使用者還不存在，就會建立新使用者。

> [!NOTE]
> 如果您需要手動建立使用者，您需要連絡 [OfficeSpace Software 支援小組](mailto:support@officespacesoftware.com)。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 OfficeSpace Software 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200] 

**若要指派 Britta Simon 到 OfficeSpace Software，請執行以下步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201] 

1. 在應用程式清單中，選取 [OfficeSpace Software]。

    ![應用程式清單中的 OfficeSpace Software 連結](./media/officespace-tutorial/tutorial_officespace_app.png)  

1. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

1. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

1. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

1. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

1. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。
    
### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在 [存取面板] 中按一下 [OfficeSpace Software] 圖格時，您應該會自動登入您的 OfficeSpace Software 應用程式。

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)



<!--Image references-->

[1]: ./media/officespace-tutorial/tutorial_general_01.png
[2]: ./media/officespace-tutorial/tutorial_general_02.png
[3]: ./media/officespace-tutorial/tutorial_general_03.png
[4]: ./media/officespace-tutorial/tutorial_general_04.png

[100]: ./media/officespace-tutorial/tutorial_general_100.png

[200]: ./media/officespace-tutorial/tutorial_general_200.png
[201]: ./media/officespace-tutorial/tutorial_general_201.png
[202]: ./media/officespace-tutorial/tutorial_general_202.png
[203]: ./media/officespace-tutorial/tutorial_general_203.png

