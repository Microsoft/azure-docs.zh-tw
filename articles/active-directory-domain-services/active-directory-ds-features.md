---
title: Azure Active Directory Domain Services：功能 | Microsoft Docs
description: Azure Active Directory 網域服務的功能
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 8d1c3eb3-1022-4add-a919-c98cc6584af1
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/30/2018
ms.author: maheshu
ms.openlocfilehash: a64401792fd034fde98fe1330340b1ffaa7dfcc1
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505407"
---
# <a name="azure-ad-domain-services"></a>Azure AD 網域服務
## <a name="features"></a>特性
下列功能適用於 Azure AD 網域服務的受控網域。

* **簡單的部署體驗：** 您只需使用數個按鍵，就能針對 Azure AD 目錄啟用 Azure AD 網域服務。 您的受控網域包含僅限雲端的使用者帳戶，以及從內部部署目錄同步的使用者帳戶。
* **支援加入網域的功能：** 您可以在可使用您的受控網域的 Azure 虛擬網路中輕鬆地將電腦加入網域。 在 Windows 用戶端和伺服器作業系統上的網域加入經驗，可針對 Azure AD 網域服務所服務的網域順暢地運作。 您也可以針對這類網域使用自動化網域加入工具。
* **每個 Azure AD 目錄都有一個網域執行個體：** 您可以針對每個 Azure AD 目錄建立單一 Active Directory 網域。
* **建立含有自訂名稱的網域：** 您可以使用 Azure AD 網域服務，來建立含有自訂名稱的網域 (例如 'contoso100.com')。 您可以使用已驗證或未驗證的網域名稱。 您也可以選擇性地使用 Azure AD 目錄提供的內建網域尾碼 (亦即 '*.onmicrosoft.com') 來建立網域。
* **與 Azure AD 整合：** 您不需要設定或管理對 Azure AD 網域服務的複寫。 Azure AD 網域服務中會自動提供來自 Azure AD 目錄的使用者帳戶、群組成員資格和使用者認證 (密碼)。 新使用者、群組，或 Azure AD 租用戶或內部部署目錄中的屬性變更，都會自動同步處理到 Azure AD 網域服務。
* **NTLM 和 Kerberos 驗證：** 利用對 NTLM 和 Kerberos 驗證的支援，您就能夠部署依賴 Windows 整合式驗證的應用程式。
* **使用公司認證/密碼：** Azure AD 租用戶中使用者的密碼可以與 Azure AD 網域服務搭配使用。 使用者可以使用其公司認證來將機器加入網域，以互動方式或透過遠端桌面登入，以及對受控網域進行驗證。
* **LDAP 繫結與 LDAP 讀取支援：** 您可以使用依賴 LDAP 繫結的應用程式，在 Azure AD 網域服務所服務的網域中驗證使用者。 此外，使用 LDAP 讀取作業來從目錄查詢使用者/電腦屬性的應用程式也可以針對 Azure AD 網域服務運作。
* **安全 LDAP (LDAPS)：** 透過安全 LDAP (LDAPS) 啟用目錄的存取。 預設可在虛擬網路內使用安全 LDAP 存取。 不過，您也可以選擇為網際網路啟用安全 LDAP 存取。
* **群組原則：** 您可以對每個使用者和電腦容器使用單一內建的 GPO，以針對使用者帳戶和已加入網域的電腦強制採用符合安全性原則的規範。 您也可以建立自己的自訂 GPO，然後將它們指派給自訂組織單位來[管理群組原則](active-directory-ds-admin-guide-administer-group-policy.md)。
* **管理 DNS：**「AAD DC 系統管理員」群組成員可以使用熟悉的 DNS 系統管理工具 (例如 DNS 系統管理 MMC 嵌入式管理單元)，來管理您受控網域的 DNS。
* **建立自訂的組織單位 (OU)：**「AAD DC 系統管理員」群組成員可以在受控網域中建立自訂的 OU。 這些使用者會被授與自訂 OU 的完整系統管理權限，讓他們可以在這些自訂的 OU 內新增或移除服務帳戶、電腦、群組等。
* **可在許多 Azure 全球區域中使用：** 請參閱[依區域提供的 Azure 服務](https://azure.microsoft.com/regions/#services/)頁面，以了解可使用 Azure AD 網域服務的 Azure 區域。
* 
  **高可用性：** Azure AD 網域服務可針對您的網域提供高可用性。 此功能提供了對於更高的服務運作時間以及失敗恢復措施的保證。 內建的狀況監控功能可提供失敗的自動補救措施，做法是啟動新的執行個體來取代失敗的執行個體，並為您的網域提供繼續執行的服務。
* **AD 帳戶鎖定保護：** 如果在 2 分鐘內使用 5 個無效的密碼，使用者帳戶會被鎖定 30 分鐘。 30 分鐘後，帳戶會自動解除鎖定。
* **使用熟悉的管理工具：** 您可以使用 Active Directory 管理中心或 PowerShell 的 Active Directory 等熟悉的 Windows Server Active Directory 管理工具，管理受控網域。
