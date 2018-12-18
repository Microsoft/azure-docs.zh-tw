---
title: 如何為自訂開發的應用程式變更權杖存留期預設值 | Microsoft Docs
description: 如何為針對 Azure AD 開發的應用程式更新權杖存留期原則
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
ms.openlocfilehash: 059920c710b202e22a22f8431c536c5dfa19f2b9
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44723687"
---
# <a name="how-to-change-the-token-lifetime-defaults-for-a-custom-developed-application"></a>如何為自訂開發的應用程式變更權杖存留期預設值

Azure AD Premium 可讓應用程式開發人員與租用戶系統管理員為針對非機密用戶端簽發的權杖設定存留期。 權杖存留期原則是以整個租用戶為基礎所設定，或針對要存取的資源所設定。

 * 若要設定權杖存留期原則，您必須下載 [Azure AD PowerShell 模組 (英文)](https://www.powershellgallery.com/packages/AzureADPreview)。

 * 執行 **Connect-AzureAD -Confirm** 命令。

 * 下列範例原則會設定單一要素重新整理權杖存留期上限。 建立原則：```New-AzureADPolicy -Definition @('{"TokenLifetimePolicy":{"Version":1, "MaxAgeSingleFactor":"until-revoked"}}') -DisplayName "OrganizationDefaultPolicyScenario" -IsOrganizationDefault $true -Type "TokenLifetimePolicy"```

 * 請參閱[設定權杖存留期](https://docs.microsoft.com/azure/active-directory/active-directory-configurable-token-lifetimes)一文，以了解如何建立其他自訂。

## <a name="next-steps"></a>後續步驟
[設定權杖存留期](https://docs.microsoft.com/azure/active-directory/active-directory-configurable-token-lifetimes)<br>

[Azure AD 權杖參考](https://docs.microsoft.com/azure/active-directory/develop/active-directory-token-and-claims)

