---
title: Azure Active Directory 驗證和 Resource Manager | Microsoft Docs
description: 使用 Azure Resource Manager API 與 Azure Active Directory 進行驗證以整合應用程式與其他 Azure 訂用帳戶的開發人員指南。
services: azure-resource-manager,active-directory
documentationcenter: na
author: dushyantgill
manager: timlt
editor: tysonn
ms.assetid: 17b2b40d-bf42-4c7d-9a88-9938409c5088
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/12/2018
ms.author: dugill
ms.openlocfilehash: b841a1104a0cc1e74d9ab1f16ef39d3892ba7d55
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996684"
---
# <a name="use-resource-manager-authentication-api-to-access-subscriptions"></a>使用 Resource Manager 驗證 API 來存取訂用帳戶
## <a name="introduction"></a>簡介
如果您是必須建立應用程式來管理客戶的 Azure 資源的軟體開發人員，本文說明如何使用 Azure Resource Manager API 來進行驗證，並取得其他訂用帳戶中資源的存取權。

您的應用程式可透過數種方式存取資源管理員 API︰

1. **使用者 + 應用程式存取**︰適用於代表登入使用者存取資源的應用程式。 此方式適用於僅處理「互動式管理」Azure 資源的應用程式，例如 Web 應用程式和命令列工具。
2. **僅限應用程式存取**︰適用於執行協助程式服務和已排程之作業的應用程式。 應用程式的身分識別會獲得資源的直接存取權。 此方式適用於需要長期無周邊 (自動) 存取 Azure 的應用程式。

本文提供建立應用程式來運用這兩種授權方法的逐步指示。 它會說明如何使用 REST API 或 C# 執行每個步驟。 在 [https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense](https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense) 可取得完整的 ASP.NET MVC 應用程式。

## <a name="what-the-web-app-does"></a>Web 應用程式的功用
Web 應用程式：

1. 登入 Azure 使用者。
2. 要求使用者授與 Resource Manager 的存取權給 Web 應用程式。
3. 取得使用者 + 應用程式的存取權杖來存取 Resource Manager。
4. 使用權杖 (來自步驟 3) 將應用程式的服務主體指派給訂用帳戶中的角色。 此步驟可讓應用程式長期存取訂用帳戶。
5. 取得僅限應用程式存取權杖。
6. 使用權杖 (來自步驟 5) 以透過 Resource Manager 管理訂用帳戶中的資源。

以下是 Web 應用程式的流程。

![Resource Manager 驗證流程](./media/resource-manager-api-authentication/Auth-Swim-Lane.png)

身為使用者，您必須提供所要使用之訂用帳戶的訂用帳戶識別碼︰

![提供訂用帳戶識別碼](./media/resource-manager-api-authentication/sample-ux-1.png)

選取要用於登入的帳戶。

![選取帳戶](./media/resource-manager-api-authentication/sample-ux-2.png)

提供您的認證。

![提供認證](./media/resource-manager-api-authentication/sample-ux-3.png)

授與應用程式存取您的 Azure 訂用帳戶︰

![授與存取權](./media/resource-manager-api-authentication/sample-ux-4.png)

管理連接的訂用帳戶：

![連線訂用帳戶](./media/resource-manager-api-authentication/sample-ux-7.png)

## <a name="register-application"></a>註冊應用程式
在開始撰寫程式碼之前，請先使用 Azure Active Directory (AD) 註冊 Web 應用程式。 應用程式註冊會為您在 Azure AD 中的應用程式建立中央身分識別。 它會保留您的應用程式的基本資訊，例如您的應用程式用來驗證和存取 Azure Resource Manager API 的 OAuth 用戶端識別碼、回覆 URL 和認證。 應用程式註冊也會記錄您的應用程式需要的各種委派權限，以便代表使用者存取 Microsoft API。

由於應用程式會存取其他訂用帳戶，您必須將它設定為多租用戶應用程式。 為了通過驗證，請提供與 Azure Active Directory 相關聯的網域。 若要查看與 Azure Active Directory 相關聯的網域，請登入入口網站。

