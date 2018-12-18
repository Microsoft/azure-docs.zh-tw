---
title: Azure Active Directory 驗證程式庫 | Microsoft Docs
description: Azure AD 驗證程式庫 (ADAL) 可讓用戶端應用程式開發人員輕鬆地向雲端或內部部署 Active Directory (AD) 驗證使用者，然後取得存取權杖來保護 API 呼叫。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 2e4fc79a-0285-40be-8c77-65edee408a22
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/13/2018
ms.author: celested
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: d3f34ac3d284d61caf79cbd4929689106fdf7999
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980639"
---
# <a name="azure-active-directory-authentication-libraries"></a>Azure Active Directory 驗證程式庫

Azure Active Directory 驗證程式庫 (ADAL) v1.0 可讓應用程式開發人員向雲端或內部部署 Active Directory (AD) 驗證使用者，並取得權杖來保護 API 呼叫。 ADAL 透過下列功能使開發人員更容易驗證：

- 儲存存取權杖和重新整理權杖的可設定權杖快取
- 存取權杖到期而有重新整理權杖可用時的自動權杖重新整理
- 支援非同步方法呼叫

> [!NOTE]
> 在尋找 Azure AD v2.0 程式庫 (MSAL) 嗎？ 請參閱 [MSAL 程式庫指南](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-libraries)。
>
>

## <a name="microsoft-supported-client-libraries"></a>Microsoft 支援的用戶端程式庫

