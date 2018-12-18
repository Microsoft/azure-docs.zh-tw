---
title: 在 Azure Active Directory B2C 中使用用戶端憑證保護 RESTful 服務 | Microsoft Docs
description: 使用用戶端憑證在您的 Azure AD B2C 中保護您的自訂 REST API 宣告交換
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/25/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 7bf7add75f60bf64f64119979e5eee81be0f6e7b
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344960"
---
# <a name="secure-your-restful-service-by-using-client-certificates"></a>使用用戶端憑證保護您的 RESTful 服務

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

在相關的文章中，您可以[建立 RESTful 服務](active-directory-b2c-custom-rest-api-netfw.md)，與 Azure Active Directory B2C (Azure AD B2C) 互動。

在本文章中，您會了解如何使用用戶端憑證，限制 Azure Web 應用程式 (RESTful API) 的存取。 此機制稱為 TLS 相互驗證或*用戶端憑證驗證*。 只有具有適當憑證 (例如 Azure AD B2C) 的服務才能存取您的服務。

>[!NOTE]
>您也可以使用 [HTTP 基本驗證](active-directory-b2c-custom-rest-api-netfw-secure-basic.md)來保護 RESTful 服務。 不過，相較於用戶端憑證，HTTP 基本驗證被認為較不安全。 我們建議是使用用戶端憑證驗證來保護 RESTful 服務的安全，如本文章中所述。

這篇文章詳細說明如何：
* 將您的 Web 應用程式設定為使用用戶端憑證驗證。
* 將憑證上傳至 Azure AD B2C 原則金鑰。
* 將您的自訂原則設定為使用用戶端憑證。

## <a name="prerequisites"></a>必要條件
* 完成[整合 REST API 宣告交換](active-directory-b2c-custom-rest-api-netfw.md)文章中的步驟。
* 取得有效的憑證 (具有私密金鑰的 .pfx 檔案)。

## <a name="step-1-configure-a-web-app-for-client-certificate-authentication"></a>步驟 1：設定 Web 應用程式進行用戶端憑證驗證
若要設定 **Azure App Service** 以要求用戶端憑證，請將 Web 應用程式 `clientCertEnabled` 站台設定設為 *true*。 若要進行此變更，請在 Azure 入口網站中，開啟 Web 應用程式頁面。 在左側導覽的 [設定] 底下，選取 [SSL 設定]。 在 [用戶端憑證] 區段中，開啟 [連入用戶端憑證] 選項。

>[!NOTE]
>請確定您的 Azure App Service 方案至少為標準版本。 如需詳細資訊，請參閱 [Azure App Service 方案深入概觀](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)。

