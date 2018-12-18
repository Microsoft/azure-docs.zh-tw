---
title: PIM 中的合格指派和資源可見度 - Azure | Microsoft Docs
description: 說明如何在 Azure AD Privileged Identity Management (PIM) 中，指派 Azure 資源角色的合格成員。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: pim
ms.date: 04/02/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: fb52bc92c86261831d0e8d8e9e863a4863fe8fb9
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666884"
---
# <a name="eligible-assignments-and-resource-visibility-in-pim"></a>PIM 中的合格指派和資源可見度

適用於 Azure 資源角色的 Privileged Identity Management (PIM) 可為具有重要 Azure 資源的組織，提供增強式安全性。 資源管理員可以使用 PIM 將成員指派為合格的資源角色。 深入了解不同的指派類型和下列各節中 Azure 資源角色的指派狀態。 

## <a name="assignment-types"></a>指派類型

適用於 Azure 資源的 PIM 提供兩種不同的指派類型：

- 合格
- Active

合格的指派會要求角色成員先執行某個動作才能使用此角色。 動作可能包含成功執行多重要素驗證檢查、提供業務理由，或是向指定的核准者要求核准。

有效的指派不要求成員先執行某個動作才能使用此角色。 指派為有效的成員隨時具有指派給角色的權限。

## <a name="assignment-duration"></a>指派持續時間

資源管理員可以在進行角色的 PIM 設定時，針對每個指派類型從兩個選項中選擇。 將成員指派給 PIM 中的角色後，下列選項才會變成預設最長持續時間。 

管理員可以從下列指派類型中選擇一個：

- 允許永久合格的指派
- 允許永久有效的指派

或者管理員可以從下列指派類型中選擇一個：

- 合格指派的有效期限
- 有效指派的有效期限

如果資源管理員選擇 [允許永久合格的指派] 或 [允許永久有效的指派]，將成員指派給此資源的所有管理員都可以指派永久的成員資格。

如果資源管理員選擇 [合格指派的有效期限] 或 [有效指派的有效期限]，資源管理員即可控制指派生命週期，做法是要求所有的指派都具有指定的開始和結束日期。

> [!NOTE] 
> 資源管理員可以更新所有具有指定結束日期的指派。 此外，成員可以將自助服務要求初始化，以[擴充或更新指派](pim-resource-roles-renew-extend.md)。


## <a name="assignment-states"></a>指派狀態

適用於 Azure 資源的 PIM 有兩個不同的指派狀態，這兩個狀態會顯示在 PIM 的 [我的角色]、[角色] 和 [成員] 檢視中的 [有效角色] 索引標籤上。 這些狀態包括：

- 已指派
- 已啟用

檢視 [有效角色] 中所列的成員資格時，您可以使用 [狀態] 資料行中的值區分「已指派」為有效的使用者與「已啟用」合格指派且現在有效的使用者。

## <a name="azure-resource-role-approval-workflow"></a>Azure 資源角色核准工作流程

系統管理員可以使用 PIM 中適用於 Azure 資源角色的核准工作流程，進一步保護或限制對重要資源的存取。 也就是說，系統管理員需要核准，才能啟用角色指派。

資源階層的概念是 Azure 資源角色獨有的。 此階層能夠繼承從父資源物件向下到父容器內所有子資源的角色指派。 

例如：資源系統管理員 Bob 使用 PIM，將 Alice 指派為 Contoso 訂用帳戶中擁有者角色的合格成員。 透過此指派，Alice 就是 Contoso 訂用帳戶內所有資源群組容器的合格擁有者。 Alice 也是訂用帳戶每個資源群組內所有資源 (例如虛擬機器) 的合格擁有者。

讓我們假設 Contoso 訂用帳戶中有三個資源群組：Fabrikam Test、Fabrikam Dev 和 Fabrikam Prod。每個資源群組都包含單一虛擬機器。

資源的每個角色都會設定 PIM 設定。 與指派不同的是，這些設定不會繼承且僅適用於資源角色。

繼續上述範例：Bob 使用 PIM，要求 Contoso 訂用帳戶擁有者角色的所有成員都需要核准才能啟用。 為了協助保護 Fabrikam Prod 資源群組中的資源，Bob 也會要求此資源的擁有者角色成員必須取得核准。 Fabrikam Test 和 Fabrikam Dev 中的擁有者角色不需要取得核准就能啟用。

當 Alice 要求啟用其 Contoso 訂用帳戶擁有者角色時，核准者必須先核准或拒絕她的要求，之後她才能成為有效角色。 如果 Alice 決定[將其啟用範圍侷限](pim-resource-roles-activate-your-roles.md)在 Fabrikam Prod 資源群組，核准者也必須核准或拒絕此要求。 不過，如果 Alice 決定將其啟用範圍侷限在Fabrikam Test 和/或 Fabrikam Dev，則不需要核准。

角色成員不一定都需要進行核准工作流程。 假設您的組織雇用數名約聘夥伴，協助開發將在 Azure 訂用帳戶中執行的應用程式。 身為資源系統管理員，您希望員工不需核准就具有合格的存取權，但約聘夥伴必須要求核准。 若要只針對約聘夥伴設定核准工作流程，您可以建立自訂角色，再賦予其與員工所獲派角色相同的權限。 您需要核准，才可啟用該自訂角色。 [深入了解自訂角色](pim-resource-roles-custom-role-policy.md)。

## <a name="next-steps"></a>後續步驟

- [在 PIM 中指派 Azure 資源角色](pim-resource-roles-assign-roles.md)
