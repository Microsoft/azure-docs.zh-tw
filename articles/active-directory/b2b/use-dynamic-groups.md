---
title: 動態群組與 Azure Active Directory B2B 共同作業 | Microsoft Docs
description: 說明如何搭配 Azure Active Directory B2B 共同作業使用 Azure AD 動態群組
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 287c5a75d1c15a9ebc7a23d263ce7ff5516bcfab
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45981859"
---
# <a name="dynamic-groups-and-azure-active-directory-b2b-collaboration"></a>動態群組與 Azure Active Directory B2B 共同作業

## <a name="what-are-dynamic-groups"></a>什麼是動態群組？
[Azure 入口網站](https://portal.azure.com)允許您動態設定 Azure Active Directory (Azure AD) 的安全性群組成員資格。 系統管理員可以設定規則，以根據使用者屬性 (例如 userType、部門或國家/地區) 填入在 Azure AD 中建立的群組。 在安全性群組中，可以根據成員的屬性而自動新增或移除成員。 這些群組可以提供對應用程式或雲端資源 (SharePoint 網站和文件) 的存取，以及將授權指派給成員。 若要深入了解動態群組，請參閱 [Azure Active Directory 中的專用群組](../active-directory-accessmanagement-dedicated-groups.md)。

需要有適當的 [Azure AD Premium P1 或 P2 授權](https://azure.microsoft.com/pricing/details/active-directory/)，才能建立和使用動態群組。 請參閱[在 Azure Active Directory 中針對動態群組成員資格建立以屬性為基礎的規則](../users-groups-roles/groups-dynamic-membership.md)一文，以深入了解。

## <a name="what-are-the-built-in-dynamic-groups"></a>什麼是內建動態群組？
「所有使用者」動態群組讓租用戶系統管理員按一下滑鼠即可建立包含租用戶中所有使用者的群組。 根據預設值，「所有使用者」群組包括目錄中的所有使用者 (包括「成員」與「來賓」)。
在新的 Azure Active Directory 系統管理員入口網站中，您可以選擇在 [群組設定] 檢視中啟用「所有使用者」群組。

![顯示設定為 [是] 的 [啟用所有使用者群組]](media/use-dynamic-groups/enable-all-users-group.png)

## <a name="hardening-the-all-users-dynamic-group"></a>強化「所有使用者」動態群組
根據預設值，**所有使用者**群組也包含您的 B2B 共同作業 (來賓) 使用者。 您可以使用規則移除來賓使用者，以進一步保護**所有使用者**群組。 下圖顯示已修改為排除來賓的**所有使用者**群組。

![顯示使用者類型不等於來賓的規則](media/use-dynamic-groups/exclude-guest-users.png)

您可能也會發現，建立一個只包含來賓使用者的新動態群組很有用，這樣可讓您對他們套用原則 (例如 Azure AD 條件式存取原則)。
這種群組可能如下所示：

![顯示使用者類型等於來賓的規則](media/use-dynamic-groups/only-guest-users.png)

## <a name="next-steps"></a>後續步驟

- [B2B 共同作業使用者屬性](user-properties.md)
- [將 B2B 共同作業使用者新增至角色](add-guest-to-role.md)
- [B2B 共同作業使用者的條件式存取](conditional-access.md)

