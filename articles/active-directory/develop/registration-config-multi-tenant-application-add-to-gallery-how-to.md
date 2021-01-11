---
title: 將多租用戶應用程式新增至 Azure AD 應用程式庫
description: 說明如何在 Azure AD 應用程式庫中列出自訂開發的多租用戶應用程式。
services: active-directory
documentationCenter: na
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.reviewer: jeedes
ROBOTS: NOINDEX
ms.openlocfilehash: de0231a49c4f806660ea0cb305fdc1a70e2b36d3
ms.sourcegitcommit: 2488894b8ece49d493399d2ed7c98d29b53a5599
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/11/2021
ms.locfileid: "98063122"
---
# <a name="add-a-multitenant-application-to-the-azure-ad-application-gallery"></a>將多租用戶應用程式新增至 Azure AD 應用程式庫

## <a name="what-is-the-azure-ad-application-gallery"></a>什麼是 Azure AD 應用程式庫？

Azure Active Directory (Azure AD) 是雲端式識別服務。 [Azure AD 應用程式庫](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory)位於 Azure Marketplace 應用程式存放區，所有應用程式連接器都會在此處發佈以供進行單一登入和使用者佈建。 使用 Azure AD 作為識別提供者的客戶可找到在此發佈的不同 SaaS 應用程式連接器。 IT 系統管理員可從應用程式庫新增連接器，然後設定並使用這些連接器進行單一登入和佈建。 Azure AD 支援以所有主要的同盟通訊協定進行單一登入，包括 SAML 2.0、OpenID Connect、OAuth 及 WS-Fed。 

## <a name="if-your-application-supports-saml-or-openidconnect"></a>如果您的應用程式支援 SAML 或 OpenIDConnect
如果您有想要列在 Azure AD 應用程式庫中的多租用戶應用程式，您必須先確定該應用程式支援下列其中一種單一登入技術：

- **OpenID Connect**：若要列出您的應用程式，請在 Azure AD 中建立多租用戶應用程式，然後為您的應用程式實作 [Azure AD 同意架構](./consent-framework.md)。 將登入要求傳送給通用端點，以便讓所有客戶都可以向該應用程式表示同意。 您可以根據租用戶識別碼和在權杖中收到的使用者 UPN 來控制使用者存取權。 使用[在 Azure Active Directory 應用程式庫中列出您的應用程式](./v2-howto-app-gallery-listing.md)中所述的程序，提交此應用程式。

- **SAML**：如果您的應用程式支援 SAML 2.0，此應用程式即可列在資源庫中。 依照[在 Azure Active Directory 應用程式庫中列出您的應用程式](./v2-howto-app-gallery-listing.md)中的指示操作。

## <a name="if-your-application-does-not-support-saml-or-openidconnect"></a>如果您的應用程式不支援 SAML 或 OpenIDConnect
不支援 SAML 或 OpenIDConnect 的應用程式仍可透過密碼單一登入技術整合到應用程式庫中。

密碼單一登入也稱為密碼儲存庫存，可讓您管理使用者對不支援身分識別同盟之 Web 應用程式的存取和密碼。 如果有數個使用者需要共用單一帳戶 (例如共用組織的社交媒體應用程式帳戶)，這也很有用處。 

如果您想要利用這項技術列出您的應用程式：
1. 建立一個具有可設定[密碼單一登入](../manage-apps/what-is-single-sign-on.md)之 HTML 登入頁面的 Web 應用程式。 
2. 如[在 Azure Active Directory 應用程式庫中列出您的應用程式](./v2-howto-app-gallery-listing.md)所述提交要求。

## <a name="escalations"></a>呈報

若要呈報任何事項，請傳送電子郵件至 [Azure AD SSO 整合小組](<mailto:SaaSApplicationIntegrations@service.microsoft.com>)，我們會儘快回覆。

## <a name="next-steps"></a>後續步驟
了解如何[在 Azure Active Directory 應用程式庫中列出您的應用程式](./v2-howto-app-gallery-listing.md)。