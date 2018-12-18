---
title: Azure Active Directory 同意架構
description: 了解 Azure Active Directory 的同意架構，以及如何輕鬆地開發多租用戶的 Web 應用程式和原生用戶端應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: zachowd, lenalepa, jesakowi
ms.custom: aaddev
ms.openlocfilehash: b01cd43d1c38a9db50937a6cff9b416b8c6acd0e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46946471"
---
# <a name="azure-active-directory-consent-framework"></a>Azure Active Directory 同意架構

Azure Active Directory (Azure AD) 同意架構可讓您輕鬆地開發多租用戶的 Web 應用程式和原生用戶端應用程式。 這些應用程式允許與應用程式註冊所在租用戶不同的 Azure AD 租用戶中的使用者帳戶登入。 這些應用程式除了存取自己的 Web API 之外，可能也需要存取 Microsoft Graph API 之類的 Web API (用以存取 Azure AD、Intune 及 Office 365 服務) 以及其他 Microsoft 服務的 API。

此架構以使用者或系統管理員為根據，他們可同意讓應用程式在他們的目錄中註冊，並可能包括存取目錄資料。 例如，如果 Web 用戶端應用程式需要從 Office 365 讀取有關使用者的行事曆資訊，該使用者必須先同意此用戶端應用程式。 取得同意之後，用戶端應用程式可以代表使用者呼叫 Microsoft Graph API，並視需要使用行事曆資訊。 [Microsoft Graph API](https://graph.microsoft.io) 可供存取 Office 365 中的資料 (例如 Exchange 中的行事曆和訊息、SharePoint 中的網站和清單、OneDrive 中的文件、OneNote 中的筆記本、Planner 中的工作，以及 Excel 中的活頁簿)，以及 Azure AD 中的使用者和群組和多項 Microsoft 雲端服務中的其他資料物件。

同意架構建置在 OAuth 2.0 和各種不同流程上，例如授權碼授與和用戶端認證授與，使用公用或機密的用戶端。 藉由使用 OAuth 2.0，Azure AD 就可以建置許多不同類型的用戶端應用程式 (例如在電話、平板電腦、伺服器或 web 應用程式上)，並且存取所需的資源。

如需搭配 OAuth2.0 授權授與使用同意架構的詳細資訊，請參閱[使用 OAuth 2.0 和 Azure AD 授權存取 Web 應用程式](v1-protocols-oauth-code.md)和 [Azure AD 的驗證案例](authentication-scenarios.md)。 如需透過 Microsoft Graph 取得 Office 365 的授權存取權的相關資訊，請參閱[使用 Microsoft Graph 進行應用程式驗證](https://graph.microsoft.io/docs/authorization/auth_overview)。

## <a name="consent-experience---an-example"></a>同意體驗 - 範例

下列步驟會示範如何將同意體驗用於應用程式開發人員和使用者。

1. 假設您擁有一個 Web 用戶端應用程式，它需要要求特定權限以存取資源/API。 您會在下一節了解如何進行這項設定，但基本上會使用 Azure 入口網站在設定階段宣告權限要求。 就和其他組態設定一樣，它們會成為應用程式之 Azure AD 註冊的一部分：

  ![其他應用程式的權限](./media/quickstart-v1-integrate-apps-with-azure-ad/requiredpermissions.png)

1. 請考量您的應用程式權限是否已更新、是否正在執行應用程式，以及使用者是否即將第一次使用它。 首先，應用程式必須從 Azure AD 的 `/authorize` 端點取得授權碼。 授權碼接著可用來取得新的存取權和重新整理權杖。

1. 如果使用者尚未經過驗證，Azure AD 的 `/authorize` 端點會提示使用者進行登入。

  ![使用者或系統管理員登入 Azure AD](./media/quickstart-v1-integrate-apps-with-azure-ad/usersignin.png)

1. 使用者登入之後，Azure AD 會判斷是否需要向使用者顯示同意頁面。 此判斷根據使用者 (或其組織的系統管理員) 是否已經同意應用程式。 如果尚未授與同意，Azure AD 會提示使用者取得同意，並顯示其運作所需的必要權限。 同意對話方塊中顯示的權限集，會和在 Azure 入口網站 [委派的權限] 中選取的權限相符。

  ![使用者同意體驗](./media/quickstart-v1-integrate-apps-with-azure-ad/consent.png)

1. 使用者同意後，授權碼會傳回您的應用程式，藉以兌換取得存取權杖和重新整理權杖。 如需此流程的詳細資訊，請參閱 [Web API 應用程式類型](web-api.md)。

1. 身為系統管理員，您也可以代表租用戶中的所有使用者，同意應用程式的委派權限。 系統管理員同意可避免對租用戶的每個使用者都顯示同意對話方塊，並且可由具有系統管理員角色的使用者在 [Azure 入口網站](https://portal.azure.com)中完成。 在應用程式的 [設定] 頁面中，選取 [所需的權限]，然後按一下 [授與權限] 按鈕。

  ![授與明確的系統管理員同意權限](./media/quickstart-v1-integrate-apps-with-azure-ad/grantpermissions.png)

  > [!IMPORTANT]
  > 使用 ADAL.js 的單一頁面應用程式 (SPA) 目前必須使用 [授與權限] 按鈕來授與明確的同意。 否則，應用程式會在要求存取權杖時失敗。

## <a name="next-steps"></a>後續步驟

* 請參閱[如何將應用程式轉換成多租用戶](howto-convert-app-to-be-multi-tenant.md)
* 深入了解[授權碼授與流程期間 OAuth 2.0 通訊協定層如何支援同意](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code#request-an-authorization-code)。
