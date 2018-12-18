---
title: 了解 Azure AD 中的 OAuth2 隱含授與流程 | Microsoft Docs
description: 深入了解 Azure Active Directory 的 OAuth2 隱含授與流程實作，以及它是否適合您的應用程式。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 90e42ff9-43b0-4b4f-a222-51df847b2a8d
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
ms.openlocfilehash: acdc3417643484fa98b16c4be1b83a44a8b73fc6
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962994"
---
# <a name="understanding-the-oauth2-implicit-grant-flow-in-azure-active-directory-ad"></a>了解 Azure Active Directory (AD) 中的 OAuth2 隱含授與流程

OAuth2 隱含授與是 OAuth2 規格中安全性疑慮最多的授與方式，因此聲名狼藉。 然而，這卻是 ADAL JS 所實作的方法，也是我們建議用來撰寫 SPA 應用程式的方法。 為何會這樣呢？ 這全是權衡利弊之後的結果︰而且事實也證明，對於透過 JavaScript 從瀏覽器使用 Web API 的應用程式來說，隱含授與是您所能找到的最好方法。

## <a name="what-is-the-oauth2-implicit-grant"></a>什麼是 OAuth2 隱含授與？

典型的 [OAuth2 授權碼授與](https://tools.ietf.org/html/rfc6749#section-1.3.1) \(英文\) 是使用兩個不同端點的授權授與。 授權端點會用於使用者互動階段，進而產生授權碼。 接著，用戶端會使用權杖端點來交換存取權杖的代碼，而且經常也會交換重新整理權杖。 Web 應用程式必須向權杖端點提交自己的應用程式認證，授權伺服器才能驗證用戶端。

[OAuth2 隱含授與](https://tools.ietf.org/html/rfc6749#section-1.3.2)是其他授權授與的一種變體。 它可讓用戶端直接從授權端點取得存取權杖 (使用 [OpenId Connect](http://openid.net/specs/openid-connect-core-1_0.html) 時，還可取得 id_token)，既不需要連絡權杖端點，也不需要驗證用戶端應。 這個變體是針對在網頁瀏覽器中執行的 JavaScript 型應用程式所設計︰在原始的 OAuth2 規格中，權杖會在 URI 片段中傳回。 這可讓權杖位元可供用戶端中的 JavaScript 程式碼使用，卻又保證權杖位元不會包含在朝向伺服器的重新導向中。 透過瀏覽器傳回權杖會直接從授權端點重新導向。 它也有不需要跨原始來源呼叫的優點，如果需要 JavaScript 應用程式才能連絡權杖端點，就需要這些呼叫。

OAuth2 隱含授與的重要特性就是，這類流程絕對不會將重新整理權杖傳回到用戶端的事實。 下一節說明為何這是不必要的，而且事實上會成為安全性問題。

## <a name="suitable-scenarios-for-the-oauth2-implicit-grant"></a>OAuth2 隱含授與的適用案例

OAuth2 規格聲明設計出隱含授與是為了實現使用者代理程式應用程式，也就是在瀏覽器內執行的 JavaScript 應用程式。 這類應用程式的鮮明特徵是，JavaScript 程式碼可用於存取伺服器資源 (通常是 Web API)，並據以更新應用程式使用者體驗。 請想想 Gmail 或 Outlook Web Access 之類的應用程式︰當您選取收件匣中的訊息時，只有訊息視覺化面板會變更以顯示新的選取內容，該頁面的其餘部分則維持不變。 這個特徵明顯不同於傳統的重新導向型 Web 應用程式，在後者中，每個使用者互動都會造成整頁回傳，並針對整頁轉譯做出新的伺服器回應。

採用極端 JavaScript 型方法的應用程式稱為單一頁面應用程式，或 SPA。 其概念是，這些應用程式只會提供初始的 HTML 網頁和相關聯的 JavaScript，至於所有後續的互動，則由透過 JavaScript 所執行的 Web API 呼叫來推動。 不過，應用程式大多是由回傳所驅動，但偶爾會執行 JS 呼叫的混合式方法也並非罕見；關於隱含流程使用的討論也與這些方法有關。

重新導向型應用程式通常會透過 Cookie 來保護其要求，不過，這種方法並非也適用於 JavaScript 應用程式。 Cookie 只會針對其產生網域發生作用，但 JavaScript 呼叫卻可能導向到其他網域。 事實上，情況往往是如此︰請想想叫用 Microsoft Graph API、Office API、Azure API 的應用程式，這些應用程式全都位於提供應用程式的網域之外。 JavaScript 應用程式的發展趨勢是完全沒有後端，全部依靠協力廠商 Web API 來實作其商務功能。

目前，保護 Web API 呼叫的慣用方法是使用 OAuth2 持有人權杖方法，在此方法中，每個呼叫都會伴隨一個 OAuth2 存取權杖。 Web API 會檢查傳入的存取權杖，而且如果它在權杖中找到所需的範圍，便會授與所要求作業的存取權。 隱含流程提供了方便的機制供 JavaScript 應用程式取得 Web API 的存取權杖，並提供很多關於 Cookie 的優點︰

* 可以可靠地取得權杖而不需要跨原始來源呼叫 – 強制註冊權杖要傳回到的重新導向 URI 可保證權杖不會移到其他位置
* JavaScript 應用程式可以針對任意數量的鎖定 Web API 取得所需數量的存取權杖 – 不限網域
* 工作階段或本機儲存體等 HTML5 功能可授與權杖快取和存留期管理的完全控制權，但是應用程式則無法處理 Cookie 管理
* 存取權杖不容易遭受跨網站偽造要求 (CSRF) 攻擊

隱含授與流程不會簽發重新整理權杖，其原因大多是安全性考量。 重新整理權杖的範圍不像存取權杖那麼窄，前者會授與更多權力，因此萬一洩露出去，將會造成更大的損害。在隱含流程中，權杖會在 URL 中傳遞，因此遭到攔截的風險高於授權碼授與。

不過，JavaScript 應用程式可以選擇使用另一種機制來更新存取權杖，而不會重複提示使用者提供認證。 應用程式可以使用隱藏的 iframe 來針對 Azure AD 的授權端點執行新的權杖要求︰只要瀏覽器仍有針對 Azure AD 網域作用的工作階段 (read: 有工作階段 Cookie)，驗證要求就可以順利執行而不需要使用者互動。

此模型會對 JavaScript 應用程式授與能力，使其能夠獨立更新存取權杖，甚至是取得新 API 的新存取權杖 (前提是使用者已同意)。 這可避免取得、維護和保護高價值構件 (例如重新整理權杖) 的額外負擔。 讓無訊息更新得以實現的成品 (Azure AD 工作階段 Cookie) 是在應用程式之外進行管理的。 這種方法的另一個優點是使用者可以使用任何已登入 Azure AD，並在任何瀏覽器索引標籤中執行的應用程式，從 Azure AD 登出。 這會導致 Azure AD 工作階段 Cookie 遭到刪除，而且 JavaScript 應用程式會自動失去為已登出使用者更新權杖的能力。

## <a name="is-the-implicit-grant-suitable-for-my-app"></a>我的應用程式適用隱含授與嗎？

隱含授與所帶來的風險高於其他授與，您需要注意的地方都有詳細的文件記載。 例如，[在隱含流程中誤用存取權杖來模擬資源擁有者 (英文)][OAuth2-Spec-Implicit-Misuse]和 [OAuth 2.0 威脅模型和安全性考量 (英文)][OAuth2-Threat-Model-And-Security-Implications]。 不過，風險概況之所以較高，主要是因為它要啟用執行作用中程式碼的應用程式，並由遠端資源提供給瀏覽器。 如果您正在規劃 SPA 架構，沒有後端元件或想要透過 JavaScript 叫用 Web API，則建議使用隱含流程來取得權杖。

如果應用程式是原生用戶端，則不適用隱含流程。 原生用戶端環境中沒有Azure AD 工作階段 Cookie，將會讓應用程式沒有辦法維持長時間執行的工作階段。 這表示應用程式在取得新資源的存取權杖時會重複提示使用者。

如果您要開發含有後端的 Web 應用程式，並從它的後端程式碼取用 API，則也不適用隱含流程。 其他授與可提供您更多權力。 例如，OAuth2 用戶端認證授與可讓您取得權杖以反映指派給應用程式本身 (而不是使用者委派) 的權限。 這表示用戶端可在使用者不積極參與工作階段的情況下也能維護資源的程式設計存取等等。 不只如此，這類授與也能提供較高的安全性保證。 例如，存取權杖永遠不會透過使用者瀏覽器來傳輸，它們不會因為儲存在瀏覽器歷程記錄中而發生危險等等。 用戶端應用程式也可以在要求權杖時執行增強式驗證。

## <a name="next-steps"></a>後續步驟

* 如需完整的開發人員資源清單 (包括 Azure AD 所支援的通訊協定和 OAuth2 授權授與流程的參考資訊)，請參閱 [Azure AD 開發人員指南][AAD-Developers-Guide]
* 如需更加深入地了解應用程式整合程序，請參閱[如何將應用程式與 Azure AD 整合][ACOM-How-To-Integrate]。

<!--Image references-->

<!--Reference style links in use-->
[AAD-Developers-Guide]:azure-ad-developers-guide.md
[ACOM-How-And-Why-Apps-Added-To-AAD]: active-directory-how-applications-are-added.md
[ACOM-How-To-Integrate]: active-directory-how-to-integrate.md
[OAuth2-Spec-Implicit-Misuse]: https://tools.ietf.org/html/rfc6749#section-10.16
[OAuth2-Threat-Model-And-Security-Implications]: https://tools.ietf.org/html/rfc6819
