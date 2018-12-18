---
title: 如何使用 ADAL 在 iOS 上啟用跨應用程式的 SSO | Microsoft Docs
description: 如何使用 ADAL SDK 的功能來啟用跨應用程式的單一登入。
services: active-directory
author: CelesteDG
manager: mtillman
ms.assetid: d042d6da-7503-4e20-bb55-06917de01fcd
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: e9598cb464360e35a86b6fe35d8c965a5e7fb51d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963027"
---
# <a name="how-to-enable-cross-app-sso-on-ios-using-adal"></a>操作說明：使用 ADAL 在 iOS 上啟用跨應用程式的 SSO

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

單一登入 (SSO) 可讓使用者只輸入認證一次，就讓這些認證跨應用程式及跨其他應用程式可能會使用的平台 (例如 Microsoft 帳戶或來自 Microsoft 365 的公司帳戶) 自動運作，無論發行者是誰。

Microsoft 的身分識別平台搭配 SDK，能讓您在整個裝置中，更輕鬆地在您自己的應用程式套件內，或使用訊息代理程式功能和 Authenticator 應用程式來啟用 SSO。

在本操作說明中，您會了解如何在應用程式中設定 SDK 以提供 SSO 給客戶。

此操作說明適用於：

* Azure Active Directory (Azure Active Directory)
* Azure Active Directory B2C
* Azure Active Directory B2B
* Azure Active Directory 條件式存取

## <a name="prerequisites"></a>必要條件

此操作說明假設您知道如何：

