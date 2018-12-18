---
title: 啟用所有 Azure 系統管理員的 MFA
description: 啟用全域系統管理員的指引
ms.service: security
author: barclayn
manager: mbaldwin
editor: TomSh
ms.topic: article
ms.date: 03/20/2018
ms.author: barclayn
ms.openlocfilehash: a7a08c54fe0c59e1e100e1c46e7a640da0692077
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32181009"
---
# <a name="enforce-multi-factor-authentication-mfa-for-subscription-administrators"></a>對訂用帳戶系統管理員強制執行多重要素驗證 (MFA)。

在建立您的系統管理員時 (包括全域系統管理員帳戶)，請務必使用非常強力的驗證方法。

您可以視需要將特定的系統管理員角色指派給 IT 人員的使用者帳戶，例如 Exchange 系統管理員或密碼系統管理員，以便執行例行管理作業。
此外，為系統管理員啟用 [Azure 多重要素驗證 (MFA)](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication) 可以加強防護使用者登入和交易的安全。 Azure MFA 也可協助 IT 降低遭入侵的認證能夠存取公司資料的可能性。

例如︰對您的使用者強制執行 Azure MFA，並將它設定為使用電話或簡訊進行驗證。 如果使用者的認證遭到入侵，攻擊者將無法存取任何資源，因為他無法存取使用者的電話。 未新增額外身分識別保護層的組織會更容易受到認證竊取攻擊，這可能會導致資料洩漏。

想要將整個驗證控制保留於內部部署的組織有一個替代方法，就是使用 [Azure Multi-factor Authentication Server](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-server) (也稱為「MFA 內部部署」)。 使用此方法，您仍可強制執行 Multi-Factor Authentication，同時保留 MFA 伺服器內部部署。

若要查看您的組織中有哪些人擁有系統管理權限，您可以使用下列 Microsoft Azure AD V2 PowerShell 命令進行確認：

```azurepowershell-interactive
Get-AzureADDirectoryRole | Where { $_.DisplayName -eq "Company Administrator" } | Get-AzureADDirectoryRoleMember | Ft DisplayName
```

## <a name="enabling-mfa"></a>啟用 MFA

在繼續前先檢閱 [MFA](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-whats-next) 的運作方式。

只要您的使用者具有包含 Azure Multi-Factor Authentication 的授權，您就不需要手動開啟 Azure MFA。 您可以開始要求對個別使用者進行雙步驟驗證。 可啟用 Azure MFA 的授權如下︰

- Azure Multi-Factor Authentication
- Azure Active Directory Premium
- Enterprise Mobility + Security

## <a name="turn-on-two-step-verification-for-users"></a>對使用者開啟雙步驟驗證

使用[如何要求使用者或群組進行雙步驟驗證](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-user-states)中所列的其中一個程序，開始使用 Azure MFA。 您可以選擇對所有登入強制執行雙步驟驗證，也可以建立條件式存取原則，只有在與您有關時要求雙步驟驗證。

