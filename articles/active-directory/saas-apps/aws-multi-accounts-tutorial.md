---
title: 教學課程：Azure Active Directory 與 Amazon Web Services (AWS) 整合以連接多個帳戶 | Microsoft Docs
description: 了解如何設定 Azure AD 與 Amazon Web Services (AWS) 的多個帳戶之間的單一登入。
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
ms.openlocfilehash: a7d77df4d6be1572d2076684cfa4702cb32b5ed6
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391907"
---
# <a name="tutorial-azure-active-directory-integration-with-multiple-amazon-web-services-aws-accounts"></a>教學課程：Azure Active Directory 與多個 Amazon Web Services (AWS) 帳戶整合

在本教學課程中，您將了解如何整合 Azure Active Directory (Azure AD) 與 Amazon Web Services (AWS) 的多個帳戶。

Amazon Web Services (AWS) 與 Azure AD 整合提供下列優點：

- 您可以在 Azure AD 中控制可存取 Amazon Web Services (AWS) 的人員。
- 您可以讓使用者使用他們的 Azure AD 帳戶自動登入 Amazon Web Services (AWS) (單一登入)。
- 您可以在 Azure 入口網站中集中管理您的帳戶。

如果您想要了解有關 SaaS 應用程式與 Azure AD 之整合的更多詳細資料，請參閱[什麼是搭配 Azure Active Directory 的應用程式存取和單一登入](../manage-apps/what-is-single-sign-on.md)。

![結果清單中的 Amazon Web Services (AWS)](./media/aws-multi-accounts-tutorial/amazonwebservice.png)

## <a name="prerequisites"></a>必要條件

若要設定 Azure AD 與 Amazon Web Services (AWS) 整合，您需要下列項目：

- Azure AD 的 Basic 或Premium 訂用帳戶
- 已啟用 Amazon Web Services (AWS) 多重單一登入的帳戶

> [!NOTE]
> 若要測試本教學課程中的步驟，我們不建議使用生產環境。

若要測試本教學課程中的步驟，您應該遵循這些建議：

