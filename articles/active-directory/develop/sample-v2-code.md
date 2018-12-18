---
title: Azure Active Directory 程式碼範例 | Microsoft Docs
description: 提供可用 Azure Active Directory (V2 端點) 程式碼範例的索引 (依案例分類)。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: a242a5ff-7300-40c2-ba83-fb6035707433
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: jmprieur
ms.custom: aaddev
ms.openlocfilehash: e76097c0d0cbaf14f2fc2b1a407bc2d320a2091d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964381"
---
# <a name="azure-active-directory-code-samples-v2-endpoint"></a>Azure Active Directory 程式碼範例 (V2 端點)

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

您可以使用 Microsoft Azure Active Directory (Azure AD)：

- 將驗證和授權新增至您的 Web 應用程式與 Web API。
- 要求存取權杖以存取受保護的 Web API。

本文簡短描述 Azure AD V2 端點，並提供範例連結。 這些範例將告訴您如何進行，並顯示可在應用程式中使用的程式碼片段。 在程式碼範例頁面上，您可以找到詳細的讀我主題，協助您了解需求並進行安裝及設定。 程式碼內有註解可幫助您了解重要區段。

> [!NOTE]
> 如果您對 V1 範例感興趣，請參閱 [Azure AD 程式碼範例 (V1 端點)](sample-v1-code.md)。

若要了解每個範例類型的基本情節，請參閱 [Azure Active Directory v2.0 端點的應用程式類型](v2-app-types.md)。

您也可以在 GitHub 上參與範例。 若要了解做法，請參閱 [Microsoft Azure Active Directory 範例與文件](https://github.com/Azure-Samples?page=3&query=active-directory)。

## <a name="single-page-applications-spa"></a>單頁應用程式 (SPA)

這個範例示範如何撰寫使用 Azure AD 保護的單頁應用程式。 這些範例會使用其中一種 MSAL.js：[Microsoft Authentication Library for JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)、[Microsoft Authentication Library for Angular](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)、[Microsoft Authentication Library for AngularJS](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs)

 平台 |  呼叫 Microsoft Graph
 -------- |  ---------------------
![JavaScript](media/sample-v2-code/logo_js.png) JavaScript (msal.js)  | [javascript-graphapi-web-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-web-v2) (英文)
![Angular JS](media/sample-v2-code/logo_angular.png) JavaScript (msal AngularJS) | [MsalAngularjsDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angularjs/samples/MsalAngularjsDemoApp)
![Angular](media/sample-v2-code/logo_angular.png) JavaScript (msal Angular) | [MSALAngularDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular/samples/MSALAngularDemoApp)

## <a name="web-applications"></a>Web 應用程式

下列範例會說明登入使用者的 Web 應用程式。 部分範例也會示範呼叫 Microsoft Graph 或使用使用者身分識別呼叫您自有 Web API 的應用程式。

 平台 | 僅登入使用者 | 登入使用者與呼叫 Microsoft Graph
 -------- | ------------------- | ---------------------------------
![ASP.NET](media/sample-v2-code/logo_NETframework.png)<p/> ASP.NET | [appmodelv2-webapp-openIDConnect-dotNet](https://GitHub.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) (英文) <p/> [dotnet-webapp-openidconnect-v2](https://GitHub.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2) (英文)  |              [aspnet-connect-rest-sample](https://github.com/microsoftgraph/aspnet-connect-rest-sample) (英文)
![ASP.NET](media/sample-v2-code/logo_NETcore.png)<p/>ASP.NET Core 2.0 | [aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2) (英文) |              [aspnetcore-connect-sample](https://github.com/microsoftgraph/aspnetcore-connect-sample) (英文)
![Node.js](media/sample-v2-code/logo_nodejs.png)  |                   | [AppModelv2-WebApp-OpenIDConnect-nodejs](https://github.com/azureadquickstarts/appmodelv2-webapp-openidconnect-nodejs) (英文)
![Ruby](media/sample-v2-code/logo_ruby.png) |                   | [ruby-connect-rest-sample](https://github.com/microsoftgraph/ruby-connect-rest-sample) (英文)

## <a name="desktop-and-mobile-public-client-apps"></a>桌上型與行動裝置公開用戶端應用程式

下列範例說明會代表使用者使用互動式登入存取 Microsoft Graph 或自有 Web API 的公用用戶端應用程式 (傳統型應用程式/行動應用程式)。 這些用戶端應用程式全都會使用 MicroSoft 驗證程式庫 (MSAL)

用戶端應用程式 | 平台 | 呼叫 Microsoft Graph | 呼叫 ASP.NET Core 2.0 Web API
------------------ | -------- |  -------------------- | -------------------------
桌上型 (WPF)      | ![.NET/C#](media/sample-v2-code/logo_NET.png) | [dotnet-desktop-msgraph-v2](http://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) (英文) <p/> [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) (英文) | [dotnet-native-aspnetcore-v2](https://GitHub.com/azure-samples/active-directory-dotnet-native-aspnetcore-v2) (英文)
行動裝置 (UWP)   | ![.NET/C# (UWP)](media/sample-v2-code/logo_windows.png) | [dotnet-native-uwp-v2](https://github.com/azure-samples/active-directory-dotnet-native-uwp-v2) (英文) |
行動裝置 (Android、iOS、UWP)   | ![.NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [xamarin-native-v2](https://Github.com/azure-samples/active-directory-xamarin-native-v2) (英文) |
行動裝置 (iOS)       | ![iOS/Objective C 或 swift](media/sample-v2-code/logo_iOS.png) | [ios-swift-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) (英文) <p/> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) (英文) |
行動裝置 (Android)   | ![Android/Java](media/sample-v2-code/logo_Android.png) |   [android-native-v2](https://github.com/azure-samples/active-directory-android-native-v2 ) (英文) |

## <a name="daemon-applications"></a>精靈應用程式

下列範例說明會使用其自有身分識別 (無使用者) 存取 Microsoft Graph 的應用程式。

用戶端應用程式 | 平台 | 流程/授與 | 呼叫 Microsoft Graph
------------------ | -------- | ---------- | --------------------
Web 應用程式 | ![ASP.NET](media/sample-v2-code/logo_NETframework.png)<p/> ASP.NET  | 用戶端認證 | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) (英文)

> 顯示桌面精靈應用程式的範例已列入待辦項目。

## <a name="web-apis"></a>Web API

以下範例示範如何透過 Azure AD V2 端點保護 Web API。 此 API 會由 WPF 應用程式執行 (但實際上可由任何應用程式加以呼叫)

平台 | 範例
 -------- | -------------------
![.NET/C#](media/sample-v2-code/logo_NET.png) | [dotnet-native-aspnetcore-v2](https://GitHub.com/azure-samples/active-directory-dotnet-native-aspnetcore-v2) (英文)

## <a name="other-microsoft-graph-samples"></a>其他 Microsoft Graph 範例

若要了解示範 Microsoft Graph API 不同使用模式 (包括使用 Azure AD 驗證) 的[範例](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) (英文) 和教學課程，請參閱 [Microsoft Graph 社群範例和教學課程](https://github.com/microsoftgraph/msgraph-community-samples) (英文).

## <a name="see-also"></a>另請參閱

[Azure Active Directory 開發人員指南](azure-ad-developers-guide.md)

[Azure AD 圖形 API 的概念和參考](https://msdn.microsoft.com/library/azure/hh974476.aspx)

[Azure AD Graph API 協助程式程式庫](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)
