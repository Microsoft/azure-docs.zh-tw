---
title: 在 Azure Active Directory B2C 中使用 Android 應用程式取得權杖 | Microsoft Docs
description: 本文將說明如何建立 Android 應用程式，以使用 AppAuth 和 Azure Active Directory B2C 來管理使用者身分識別和驗證使用者。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/06/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 716cf9e47cd71d003513066d390f9dccb5c83dcb
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43344121"
---
# <a name="azure-ad-b2c-sign-in-using-an-android-application"></a>Azure AD B2C︰使用 Android 應用程式登入

Microsoft 身分識別平台會使用開放式標準，例如 OAuth2 和 OpenID Connect。 這些標準可讓您利用想要使用的任何程式庫，與 Azure Active Directory B2C 整合。 為了協助您使用其他程式庫，您可以使用這個逐步解說，它示範如何設定第三方程式庫以連線至 Microsoft 身分識別平台。 大部分實作 [RFC6749 OAuth2 規格](https://tools.ietf.org/html/rfc6749)的程式庫都能連線到 Microsoft 身分識別平台。

> [!WARNING]
> Microsoft 並不提供第三方程式庫的修正程式，也尚未審查這些程式庫。 此範例使用已藉由 Azure AD B2C 在基本案例中進行過相容性測試的第三方程式庫，稱為 AppAuth。 問題和功能要求應導向到程式庫的開放原始碼專案。 如需詳細資訊，請參閱[這篇文章](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-libraries)。  
>
>

如果您是 OAuth2 或 OpenID Connect 新手，此範例組態可能諸多不太適合您。 建議您查看 [我們在此記載的通訊協定簡短概觀](active-directory-b2c-reference-protocols.md)。

## <a name="get-an-azure-ad-b2c-directory"></a>取得 Azure AD B2C 目錄

您必須先建立目錄或租用戶，才可使用 Azure AD B2C。 目錄為所有使用者、應用程式、群組等項目的容器。 如果您還沒有此資源，請先 [建立 B2C 目錄](active-directory-b2c-get-started.md) 再繼續進行。

## <a name="create-an-application"></a>建立應用程式

接著，您必須在 B2C 目錄中建立應用程式。 這會提供必要資訊給 Azure AD，讓它與應用程式安全地通訊。 如果要建立行動應用程式，請遵循[這些指示](active-directory-b2c-app-registration.md)。 請務必：

* 在應用程式中加入**原生用戶端**。
* 複製指派給您的應用程式的 **應用程式識別碼** 。 稍後您將會需要此資訊。
* 設定原生用戶端**重新導向 URI** (例如，com.onmicrosoft.fabrikamb2c.exampleapp://oauth/redirect)。 稍後您也會需要此資訊。

## <a name="create-your-policies"></a>建立您的原則

在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 此應用程式包含一個身分識別體驗：合併登入和註冊。 您必須建立此原則，如[原則參考文章](active-directory-b2c-reference-policies.md#create-a-sign-up-policy)所述。 建立此原則時，請務必：

* 在原則中選擇 [顯示名稱] 做為註冊屬性。
* 在每個原則中，選擇 [顯示名稱] 和 [物件識別碼] 應用程式宣告。 您也可以選擇其他宣告。
* 在您建立每個原則之後，請複製原則的 [名稱]  。 其前置詞應該為 `b2c_1_`。  您稍後需要用到此原則名稱。

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

建立您的原則後，就可以開始建置您的應用程式。

## <a name="download-the-sample-code"></a>下載範例程式碼

我們[在 GitHub 上](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c)提供一個使用 AppAuth 和 Azure AD B2C 的操作範例。 您可以下載程式碼並執行。 您可以依照 [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md) 中的指示，使用您自己的 Azure AD B2C 組態，快速開始使用您自己的應用程式。

這個範例修改自 [AppAuth](https://openid.github.io/AppAuth-Android/) 提供的範例。 請瀏覽其頁面，以深入了解 AppAuth 和相關功能。

## <a name="modifying-your-app-to-use-azure-ad-b2c-with-appauth"></a>修改您的應用程式來使用 Azure AD B2C 和 AppAuth

> [!NOTE]
> AppAuth 支援 Android API 16 (Jellybean) 和更新版本。 我們建議使用 API 23 和更新版本。
>

### <a name="configuration"></a>組態

您可以指定探索 URI，或指定授權端點和權杖端點 URI，以設定與 Azure AD B2C 通訊。 不論何者，您都需要下列資訊：

* 租用戶識別碼 (例如，contoso.onmicrosoft.com)
* 原則名稱 (例如，B2C\_1\_SignUpIn)

如果您選擇自動探索授權和權杖端點 URI，您必須從探索 URI 擷取資訊。 取代下列 URL 中的 Tenant\_ID 和Policy\_Name，即可產生探索 URI︰

```java
String mDiscoveryURI = "https://<Tenant_name>.b2clogin.com/<Tenant_ID>/v2.0/.well-known/openid-configuration?p=<Policy_Name>";
```

然後，您可以取得授權和權杖端點 URI，並執行下列命令來建立 AuthorizationServiceConfiguration 物件︰

```java
final Uri issuerUri = Uri.parse(mDiscoveryURI);
AuthorizationServiceConfiguration config;

AuthorizationServiceConfiguration.fetchFromIssuer(
    issuerUri,
    new RetrieveConfigurationCallback() {
      @Override public void onFetchConfigurationCompleted(
          @Nullable AuthorizationServiceConfiguration serviceConfiguration,
          @Nullable AuthorizationException ex) {
        if (ex != null) {
            Log.w(TAG, "Failed to retrieve configuration for " + issuerUri, ex);
        } else {
            // service configuration retrieved, proceed to authorization...
        }
      }
  });
```

除了使用探索來取得授權和權杖端點 URI，您也可以取代下列 URL 中的 Tenant\_ID 和 Policy\_Name，以明確指定它們︰

```java
String mAuthEndpoint = "https://<Tenant_name>.b2clogin.com/<Tenant_ID>/oauth2/v2.0/authorize?p=<Policy_Name>";

String mTokenEndpoint = "https://<Tenant_name>.b2clogin.com/<Tenant_ID>/oauth2/v2.0/token?p=<Policy_Name>";
```

執行下列程式碼來建立 AuthorizationServiceConfiguration 物件︰

```java
AuthorizationServiceConfiguration config =
        new AuthorizationServiceConfiguration(name, mAuthEndpoint, mTokenEndpoint);

// perform the auth request...
```

### <a name="authorizing"></a>授權

設定或擷取授權服務組態之後，就可以建構授權要求。 若要建立要求，您需要下列資訊︰

* 用戶端識別碼 (例如，00000000-0000-0000-0000-000000000000)
* 使用自訂配置的重新導向 URI (例如，com.onmicrosoft.fabrikamb2c.exampleapp://oauthredirect)

這兩個項目應該已在您[註冊應用程式](#create-an-application)時儲存。

```java
AuthorizationRequest req = new AuthorizationRequest.Builder(
    config,
    clientId,
    ResponseTypeValues.CODE,
    redirectUri)
    .build();
```

有關如何完成此程序的其餘部分，請參閱 [AppAuth 指南](https://openid.github.io/AppAuth-Android/)。 如果您需要快速開始使用一個可操作的應用程式，請參閱[我們的範例](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c)。 請依照 [README.md](https://github.com/Azure-Samples/active-directory-android-native-appauth-b2c/blob/master/README.md) 中的步驟，輸入自己的 Azure AD B2C 組態。

我們歡迎意見反應和建議！ 如果您對本文有任何問題，或有改進此內容的建議，非常歡迎您在頁面底部提供意見反應。 對於功能要求，請將它們新增到 [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160596-b2c)。

