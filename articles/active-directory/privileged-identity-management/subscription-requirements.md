---
title: 使用 PIM 的訂用帳戶需求 - Azure | Microsoft Docs
description: 說明要使用 Azure AD Privileged Identity Management (PIM) 的訂用帳戶和授權需求。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: markwahl-msft
ms.assetid: 34367721-8b42-4fab-a443-a2e55cdbf33d
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: pim
ms.date: 06/01/2017
ms.author: rolyon
ms.custom: pim
ms.openlocfilehash: 1554895dcba0c09a3a2e19c284a1cd6f0416cfe1
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190605"
---
# <a name="subscription-requirements-to-use-pim"></a>使用 PIM 的訂用帳戶需求

Azure AD Privileged Identity Management 可在 Azure AD 的 Premium P2 Edition 中使用。 如需 P2 的其他功能資訊以及其與 Premium P1 的比較詳細資訊，請參閱 [Azure Active Directory 版本](../active-directory-editions.md)。

>[!NOTE]
Azure Active Directory (Azure AD) Privileged Identity Management 處於預覽狀態時，不會對嘗試服務的租用戶進行任何授權檢查。  現在，Azure AD Privileged Identity Management 已公開上市，必須存在試用或付費訂用帳戶，租用戶才能在 2016 年 12 月之後繼續使用 Privileged Identity Management。
  

## <a name="confirm-your-trial-or-paid-subscription"></a>確認試用或付費訂用帳戶

如果您不確定您的組織是否有試用或購買的訂用帳戶時，您可以使用適用於 Windows PowerShell V1 的 Azure Active Directory 模組中包含的命令，檢查您的租用戶中是否有訂用帳戶。 
1. 開啟 PowerShell 視窗。
2. 輸入 `Connect-MsolService` 以在您的租用戶中以使用者身分進行驗證。
3. 輸入 `Get-MsolSubscription | ft SkuPartNumber,IsTrial,Status`。

此命令會擷取您租用戶中的訂用帳戶清單。 如果未傳回任何資料行，您必須取得 Azure AD Premium P2 試用版、購買 Azure AD Premium P2 訂用帳戶或 EMS E5 訂用帳戶，才能使用 Azure AD Privileged Identity Management。  若要取得試用版並開始使用 Azure AD Privileged Identity Management，請參閱[開始使用 Azure AD Privileged Identity Management](pim-getting-started.md)。

如果此命令傳回一個資料行，其中的 SkuPartNumber 為 "AAD_PREMIUM_P2" 或 "EMSPREMIUM" 且 IsTrial 為 "True"，這表示 Azure AD Premium P2 試用版已存在於租用戶。  如果訂用帳戶狀態為未啟用，而且您未購買 Azure AD Premium P2 或 EMS E5 訂用帳戶，則您必須購買 Azure AD Premium P2 訂用帳戶或 EMS E5 訂用帳戶，才能繼續使用 Azure AD Privileged Identity Management。

您可以透過 [Microsoft Enterprise 合約](https://www.microsoft.com/en-us/licensing/licensing-programs/enterprise.aspx)、[Open Volume 授權方案](https://www.microsoft.com/en-us/licensing/licensing-programs/open-license.aspx)與[雲端解決方案提供者方案](https://partner.microsoft.com/cloud-solution-provider)取得 Azure AD Premium P2。 Azure 和 Office 365 訂閱者也可以線上購買 Azure AD Premium P2。  如需關於 Azure AD Premium 價格或線上訂購方式的詳細資訊，請瀏覽 [Azure Active Directory 價格](https://azure.microsoft.com/pricing/details/active-directory/)。

## <a name="azure-ad-privileged-identity-management-is-not-available-in-tenant"></a>Azure AD Privileged Identity Management 不適用於租用戶

在下列情況下，Azure AD Privileged Identity Management 無法再適用於租用戶︰
- 您的組織使用預覽狀態的 Azure AD Privileged Identity Management，且未購買 Azure AD Premium P2 訂用帳戶或 EMS E5 訂用帳戶。
- 您的組織擁有的 Azure AD Premium P2 或 EMS E5 試用版已過期。
- 您的組織購買的訂用帳戶已過期。

當 Azure AD Premium P2 訂用帳戶或 EMS E5 訂用帳戶過期，或使用預覽狀態 Azure AD Privileged Identity Management 的組織未取得 Azure AD Premium P2 或 EMS E5 訂用帳戶時：

- Azure AD 角色的永久角色指派不會受到影響。
- 使用者無法再利用 Azure 入口網站中的 Azure AD Privileged Identity Management 擴充以及 Azure AD Privileged Identity Management 的 Graph API Cmdlet 和 PowerShell 介面，來啟動特殊權限的角色、管理特殊權限的存取權，或執行特殊權限角色的存取檢閱。
- Azure AD 角色的合格角色指派將會遭到移除，因為使用者將無法再啟動特殊權限的角色。
- Azure AD 角色的任何進行中存取檢閱將會結束，而且 Azure AD Privileged Identity Management 組態設定將會遭到移除。
- Azure AD Privileged Identity Management 將不再傳送有關角色指派變更的電子郵件。

## <a name="next-steps"></a>後續步驟

- [開始使用 PIM](pim-getting-started.md)
- [您可以在 PIM 中管理的 Azure AD 目錄角色](pim-roles.md)