- 除非必要，否則請勿使用生產環境。
- 如果您沒有 Azure AD 試用環境，您可以[取得一個月試用](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="scenario-description"></a>案例描述
在本教學課程中，您會在測試環境中測試 Azure AD 單一登入。 本教學課程中說明的案例由二項主要的基本工作組成：

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

    ![結果清單中的 Amazon Web Services (AWS)](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_addfromgallery.png)

5. 新增應用程式之後，移至 [屬性] 頁面，然後複製 [物件識別碼]。

    ![結果清單中的 Amazon Web Services (AWS)](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_properties.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>設定和測試 Azure AD 單一登入

在本節中，您會以名為 "Britta Simon" 的測試使用者為基礎，設定及測試與 Amazon Web Services (AWS) 搭配運作的 Azure AD 單一登入。

若要讓單一登入運作，Azure AD 必須知道 Amazon Web Services (AWS) 與 Azure AD 中互相對應的使用者。 換句話說，必須在 Azure AD 使用者和 Amazon Web Services (AWS) 中的相關使用者之間建立連結關聯性。

在 Amazon Web Services (AWS) 中，將 Azure AD 中**使用者名稱**的值指派為 **Username** 的值，以建立連結關聯性。

若要使用 Amazon Web Services (AWS) 來設定並測試 Azure AD 單一登入，您需要完成下列建置組塊：

1. **[設定 Azure AD 單一登入](#configure-azure-ad-single-sign-on)** - 讓您的使用者能夠使用此功能。
2. **[測試單一登入](#test-single-sign-on)**，驗證組態是否能運作。

### <a name="configure-azure-ad-single-sign-on"></a>設定 Azure AD 單一登入

在本節中，您會在 Azure 入口網站中啟用 Azure AD 單一登入，然後在您的 Amazon Web Services (AWS) 應用程式中設定單一登入。

**若要使用 Amazon Web Services (AWS) 設定 Azure AD 單一登入，請執行下列步驟：**

1. 在 Azure 入口網站的 [Amazon Web Services (AWS)] 應用程式整合頁面上，按一下 [單一登入]。

    ![設定單一登入連結][4]

2. 在 [單一登入] 對話方塊上，於 [模式] 選取 [SAML 登入]，以啟用單一登入。
 
    ![單一登入對話方塊](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_samlbase.png)

3. 在 [Amazon Web Services (AWS) 網域和 URL] 區段中，使用者不需要執行任何步驟，因為應用程式已經與 Azure 預先整合。

    ![Amazon Web Services (AWS) 網域和 URL 單一登入資訊](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_url.png)

4. Amazon Web Services (AWS) 軟體應用程式預期要有特定格式的 SAML 判斷提示。 設定此應用程式的下列宣告。 您可以在應用程式整合頁面的 [使用者屬性] 區段中管理這些屬性的值。 以下螢幕擷取畫面顯示上述的範例。

    ![設定單一登入屬性](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_attribute.png)    

5. 在 [單一登入] 對話方塊的 [使用者屬性] 區段中，如上圖所示設定 SAML 權杖屬性，然後執行下列步驟：

    | 屬性名稱  | 屬性值 | 命名空間 |
    | --------------- | --------------- | --------------- |
    | RoleSessionName | user.userprincipalname | https://aws.amazon.com/SAML/Attributes |
    | 角色            | user.assignedroles |  https://aws.amazon.com/SAML/Attributes |
    | SessionDuration             | 「根據您的需要來提供工作階段持續時間的值」 |  https://aws.amazon.com/SAML/Attributes |

    >[!TIP]
    >您需要設定在 Azure AD 中的使用者佈建，以從 AWS 主控台擷取所有角色。 請參閱下面的佈建步驟。

    a. 按一下 [新增屬性] 來開啟 [新增屬性] 對話方塊。

    ![設定單一登入的新增](./media/aws-multi-accounts-tutorial/tutorial_attribute_04.png)

    ![設定單一登入屬性](./media/aws-multi-accounts-tutorial/tutorial_attribute_05.png)

    b. 在 [名稱] 文字方塊中，輸入該資料列所顯示的屬性名稱。

    c. 在 [值] 清單中，選取該列所顯示的值。

    d. 在 [命名空間] 文字方塊中，輸入該資料列顯示的命名空間值。

    d. 按一下 [確定] 。

6. 在 [SAML 簽署憑證] 區段上，按一下 [中繼資料 XML]，然後將中繼資料檔案儲存在您的電腦上。

    ![憑證下載連結](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_certificate.png) 

7. 按一下 [儲存]  按鈕。

    ![設定單一登入儲存按鈕](./media/aws-multi-accounts-tutorial/tutorial_general_400.png)

8. 在不同的瀏覽器視窗中，以系統管理員身分登入您的 Amazon Web Services (AWS) 公司網站。

9. 按一下 [AWS Home] \(AWS 首頁\)。

    ![設定單一登入首頁][11]

10. 按一下 [IAM] (身分識別與存取管理)。

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

16. 在 [Attach Permissions Policies] \(附加權限原則\) 對話方塊上，按一下 [Next: Review] \(下一步：檢閱\)。  

    ![設定單一登入原則][33]

17. 在 [檢閱]  對話方塊上，執行下列步驟：

    ![設定單一登入檢閱][34]

    a. 在 [Role name] \(角色名稱\) 文字方塊中，輸入您的角色名稱。

    b. 在 [Role description] \(文字描述\) 文字方塊中，輸入描述。

    a. 按一下 [建立角色]。

    b. 建立所需數量的角色，並將它們對應至識別提供者。

18. 從目前的 AWS 帳戶登出，然後以您要設定 Azure AD 的單一登入的其他帳戶登入。

19. 執行步驟 9 到步驟 17，以建立您要為此帳戶設定的多個角色。 如果您有兩個以上的帳戶，請為所有帳戶執行相同的步驟，以建立其角色。

20. 在帳戶中建立所有角色後，角色會顯示在這些帳戶的 [角色] 清單中。

    ![角色設定](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_listofroles.png)

21. 對於跨所有帳戶的各個角色，我們必須擷取所有角色 ARN 和受信任的實體，因為我們需要手動將其與 Azure AD 應用程式對應。 

22. 按一下角色以複製 [角色 ARN] 和 [信任實體] 值。 對於您必須在 Azure AD 中建立的所有角色，您都需要這些值。

    ![角色設定](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_role_summary.png)

23. 對所有帳戶中的所有角色執行上述步驟，並以**角色 ARN,信任實體**的格式將其全部儲存在記事本中。

24. 在另一個視窗中開啟 [Azure AD Graph 總管](https://developer.microsoft.com/graph/graph-explorer)。

    a. 使用您租用戶的全域管理員/共同管理員認證來登入 [Graph 總管] 網站。

    b. 您必須有足夠的權限才能建立角色。 按一下 [修改權限] 以取得必要的權限。

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new9.png)

    c. 從清單中選取下列權限 (如果您還沒有)，然後按一下 [修改權限] 

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new10.png)

    d. 此時系統會要求您重新登入並接受此要求。 接受此要求後，您會重新登入 Graph 總管。

    e. 將版本下拉式清單變更為 **Beta**。 若要從您的租用戶擷取所有服務主體，請使用下列查詢：

     `https://graph.microsoft.com/beta/servicePrincipals`

    如果您使用多個目錄，則可以使用下列模式，其中包含您的主要網域：`https://graph.microsoft.com/beta/contoso.com/servicePrincipals`

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new1.png)

    f. 從所擷取的服務主體清單中，取得您需要修改的服務主體。 您也可以使用 Ctrl+F，從所有列出的服務主體中搜尋應用程式。 您可以透過您從 Azure AD 的 [屬性] 頁面複製的**物件識別碼**，使用下列查詢移至各自的「服務主體」。

    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`。

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new2.png)

    g. 從服務主體物件擷取 appRoles 屬性。

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new3.png)

    h. 現在，您必須為您的應用程式產生新角色。 

    i. 以下 JSON 是 appRoles 物件的範例。 請建立類似的物件，以新增您的應用程式所需的角色。

    ```
    {
    "appRoles": [
        {
            "allowedMemberTypes": [
                "User"
            ],
            "description": "msiam_access",
            "displayName": "msiam_access",
            "id": "7dfd756e-8c27-4472-b2b7-38c17fc5de5e",
            "isEnabled": true,
            "origin": "Application",
            "value": null
        },
        {
            "allowedMemberTypes": [
                "User"
            ],
            "description": "Admin,WAAD",
            "displayName": "Admin,WAAD",
            "id": "4aacf5a4-f38b-4861-b909-bae023e88dde",
            "isEnabled": true,
            "origin": "ServicePrincipal",
            "value": "arn:aws:iam::12345:role/Admin,arn:aws:iam::12345:saml-provider/WAAD"
        },
        {
            "allowedMemberTypes": [
                "User"
            ],
            "description": "Auditors,WAAD",
            "displayName": "Auditors,WAAD",
            "id": "bcad6926-67ec-445a-80f8-578032504c09",
            "isEnabled": true,
            "origin": "ServicePrincipal",
            "value": "arn:aws:iam::12345:role/Auditors,arn:aws:iam::12345:saml-provider/WAAD"
        }    ]
    }
    ```

    > [!Note]
    > 您只能在修補作業的 **msiam_access** 之後新增角色。 此外，您可以根據組織的需求新增任意數目的角色。 Azure AD 會在 SAML 回應中以宣告值的形式傳送這些角色的**值**。

    j. 返回 Graph 總管，並將方法從 **GET** 變更為 **PATCH**。 藉由將 appRoles 屬性更新為前述範例中的類似屬性，將服務主體物件修補成具有所需的角色。 按一下 [執行查詢] 以執行修補作業。 此時會出現成功訊息，確認 Amazon Web Services 應用程式的角色已建立。

    ![Graph 總管對話方塊](./media/aws-multi-accounts-tutorial/graph-explorer-new11.png)

25. 以更多角色修補服務主體之後，您即可將使用者/群組指派給各自的角色。 移至入口網站，然後瀏覽至 Amazon Web Services 應用程式，即可完成此作業。 按一下頂端的 [使用者和群組] 索引標籤。 

26. 建議您為每個 AWS 角色建立新群組，以便在該群組中指派這個角色。 請注意，這是一個群組與一個角色之間的一對一對應。 接著，您可以新增屬於該群組的成員。

27. 群組建立後，請選取群組並指派給應用程式。

    ![設定單一登入的新增](./media/aws-multi-accounts-tutorial/graph-explorer-new5.png)

> [!Note]
> 指派群組時，不支援巢狀群組。

28. 若要將角色指派給群組，請選取角色，然後按一下頁面底部的 [指派] 按鈕。

    ![設定單一登入的新增](./media/aws-multi-accounts-tutorial/graph-explorer-new6.png)

> [!Note]
> 請注意，您必須在 Azure 入口網站中重新整理您的工作階段，才能看到新角色。

### <a name="test-single-sign-on"></a>測試單一登入

在本節中，您會使用存取面板來測試您的 Azure AD 單一登入設定。

當您在存取面板中按一下 [Amazon Web Services (AWS)] 圖格時，應該會進入有選項可供選取角色的 Amazon Web Services (AWS) 應用程式頁面。

![設定單一登入的新增](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_test_screen.png)

您也可以驗證 SAML 回應，以查看以宣告的形式傳遞的角色。

![設定單一登入的新增](./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices(aws)_test_saml.png)

如需「存取面板」的詳細資訊，請參閱[存取面板簡介](../active-directory-saas-access-panel-introduction.md)。 

## <a name="additional-resources"></a>其他資源

* [如何與 Azure Active Directory 整合 SaaS 應用程式的教學課程清單](tutorial-list.md)
* [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/aws-multi-accounts-tutorial/tutorial_general_01.png
[2]: ./media/aws-multi-accounts-tutorial/tutorial_general_02.png
[3]: ./media/aws-multi-accounts-tutorial/tutorial_general_03.png
[4]: ./media/aws-multi-accounts-tutorial/tutorial_general_04.png

[100]: ./media/aws-multi-accounts-tutorial/tutorial_general_100.png

[200]: ./media/aws-multi-accounts-tutorial/tutorial_general_200.png
[201]: ./media/aws-multi-accounts-tutorial/tutorial_general_201.png
[202]: ./media/aws-multi-accounts-tutorial/tutorial_general_202.png
[203]: ./media/aws-multi-accounts-tutorial/tutorial_general_203.png
[11]: ./media/aws-multi-accounts-tutorial/ic795031.png
[12]: ./media/aws-multi-accounts-tutorial/ic795032.png
[13]: ./media/aws-multi-accounts-tutorial/ic795033.png
[14]: ./media/aws-multi-accounts-tutorial/ic795034.png
[15]: ./media/aws-multi-accounts-tutorial/ic795035.png
[16]: ./media/aws-multi-accounts-tutorial/ic795022.png
[17]: ./media/aws-multi-accounts-tutorial/ic795023.png
[18]: ./media/aws-multi-accounts-tutorial/ic795024.png
[19]: ./media/aws-multi-accounts-tutorial/ic795025.png
[32]: ./media/aws-multi-accounts-tutorial/ic7950251.png
[33]: ./media/aws-multi-accounts-tutorial/ic7950252.png
[35]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_provisioning.png
[34]: ./media/aws-multi-accounts-tutorial/ic7950253.png
[36]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_securitycredentials.png
[37]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_securitycredentials_continue.png
[38]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_createnewaccesskey.png
[39]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_provisioning_automatic.png
[40]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_provisioning_testconnection.png
[41]: ./media/aws-multi-accounts-tutorial/tutorial_amazonwebservices_provisioning_on.png

