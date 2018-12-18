---
title: 在 Azure Stack 中管理每位使用者對資源的權限 | Microsoft Docs
description: 您身為服務系統管理員或租用戶，應了解如何管理 RBAC 權限。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.assetid: cccac19a-e1bf-4e36-8ac8-2228e8487646
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: patricka
ms.reviewer: ''
ms.openlocfilehash: 2e36f52568d349aeecd47f90bf9431f096cc4fdc
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41946440"
---
# <a name="manage-access-to-resources-with-azure-stack-role-based-access-control"></a>使用 Azure Stack 角色型存取控制來管理資源存取

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

Azure Stack 支援角色型存取控制 (RBAC)，這是與 Microsoft Azure 所用的[存取管理安全性模型](https://docs.microsoft.com/azure/role-based-access-control/overview) \(機器翻譯\) 相同。 您可以使用 RBAC 來管理使用者、群組或應用程式對於訂用帳戶、資源和服務的存取。

## <a name="basics-of-access-management"></a>存取管理的基礎

角色型存取控制可以提供更細緻的存取控制，以用於保護環境。 您可以在特定範圍中指派 RBAC 角色，以提供使用者所需的確實權限。 角色指派的範圍可以是訂用帳戶、資源群組或單一資源。 如需存取管理的詳細資訊，請參閱 [Azure 入口網站中的角色型存取控制](https://docs.microsoft.com/azure/role-based-access-control/overview) \(機器翻譯\) 文章。

### <a name="built-in-roles"></a>內建角色

Azure Stack 有三個基本角色，可套用在所有資源類型：

* **擁有者**可以管理所有事項，包括對於資源的存取。
* **參與者**可以管理所有事項，但不包括對於資源的存取。
* **讀者**可以檢視所有事項，但無法進行任何變更。

### <a name="resource-hierarchy-and-inheritance"></a>資源階層和繼承

Azure Stack 具有下列資源階層：

* 每個訂用帳戶隸屬於單一目錄。
* 每個資源群組隸屬於單一訂用帳戶。
* 每個資源隸屬於單一資源群組。

在父系範圍授與的存取權會在子系範圍繼承。 例如︰

* 您可將讀者角色指派給訂用帳戶範圍內的 Azure AD 群組。 該群組的成員可以檢視訂用帳戶中的每個資源群組和資源。
* 您可將參與者角色指派給資源群組範圍內的應用程式。 應用程式可以管理該資源群組中所有類型的資源，但是無法管理訂用帳戶中的其他資源群組。

### <a name="assigning-roles"></a>指派角色

您可以將多個角色指派給使用者，而且每個角色可以關聯不同的範圍。 例如︰

* 您指派了 TestUser-A 讀者角色給 Subscription-1。
* 您指派了 TestUser-A 擁有者角色給 TestVM-1。

Azure [角色指派](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) \(機器翻譯\) 文章提供了檢視、指派與刪除角色的詳細資訊。

### <a name="resource-hierarchy-and-inheritance"></a>資源階層和繼承

Azure Stack 具有下列資源階層：

* 每個訂用帳戶隸屬於單一目錄。
* 每個資源群組隸屬於單一訂用帳戶。
* 每個資源隸屬於單一資源群組。

在父系範圍授與的存取權會在子系範圍繼承。 例如︰

* 您在訂用帳戶範圍中將**讀者**角色指派給 Azure AD 群組。 該群組的成員可以檢視訂用帳戶中的每個資源群組和資源。
* 您在資源群組範圍中將**參與者**角色指派給應用程式。 應用程式可以管理該資源群組中所有類型的資源，但是無法管理訂用帳戶中的其他資源群組。

### <a name="assigning-roles"></a>指派角色

您可以將多個角色指派給使用者，而且每個角色可以關聯不同的範圍。 例如︰

* 您指派了 TestUser-A 讀者角色給 Subscription-1。
* 您指派了 TestUser-A 擁有者角色給 TestVM-1。

Azure [角色指派](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) \(機器翻譯\) 文章提供了檢視、指派與刪除角色的詳細資訊。

## <a name="set-access-permissions-for-a-user"></a>設定使用者的存取權限

下列步驟說明如何設定使用者的權限。

1. 使用具備擁有者權限的帳戶登入您想要管理的資源。
2. 在左側導覽窗格中，選取 [資源群組]。
3. 選擇想要設定權限的資源群組名稱。
4. 在資源群組瀏覽窗格中，選擇 [存取控制 (IAM)]。 [存取控制] 檢視會列出可存取資源群組的項目。 您可以篩選這些結果，並使用功能表列來 [新增] 或 [移除] 權限。
5. 在 [存取控制] 功能表列上，選擇 [+ 新增]。
6. 在 [新增權限] 上：

   * 從 [角色] 下拉式清單中，選擇想要從指派的角色。
   * 從 [存取權指派對象為] 下拉式清單中，選擇想要從指派的資源。
   * 選取目錄中您要授與存取權的使用者、群組或應用程式。 您可以使用顯示名稱、電子郵件地址和物件識別碼來搜尋目錄。

7. 選取 [ **儲存**]。

## <a name="next-steps"></a>後續步驟

[建立服務主體](azure-stack-create-service-principals.md)