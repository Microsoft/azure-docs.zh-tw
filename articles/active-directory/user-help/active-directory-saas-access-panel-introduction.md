---
title: 什麼是 Azure Active Directory 中的 MyApps 入口網站？ | Microsoft Docs
description: 了解如何使用各種 MyApps 入口網站 (網頁瀏覽器、Android 應用程式、iPhone 和 iPad 應用程式) 存取 SaaS 應用程式。
services: active-directory
author: eross-msft
manager: mtillman
ms.assetid: c0252d01-7e6e-4f79-a70e-600479577dfd
ms.service: active-directory
ms.component: user-help
ms.workload: identity
ms.topic: conceptual
ms.date: 05/11/18
ms.author: lizross
ms.reviewer: asteen
ms.openlocfilehash: e382f53678806aa3442151703277c3a57bb30830
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302272"
---
# <a name="what-is-the-myapps-portal"></a>什麼是 MyApps 入口網站？

如果您在 Azure Active Directory (Azure AD) 中具有公司或學校帳戶，可以使用 MyApps 網頁型入口網站檢視和啟動 Azure AD 系統管理員已授權您存取的雲端式應用程式。 您也可以透過 MyApps 入口網站使用自助群組和應用程式管理功能。

MyApps 入口網站會與 Azure 入口網站分開。 您不需要具有 Azure 訂用帳戶。

![MyApps 入口網站][1] 您可以使用 MyApps 入口網站編輯某些設定檔的設定，並執行下列作業：

- 變更與公司或學校帳戶相關聯的密碼。

- 編輯密碼重設設定。

- 編輯與 Multi-Factor Authentication 相關的連絡人和喜好設定的設定 (系統管理員已要求使用此功能的帳戶)。

- 檢視帳戶詳細資料，例如，使用者識別碼、替代電子郵件、行動電話和辦公室電話號碼以及裝置。

- 檢視和啟動 Azure AD 系統管理員已授權您存取的雲端式應用程式。 

- 自我管理群組。 系統管理員可以在 Azure AD 中建立及管理安全性群組，並要求安全性群組成員資格。 如需詳細資訊，請參閱 [Azure AD 中使用者的自助群組管理](../users-groups-roles/groups-self-service-management.md)與[管理群組](../fundamentals/active-directory-manage-groups.md)。

## <a name="access-the-myapps-portal"></a>存取 MyApps 入口網站

您可以前往 `http://myapps.microsoft.com` 存取 MyApps 入口網站。

如果您已經為您的登入頁面設定自訂商標，就可將貴組織的網域附加到 URL (例如，`http://myapps.microsoft.com/<your domain>.com`)，以載入此商標。

您可以使用已在 Azure 入口網站中設定的任何使用中或已驗證網域名稱，如下所示：![Wingtip Toys 網域名稱][2]  

將 URL 散佈給要登入與 Azure AD 整合之應用程式的所有使用者。

## <a name="authentication"></a>驗證

若要前往 MyApps 入口網站，您必須透過 Azure AD 中的公司或學校帳戶進行驗證。 Azure AD 可以直接驗證您。 或者，如果組織已經使用 Active Directory 同盟服務 (ADFS) 或其他技術設定同盟，則可由 Windows Server Active Directory 驗證您。

如果您具備 Azure 或 Office 365 的訂用帳戶，而且已在使用 Azure 入口網站或 Office 365 應用程式，就可以檢視應用程式清單，而不需再次登入。 如果您未經過驗證，系統會提示您在 Azure AD 中使用您帳戶的使用者名稱和密碼進行登入。 如果您的組織已設定同盟，則輸入使用者名稱已經足夠。

經過驗證後，您就能夠與由系統管理員整合到目錄的應用程式互動。 若要了解如何整合應用程式與 Azure AD，請參閱 [什麼是搭配 Azure Active Directory 的應用程式存取和單一登入？](../manage-apps/what-is-single-sign-on.md)。

