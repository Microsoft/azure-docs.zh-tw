---
title: Azure 單一登入 SAML 通訊協定 | Microsoft Docs
description: 本文說明 Azure Active Directory 中的單一登入 SAML 通訊協定
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: ad8437f5-b887-41ff-bd77-779ddafc33fb
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: celested
ms.custom: aaddev
ms.reviewer: hirsin
ms.openlocfilehash: edb8ae501548775932a259621c19acece474018d
ms.sourcegitcommit: 615403e8c5045ff6629c0433ef19e8e127fe58ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39580130"
---
# <a name="single-sign-on-saml-protocol"></a>單一登入 SAML 通訊協定

本文涵蓋 Azure Active Directory (Azure AD) 針對單一登入所支援的 SAML 2.0 驗證要求和回應。

下面的通訊協定圖表說明單一登入順序。 雲端服務 (服務提供者) 使用 HTTP 重新導向繫結傳遞 `AuthnRequest` (驗證要求) 元素至 Azure AD (識別提供者)。 Azure AD 接著使用 HTTP POST 繫結將 `Response` 元素張貼至雲端服務。

![單一登入工作流程](./media/single-sign-on-saml-protocol/active-directory-saml-single-sign-on-workflow.png)

## <a name="authnrequest"></a>AuthnRequest

為了要求使用者驗證，雲端服務會將 `AuthnRequest` 元素傳送至 Azure AD。 SAML 2.0 `AuthnRequest` 範例會像下面這樣：

```
<samlp:AuthnRequest
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="id6c1c178c166d486687be4aaf5e482730"
Version="2.0" IssueInstant="2013-03-18T03:28:54.1839884Z"
xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
</samlp:AuthnRequest>
```

