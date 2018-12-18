---
title: v1.0 中的應用程式類型 | Azure
description: 說明 Azure Active Directory v2.0 端點所支援的應用程式類型與案例。
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
ms.openlocfilehash: 5a8e43934393c2e6a97b81809284781a5c207e41
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48816339"
---
# <a name="application-types-in-v10"></a>v1.0 中的應用程式類型

Azure Active Directory (Azure AD) 支援各種新型應用程式架構的驗證，它們全都以產業標準通訊協定 OAuth 2.0 或 OpenID Connect 為基準。

下圖說明這些案例和應用程式類型，以及如何新增不同的元件：

![應用程式類型和案例](./media/authentication-scenarios/application_types_and_scenarios.png)

以下是 Azure AD 支援的五個主要應用程式案例：

- **[單頁應用程式 (SPA)](single-page-application.md)**：使用者必須登入 Azure AD 所保護的單頁應用程式。
- **[網頁瀏覽器到 Web 應用程式](web-app.md)**：使用者必須登入 Azure AD 所保護的 Web 應用程式。
- **[原生應用程式到 Web API](native-app.md)**：在手機、平板電腦或電腦上執行的原生應用程式必須驗證使用者，才能從 Azure AD 所保護的 Web API 取得資源。
- **[Web 應用程式到 Web API](web-api.md)**：Web 應用程式必須從 Azure AD 所保護的 Web API 取得資源。
- **[精靈或伺服器應用程式到 Web API](service-to-service.md)**：無 Web 使用者介面的精靈應用程式或伺服器應用程式，必須從 Azure AD 所保護的 Web API 取得資源。

在開始使用程式碼之前，請參考相關連結以深入了解各種類型的應用程式，並了解概要的案例。 您也可以了解在撰寫與 v1.0 端點或 v2.0 端點搭配運作的特定應用程式時所需了解的差異。

> [!NOTE]
> v2.0 端點並未支援所有的 Azure AD 案例和功能。 若要判斷您是否應該使用 v2.0 端點，請參閱 [v2.0 限制](active-directory-v2-limitations.md)。