## <a name="web-browser-requirements"></a>網頁瀏覽器需求

MyApps 入口網站至少需要有支援 JavaScript 且啟用 CSS 的瀏覽器。 若要透過密碼型單一登入 (SSO) 登入應用程式，您必須在瀏覽器中安裝 MyApps 入口網站延伸模組。 當您選取已設定密碼型 SSO 的應用程式時，系統就會自動下載此擴充功能。

安裝程式是架構特定的。 如果您按一下下載連結，將只會取得適用於您目前執行之 OS 架構的安裝程式。 如果您是應用程式部署管理員，請務必從 64 位元和 32 位元裝置瀏覽下載連結，以取得這兩種安裝程式。


MyApps 入口網站延伸模組目前適用於：
- **Edge**：在 Windows 10 Anniversary Edition 或更新版本上。 
- **Chrome**：在 Windows 7 或更新版本，和在 MacOS X 或更新版本上。
- **Firefox 26.0 或更新版本**：在 Windows XP SP2 或更新版本，和在 Mac OS X 10.6 或更新版本上。
- **Internet Explorer 11**：在 Windows 7 或更新版本上 (部分支援)。

## <a name="my-apps-secure-sign-in-extension"></a>我的應用程式安全登入延伸模組
若要登入密碼型單一登入，您就必須使用擴充功能。 在安裝擴充功能之後，您可以選取 [登入以開始使用]，登入擴充功能來啟用其他功能。 

- 您可以使用應用程式的 [登入 URL] 來直接登入應用程式。 當您使用應用程式的 URL 時，擴充功能會偵測動作，並讓您選擇從擴充功能登入。
- 您可以從 MyApps 入口網站使用延伸模組的 [快速搜尋] 功能，來啟動其任何應用程式。 
- 擴充功能會顯示您在 [最近使用過] 區段中啟動的最後三個應用程式。
- 您可以在遠端透過[應用程式 Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-get-started) 時使用公司內部 URL

