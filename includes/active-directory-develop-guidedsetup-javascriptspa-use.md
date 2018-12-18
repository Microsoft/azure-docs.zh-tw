---
title: 包含檔案
description: 包含檔案
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: mtillman
editor: ''
ms.service: active-directory
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/17/2018
ms.author: nacanuma
ms.custom: include file
ms.openlocfilehash: 77400453e455ff2ebf20f59f888a3e3d641bcf07
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48843113"
---
## <a name="use-the-microsoft-authentication-library-msal-to-sign-in-the-user"></a>使用 Microsoft Authentication Library (MSAL) 登入使用者

1. 將下列程式碼新增至 `index.html` 檔案的 `<script></script>` 標記內：

```javascript
//Pass null for default authority (https://login.microsoftonline.com/common)
var myMSALObj = new Msal.UserAgentApplication(applicationConfig.clientID, null, acquireTokenRedirectCallBack,
    {storeAuthStateInCookie: true, cacheLocation: "localStorage"});

function signIn() {
    myMSALObj.loginPopup(applicationConfig.graphScopes).then(function (idToken) {
        //Login Success
        showWelcomeMessage();
        acquireTokenPopupAndCallMSGraph();
    }, function (error) {
        console.log(error);
    });
}

function acquireTokenPopupAndCallMSGraph() {
    //Call acquireTokenSilent (iframe) to obtain a token for Microsoft Graph
    myMSALObj.acquireTokenSilent(applicationConfig.graphScopes).then(function (accessToken) {
        callMSGraph(applicationConfig.graphEndpoint, accessToken, graphAPICallback);
    }, function (error) {
        console.log(error);
        // Call acquireTokenPopup (popup window) in case of acquireTokenSilent failure due to consent or interaction required ONLY
        if (error.indexOf("consent_required") !== -1 || error.indexOf("interaction_required") !== -1 || error.indexOf("login_required") !== -1) {
            myMSALObj.acquireTokenPopup(applicationConfig.graphScopes).then(function (accessToken) {
                callMSGraph(applicationConfig.graphEndpoint, accessToken, graphAPICallback);
            }, function (error) {
                console.log(error);
            });
        }
    });
}

function graphAPICallback(data) {
    //Display user data on DOM
    var divWelcome = document.getElementById('WelcomeMessage');
    divWelcome.innerHTML += " to Microsoft Graph API!!";
    document.getElementById("json").innerHTML = JSON.stringify(data, null, 2);
}

function showWelcomeMessage() {
    var divWelcome = document.getElementById('WelcomeMessage');
    divWelcome.innerHTML += 'Welcome ' + myMSALObj.getUser().name;
    var loginbutton = document.getElementById('SignIn');
    loginbutton.innerHTML = 'Sign Out';
    loginbutton.setAttribute('onclick', 'signOut();');
}

// This function can be removed if you do not need to support IE
function acquireTokenRedirectAndCallMSGraph() {
    //Call acquireTokenSilent (iframe) to obtain a token for Microsoft Graph
    myMSALObj.acquireTokenSilent(applicationConfig.graphScopes).then(function (accessToken) {
      callMSGraph(applicationConfig.graphEndpoint, accessToken, graphAPICallback);
    }, function (error) {
        console.log(error);
        //Call acquireTokenRedirect in case of acquireToken Failure
        if (error.indexOf("consent_required") !== -1 || error.indexOf("interaction_required") !== -1 || error.indexOf("login_required") !== -1) {
            myMSALObj.acquireTokenRedirect(applicationConfig.graphScopes);
        }
    });
}

function acquireTokenRedirectCallBack(errorDesc, token, error, tokenType)
{
 if(tokenType === "access_token")
 {
     callMSGraph(applicationConfig.graphEndpoint, accessToken, graphAPICallback);
 } else {
     console.log("token type is:"+tokenType);
 }
}


// Browser check variables
var ua = window.navigator.userAgent;
var msie = ua.indexOf('MSIE ');
var msie11 = ua.indexOf('Trident/');
var msedge = ua.indexOf('Edge/');
var isIE = msie > 0 || msie11 > 0;
var isEdge = msedge > 0;

//If you support IE, our recommendation is that you sign-in using Redirect APIs
//If you as a developer are testing using Edge InPrivate mode, please add "isEdge" to the if check
if (!isIE) {
    if (myMSALObj.getUser()) {// avoid duplicate code execution on page load in case of iframe and popup window.
        showWelcomeMessage();
        acquireTokenPopupAndCallMSGraph();
    }
}
else {
    document.getElementById("SignIn").onclick = function () {
        myMSALObj.loginRedirect(applicationConfig.graphScopes);
    };
    if (myMSALObj.getUser() && !myMSALObj.isCallback(window.location.hash)) {// avoid duplicate code execution on page load in case of iframe and popup window.
        showWelcomeMessage();
        acquireTokenRedirectAndCallMSGraph();
    }
}
```

<!--start-collapse-->
### <a name="more-information"></a>相關資訊

