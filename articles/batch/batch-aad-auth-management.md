---
title: 使用 Azure Active Directory 驗證 Batch Management 解決方案 | Microsoft Docs
description: 使用 Azure Resource Manager 和 Batch 資源提供者建置的應用程式會使用 Azure AD 進行驗證。
services: batch
documentationcenter: .net
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: batch
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: big-compute
ms.date: 04/27/2017
ms.author: danlep
ms.openlocfilehash: a52dbdad3842760c447e22f4274ab944750840e7
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816781"
---
# <a name="authenticate-batch-management-solutions-with-active-directory"></a>使用 Active Directory 驗證 Batch Management 解決方案

呼叫 Azure Batch Management 服務的應用程式會使用 [Azure Active Directory][aad_about] (Azure AD) 進行驗證。 Azure AD 是 Microsoft 的多租用戶雲端型目錄和身分識別管理服務。 Azure 本身會使用 Azure AD 來驗證其客戶、服務管理員和組織的使用者。

Batch 管理 .NET 程式庫會公開使用 Batch 帳戶、帳戶金鑰、應用程式和應用程式封裝的類型。 Batch Management .NET 程式庫是 Azure 資源提供者用戶端，並與 [Azure Resource Manager][resman_overview] 搭配使用，以程式設計方式管理這些資源。 驗證透過任何 Azure 資源提供者用戶端 (包括 Batch Management .NET 程式庫)，以及透過 [Azure Resource Manager][resman_overview] 所提出的要求時，都需要 Azure AD。

在本文中，我們將探討從使用 Batch Management .NET 程式庫的應用程式，使用 Azure AD 進行驗證。 我們會示範如何利用整合式驗證，使用 Azure AD 來驗證訂用帳戶管理員或共同管理員。 我們會使用 GitHub 上提供的 [AccountManagment][acct_mgmt_sample] 範例專案，來逐步解說如何搭配使用 Azure AD 與 Batch Management .NET 程式庫。

若要深入了解使用 Batch 管理 .NET 程式庫和 AccountManagement 範例，請參閱[使用適用於 .NET 的 Batch 管理用戶端程式庫來管理 Batch 帳戶和配額](batch-management-dotnet.md)。

## <a name="register-your-application-with-azure-ad"></a>向 Azure AD 註冊您的應用程式

Azure [Active Directory Authentication Library][aad_adal] (ADAL) 為 Azure AD 提供程式設計介面以供您的應用程式使用。 若要從您的應用程式呼叫 ADAL，您必須在 Azure AD 租用戶中註冊您的應用程式。 當您註冊應用程式時，要提供 Azure AD 有關您應用程式的資訊，包括 Azure AD 租用戶內的名稱。 Azure AD 接著會提供您在執行階段用來將應用程式與 Azure AD 產生關聯的應用程式識別碼。 若要深入了解應用程式識別碼，請參閱[Azure Active Directory 中的應用程式物件和服務主體物件之間的關聯性討論](../active-directory/develop/app-objects-and-service-principals.md)。

若要註冊 AccountManagement 範例應用程式，遵循[整合應用程式與 Azure Active Directory][aad_integrate] 之[新增應用程式](../active-directory/develop/quickstart-v1-add-azure-ad-app.md)一節中的步驟。 指定 [原生用戶端應用程式] 作為應用程式類型。 適用於**重新導向 URI** 的業界標準 OAuth 2.0 URI 是 `urn:ietf:wg:oauth:2.0:oob`。 不過，您可以針對**重新導向 URI** 指定任何有效的 URI (例如 `http://myaccountmanagementsample`)，因為它不需要是實際的端點：

![](./media/batch-aad-auth-management/app-registration-management-plane.png)

完成註冊程序後，您會看到應用程式識別碼以及針對應用程式列出的物件 (服務主體) 識別碼。  

![](./media/batch-aad-auth-management/app-registration-client-id.png)

## <a name="grant-the-azure-resource-manager-api-access-to-your-application"></a>授與 Azure Resource Manager API 存取給您的應用程式

接下來，您必須將應用程式的存取委派給 Azure Resource Manager API。 Resource Manager API 的 Azure AD 識別項是 **Windows Azure 服務管理 API**。

在 Azure 入口網站中遵循下列步驟：

1. 在 Azure 入口網站的左側導覽窗格中，選擇 [所有服務]，按一下 [應用程式註冊]，然後按一下 [新增]。
2. 在應用程式註冊清單中搜尋您應用程式的名稱︰

    ![搜尋您的應用程式名稱](./media/batch-aad-auth-management/search-app-registration.png)

3. 顯示 [設定] 刀鋒視窗。 在 [API 存取] 區段中，選取 [必要權限]。
4. 按一下 [新增] 以新增必要權限。 
5. 在步驟 1 中，輸入 **Windows Azure 服務管理 API**，從結果清單選取該 API，然後按一下 [選取] 按鈕。
6. 在步驟 2 中，選取**存取 Azure 傳統部署模型做為組織使用者**旁的核取方塊，然後按一下 [選取] 按鈕。
7. 按一下 [完成] 按鈕。

