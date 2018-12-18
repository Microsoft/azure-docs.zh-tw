---
title: 關於 v2.0 | Azure
description: 了解 v2.0 端點與平台。
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: adcb1b15ed52e6954846ea09be0a87a118222c10
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46989579"
---
# <a name="about-v20"></a>關於 v2.0

v2.0 端點與平台已在預覽中，並持續增強。 現在，JavaScript 單頁應用程式 (SPA) 案例已功能完整，我們邀請您使用 MSAL.js 建置瀏覽器型應用程式並向我們提供意見反應，以便我們可以將狀態從預覽更新為公開推出 (GA)。

> [!NOTE]
> MSAL Android、iOS 與 .NET 仍有開發中的功能。 您可以使用它們來建置應用程式，並向我們傳送意見反應。

Azure 入口網站開發人員體驗已經大幅更新，現在包含使用 ADAL 或 MSAL 建置的所有應用程式，並改善可用性。

在過去，應用程式開發人員需要同時支援個人 Microsoft 帳戶和 Azure Active Directory (Azure AD) 中的公司帳戶時，必須整合這兩個不同的系統。 v2.0 端點與平台提供了一個簡化此程序的驗證 API 版本。 它使用單一整合讓使用者可從兩種類型的帳戶登入。 使用 v2.0 端點的應用程式也可以利用這其中一種帳戶，從 [Microsoft Graph API](https://graph.microsoft.io) 取用 REST API。

## <a name="getting-started"></a>開始使用

從下列清單選擇您最愛的平台，使用 Microsoft 開放原始碼程式庫與架構來建置應用程式。 您也可以使用 OAuth 2.0 和 OpenID Connect 通訊協定直接傳送和接收通訊協定訊息，而不使用驗證程式庫。

[!INCLUDE [v2.0 endpoint platforms](../../../includes/active-directory-v2-quickstart-table.md)]

## <a name="learn-more-about-the-v20-endpoint-and-platform"></a>深入了解 v2.0 端點與平台

深入了解您可以使用 Azure AD v2.0 端點執行的內容：

* 探索[您可以使用 Azure AD v2.0 端點來建置的應用程式類型](v2-app-types.md)。
* 了解使用 Azure AD v2.0 端點的[限制和條件約束](active-directory-v2-limitations.md)。
* 如需 Azure AD v2.0 端點的概觀，請觀看這段影片：

>[!VIDEO https://channel9.msdn.com/Events/Build/2017/P4031/player]

## <a name="additional-resources"></a>其他資源

探索有關 v2.0 的深入資訊：

* [v2.0 通訊協定參考](active-directory-v2-protocols.md)
* [存取權杖參考](access-tokens.md)
* [`id_tokens` 參考](id-tokens.md)
* [v2.0 驗證程式庫參考](reference-v2-libraries.md)
* [v2.0 的範圍和同意](v2-permissions-and-consent.md)
* [Microsoft Graph API](https://graph.microsoft.io)

> [!NOTE]
> 如果您只需要從 Azure Active directory 登入公司與學校帳戶，請先閱讀 [Azure AD 開發人員指南](azure-ad-developers-guide.md)。 v2.0 端點的適用對象是明確需要登入 Microsoft 個人帳戶的開發人員。

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]
