---
title: Azure AD 同盟中繼資料 | Microsoft Docs
description: 本文說明 Azure Active Directory 針對接受 Azure Active Directory 權杖之服務所發佈的同盟中繼資料文件。
services: active-directory
documentationcenter: .net
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: c2d5f80b-aa74-452c-955b-d8eb3ed62652
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/07/2017
ms.author: celested
ms.reviewer: hirsin, dastrock
ms.custom: aaddev
ms.openlocfilehash: cf8de9e6061df59782a6edbccf47341222d256f4
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162216"
---
# <a name="federation-metadata"></a>同盟中繼資料
Azure Active Directory (Azure AD) 會針對服務發佈同盟中繼資料文件，而該服務已設定為接受由 Azure AD 簽發的安全性權杖。 [Web 服務同盟語言 (WS-Federation) 1.2 版](http://docs.oasis-open.org/wsfed/federation/v1.2/os/ws-federation-1.2-spec-os.html) (該說明為[適用於 OASIS 安全性聲明標記語言 (SAML) 的中繼資料 2.0 版 的延伸資訊](http://docs.oasis-open.org/security/saml/v2.0/saml-metadata-2.0-os.pdf)) 中說明了同盟中繼資料文件格式。

## <a name="tenant-specific-and-tenant-independent-metadata-endpoints"></a>租用戶專屬與租用戶獨立中繼資料端點
Azure AD 會發佈租用戶專屬與租用戶獨立端點。

租用戶專屬端點是專為特定租用戶所設計的。 租用戶專屬同盟中繼資料包含租用戶相關資訊，其中包括租用戶專屬簽發者和端點資訊。 限制存取單一租用戶的應用程式會使用租用戶專屬端點。

租用戶獨立端點會提供通用資訊給所有 Azure AD 租用戶。 本資訊適用裝載於 login.microsoftonline.com  的租用戶，並由所有租用戶共用。 建議多租用戶應用程式使用租用戶獨立端點，因為它們並不與任何特定租用戶相關聯。

## <a name="federation-metadata-endpoints"></a>同盟中繼資料端點
Azure AD 會在 `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`發佈同盟中繼資料。

若為**租用戶專屬端點**，`TenantDomainName` 可以是下列其中一種類型：

* Azure AD 租用戶已註冊的網域名稱，例如： `contoso.onmicrosoft.com`。
* 網域的不可變租用戶識別碼，例如 `72f988bf-86f1-41af-91ab-2d7cd011db45`。

若為**租用戶獨立端點**，`TenantDomainName` 是 `common`。 本文件只列出通用於所有 Azure AD 租用戶 (裝載於 login.microsoftonline.com) 的同盟中繼資料項目。

例如，租用戶專屬端點可能是 `https://login.microsoftonline.com/contoso.onmicrosoft.com/FederationMetadata/2007-06/FederationMetadata.xml`。 租用戶獨立端點是 [https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml](https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml)。 您可以在瀏覽器中輸入此 URL 以檢視同盟中繼資料文件。

## <a name="contents-of-federation-metadata"></a>同盟中繼資料內容
以下小節提供使用 Azure AD 所簽發之權杖的服務所需的資訊。

### <a name="entity-id"></a>實體識別碼
`EntityDescriptor` 項目包含 `EntityID` 屬性。 `EntityID` 屬性的值代表簽發者 (也就是簽發權杖的 Security Token Service (STS))。 請務必在收到權杖時驗證簽發者。

下列中繼資料說明了含有 `EntityID` 項目的範例租用戶專屬 `EntityDescriptor` 項目。

```
<EntityDescriptor
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="_b827a749-cfcb-46b3-ab8b-9f6d14a1294b"
entityID="https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db45/">
```
您可以使用租用戶識別碼來取代租用戶獨立端點中的租用戶識別碼，以建立租用戶特定的 `EntityID` 值。 產生的值會與權杖簽發者相同。 這項策略可讓多租用戶應用程式驗證指定租用戶的簽發者。

下列中繼資料說明了範例租用戶獨立 `EntityID` 項目。 請注意， `{tenant}` 是常值，而不是預留位置。

```
<EntityDescriptor
xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
ID="="_0e5bd9d0-49ef-4258-bc15-21ce143b61bd"
entityID="https://sts.windows.net/{tenant}/">
```

### <a name="token-signing-certificates"></a>權杖簽署憑證
當服務接收由 Azure AD 租用戶簽發的權杖時，權杖的簽章必須使用同盟中繼資料文件中發佈的簽署金鑰驗證。 同盟中繼資料包含租用戶用於權杖簽署之憑證的公開部分。 憑證原始位元組會出現在 `KeyDescriptor` 項目中。 當 `use` 屬性值為 `signing` 時，權杖簽署憑證僅能適用於簽署。

Azure AD 所發佈之同盟中繼資料文件可以有多組簽署金鑰 (例如，Azure AD 準備更新簽署憑證時)。 當同盟中繼資料文件包含多個憑證時，正在驗證權杖的服務應會支援該文件中的所有憑證。

下列中繼資料說明了具有簽署金鑰的範例 `KeyDescriptor` 項目。

```
<KeyDescriptor use="signing">
<KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
<X509Data>
<X509Certificate>
MIIDPjCCAiqgAwIBAgIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAMC0xKzApBgNVBAMTImFjY291bnRzLmFjY2Vzc2NvbnRyb2wud2luZG93cy5uZXQwHhcNMTIwNjA3MDcwMDAwWhcNMTQwNjA3MDcwMDAwWjAtMSswKQYDVQQDEyJhY2NvdW50cy5hY2Nlc3Njb250cm9sLndpbmRvd3MubmV0MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArCz8Sn3GGXmikH2MdTeGY1D711EORX/lVXpr+ecGgqfUWF8MPB07XkYuJ54DAuYT318+2XrzMjOtqkT94VkXmxv6dFGhG8YZ8vNMPd4tdj9c0lpvWQdqXtL1TlFRpD/P6UMEigfN0c9oWDg9U7Ilymgei0UXtf1gtcQbc5sSQU0S4vr9YJp2gLFIGK11Iqg4XSGdcI0QWLLkkC6cBukhVnd6BCYbLjTYy3fNs4DzNdemJlxGl8sLexFytBF6YApvSdus3nFXaMCtBGx16HzkK9ne3lobAwL2o79bP4imEGqg+ibvyNmbrwFGnQrBc1jTF9LyQX9q+louxVfHs6ZiVwIDAQABo2IwYDBeBgNVHQEEVzBVgBCxDDsLd8xkfOLKm4Q/SzjtoS8wLTErMCkGA1UEAxMiYWNjb3VudHMuYWNjZXNzY29udHJvbC53aW5kb3dzLm5ldIIQVWmXY/+9RqFA/OG9kFulHDAJBgUrDgMCHQUAA4IBAQAkJtxxm/ErgySlNk69+1odTMP8Oy6L0H17z7XGG3w4TqvTUSWaxD4hSFJ0e7mHLQLQD7oV/erACXwSZn2pMoZ89MBDjOMQA+e6QzGB7jmSzPTNmQgMLA8fWCfqPrz6zgH+1F1gNp8hJY57kfeVPBiyjuBmlTEBsBlzolY9dd/55qqfQk6cgSeCbHCy/RU/iep0+UsRMlSgPNNmqhj5gmN2AFVCN96zF694LwuPae5CeR2ZcVknexOWHYjFM0MgUSw0ubnGl0h9AJgGyhvNGcjQqu9vd1xkupFgaN+f7P3p3EVN5csBg5H94jEcQZT7EKeTiZ6bTrpDAnrr8tDCy8ng
</X509Certificate>
</X509Data>
</KeyInfo>
</KeyDescriptor>
  ```

`KeyDescriptor` 項目會出現在同盟中繼資料文件中的兩個位置：WS 同盟專屬區段和 SAML 專屬區段。 發佈在這兩個區段的憑證將會是相同的。

在 WS 同盟專屬區段中，WS 同盟中繼資料讀取器會從 `RoleDescriptor` 項目中讀取類型為 `SecurityTokenServiceType` 的憑證。

下列中繼資料說明範例 `RoleDescriptor` 項目。

```
<RoleDescriptor xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:fed="http://docs.oasis-open.org/wsfed/federation/200706" xsi:type="fed:SecurityTokenServiceType"protocolSupportEnumeration="http://docs.oasis-open.org/wsfed/federation/200706">
```

在 SAML 專屬區段中，WS 同盟中繼資料讀取器會讀取來自 `IDPSSODescriptor` 項目的憑證。

下列中繼資料說明範例 `IDPSSODescriptor` 項目。

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
```
租用戶專屬與租用戶獨立憑證格式並沒有差異。

### <a name="ws-federation-endpoint-url"></a>WS 同盟端點 URL
同盟中繼資料包含 Azure AD 用於 WS-同盟通訊協定中之單一登入和單一登出的 URL。 此端點會出現在 `PassiveRequestorEndpoint` 項目中。

下列中繼資料說明了租用戶專屬端點的範例 `PassiveRequestorEndpoint` 項目。

```
<fed:PassiveRequestorEndpoint>
<EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
<Address>
https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db45/wsfed
</Address>
</EndpointReference>
</fed:PassiveRequestorEndpoint>
```
若為租用戶獨立端點，WS 同盟 URL 會出現在 WS 同盟端點中，如下列範例所示。

```
<fed:PassiveRequestorEndpoint>
<EndpointReference xmlns="http://www.w3.org/2005/08/addressing">
<Address>
https://login.microsoftonline.com/common/wsfed
</Address>
</EndpointReference>
</fed:PassiveRequestorEndpoint>
```

### <a name="saml-protocol-endpoint-url"></a>SAML 通訊協定端點 URL
同盟中繼資料包含 Azure AD 用於 SAML 2.0 通訊協定中之單一登入和單一登出的 URL。 這些端點會出現在 `IDPSSODescriptor` 項目中。

登入和登出 URL 會出現在 `SingleSignOnService` 和 `SingleLogoutService` 項目中。

下列中繼資料說明了租用戶專屬端點的範例 `PassiveResistorEndpoint` 。

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
…
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.onmicrosoft.com/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/contoso.onmicrosoft.com /saml2" />
  </IDPSSODescriptor>
```

同樣地，適用於一般 SAML 2.0 通訊協定端點的端點也會發佈在租用戶獨立同盟中繼資料中，如下列範例所示。

```
<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
…
    <SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
    <SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://login.microsoftonline.com/common/saml2" />
  </IDPSSODescriptor>
```
