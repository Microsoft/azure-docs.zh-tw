---
title: 如何管理已加入 Azure AD 的裝置上的本機系統管理員群組 | Microsoft Docs
description: 了解如何將 Azure 角色指派給 Windows 裝置的本機系統管理員群組。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
ms.assetid: 54e1b01b-03ee-4c46-bcf0-e01affc0419d
ms.service: active-directory
ms.component: devices
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2018
ms.author: markvi
ms.reviewer: ravenn
ms.openlocfilehash: cde364cb5231c1cc0b1947da35994862cf45b571
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43042156"
---
# <a name="how-to-manage-the-local-administrators-group-on-azure-ad-joined-devices"></a>如何管理已加入 Azure AD 的裝置上的本機系統管理員群組

若要管理 Windows 裝置，您必須是本機系統管理員群組的成員。 在 Azure Active Directory (Azure AD) 加入程序中，Azure AD 會在裝置上更新此群組的成員資格。 您可以自訂成員資格更新，以符合自己的商務需求。 舉例來說，如果您想要讓技術服務人員能夠在裝置上執行需要系統管理員權限的工作，成員資格更新就有其效益。

本文說明成員資格更新的運作方式，以及如何在 Azure AD Join 期間加以自訂。 本文的內容不適用於**混合式** Azure AD Join。


## <a name="how-it-works"></a>運作方式

當您使用 Azure AD Join 連接 Windows 裝置與 Azure AD 時，Azure AD 會將下列安全性準則新增至裝置的本機系統管理員群組：

- Azure AD 全域管理員角色
- Azure AD 裝置管理員角色 
- 執行 Azure AD Join 的使用者   

藉由將 Azure AD 角色新增至本機系統管理員群組，您可以直接更新能夠隨時在 Azure AD 中管理裝置的使用者，而無須對裝置進行任何修改。 目前，您無法將群組指派給系統管理員角色。
Azure AD 也會將 Azure AD 裝置管理員角色新增至本機系統管理員群組，以支援最低權限 (PoLP) 的準則。 除了全域管理員以外，您也可以讓*僅*被指派裝置管理員角色的使用者管理裝置。 


## <a name="manage-the-global-administrators-role"></a>管理全域管理員角色

若要檢視及更新全域管理員角色的成員資格，請參閱：

- [檢視 Azure Active Directory 中的所有系統管理員角色成員](../users-groups-roles/directory-manage-roles-portal.md)

- [在 Azure Active Directory 中將使用者指派給系統管理員角色](../fundamentals/active-directory-users-assign-role-azure-portal.md)


## <a name="manage-the-device-administrator-role"></a>管理裝置管理員角色 

在 Azure 入口網站中，您可以在 [裝置] 頁面上管理裝置管理員角色。 若要開啟 [裝置] 頁面：

1. 以全域管理員或裝置管理員身分登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側導覽列上，按一下 [Azure Active Directory]。 
3. 在 [管理] 區段中，按一下 [裝置]。
4. 在 [裝置] 頁面上，按一下 [裝置設定]。

若要修改裝置管理員角色，請設定**加入 Azure AD 的裝置上的其他本機系統管理員**。  

![其他本機系統管理員](./media/assign-local-admin/10.png)

 
裝置管理員會指派給所有加入 Azure AD 的裝置。 您無法將裝置管理員的範圍設定為一組特定的裝置。 更新裝置管理員角色不一定會對受影響的使用者產生直接的影響。 就使用者已登入的裝置而言，權限會在下列時機進行更新：
     

- 當使用者登出時。
- 經過 4 小時後，當新的主要重新整理權杖發出時。 




## <a name="manage-regular-users"></a>管理一般使用者

根據預設，Azure AD 會將執行 Azure AD Join 的使用者新增至裝置的系統管理員群組。 如果您想要防止一般使用者成為本機系統管理員，您會有下列選項：

- [Windows Autopilot](https://docs.microsoft.com/windows/deployment/windows-autopilot/windows-10-autopilot) - Windows Autopilot 可讓您選擇防止執行加入的主要使用者成為本機系統管理員。 您可以藉由[建立 Autopilot 設定檔](https://docs.microsoft.com/intune/enrollment-autopilot#create-an-autopilot-deployment-profile)來完成這項作業。
 
- [大量註冊](https://docs.microsoft.com/intune/windows-bulk-enroll) - 在大量註冊的內容中執行的 Azure AD Join，會在自動建立的使用者內容中執行。 在裝置加入後登入的使用者不會新增至系統管理員群組。   



## <a name="manually-elevate-a-user-on-a-device"></a>手動提高使用者在裝置上的權限 

除了使用 Azure AD Join 程序以外，您也可以手動提高一般使用者的權限，使其成為一個特定裝置的本機系統管理員。 若要執行此步驟，您必須已是本機系統管理員群組的成員。 

自 **Windows 10 1709** 版本起，您可以藉由選取 [新增工作或學校使用者]，從 [設定] -> [帳戶] -> [其他使用者] 執行這項工作。
 
此外，您也可以使用命令提示字元來新增使用者：

- 如果您的租用戶使用者是從內部部署 Active Directory 同步處理的，請使用 `net localgroup administrators /add “Contoso\username”`。

- 如果您的租用戶使用者是在 Azure AD 中建立的，請使用 `net localgroup administrators /add “AzureAD\UserUpn”`


## <a name="considerations"></a>考量 

您無法將群組指派給裝置管理員角色，而只能指派個別使用者。

裝置管理員會指派給所有加入 Azure AD 的裝置。 這些管理員無法以一組特定的裝置為範圍。

當您從裝置管理員角色中移除使用者時，這些使用者只要仍在裝置上處於登入狀態，就仍具有本機系統管理員權限。 權限會在下一次登入期間撤銷，或在經過 4 小時後，在新的主要重新整理權杖發出時撤銷。



## <a name="next-steps"></a>後續步驟

- 若要取得在 Azure 入口網站中管理裝置的概觀，請參閱[使用 Azure 入口網站來管理裝置](device-management-azure-portal.md)

- 若要深入了解裝置型條件式存取，請參閱[設定 Azure Active Directory 裝置型條件式存取原則](../conditional-access/require-managed-devices.md)。


