---
title: 開始在 Visual Studio WebApi 專案中使用 Azure AD
description: 使用 Visual Studio 已連接服務連接 Azure AD 或建立 Azure AD 後，如何在 WebApi 專案中開始使用 Azure Active Directory
services: active-directory
author: ghogen
manager: douge
ms.assetid: bf1eb32d-25cd-4abf-8679-2ead299fedaa
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: aaddev, vs-azure
ms.openlocfilehash: 69f17b366b2480b34873d8fede223715619d0e66
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140076"
---
# <a name="get-started-with-azure-active-directory-webapi-projects"></a>開始使用 Azure Active Directory (WebApi 專案)

> [!div class="op_single_selector"]
> - [開始使用](vs-active-directory-webapi-getting-started.md)
> - [發生什麼情形](vs-active-directory-webapi-what-happened.md)

本文提供您在透過 Visual Studio 的 [專案] > [連線服務] 命令將 Active Directory 新增至 ASP.NET WebAPI 專案之後所需的其他指引。 如果您尚未將服務新增至專案，您可以隨時執行此動作。

請參閱[我的 WebApI 專案發生什麼情形？](vs-active-directory-webapi-what-happened.md)，了解您的專案在新增連線服務時所做的變更。

## <a name="requiring-authentication-to-access-controllers"></a>存取控制器之前需要驗證

專案中的所有控制器都會加上 `[Authorize]` 屬性作為裝飾。 此屬性要求使用者必須經過驗證，才能存取這些控制器所定義的 API。 若要允許以匿名方式存取控制器，請從控制器中移除此屬性。 如果您要以更精確地設定權限，請將此屬性套用至每一個需要授權的方法，而非套用至控制器類別。

## <a name="next-steps"></a>後續步驟

- [Azure Active Directory 的驗證案例](authentication-scenarios.md)
- [將「使用 Microsoft 登入」新增至 ASP.NET Web 應用程式](quickstart-v1-aspnet-webapp.md)
