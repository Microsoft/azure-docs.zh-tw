---
title: 疑難排解群組的動態成員資格 | Microsoft Docs
description: Azure AD 中群組動態成員資格的疑難排解秘訣。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 09/11/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: e189fb8b2bc5079d1560d3b7a54fea2db7366fe7
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46293962"
---
# <a name="troubleshooting-dynamic-memberships-for-groups"></a>疑難排解群組的動態成員資格

**我針對某個群組設定了一個規則，但是群組中的成員資格沒有任何更新**<br/>請針對規則確認使用者屬性的值：是否有符合規則的使用者？ 如果一切良好，請等候一段時間，以填入群組。 根據您的租用戶大小，群組可能最多需要 24 小時來完成初次填入或規則變更。

**我已設定一個規則，但是目前現有的規則成員已被移除**<br/>這是預期行為。 因為在啟用或變更規則時，現有群組的成員會遭到移除。 從評估規則所傳回的使用者會當作成員，新增至群組。

**當我新增或變更規則時，沒有立即看到成員資格變更，為什麼？**<br/>專用的成員資格評估會定期在非同步的背景處理序中完成。 該程序所需的時間，取決於您目錄中的使用者數目以及因規則所建立的群組大小。 具有少量使用者的目錄通常會在很短的時間內看到群組成員資格變更。 具有大量使用者的目錄則可能需要 30 分鐘或更長的時間填入。

**我遇到規則處理錯誤**<br/>下表列出常見動態成員資格規則錯誤及其更正方式。

| 規則剖析器錯誤 | 錯誤的使用方式 | 更正的使用方式 |
| --- | --- | --- |
| 錯誤：不支援屬性。 |(user.invalidProperty -eq "Value") |(user.department -eq "value")<br/><br/>請確定此屬性位於[支援的屬性清單](groups-dynamic-membership.md#supported-properties)中。 |
| 錯誤：屬性不支援運算子。 |(user.accountEnabled -contains true) |(user.accountEnabled -eq true)<br/><br/>屬性類型不支援使用的運算子 (在此範例中，-contains 不能在布林類型上使用)。 屬為屬性類型使用正確的運算子。 |
| 錯誤：查詢編譯錯誤。 | 1. (user.department -eq "Sales") (user.department -eq "Marketing")<br>2.  (user.userPrincipalName -match "*@domain.ext") | 1.缺少運算子。 使用這兩個 -and 或 -or 聯結述詞<br>(user.department -eq "Sales") -or (user.department -eq "Marketing")<br>2.使用 -match 的規則運算式發生錯誤<br>(user.userPrincipalName -match ".*@domain.ext")<br>或者：(user.userPrincipalName -match "@domain.ext$") |

## <a name="next-steps"></a>後續步驟

這些文章提供有關 Azure Active Directory 的其他資訊。

* [使用 Azure Active Directory 群組來管理資源的存取權](../fundamentals/active-directory-manage-groups.md)
* [Azure Active Directory 中的應用程式管理](../manage-apps/what-is-application-management.md)
* [什麼是 Azure Active Directory？](../fundamentals/active-directory-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](../hybrid/whatis-hybrid-identity.md)
