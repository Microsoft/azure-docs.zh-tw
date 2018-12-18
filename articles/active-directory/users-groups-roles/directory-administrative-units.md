---
title: 在 Azure Active Directory 中的管理單位管理預覽
description: 使用管理單位在 Azure Active Directory 中進行更細微的權限委派
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: article
ms.component: users-groups-roles
ms.workload: identity
ms.date: 08/17/2017
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: oldportal;it-pro;
ms.openlocfilehash: 0884726e59d9ab3f5a5cfe7bb0608f6b5a5da250
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37448860"
---
# <a name="administrative-units-management-in-azure-ad---public-preview"></a>在 Azure AD 中的管理單位管理 - 公用預覽版
本文說明系統管理單位，即新的 Azure Active Directory 資源容器，可用來將系統管理權限委派給使用者的子集，並將原則套用到使用者的子集。 在 Azure Active Directory 中，管理單位可讓管理中心將權限委派至區域管理員或以更細微的層級來設定原則。

這在具有獨立部門的組織，例如由許多彼此獨立的學院 (商學院和工學院等) 所組成的大型大學中非常有用。 這類部門擁有自己的 IT 管理員，可控制存取、管理使用者，以及針對其部門設定特別原則。 管理中心想要能夠針對其特定部門中的使用者，授與這些部門的管理員權限。 更具體來說，例如管理中心可以使用此範例，針對特定學院 (商學院) 建立一個管理單位，並僅在其中填入商學院使用者。 然後管理中心可以將商學院的 IT 人員加入至範圍內的角色，換句話說，僅針對學院管理單位將管理權限授與商學院的 IT 人員。

> [!IMPORTANT]
> 若要使用管理單位，管理單位範圍內的管理員須具有 Azure Active Directory Premium 授權，而管理單位中的所有使用者要有 Azure Active Directory Basic 授權。 如需詳細資訊，請參閱〈 [開始使用 Azure AD Premium](../fundamentals/active-directory-get-started-premium.md)〉。
>


從管理中心的觀點來看，管理單位是可以建立並填入資源的目錄物件。 **在此預覽版本中，這些資源僅能是使用者。** 一旦建立並填入，管理單位可用作為範圍，以限制僅能針對管理單位中所包含的資源來授與權限。

## <a name="managing-administrative-units"></a>管理管理單位
在此預覽版本中，您可以使用適用於 Windows PowerShell Cmdlet 的 Azure Active Directory 模組來建立和管理管理單位。 若要深入了解做法，請參閱 [Working with Administrative Units](https://docs.microsoft.com/powershell/azure/active-directory/working-with-administrative-units?view=azureadps-2.0) (使用管理單位)

如需軟體需求和安裝 Azure AD 模組的詳細資訊，以及使用 Azure AD 模組 Cmdlet 管理管理單位的資訊 (包括語法、參數說明和範例)，請參閱 [Azure Active Directory PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/overview?view=azureadps-2.0)。

## <a name="next-steps"></a>後續步驟
[Azure Active Directory 版本](../fundamentals/active-directory-whatis.md)
