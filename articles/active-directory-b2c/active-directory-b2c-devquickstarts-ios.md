---
title: 在 Azure Active Directory B2C 中的 iOS 應用程式使用 AppAuth | Microsoft Docs
description: 本文說明如何建立 iOS 應用程式，以使用 AppAuth 和 Azure Active Directory B2C 來管理使用者身分識別和驗證使用者。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/07/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 5f95b71497b59eafff09d4add2b4bb1c20656592
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43339353"
---
# <a name="azure-ad-b2c-sign-in-using-an-ios-application"></a>Azure AD B2C︰使用 iOS 應用程式登入

Microsoft 身分識別平台會使用開放式標準，例如 OAuth2 和 OpenID Connect。 使用開放式標準通訊協定讓開發人員在選取程式庫來與我們的服務整合時，有更多的選擇。 我們提供本逐步解說和其他類似的逐步解說，協助開發人員撰寫應用程式來連線至 Microsoft 身分識別平台。 大部分實作 [RFC6749 OAuth2 規格](https://tools.ietf.org/html/rfc6749)的程式庫都能連線至 Microsoft 身分識別平台。

> [!WARNING]
> Microsoft 並不提供第三方程式庫的修正程式，也尚未審查這些程式庫。 此範例使用已藉由 Azure AD B2C 在基本案例中進行過相容性測試的第三方程式庫，稱為 AppAuth。 問題和功能要求應導向到程式庫的開放原始碼專案。 如需詳細資訊，請參閱 [本篇文章](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-libraries)。
>
>

如果您是 OAuth2 或 OpenID Connect 新手，此範例組態可能不太適合您。 建議您查看 [我們在此記載的通訊協定簡短概觀](active-directory-b2c-reference-protocols.md)。

## <a name="get-an-azure-ad-b2c-directory"></a>取得 Azure AD B2C 目錄
您必須先建立目錄或租用戶，才可使用 Azure AD B2C。 目錄就是您所有使用者、應用程式、群組等項目的容器。 如果您還沒有此資源，請先 [建立 B2C 目錄](active-directory-b2c-get-started.md) 再繼續進行。

## <a name="create-an-application"></a>建立應用程式
接著，您必須在 B2C 目錄中建立應用程式。 應用程式註冊會提供必要資訊給 Azure AD，讓它與應用程式安全地通訊。 如果要建立行動應用程式，請遵循[這些指示](active-directory-b2c-app-registration.md)。 請務必：

* 在應用程式中加入**原生用戶端**。
* 複製指派給您的應用程式的 **應用程式識別碼** 。 您稍後需要此 GUID。
* 使用自訂配置設定**重新導向 URI** (例如，com.onmicrosoft.fabrikamb2c.exampleapp://oauth/redirect)。 您稍後需要此 URI。

## <a name="create-your-policies"></a>建立您的原則
在 Azure AD B2C 中，每個使用者體驗皆是由某個 [原則](active-directory-b2c-reference-policies.md)所定義。 此應用程式包含一個身分識別體驗：合併登入和註冊。 建立此原則，如[原則參考文章](active-directory-b2c-reference-policies.md#create-a-sign-up-policy)所述。 建立此原則時，請務必：

* 在 [註冊屬性] 下方，選取 [顯示名稱] 屬性。  您也可以選取其他屬性。
* 在 [應用程式宣告] 下方，選取 [顯示名稱] 和 [使用者的物件識別碼] 宣告。 您也可以選取其他宣告。
* 建立每個原則之後，請複製原則的 **名稱** 。 當您儲存原則時，原則名稱前面會加上 `b2c_1_`。  您稍後需要用到此原則名稱。

[!INCLUDE [active-directory-b2c-devquickstarts-policy](../../includes/active-directory-b2c-devquickstarts-policy.md)]

建立您的原則後，就可以開始建置您的應用程式。

## <a name="download-the-sample-code"></a>下載範例程式碼
我們[在 GitHub 上](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c)提供一個使用 AppAuth 和 Azure AD B2C 的操作範例。 您可以下載程式碼並執行。 若要使用自己的 Azure AD B2C 租用戶，請依照 [README.md](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c/blob/master/README.md) 中的指示。

此範例由 [GitHub 上的 iOS AppAuth 專案](https://github.com/openid/AppAuth-iOS)依照 README 指示建立。 如需此範例和程式庫運作方式的詳細資訊，請參考 GitHub 上的 AppAuth README。

## <a name="modifying-your-app-to-use-azure-ad-b2c-with-appauth"></a>修改您的應用程式來使用 Azure AD B2C 和 AppAuth

> [!NOTE]
> AppAuth 支援 iOS 7 及更新版本。  不過，若要在 Google 上支援社交登入，需要有 SFSafariViewController，而這需要 iOS 9 或更新版本。
>

### <a name="configuration"></a>組態

您可以指定授權端點和權杖端點 URI，以設定與 Azure AD B2C 通訊。  若要產生這些 URI，您需要下列資訊︰
* 租用戶識別碼 (例如，contoso.onmicrosoft.com)
* 原則名稱 (例如，B2C\_1\_SignUpIn)

取代下列 URL 中的 Tenant\_ID 和Policy\_Name，即可產生權杖端點 URI︰

```objc
static NSString *const tokenEndpoint = @"https://<Tenant_name>.b2clogin.com/te/<Tenant_ID>/<Policy_Name>/oauth2/v2.0/token";
```

取代下列 URL 中的 Tenant\_ID 和Policy\_Name，即可產生授權端點 URI︰

```objc
static NSString *const authorizationEndpoint = @"https://<Tenant_name>.b2clogin.com/te/<Tenant_ID>/<Policy_Name>/oauth2/v2.0/authorize";
```

執行下列程式碼來建立 AuthorizationServiceConfiguration 物件︰

```objc
OIDServiceConfiguration *configuration = 
    [[OIDServiceConfiguration alloc] initWithAuthorizationEndpoint:authorizationEndpoint tokenEndpoint:tokenEndpoint];
// now we are ready to perform the auth request...
```

### <a name="authorizing"></a>授權

設定或擷取授權服務組態之後，就可以建構授權要求。 若要建立要求，您需要下列資訊︰  
* 用戶端識別碼 (例如，00000000-0000-0000-0000-000000000000)
* 使用自訂配置的重新導向 URI (例如，com.onmicrosoft.fabrikamb2c.exampleapp://oauth/redirect)

這兩個項目應該已在您[註冊應用程式](#create-an-application)時儲存。

```objc
OIDAuthorizationRequest *request = 
    [[OIDAuthorizationRequest alloc] initWithConfiguration:configuration
                                                  clientId:kClientId
                                                    scopes:@[OIDScopeOpenID, OIDScopeProfile]
                                               redirectURL:[NSURL URLWithString:kRedirectUri]
                                              responseType:OIDResponseTypeCode
                                      additionalParameters:nil];

AppDelegate *appDelegate = (AppDelegate *)[UIApplication sharedApplication].delegate;
appDelegate.currentAuthorizationFlow = 
    [OIDAuthState authStateByPresentingAuthorizationRequest:request
                                   presentingViewController:self
                                                   callback:^(OIDAuthState *_Nullable authState, NSError *_Nullable error) {
        if (authState) {
            NSLog(@"Got authorization tokens. Access token: %@", authState.lastTokenResponse.accessToken);
            [self setAuthState:authState];
        } else {
            NSLog(@"Authorization error: %@", [error localizedDescription]);
            [self setAuthState:nil];
        }
    }];
```

若要設定應用程式以處理使用自訂配置的 URI 重新導向，您需要更新 Info.pList 中的 [URL 配置] 清單︰
* 開啟 Info.pList。
* 將滑鼠停留在像是 [套件組合 OS 類型代碼] 的資料列，然後按一下 \+ 符號。
* 重新命名新的資料列 [URL 類型]。
* 按一下 [URL 類型] 左邊的箭頭以開啟樹狀結構。
* 按一下 [項目 0] 左邊的箭頭以開啟樹狀結構。
* 將 [項目 0] 下方的第一個項目重新命名為 [URL 配置]。
* 按一下 [URL 配置] 左邊的箭頭以開啟樹狀結構。
* 在 [值] 資料行中，[URL 配置] 下方的 [項目 0] 左邊有一個空白欄位。  將此值設為應用程式的唯一配置。  建立 OIDAuthorizationRequest 物件時，此值必須符合 redirectURL 中使用的配置。  在範例中，使用配置 'com.onmicrosoft.fabrikamb2c.exampleapp'。

有關如何完成此程序的其餘部分，請參閱 [AppAuth 指南](https://openid.github.io/AppAuth-iOS/)。 如果您需要快速開始使用一個可操作的應用程式，請參閱[範例](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c) \(英文\)。 請依照 [README.md](https://github.com/Azure-Samples/active-directory-ios-native-appauth-b2c/blob/master/README.md) 中的步驟，輸入自己的 Azure AD B2C 組態。

我們歡迎意見反應和建議！ 如果您對本文有任何問題，或有改進此內容的建議，非常歡迎您在頁面底部提供意見反應。 對於功能要求，請將它們新增到 [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160596-b2c)。