[必要權限] 刀鋒視窗現在會顯示已向 ADAL 和 Resource Manager API 授與您應用程式的權限。 當您第一次向 Azure AD 註冊您的應用程式時，權限依預設會授與給 ADAL。

![委派權限給 Azure Resource Manager API](./media/batch-aad-auth-management/required-permissions-management-plane.png)

## <a name="azure-ad-endpoints"></a>Azure AD 端點

若要使用 Azure AD 驗證您的 Batch Management 解決方案，需要兩個已知的端點。

- 若未提供特定的租用戶，**Azure AD 通用端點**會提供一般認證蒐集介面，如同整合式驗證的情況：

    `https://login.microsoftonline.com/common`

- **Azure Resource Manager 端點**可用來取得權杖，以驗證對 Batch Management 服務的要求：

    `https://management.core.windows.net/`

AccountManagement 範例應用程式會定義這些端點的常數。 將這些常數保留不變︰

```csharp
// Azure Active Directory "common" endpoint.
private const string AuthorityUri = "https://login.microsoftonline.com/common";
// Azure Resource Manager endpoint 
private const string ResourceUri = "https://management.core.windows.net/";
```

## <a name="reference-your-application-id"></a>參考您的應用程式識別碼 

用戶端應用程式使用應用程式識別碼 (也稱為用戶端識別碼) 在執行階段存取 Azure AD。 在 Azure 入口網站中註冊您的應用程式後，更新程式碼以使用 Azure AD 針對已註冊應用程式所提供的應用程式識別碼。 在 AccountManagement 範例應用程式中，從 Azure 入口網站將應用程式識別碼複製到適當的常數︰

```csharp
// Specify the unique identifier (the "Client ID") for your application. This is required so that your
// native client application (i.e. this sample) can access the Microsoft Azure AD Graph API. For information
// about registering an application in Azure Active Directory, please see "Adding an Application" here:
// https://azure.microsoft.com/documentation/articles/active-directory-integrating-applications/
private const string ClientId = "<application-id>";
```
同時，複製您在註冊程序期間指定的重新導向 URI。 您程式碼中指定的重新導向 URI 必須符合您註冊應用程式時所提供的重新導向 URI。

```csharp
// The URI to which Azure AD will redirect in response to an OAuth 2.0 request. This value is
// specified by you when you register an application with AAD (see ClientId comment). It does not
// need to be a real endpoint, but must be a valid URI (e.g. https://accountmgmtsampleapp).
private const string RedirectUri = "http://myaccountmanagementsample";
```

## <a name="acquire-an-azure-ad-authentication-token"></a>取得 Azure AD 驗證權杖

在 Azure AD 租用戶中註冊 AccountManagement 範例，並使用您的值更新範例原始程式碼之後，範例就準備好使用 Azure AD 進行驗證。 當您執行範例時，ADAL 會嘗試取得驗證權杖。 在此步驟中，它會提示您輸入您的 Microsoft 認證︰ 

```csharp
// Obtain an access token using the "common" AAD resource. This allows the application
// to query AAD for information that lies outside the application's tenant (such as for
// querying subscription information in your Azure account).
AuthenticationContext authContext = new AuthenticationContext(AuthorityUri);
AuthenticationResult authResult = authContext.AcquireToken(ResourceUri,
                                                        ClientId,
                                                        new Uri(RedirectUri),
                                                        PromptBehavior.Auto);
```

您提供認證之後，範例應用程式可以繼續發出已驗證要求給 Batch 管理服務。 

## <a name="next-steps"></a>後續步驟

如需執行 [AccountManagement 範例應用程式][acct_mgmt_sample]的相關資訊，請參閱[使用適用於 .NET 的 Batch 管理用戶端程式庫來管理 Batch 帳戶和配額](batch-management-dotnet.md)。

若要深入了解 Azure AD，請參閱 [Azure Active Directory 文件](https://docs.microsoft.com/azure/active-directory/)。 [Azure 程式碼範例](https://azure.microsoft.com/resources/samples/?service=active-directory)程式庫中有深入的範例示範如何使用 ADAL。

若要使用 Azure AD 驗證 Batch 服務應用程式，請參閱[使用 Active Directory 驗證 Batch 服務解決方案](batch-aad-auth.md)。 


[aad_about]:../active-directory/fundamentals/active-directory-whatis.md "什麼是 Azure Active Directory？"
[aad_adal]: ../active-directory/active-directory-authentication-libraries.md
[aad_auth_scenarios]:../active-directory/develop/authentication-scenarios.md "Azure AD 的驗證案例"
[aad_integrate]: ../active-directory/active-directory-integrating-applications.md "整合應用程式與 Azure Active Directory"
[acct_mgmt_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/AccountManagement
[azure_portal]: http://portal.azure.com
[resman_overview]: ../azure-resource-manager/resource-group-overview.md
