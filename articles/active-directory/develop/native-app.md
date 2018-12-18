---
title: Azure Active Directory 中的原生應用程式
description: 說明何謂原生應用程式，以及此應用程式類型的通訊協定流程、註冊與權杖到期的基本概念。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ms.openlocfilehash: 8879cc331c5d0e8c6aab24e8c6178d96ab23bf9d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46955393"
---
# <a name="native-apps"></a>原生應用程式

原生應用程式是可代表使用者呼叫 Web API 的應用程式。 此案例是根據 OAuth 2.0 授權碼授與類型和公用用戶端，如 [OAuth 2.0 規格](http://tools.ietf.org/html/rfc6749)第 4.1 節所述。 此原生應用程式使用 OAuth 2.0 通訊協定，為使用者取得存取權杖。 接著，此存取權杖隨著要求傳送至 Web API，Web API 再授權使用者並傳回所需的資源。

## <a name="diagram"></a>圖表

![原生應用程式到 Web API 圖表](./media/authentication-scenarios/native_app_to_web_api.png)

## <a name="protocol-flow"></a>通訊協定流程

如果您使用 AD 驗證程式庫，則會為您處理如下所述的大部分通訊協定細節，例如瀏覽器快顯視窗、權杖快取和重新整理權杖的處理。

1. 原生應用程式使用瀏覽器快顯視窗，對 Azure AD 中的授權端點提出要求。 此要求包含原生應用程式的應用程式識別碼和重新導向 URI (如 Azure 入口網站所示)，以及 Web API 的應用程式識別碼 URI。 如果使用者尚未登入，系統會提示他們再次登入。
1. Azure AD 驗證使用者。 如果是多租用戶應用程式，且需要同意才能使用應用程式，則會要求使用者同意 (如果他們還沒有同意)。 表示同意且成功驗證之後，Azure AD 會發出授權碼回應傳回至用戶端應用程式的重新導向 URI。
1. 當 Azure AD 發出授權碼回應傳回至重新導向 URI 時，用戶端應用程式會停止瀏覽器互動，並從回應中擷取授權碼。 用戶端應用程式會使用此授權碼，傳送要求至 Azure AD 的權杖端點，此要求包含授權碼、用戶端應用程式的詳細資料 應用程式識別碼和重新導向 URI)，以及所需的資源 (Web API 的應用程式識別碼 URI)。
1. Azure AD 驗證授權碼及用戶端應用程式和 Web API 的相關資訊。 成功驗證後，Azure AD 會傳回兩個權杖：JWT 存取權杖和 JWT 重新整理權杖。 此外，Azure AD 會傳回使用者的基本資訊，例如其顯示名稱和租用戶識別碼。
1. 用戶端應用程式使用傳回的 JWT 存取權杖，透過 HTTPS，在對 Web API 的要求的 Authorization 標頭中加上 JWT 字串並指定 "Bearer"。 接著，Web API 驗證 JWT 權杖，如果驗證成功，則傳回所需的資源。
1. 當存取權杖到期時，用戶端應用程式會收到錯誤，指出使用者必須再次驗證。 如果應用程式具有有效的重新整理權杖，它可用來取得新的存取權杖，不會提示使用者重新登入。 如果重新整理權杖到期，應用程式必須再一次以互動方式驗證使用者。

> [!NOTE]
> Azure AD 所簽發的重新整理權杖可用來存取多個資源。 例如，如果您的用戶端應用程式有權限呼叫兩個 Web API，重新整理權杖也可用來取得其他 Web API 的存取權杖。

## <a name="code-samples"></a>程式碼範例

請參閱「原生應用程式到 Web API」案例的程式碼範例。 請經常回來查看，我們會頻繁新增新的範例。 [原生應用程式到 Web API](sample-v1-code.md#desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api)。

## <a name="app-registration"></a>應用程式註冊

若要註冊應用程式與 Azure AD v1.0 端點，請參閱[使用 Azure AD v1.0 端點註冊應用程式](quickstart-v1-add-azure-ad-app.md)。

* 單一租用戶 - 原生應用程式和 Web API 必須註冊在 Azure AD 的相同目錄中。 Web API 可以設定為公開一組權限，用以限制原生應用程式對其資源的存取權。 用戶端應用程式即可從 Azure 入口網站的 [其他應用程式的權限] 下拉式功能表中，選取所需的權限。
* 多租用戶 - 首先，原生應用程式僅註冊在開發人員或發行者的目錄中。 第二，設定原生應用程式來指出它運作所需的權限。 當目的地目錄中的使用者或系統管理員同意應用程式時 (使得應用程式可供組織使用)，這份必要權限清單會顯示在對話方塊中。 有些應用程式只需要使用者層級權限，亦即組織中的任何使用者都可以同意應用程式。 其他應用程式需要系統管理員層級權限，亦即組織中的使用者無法同意應用程式。 只有目錄管理員才能對需要此權限層級的應用程式表示同意。 當使用者或系統管理員同意時，只有 Web API 會註冊在他們的目錄中。 

## <a name="token-expiration"></a>權杖到期

當原生應用程式使用其授權碼來取得 JWT 存取權杖時，它也會收到 JWT 重新整理權杖。 當存取權杖到期時，重新整理權杖可用來重新驗證使用者，而不需要他們再次登入。 然後，此重新整理權杖用來驗證使用者，結果會產生新的存取權杖和重新整理權杖。

## <a name="next-steps"></a>後續步驟

- 深入了解其他[應用程式類型和案例](app-types.md)
- 深入了解 Azure AD [驗證基本概念](authentication-scenarios.md)
