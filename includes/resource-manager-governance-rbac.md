---
title: 包含檔案
description: 包含檔案
services: azure-resource-manager
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 02/16/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: b4b06119b9d46781b967fc8d98808c60d2b41ccb
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38753632"
---
您想要確定組織中的使用者具有這些資源的正確存取權等級。 您不想要授與不受限制的存取權給使用者，但是您又必須確定他們可以執行其工作。 角色型存取控制 (RBAC) 可讓您管理哪些使用者具有權限可以在一個範圍內完成特定動作。 角色會定義一組允許的動作。 您可將角色指派給某個範圍，並指定哪些使用者屬於此範圍的該角色。

在規劃存取控制策略時，授與使用者完成其工作的最低權限。 下圖顯示指派 RBAC 的建議模式。

![影響範圍](./media/resource-manager-governance-rbac/role-examples.png)

以下三個角色適用於所有的資源：擁有者、參與者和讀者。 任何獲得「擁有者」角色指派的帳戶都應該受到緊密控制，且鮮少使用。 只需要觀察解決方案狀態的使用者應被授與「讀者」角色。

大部分使用者會在訂用帳戶或資源群組層級被授與[資源特定角色](../articles/role-based-access-control/built-in-roles.md)或[自訂角色](../articles/role-based-access-control/custom-roles.md)。 這些角色會嚴格定義允許的動作。 藉由為使用者指派這些角色，您可對使用者授與必要的存取權，但不允許有太多的控制權。 您可以將帳戶指派給一個以上的角色，而該使用者會取得角色的合併權限。 在資源層級授與存取權通常對使用者有太多限制，但是可用於針對特定工作所設計的自動化程序。

### <a name="who-can-assign-roles"></a>誰可以指派角色

若要建立和移除角色指派，使用者必須具有 `Microsoft.Authorization/roleAssignments/*` 存取權。 此存取權是透過擁有者或使用者存取系統管理員角色來授與。