| 參數 |  | 說明 |
| --- | --- | --- |
| ID | 必要 | Azure AD 使用這個屬性來填入所傳回回應的 `InResponseTo` 屬性。 識別碼的開頭不能是數字，因此常見的策略是在 GUID 的字串表示法前面加上 "id" 等字串。 例如， `id6c1c178c166d486687be4aaf5e482730` 便是有效的識別碼。 |
| 版本 | 必要 | 此參數應該設定為 **2.0**。 |
| IssueInstant | 必要 | 這是具有 UTC 值和 [來回行程格式 ("o")](https://msdn.microsoft.com/library/az4se3k1.aspx)的日期時間字串。 Azure AD 必須要有這種類型的日期時間值，但不會評估或使用此值。 |
| AssertionConsumerServiceUrl | 選用 | 如果提供，此參數必須符合 Azure AD 中雲端服務的 `RedirectUri`。 |
| ForceAuthn | 選用 | 這是布林值。 如果為 true，表示即使使用者在 Azure AD 中具有有效的工作階段，也會強制使用者重新驗證。 |
| IsPassive | 選用 | 這是布林值，指定 Azure AD 是否以無訊息模式驗證使用者，不需要使用者互動，如果有工作階段 cookie 的話則使用此 cookie。 如果是這種情況，Azure AD 會嘗試使用工作階段 cookie 驗證使用者。 |

其他所有 `AuthnRequest` 屬性 (例如 Consent、Destination、AssertionConsumerServiceIndex、AttributeConsumerServiceIndex 和 ProviderName) 會 **遭到忽略**。

Azure AD 也會忽略 `AuthnRequest` 中的 `Conditions` 元素。

### <a name="issuer"></a>簽發者

`AuthnRequest` 中的 `Issuer` 元素必須完全符合 Azure AD 中雲端服務的其中一個 **ServicePrincipalNames**。 一般而言，這會設定為應用程式註冊期間指定的 **應用程式識別碼 URI** 。

包含 `Issuer` 元素的 SAML 摘錄看起來會像下列範例︰

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://www.contoso.com</Issuer>
```

### <a name="nameidpolicy"></a>NameIDPolicy

這個元素要求回應中使用特定名稱識別碼格式，在傳送至 Azure AD 的 `AuthnRequest` 元素中是選擇性的。

`NameIdPolicy` 元素看起來會像下列範例︰

```
<NameIDPolicy Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent"/>
```

如果提供 `NameIDPolicy`，您可以包含其選擇性的 `Format` 屬性。 `Format` 屬性只能有下列其中一個值；其他任何值都會產生錯誤。

* `urn:oasis:names:tc:SAML:2.0:nameid-format:persistent`：Azure Active Directory 發出 NameID 宣告來做為成對識別碼。
* `urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress`：Azure Active Directory 發出電子郵件地址格式的 NameID 宣告。
* `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`︰這個值允許 Azure Active Directory 選取宣告格式。 Azure Active Directory 發出 NameID 來做為成對識別碼。
* `urn:oasis:names:tc:SAML:2.0:nameid-format:transient`：Azure Active Directory 發出 NameID 宣告做為隨機產生的值，此值對目前的 SSO 作業來說是唯一值。 這表示此值只是暫時性的，而且不能用來識別驗證的使用者。

Azure AD 會忽略 `AllowCreate` 屬性。

### <a name="requestauthncontext"></a>RequestAuthnContext
`RequestedAuthnContext` 元素會指定所需的驗證方法。 在傳送至 Azure AD 的 `AuthnRequest` 元素中，它是選擇性的。 Azure AD 只支援一個 `AuthnContextClassRef` 值︰`urn:oasis:names:tc:SAML:2.0:ac:classes:Password`。

### <a name="scoping"></a>範圍
包含識別提供者清單的 `Scoping` 元素在傳送至 Azure AD 的 `AuthnRequest` 元素中是選擇性的。

如果提供，請勿包含 `ProxyCount` 屬性、`IDPListOption` 或 `RequesterID` 元素，因為它們不受支援。

### <a name="signature"></a>簽章
請勿在 `AuthnRequest` 元素中包含 `Signature` 元素，因為 Azure AD 不支援簽署的驗證要求。

### <a name="subject"></a>主體
Azure AD 會忽略 `AuthnRequest` 元素中的 `Subject` 元素。

## <a name="response"></a>Response
當要求的登入成功完成時，Azure AD 會將回應張貼至雲端服務。 登入嘗試成功的回應看起來會像下列範例︰

```
<samlp:Response ID="_a4958bfd-e107-4e67-b06d-0d85ade2e76a" Version="2.0" IssueInstant="2013-03-18T07:38:15.144Z" Destination="https://contoso.com/identity/inboundsso.aspx" InResponseTo="id758d0ef385634593a77bdf7e632984b6" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
    ...
  </ds:Signature>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
  </samlp:Status>
  <Assertion ID="_bf9c623d-cc20-407a-9a59-c2d0aee84d12" IssueInstant="2013-03-18T07:38:15.144Z" Version="2.0" xmlns="urn:oasis:names:tc:SAML:2.0:assertion">
    <Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
    <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
      ...
    </ds:Signature>
    <Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
    </Subject>
    <Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
    </Conditions>
    <AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
    </AttributeStatement>
    <AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
    </AuthnStatement>
  </Assertion>
</samlp:Response>
```

### <a name="response"></a>Response

`Response` 元素包含授權要求的結果。 Azure AD 會設定 `Response` 元素中的 `ID`、`Version` 和 `IssueInstant` 值。 它也會設定下列屬性︰

* `Destination`︰當登入順利完成時，這會設定為服務提供者 (雲端服務) 的 `RedirectUri`。
* `InResponseTo`︰這會設定為起始回應的 `AuthnRequest` 元素的 `ID` 屬性。

### <a name="issuer"></a>簽發者

Azure AD 會將 `Issuer` 元素設為 `https://login.microsoftonline.com/<TenantIDGUID>/`，其中，<TenantIDGUID> 是 Azure AD 租用戶的租用戶識別碼。

例如，具有 Issuer 元素的回應看起來會像下列範例︰

```
<Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion"> https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

### <a name="status"></a>狀態

`Status` 元素會傳遞登入的成功或失敗。 它包含 `StatusCode` 元素，此元素中包含一個代碼或一組巢狀代碼來表示要求的狀態。 它也包含 `StatusMessage` 元素，此元素中包含登入程序期間所產生的自訂錯誤訊息。

<!-- TODO: Add a authentication protocol error reference -->

下列範例是登入嘗試失敗的 SAML 回應。

```
<samlp:Response ID="_f0961a83-d071-4be5-a18c-9ae7b22987a4" Version="2.0" IssueInstant="2013-03-18T08:49:24.405Z" InResponseTo="iddce91f96e56747b5ace6d2e2aa9d4f8c" xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
  <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">https://sts.windows.net/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
  <samlp:Status>
    <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Requester">
      <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:RequestUnsupported" />
    </samlp:StatusCode>
    <samlp:StatusMessage>AADSTS75006: An error occurred while processing a SAML2 Authentication request. AADSTS90011: The SAML authentication request property 'NameIdentifierPolicy/SPNameQualifier' is not supported.
Trace ID: 66febed4-e737-49ff-ac23-464ba090d57c
Timestamp: 2013-03-18 08:49:24Z</samlp:StatusMessage>
  </samlp:Status>
```

### <a name="assertion"></a>Assertion

除了 `ID`、`IssueInstant` 和 `Version`，Azure AD 還會在回應的 `Assertion` 元素中設定下列元素。

#### <a name="issuer"></a>簽發者

這會設為 `https://sts.windows.net/<TenantIDGUID>/`，其中，<TenantIDGUID> 是 Azure AD 租用戶的租用戶識別碼。

```
<Issuer>https://login.microsoftonline.com/82869000-6ad1-48f0-8171-272ed18796e9/</Issuer>
```

#### <a name="signature"></a>簽章

Azure AD 會簽署判斷提示以回應成功的登入。 `Signature` 元素包含數位簽章，可供雲端服務用來驗證來源以確認判斷提示的完整性。

為了產生此數位簽章，Azure AD 會在其中繼資料文件的 `IDPSSODescriptor` 元素中使用簽署金鑰。

```
<ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
      digital_signature_here
    </ds:Signature>
```

#### <a name="subject"></a>主體

這會指定判斷提示中陳述式主旨的主體。 它包含 `NameID` 元素，其代表已驗證的使用者。 `NameID` 值為目標識別碼，其只會導向身為權杖對象的服務提供者。 它是持續性的 - 可撤銷，但絕對不會重新指派。 它也是不透明的，因為它不會揭露使用者的相關資訊，也不能當做屬性查詢的識別碼。

`SubjectConfirmation` 元素的 `Method` 屬性一律會設定為 `urn:oasis:names:tc:SAML:2.0:cm:bearer`。

```
<Subject>
      <NameID>Uz2Pqz1X7pxe4XLWxV9KJQ+n59d573SepSAkuYKSde8=</NameID>
      <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
        <SubjectConfirmationData InResponseTo="id758d0ef385634593a77bdf7e632984b6" NotOnOrAfter="2013-03-18T07:43:15.144Z" Recipient="https://contoso.com/identity/inboundsso.aspx" />
      </SubjectConfirmation>
</Subject>
```

#### <a name="conditions"></a>條件

這個元素會指定條件來定義可接受的 SAML 判斷提示用法。

```
<Conditions NotBefore="2013-03-18T07:38:15.128Z" NotOnOrAfter="2013-03-18T08:48:15.128Z">
      <AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
      </AudienceRestriction>
</Conditions>
```

`NotBefore` 和 `NotOnOrAfter` 屬性會指定判斷提示的有效間隔期間。

* `NotBefore` 屬性值等於或稍微晚於 (不到一秒) `Assertion` 元素的 `IssueInstant` 屬性值。 Azure AD 不會考慮本身與雲端服務 (服務提供者) 之間的任何時間差，而且不會對此時間加上任何緩衝。
* `NotOnOrAfter` 屬性值比 `NotBefore` 屬性值晚 70 分鐘。

#### <a name="audience"></a>對象

這包含可識別適用對象的 URI。 Azure AD 會將這個元素的值設定為起始登入的 `AuthnRequest` 的 `Issuer` 元素值。 若要評估 `Audience` 值，請使用應用程式註冊期間指定的 `App ID URI` 值。

```
<AudienceRestriction>
        <Audience>https://www.contoso.com</Audience>
</AudienceRestriction>
```

和 `Issuer` 值一樣，`Audience` 值必須完全符合代表 Azure AD 中雲端服務的其中一個服務主體名稱。 不過，如果 `Issuer` 元素值不是 URI 值，回應中的 `Audience` 值是前面加上 `spn:` 的 `Issuer` 值。

#### <a name="attributestatement"></a>AttributeStatement

這包含有關主體或使用者的宣告。 下列摘錄包含範例 `AttributeStatement` 元素。 省略符號表示此元素可能包含多個屬性和屬性值。

```
<AttributeStatement>
      <Attribute Name="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name">
        <AttributeValue>testuser@contoso.com</AttributeValue>
      </Attribute>
      <Attribute Name="http://schemas.microsoft.com/identity/claims/objectidentifier">
        <AttributeValue>3F2504E0-4F89-11D3-9A0C-0305E82C3301</AttributeValue>
      </Attribute>
      ...
</AttributeStatement>
```        

* **Name 宣告** - `Name` 屬性的值 (`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`) 是已驗證使用者的使用者主體名稱，例如 `testuser@managedtenant.com`。
* **ObjectIdentifier 宣告** - `ObjectIdentifier` 屬性的值 (`http://schemas.microsoft.com/identity/claims/objectidentifier`) 是目錄物件的 `ObjectId`，代表 Azure AD 中已驗證的使用者。 `ObjectId` 是不可變的、全域唯一的，且重複使用已驗證使用者的安全識別碼。

#### <a name="authnstatement"></a>AuthnStatement

此元素會宣稱判斷提示主體已在特定時間以特定方式進行驗證。

* `AuthnInstant` 屬性會指定使用者向 Azure AD 驗證的時間。
* `AuthnContext` 元素會指定用來驗證使用者的驗證內容。

```
<AuthnStatement AuthnInstant="2013-03-18T07:33:56.000Z" SessionIndex="_bf9c623d-cc20-407a-9a59-c2d0aee84d12">
      <AuthnContext>
        <AuthnContextClassRef> urn:oasis:names:tc:SAML:2.0:ac:classes:Password</AuthnContextClassRef>
      </AuthnContext>
</AuthnStatement>
```
