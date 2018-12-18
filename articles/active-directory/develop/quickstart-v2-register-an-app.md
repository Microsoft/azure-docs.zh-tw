---
title: 向 Azure AD v2.0 端點註冊應用程式 | Microsoft Docs
description: 了解如何向 Microsoft 註冊應用程式，以使用 Azure AD v2.0 端點啟用登入及存取 Microsoft 服務。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: bb2f701f-3bc3-4759-94a5-8b9d53a8a0b6
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: lenalepa
ms.custom: aaddev
ms.openlocfilehash: b2dea11b6573be8f574bd18fa69ee76658d6d698
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46975805"
---
# <a name="quickstart-register-an-app-with-the-azure-active-directory-v20-endpoint"></a>快速入門：向 Azure Active Directory v2.0 端點註冊應用程式

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

若要建置同時接受個人 Microsoft 帳戶 (MSA) 與公司或學校帳戶 (Azure AD) 登入的應用程式，您必須向 Azure Active Directory (Azure AD) v2.0 端點註冊該應用程式。 您目前無法使用任何現有的 app 搭配 Azure AD 或 MSA - 您需要建立一個全新的 app。

> [!NOTE]
> v2.0 端點並未支援每個 Azure AD 案例和功能。 若要判斷是否應該使用 v2.0 端點，請閱讀 [v2.0 限制](active-directory-v2-limitations.md)。

## <a name="step-1-sign-in-to-the-microsoft-application-registration-portal"></a>步驟 1：登入 Microsoft 應用程式註冊入口網站

1. 巡覽至 Microsoft 應用程式註冊入口網站，網址為 [https://apps.dev.microsoft.com/](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList)。
1. 使用個人或公司或學校的 Microsoft 帳戶進行登入。 如果您沒有任何帳戶，請註冊新的個人帳戶。
1. 完成了嗎？ 您現在應該看一下您的 Microsoft 應用程式清單，該清單有可能空白。 讓我們改變這點。

## <a name="step-2-register-an-app"></a>步驟 2：註冊應用程式

1. 選取 [新增應用程式]，並為它命名。
    入口網站將會指派全域唯一的應用程式識別碼給您的應用程式，以便稍後在您的程式碼中使用。 如果您的應用程式包含的伺服器端元件需要用來呼叫 API (例如：Office、Azure 或您自己的 Web API) 的存取權杖，您也會想在此處建立**應用程式密碼**。
1. 接下來，新增您的應用程式將使用的**平台**。
    * 針對 Web 架構的應用程式，提供可傳送登入訊息的**重新導向 URI**。
    * 針對行動應用程式，複製系統自動為您建立的預設重新導向 URI。
    * 針對 Web API，系統會自動為您建立用來存取 Web API 的預設範圍。
        您可以使用 [新增範圍] 按鈕新增其他範圍。 您也可以使用 [已預先授權應用程式] 表單新增任何已預先獲得授權可以使用您 Web API 的應用程式。
1. 您可以選擇性地在 [設定檔] 區段中自訂登入頁面的外觀及操作方式。 
1. 在繼續之前，請先 [儲存] 您的變更。

> [!NOTE]
> 當您使用 [https://apps.dev.microsoft.com/](https://apps.dev.microsoft.com/?referrer=https://azure.microsoft.com/documentation/articles&deeplink=/appList) 註冊應用程式時，系統會將該應用程式註冊於您用來登入入口網站之帳戶的主租用戶中。 這表示您不能使用個人的 Microsoft 帳戶，在 Azure AD 租用戶中註冊應用程式。 如果您明確地想要在特定的租用戶中註冊應用程式，請使用原本在該租用戶中建立的帳戶登入。

## <a name="next-steps"></a>後續步驟

您現在已有 Microsoft 應用程式，您可以繼續完成任何一個 v2.0 快速入門。 以下是一些建議：

[!INCLUDE [active-directory-v2-quickstart-table](../../../includes/active-directory-v2-quickstart-table.md)]
