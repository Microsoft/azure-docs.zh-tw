---
title: 使用 OAuth2.0 進行 Azure AD 服務對服務驗證 | Microsoft Docs
description: 本文說明如何使用 HTTP 訊息，以利用 OAuth2.0 用戶端認證授與流程實作服務對服務驗證。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: a7f939d9-532d-4b6d-b6d3-95520207965d
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/08/2017
ms.author: celested
ms.reviewer: nacanuma
ms.custom: aaddev
ms.openlocfilehash: eb43589b04be8b2b9c3be45d33c94592e97d308e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46948259"
---
# <a name="service-to-service-calls-using-client-credentials-shared-secret-or-certificate"></a>使用用戶端認證 (共用密碼或憑證) 的服務對服務呼叫
OAuth 2.0 用戶端認證授與流程可允許 Web 服務 (「機密用戶端」) 在呼叫另一個 Web 服務時，使用它自己的認證來進行驗證，而不是模擬使用者。 在此案例中，用戶端通常是中介層 Web 服務、精靈服務或網站。 對於較高層級的保證，Azure AD 也可讓呼叫服務使用憑證 (而非共用密碼) 做為認證。

## <a name="client-credentials-grant-flow-diagram"></a>用戶端認證授與流程圖
下圖說明用戶端認證授與流程在 Azure Active Directory (Azure AD) 中的作用方式。

![OAuth2.0 用戶端認證授與流程](./media/v1-oauth2-client-creds-grant-flow/active-directory-protocols-oauth-client-credentials-grant-flow.jpg)

1. 用戶端應用程式會向 Azure AD 權杖發行端點進行驗證，並要求存取權杖。
2. Azure AD 權杖發行端點會發行存取權杖。
3. 存取權杖可用來向受保護的資源進行驗證。
4. 來自受保護資源的資料會傳回用戶端應用程式。

## <a name="register-the-services-in-azure-ad"></a>在 Azure AD 中註冊服務
請在 Azure Active Directory (Azure AD) 中同時註冊呼叫端服務和接收端服務。 如需詳細指示，請參閱[整合應用程式與 Azure Active Directory](quickstart-v1-integrate-apps-with-azure-ad.md)。

## <a name="request-an-access-token"></a>要求存取權杖
若要要求存取權杖，請對租用戶特定的 Azure AD 端點使用 HTTP POST。

```
https://login.microsoftonline.com/<tenant id>/oauth2/token
```

## <a name="service-to-service-access-token-request"></a>服務對服務存取權杖要求
有兩種情況，取決於用戶端應用程式是選擇透過共用密碼或憑證來保護。

### <a name="first-case-access-token-request-with-a-shared-secret"></a>第一種情況︰使用共用密碼的存取權杖要求
使用共用密碼時，服務對服務存取權杖要求包含下列參數：