| 平台 | 程式庫 | 下載 | 原始程式碼 | 範例 | 參考
| --- | --- | --- | --- | --- | --- |
| .NET 用戶端、Windows 市集、UWP、Xamarin iOS 和 Android |ADAL .NET v3 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) | [傳統型應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-devquickstarts-dotnet) |[參考](https://docs.microsoft.com/dotnet/api/?view=identitymodelclientsad-3.13.9) |
| .NET 用戶端、Windows 市集、Windows Phone 8.1 |ADAL .NET v2 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/2.28.4) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/releases/tag/v2.28.4) | [傳統型應用程式](https://github.com/AzureADQuickStarts/NativeClient-DotNet/releases/tag/v2.X) | |
| JavaScript |ADAL.js |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[單一頁面應用程式](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) | |
| iOS、macOS |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc) |[iOS App](https://docs.microsoft.com/azure/active-directory/active-directory-devquickstarts-ios) | [參考](http://cocoadocs.org/docsets/ADAL/2.5.1/)|
| Android |ADAL |[中央儲存機制](http://search.maven.org/remotecontent?filepath=com/microsoft/aad/adal/) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-android) |[Android App](https://docs.microsoft.com/azure/active-directory/active-directory-devquickstarts-android) | [JavaDocs](http://javadoc.io/doc/com.microsoft.aad/adal/)|
| Node.js |ADAL |[npm](https://www.npmjs.com/package/adal-node) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) | [Node.js Web 應用程式](https://github.com/Azure-Samples/active-directory-node-webapp-openidconnect)|[參考](https://docs.microsoft.com/javascript/api/adal-node/?view=azure-node-latest) |
| Java |ADAL4J |[Maven](http://search.maven.org/#search%7Cga%7C1%7Ca%3Aadal4j%20g%3Acom.microsoft.azure) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-java) |[Java Web 應用程式](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect) |[參考](http://javadoc.io/doc/com.microsoft.azure/adal4j) |
| Python |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[Python Web 應用程式](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi) |[參考](http://adal-python.readthedocs.io/) |

## <a name="microsoft-supported-server-libraries"></a>Microsoft 支援的伺服器程式庫

| 平台 | 程式庫 | 下載 | 原始程式碼 | 範例 | 參考
| --- | --- | --- | --- | --- | --- |
| .NET |適用於 AzureAD 的 OWIN|[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.ActiveDirectory/) |[CodePlex](http://katanaproject.codeplex.com) |[MVC 應用程式](https://docs.microsoft.com/azure/active-directory/active-directory-devquickstarts-webapp-dotnet) | |
| .NET |適用於 OpenIDConnect 的 OWIN |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect) |[CodePlex](http://katanaproject.codeplex.com) |[Web 應用程式](https://github.com/AzureADSamples/WebApp-OpenIDConnect-DotNet) | |
| .NET |適用於 WS-同盟的 OWIN |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.WsFederation) |[CodePlex](http://katanaproject.codeplex.com) |[MVC Web 應用程式](https://github.com/AzureADSamples/WebApp-WSFederation-DotNet) | |
| .NET |適用於 .NET 4.5 的身分識別通訊協定延伸模組 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocol.Extensions) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| .NET |適用於 .NET 4.5 的 JWT 處理常式 |[NuGet](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| Node.js |Azure AD Passport |[npm](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [Web API](https://docs.microsoft.com/azure/active-directory/active-directory-devquickstarts-webapi-nodejs)| |

## <a name="scenarios"></a>案例

以下為三種在存取遠端資源的用戶端中使用 ADAL 的常見案例：

### <a name="authenticating-users-of-a-native-client-application-running-on-a-device"></a>驗證在裝置上執行原生用戶端應用程式的使用者

在此案例中，開發人員具有的行動用戶端或桌面應用程式必須存取遠端資源，例如 Web API。 Web API 不允許匿名呼叫，而且必須在通過驗證之使用者的內容中呼叫。 Web API 會預先設定成信任由特定 Azure AD 租用戶發出的存取權杖。 Azure AD 會預先設定成發行該資源的存取權杖。 若要從用戶端叫用 Web API，則開發人員會使用 ADAL 來協助向 Azure AD 驗證。 使用 ADAL 最安全的方法，就是讓其提供用來收集使用者認證的使用者介面 (顯示為瀏覽器視窗)。

ADAL 使得驗證使用者變得輕鬆，只需取得存取權杖並從 Azure AD 重新整理權杖，然後使用該存取權杖呼叫 Web API。

如需示範如何向 Azure AD 進行驗證的程式碼範例，請參閱 [原生用戶端 WPF 應用程式至 Web API](https://github.com/azureadsamples/nativeclient-dotnet)。

### <a name="authenticating-a-confidential-client-application-running-on-a-web-server"></a>驗證網頁伺服器上執行的機密用戶端應用程式

在此案例中，開發人員具有在伺服器上執行且需要存取遠端資源 (例如 Web API) 的應用程式。 Web API 不允許匿名呼叫，且必須從授權的服務呼叫。 Web API 會預先設定成信任由特定 Azure AD 租用戶發出的存取權杖。 Azure AD 預先設定成將該資源的存取權杖發給具有用戶端認證 (用戶端 ID 和祕密) 的服務。 ADAL 可促進透過 Azure AD 進行服務驗證，Azure AD 會傳回可用於呼叫 Web API 的存取權杖。 ADAL 也會處理存取權杖的存留期的管理，方法是快取它並視需要更新。 如需示範此案例的程式碼範例，請參閱[精靈主控台應用程式至 Web API](https://github.com/AzureADSamples/Daemon-DotNet)。

### <a name="authenticating-a-confidential-client-application-running-on-a-server-on-behalf-of-a-user"></a>代表使用者，驗證伺服器上執行的機密用戶端應用程式

在此案例中，開發人員具有在伺服器上執行的 Web 應用程式，需要存取由 Azure AD 所保護的遠端資源，例如 Web API。 Web API 不允許匿名呼叫，因此必須從代表經驗證使用者的授權服務呼叫。 Web API 預先設定為信任由特定 Azure AD 租用戶所發出的存取權杖，Azure AD 則預先設定為將該資源的存取權杖發給具有用戶端認證的服務。 一旦在 Web 應用程式中驗證了使用者，應用程式便可以從 Azure AD 取得使用者的授權碼。 然後，Web 應用程式可以使用 ADAL 代表使用者，利用與應用程式相關聯的授權碼和用戶端認證從 Azure AD 取得存取權杖和更新權杖。 在 Web 應用程式取得存取權杖之後，它可以呼叫 Web API，直到權杖到期。 權杖到期時，Web 應用程式可以使用 ADAL 透過先前收到的更新權杖來取得新的存取權杖。 如需示範此案例的程式碼範例，請參閱[原生用戶端至 Web API](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof)。

## <a name="see-also"></a>另請參閱

- [Azure Active Directory 開發人員指南](azure-ad-developers-guide.md)
- [Azure Active Directory 的驗證案例](authentication-scenarios.md)
- [Azure Active Directory 程式碼範例](sample-v1-code.md)
