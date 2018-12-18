---
title: 啟用租用戶中的訂用帳戶管理 - Azure | Microsoft Docs
description: 了解如何在使用 Azure AD Privileged Identity Management (PIM) 時，啟用您租用戶中的訂用帳戶管理。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: pim
ms.date: 03/27/2018
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 89bb6fd48c58b7672b7a2251a172cc169093d368
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189990"
---
# <a name="enable-subscription-management-in-your-tenant"></a>啟用租用戶中的訂用帳戶管理

您身為目錄的全域管理員，可能沒有您租用戶中所有訂用帳戶資源的預設存取權。 本文章會概略說明各項步驟，讓您得以存取您租用戶中的所有訂用帳戶。 也會提供建議的作法，以便在獲得存取權，後仍能符合貴組織對任何安全控制項的規範。

## <a name="who-can-enable-management-of-subscriptions-in-my-directory"></a>有誰可以啟用我目錄中訂用帳戶的管理？

指派給全域管理員角色的每位使用者都必須遵循下列步驟來啟用訂用帳戶管理。 您自行啟用了訂用帳戶管理後，就可以新增其他也可能需要資源存取權的全域管理員。 沒有任何目錄設定可為全域管理員角色的所有成員啟用存取權。

## <a name="sign-in-to-the-azure-portal"></a>登入 Azure 入口網站

使用屬於全域管理員角色合格或有效成員的帳戶來登入 Azure 入口網站。 如果此帳戶是合格的全域管理員，您必須先啟用該角色，再繼續下一個步驟。

## <a name="access-the-azure-active-directory-admin-center"></a>存取 Azure Active Directory 管理中心

既然您已使用全域管理員身分登入 Azure 入口網站，就可以編輯提供 Azure 訂用帳戶存取權的設定。 瀏覽至 Azure Active Directory (Azure AD) 管理中心，並選取 [屬性]。

![Azure AD 管理中心的螢幕擷取畫面，以反白顯示 [屬性]](media/azure-pim-resource-rbac/aad_properties.png)

在屬性清單中，於 [全域管理員可管理 Azure 訂用帳戶] 下方選取 [是]。

![[屬性] 頁面的螢幕擷取畫面，設定切換為 [是]](media/azure-pim-resource-rbac/aad_properties_save.png)

現在您的帳戶會自動新增至租用戶中每個訂用帳戶資源的「使用者存取權管理員」角色。

## <a name="browse-to-azure-ad-pim"></a>瀏覽至 Azure AD PIM

 從此處前往 Azure AD Privileged Identity Management (PIM)。 在 [管理] 下方，選取 [Azure 資源]。

![PIM 的螢幕擷取畫面，以反白顯示 [Azure 資源]](media/azure-pim-resource-rbac/aadpim_manage_azure_resources.png)

## <a name="manage-and-discover-resources"></a>管理及探索資源

如果貴組織已經使用 Azure AD PIM 來保護 Azure AD 中的管理員，您會在刀鋒視窗載入時看到訂用帳戶清單。

![PIM 的螢幕擷取畫面，在刀鋒視窗中顯示訂用帳戶清單](media/azure-pim-resource-rbac/aadpim_manage_azure_resource_some_there.png)

> [!NOTE]
> 如果看不到任何資源，請確認：
>- 您的全域管理員角色並未過期。 
>- 您的組織具有 Azure 訂用帳戶。

![PIM 的螢幕擷取畫面，顯示空白的資源清單](media/azure-pim-resource-rbac/aadpim_rbac_empty_resource_list.png)

## <a name="configure-assignments"></a>設定指派

選取清單中的訂用帳戶，然後將您自己 (或您所隸屬的群組) 指派為資源的合格擁有者。 
[深入了解指派角色](pim-resource-roles-assign-roles.md)。

繼續下一個步驟前，請針對每個資源重複此程序。

## <a name="clean-up-standing-access"></a>清除常設存取權

既然您具有貴組織中重要訂用帳戶的合格指派，您可藉由停用目錄屬性中的選項來清除常設存取權。

![[屬性] 頁面的螢幕擷取畫面，設定切換為 [否]](media/azure-pim-resource-rbac/aad_properties_no.png)

## <a name="next-steps"></a>後續步驟

- [在 PIM 中探索要管理的 Azure 資源](pim-resource-roles-discover-resources.md)
- [在 PIM 中設定 Azure 資源角色設定](pim-resource-roles-configure-role-settings.md)
