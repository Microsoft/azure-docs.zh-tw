---
title: Azure Active Directory 中的單一和多租用戶應用程式
description: 深入了解 Azure AD 中單一租用戶與多租用戶應用程式之間的功能和差異。
services: active-directory
documentationcenter: ''
author: CelesteDG
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: celested
ms.reviewer: justhu
ms.custom: aaddev
ms.openlocfilehash: 8986f18b9f1f2953ad0612679d31996628870cb1
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46988218"
---
# <a name="tenancy-in-azure-active-directory"></a>Azure Active Directory 中的租用戶

Azure Active Directory (Azure AD) 會將例如使用者和應用程式的物件，組織到稱為「租用戶」的群組。 租用戶允許系統管理員對組織內的使用者和組織擁有的應用程式設定原則，以符合其安全性和作業原則。 

## <a name="who-can-sign-in-to-your-app"></a>誰可以登入您的應用程式？

來到開發應用程式階段時，開發人員可以在應用程式註冊期間，在 [Azure 入口網站](http://portal.azure.com)中選擇將其應用程式設為單一租用戶或多租用戶。
* 單一租用戶應用程式僅適用於他們註冊所在的租用戶，也稱為主租用戶。
* 多租用戶應用程式同時適用於主租用戶和其他租用戶中的使用者。

在 Azure 入口網站中，您可以藉由如下所示設定對象，將您的應用程式設為單一租用戶或多租用戶。

| 對象 | 單一/多租用戶 | 誰可以登入 | 
|----------|--------| ---------|
| 只有這個目錄中的帳戶 | 單一租用戶 | 您目錄中的所有使用者和來賓帳戶都可以使用您的應用程式或 API。<br>若您的目標對象是組織內部的人員，請使用此選項。 |
| 任何 Azure AD 目錄中的帳戶 | 多租用戶 | 所有使用者及來賓只要具備 Microsoft 提供的公司或學校帳戶，都能使用您的應用程式或 API。 這包括使用 Office 365 的學校及公司。<br>若您的目標對象是商務界或教育界的客戶，請使用此選項。 |
| 任何 Azure AD 目錄中的帳戶和個人 Microsoft 帳戶 (例如 Skype、Xbox、Outlook.com) | 多租用戶 | 任何使用者只要具有公司、學校或個人的 Microsoft 帳戶，都能使用您的應用程式或 API。 這也包括了使用 Office 365 的學校和公司，以及用來登入例如 Xbox 和 Skype 服務的個人帳戶。<br>使用此選項將目標設為 Microsoft 帳戶的最廣泛集合。 | 

## <a name="best-practices-for-multi-tenant-apps"></a>多租用戶應用程式的最佳做法

建置絕佳的多租用戶應用程式是一項挑戰，因為 IT 系統管理員在其租用戶中可以設定許多不同的原則。 如果您選擇建置多租用戶應用程式，請遵循這些最佳做法：

* 在已設定[條件式存取原則](conditional-access-dev-guide.md)的租用戶中測試您的應用程式。
* 請遵循最低使用者存取權的原則，以確保您的應用程式只會要求實際需要的權限。 請避免要求需要系統管理員同意的權限，因為這樣可能會在某些組織中完全阻止使用者取得您的應用程式。 
* 請為您公開為應用程式一部分的任何權限，提供適當的名稱和描述。 這有助於使用者和系統管理員在他們嘗試使用您應用程式的 API 時，知道他們同意了什麼。 如需詳細資訊，請參閱[權限指南](v1-permissions-and-consent.md)中的最佳做法一節。

## <a name="next-steps"></a>後續步驟

* [如何將應用程式轉換成多租用戶](howto-convert-app-to-be-multi-tenant.md)
