---
title: 註冊呼叫 web Api 的 web API |蔚藍
titleSuffix: Microsoft identity platform
description: 瞭解如何建立 web API，以呼叫 (應用程式註冊) 的下游 web Api。
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 8349b7a53eb7b03b27c695bb24c8cb9bc665a1ac
ms.sourcegitcommit: 5cdd0b378d6377b98af71ec8e886098a504f7c33
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/25/2021
ms.locfileid: "98756352"
---
# <a name="a-web-api-that-calls-web-apis-app-registration"></a>呼叫 web Api 的 web API：應用程式註冊

呼叫下游 web Api 的 web API 與受保護的 web API 具有相同的註冊。 因此，您必須依照 [受保護的 WEB API：應用程式註冊](scenario-protected-web-api-app-registration.md)中的指示進行。

由於 web 應用程式現在會呼叫 web Api，因此它會成為機密用戶端應用程式。 這就是為什麼需要額外的註冊資訊：應用程式需要與 Microsoft 身分識別平臺) 共用 (用戶端認證的秘密。

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-registration-client-secrets.md)]

## <a name="api-permissions"></a>API 權限

Web apps 會代表已收到持有人權杖的使用者呼叫 Api。 Web 應用程式需要要求委派的許可權。 如需詳細資訊，請參閱 [新增存取 WEB API 的許可權](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api)。

## <a name="next-steps"></a>後續步驟

請繼續進行此案例中的下一篇文章，也就是 [應用程式程式碼](scenario-web-api-call-api-app-configuration.md)設定。
