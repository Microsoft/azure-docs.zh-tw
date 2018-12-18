---
title: 了解 Azure AD 支援的不同權杖和宣告類型 | Microsoft Docs
description: 可供了解及評估 Azure Active Directory (AAD) 所簽發之 SAML 2.0 和 JSON Web Token (JWT) 權杖中的宣告的指南。
documentationcenter: na
author: CelesteDG
services: active-directory
manager: mtillman
editor: ''
ms.assetid: 166aa18e-1746-4c5e-b382-68338af921e2
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/22/2018
ms.author: celested
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 8f24f25a4eb4db9e9f3e0b4569874dab94ef11b5
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48900953"
---
# <a name="azure-ad-saml-token-reference"></a>Azure AD SAML 權杖參考

Azure Active Directory (Azure AD) 會在處理每個驗證流程時發出數種安全性權杖。 本文件說明每種權杖的格式、安全性特性和內容。

## <a name="claims-in-saml-tokens"></a>SAML 權杖中的宣告

> [!div class="mx-codeBreakAll"]
| 名稱 | 對等的 JWT 宣告 | 說明 | 範例 |
| --- | --- | --- | ------------|
|對象 | `aud` |權杖的預定接收者。 接收權杖的應用程式必須確認對象值正確無誤，並拒絕任何適用於不同對象的權杖。 | `<AudienceRestriction>`<br>`<Audience>`<br>`https://contoso.com`<br>`</Audience>`<br>`</AudienceRestriction>`  |
| 驗證時刻 | |記錄驗證發生的日期和時間。 | `<AuthnStatement AuthnInstant="2011-12-29T05:35:22.000Z">` | 
|驗證方法 | `amr` |識別如何驗證權杖的主體。 | `<AuthnContextClassRef>`<br>`http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod/password`<br>`</AuthnContextClassRef>` |
|名字 | `given_name` |提供使用者的名字 (如 Azure AD 使用者物件上所設定)。 | `<Attribute Name=”http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname”>`<br>`<AttributeValue>Frank<AttributeValue>`  |
|群組 | `groups` |提供代表主體群組成員資格的物件識別碼。 這些值都是唯一的 (請參閱「物件識別碼」)，而且可安全地用來管理存取權，例如強制授權以存取資源。 群組宣告中包含的群組會透過應用程式資訊清單的 "groupMembershipClaims"屬性，針對每個應用程式進行設定。 Null 值將會排除所有群組，"SecurityGroup" 值只會包含 Active Directory 安全性群組成員資格，而 "All" 值將會包含安全性群組和 Office 365 通訊群組清單。 <br><br> **注意**： <br> 如果使用者所屬的群組數目超過限制 (SAML 為 150，JWT 為 200)，則會將超額宣告新增至指向宣告來源 (包含使用者群組清單的 Graph 端點)。 (in . | `<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">`<br>`<AttributeValue>07dd8a60-bf6d-4e17-8844-230b77145381</AttributeValue>` |
| 群組超額指標 | `groups:src1` | 若為沒有長度限制 (請參閱上面的 `hasgroups`) 但是對權杖而言仍然太大的權杖要求，則會包含使用者的完整 groups 清單連結。 對於 SAML，這會新增為新的宣告，取代 `groups` 宣告。 | `<Attribute Name=” http://schemas.microsoft.com/claims/groups.link”>`<br>`<AttributeValue>https://graph.windows.net/{tenantID}/users/{userID}/getMemberObjects<AttributeValue>` |
|身分識別提供者 | `idp` |記錄驗證權杖主體的身分識別提供者。 除非使用者帳戶位於與簽發者不同的租用戶中，否則這個值與 Issuer 宣告值相同。 | `<Attribute Name=” http://schemas.microsoft.com/identity/claims/identityprovider”>`<br>`<AttributeValue>https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/<AttributeValue>` |
|IssuedAt | `iat` |儲存權杖簽發的時間。 這通常用來測量權杖有效時間。 | `<Assertion ID="_d5ec7a9b-8d8f-4b44-8c94-9812612142be" IssueInstant="2014-01-06T20:20:23.085Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">` |
|簽發者 | `iss` |識別負責建構並傳回權杖的 Security Token Service (STS)。 在 Azure AD 傳回的權杖中，簽發者為 sts.windows.net。 簽發者宣告值中的 GUID 是 Azure AD 目錄的租用戶識別碼。 租用戶識別碼是目錄的不可變且可靠的識別碼。 | `<Issuer>https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/</Issuer>` |
|姓氏 | `family_name` |提供使用者的姓氏 (如 Azure AD 使用者物件中所定義)。 | `<Attribute Name=” http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname”>`<br>`<AttributeValue>Miller<AttributeValue>` |
|名稱 | `unique_name` |提供人類看得懂的值，用以識別權杖的主體。 此值不保證是租用戶中的唯一值，而且僅限用於顯示目的。 | `<Attribute Name=”http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name”>`<br>`<AttributeValue>frankm@contoso.com<AttributeValue>`|
|物件識別碼 | `oid` |包含 Azure AD 中物件的唯一識別碼。 這個值不可變，而且無法重新指派或重複使用。 使用物件識別碼來識別 Azure AD 查詢中的物件。 | `<Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">`<br>`<AttributeValue>528b2ac2-aa9c-45e1-88d4-959b53bc7dd0<AttributeValue>` |
|角色 | `roles` |表示透過群組成員資格，直接或間接授與主體的所有應用程式角色，而且可用來強制執行角色型存取控制。 應用程式角色是透過應用程式資訊清單的 `appRoles` 屬性，針對每個應用程式進行定義。 每個應用程式角色的 `value` 屬性就是出現在角色宣告中的值。 | `<Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/role">`|
|主體 | `sub` |識別權杖判斷提示其相關資訊的主體，例如應用程式的使用者。 這個值不可變，而且無法重新指派或重複使用，因此可用來安全地執行授權檢查。 因為主體一律存在於 Azure AD 簽發的權杖中，所以建議您將此值使用於一般用途授權系統中。 <br> `SubjectConfirmation` 不是宣告。 它會描述如何驗證權杖的主體。 `Bearer` 表示主體已經由持有權杖來確認。 | `<Subject>`<br>`<NameID>S40rgb3XjhFTv6EQTETkEzcgVmToHKRkZUIsJlmLdVc</NameID>`<br>`<SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />`<br>`</Subject>`|
|租用戶識別碼 | `tid` |不可變、無法重複使用的識別碼，用以識別簽發權杖的目錄租用戶。 您可以使用此值來存取多租用戶應用程式中的租用戶特定目錄資源。 例如，您可以使用此值來識別在圖形 API 呼叫中的租用戶。 | `<Attribute Name=”http://schemas.microsoft.com/identity/claims/tenantid”>`<br>`<AttributeValue>cbb1a5ac-f33b-45fa-9bf5-f37db0fed422<AttributeValue>`|
|權杖存留期 | `nbf`、`exp` |定義權杖有效的時間間隔。 驗證權杖的服務應確認目前的日期在權杖存留期內，否則應該拒絕此權杖。 此服務最多允許超過權杖存留期範圍五分鐘，以考量 Azure AD 與服務之間的時鐘時間差異 (「時間偏差」)。 | `<Conditions`<br>`NotBefore="2013-03-18T21:32:51.261Z"`<br>`NotOnOrAfter="2013-03-18T22:32:51.261Z"`<br>`>` <br>|

## <a name="sample-saml-token"></a>SAML 權杖範例

這是典型的 SAML 權杖範例。

    <?xml version="1.0" encoding="UTF-8"?>
    <t:RequestSecurityTokenResponse xmlns:t="http://schemas.xmlsoap.org/ws/2005/02/trust">
      <t:Lifetime>
        <wsu:Created xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T05:15:47.060Z</wsu:Created>
        <wsu:Expires xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">2014-12-24T06:15:47.060Z</wsu:Expires>
      </t:Lifetime>
      <wsp:AppliesTo xmlns:wsp="http://schemas.xmlsoap.org/ws/2004/09/policy">
        <EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
          <Address>https://contoso.onmicrosoft.com/MyWebApp</Address>
        </EndpointReference>
      </wsp:AppliesTo>
      <t:RequestedSecurityToken>
        <Assertion xmlns="urn:oasis:names:tc:SAML:2.0:assertion" ID="_3ef08993-846b-41de-99df-b7f3ff77671b" IssueInstant="2014-12-24T05:20:47.060Z" Version="2.0">
          <Issuer>https://sts.windows.net/b9411234-09af-49c2-b0c3-653adc1f376e/</Issuer>
          <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
              <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
              <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
              <ds:Reference URI="#_3ef08993-846b-41de-99df-b7f3ff77671b">
                <ds:Transforms>
                  <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                  <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                </ds:Transforms>
                <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
                <ds:DigestValue>cV1J580U1pD24hEyGuAxrbtgROVyghCqI32UkER/nDY=</ds:DigestValue>
              </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>j+zPf6mti8Rq4Kyw2NU2nnu0pbJU1z5bR/zDaKaO7FCTdmjUzAvIVfF8pspVR6CbzcYM3HOAmLhuWmBkAAk6qQUBmKsw+XlmF/pB/ivJFdgZSLrtlBs1P/WBV3t04x6fRW4FcIDzh8KhctzJZfS5wGCfYw95er7WJxJi0nU41d7j5HRDidBoXgP755jQu2ZER7wOYZr6ff+ha+/Aj3UMw+8ZtC+WCJC3yyENHDAnp2RfgdElJal68enn668fk8pBDjKDGzbNBO6qBgFPaBT65YvE/tkEmrUxdWkmUKv3y7JWzUYNMD9oUlut93UTyTAIGOs5fvP9ZfK2vNeMVJW7Xg==</ds:SignatureValue>
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
              <X509Data>
                <X509Certificate>MIIDPjCCAabcAwIBAgIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTQwMTAxMDcwMDAwWhcNMTYwMTAxMDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAkSCWg6q9iYxvJE2NIhSyOiKvqoWCO2GFipgH0sTSAs5FalHQosk9ZNTztX0ywS/AHsBeQPqYygfYVJL6/EgzVuwRk5txr9e3n1uml94fLyq/AXbwo9yAduf4dCHTP8CWR1dnDR+Qnz/4PYlWVEuuHHONOw/blbfdMjhY+C/BYM2E3pRxbohBb3x//CfueV7ddz2LYiH3wjz0QS/7kjPiNCsXcNyKQEOTkbHFi3mu0u13SQwNddhcynd/GTgWN8A+6SN1r4hzpjFKFLbZnBt77ACSiYx+IHK4Mp+NaVEi5wQtSsjQtI++XsokxRDqYLwus1I1SihgbV/STTg5enufuwIDAQABo2IwYDBeBgNVHQEEVzBVgBDLebM6bK3BjWGqIBrBNFeNoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQsRiM0jheFZhKk49YD0SK1TAJBgUrDgMCHQUAA4IBAQCJ4JApryF77EKC4zF5bUaBLQHQ1PNtA1uMDbdNVGKCmSp8M65b8h0NwlIjGGGy/unK8P6jWFdm5IlZ0YPTOgzcRZguXDPj7ajyvlVEQ2K2ICvTYiRQqrOhEhZMSSZsTKXFVwNfW6ADDkN3bvVOVbtpty+nBY5UqnI7xbcoHLZ4wYD251uj5+lo13YLnsVrmQ16NCBYq2nQFNPuNJw6t3XUbwBHXpF46aLT1/eGf/7Xx6iy8yPJX4DyrpFTutDz882RWofGEO5t4Cw+zZg70dJ/hH/ODYRMorfXEW+8uKmXMKmX2wyxMKvfiPbTy5LmAU8Jvjs2tLg4rOBcXWLAIarZ</X509Certificate>
              </X509Data>
            </KeyInfo>
          </ds:Signature>
          <Subject>
            <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">m_H3naDei2LNxUmEcWd0BZlNi_jVET1pMLR6iQSuYmo</NameID>
            <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer" />
          </Subject>
          <Conditions NotBefore="2014-12-24T05:15:47.060Z" NotOnOrAfter="2014-12-24T06:15:47.060Z">
            <AudienceRestriction>
              <Audience>https://contoso.onmicrosoft.com/MyWebApp</Audience>
            </AudienceRestriction>
          </Conditions>
          <AttributeStatement>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
              <AttributeValue>a1addde8-e4f9-4571-ad93-3059e3750d23</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/tenantid">
              <AttributeValue>b9411234-09af-49c2-b0c3-653adc1f376e</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
              <AttributeValue>sample.admin@contoso.onmicrosoft.com</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname">
              <AttributeValue>Admin</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname">
              <AttributeValue>Sample</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/ws/2008/06/identity/claims/groups">
              <AttributeValue>5581e43f-6096-41d4-8ffa-04e560bab39d</AttributeValue>
              <AttributeValue>07dd8a89-bf6d-4e81-8844-230b77145381</AttributeValue>
              <AttributeValue>0e129f4g-6b0a-4944-982d-f776000632af</AttributeValue>
              <AttributeValue>3ee07328-52ef-4739-a89b-109708c22fb5</AttributeValue>
              <AttributeValue>329k14b3-1851-4b94-947f-9a4dacb595f4</AttributeValue>
              <AttributeValue>6e32c650-9b0a-4491-b429-6c60d2ca9a42</AttributeValue>
              <AttributeValue>f3a169a7-9a58-4e8f-9d47-b70029v07424</AttributeValue>
              <AttributeValue>8e2c86b2-b1ad-476d-9574-544d155aa6ff</AttributeValue>
              <AttributeValue>1bf80264-ff24-4866-b22c-6212e5b9a847</AttributeValue>
              <AttributeValue>4075f9c3-072d-4c32-b542-03e6bc678f3e</AttributeValue>
              <AttributeValue>76f80527-f2cd-46f4-8c52-8jvd8bc749b1</AttributeValue>
              <AttributeValue>0ba31460-44d0-42b5-b90c-47b3fcc48e35</AttributeValue>
              <AttributeValue>edd41703-8652-4948-94a7-2d917bba7667</AttributeValue>
            </Attribute>
            <Attribute Name="http://schemas.microsoft.com/identity/claims/identityprovider">
              <AttributeValue>https://sts.windows.net/b9411234-09af-49c2-b0c3-653adc1f376e/</AttributeValue>
            </Attribute>
          </AttributeStatement>
          <AuthnStatement AuthnInstant="2014-12-23T18:51:11.000Z">
            <AuthnContext>
              <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
            </AuthnContext>
          </AuthnStatement>
        </Assertion>
      </t:RequestedSecurityToken>
      <t:RequestedAttachedReference>
        <SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
          <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
        </SecurityTokenReference>
      </t:RequestedAttachedReference>
      <t:RequestedUnattachedReference>
        <SecurityTokenReference xmlns="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:d3p1="http://docs.oasis-open.org/wss/oasis-wss-wssecurity-secext-1.1.xsd" d3p1:TokenType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0">
          <KeyIdentifier ValueType="http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLID">_3ef08993-846b-41de-99df-b7f3ff77671b</KeyIdentifier>
        </SecurityTokenReference>
      </t:RequestedUnattachedReference>
      <t:TokenType>http://docs.oasis-open.org/wss/oasis-wss-saml-token-profile-1.1#SAMLV2.0</t:TokenType>
      <t:RequestType>http://schemas.xmlsoap.org/ws/2005/02/trust/Issue</t:RequestType>
      <t:KeyType>http://schemas.xmlsoap.org/ws/2005/05/identity/NoProofKey</t:KeyType>
    </t:RequestSecurityTokenResponse>

## <a name="related-content"></a>相關內容
* 若要深入了解透過 Azure AD Graph API 管理權杖存留期原則，請參閱 Azure AD Graph [原則作業](https://msdn.microsoft.com/library/azure/ad/graph/api/policy-operations)和[原則實體](https://msdn.microsoft.com/library/azure/ad/graph/api/entity-and-complex-type-reference#policy-entity)。
* 如需透過 PowerShell Cmdlet 管理原則的詳細資訊和範例，包括範例，請參閱[在 Azure AD 中設定權杖存留期](active-directory-configurable-token-lifetimes.md)。 
* 將[自訂和選擇性宣告](active-directory-optional-claims.md)新增至應用程式的權杖。
* 搭配使用 [SAML 與單一登入 (SSO)](single-sign-on-saml-protocol.md)。
* 使用 [Azure 單一登出 SAML 通訊協定](single-sign-out-saml-protocol.md)