> [!NOTE]
> 其他功能僅適用於 Edge、Chrome 和 Firefox。
>
您也可以直接從下列網站下載延伸模組：
- [Chrome](https://go.microsoft.com/fwlink/?linkid=866367)
- [Edge](https://go.microsoft.com/fwlink/?linkid=845176)
- [Firefox](https://go.microsoft.com/fwlink/?linkid=866366)

如果您是使用 `https://myapps.microsoft.com` 以外的 My Apps URL，請執行下列作業來設定您的預設 URL：
1. 在尚未登入擴充功能時，以滑鼠右鍵按一下擴充功能圖示。
2. 在功能表上，選取 [My Apps URL]。
3. 選取預設 URL。
4. 按一下擴充功能圖示。
5. 選取 [登入以開始使用]。

若要在遠端使用延伸模組時使用公司內部 URL，請執行下列作業：
1. 在您的租用戶上[設定應用程式 Proxy](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-enable)。
2. 透過應用程式 Proxy [發佈應用程式](https://docs.microsoft.com/azure/active-directory/application-proxy-publish-azure-portal)和 URL。
3. 安裝延伸模組，然後藉由選取 [登入以開始使用] 來登入該延伸模組。
4. 即使在遠端，您還是可以立即瀏覽至公司內部 URL。

> [!NOTE]
> 您也可能關閉自動重新導向至公司 URL，方法是選取主功能表上的設定齒輪，然後針對公司內部 URL 重新導向選項選取 [關閉]。


## <a name="mobile-app-support"></a>行動應用程式支援

Azure Active Directory 團隊發行「我的應用程式」行動應用程式。 當您安裝此應用程式時，可以在 iOS 和 Android 裝置上登入密碼 SSO 應用程式。

> [!NOTE]
> 您不需要外掛程式或行動應用程式，就能在幾乎任何裝置的任何網頁瀏覽器上，登入支援與 Azure AD 同盟的應用程式 (包括 Salesforce、Google Apps、Dropbox、Box、Concur、Workday、Office 365 和其他 70 種以上的應用程式)。 其他 [MyApps 入口網站體驗](https://myapps.microsoft.com/)也不需要 My Apps 行動應用程式，就能在行動裝置上使用。
>
>

### <a name="my-apps-for-android"></a>My Apps for Android

執行 Android 4.1 版和更新版本的任何 Android 裝置支援 My Apps for Android。  

已在 [Google Play 商店](https://play.google.com/store/apps/details?id=com.microsoft.myapps)中提供。

![My Apps for Android][3]   

### <a name="my-apps-for-iphone-and-ipad"></a>My Apps for iPhone 和 My Apps for iPad

執行 iOS 第 7 版和更新版本的任何 iPhone 或 iPad 皆支援 My Apps for iOS。  

已在 [Apple App Store](https://itunes.apple.com/us/app/my-apps-azure-active-directory/id824048653?mt=8) 中提供。

![My Apps for iOS][4]    


## <a name="managed-browser-for-my-apps"></a>適用於 My Apps 的受控瀏覽器

My Apps 也會與 Intune Managed Browser 整合。 在協助確保行動裝置上的資料安全無虞之際，適用於 iOS 和 Android 裝置的 Intune Managed Browser 扮演了重要角色。 瀏覽器可讓您安全地檢視和瀏覽可能包含公司資訊的網頁，並協助提供安全的網頁瀏覽體驗。  

您可在 Managed Browser 首頁上以及您的書籤中快速存取 My Apps，只需按幾下滑鼠就能觸達您需要存取的任何應用程式。

Intune Managed Browser 已在 [Apple App Store](https://itunes.apple.com/us/app/microsoft-intune-managed-browser/id943264951?mt=8) 和 [Google Play 商店](https://play.google.com/store/apps/details?id=com.microsoft.intune.mam.managedbrowser&hl=en)中提供。

![適用於 My Apps 的受控瀏覽器][5]    


## <a name="tips-for-testing-the-user-experience"></a>測試使用者體驗的秘訣

如果您是 Azure 系統管理員，而且已使用目錄中的帳戶登入 Azure 入口網站，即會自動以目前的帳戶登入 MyApps 入口網站。 此檢視會顯示已指派給您的所有應用程式。

若要在不同使用者帳戶中進行測試，請執行下列作業：

1. 在 Azure 入口網站或 MyApps 入口網站右上角選取 [登出]。 
2. 前往 [MyApps 入口網站](http://myapps.microsoft.com)。
3. 在登入頁面上，在您需要測試的目錄中，輸入帳戶的使用者名稱和密碼。


## <a name="starting-applications"></a>啟動應用程式

本節將討論可能出現在 MyApps 入口網站上的數種應用程式。

### <a name="office-365-applications"></a>Office 365 應用程式

如果組織使用 Office 365 應用程式，且您已取得其授權，則 Office 365 應用程式會出現在您的 MyApps 入口網站上。

當您選取 Office 365 應用程式的應用程式圖格時，系統會將您重新導向至該應用程式，並自動登入。

### <a name="microsoft-and-third-party-applications-configured-with-federation-based-sso"></a>已設定同盟 SSO 的 Microsoft 及第三方應用程式

系統管理員可以在 Azure 入口網站的 Active Directory 區段中新增應用程式，並將 SSO 模式設定為 [Azure AD 單一登入]。 只有在系統管理員已明確授權您存取應用程式時，您才能看到這些應用程式。

當您選取應用程式的圖格時，系統會將您重新導向並自動登入。

### <a name="password-based-sso-without-identity-provisioning"></a>不含身分識別佈建的密碼 SSO

系統管理員可以在 Azure 入口網站的 Active Directory 區段中新增應用程式，並將 SSO 模式設定為 [密碼單一登入]。 目錄中的所有使用者都可以看到已設定為此模式的所有應用程式。

您第一次選取應用程式圖格時，系統會提示您在 Internet Explorer 或 Chrome 中安裝密碼 SSO 外掛程式。 安裝之後，您可能需要重新啟動網頁瀏覽器。 當您返回 MyApps 入口網站，並再次選取應用程式圖格時，系統會提示您輸入應用程式的使用者名稱和密碼。 當您輸入使用者名稱和密碼之後，這些認證會安全地儲存並連結至您在 Azure AD 中的帳戶。

您下次選取應用程式圖格時，就會自動登入應用程式。  

您不需要再次輸入認證，或安裝密碼 SSO 外掛程式。

如果您的認證已在目標第三方應用程式中變更，那麼您也必須更新儲存在 Azure AD 中的認證。 

若要更新您的認證，請執行下列作業：

1. 選取應用程式圖格上的圖示。
2. 選取 [更新認證]，重新輸入應用程式的使用者名稱和密碼。


### <a name="password-based-sso-with-identity-provisioning"></a>含身分識別佈建的密碼 SSO

系統管理員可以在 Azure 入口網站的 [Active Directory] 區段中新增應用程式，並將 SSO 模式設定為 [密碼單一登入]，同時佈建身分識別。

您第一次選取應用程式圖格時，系統會提示您在 Internet Explorer 或 Chrome 中安裝密碼 SSO 外掛程式。 安裝之後，您可能需要重新啟動網頁瀏覽器。  

當您返回 MyApps 入口網站，並再次選取應用程式圖格時，就會自動登入應用程式。

有些應用程式可能會要求您在第一次登入時變更密碼。 如果您的認證已在目標第三方應用程式中變更，那麼您也必須更新儲存在 Azure AD 中的認證。 

若要更新您的認證，請執行下列作業：

1. 選取應用程式圖格上的圖示。
2. 選取 [更新認證]，重新輸入應用程式的使用者名稱和密碼。


### <a name="application-with-existing-sso-solutions"></a>含現有 SSO 解決方案的應用程式

若要為應用程式設定 SSO，Azure 入口網站會提供名為 [現有的單一登入] 的第三個選項。 這個選項可讓系統管理員建立應用程式的連結，並將連結放在選定使用者的 MyApps 入口網站上。

例如，如果應用程式是設定為使用 AD FS 2.0 來驗證使用者，系統管理員就可以使用 [現有的單一登入] 選項，在 MyApps 入口網站上建立該應用程式的連結。 當您存取連結時，將會透過 AD FS 2.0 或應用程式提供的任何現有 SSO 解決方案來驗證您。


## <a name="next-steps"></a>後續步驟

- 若要檢視與應用程式管理相關的所有主題清單，請參閱 [Azure Active Directory 中的應用程式管理文章索引](../active-directory-apps-index.md)。
 
- 若要深入了解如何將 SaaS 應用程式與 Azure AD 整合，請參閱[有關如何整合 SaaS 應用程式的教學課程清單](../saas-apps/tutorial-list.md)。
 
- 若要深入了解如何使用 Azure AD 管理應用程式，請參閱[單一登入及使用 Azure Active Directory 管理應用程式存取的簡介](../manage-apps/what-is-single-sign-on.md)。
 
- 若要深入了解使用者佈建，請參閱 [SaaS 應用程式的自動化使用者佈建和解除佈建](../active-directory-saas-app-provisioning.md)。

<!--Image references-->
[1]: ./media/active-directory-saas-access-panel-introduction/01.png
[2]: ./media/active-directory-saas-access-panel-introduction/02.png
[3]: ./media/active-directory-saas-access-panel-introduction/03.png
[4]: ./media/active-directory-saas-access-panel-introduction/04.png
[5]: ./media/active-directory-saas-access-panel-introduction/05.png
