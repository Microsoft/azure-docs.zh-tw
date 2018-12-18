---
title: 在 Azure Active Directory 中建立動態群組和檢查狀態 |Microsoft Docs
description: 如何在 Azure 入口網站中建立群組成員資格規則，以及檢查狀態。
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.component: users-groups-roles
ms.topic: article
ms.date: 09/20/2018
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.openlocfilehash: e418316a74ccf27ec730261957a8b6c64de5d063
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040596"
---
# <a name="create-a-dynamic-group-and-check-status"></a>建立動態群組並檢查狀態

在 Azure Active Directory (Azure AD) 中，您可套用規則，以根據使用者或裝置屬性來判斷成員資格，進而建立群組。 當使用者或裝置的屬性變更時，Azure AD 會評估 Azure AD 租用戶中的所有動態群組規則，並執行任何的新增或移除。 如果使用者或裝置滿足某個群組的規則，他/它們就會新增為成員，而當他/它們不再滿足規則時，則會遭到移除。

本文詳細說明如何針對安全性群組或 Office 365 群組的動態成員資格，在 Azure 入口網站中設定規則。 如需規則語法的範例以及成員資格規則支援的完整屬性、運算子和值清單，請參閱 [Azure Active Directory 中群組的動態成員資格規則](groups-dynamic-membership.md)。

## <a name="to-create-a-group-membership-rule"></a>建立群組成員資格規則

1. 使用具備租用戶中「全域管理員」、「Intune 服務管理員」或「使用者帳戶管理員」身分的帳戶來登入 [Azure AD 系統管理中心](https://aad.portal.azure.com)。
2. 選取 [群組]。
3. 選取 [所有群組]，然後選取 [新增群組]。

   ![Add new group](./media/groups-create-rule/new-group-creation.png)

4. 在 [群組]  刀鋒視窗上，輸入新群組的名稱和描述。 依據您是要為使用者還是裝置建立規則，在 [成員資格類型] 選取 [動態使用者] 或 [動態裝置]，然後選取 [新增動態查詢]。 您可以使用規則建立器來建立簡單的規則，或自行撰寫成員資格規則。 這篇文章包含可用的使用者和裝置屬性的詳細資訊，以及成員資格規則的範例。

   ![新增動態成員資格規則](./media/groups-create-rule/add-dynamic-group-rule.png)

5. 建立規則之後，在刀鋒視窗的底部選取 [新增查詢]。
6. 選取 [更多服務]  on the  來建立群組。

> [!TIP]
> 如果您所輸入規則的格式不正確或無效，則群組建立會失敗。 入口網站右上角會顯示通知，當中包含為何無法處理規則的解釋。 請仔細閱讀，了解您需要如何調整規則而讓規則有效。

## <a name="check-processing-status-for-a-membership-rule"></a>檢查成員資格規則的處理狀態

您可以在群組的 [概觀] 頁面上看到成員資格處理狀態和上次更新日期。
  
  ![動態群組狀態顯示](./media/groups-create-rule/group-status.png)

可能會針對**成員資格處理**狀態顯示下列狀態訊息：

* **評估中**：已收到群組變更，而且正在評估更新。
* **處理中**：正在處理更新。
* **更新完成**：處理已完成，並已建立所有適用的更新。
* **處理錯誤**：評估成員資格規則時發生錯誤，因此無法完成處理。
* **已暫停更新**：系統管理員已暫停動態成員資格規則更新。 MembershipRuleProcessingState 設定為「已暫停」。

可能會針對**成員資格上次更新**狀態顯示下列狀態訊息：

* &lt;**日期和時間**&gt;：上次更新成員資格的時間。
* **進行中**：目前正在更新。
* **未知**：無法擷取上次更新時間。 原因可能是正在新建立群組。

如果處理特定群組的成員資格規則時發生錯誤，則會在群組的 [概觀] 頁面頂端顯示警示。 如果無法處理租用戶內所有群組 24 小時內的暫止動態成員資格更新，則會在 [所有群組] 頂端顯示警示。

![處理錯誤訊息](./media/groups-create-rule/processing-error.png)

這些文章提供有關 Azure Active Directory 中群組的其他資訊。

* [查看現有的群組](../fundamentals/active-directory-groups-view-azure-portal.md)
* [建立新群組並新增成員](../fundamentals/active-directory-groups-create-azure-portal.md)
* [管理群組的設定](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [管理群組的成員資格](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [管理群組中使用者的動態規則](groups-dynamic-membership.md)