在使用者第一次按一下 [登入] 按鈕之後，`signIn` 方法會呼叫 `loginPopup` 以登入使用者。 這個方法會導致 Microsoft Azure Active Directory v2 端點開啟快顯視窗，以提示及驗證使用者的認證。 如果成功登入，使用者會被重新導向回到原始 index.html 分頁，並且會收到權杖，由 `msal.js` 處理，並且會快取權杖中包含的資訊。 此權杖也稱為「ID 權杖」且包含使用者的基本資訊，例如使用者顯示名稱。 如果您打算將此權杖所提供的任何資料用於任何目的，您必須確定後端伺服器已驗證此權杖，以保證權杖是發給您應用程式的有效使用者。

本指南所產生的 SPA 會呼叫 `acquireTokenSilent` 和/或 `acquireTokenPopup`，以取得用來查詢 Microsoft Graph API 中使用者設定檔資訊的「存取權杖」。 如果您需要可驗證 ID 權杖的範例，請看一下 GitHub 中的 [這個](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2 "Github active-directory-javascript-singlepageapp-dotnet-webapi-v2 範例")範例應用程式 – 此範例使用 ASP.NET Web API 進行權杖驗證。

#### <a name="getting-a-user-token-interactively"></a>以互動方式取得使用者權杖

初始登入之後，您不希望在使用者每次必須要求權杖來存取資源時，都要求使用者重新驗證，因此，通常應該使用 *acquireTokenSilent* 以取得權杖。 然而，在某些情況下您需要強制使用者來與 Azure Active Directory v2 端點互動，部分範例包括：
- 使用者可能需要重新輸入其認證，因為密碼已過期
- 您的應用程式要求的資源存取權，需要使用者同意
- 需要雙因素驗證

呼叫 *acquireTokenPopup(scope)* 會導致快顯示窗 (或者呼叫 *acquireTokenRedirect(scope)* 會導致將使用者重新導向至 Azure Active Directory v2 端點)，使用者必須藉由確認其認證、同意必要的資源，或完成雙因素驗證來進行互動。

#### <a name="getting-a-user-token-silently"></a>以無訊息方式取得使用者權杖
` acquireTokenSilent` 方法會處理權杖取得和更新作業，不需要與使用者進行任何互動。 `loginPopup` (或 `loginRedirect`) 在第一次執行之後，`acquireTokenSilent` 就會成為用來取得權杖的常用方法，以在後續呼叫中使用那些權杖存取受保護的資源，並且會以無訊息方式進行要求或更新權杖的呼叫。
`acquireTokenSilent` 在某些情況下可能會失敗 – 例如，使用者的密碼已過期。 您的應用程式可以透過兩種方式處理此例外狀況：

1.  立即對 `acquireTokenPopup` 進行呼叫，這會促使系統提示使用者登入。 此模式通常用於應用程式中沒有未經驗證內容可供使用者使用的線上應用程式。 此引導式設定所產生的範例應用程式會使用此模式。

2. 應用程式也可以提供視覺指示，讓使用者知道需要透過互動方式登入，使用者就能選取正確的登入時機，或應用程式可以在之後重試 `acquireTokenSilent`。 此方式通常用於使用者可以使用應用程式的其他功能，不需要因此中斷作業的情況，例如，應用程式中有未經驗證內容可供使用者使用。 在此情況下，使用者可以決定何時登入以存取受保護的資源，或者重新整理過期的資訊。

> [!NOTE]
> 如果因為 Internet Explorer 瀏覽器處理快顯視窗相關[已知問題](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser)，而使用 Internet Explorer 瀏覽器，則上述程式碼會使用 `loginRedirect` 和 `acquireTokenRedirect` 方法。
<!--end-collapse-->

## <a name="call-the-microsoft-graph-api-using-the-token-you-just-obtained"></a>使用您剛剛取得的權杖呼叫 Microsoft 圖形 API

將下列程式碼新增至 `index.html` 檔案的 `<script></script>` 標記內：

```javascript
function callMSGraph(theUrl, accessToken, callback) {
    var xmlHttp = new XMLHttpRequest();
    xmlHttp.onreadystatechange = function () {
        if (this.readyState == 4 && this.status == 200)
            callback(JSON.parse(this.responseText));
    }
    xmlHttp.open("GET", theUrl, true); // true for asynchronous
    xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
    xmlHttp.send();
}
```
<!--start-collapse-->

### <a name="more-information-on-making-a-rest-call-against-a-protected-api"></a>針對受保護 API 進行 REST 呼叫的詳細資訊

在本指南建立的範例應用程式中，`callMSGraph()` 方法是用來針對受保護資源提出 HTTP `GET` 要求，那些受保護資源需要權杖才能存取，然後再將內容傳回給使用者。 此方法會在「HTTP 授權標頭」中加入取得的權杖。 對於本指南建立的範例應用程式，資源為 Microsoft 圖形 API *me* 端點，它會顯示使用者的設定檔資訊。

<!--end-collapse-->

## <a name="add-a-method-to-sign-out-the-user"></a>新增方法來將使用者登出

將下列程式碼新增至 `index.html` 檔案的 `<script></script>` 標記內：

```javascript
/**
 * Sign out the user
 */
 function signOut() {
     myMSALObj.logout();
 }
```