下列範例示範如何使用 Azure PowerShell 註冊應用程式。 您必須擁有最新版 (2016 年 8 月) Azure PowerShell 才能讓此命令生效。

```azurepowershell-interactive
$app = New-AzureRmADApplication -DisplayName "{app name}" -HomePage "https://{your domain}/{app name}" -IdentifierUris "https://{your domain}/{app name}" -Password "{your password}" -AvailableToOtherTenants $true
```

若要以 AD 應用程式的身分登入，您需要應用程式的識別碼和密碼。 若要查看前一個命令所傳回的應用程式識別碼，請使用︰

```azurepowershell-interactive
$app.ApplicationId
```

下列範例示範如何使用 Azure CLI 註冊應用程式。

```azurecli-interactive
az ad app create --display-name {app name} --homepage https://{your domain}/{app name} --identifier-uris https://{your domain}/{app name} --password {your password} --available-to-other-tenants true
```

結果內包含 AppId，以應用程式的形式進行驗證時會需要此資料。

### <a name="optional-configuration---certificate-credential"></a>選擇性組態 - 憑證認證
Azure AD 也支援應用程式的憑證認證︰您建立自我簽署憑證、保留私密金鑰，然後將公開金鑰新增至 Azure AD 應用程式註冊。 若為驗證，您的應用程式會使用您的私密金鑰將小裝載傳送至簽署的 Azure AD，且 Azure AD 會使用您註冊的公開金鑰來驗證簽章。

