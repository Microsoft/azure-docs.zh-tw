---
title: 在 Azure Active Directory 中新增 Azure Stack 租用戶帳戶 | Microsoft Docs
description: 在部署 Microsoft Azure Stack 開發套件之後，您將需要建立至少一個租用戶使用者帳戶，以便可以瀏覽租用戶入口網站。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: a75d5c88-5b9e-4e9a-a6e3-48bbfa7069a7
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/17/2018
ms.author: jeffgilb
ms.reviewer: unknown
ms.openlocfilehash: 9a4d7200a2bc2445fcdfefc0332d67a045b5a2e1
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038012"
---
# <a name="add-a-new-azure-stack-tenant-account-in-azure-active-directory"></a>在 Azure Active Directory 中新增 Azure Stack 租用戶帳戶

在[部署 Azure Stack 開發套件](azure-stack-run-powershell-script.md)之後，您將需要租用戶使用者帳戶，以便可以瀏覽租用戶入口網站並測試您的供應項目與方案。 您可以[使用 Azure 入口網站](#create-an-azure-stack-tenant-account-using-the-azure-portal)或[使用 PowerShell](#create-an-azure-stack-tenant-account-using-powershell) 來建立租用戶帳戶。

## <a name="create-an-azure-stack-tenant-account-using-the-azure-portal"></a>使用 Azure 入口網站建立 Azure Stack 租用戶帳戶

您必須擁有 Azure 訂用帳戶，才能使用 Azure 入口網站。

1. 登入[Azure](https://portal.azure.com)。
2. 在左側導覽列中，選取 **Active Directory** 並切換到您想用於 Azure Stack 的目錄，或建立一個新的目錄。
3. 選取 [Azure Active Directory] > [使用者] > [新增使用者]。

    ![使用者 - 已醒目提示 [新增使用者] 的 [所有使用者] 頁面](media/azure-stack-add-new-user-aad/new-user-all-users.png)

4. 在 [使用者] 頁面上，填妥必要資訊。

    ![新增使用者，包含使用者資訊的 [使用者] 頁面](media/azure-stack-add-new-user-aad/new-user-user.png)

    - **名稱 (必要)。** 新使用者的姓氏與名字。 例如，Mary Parker。
    - **使用者名稱 (必要)。** 新使用者的使用者名稱。 例如： mary@contoso.com。
        使用者名稱的網域部分必須使用初始預設網域名稱 (<_yourdomainname_>.onmicrosoft.com)，或自訂網域名稱，例如 contoso.com。 如需如何建立自訂網域名稱的詳細資訊，請參閱[將自訂網域名稱新增至 Azure Active Directory](../active-directory/fundamentals/add-custom-domain.md)。
    - **設定檔。** (選擇性) 您可以新增更多有關使用者的資訊。 您也可以稍後新增使用者資訊。 如需新增使用者資訊的詳細資訊，請參閱[如何新增或變更使用者設定檔資訊](../active-directory/fundamentals/active-directory-users-profile-azure-portal.md)。
    - **目錄角色。**  選擇 [使用者]。

5. 勾選 [顯示密碼]，並且複製在 [密碼] 方塊中提供的自動產生密碼。 您在初始登入程序中需要此密碼。

6. 選取 [建立] 。

    使用者已建立並新增至您的 Azure AD 租用戶。

7. 以新的帳戶登入 Microsoft Azure 入口網站。 在系統提示時變更密碼。
8. 以新的帳戶登入 `https://portal.local.azurestack.external`，查看租用戶入口網站。

## <a name="create-an-azure-stack-tenant-account-using-powershell"></a>使用 PowerShell 建立 Azure Stack 租用戶帳戶

如果您沒有 Azure 訂用帳戶，就無法使用 Azure 入口網站來新增租用戶使用者帳戶。 在此情況下，您可以改為使用適用於 Windows PowerShell 的 Azure Active Directory 模組。

> [!NOTE]
> 如果您使用 Microsoft 帳戶 (Live ID) 來部署 Azure Stack 開發套件，就無法使用 AAD PowerShell 來建立租用戶帳戶。 
> 
> 

1. 安裝[適用於 IT 專業人員的 Microsoft Online Services 登入小幫手 RTW](https://www.microsoft.com/en-us/download/details.aspx?id=41950)。
2. 安裝[適用於 Windows PowerShell (64 位元版本) 的 Azure Active Directory 模組](http://go.microsoft.com/fwlink/p/?linkid=236297) \(英文\)，並將它開啟。
3. 執行下列 Cmdlet：

    ```powershell
    # Provide the AAD credential you use to deploy Azure Stack Development Kit

            $msolcred = get-credential

    # Add a tenant account "Tenant Admin <username>@<yourdomainname>" with the initial password "<password>".

            connect-msolservice -credential $msolcred
            $user = new-msoluser -DisplayName "Tenant Admin" -UserPrincipalName <username>@<yourdomainname> -Password <password>
            Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberType User -RoleMemberObjectId $user.ObjectId

    ```

1. 以新的帳戶登入 Microsoft Azure。 在系統提示時變更密碼。
2. 以新的帳戶登入 `https://portal.local.azurestack.external`，查看租用戶入口網站。