| 參數 |  | 說明 |
| --- | --- | --- |
| grant_type |必要 |指定要求的授與類型。 在用戶端認證授與流程中，值必須是 **client_credentials**。 |
| client_id |必要 |指定呼叫端 Web 服務的 Azure AD 用戶端識別碼。 若要尋找呼叫應用程式的用戶端識別碼，請在 [Azure 入口網站](https://portal.azure.com)中按一下 [Azure Active Directory]，並按一下 [應用程式註冊]，再按一下應用程式。 client_id 是*應用程式 ID* |
| client_secret |必要 |輸入在 Azure AD 中針對呼叫端 Web 服務或精靈應用程式所註冊的金鑰。 若要建立金鑰，請在 Azure 入口網站中按一下 [Azure Active Directory]，並按一下 [應用程式註冊]，按一下應用程式，接著依序按一下 [設定] 和 [金鑰]，並新增金鑰。  在提供此祕密時對其進行 URL 編碼。 |
| resource |必要 |輸入接收端 Web 服務的應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 入口網站中，按一下 [Azure Active Directory]，並按一下 [應用程式註冊]，再按一下服務應用程式，接著按一下 [設定] 和 [屬性]。 |

#### <a name="example"></a>範例
下列 HTTP POST 會要求提供 https://service.contoso.com/ Web 服務的[存取權杖](access-tokens.md)。 `client_id` 會識別要求存取權杖的 Web 服務。

```
POST /contoso.com/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=625bc9f6-3bf6-4b6d-94ba-e97cf07a22de&client_secret=qkDwDJlDfig2IpeuUZYKH1Wb8q1V0ju6sILxQQqhJ+s=&resource=https%3A%2F%2Fservice.contoso.com%2F
```

### <a name="second-case-access-token-request-with-a-certificate"></a>第二種情況︰使用憑證的存取權杖要求
使用憑證的服務對服務存取權杖要求包含下列參數：

| 參數 |  | 說明 |
| --- | --- | --- |
| grant_type |必要 |指定要求的回應類型。 在用戶端認證授與流程中，值必須是 **client_credentials**。 |
| client_id |必要 |指定呼叫端 Web 服務的 Azure AD 用戶端識別碼。 若要尋找呼叫應用程式的用戶端識別碼，請在 [Azure 入口網站](https://portal.azure.com)中按一下 [Azure Active Directory]，並按一下 [應用程式註冊]，再按一下應用程式。 client_id 是*應用程式 ID* |
| client_assertion_type |必要 |值必須是 `urn:ietf:params:oauth:client-assertion-type:jwt-bearer` |
| client_assertion |必要 | 您必須建立判斷提示 (JSON Web 權杖)，並使用註冊的憑證來簽署，以作為應用程式的認證。 請參閱[憑證認證](active-directory-certificate-credentials.md)，以了解如何註冊您的憑證與判斷提示的格式。|
| 資源 | 必要 |輸入接收端 Web 服務的應用程式識別碼 URI。 若要尋找應用程式識別碼 URI，請在 Azure 入口網站中，按一下 [Azure Active Directory]，並按一下 [應用程式註冊]，再按一下服務應用程式，接著按一下 [設定] 和 [屬性]。 |

請注意，在透過共用密碼要求的情況中，參數幾乎相同，不同之處在於使用下列兩個參數來取代 client_secret 參數：client_assertion_type 和 client_assertion。

#### <a name="example"></a>範例
下列 HTTP POST 會使用憑證要求提供 https://service.contoso.com/ Web 服務的存取權杖。 `client_id` 會識別要求存取權杖的 Web 服務。

```
POST /<tenant_id>/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

resource=https%3A%2F%contoso.onmicrosoft.com%2Ffc7664b4-cdd6-43e1-9365-c2e1c4e1b3bf&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg&grant_type=client_credentials
```

### <a name="service-to-service-access-token-response"></a>服務對服務存取權杖回應

成功的回應中包含 JSON OAuth 2.0 回應與下列參數：

| 參數 | 說明 |
| --- | --- |
| access_token |所要求的存取權杖。 呼叫端 Web 服務可以使用此權杖來向接收端 Web 服務進行驗證。 |
| token_type |表示權杖類型值。 Azure AD 唯一支援的類型是 [持有人] 。 如需持有人權杖的詳細資訊，請參閱 [OAuth 2.0 授權架構︰持有人權杖用法 (RFC 6750)](http://www.rfc-editor.org/rfc/rfc6750.txt)。 |
| expires_in |存取權杖的有效期 (以秒為單位)。 |
| expires_on |存取權杖的到期時間。 日期會表示為從 1970-01-01T0:0:0Z UTC 至到期時間的秒數。 這個值用來判斷快取權杖的存留期。 |
| not_before |存取權杖自此時間開始可使用。 日期會表示為從 1970-01-01T0:0:0Z UTC 至權杖的有效時間。|
| resource |接收端 Web 服務的應用程式識別碼 URI。 |

#### <a name="example-of-response"></a>回應範例
下列範例顯示向 Web 服務所提出之存取權杖要求的成功回應。

```
{
"access_token":"eyJhbGciOiJSUzI1NiIsIng1dCI6IjdkRC1nZWNOZ1gxWmY3R0xrT3ZwT0IyZGNWQSIsInR5cCI6IkpXVCJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0ODI2NywibmJmIjoxMzg4NDQ4MjY3LCJleHAiOjEzODg0NTIxNjcsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6ImE5OTE5MTYyLTkyMTctNDlkYS1hZTIyLWYxMTM3YzI1Y2RlYSIsInN1YiI6ImE5OTE5MTYyLTkyMTctNDlkYS1hZTIyLWYxMTM3YzI1Y2RlYSIsImlkcCI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZS8iLCJhcHBpZCI6ImQxN2QxNWJjLWM1NzYtNDFlNS05MjdmLWRiNWYzMGRkNThmMSIsImFwcGlkYWNyIjoiMSJ9.aqtfJ7G37CpKV901Vm9sGiQhde0WMg6luYJR4wuNR2ffaQsVPPpKirM5rbc6o5CmW1OtmaAIdwDcL6i9ZT9ooIIicSRrjCYMYWHX08ip-tj-uWUihGztI02xKdWiycItpWiHxapQm0a8Ti1CWRjJghORC1B1-fah_yWx6Cjuf4QE8xJcu-ZHX0pVZNPX22PHYV5Km-vPTq2HtIqdboKyZy3Y4y3geOrRIFElZYoqjqSv5q9Jgtj5ERsNQIjefpyxW3EwPtFqMcDm4ebiAEpoEWRN4QYOMxnC9OUBeG9oLA0lTfmhgHLAtvJogJcYFzwngTsVo6HznsvPWy7UP3MINA",
"token_type":"Bearer",
"expires_in":"3599",
"expires_on":"1388452167",
"resource":"https://service.contoso.com/"
}
```

## <a name="see-also"></a>另請參閱
* [Azure AD 中的 OAuth 2.0](v1-protocols-oauth-code.md)
* [使用共用密碼的服務對服務呼叫的 C# 中的範例](https://github.com/Azure-Samples/active-directory-dotnet-daemon)和[使用憑證的服務對服務呼叫的 C# 中範例](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)
