---
title: Azure Active Directory B2C 中的密碼複雜度 | Microsoft Docs
description: 如何設定 Azure Active Directory B2C 中取用者所提供的密碼複雜度需求。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 08/16/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 4b027f6cd57dfa48ba2e230371ffcad97b1f8ec4
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445104"
---
# <a name="azure-ad-b2c-configure-complexity-requirements-for-passwords"></a>Azure AD B2C：設定密碼複雜度需求

> [!NOTE]
> **這項功能處於公開預覽狀態。**

Azure Active Directory B2C (Azure AD B2C) 支援的變更密碼複雜度需求，是在建立帳戶時由使用者所提供的。  根據預設，Azure AD B2C 是使用 `Strong` 密碼。  Azure AD B2C 也支援組態選項，可控制客戶可以使用的密碼複雜度。

## <a name="when-password-rules-are-enforced"></a>當強制執行密碼規則時

在註冊期間或密碼重設期間，使用者必須提供符合複雜度規則的密碼。  會依每個原則強制執行密碼複雜度規則。  可能在註冊期間一個原則會需要四位數的 PIN，同時另一個原則在註冊期間會需要八個字元字串。  例如，您可能會針對成人的原則與針對兒童的原則使用不同的密碼複雜度。

登入期間一律不會強制要求密碼複雜度。  系統一律不會在登入時提示使用者變更其密碼，因為它不符合目前的複雜度需求。

以下是可設定密碼複雜度的原則類型：

* 註冊或登入原則
* 密碼重設原則
* 自訂原則 ([在自訂原則中設定密碼複雜度](active-directory-b2c-reference-password-complexity-custom.md))

## <a name="how-to-configure-password-complexity"></a>如何設定密碼複雜度

1. 遵循下列步驟，以[瀏覽至 Azure AD B2C 設定](active-directory-b2c-app-registration.md#navigate-to-b2c-settings)。
1. 開啟 [註冊或登入原則]。
1. 選取原則，然後按一下 [編輯]。
1. 開啟 [密碼複雜度]。
1. 將這個原則的密碼複雜度變更為 [簡單]、[強式] 或 [自訂]。

### <a name="comparison-chart"></a>比較圖表

| 複雜度 | 說明 |
| --- | --- |
| 簡單 | 密碼至少為 8 到 64 個字元。 |
| 強式 | 密碼至少為 8 到 64 個字元。 它需要小寫字母、大寫字母、數字或符號 4 個之中的 3 個。 |
| 自訂 | 此選項會對密碼複雜度規則提供最多的控制權。  它允許設定自訂的長度。  它也可以接受僅限數字的密碼 (PIN)。 |

## <a name="options-available-under-custom"></a>[自訂] 下的可用選項

### <a name="character-set"></a>字元集

可讓您只接受數字 (PIN) 或完整的字元集。

* **僅限數字**在輸入密碼時只允許數字 (0-9)。
* **所有**允許任何字母、數字或符號。

### <a name="length"></a>長度

可讓您控制密碼的長度要求。

* **最小長度**必須至少為 4。
* **最大長度**必須大於或等於最小長度，且最多可以有 64 個字元。

### <a name="character-classes"></a>字元類別

可讓您控制密碼中使用的不同字元類型。

* **4 個其中 2 個：小寫字元、大寫字元、數字 (0-9)、符號**可確保密碼包含至少兩個字元類型。 例如，數字和小寫字元。
* **4 個其中 3 個：小寫字元、大寫字元、數字 (0-9)、符號**可確保密碼包含至少兩個字元類型。 例如，數字、小寫字元和大寫字元。
* **4 個其中 4 個：小寫字元、大寫字元、數字 (0-9)、符號**可確保密碼包含所有的字元類型。

    > [!NOTE]
    > 要求 **4 個其中 4 個**可能會導致使用者感到挫折。 一些根據研究顯示，這項需求並未改善密碼熵。 請參閱 [NIST 密碼指導方針](https://pages.nist.gov/800-63-3/sp800-63b.html#appA)
