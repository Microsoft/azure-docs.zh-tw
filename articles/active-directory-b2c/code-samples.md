---
title: Azure Active Directory B2C 程式碼範例 | Microsoft Docs
description: Azure Active Directory B2C 行動裝置、傳統型、Web 和單頁應用程式的程式碼範例。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.author: davidmu
ms.date: 01/23/2018
ms.custom: mvc
ms.topic: sample
ms.service: active-directory
ms.component: B2C
ms.openlocfilehash: ffb60ab07ef21a94a7ba978fe354948f18479a37
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37446555"
---
# <a name="azure-active-directory-b2c-code-samples"></a>Azure Active Directory B2C 程式碼範例

下表提供 iOS、Android、.NET 和 Node.js 等應用程式的範例連結。

## <a name="mobile-and-desktop-apps"></a>行動和傳統型應用程式

| 範例 | 說明 |
|--------| ----------- |
| [ios-swift-native-msal](https://github.com/Azure-Samples/active-directory-b2c-ios-swift-native-msal) | 以 Swift 撰寫的 iOS 範例，其使用 OAuth 2.0 驗證 Azure AD B2C 使用者和呼叫 API |
| [android-native-msal](https://github.com/Azure-Samples/active-directory-b2c-android-native-msal) | 簡單的 Android 應用程式，其展示如何透過 Azure Active Directory B2C 使用 MSAL 來驗證使用者，以及使用所產生的權杖 MSAL 來存取 Web API。 |
| [ios-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-ios-native-appauth) | 此範例示範如何使用第三方程式庫以 Objective-C 建置 iOS 應用程式，而該應用程式會向我們的 Azure AD B2C 識別服務驗證 Microsoft 身分識別使用者。 |
| [android-native-appauth](https://github.com/Azure-Samples/active-directory-b2c-android-native-appauth) | 此範例示範如何使用第三方程式庫建置 Android 應用程式，而該應用程式會向我們的 B2C 識別服務驗證 Microsoft 身分識別使用者並使用 OAuth 2.0 存取權杖呼叫 Web API。 |
| [dotnet-desktop](https://github.com/Azure-Samples/active-directory-b2c-dotnet-desktop) | 此範例示範 Windows Desktop .NET (WPF) 應用程式如何使用 Azure AD B2C 登入使用者、使用 MSAL.NET 取得存取權杖，以及呼叫 API。 | 
| [xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | 簡單的 Xamarin Forms 應用程式，其展示如何透過 Azure Active Directory B2C 使用 MSAL 來驗證使用者，以及使用所產生的權杖 MSAL 來存取 Web API。 |

## <a name="web-apps-and-apis"></a>Web Apps 和 API

| 範例 | 說明 |
|--------| ----------- |
| [dotnet-webapp-and-webapi](https://github.com/Azure-Samples/active-directory-b2c-dotnet-webapp-and-webapi) | .NET Web 應用程式的合併範例，該應用程式會呼叫 .NET Web API (兩者均使用 Azure AD B2C 保護)。 |
| [dotnetcore-webapp](https://github.com/Azure-Samples/active-directory-b2c-dotnetcore-webapp) | ASP.NET Core Web 應用程式，該應用程式可以使用 Azure AD B2C 登入使用者、使用 MSAL.NET 取得存取權杖，以及呼叫 API。 |
| [openidconnect-nodejs](https://github.com/AzureADQuickStarts/B2C-WebApp-OpenIDConnect-NodeJS) | Node.js 應用程式，該應用程式可供輕鬆快速地使用 OpenID Connect 設定 Web 應用程式與 Express。 |
| [javascript-nodejs-webapp](https://github.com/AzureADQuickStarts/active-directory-b2c-javascript-nodejs-webapp) | Node.js 伺服器，該伺服器可供輕鬆快速地使用 OAuth2 通訊協定設定 REST API 服務。 |
| [javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) | 適用於 Azure AD B2C 的小型 Node.js Web API，示範如何使用 passport.js 保護您的 Web API 並接受 B2C 存取權杖。 |

## <a name="single-page-apps"></a>單一頁面應用程式

| 範例 | 說明 |
|--------| ----------- |
| [javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) | 呼叫 Web API 的單頁應用程式 (SPA)。 運用 MSAL.js 向 Azure AD B2C 進行驗證。 | 
| [javascript-hellojs-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-hellojs-singlepageapp) | 使用 ASP.NET Web API 後端實作的單頁應用程式，該應用程式會使用 Azure AD B2C 來註冊及登入使用者，以及使用 OAuth 2.0 存取權杖來呼叫 Web API。 |