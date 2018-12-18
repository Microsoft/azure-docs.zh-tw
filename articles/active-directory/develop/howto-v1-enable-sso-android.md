---
title: 如何使用 ADAL 在 Android 上啟用跨應用程式的 SSO | Microsoft Docs
description: 如何使用 ADAL SDK 的功能來啟用跨應用程式的單一登入。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 40710225-05ab-40a3-9aec-8b4e96b6b5e7
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: android
ms.devlang: java
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: dadobali
ms.custom: aaddev
ms.openlocfilehash: c7d1665eece966053f52e118bb0d6f7f5ec4cba6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46957858"
---
# <a name="how-to-enable-cross-app-sso-on-android-using-adal"></a>操作說明：使用 ADAL 在 Android 上啟用跨應用程式的 SSO

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

單一登入 (SSO) 可讓使用者只輸入認證一次，並讓這些認證跨應用程式及跨其他應用程式可能會使用的平台自動處理 (例如 Microsoft 帳戶或來自 Microsoft 365 的公司帳戶)，無論發行者為何。

Microsoft 的身分識別平台搭配 SDK，能讓您在整個裝置中，更輕鬆地在您自己的應用程式套件內，或使用訊息代理程式功能和 Authenticator 應用程式來啟用單一登入。

在本操作說明中，您會了解如何在應用程式中設定 SDK 以提供 SSO 給客戶。

## <a name="prerequisites"></a>必要條件

此操作說明會假設您知道如何：

