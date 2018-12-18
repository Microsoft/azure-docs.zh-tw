---
title: Azure Active Directory B2B 共同作業的自助式註冊入口網站 | Microsoft Docs
description: Azure Active Directory B2B 共同作業讓企業合作夥伴選擇性地存取您的公司應用程式，以支援公司間的關係
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: sample
ms.date: 05/08/2018
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: cb88645530f904bc35c70dfa12b09b76fcd1eb7c
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45985255"
---
# <a name="self-service-portal-for-azure-ad-b2b-collaboration-sign-up"></a>Azure AD B2B 共同作業註冊的自助入口網站

客戶可以使用透過 [Azure 入口網站](https://portal.azure.com)公開的內建功能進行許多工作，而終端使用者，則可使用透過[應用程式存取面板](https://myapps.microsoft.com)公開的內建功能。 不過，您可能需要自訂適用於 B2B 使用者的上線工作流程，以符合貴組織的需求。 您可以利用[邀請 API](https://developer.microsoft.com/graph/docs/api-reference/v1.0/resources/invitation) 來執行該作業。

提出邀請的組織可能事先不知道需要存取其資源的外部共同作業者分別有誰。 您必須讓合作夥伴公司的使用者能夠自行註冊，並以提出邀請的組織所能控制的一組原則來加以規範。 此案例可透過 API 實現。 [GitHub 上有個範例專案](https://github.com/Azure/active-directory-dotnet-graphapi-b2bportal-web)可供您參考。

此 GitHub 專案會說明組織如何使用 API，為您信任的合作夥伴提供原則型自助式註冊功能，而且這些功能會有規則可以判斷合作夥伴能存取的應用程式。 合作夥伴使用者可以在需要時取得資源的存取權。 他們可以安全地存取資源，而不需要提出邀請的組織手動讓使用者上線。 您可以輕鬆地將專案部署到您選擇的 Azure 訂用帳戶中。

## <a name="as-is-code"></a>現狀程式碼

此程式碼只當作範例供您參考，目的是要示範如何使用 Azure Active Directory B2B 邀請 API。 您應該讓開發小組或合作夥伴自訂此程式碼，並在檢閱過後再部署到生產案例中。

## <a name="next-steps"></a>後續步驟

* [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
* [Azure AD B2B 共同作業授權](licensing-guidance.md)
* [Azure Active Directory B2B 共同作業常見問題 (FAQ)](faq.md)