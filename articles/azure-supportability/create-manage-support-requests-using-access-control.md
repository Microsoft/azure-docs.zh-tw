---
title: 使用 Azure 角色型存取控制 (RBAC) 控制建立及管理支援要求的存取權 | Microsoft Docs
description: 使用 Azure 角色型存取控制 (RBAC) 控制建立及管理支援要求的存取權
author: ganganarayanan
ms.author: gangan
ms.date: 1/31/2017
ms.topic: article
ms.service: microsoft-docs
ms.assetid: 58a0ca9d-86d2-469a-9714-3b8320c33cf5
ms.openlocfilehash: c3557d54b9fb3fd0173eec1e327050ac7cc80fc2
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31392896"
---
# <a name="azure-role-based-access-control-rbac-to-control-access-rights-to-create-and-manage-support-requests"></a>使用 Azure 角色型存取控制 (RBAC) 控制建立及管理支援要求的存取權

[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview) 提供適用於 Azure 的詳細存取權管理。
Azure 入口網站 ([portal.azure.com](https://portal.azure.com)) 中建立支援要求的功能是使用 Azure 的 RBAC 模型來定義可以建立及管理支援要求的人員。
在訂用帳戶、資源群組和應用程式等特定範圍，將適當的 RBAC 角色指派給使用者、群組和應用程式，即可授與存取權。

我們來看一個範例：您是資源群組擁有者，且擁有訂用帳戶範圍讀取權限，您可以管理資源群組中的所有資源，如網站、虛擬機器，以及子網路。
不過，當您嘗試建立虛擬機器資源的支援要求時，會遇到錯誤如下

![訂用帳戶錯誤](./media/create-manage-support-requests-using-access-control/subscription-error.png)

在支援要求管理中，您需要有寫入權限，或是在訂用帳戶範圍有 Microsoft.Support/* 支援動作的角色，才能建立及管理支援要求。

以下文章說明您可以如何使用 Azure 的自訂角色型存取控制 (RBAC)，在 Azure 入口網站中建立及管理支援要求。

## <a name="getting-started"></a>開始使用

如果訂用帳戶擁有者指派您訂用帳戶的自訂 RBAC 角色，您就能使用上述的範例為資源建立支援要求。
使用 Azure PowerShell、Azure 命令列介面 (CLI) 和 REST API，可以建立[自訂 RBAC 角色](https://azure.microsoft.com/documentation/articles/role-based-access-control-custom-roles/)。

自訂角色的動作屬性會指定角色授與存取權的 Azure 作業。
若要建立支援要求管理的自訂角色，該角色必須有 Microsoft.Support/* 動作。

以下是自訂角色的範例，它可以建立及管理支援要求。
我們將此角色命名為 "Support Request Contributor" (支援要求參與者)，在本文中這個名稱指的就是自訂角色。

``` Json
{
    "Name":  "Support Request Contributor",
    "Id":  "1f2aad59-39b0-41da-b052-2fb070bd7942",
    "IsCustom":  true,
    "Description":  "Lets you create and manage support tickets.",
    "Actions":  [
                    "Microsoft.Support/*"
                ],
    "NotActions":  [
                   ],
    "AssignableScopes":  [
                             "/"
                         ]
}
```

遵循[影片](https://www.youtube.com/watch?v=-PaBaDmfwKI)中的步驟以了解如何為您的訂用帳戶建立自訂角色。

## <a name="create-and-manage-support-requests-in-the-azure-portal"></a>在 Azure 入口網站中建立及管理彈性工作

我們來看一個範例：您是 "Visual Studio MSDN Subscription" 訂用帳戶的擁有者。
Joe 是此訂用帳戶內某資源群組的擁有者，和你是對等關係，並且擁有訂用帳戶的讀取權限。
您希望提供 Joe 存取權，讓他能建立及管理此訂用帳戶的支援票證。

1. 首先移置訂用帳戶，您會在 [設定] 下看到使用者清單。 按一下使用者 Joe (他有訂用帳戶的讀取者權限)，然後指派自訂角色給他。

    ![新增角色](./media/create-manage-support-requests-using-access-control/add-role.png)

2. 按一下 [使用者] 刀鋒視窗中的 [新增]。 選取角色清單中的自訂角色 "Support Request Contributor"

    ![新增支援參與者角色](./media/create-manage-support-requests-using-access-control/add-support-contributor-role.png)

3. 選取角色名稱之後，按一下 [新增使用者] 並輸入 Joe 的電子郵件認證。 按一下 [選取]

    ![新增使用者](./media/create-manage-support-requests-using-access-control/add-users.png)

4. 按一下 [確定] 以繼續進行

    ![新增存取](./media/create-manage-support-requests-using-access-control/add-access.png)

5. 現在，剛才新增自訂角色 "Support Request Contributor" 的使用者會顯示在您擁有的訂用帳戶下

    ![新增的使用者](./media/create-manage-support-requests-using-access-control/user-added.png)

    當 Joe 登入入口網站時，他會看到已新增的訂用帳戶。

7. Joe 只要按一下 [說明及支援] 刀鋒視窗中的 [新增支援要求]，就能建立「Visual Studio Ultimate 含 MSDN」的支援要求

    ![新增支援要求](./media/create-manage-support-requests-using-access-control/new-support-request.png)

8. 按一下 [所有支援要求]，Joe 可以看到為此訂用帳戶建立的所有支援要求![案例詳細資料檢視](./media/create-manage-support-requests-using-access-control/case-details-view.png)

## <a name="remove-support-request-access-in-the-azure-portal"></a>在 Azure 入口網站中移除支援要求存取權

如同可以授與使用者建立及管理支援要求的存取權，您也可以移除使用者的存取權。
若要移除建立及管理支援要求的能力，請移至該訂用帳戶，按一下 [設定] 然後按一下使用者 (此範例中為 Joe)。
按一下角色名稱 [Support Request Contributor]，再按一下 [移除]

![移除支援要求存取權](./media/create-manage-support-requests-using-access-control/remove-support-request-access.png)

當 Joe 登入入口網站並嘗試建立支援要求時，他會遇到錯誤如下

![訂用帳戶錯誤-2](./media/create-manage-support-requests-using-access-control/subscription-error-2.png)

當 Joe 按一下 [所有支援要求] 時，他不會看到任何支援要求

![案例詳細資料檢視-2](./media/create-manage-support-requests-using-access-control/case-details-view-2.png)
