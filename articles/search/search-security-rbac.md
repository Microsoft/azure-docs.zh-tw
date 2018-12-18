---
title: 在入口網站中設定 Azure 搜尋服務管理存取權的 RBAC 角色 | Microsoft Docs
description: Azure 入口網站中的角色型管理控制項。
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 03/20/2018
ms.author: heidist
ms.openlocfilehash: 1a463c9eb52ffe2a667cdeace7478e67233a0806
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31792921"
---
# <a name="set-rbac-roles-for-administrative-access"></a>設定系統管理存取權的 RBAC 角色

Azure 特別為透過入口網站或 Resource Manager API 管理的所有服務提供[全域角色型授權模型](../role-based-access-control/role-assignments-portal.md)。 「擁有者」、「參與者」和「讀取者」角色可針對指派給各角色的 Active Directory 使用者、群組和安全性主體，決定*服務管理*層級。 

> [!Note]
> 沒有角色型存取控制項來保護部分索引或文件子集。 針對搜尋結果的身分識別型存取，您可以建立安全性篩選，依身分識別修剪結果、移除要求者不應具備存取權的文件。 如需詳細資訊，請參閱[安全性篩選](search-security-trimming-for-azure-search.md)和[使用 Active Directory 保護安全](search-security-trimming-for-azure-search-with-aad.md)。

## <a name="management-tasks-by-role"></a>依角色管理工作

對於 Azure 搜尋服務，角色與支援以下管理工作的權限等級相關聯：

| 角色 | Task |
| --- | --- |
| 擁有者 |建立或刪除服務或服務上的任何物件，包括 api 索引鍵、索引、索引子、索引子資料來源和索引子排程。<p>檢視服務狀態，包括計數和儲存體大小。<p>新增或刪除角色成員資格 (只有「擁有者」可以管理角色成員資格)。<p>訂用帳戶管理員和服務擁有者在擁有者角色具有自動成員資格。 |
| 參與者 |與「擁有者」相同層級的存取權，減去 RBAC 角色管理。 例如，參與者可以建立或刪除物件，或檢視和重新產生 [API 金鑰](search-security-api-keys.md)，但不能修改角色成員資格。 |
| [搜尋服務參與者內建角色](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#search-service-contributor) | 等同於參與者角色。 |
| 讀取者 |檢視服務的基本資訊和計量。 此角色的成員無法檢視索引、索引子、資料來源，或索引鍵資訊。  |

角色不會授與服務端點的存取權限。 搜尋服務作業 (例如索引管理、索引母體擴展，以及搜尋資料查詢) 可透過 api-key 而非角色來控制。 如需詳細資訊，請參閱[管理 API 金鑰](search-security-api-keys.md)。

## <a name="see-also"></a>另請參閱

+ [使用 Powershell 管理](search-manage-powershell.md) 
+ [Azure 搜尋服務中的效能和最佳化](search-performance-optimization.md)
+ [在 Azure 入口網站中開始使用角色型存取控制](../role-based-access-control/overview.md)。