* 使用 Azure AD 的傳統入口網站佈建應用程式。 如需詳細資訊，請參閱[向 Azure AD v1.0 端點註冊應用程式](quickstart-v1-add-azure-ad-app.md)
* 整合應用程式與 [Azure AD iOS SDK](https://github.com/AzureAD/azure-activedirectory-library-for-objc)。

## <a name="single-sign-on-concepts"></a>單一登入概念

### <a name="identity-brokers"></a>身分識別訊息代理程式

Microsoft 會提供適用於各種行動平台的應用程式，既能夠橋接不同廠商應用程式的認證，又能夠提供需要單一安全位置來驗證認證的增強功能。 這些應用程式稱為**訊息代理程式**。

在 iOS 和 Android 上，會透過可下載的應用程式提供這些訊息代理程式，客戶可以單獨加以安裝，或由為員工管理部分或所有裝置的公司推送至裝置。 訊息代理程式支援只管理某些應用程式或整個裝置的安全性，取決於 IT 管理員設定。 在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。

### <a name="patterns-for-logging-in-on-mobile-devices"></a>在行動裝置上登入的模式

在裝置上存取認證的動作遵循兩個基本模式︰

* 非訊息代理程式協助登入
* 訊息代理程式協助登入

#### <a name="non-broker-assisted-logins"></a>非訊息代理程式協助登入

非訊息代理程式協助登入是在應用程式內發生的登入體驗，並且使用該應用程式之裝置上的本機儲存體。 儲存體可能會跨應用程式共用，但認證會緊密繫結至使用該認證的應用程式或應用程式套件。 當您在應用程式本身內部輸入使用者名稱和密碼時，最可能已在許多行動應用程式中經歷此體驗。

這些登入具備下列優點︰

* 使用者體驗完全存在於應用程式內部。
* 認證可以跨由相同憑證登入的應用程式共用，提供單一登入體驗給您的應用程式套件。
* 在登入前後，會提供登入體驗的控制項給應用程式。

這些登入具備下列缺點︰

* 使用者無法跨所有使用 Microsoft 身分識別的應用程式體驗單一登入，只會跨應用程式已設定的那些 Microsoft 身分識別。
* 您的應用程式無法搭配更進階的商務功能使用 (例如條件式存取)，或使用產品的 InTune 套件。
* 您的應用程式無法針對商務使用者支援以憑證為基礎的驗證。

以下說明 SDK 如何使用應用程式的共用儲存體來啟用 SSO：

```
+------------+ +------------+  +-------------+
|            | |            |  |             |
|   App 1    | |   App 2    |  |   App 3     |
|            | |            |  |             |
|            | |            |  |             |
+------------+ +------------+  +-------------+
| ADAL SDK  |  |  ADAL SDK  |  |  ADAL SDK   |
+------------+-+------------+--+-------------+
|                                            |
|            App Shared Storage              |
+--------------------------------------------+
```

#### <a name="broker-assisted-logins"></a>訊息代理程式協助登入

訊息代理程式協助登入是在訊息代理程式應用程式中發生的登入體驗，並且使用訊息代理程式的儲存體和安全性，跨套用身分識別平台的裝置上所有應用程式共用認證。 這表示您的應用程式依賴訊息代理程式來讓使用者登入。 在 iOS 和 Android 上，會透過可下載的應用程式提供這些訊息代理程式，客戶可單獨加以安裝，或可由為使用者管理裝置的公司推送至裝置。 此類型應用程式的範例為 iOS 上的 Microsoft Authenticator 應用程式。 在 Windows 中，內建於作業系統的帳戶選擇器會提供此功能，在技術上稱為 Web 驗證訊息代理程式。

體驗會依平台而有所不同，如果未正確管理，有時可能會干擾使用者。 如果您已安裝 Facebook 應用程式，並在另一個應用程式中使用 Facebook Connect，您可能最熟悉這種模式。 身分識別平台會使用相同的模式。

對於 iOS 而言，這會前往「轉換」動畫，您的應用程式已在其中傳送到背景工作，而 Microsoft Authenticator 應用程式則來到前景讓使用者選取他們想要登入的帳戶。

對於 Android 和 Windows 而言，帳戶選擇器會顯在較不干擾使用者的應用程式頂端。

#### <a name="how-the-broker-gets-invoked"></a>如何叫用訊息代理程式

如果在裝置上安裝相容的訊息代理程式，例如 Microsoft Authenticator 應用程式，當使用者指出他們想要使用身分識別平台的任何帳戶登入時，SDK 會自動為您叫用訊息代理程式。 這個帳戶可以是個人的 Microsoft 帳戶、工作或學校帳戶，或您提供的帳戶，以及在 Azure 中使用 B2C 和 B2B 產品的主機。

#### <a name="how-we-ensure-the-application-is-valid"></a>我們如何確保應用程式有效

需要確保應用程式的身分識別會呼叫訊息代理程式，對於我們在訊息代理程式協助登入中所提供的安全性而言相當重要。 IOS 和 Android 都不會強制執行僅對特定應用程式有效的唯一識別碼，因此，惡意應用程式可能會「詐騙」合法應用程式的識別碼，並接收適用於合法應用程式的權杖。 若要確定我們一律會在執行階段與正確的應用程式進行通訊，我們會要求開發人員在向 Microsoft 註冊應用程式時提供自訂的 redirectURI。 以下將詳細討論開發人員應該如何製作此重新導向 URI。 此自訂的 redirectURI 包含應用程式的搭售識別碼，並透過 Apple App Store 確保它對應用程式而言是唯一的。 當應用程式呼叫訊息代理程式時，訊息代理程式會要求 iOS 作業系統搭配呼叫訊息代理程式的配套識別碼來提供它。 訊息代理程式會在對我們身分識別系統的呼叫中將此配套識別碼提供給 Microsoft。 如果應用程式的配套識別碼不符合開發人員在註冊期間提供給我們的配套識別碼，我們將會拒絕存取應用程式所要求之資源的權杖。 這項檢查可確保只有開發人員所註冊的應用程式能夠接收權杖。

**無論 SDK 呼叫訊息代理程式或使用非訊息代理程式協助流程，開發人員都有這個選項。** 不過，如果開發人員選擇不使用訊息代理程式協助流程，他們會失去使用單一登入認證的優點，使用者可能已在裝置上新增這些認證，並防止 Microsoft 提供給客戶的商務功能使用其應用程式，例如條件式存取、Intune 管理功能和以憑證為基礎的驗證。

這些登入具備下列優點︰

* 無論廠商為何，使用者都可以跨所有應用程式體驗 SSO。
* 您的應用程式可以使用更進階的商務功能 (例如條件式存取)，或使用產品的 InTune 套件。
* 您的應用程式可針對商務使用者支援以憑證為基礎的驗證。
* 由訊息代理程式利用額外的安全性演算法和加密來驗證應用程式和使用者身分識別，可享有更多安全的登入體驗。

這些登入具備下列缺點︰

* 在 iOS 中，使用者可在選擇認證時轉換出您的應用程式體驗。
* 喪失在應用程式內為客戶管理登入體驗的功能。

以下說明 SDK 如何使用訊息代理程式應用程式來啟用單一登入：

```
+------------+ +------------+   +-------------+
|            | |            |   |             |
|   App 1    | |   App 2    |   |   Someone   |
|            | |            |   |    Else's   |
|            | |            |   |     App     |
+------------+ +------------+   +-------------+
| Azure SDK  | | Azure SDK  |   | Azure SDK   |
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

## <a name="enabling-cross-app-sso-using-adal"></a>使用 ADAL 啟用跨應用程式的 SSO

我們會在此處使用 ADAL iOS SDK 執行以下動作：

* 開啟應用程式套件的非訊息代理程式協助 SSO
* 開啟訊息代理程式協助 SSO 的支援

### <a name="turning-on-sso-for-non-broker-assisted-sso"></a>開啟非訊息代理程式協助 SSO 的 SSO

對於跨應用程式的非訊息代理程式協助單一登入，SDK 會為您管理單一登入的大部分複雜內容。 這包括在快取中尋找適當的使用者，以及維護登入使用者的清單以便您查詢。

若要跨您擁有的應用程式啟用 SSO，您需要執行下列動作︰

1. 請確定您所有的應用程式使用相同的用戶端識別碼或應用程式識別碼。
2. 請確定您所有的應用程式共用來自 Apple 的相同簽署憑證，以便您可以共用金鑰鏈。
3. 要求每個應用程式的相同金鑰鏈權利。
4. 告知 SDK 您想要我們使用的共用金鑰鏈。

#### <a name="using-the-same-client-id--application-id-for-all-the-applications-in-your-suite-of-apps"></a>將相同的用戶端識別碼 / 應用程式識別碼使用於應用程式套件中的所有應用程式

為了讓身分識別平台知道它可以跨應用程式共用權杖，您的每個應用程式必須共用相同的用戶端識別碼或應用程式識別碼。 這是您在入口網站中註冊第一個應用程式時提供給您的唯一識別碼。

如果 Microsoft 識別服務使用相同的應用程式識別碼，重新導向 URI 可讓您識別此服務的不同應用程式。 每個應用程式可以在上架的入口網站中註冊多個重新導向 URI。 組件中的每個應用程式將會有不同的重新導向 URI。 其外觀的範例如下：

App1 重新導向 URI： `x-msauth-mytestiosapp://com.myapp.mytestapp`

App2 重新導向 URI： `x-msauth-mytestiosapp://com.myapp.mytestapp2`

App3 重新導向 URI： `x-msauth-mytestiosapp://com.myapp.mytestapp3`

....

它們在相同的用戶端識別碼 / 應用程式識別碼之下是巢狀的，並且可根據您在 SDK 組態中傳回給我們的重新導向 URI 查看。

```
+-------------------+
|                   |
|  Client ID        |
+---------+---------+
          |
          |           +-----------------------------------+
          |           |  App 1 Redirect URI               |
          +----------^+                                   |
          |           +-----------------------------------+
          |
          |           +-----------------------------------+
          +----------^+  App 2 Redirect URI               |
          |           |                                   |
          |           +-----------------------------------+
          |
          +----------^+-----------------------------------+
                      |  App 3 Redirect URI               |
                      |                                   |
                      +-----------------------------------+

```

以下將說明這些重新導向 URI 的格式。 您可以使用任何重新導向 URI，除非您想要支援訊息代理程式，在此情況下，它們的外觀必須如上所示*

#### <a name="create-keychain-sharing-between-applications"></a>建立應用程式之間共用的金鑰鏈

啟用金鑰鏈共用已超出本文範圍，包含在 Apple 的 [新增功能](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html)文件中。 您必須決定金鑰鍊的名稱，並且跨所有應用程式新增該功能。

當您確實已正確設定權利時，您應該會在標題為 `entitlements.plist` 的專案目錄中看到檔案，其中包含的項目外觀如下︰

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>keychain-access-groups</key>
    <array>
        <string>$(AppIdentifierPrefix)com.myapp.mytestapp</string>
        <string>$(AppIdentifierPrefix)com.myapp.mycache</string>
    </array>
</dict>
</plist>
```

一旦您在每個應用程式中啟用金鑰鏈權利，而且您已準備好使用單一登入，請在使用下列設定的 `ADAuthenticationSettings` 中，使用下列設定告訴身分識別 SDK 關於金鑰鏈的資訊：

```
defaultKeychainSharingGroup=@"com.myapp.mycache";
```

> [!WARNING]
> 當您跨應用程式共用金鑰鍊時，任何應用程式都可以刪除使用者，更糟的是可以跨應用程式刪除所有權杖。 如果您的應用程式依賴這些權杖來執行背景工作，這樣會造成可怕的災難。 共用金鑰鏈表示您在整個身分識別 SDK 必須非常小心進行任何移除作業。

就這麼簡單！ SDK 現在會跨所有應用程式共用認證。 使用者清單也會跨應用程式執行個體共用。

### <a name="turning-on-sso-for-broker-assisted-sso"></a>開啟訊息代理程式協助 SSO 的 SSO

應用程式使用任何已安裝在裝置上之訊息代理程式的功能 **預設為關閉**。 若要搭配使用應用程式與訊息代理程式，您必須執行一些額外的設定，並將一些程式碼新增至您的應用程式。

要遵循的步驟如下：

1. 在應用程式程式碼呼叫 MS SDK 時啟用訊息代理程式模式。
2. 建立新的重新導向 URI，並將其提供給應用程式與應用程式註冊。
3. 註冊 URL 配置。
4. iOS9 支援︰新增權限至 info.plist 檔案。

#### <a name="step-1-enable-broker-mode-in-your-application"></a>步驟 1︰在應用程式中啟用訊息代理程式模式

當您建立「內容」或驗證物件的初始設定時，已開啟應用程式使用訊息代理程式的功能。 您可以設定程式碼中的認證類型就可以辦到︰

```
/*! See the ADCredentialsType enumeration definition for details */
@propertyADCredentialsType credentialsType;
```
`AD_CREDENTIALS_AUTO` 設定可讓 SDK 嘗試呼叫訊息代理程式，`AD_CREDENTIALS_EMBEDDED` 會防止 SDK 呼叫訊息代理程式。

#### <a name="step-2-registering-a-url-scheme"></a>步驟 2︰註冊 URL 配置

身分識別平台會使用 URL 叫用訊息代理程式，然後將控制項傳回至您的應用程式。 若要完成該往返行程，您需要為應用程式註冊的 URL 配置，身分識別平台將會了解該配置。 此配置可以是您先前可能已向應用程式註冊之任何其他應用程式配置以外的配置。

> [!WARNING]
> 建議您讓 URL 配置唯一無二，將其他應用程式使用相同 URL 配置的機會降到最低。 Apple 不會強制要求在應用程式存放區中註冊之 URL 配置的獨特性。

以下是這個情形會如何出現在您的專案組態的範例。 您可能也會在 XCode 中這樣做︰

```
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.myapp.mytestapp</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>x-msauth-mytestiosapp</string>
        </array>
    </dict>
</array>
```

#### <a name="step-3-establish-a-new-redirect-uri-with-your-url-scheme"></a>步驟 3︰利用 URL 配置建立新的重新導向 URI

為了確保我們永遠傳回認證權杖給正確的應用程式，我們必須確定 iOS 作業系統可以確認我們回呼應用程式的方式。 iOS 作業系統會向 Microsoft 訊息代理程式應用程式回報呼叫它之應用程式的組合識別碼。 這樣就不會讓惡意應用程式假冒。 因此，我們利用這個方法搭配訊息代理程式應用程式的 URI，以確保將權杖傳回給正確的應用程式。 我們需要您在應用程式中建立此唯一重新導向 URI，並在我們的開發人員入口網站中設定為重新導向 URI。

您的重新導向 URI 必須是適當的格式︰

`<app-scheme>://<your.bundle.id>`

例如︰x-msauth-mytestiosapp://com.myapp.mytestapp 

此重新導向 URI 必須使用 [Azure 入口網站](https://portal.azure.com/)在應用程式註冊中指定。 如需 Azure AD 應用程式註冊的詳細資訊，請參閱 [與 Azure Active Directory 整合](active-directory-how-to-integrate.md)。

##### <a name="step-3a-add-a-redirect-uri-in-your-app-and-dev-portal-to-support-certificate-based-authentication"></a>步驟 3a︰在應用程式與開發人員入口網站中新增重新導向 URI 以支援以憑證為基礎的驗證

若要支援以憑證為基礎的驗證，必須在您的應用程式與 [Azure 入口網站](https://portal.azure.com/)中註冊第二個 "msauth"，才能在應用程式中新增該支援。

`msauth://code/<broker-redirect-uri-in-url-encoded-form>`

例如︰msauth://code/x-msauth-mytestiosapp%3A%2F%2Fcom.myapp.mytestapp

#### <a name="step-4-ios9-add-a-configuration-parameter-to-your-app"></a>步驟 4：iOS9︰將組態參數新增至您的應用程式

ADAL 會使用 –canOpenURL: 檢查裝置上是否已安裝訊息代理程式。 在 iOS 9 中，Apple 會鎖定應用程式可以查詢的配置。 您必須將 “msauth” 新增至 `info.plist file`的 LSApplicationQueriesSchemes 區段。

```
<key>LSApplicationQueriesSchemes</key> <array><string>msauth</string></array>
```

### <a name="youve-configured-sso"></a>您已設定 SSO！

現在身分識別 SDK 會自動跨應用程式共用認證，並在訊息代理程式出現在其裝置上時叫用它。

## <a name="next-steps"></a>後續步驟

* 了解[單一登入 SAML 通訊協定](single-sign-on-saml-protocol.md)