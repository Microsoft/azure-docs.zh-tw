---
title: 如何為指定的 API 選取權限 | Microsoft Docs
description: 如何為針對 Azure AD 開發的應用程式或要向 Azure AD 註冊的自訂應用程式尋找驗證端點。
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
ms.openlocfilehash: f6c2540d8f0bb49491a428b085d20067a36d970a
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44723679"
---
# <a name="how-to-select-permissions-for-a-given-api"></a>如何為指定的 API 選取權限

您可以在 [Azure 入口網站](https://portal.azure.com)尋找應用程式的驗證端點。

-   瀏覽至 [Azure 入口網站](https://portal.azure.com)。

-   從左方的瀏覽窗格，按一下 [Azure Active Directory]。

-   按一下 [應用程式註冊]，然後選擇 [端點]。

-   這樣會開啟 [端點] 頁面，其中列出您租用戶的所有驗證端點。

-   使用您所使用之驗證通訊協定特定的端點搭配應用程式識別碼，來製作您應用程式的特定驗證要求。

## <a name="next-steps"></a>後續步驟
[Azure Active Directory 開發人員指南](https://docs.microsoft.com/azure/active-directory/develop/active-directory-developers-guide#authentication-and-authorization-protocols)
