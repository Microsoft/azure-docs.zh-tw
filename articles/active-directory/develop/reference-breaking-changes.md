---
title: Azure Active Directory 中斷性變更參考 | Microsoft Docs
description: 對 Azure AD 通訊協定所做的變更可能會影響您的應用程式。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 68517c83-1279-4cc7-a7c1-c7ccc3dbe146
ms.service: active-directory
ms.component: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/31/2018
ms.author: celested
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 14217d03cdb56c5c641ab8f8c3fc0748e8e815a2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987749"
---
# <a name="whats-new-for-authentication"></a>驗證有什麼新功能？ 

|
>藉由將此 [URL](https://docs.microsoft.com/api/search/rss?search=%22whats%20new%20for%20authentication%22&locale=en-us) 新增至您的 RSS 摘要讀取程式，即可獲知何時要再造訪此頁面來進行更新。

驗證系統會持續改變並且新增功能，以改善安全性及改善標準合規性。 為了讓您隨時掌握最新的開發訊息，本文提供下列相關資訊：

- 最新的功能
- 已知問題
- 通訊協定變更
- 已被取代的功能

> [!TIP] 
> 此頁面會定期更新，因此請時常瀏覽。 除非另有註明，否則這些變更僅會針對新註冊的應用程式施行。  

## <a name="upcoming-changes"></a>即將推出的變更

### <a name="authorization-codes-can-no-longer-be-reused"></a>授權碼再也無法重複使用

**生效日期**：2018 年 10 月 10 日**受影響的端點**：1.0 版與 2.0 版**受影響的通訊協定**：[程式碼流程](v2-oauth2-auth-code-flow.md)

從 2018 年 10 月 10 日開始，Azure AD 將不再接受將先前使用的驗證碼用於新的應用程式。 在 2018 年 10 月 10 日之前建立的任何應用程式，仍然可以重複使用驗證碼。 這項安全性變更有助於讓 Azure AD 與 OAuth 規格一致，且將會對 v1 和 v2 端點強制執行。

如果您的應用程式重複使用授權碼取得多個資源的權杖，建議您先使用授權碼取得重新整理權杖，再使用該重新整理權杖取得其他資源的額外權杖。 授權碼只能使用一次，但重新整理權杖可跨多個資源多次使用。 在 OAuth 程式碼流程期間嘗試重複使用驗證碼的任何新應用程式，會收到 invalid_grant 錯誤。

如需關於重新整理權杖的詳細資訊，請參閱[重新整理存取權杖](v1-protocols-oauth-code.md#refreshing-the-access-tokens)。

## <a name="may-2018"></a>2018 年 5 月

### <a name="id-tokens-cannot-be-used-for-the-obo-flow"></a>識別碼權杖無法用於 OBO 流程

**日期**：2018 年 5 月 1 日**受影響的端點**：1.0 版與 2.0 版**受影響的通訊協定**：隱含流程和 [OBO 流程](v1-oauth2-on-behalf-of-flow.md)

在 2018 年 5 月 1 日以後，id_tokens 無法用來作為新應用程式 OBO 流程中的判斷提示。  應該改為使用存取權杖來保護 API，即使是在相同應用程式的用戶層與中介層之間也是一樣。  在 2018 年 5 月 1 日以前註冊的應用程式會持續運作，並且可以用 id_tokens 交換存取權杖 - 但是，這並不是最佳做法。

為了因應這項變更，您可以執行下列作業：

1. 為您的中介層應用程式建立 Web API，具有一或多個範圍。  這樣可以獲得更精細的控制權和安全性。
1. 在您的應用程式資訊清單、[Azure 入口網站](https://portal.azure.com)或[應用程式註冊入口網站](https://apps.dev.microsoft.com)中，確保應用程式可以透過隱含流程發出存取權杖。 這是透過 `oauth2AllowImplicitFlow` 金鑰進行控制的。
1. 當您的用戶端應用程式透過 `response_type=id_token` 要求 id_token 時，也會為上述建立的 Web API 要求存取權杖 (`response_type=token`)。  因此，當使用 v2.0 端點時，`scope` 參數看起來應該像 `api://GUID/SCOPE`。  在 v1.0 端點上，`resource` 參數應該是 Web API 的應用程式 URI。
1. 將此存取權杖傳遞至中介層以取代 id_token。  