如需使用憑證建立 AD 應用程式的相關資訊，請參閱[使用 Azure PowerShell 建立用來存取資源的服務主體](resource-group-authenticate-service-principal.md#create-service-principal-with-certificate-from-certificate-authority)或[使用 Azure CLI 建立用來存取資源的服務主體](resource-group-authenticate-service-principal-cli.md)。

## <a name="get-tenant-id-from-subscription-id"></a>從訂用帳戶識別碼取得租用戶識別碼
若要要求可用來呼叫 Resource Manager 的權杖，應用程式必須知道裝載 Azure 訂用帳戶之 Azure AD 租用戶的租用戶識別碼。 使用者很可能知道其訂用帳戶識別碼，但他們可能不知道其用於 Azure Active Directory 的租用戶識別碼。 若要取得使用者的租用戶識別碼，請要求使用者提供訂用帳戶識別碼。 在傳送有關訂用帳戶的要求時，請提供該訂用帳戶識別碼：

    https://management.azure.com/subscriptions/{subscription-id}?api-version=2015-01-01

要求會因為使用者尚未登入而失敗，但您可以從回應中擷取出租用戶識別碼。 在該例外狀況中，請從 **WWW-Authenticate**的回應標頭值中擷取出租用戶識別碼。 您可以在 [GetDirectoryForSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L20) 方法中看到此實作。

## <a name="get-user--app-access-token"></a>取得使用者 + 應用程式的存取權杖
您的應用程式會使用 OAuth 2.0 授權要求，將使用者重新導向到 Azure AD - 以驗證使用者的認證及取回授權碼。 應用程式會使用授權碼來存取 Resource Manager 的權杖。 [ConnectSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/Controllers/HomeController.cs#L42) 方法會建立授權要求。

本文說明用來驗證使用者的 REST API 要求。 您也可以使用協助程式庫以在程式碼中執行驗證。 如需這些程式庫的詳細資訊，請參閱 [Azure Active Directory 驗證程式庫](../active-directory/active-directory-authentication-libraries.md)。 如需在應用程式中整合身分識別管理的指引，請參閱 [Azure Active Directory 開發人員指南](../active-directory/develop/azure-ad-developers-guide.md)。

### <a name="auth-request-oauth-20"></a>驗證要求 (OAuth 2.0)
將開啟識別碼連線/OAuth2.0 授權要求發給 Azure AD 授權端點︰

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize

[要求授權碼](../active-directory/develop/v1-protocols-oauth-code.md#request-an-authorization-code)一文說明適用於此要求的查詢字串參數。

下列範例示範如何要求 OAuth2.0 授權︰

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=query&response_type=code&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&domain_hint=live.com

Azure AD 驗證使用者，並在需要時要求使用者將權限授與應用程式。 它會將授權碼傳回至應用程式的回覆 URL。 根據要求的 response_mode，Azure AD 會在查詢字串中傳回資料，或作為張貼資料。

    code=AAABAAAAiL****FDMZBUwZ8eCAA&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### <a name="auth-request-open-id-connect"></a>驗證要求 (Open ID Connect)
如果您不只想要代表使用者存取 Azure Resource Manager，並且允許使用者使用其 Azure AD 帳戶登入您的應用程式，請發出 Open ID Connect 授權要求。 使用 Open ID Connect，您的應用程式也會收到 Azure AD 的 id_token，讓應用程式用來登入使用者。

[傳送登入要求](../active-directory/develop/v1-protocols-openid-connect-code.md#send-the-sign-in-request)一文說明適用於此要求的查詢字串參數。

Open ID Connect 的要求範例是︰

     https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=form_post&response_type=code+id_token&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&scope=openid+profile&nonce=63567Dc4MDAw&domain_hint=live.com&state=M_12tMyKaM8

Azure AD 驗證使用者，並在需要時要求使用者將權限授與應用程式。 它會將授權碼傳回至應用程式的回覆 URL。 根據要求的 response_mode，Azure AD 會在查詢字串中傳回資料，或作為張貼資料。

Open ID Connect 回應的範例是︰

    code=AAABAAAAiL*****I4rDWd7zXsH6WUjlkIEQxIAA&id_token=eyJ0eXAiOiJKV1Q*****T3GrzzSFxg&state=M_12tMyKaM8&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### <a name="token-request-oauth20-code-grant-flow"></a>權杖要求 (OAuth2.0 程式碼授與流程)
您的應用程式現在已從 Azure AD 收到授權碼，現在是取得 Azure Resource Manager 的存取權杖的時候了。  將 OAuth2.0 程式碼授與權杖要求張貼至 Azure AD 權杖端點︰

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Token

[使用授權碼](../active-directory/develop/v1-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token)一文說明適用於此要求的查詢字串參數。

下列範例顯示如何使用密碼認證來要求程式碼授與權杖：

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

使用憑證認證時，使用您的應用程式的憑證認證的私密金鑰來建立 JSON Web 權杖 (JWT) 和登入 (RSA SHA256)。 [用戶端認證流程](../active-directory/develop/v1-oauth2-client-creds-grant-flow.md#second-case-access-token-request-with a-certificate)中說明如何建置此權杖。  如需參考，請參閱 [Active Directory 驗證程式庫 (.NET) 程式碼](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/dev/src/ADAL.PCL.Desktop/CryptographyHelper.cs) 來簽署用戶端判斷提示 JWT 權杖。

請參閱 [Open ID Connect 規格](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication) 來取得用戶端驗證的詳細資訊。

下列範例顯示如何使用憑證認證來要求程式碼授與權杖：

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbG*****Y9cYo8nEjMyA

程式碼授與權杖回應的範例︰

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039858","not_before":"1432035958","resource":"https://management.core.windows.net/","access_token":"eyJ0eXAiOiJKV1Q****M7Cw6JWtfY2lGc5A","refresh_token":"AAABAAAAiL9Kn2Z****55j-sjnyYgAA","scope":"user_impersonation","id_token":"eyJ0eXAiOiJKV*****-drP1J3P-HnHi9Rr46kGZnukEBH4dsg"}

#### <a name="handle-code-grant-token-response"></a>處理程式碼授與權杖回應
成功的權杖回應會包含 Azure Resource Manager 的 (使用者 + 應用程式) 存取權杖。 應用程式會使用此存取權杖來代表使用者存取 Resource Manager。 Azure AD 所發出的存取權杖存留期是一小時。 Web 應用程式不太可能需要更新 (使用者 + 應用程式) 存取權杖。 如果需要更新存取權杖，請使用應用程式在權杖回應中收到的重新整理權杖。 將 OAuth2.0 權杖要求張貼至 Azure AD 權杖端點︰

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Token

[重新整理存取權杖](../active-directory/develop/v1-protocols-oauth-code.md#refreshing-the-access-tokens)中會說明要與重新整理要求搭配使用的參數。

下列範例顯示如何使用重新整理權杖：

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=refresh_token&refresh_token=AAABAAAAiL9Kn2Z****55j-sjnyYgAA&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

雖然重新整理權杖可用來取得 Azure Resource Manager 的新存取權杖，但它們並不適合您的應用程式離線存取。 重新整理權杖存留期有限，且重新整理權杖會繫結至使用者。 如果使用者離開組織，使用重新整理權杖的應用程式會無法存取。 這個方法不適合小組用來管理其 Azure 資源的應用程式。

## <a name="check-if-user-can-assign-access-to-subscription"></a>檢查使用者是否可以指派訂用帳戶的存取權
您的應用程式現在具有權杖，可代表使用者存取 Azure Resource Manager。 下一步是將應用程式連接到訂用帳戶。 在連接之後，即使使用者不存在 (長期離線存取)，您仍然可以管理這些訂用帳戶。

針對每個要連接的訂用帳戶，請呼叫 [Resource Manager 列出權限](https://docs.microsoft.com/rest/api/authorization/permissions) API 來判斷使用者是否具有訂用帳戶的存取管理權限。

ASP.NET MVC 範例應用程式的 [UserCanManagerAccessForSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L44) 方法會實作這個呼叫。

下列範例示範如何要求使用者在訂用帳戶上的權限。 83cfe939-2402-4581-b761-4f59b0a041e4 是訂用帳戶的識別碼。

    GET https://management.azure.com/subscriptions/83cfe939-2402-4581-b761-4f59b0a041e4/providers/microsoft.authorization/permissions?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiLC***lwO1mM7Cw6JWtfY2lGc5A

以下是針對取得使用者在訂用帳戶上的權限所做的回應範例︰

    HTTP/1.1 200 OK

    {"value":[{"actions":["*"],"notActions":["Microsoft.Authorization/*/Write","Microsoft.Authorization/*/Delete"]},{"actions":["*/read"],"notActions":[]}]}

權限 API 會傳回多個權限。 每個權限包含允許的動作 (**actions**) 和不允許的動作 (**notactions**)。 如果動作出現在任何權限的允許 actions 中，而且不存在於該權限的不允許動作中，則使用者可執行該動作。 **microsoft.authorization/roleassignments/write** 是授與存取管理權限的動作。 您的應用程式必須剖析權限結果，才能在每個權限的 **actions** 和 **notactions** 中的此動作字串上，尋找 regex 相符項。

## <a name="get-app-only-access-token"></a>取得僅限應用程式存取權杖
現在，您已知道使用者是否可以指派 Azure 訂用帳戶的存取權。 後續步驟如下︰

1. 將適當的 RBAC 角色指派給您的應用程式在訂用帳戶上的身分識別。
2. 查詢訂用帳戶上的應用程式權限，或使用僅限應用程式權杖來存取 Resource Manager，藉此驗證存取指派。
3. 記錄應用程式「已連線的訂用帳戶」資料結構中的連線 - 保存訂用帳戶的識別碼。

讓我們更仔細地檢視第一個步驟。 若要將適當的 RBAC 角色指派給應用程式身分識別，您必須判斷︰

* 您在使用者的 Azure Active Directory 中的應用程式身分識別的物件識別碼
* 您的應用程式在訂用帳戶上需要的 RBAC 角色的識別碼

當您的應用程式驗證來自 Azure AD 的使用者時，它會在該 Azure AD 中建立應用程式的服務主體物件。 Azure 允許將 RBAC 角色指派給服務主體，以便將直接存取權授與 Azure 資源上的對應應用程式。 這就是您想要做的確切動作。 查詢 Azure AD 圖形 API，判斷應用程式在登入使用者的 Azure AD 中的服務主體識別項。

您只有 Azure Resource Manager 的存取權杖 - 您需要新的存取權杖以呼叫 Azure AD Graph API。 Azure AD 中的每個應用程式都有權查詢其本身的服務主體物件，因此僅限應用程式存取權杖就已足夠。

<a id="app-azure-ad-graph" />

### <a name="get-app-only-access-token-for-azure-ad-graph-api"></a>取得 Azure AD Graph API 的僅限應用程式存取權杖
若要驗證您的應用程式，並取得 Azure AD Graph API 的權杖，請向 Azure AD 權杖端點發出用戶端認證授與 OAuth2.0 流程權杖要求 (**https://login.microsoftonline.com/{directory_domain_name}/OAuth2/Token**)。

ASP.net MVC 範例應用程式的 [GetObjectIdOfServicePrincipalInOrganization](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs) 方法使用 Active Directory Authentication Library for .NET，以取得 Graph API 的僅限應用程式存取權杖。

[要求存取權杖](../active-directory/develop/v1-oauth2-client-creds-grant-flow.md#request-an-access-token)一文說明適用於此要求的查詢字串參數。

用戶端認證授與權杖要求範例︰

    POST https://login.microsoftonline.com/62e173e9-301e-423e-bcd4-29121ec1aa24/oauth2/token HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 187</pre>
    <pre>grant_type=client_credentials&client_id=a0448380-c346-4f9f-b897-c18733de9394&resource=https%3A%2F%2Fgraph.windows.net%2F &client_secret=olna8C*****Og%3D

用戶端認證授與權杖回應範例︰

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039862","not_before":"1432035962","resource":"https://graph.windows.net/","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRv****G5gUTV-kKorR-pg"}

### <a name="get-objectid-of-application-service-principal-in-user-azure-ad"></a>取得使用者 Azure AD 中的應用程式服務主體的 ObjectId
現在，使用僅限應用程式存取權杖來查詢 [Azure AD Graph 服務主體](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity) API，以判斷目錄中應用程式的服務主體的物件識別碼。

ASP.net MVC 範例應用程式的 [GetObjectIdOfServicePrincipalInOrganization](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs#) 方法會實作這個呼叫。

下列範例示範如何要求應用程式的服務主體。 a0448380-c346-4f9f-b897-c18733de9394 是應用程式的用戶端識別碼。

    GET https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/servicePrincipals?api-version=1.5&$filter=appId%20eq%20'a0448380-c346-4f9f-b897-c18733de9394' HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJK*****-kKorR-pg

下列範例顯示應用程式的服務主體要求的回應

    HTTP/1.1 200 OK

    {"odata.metadata":"https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/$metadata#directoryObjects/Microsoft.DirectoryServices.ServicePrincipal","value":[{"odata.type":"Microsoft.DirectoryServices.ServicePrincipal","objectType":"ServicePrincipal","objectId":"9b5018d4-6951-42ed-8a92-f11ec283ccec","deletionTimestamp":null,"accountEnabled":true,"appDisplayName":"CloudSense","appId":"a0448380-c346-4f9f-b897-c18733de9394","appOwnerTenantId":"62e173e9-301e-423e-bcd4-29121ec1aa24","appRoleAssignmentRequired":false,"appRoles":[],"displayName":"CloudSense","errorUrl":null,"homepage":"http://www.vipswapper.com/cloudsense","keyCredentials":[],"logoutUrl":null,"oauth2Permissions":[{"adminConsentDescription":"Allow the application to access CloudSense on behalf of the signed-in user.","adminConsentDisplayName":"Access CloudSense","id":"b7b7338e-683a-4796-b95e-60c10380de1c","isEnabled":true,"type":"User","userConsentDescription":"Allow the application to access CloudSense on your behalf.","userConsentDisplayName":"Access CloudSense","value":"user_impersonation"}],"passwordCredentials":[],"preferredTokenSigningKeyThumbprint":null,"publisherName":"vipswapper"quot;,"replyUrls":["http://www.vipswapper.com/cloudsense","http://www.vipswapper.com","http://vipswapper.com","http://vipswapper.azurewebsites.net","http://localhost:62080"],"samlMetadataUrl":null,"servicePrincipalNames":["http://www.vipswapper.com/cloudsense","a0448380-c346-4f9f-b897-c18733de9394"],"tags":["WindowsAzureActiveDirectoryIntegratedApp"]}]}

### <a name="get-azure-rbac-role-identifier"></a>取得 Azure RBAC 角色識別碼
若要將適當的 RBAC 角色指派給服務主體，您必須判斷 Azure RBAC 角色的識別碼。

您的應用程式的正確 RBAC 角色︰

* 如果您的應用程式只會監視訂用帳戶而不進行任何變更，它只需要訂用帳戶上的讀取器權限。 指派 **讀取器** 角色。
* 如果您的應用程式管理 Azure 訂用帳戶，建立/修改/刪除實體，它需要其中一個參與者權限。
  * 若要管理特定類型的資源，請指派資源的特定參與者角色 (虛擬機器參與者、虛擬網路參與者、儲存體帳戶參與者等)
  * 若要管理任何資源類型，請指派 **參與者** 角色。

您應用程式的角色指派會對使用者顯示，因此請選取必要的最低權限。

呼叫 [Resource Manager 角色定義 API](https://docs.microsoft.com/rest/api/authorization/roledefinitions) 來列出所有 Azure RBAC 角色和搜尋，然後逐一查看結果，按名稱尋找所需的角色定義。

ASP.net MVC 範例應用程式的 [GetRoleId](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L246) 方法會實作這個呼叫。

下列要求範例示範如何取得 Azure RBAC 角色識別碼。 09cbd307-aa71-4aca-b346-5f253e6e3ebb 是訂用帳戶的識別碼。

    GET https://management.azure.com/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV*****fY2lGc5

回應格式如下：

    HTTP/1.1 200 OK

    {"value":[{"properties":{"roleName":"API Management Service Contributor","type":"BuiltInRole","description":"Lets you manage API Management services, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.ApiManagement/Services/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/312a565d-c81f-4fd8-895a-4e21e48d571c","type":"Microsoft.Authorization/roleDefinitions","name":"312a565d-c81f-4fd8-895a-4e21e48d571c"},{"properties":{"roleName":"Application Insights Component Contributor","type":"BuiltInRole","description":"Lets you manage Application Insights components, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.Insights/components/*","Microsoft.Insights/webtests/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/ae349356-3a1b-4a5e-921d-050484c6347e","type":"Microsoft.Authorization/roleDefinitions","name":"ae349356-3a1b-4a5e-921d-050484c6347e"}]}

您不需要持續呼叫此 API。 一旦決定角色定義的已知 GUID，您可以將角色定義識別碼建構為︰

    /subscriptions/{subscription_id}/providers/Microsoft.Authorization/roleDefinitions/{well-known-role-guid}

以下是常用的內建角色識別碼：

| 角色 | GUID |
| --- | --- |
| 讀取者 |acdd72a7-3385-48ef-bd42-f606fba81ae7 |
| 參與者 |b24988ac-6180-42a0-ab88-20f7382dd24c |
| 虛擬機器參與者 |d73bb868-a0df-4d4d-bd69-98a00b01fccb |
| 虛擬網路參與者 |b34d265f-36f7-4a0d-a4d4-e158ca92e90f |
| 儲存體帳戶參與者 |86e8f5dc-a6e9-4c67-9d15-de283e8eac25 |
| 網站參與者 |de139f84-1756-47ae-9be6-808fbbe84772 |
| Web 方案參與者 |2cc479cb-7b4d-49a8-b449-8c00fd0f0a4b |
| SQL Server 參與者 |6d8ee4ec-f05a-4a1d-8b00-a9b17e38b437 |
| SQL DB 參與者 |9b7fa17d-e63e-47b0-bb0a-15c516ac86ec |

### <a name="assign-rbac-role-to-application"></a>將 RBAC 角色指派給應用程式
您已具備所有條件，可使用 [Resource Manager 建立角色指派](https://docs.microsoft.com/rest/api/authorization/roleassignments) API，將適當的 RBAC 角色指派給服務主體。

ASP.net MVC 範例應用程式的 [GrantRoleToServicePrincipalOnSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L170) 方法會實作這個呼叫。

將 RBAC 角色指派給應用程式的要求範例︰

    PUT https://management.azure.com/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/microsoft.authorization/roleassignments/4f87261d-2816-465d-8311-70a27558df4c?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiL*****FlwO1mM7Cw6JWtfY2lGc5
    Content-Type: application/json
    Content-Length: 230

    {"properties": {"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2"}}

在要求中，使用下列值︰

| Guid | 說明 |
| --- | --- |
| 09cbd307-aa71-4aca-b346-5f253e6e3ebb |訂用帳戶的識別碼 |
| c3097b31-7309-4c59-b4e3-770f8406bad2 |應用程式的服務主體的物件識別碼 |
| acdd72a7-3385-48ef-bd42-f606fba81ae7 |讀取器角色的識別碼 |
| 4f87261d-2816-465d-8311-70a27558df4c |為新角色指派建立的新 guid |

回應格式如下：

    HTTP/1.1 201 Created

    {"properties":{"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2","scope":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb"},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleAssignments/4f87261d-2816-465d-8311-70a27558df4c","type":"Microsoft.Authorization/roleAssignments","name":"4f87261d-2816-465d-8311-70a27558df4c"}

### <a name="get-app-only-access-token-for-azure-resource-manager"></a>取得 Azure Resource Manager 的僅限應用程式存取權杖
若要驗證應用程式具有所需的訂用帳戶存取權，請使用僅限應用程式權杖對訂用帳戶執行測試工作。

若要取得僅限應用程式存取權杖，請依照 [取得 Azure AD Graph API 的僅限應用程式存取權杖](#app-azure-ad-graph)一節的指示，為資源參數使用不同值︰

    https://management.core.windows.net/

ASP.NET MVC 範例應用程式的 [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L110) 方法使用 Active Directory Authentication Library for .net，以取得 Azure Resource Manager 的僅限應用程式存取權杖。

#### <a name="get-applications-permissions-on-subscription"></a>取得訂用帳戶上的應用程式權限
若要檢查應用程式是否具有 Azure 訂用帳戶上的所需存取權，您也可以呼叫 [Resource Manager 權限](https://docs.microsoft.com/rest/api/authorization/permissions) API。 此方式類似於您用來判斷使用者是否具有訂用帳戶存取管理權限的方式。 不過，此時會使用您在上一個步驟中收到的僅限應用程式存取權杖來呼叫 API 權限。

ASP.NET MVC 範例應用程式的 [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L110) 方法會實作這個呼叫。

## <a name="manage-connected-subscriptions"></a>管理連接的訂用帳戶
將適當的 RBAC 角色指派給應用程式在訂用帳戶上的服務主體時，您的應用程式可使用 Azure Resource Manager 的僅限應用程式存取權杖以持續監視/管理它。

如果訂用帳戶擁有者使用入口網站或命令列工具來移除應用程式的角色指派，您的應用程式便無法再存取該訂用帳戶。 在此情況下，您應該通知使用者：應用程式外部的訂用帳戶連線已中斷，然後提供他們 [修復] 連線的選項。 [修復] 會重新建立離線刪除的角色指派。

就像您讓使用者將訂用帳戶連接到您的應用程式，您也必須允許使用者中斷連線訂用帳戶。 從存取管理的觀點而言，中斷連線意味移除應用程式的服務主體已在訂用帳戶上的角色指派。 (選擇性) 也可能移除訂用帳戶的應用程式中的任何狀態。
只有在訂用帳戶上具有存取管理權限的使用者能夠中斷連線訂用帳戶。

ASP.net MVC 範例應用程式的 [RevokeRoleFromServicePrincipalOnSubscription 方法](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L200) 會實作這個呼叫。

大功告成，使用者現在能使用您的應用程式來輕鬆連接及管理其 Azure 訂用帳戶。
