---
title: 如何尋找自訂開發應用程式所需的特定 API | Microsoft Docs
description: 如何在您的自訂開發 Azure AD 應用程式中設定存取特定 API 所需的權限
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: celested
ms.openlocfilehash: 8d0b9f219d7a0bc61e3d12acfaae6015963401f9
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44723661"
---
# <a name="how-to-find-a-specific-api-needed-for-a-custom-developed-application"></a>如何尋找自訂開發應用程式所需的特定 API

存取 API 需要設定存取範圍和角色。 如果您想要將資源應用程式 Web API 公開給用戶端應用程式，您需要設定 API 的存取範圍和角色。 如果您想要讓用戶端應用程式存取 Web API，您需要設定權限以存取應用程式註冊中的 API。

## <a name="configuring-a-resource-application-to-expose-web-apis"></a>設定資源應用程式以公開 Web API

當您公開 Web API 時，該 API 會在將權限新增到應用程式註冊時顯示於 [選取 API] 清單中。 若要新增存取範圍，請依照[新增資源應用程式的存取範圍](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications#adding-access-scopes-to-your-resource-application)中概述的步驟執行。

## <a name="configuring-a-client-application-to-access-web-apis"></a>設定用戶端應用程式以存取 Web API

當您將權限新增到應用程式註冊時，您可以**新增 API 存取**以公開 Web API。 若要存取 Web API，請依照[新增認證或權限以存取 Web API](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications#to-add-credentials-or-permissions-to-access-web-apis) 中概述的步驟執行。

## <a name="next-steps"></a>後續步驟

-   [整合應用程式與 Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)

-   [了解 Azure Active Directory 應用程式資訊清單](https://docs.microsoft.com/azure/active-directory/develop/active-directory-application-manifest)


