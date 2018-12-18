---
title: Azure Active Directory Domain Services：啟用 Kerberos 限制委派 | Microsoft Docs
description: 在 Azure Active Directory Domain Services 受控網域上啟用 Kerberos 限制委派
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 938a5fbc-2dd1-4759-bcce-628a6e19ab9d
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/22/2018
ms.author: maheshu
ms.openlocfilehash: f50422caab2577ba24e3a7bc48b75e32d15d8d44
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505928"
---
# <a name="configure-kerberos-constrained-delegation-kcd-on-a-managed-domain"></a>在受控網域上設定 Kerberos 限制委派 (KCD)
許多應用程式都需要以使用者的登入身分存取資源。 Active Directory 支援稱為 Kerberos 委派的機制，可用於此使用案例。 再者，您可以限制委派，讓使用者的登入身分只能用來存取特定資源。 Azure AD Domain Services 受控網域與傳統Active Directory 網域不同，因為前者的權限更為嚴格且更為安全。

本文說明如何在 Azure AD Domain Services 受控網域上設定 Krberos 限制委派。

[!INCLUDE [active-directory-ds-prerequisites.md](../../includes/active-directory-ds-prerequisites.md)]

## <a name="kerberos-constrained-delegation-kcd"></a>Kerberos 限制委派 (KCD)
Kerberos 委派可讓帳戶模擬另一個安全性主體 (例如使用者) 以存取資源。 考慮一個可以使用者登入身分存取後端 Wb API 的 Web 應用程式。 在此案例中，Web 應用程式 (以服務帳戶身分或電腦/機器帳戶身分執行) 在存取資源 (後端 Web API) 時會模擬使用者。 Kerberos 委派不安全，因為它不會限制使用者登入身分中的模擬帳戶可存取的資源。

Kerberos 限制委派 (KCD) 會限制指定的伺服器可代表使用者存取的服務/資源。 傳統 KCD 需要網域系統管理員權限才能設定服務的網域帳戶，而且會將該帳戶限制在單一網域。

傳統 KCD 也有一些問題。 在舊版作業系統中，如果網域管理員為服務設定了帳戶型 KCD，服務管理員便沒有任何有用的方式可得知哪個前端服務被委派給他們所擁有的資源服務。 此外，任何可委派給資源服務的前端服務都會暴露一個潛在的受攻擊點。 若裝載前端服務的伺服器被入侵，而且它是設定為委派給資源服務，則資源服務可能也會被入侵。

> [!NOTE]
> 在 Azure AD 網域服務受控網域上，您沒有網域系統管理員權限。 因此，**無法在受控網域上設定傳統帳戶型 KCD**。 使用此文章中所述的資源型 KCD。 此機制也比較安全。
>
>

## <a name="resource-based-kcd"></a>資源型 KCD
服務系統管理員能夠從 Windows Server 2012 及更新版本設定其務的限制委派。 在此模型中，後端服務系統管理員可以允許或拒絕特定的前端服務使用 KCD。 此模型稱為**資源型 KCD**。

資源型 KCD 是使用 PowerShell 所設定的。 您將視模擬帳戶是電腦帳戶還是使用者帳戶/服務帳戶而定，使用 `Set-ADComputer` 或 `Set-ADUser` Cmdlet。

### <a name="configure-resource-based-kcd-for-a-computer-account-on-a-managed-domain"></a>為受控網域上的電腦帳戶設定資源型 KCD
假設您有在電腦 'contoso100-webapp.contoso100.com' 上執行的 Web 應用程式。 它需要以網域使用者的登入身分存取資源 ('contoso100-api.contoso100.com' 上執行的 Web API)。 以下是針對此案例設定資源型 KCD 的方式：

1. [建立自訂 OU](active-directory-ds-admin-guide-create-ou.md)。 您可以將管理這個自訂 OU 的權限委派給受控網域內的使用者。
2. 將兩種虛擬機器 (執行 Web 應用程式的虛擬機器和執行 Web API 的虛擬機器) 都加入受控網域中。 請在自訂 OU 內建立這些電腦帳戶。
3. 現在，使用下列 PowerShell 命令來設定資源型 KCD：

```powershell
$ImpersonatingAccount = Get-ADComputer -Identity contoso100-webapp.contoso100.com
Set-ADComputer contoso100-api.contoso100.com -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
```

> [!NOTE]
> Web 應用程式和 Web API 的電腦帳戶必須在您具備設定資源型 KCD 之權限的自訂 OU 中。 您無法為在內建之 'AAD DC Computers' 容器中的電腦帳戶設定資源型 KCD。
>

### <a name="configure-resource-based-kcd-for-a-user-account-on-a-managed-domain"></a>為受控網域上的使用者帳戶設定資源型 KCD
假設您有以服務帳戶 'appsvc' 執行的 Web 應用程式，且它需要以網域使用者登入身分存取資源 (以服務帳戶 - 'backendsvc' 執行的 Web API)。 以下是針對此案例設定資源型 KCD 的方式。

1. [建立自訂 OU](active-directory-ds-admin-guide-create-ou.md)。 您可以將管理這個自訂 OU 的權限委派給受控網域內的使用者。
2. 將執行後端 Web API/資源的虛擬機器加入受控網域中。 請在自訂 OU 內建立其電腦帳戶。
3. 建立用來執行自訂 OU 內 Web 應用程式的服務帳戶 (例如 'appsvc')。
4. 現在，使用下列 PowerShell 命令來設定資源型 KCD：

```powershell
$ImpersonatingAccount = Get-ADUser -Identity appsvc
Set-ADUser backendsvc -PrincipalsAllowedToDelegateToAccount $ImpersonatingAccount
```

> [!NOTE]
> 後端 Web API 的電腦帳戶和服務帳戶都必須在您具備設定資源型 KCD 之權限的自訂 OU 中。 您無法為在內建之 'AAD DC Computers' 容器中的電腦帳戶，或在內建之 'AAD DC Users' 容器中的使用者帳戶，設定資源型 KCD。 因此，您無法使用從 Azure AD 同步處理的使用者帳戶來設定資源型 KCD。
>

## <a name="related-content"></a>相關內容
* [Azure AD Domain Services - 入門指南](active-directory-ds-getting-started.md)
* [Kerberos 限制委派概觀](https://technet.microsoft.com/library/jj553400.aspx)
