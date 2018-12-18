---
title: Azure Active Directory 中的 Web 應用程式
description: 說明何謂 Web 應用程式，以及此應用程式類型的通訊協定流程、註冊與權杖到期等基本概念。
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
ms.openlocfilehash: 0eec1100c5246bb9f5c281971453ac60abc0339f
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225012"
---
# <a name="web-apps"></a>Web 應用程式

Web 應用程式是向 Web 應用程式驗證 Web 瀏覽器中使用者的應用程式。 在此案例中，Web 應用程式會引導使用者的瀏覽器，將他們登入 Azure AD。 Azure AD 會透過使用者的瀏覽器傳回登入回應，其中包含關於安全性權杖中的使用者宣告。 此案例支援使用 OpenID Connect、SAML 2.0 和 WS-Federation 通訊協定來登入。

## <a name="diagram"></a>圖表

![瀏覽器到 Web 應用程式的驗證流程](./media/authentication-scenarios/web_browser_to_web_api.png)

## <a name="protocol-flow"></a>通訊協定流程

1. 當使用者造訪應用程式且需要登入時，他們透過登入要求而重新導向 Azure AD 中的驗證端點。
1. 使用者在登入頁面上登入。
1. 如果驗證成功，Azure AD 會建立驗證權杖，並將登入回應傳回到 Azure 入口網站中所設的應用程式回覆 URL。 對於實際執行應用程式，此回覆 URL 應該為 HTTPS。 傳回的權杖包含應用程式驗證權杖所需的使用者與 Azure AD 宣告。
1. 應用程式會使用 Azure AD 的同盟中繼資料文件可用的公開簽署金鑰和簽發者資訊來驗證權杖。 應用程式會在驗證權杖之後，對使用者啟動新的工作階段。 此工作階段可讓使用者存取應用程式，直到過期為止。

## <a name="code-samples"></a>程式碼範例

請參閱「Web 瀏覽器到 Web 應用程式」案例的程式碼範例。 歡迎您經常回來看看，因為我們會隨時新增新的範例。

## <a name="app-registration"></a>應用程式註冊

若要註冊 Web 應用程式，請參閱[向 Azure AD v1.0 端點註冊應用程式](quickstart-v1-add-azure-ad-app.md)。

* 單一租用戶 - 如果您只要為組織建置一個應用程式，則必須使用 Azure 入口網站，將它註冊在公司的目錄中。
* 多租用戶 - 如果您要建置的應用程式可供組織外的使用者使用，它必須註冊在公司的目錄中，但也必須在每個將會使用該應用程式的組織的目錄中註冊。 若要讓您的應用程式出現在他們的目錄中，您可以為客戶加上註冊程序，讓他們同意應用程式。 當他們註冊您的應用程式時，他們會看到對話方塊顯示應用程式所需的權限，以及是否同意的選項。 根據所需的權限，其他組織的系統管理員可能必須同意。 當使用者或系統管理員同意時，應用程式就會註冊在他們的目錄中。

## <a name="token-expiration"></a>權杖到期

當 Azure AD 所簽發的權杖存留期到期時，使用者工作階段就過期。 如有需要，您的應用程式可以縮短這段時間，例如因為一段時間沒有活動而將使用者登出。 當工作階段到期時，會提示使用者重新登入。

## <a name="next-steps"></a>後續步驟

* 深入了解其他[應用程式類型和案例](app-types.md)
* 了解 Azure AD [驗證基本概念](authentication-scenarios.md)