- 使用 Azure Active Directory (Azure AD) 的傳統入口網站佈建應用程式。 如需詳細資訊，請參閱[使用 Azure AD v1.0 端點註冊應用程式](quickstart-v1-add-azure-ad-app.md)
- 整合應用程式與 [Azure AD Android SDK](https://github.com/AzureAD/azure-activedirectory-library-for-android)。

## <a name="single-sign-on-concepts"></a>單一登入概念

### <a name="identity-brokers"></a>身分識別訊息代理程式

Microsoft 為每個行動平台提供應用程式，允許跨不同廠商的應用程式橋接認證，並需要單一安全地方來驗證認證的增強功能。 我們稱之為**訊息代理程式**。

在 iOS 和 Android 上，會透過可下載的應用程式提供這些訊息代理程式，客戶可以單獨加以安裝，或為員工管理部分或所有裝置的公司推送至裝置。 訊息代理程式支援只管理某些應用程式或整個裝置的安全性，取決於 IT 管理員設定。 在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。

#### <a name="broker-assisted-login"></a>訊息代理程式協助登入

訊息代理程式協助登入是在訊息代理程式應用程式中發生的登入體驗，並且使用訊息代理程式的儲存體和安全性，跨套用身分識別平台的裝置上所有應用程式共用認證。 這表示您的應用程式將會依賴訊息代理程式才能讓使用者登入。 在 iOS 和 Android 上，會透過可下載的應用程式提供這些訊息代理程式，客戶可單獨加以安裝，或可由為使用者管理裝置的公司推送至裝置。 此類型應用程式的範例為 iOS 上的 Microsoft Authenticator 應用程式。 在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。
體驗會依平台而有所不同，如果未正確管理，有時可能會干擾使用者。 如果您已安裝 Facebook 應用程式，並在另一個應用程式中使用 Facebook Connect，您可能最熟悉這種模式。 身分識別平台會使用相同的模式。

在 Android 上，帳戶選擇器會顯在比較不干擾使用者的應用程式頂端。

#### <a name="how-the-broker-gets-invoked"></a>如何叫用訊息代理程式

如果在裝置上安裝相容的訊息代理程式，例如 Microsoft Authenticator 應用程式，當使用者指出他們想要使用身分識別平台的任何帳戶登入時，身分識別 SDK 會自動為您叫用訊息代理程式。

#### <a name="how-microsoft-ensures-the-application-is-valid"></a>Microsoft 如何確保應用程式的有效性

需要確保應用程式的身分識別會呼叫訊息代理程式，對於以訊息代理程式輔助的登入所提供的安全性而言相當重要。 IOS 和 Android 都不會強制執行僅對特定應用程式有效的唯一識別碼，因此，惡意應用程式可能會「詐騙」合法應用程式的識別碼，並接收適用於合法應用程式的權杖。 若要確定 Microsoft 一律會在執行階段與正確的應用程式進行通訊，我們會要求開發人員在向 Microsoft 註冊應用程式時提供自訂的 redirectURI。 **以下將詳細討論開發人員應該如何製作此重新導向 URI。** 此自訂的 redirectURI 包含應用程式的憑證指紋，並透過 Google Play Store 確保它對應用程式而言是唯一的。 當應用程式呼叫訊息代理程式時，訊息代理程式會要求 Android 作業系統搭配呼叫訊息代理程式的憑證指紋來提供它。 訊息代理程式會在對身分識別系統的呼叫中，將此憑證指紋提供給 Microsoft。 如果應用程式的憑證指紋不符合開發人員在註冊期間提供給我們的憑證指紋，就會拒絕存取應用程式所要求的資源權杖。 這項檢查可確保只有開發人員所註冊的應用程式能夠接收權杖。

訊息代理程式的 SSO 登入具備下列優點︰

* 無論廠商為何，使用者都可以跨所有應用程式體驗 SSO。
* 您的應用程式可以使用更進階的商務功能 (例如條件式存取) 並支援 Intune 案例。
* 您的應用程式可針對商務使用者支援以憑證為基礎的驗證。
* 由訊息代理程式利用額外的安全性演算法和加密來驗證應用程式和使用者身分識別，可享有更安全的登入體驗。

以下說明 SDK 如何使用訊息代理程式應用程式來啟用單一登入：

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
|  ADAL SDK  | |  ADAL SDK  |   |  ADAL SDK   |
+-----+------+-+-----+------+-  +-------+-----+
      |              |                  |
      |       +------v------+           |
      |       |             |           |
      |       | Microsoft   |           |
      +-------> Broker      |^----------+
              | Application
              |             |
              +-------------+
              |             |
              |   Broker    |
              |   Storage   |
              |             |
              +-------------+

```

### <a name="turning-on-sso-for-broker-assisted-sso"></a>開啟訊息代理程式協助 SSO 的 SSO

應用程式使用任何已安裝在裝置上之訊息代理程式的功能預設為關閉。 若要搭配使用應用程式與訊息代理程式，您必須執行一些額外的設定，並將一些程式碼新增至您的應用程式。

要遵循的步驟如下：

1. 在應用程式程式碼呼叫 MS SDK 時啟用訊息代理程式模式
2. 建立新的重新導向 URI，並將其提供給應用程式與應用程式註冊
3. 在 Android 資訊清單中設定正確的權限

#### <a name="step-1-enable-broker-mode-in-your-application"></a>步驟 1︰在應用程式中啟用訊息代理程式模式

當您建立「設定」或驗證執行個體的初始設定時，已開啟應用程式使用訊息代理程式的功能。 若要在您的應用程式中執行此動作：

```
AuthenticationSettings.Instance.setUseBroker(true);
```

#### <a name="step-2-establish-a-new-redirect-uri-with-your-url-scheme"></a>步驟 2︰利用 URL 配置建立新的重新導向 URI

為了確保傳回的認證權杖一律會由正確的應用程式所接收，必須要確定 Android 作業系統可以確認回呼應用程式的方式。 Android 作業系統會使用 Google Play 商店中的憑證雜湊。 惡意應用程式無法假冒此憑證雜湊。 搭配訊息代理程式應用程式的 URI，Microsoft 可確保權杖會傳回給正確的應用程式。 唯一的重新導向 URI 必須註冊於應用程式上。

您的重新導向 URI 必須是適當的格式︰

`msauth://packagename/Base64UrlencodedSignature`

例如：msauth://com.example.userapp/IcB5PxIyvbLkbFVtBI%2FitkW%2Fejk%3D

您可以使用 [Azure 入口網站](https://portal.azure.com/)在應用程式註冊中註冊此重新導向 URI。 如需 Azure AD 應用程式註冊的詳細資訊，請參閱 [與 Azure Active Directory 整合](active-directory-how-to-integrate.md)。

#### <a name="step-3-set-up-the-correct-permissions-in-your-application"></a>步驟 3：在您的應用程式中設定正確的權限

Android 訊息代理程式應用程式會使用 Android 作業系統的 Accounts Manager 功能來管理所有應用程式的認證。 若要在 Android 中使用此訊息代理程式，您的應用程式資訊清單必須有使用 AccountManager 帳戶的權限。 在[此處適用於帳戶管理員的 Google 文件](http://developer.android.com/reference/android/accounts/AccountManager.html)中，有這些權限的詳細討論

特別是以下這些權限︰

```
GET_ACCOUNTS
USE_CREDENTIALS
MANAGE_ACCOUNTS
```

### <a name="youve-configured-sso"></a>您已設定 SSO！

現在身分識別 SDK 會自動跨應用程式共用認證，並在訊息代理程式出現在其裝置上時叫用它。

## <a name="next-steps"></a>後續步驟

* 深入了解[單一登入 SAML 通訊協定](single-sign-on-saml-protocol.md)