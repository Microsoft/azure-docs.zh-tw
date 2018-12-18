---
title: Azure Active Directory B2C 中的 Multi-Factor Authentication | Microsoft Docs
description: 如何在受 Azure Active Directory B2C 保護的取用者導向應用程式中啟用 Multi-Factor Authentication。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/06/2016
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 3d18e1b2e45aba4e83989e29c533cfc7bf5033fc
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442703"
---
# <a name="azure-active-directory-b2c-enable-multi-factor-authentication-in-your-consumer-facing-applications"></a>Azure Active Directory B2C：在取用者導向應用程式中啟用 Multi-Factor Authentication
Azure Active Directory (Azure AD) B2C 直接整合 [Azure Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md) ，讓您能夠針對取用者導向應用程式的註冊與登入使用體驗，增添第二層安全性。 您連一行程式碼都不用寫，即可達成此目標。 目前我們支援撥打電話與簡訊驗證。 如果您已經建立註冊和登入原則，您仍然可以啟用 Multi-Factor Authentication。

> [!NOTE]
> Multi-Factor Authentication 也可以在建立註冊和登入原則時啟用，而非單靠編輯現有原則來啟用。
> 
> 

此功能可協助應用程式處理諸如下列的各種案例：

* 您在存取某一個應用程式時不需要 Multi-Factor Authentication，但在存取另一應用程式時需要它。 例如，取用者可以使用社交或本機帳戶登入汽車保險應用程式，但必須先驗證電話號碼，才能存取同個目錄中註冊的家庭保險應用程式。
* 一般而言，您在存取應用程式時不需要 Multi-Factor Authentication，但在存取其中的敏感部分資訊時則需要它。 例如，取用者可使用社交或本機帳戶登入銀行應用程式來檢查帳戶餘額，但必須先完成電話號碼驗證，才能嘗試進行轉帳匯款作業。

## <a name="modify-your-sign-up-policy-to-enable-multi-factor-authentication"></a>修改註冊原則以啟用 Multi-Factor Authentication
1. 遵循下列步驟以[瀏覽至 B2C 功能刀鋒視窗](active-directory-b2c-app-registration.md#navigate-to-b2c-settings) (位於 Azure 入口網站上)。
2. 按一下 [註冊原則] 。
3. 按一下以開啟註冊原則 (例如 "B2C_1_SiUp")。
4. 按一下 [多重要素驗證]，並將 [狀態] 設為 [開啟]。 按一下 [確定]。
5. 按一下刀鋒視窗頂端的 [儲存]  。

您可以使用原則上的「立即執行」功能來驗證取用者體驗。 確認下列項目：

在進行 Multi-Factor Authentication 步驟之前，會在目錄中建立取用者帳戶。 在步驟執行過程中，系統會要求取用者提供電話號碼進行驗證。 若驗證成功，會將電話號碼附加至取用者帳戶以供之後使用。 即使取用者取消或卸除，下次登入時系統仍會要求其再度驗證電話號碼 (已啟用 Multi-Factor Authentication)。

## <a name="modify-your-sign-in-policy-to-enable-multi-factor-authentication"></a>修改登入原則以啟用 Multi-Factor Authentication
1. 遵循下列步驟以[瀏覽至 B2C 功能刀鋒視窗](active-directory-b2c-app-registration.md#navigate-to-b2c-settings) (位於 Azure 入口網站上)。
2. 按一下 [登入原則] 。
3. 按一下以開啟登入原則 (例如 "B2C_1_SiIn")。 按一下刀鋒視窗頂端的 [編輯] 。
4. 按一下 [多重要素驗證]，並將 [狀態] 設為 [開啟]。 按一下 [確定]。
5. 按一下刀鋒視窗頂端的 [儲存]  。

您可以使用原則上的「立即執行」功能來驗證取用者體驗。 確認下列項目：

取用者登入 (使用社交或本機帳戶) 時，若已將通過驗證的電話號碼附加至取用者帳戶，系統會要求其進行驗證。 如果未附加電話號碼，系統會要求取用者提供電話號碼以進行驗證。 驗證成功之後，即會將電話號碼附加至取用者帳戶以供之後使用。

## <a name="multi-factor-authentication-on-other-policies"></a>其他原則上的 Multi-Factor Authentication
如同對上面的註冊和登入原則所述，也可以對註冊或登入原則和密碼重設原則啟用 Multi-Factor Authentication。 很快即可用於設定檔編輯原則。