>[!NOTE]
>如需設定 **clientCertEnabled** 屬性的詳細資訊，請參閱[設定 Web 應用程式的 TLS 相互驗證](https://docs.microsoft.com/azure/app-service-web/app-service-web-configure-tls-mutual-auth)。

## <a name="step-2-upload-your-certificate-to-azure-ad-b2c-policy-keys"></a>步驟 2：將憑證上傳至 Azure AD B2C 原則金鑰
將 `clientCertEnabled` 設為 *ture* 之後，與您的 RESTful API 的通訊需要用戶端憑證。 若要在您的 Azure AD B2C 租用戶中取得、上傳和儲存用戶端憑證，請執行下列動作： 
1. 在您的 Azure AD B2C 租用戶中，選取 [B2C 設定] > [識別體驗架構]。

2. 若要檢視租用戶中可用的金鑰，請選取 [原則金鑰]。

3. 選取 [新增] 。  
    [建立金鑰] 視窗隨即開啟。

4. 在 [選項] 方塊中，選取 [上傳]。

5. 在 [名稱] 方塊中，輸入 **B2cRestClientCertificate**。  
    即會自動新增前置詞 *B2C_1A_*。

6. 在 [檔案上傳] 方塊中，選取包含私密金鑰的憑證 .pfx 檔案。

7. 在 [密碼] 方塊中，輸入憑證的密碼。

    ![上傳原則金鑰](media/aadb2c-ief-rest-api-netfw-secure-cert/rest-api-netfw-secure-client-cert-upload.png)

7. 選取 [建立] 。

8. 若要檢視租用戶中可用的金鑰，並確認您已建立 `B2C_1A_B2cRestClientCertificate` 金鑰，請選取 [原則金鑰]。

## <a name="step-3-change-the-technical-profile"></a>步驟 3：變更技術設定檔
若要在您的自訂原則中支援用戶端憑證驗證，請執行下列動作來變更技術設定檔：

1. 在您的工作目錄中，開啟 *TrustFrameworkExtensions.xml* 擴充原則檔案。

2. 搜尋包含 `Id="REST-API-SignUp"` 的 `<TechnicalProfile>` 節點。

3. 找出 `<Metadata>` 元素。

4. 將 *AuthenticationType* 變更為 *ClientCertificate*，如下所示：

    ```xml
    <Item Key="AuthenticationType">ClientCertificate</Item>
    ```

5. 在關閉 `<Metadata>` 元素之後，立即新增下列 XML 程式碼片段： 

    ```xml
    <CryptographicKeys>
        <Key Id="ClientCertificate" StorageReferenceId="B2C_1A_B2cRestClientCertificate" />
    </CryptographicKeys>
    ```

    在新增程式碼片段之後，您的技術設定檔看起來應該類似下列 XML 程式碼：

    ![設定 ClientCertificate 驗證 XML 元素](media/aadb2c-ief-rest-api-netfw-secure-cert/rest-api-netfw-secure-client-cert-tech-profile.png)

## <a name="step-4-upload-the-policy-to-your-tenant"></a>步驟 4：將原則上傳至您的租用戶

1. 在 [Azure 入口網站](https://portal.azure.com)中，切換至[您的 Azure AD B2C 租用戶環境](active-directory-b2c-navigate-to-b2c-context.md)，然後選取 [Azure AD B2C]。

2. 選取 [識別體驗架構]。

3. 選取 [所有原則]。

4. 選取 [上傳原則]。

5. 選取 [覆寫已存在的原則] 核取方塊。

6. 上傳 TrustFrameworkExtensions.xml 檔案，然後確定它通過驗證。

## <a name="step-5-test-the-custom-policy-by-using-run-now"></a>步驟 5：使用 [立即執行] 測試自訂原則
1. 開啟 [Azure AD B2C 設定]，然後選取 [識別體驗架構]。

    >[!NOTE]
    >[立即執行] 需要在租用戶上至少預先註冊一個應用程式。 若要了解如何註冊應用程式，請參閱 Azure AD B2C [開始使用](active-directory-b2c-get-started.md)一文或[應用程式註冊](active-directory-b2c-app-registration.md)一文。

2. 開啟 **B2C_1A_signup_signin** (此為您上傳的信賴憑證者 (RP) 自訂原則)，然後選取 [立即執行]。

3. 在 [名字] 方塊中，輸入 **Test** 來測試流程。  
    Azure AD B2C 會在視窗頂端顯示錯誤訊息。    

    ![測試您的身分識別 API](media/aadb2c-ief-rest-api-netfw-secure-basic/rest-api-netfw-test.png)

4. 在 [名字] 方塊中輸入名字 ("Test" 除外)。  
    Azure AD B2C 會註冊使用者，然後將會員號碼傳送至您的應用程式。 記下此 JWT 範例中的號碼：

   ```
   {
     "typ": "JWT",
     "alg": "RS256",
     "kid": "X5eXk4xyojNFum1kl2Ytv8dlNP4-c57dO6QGTVBwaNk"
   }.{
     "exp": 1507125903,
     "nbf": 1507122303,
     "ver": "1.0",
     "iss": "https://contoso.b2clogin.com/f06c2fe8-709f-4030-85dc-38a4bfd9e82d/v2.0/",
     "aud": "e1d2612f-c2bc-4599-8e7b-d874eaca1ee1",
     "acr": "b2c_1a_signup_signin",
     "nonce": "defaultNonce",
     "iat": 1507122303,
     "auth_time": 1507122303,
     "loyaltyNumber": "290",
     "given_name": "Emily",
     "emails": ["B2cdemo@outlook.com"]
   }
   ```

   >[!NOTE]
   >如果收到錯誤訊息：*名稱無效，請提供有效的名稱*，這表示 Azure AD B2C 在呈現用戶端憑證時已成功呼叫 RESTful 服務。 下一個步驟是驗證憑證。

## <a name="step-6-add-certificate-validation"></a>步驟 6：新增憑證驗證
Azure Web 應用程式平台不會對 Azure AD B2C 傳送至 RESTful 服務的用戶端憑證進行任何驗證 (除了檢查憑證是否存在)。 驗證憑證是 Web 應用程式的責任。 

在本節中，您會基於驗證而驗證憑證內容的範例 ASP.NET 程式碼。

> [!NOTE]
>如需設定 Azure App Service 應用程式進行用戶端憑證驗證的詳細資訊，請參閱[設定 Web 應用程式的 TLS 相互驗證](https://docs.microsoft.com/azure/app-service-web/app-service-web-configure-tls-mutual-auth)。

### <a name="61-add-application-settings-to-your-projects-webconfig-file"></a>6.1 將應用程式設定新增至專案的 web.config 檔案
在您稍早建立的 Visual Studio 專案中，將下列應用程式設定新增至 *web.config* 檔案，位在 `appSettings` 元素之後：

```XML
<add key="ClientCertificate:Subject" value="CN=Subject name" />
<add key="ClientCertificate:Issuer" value="CN=Issuer name" />
<add key="ClientCertificate:Thumbprint" value="Certificate thumbprint" />
```

將憑證的**主體名稱**、**簽發者名稱**和**憑證指紋**值取代為您的憑證值。

### <a name="62-add-the-isvalidclientcertificate-function"></a>6.2 新增 IsValidClientCertificate 函式
開啟 *Controllers\IdentityController.cs* 檔案，並將下列函式新增至 `Identity` 控制器類別： 

```csharp
private bool IsValidClientCertificate()
{
    string ClientCertificateSubject = ConfigurationManager.AppSettings["ClientCertificate:Subject"];
    string ClientCertificateIssuer = ConfigurationManager.AppSettings["ClientCertificate:Issuer"];
    string ClientCertificateThumbprint = ConfigurationManager.AppSettings["ClientCertificate:Thumbprint"];

    X509Certificate2 clientCertInRequest = RequestContext.ClientCertificate;
    if (clientCertInRequest == null)
    {
        Trace.WriteLine("Certificate is NULL");
        return false;
    }

    // Basic verification
    if (clientCertInRequest.Verify() == false)
    {
        Trace.TraceError("Basic verification failed");
        return false;
    }

    // 1. Check the time validity of the certificate
    if (DateTime.Compare(DateTime.Now, clientCertInRequest.NotBefore) < 0 ||
        DateTime.Compare(DateTime.Now, clientCertInRequest.NotAfter) > 0)
    {
        Trace.TraceError($"NotBefore '{clientCertInRequest.NotBefore}' or NotAfter '{clientCertInRequest.NotAfter}' not valid");
        return false;
    }

    // 2. Check the subject name of the certificate
    bool foundSubject = false;
    string[] certSubjectData = clientCertInRequest.Subject.Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
    foreach (string s in certSubjectData)
    {
        if (String.Compare(s.Trim(), ClientCertificateSubject) == 0)
        {
            foundSubject = true;
            break;
        }
    }

    if (!foundSubject)
    {
        Trace.TraceError($"Subject name '{clientCertInRequest.Subject}' is not valid");
        return false;
    }
    
    // 3. Check the issuer name of the certificate
    bool foundIssuerCN = false;
    string[] certIssuerData = clientCertInRequest.Issuer.Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
    foreach (string s in certIssuerData)
    {
        if (String.Compare(s.Trim(), ClientCertificateIssuer) == 0)
        {
            foundIssuerCN = true;
            break;
        }
    }

    if (!foundIssuerCN)
    {
        Trace.TraceError($"Issuer '{clientCertInRequest.Issuer}' is not valid");
        return false;
    }

    // 4. Check the thumbprint of the certificate
    if (String.Compare(clientCertInRequest.Thumbprint.Trim().ToUpper(), ClientCertificateThumbprint) != 0)
    {
        Trace.TraceError($"Thumbprint '{clientCertInRequest.Thumbprint.Trim().ToUpper()}' is not valid");
        return false;
    }

    // 5. If you also want to test whether the certificate chains to a trusted root authority, you can uncomment the following code:
    //
    //X509Chain certChain = new X509Chain();
    //certChain.Build(certificate);
    //bool isValidCertChain = true;
    //foreach (X509ChainElement chElement in certChain.ChainElements)
    //{
    //    if (!chElement.Certificate.Verify())
    //    {
    //        isValidCertChain = false;
    //        break;
    //    }
    //}
    //if (!isValidCertChain) return false;
    return true;
}
```

在上述範例程式碼中，只在憑證符合下列所有條件時，我們才接受它是有效的憑證：
* 憑證未過期，並且在伺服器上目前為作用中。
* 憑證的 `Subject` 名稱等於 `ClientCertificate:Subject` 應用程式設定值。
* 憑證的 `Issuer` 名稱等於 `ClientCertificate:Issuer` 應用程式設定值。
* 憑證的 `thumbprint` 等於 `ClientCertificate:Thumbprint` 應用程式設定值。

>[!IMPORTANT]
>根據服務的敏感度，您可能需要新增更多驗證。 例如，您可能需要測試憑證是否鏈結至信任的根授權單位、簽發者組織名稱驗證，依此類推。

### <a name="63-call-the-isvalidclientcertificate-function"></a>6.3 呼叫 IsValidClientCertificate 函式
開啟 *Controllers\IdentityController.cs* 檔案，然後在 `SignUp()` 函式的開頭，新增下列程式碼片段： 

```csharp
if (IsValidClientCertificate() == false)
{
    return Content(HttpStatusCode.Conflict, new B2CResponseContent("Your client certificate is not valid", HttpStatusCode.Conflict));
}
```

在新增程式碼片段之後，您的 `Identity` 控制器看起來應該類似下列程式碼：

![新增憑證驗證程式碼](media/aadb2c-ief-rest-api-netfw-secure-cert/rest-api-netfw-secure-client-code.png)

## <a name="step-7-publish-your-project-to-azure-and-test-it"></a>步驟 7：將專案發佈至 Azure 並進行測試
1. 在 [方案總管] 中，以滑鼠右鍵按一下 **Contoso.AADB2C.API** 專案，然後選取 [發佈]。

2. 重複「步驟 6」，並使用憑證驗證重新測試您的自訂原則。 新增驗證之後，嘗試執行原則並確定一切可運作。

3. 在 *web.config* 檔案中，將 `ClientCertificate:Subject` 的值變更為 **invalid**。 再次執行此原則，您應該會看到錯誤訊息。

4. 將值變更回 **valid**，並確定該原則可以呼叫您的 REST API。

如果您需要針對這個步驟進行疑難排解，請參閱[使用 Application Insights 收集記錄](active-directory-b2c-troubleshoot-custom.md)。

## <a name="optional-download-the-complete-policy-files-and-code"></a>(選用) 下載完整的原則檔案和程式碼
* 在完成[開始使用自訂原則](active-directory-b2c-get-started-custom.md)逐步解說之後，建議您使用自己的自訂原則檔案來建置您的情節。 我們已提供[範例原則檔案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw-secure-cert)，供您參考。
* 您可以從[供參考的範例 Visual Studio 解決方案](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/aadb2c-ief-rest-api-netfw/Contoso.AADB2C.API)中下載完整程式碼。 
