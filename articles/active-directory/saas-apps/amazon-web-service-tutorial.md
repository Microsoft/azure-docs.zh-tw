---
title: 教學課程：Azure Active Directory 與 Amazon Web Services (AWS) 整合 | Microsoft Docs
description: 了解如何設定 Azure Active Directory 與 Amazon Web Services (AWS) 之間的單一登入。
services: active-directory
documentationCenter: na
author: jeevansd
manager: femila
ms.reviewer: joflore
ms.assetid: 7561c20b-2325-4d97-887f-693aa383c7be
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/14/2018
ms.author: jeedes
ms.openlocfilehash: 8ddcd66d0675603f4e130a9ca367cc4eed7353e7
ms.sourcegitcommit: 4ea0cea46d8b607acd7d128e1fd4a23454aa43ee
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/15/2018
ms.locfileid: "42145319"
---
# <a name="tutorial-azure-active-directory-integration-with-amazon-web-services-aws"></a>教學課程：Azure Active Directory 與 Amazon Web Services (AWS) 整合

在本教學課程中，您將了解如何整合 Amazon Web Services (AWS) 與 Azure Active Directory (Azure AD)。

Amazon Web Services (AWS) 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Amazon Web Services (AWS) 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Amazon Web Services (AWS) (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

![Amazon Web Services (AWS)](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_image.png)

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Amazon Web Services (AWS) 整合，您需要下列項目：

- Azure AD 訂用帳戶
- 已啟用 Amazon Web Services (AWS) 單一登入的訂用帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

> [!Note]
> 如果您想將多個 AWS 帳戶整合至一個 Azure 帳戶，以使用單一登入，請參閱[本文](https://docs.microsoft.com/azure/active-directory/active-directory-saas-aws-multi-accounts-tutorial)。

## <a name="scenario-description"></a>案例描述

在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。
本教學課程中說明的案例由二項主要的基本工作組成：

1. 從資源庫新增 Amazon Web Services (AWS)
2. 設定並測試 Azure AD 單一登入

## <a name="adding-amazon-web-services-aws-from-the-gallery"></a>從資源庫新增 Amazon Web Services (AWS)
若要設定 Amazon Web Services (AWS) 與 Azure AD 整合，您需要從資源庫將 Amazon Web Services (AWS) 新增到受控 SaaS App 清單。

**若要從資源庫新增 Amazon Web Services (AWS)，請執行下列步驟：**

1. 在 **[Azure 入口網站](https://portal.azure.com)** 的左方瀏覽窗格中，按一下 [Azure Active Directory] 圖示。

    ![Azure Active Directory 按鈕][1]

2. 瀏覽至 [企業應用程式]。 然後移至 [所有應用程式]。

    ![企業應用程式刀鋒視窗][2]

3. 若要新增新的應用程式，請按一下對話方塊頂端的 [新增應用程式] 按鈕。

    ![新增應用程式按鈕][3]

4. 在搜尋方塊中，鍵入 **Amazon Web Services (AWS)**，並從結果面板中選取 [Amazon Web Services (AWS)]，然後按一下 [新增] 按鈕新增應用程式。

    ![結果清單中的 Amazon Web Services (AWS)](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Amazon Web Services (AWS) 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Amazon Web Services (AWS) 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Amazon Web Services (AWS) 中的相關使用者之間建立連結關聯性。

在 Amazon Web Services (AWS) 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 Amazon Web Services (AWS) 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[建立 Azure AD 測試使用者](#create-an-azure-ad-test-user)** - 使用 Britta Simon 測試 Azure AD 單一登入。
3. **[建立 Amazon Web Services (AWS) 測試使用者](#create-an-amazon-web-services-aws-test-user)** - 使 Amazon Web Services (AWS) 中對應的 Britta Simon 連結到使用者在 Azure AD 中的代表項目。
4. **[指派 Azure AD 測試使用者](#assign-the-azure-ad-test-user)** - 讓 Britta Simon 能夠使用 Azure AD 單一登入。
5. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Amazon Web Services (AWS) 應用程式中設定單一登入。

**若要使用 Amazon Web Services (AWS) 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Amazon Web Services (AWS)] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。

    ![單一登入對話方塊](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_samlbase.png)

3. 在 [Amazon Web Services (AWS) 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已經與 Azure 預先整合。

    ![Amazon Web Services (AWS) 網域和 URL 單一登入資訊](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_url.png)

4. Amazon Web Services (AWS) 軟體應用程式預期要有特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入屬性](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_attribute.png)

5. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱  | 屬性值 | 命名空間 |
    | --------------- | --------------- | --------------- |
    | RoleSessionName | user.userprincipalname | https://aws.amazon.com/SAML/Attributes |
    | 角色            | user.assignedroles |  https://aws.amazon.com/SAML/Attributes |
    | SessionDuration             | 「根據您的需要來提供工作階段持續時間的值」 |  https://aws.amazon.com/SAML/Attributes |

    >[!TIP]
    >您需要設定在 Azure AD 中的使用者佈建，以從 AWS 主控台擷取所有角色。 請參閱下面的佈建步驟。

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入的新增](./media/amazon-web-service-tutorial/tutorial_attribute_04.png)

    ![設定單一登入的新增屬性](./media/amazon-web-service-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 在 [命名空間] 文字方塊中，輸入該資料列顯示的命名空間值。

    d. 按一下 [確定] 。

6. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_certificate.png)

7. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/amazon-web-service-tutorial/tutorial_general_400.png)

8. 在不同的瀏覽器視窗中，以系統管理員身分登入您的 Amazon Web Services (AWS) 公司網站。

9. 按一下 [AWS Home] \(AWS 首頁\)。

    ![設定單一登入首頁][11]

10. 按一下 [身分識別與存取管理] 。

    ![設定單一登入身分識別][12]

11. 按一下 [識別提供者]，然後按一下 [建立提供者]。

    ![設定單一登入提供者][13]

12. 在 [設定提供者]  對話方塊頁面上，執行下列步驟：

    ![設定單一登入對話方塊][14]

    a. 針對 [提供者類型]，選取 [SAML]。

    b. 在 [提供者名稱] 文字方塊中，鍵入提供者名稱 (例如：WAAD)。

    c. 若要上傳從 Azure 入口網站下載的**中繼資料檔案**，請按一下 [選擇檔案]。

    d. 按一下頁面底部的 [新增] 。

13. 在 [驗證提供者資訊] 對話方塊頁面上，按一下 [建立]。

    ![設定單一登入驗證][15]

14. 按一下 [Roles] \(角色\)，然後按一下 [Create role] \(建立角色\)。

    ![設定單一登入角色][16]

15. 在 [Create role] \(建立角色\) 頁面上，執行下列步驟：  

    ![設定單一登入信任][19]

    a. 在 [Select type of trusted entity] \(選取信任的實體類型\) 底下，選取 [SAML 2.0 federation] \(SAML 2.0 同盟\)。

    b. 在 [Choose a SAML 2.0 Provider] \(選擇 SAML 2.0 提供者\) 區段底下，選取您先前建立的 [SAML provider] \(SAML 提供者\) (例如：*WAAD*)

    c. 選取 [Allow programmatic and AWS Management Console access] \(允許透過程式設計方式和 AWS 管理主控台存取\)。
  
    d. 按一下 [Next: Permissions] \(下一步：權限\)。

16. 在 [附加權限原則] 對話方塊中，您不需要附加任何原則。 按一下 [下一步：檢閱]。  

    ![設定單一登入原則][33]

17. 在 [檢閱]  對話方塊上，執行下列步驟：

    ![設定單一登入檢閱][34]

    a. 在 [Role name] \(角色名稱\) 文字方塊中，輸入您的角色名稱。

    b. 在 [Role description] \(文字描述\) 文字方塊中，輸入描述。

    c. 按一下 [建立角色]。

    d. 建立所需數量的角色，並將它們對應至識別提供者。

18. 使用 AWS 服務帳戶認證，以透過 Azure AD 使用者佈建中的 AWS 帳戶來擷取角色。 若要這麼做，請開啟 AWS 主控台首頁。

19. 按一下 [服務] -> [Security,Identity& Compliance (安全性、身分識別和合規性)] -> [IAM]。

    ![從 AWS 帳戶擷取角色](./media/amazon-web-service-tutorial/fetchingrole1.png)

20. 選取 IAM 區段中的 [原則] 索引標籤。

    ![從 AWS 帳戶擷取角色](./media/amazon-web-service-tutorial/fetchingrole2.png)

21. 按一下 [Create policy] \(建立原則\) 來建立新原則，以從「Azure AD 使用者佈建」中的 AWS 帳戶擷取角色。

    ![建立新的原則](./media/amazon-web-service-tutorial/fetchingrole3.png)

22. 建立您自己的原則，以從 AWS 帳戶擷取所有角色，方法是執行下列步驟：

    ![建立新的原則](./media/amazon-web-service-tutorial/policy1.png)

    a. 在 [建立原則] 區段中按一下 [JSON] 索引標籤。

    b. 在原則文件中，新增下列 JSON。

    ```

    {

    "Version": "2012-10-17",

    "Statement": [

    {

    "Effect": "Allow",

    "Action": [

    "iam:ListRoles"

    ],

    "Resource": "*"

    }

    ]

    }

    ```

    c. 按一下 [檢閱原則] 按鈕以驗證原則。

    ![定義新的原則](./media/amazon-web-service-tutorial/policy5.png)

23. 執行下列步驟，以定義**新原則**：

    ![定義新的原則](./media/amazon-web-service-tutorial/policy2.png)

    a. 將 [原則名稱] 提供為 **AzureAD_SSOUserRole_Policy**。

    b. 您可以將**描述**提供給原則，因為**此原則將允許從 AWS 帳戶擷取角色**。

    c. 按一下 [建立原則] 按鈕。

24. 執行下列步驟，以在 AWS IAM 服務中建立新的使用者帳戶：

    a. 按一下 AWS IAM 主控台中的 [使用者] 導覽。

    ![定義新的原則](./media/amazon-web-service-tutorial/policy3.png)

    b. 按一下 [新增使用者] 按鈕，以建立新的使用者。

    ![新增使用者](./media/amazon-web-service-tutorial/policy4.png)

    c. 在 [新增使用者] 區段中，執行下列步驟：

    ![新增使用者](./media/amazon-web-service-tutorial/adduser1.png)

    * 將使用者名稱輸入為 **AzureADRoleManager**。

    * 在 [存取類型] 中，選取 [以程式設計方式存取] 選項。 因此，使用者可以叫用 API，以及從 AWS 帳戶擷取角色。

    * 按一下右下角的 [Next Permissions] (接下來的權限) 按鈕。

25. 現在，執行下列步驟，以建立此使用者的新原則：

    ![新增使用者](./media/amazon-web-service-tutorial/adduser2.png)

    a. 按一下 [Attach existing policies directly] (直接連接現有原則) 按鈕。

    b. 在篩選區段 **AzureAD_SSOUserRole_Policy** 中，搜尋新建立的原則。

    c. 選取 [原則]，然後按一下 [下一步: 評論] 按鈕。

26. 執行下列步驟，以檢閱所連接使用者的原則：

    ![新增使用者](./media/amazon-web-service-tutorial/adduser3.png)

    a. 檢閱使用者名稱、存取類型，以及對應至使用者的原則。

    b. 按一下右下角的 [建立使用者] 按鈕，以建立使用者。

27. 執行下列步驟，以下載使用者的使用者認證：

    ![新增使用者](./media/amazon-web-service-tutorial/adduser4.png)

    a. 複製使用者**存取金鑰識別碼**和**祕密存取金鑰**。

    b. 在 Azure AD 使用者佈建區段中輸入這些認證，以從 AWS 主控台擷取角色。

    c. 按一下底部的 [關閉] 按鈕。

28. 巡覽至 Azure AD 管理入口網站中 Amazon Web Services 應用程式的 [使用者佈建] 區段。

    ![新增使用者](./media/amazon-web-service-tutorial/provisioning.png)

29. 在 [用戶端祕密] 和 [祕密權杖] 欄位中，分別輸入 [存取金鑰] 和 [祕密]。

    ![新增使用者](./media/amazon-web-service-tutorial/provisioning1.png)

    a. 在 **clientsecret** 欄位中，輸入 AWS 使用者存取金鑰。

    b. 在 [祕密權杖] 欄位中，輸入 AWS 使用者祕密。

    c. 按一下 [測試連線] 按鈕，您應該可以成功測試此連線。

    d. 按一下頂端的 [儲存] 按鈕，以儲存設定。

30. 現在，請確定您開啟開關，然後按一下頂端的 [儲存] 按鈕，以在 [設定] 區段中 [開啟] [佈建狀態]。

    ![新增使用者](./media/amazon-web-service-tutorial/provisioning2.png)

### <a name="create-an-azure-ad-test-user"></a>建立 Azure AD 測試使用者

本節的目標是要在 Azure 入口網站中建立一個名為 Britta Simon 的測試使用者。

   ![建立 Azure AD 測試使用者][100]

**若要在 Azure AD 中建立測試使用者，請執行下列步驟：**

1. 在 Azure 入口網站的左窗格中，按一下 [Azure Active Directory] 按鈕。

    ![Azure Active Directory 按鈕](./media/amazon-web-service-tutorial/create_aaduser_01.png)

2. 若要顯示使用者清單，請移至 [使用者和群組]，然後按一下 [所有使用者]。

    ![[使用者和群組] 與 [所有使用者] 連結](./media/amazon-web-service-tutorial/create_aaduser_02.png)

3. 若要開啟 [使用者] 對話方塊，按一下 [所有使用者] 對話方塊頂端的 [新增]。

    ![[新增] 按鈕](./media/amazon-web-service-tutorial/create_aaduser_03.png)

4. 在 [使用者] 對話方塊中，執行下列步驟：

    ![[使用者] 對話方塊](./media/amazon-web-service-tutorial/create_aaduser_04.png)

    a. 在 [名稱] 方塊中，輸入 **BrittaSimon**。

    b. 在 [使用者名稱] 方塊中，輸入使用者 Britta Simon 的電子郵件地址。

    c. 選取 [顯示密碼] 核取方塊，然後記下 [密碼] 方塊中顯示的值。

    d. 按一下頁面底部的 [新增] 。

### <a name="create-an-amazon-web-services-aws-test-user"></a>建立 Amazon Web Services (AWS) 測試使用者

本節目標是在 Amazon Web Services (AWS) 中建立名為 Britta Simon 的使用者。 Amazon Web Services (AWS) 不需要在其系統中針對 SSO 建立使用者，因此您不需要在這裡執行任何動作。

### <a name="assign-the-azure-ad-test-user"></a>指派 Azure AD 測試使用者

在本節中，您會將 Amazon Web Services (AWS) 的存取權授與 Britta Simon，讓她能夠使用 Azure 單一登入。

![指派使用者角色][200]

**若要將 Britta Simon 指派給 Amazon Web Services (AWS)，請執行下列步驟：**

1. 在 Azure 入口網站中，開啟應用程式檢視，接著瀏覽至目錄檢視並移至 [企業應用程式]，然後按一下 [所有應用程式]。

    ![指派使用者][201]

2. 在應用程式清單中，選取 [Amazon Web Services (AWS)] 。

    ![[應用程式] 清單中的 Amazon Web Services (AWS) 連結](./media/amazon-web-service-tutorial/tutorial_amazonwebservices(aws)_app.png)  

3. 在左側功能表中，按一下 [使用者和群組]。

    ![[使用者和群組] 連結][202]

4. 按一下 [新增] 按鈕。 然後選取 [新增指派] 對話方塊上的 [使用者和群組]。

    ![[新增指派] 窗格][203]

5. 在 [使用者和群組] 對話方塊上，選取 [使用者] 清單中的 [Britta Simon]。

6. 按一下 [使用者和群組] 對話方塊上的 [選取] 按鈕。

7. 按一下 [新增指派] 對話方塊上的 [指派] 按鈕。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Amazon Web Services (AWS)] 圖格時，應該會自動登入您的 Amazon Web 服務 (AWS) 應用程式。
如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/amazon-web-service-tutorial/tutorial_general_01.png
[2]: ./media/amazon-web-service-tutorial/tutorial_general_02.png
[3]: ./media/amazon-web-service-tutorial/tutorial_general_03.png
[4]: ./media/amazon-web-service-tutorial/tutorial_general_04.png

[100]: ./media/amazon-web-service-tutorial/tutorial_general_100.png

[200]: ./media/amazon-web-service-tutorial/tutorial_general_200.png
[201]: ./media/amazon-web-service-tutorial/tutorial_general_201.png
[202]: ./media/amazon-web-service-tutorial/tutorial_general_202.png
[203]: ./media/amazon-web-service-tutorial/tutorial_general_203.png
[11]: ./media/amazon-web-service-tutorial/ic795031.png
[12]: ./media/amazon-web-service-tutorial/ic795032.png
[13]: ./media/amazon-web-service-tutorial/ic795033.png
[14]: ./media/amazon-web-service-tutorial/ic795034.png
[15]: ./media/amazon-web-service-tutorial/ic795035.png
[16]: ./media/amazon-web-service-tutorial/ic795022.png
[17]: ./media/amazon-web-service-tutorial/ic795023.png
[18]: ./media/amazon-web-service-tutorial/ic795024.png
[19]: ./media/amazon-web-service-tutorial/ic795025.png
[32]: ./media/amazon-web-service-tutorial/ic7950251.png
[33]: ./media/amazon-web-service-tutorial/ic7950252.png
[35]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning.png
[34]: ./media/amazon-web-service-tutorial/ic7950253.png
[36]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_securitycredentials.png
[37]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_securitycredentials_continue.png
[38]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_createnewaccesskey.png
[39]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_automatic.png
[40]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_testconnection.png
[41]: ./media/amazon-web-service-tutorial/tutorial_amazonwebservices_provisioning_on.png

