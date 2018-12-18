---
title: Azure Active Directory 條件式存取常見問題集 | Microsoft Docs
description: 取得 Azure Active Directory 條件式存取常見問題集的解答。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
ms.assetid: 14f7fc83-f4bb-41bf-b6f1-a9bb97717c34
ms.service: active-directory
ms.component: conditional-access
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.author: markvi
ms.reviewer: calebb
ms.openlocfilehash: e9ce5e1d86f6c1756f8871dc45d60d36cea74515
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39629288"
---
# <a name="azure-active-directory-conditional-access-faqs"></a>Azure Active Directory 條件式存取常見問題集

## <a name="which-applications-work-with-conditional-access-policies"></a>哪些應用程式會使用條件式存取原則？

如需有關使用條件式存取原則之應用程式的資訊，請參閱[在 Azure Active Directory 中使用條件式存取規則的應用程式和瀏覽器](technical-reference.md)。

## <a name="are-conditional-access-policies-enforced-for-b2b-collaboration-and-guest-users"></a>是否會針對 B2B 共同作業與來賓使用者強制套用條件式存取原則？

系統會針對企業對企業 (B2B) 共同作業使用者強制執行原則。 不過，在某些情況下，使用者可能無法符合原則需求。 例如，來賓使用者的組織可能不支援多重要素驗證。 



## <a name="does-a-sharepoint-online-policy-also-apply-to-onedrive-for-business"></a>SharePoint Online 原則也適用於商務用 OneDrive 嗎？

是。 SharePoint Online 原則也會套用至商務用 OneDrive。


## <a name="why-cant-i-set-a-policy-on-client-apps-like-word-or-outlook"></a>為什麼我無法在用戶端應用程式 (例如 Word 或 Outlook) 上設定原則？

條件式存取原則會設定服務的存取需求。 在向該服務進行驗證時會強制執行。 原則並不直接在用戶端應用程式上設定， 而是在用戶端呼叫服務時套用。 例如，在 SharePoint 上設定的原則會套用至呼叫 SharePoint 的用戶端。 在 Exchange 上設定的原則會套用至 Outlook。

## <a name="does-a-conditional-access-policy-apply-to-service-accounts"></a>是否會對服務帳戶套用條件式存取原則？

條件式存取原則會套用至所有使用者帳戶。 這包括作為服務帳戶的使用者帳戶。 通常，自動執行的服務帳戶無法滿足條件式存取原則的需求。 例如，可能需要多重要素驗證。 您可以利用條件式存取原則管理設定，將服務帳戶排除在原則之外。 

## <a name="are-graph-apis-available-for-configuring-conditional-access-policies"></a>是否有提供「圖形 API」來設定條件式存取原則？

目前沒有。 

## <a name="what-is-the-default-exclusion-policy-for-unsupported-device-platforms"></a>對於不支援的裝置平台，預設的排除原則是什麼？

目前，只會對 iOS 和 Android 裝置的使用者強制執行條件式存取原則。 根據預設，iOS 和 Android 裝置的條件式存取原則不會影響其他裝置平台上的應用程式。 租用戶管理員可選擇覆寫全域原則，以禁止不受支援平台上的使用者存取。


## <a name="how-do-conditional-access-policies-work-for-microsoft-teams"></a>條件式存取原則對 Microsoft Teams 有何用途？

Microsoft Teams 在核心產能情節中非常依賴 Exchange Online 和 SharePoint Online，例如會議、行事曆和檔案共用。 當使用者直接登入 Microsoft Teams 時，對這些雲端應用程式設定的條件式存取原則會套用至 Microsoft Teams。

在 Azure Active Directory 條件式存取原則中，也支援將 Microsoft Teams 當作個別的雲端應用程式。 當使用者登入時，對雲端應用程式設定的條件式存取原則會套用至 Microsoft Teams。 不過，如果其他應用程式 (例如 Exchange Online 和 SharePoint Online) 沒有正確的原則，使用者可能仍然可以直接存取這些資源。

適用於 Windows 和 Mac 的 Microsoft Teams 桌面用戶端支援新式驗證。 新式驗證以 Azure Active Directory 驗證程式庫 (ADAL) 為基礎，支援 Microsoft Office 用戶端應用程式跨平台登入。