您可以使用多種語言和平台開發本文所說明的任何應用程式和案例。 這些案例都有完整的程式碼範例，您可以在程式碼範例指南中取得：[依據案例的 v1.0 程式碼範例](sample-v1-code.md)和[依據案例的 v2.0 程式碼範例](sample-v2-code.md)。 您可以直接從對應的 [GitHub 範例存放庫](https://github.com/Azure-Samples?q=active-directory)中下載程式碼範例。

此外，如果您的應用程式需要端對端案例的特定片段或區段，在大部分情況下可以獨立加入這項功能。 例如，如果您有一個呼叫 Web API 的原生應用程式，您可以輕鬆加入也會呼叫該 Web API 的 Web 應用程式。

## <a name="app-registration"></a>應用程式註冊

### <a name="registering-an-app-that-uses-the-azure-ad-v10-endpoint"></a>註冊使用 Azure AD v1.0 端點的應用程式

將驗證外包給 Azure AD 的任何應用程式必須在目錄中註冊。 此步驟涉及向 Azure AD 說明您的應用程式，包括它所在的 URL、驗證之後用來傳送回覆的 URL、用來識別應用程式的 URI 等。 需要這項資訊有幾個主要理由：

* Azure AD 處理登入或交換權杖時，需要此項目以與應用程式通訊。 Azure AD 和應用程式之間傳遞的資訊包括以下：
  
  * **應用程式識別碼 URI** - 應用程式的識別碼。 這個值會在驗證期間傳送至 Azure AD，以指出呼叫端想要哪一個應用程式的權杖。 此外，這個值包含在權杖中，應用程式由此可知它就是預定的目標。
  * **回覆 URL** 和**重新導向 URI** - 針對 Web API 或 Web 應用程式，Azure AD 會將驗證回應傳送到回覆 URL，包括驗證成功時的權杖。 在原生應用程式中，重新導向 URI 是唯一識別碼；Azure AD 會在 OAuth 2.0 要求中將使用者代理程式重新導向至該處。
  * **應用程式識別碼** - 註冊應用程式時由 Azure AD 產生的應用程式識別碼。 當要求授權碼或權杖時，系統會在驗證期間將應用程式識別碼和金鑰傳送至 Azure AD。
  * **金鑰** - 在進行 Azure AD 驗證以呼叫 Web API 時，隨應用程式識別碼一起傳送的金鑰。
* Azure AD 需要確定應用程式具有必要權限以存取您的目錄資料和組織中其他應用程式等等。

如需詳細資訊，請了解如何[使用 Azure AD v1.0 端點註冊應用程式](quickstart-v1-add-azure-ad-app.md)。

## <a name="single-tenant-and-multi-tenant-apps"></a>單一租用戶與多租用戶應用程式

當您了解可以開發及與 Azure AD 整合的應用程式有兩種類別時，佈建就變得更清楚：

* **單一租用戶應用程式** - 單一租用戶應用程式主要是供單一組織使用。 這些通常是由企業開發人員撰寫的企業營運 (LoB) 應用程式。 單一租用戶應用程式只需要由一個目錄中的使用者存取，因此，它只需要佈建在一個目錄中。 這些應用程式通常是由組織中的開發人員註冊。
* **多租用戶應用程式** - 多租用戶應用程式可供許多組織使用，而不限單一組織。 這些通常是由獨立軟體廠商 (ISV) 撰寫的軟體即服務 (SaaS) 應用程式。 多租用戶應用程式需要佈建在將會用到它們的每個目錄中，而這需要使用者或系統管理員同意才能註冊。 當應用程式目錄中已在目錄中註冊並獲權存取 Graph API 或其他 Web API 時，此同意程序會啟動。 當不同組織的使用者或系統管理員註冊來使用應用程式時，他們會看到一個對話方塊顯示應用程式需要的權限。 使用者或系統管理員可以同意應用程式，讓應用程式存取所述的資料，最後將應用程式註冊在他們的目錄中。 如需詳細資訊，請參閱 [同意架構的概觀](consent-framework.md)。

### <a name="additional-considerations-when-developing-single-tenant-or-multi-tenant-apps"></a>開發單一租用戶或多租用戶應用程式的其他考量

開發多租用戶應用程式，而非單一租用戶應用程式時，有一些其他考量需要注意。 例如，如果要讓您的應用程式供多個目錄中的使用者使用，您需要有機制來判斷他們所在的租用戶。 單一租用戶應用程式只需要在它自己的目錄中查看使用者，但多租用戶應用程式需要從 Azure AD 的所有目錄中識別特定的使用者。 為了完成這項工作，Azure AD 提供一個共同驗證端點，供任何多租用戶應用程式引導登入要求，而非提供租用戶特定的端點。 針對所有 Azure AD 目錄，這個端點是 https://login.microsoftonline.com/common，而租用戶專用端點可能是 https://login.microsoftonline.com/contoso.onmicrosoft.com。 開發您的應用程式時尤其必須考量共同端點，因為在登入、登出和權杖驗證期間，您需要必要的邏輯來處理多個租用戶。

如果您目前正在開發單一租用戶應用程式，但想要提供給許多組織使用，您可以在 Azure AD 中輕鬆地變更應用程式及其組態，將它變成具備多租用戶功能。 此外，不論您是在單一租用戶或多租用戶應用程式中提供驗證，Azure AD 對所有目錄中的所有權杖都使用相同的簽署金鑰。

本文件列出的每個案例都有一個小節來說明其佈建需求。 如需如何在 Azure AD 中佈建應用程式的深入資訊，並了解單一和多租用戶應用程式之間的差異，請參閱[整合應用程式與 Azure Active Directory](quickstart-v1-integrate-apps-with-azure-ad.md)，以取得詳細資訊。 繼續閱讀來了解 Azure AD 中常見的應用程式案例。

## <a name="next-steps"></a>後續步驟

- 深入了解其他 Azure AD [驗證基本概念](authentication-scenarios.md)
