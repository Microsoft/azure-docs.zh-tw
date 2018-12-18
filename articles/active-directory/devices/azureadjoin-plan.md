---
title: 適用於 Azure AD Join 的使用案例和部署考量| Microsoft Docs
description: 說明系統管理員如何為其使用者 (員工、學生、其他使用者) 設定 Azure AD Join。 其中也會討論使用 Azure AD Join 時出現的各種真實案例。
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: mtillman
editor: ''
tags: azure-classic-portal
ms.component: devices
ms.assetid: 81d4461e-21c8-4fdd-9076-0e4991979f62
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2018
ms.author: markvi
ms.openlocfilehash: a145b4184fbebd9c4f447face1c47bec20c0ec32
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39415631"
---
# <a name="usage-scenarios-and-deployment-considerations-for-azure-ad-join"></a>適用於 Azure AD Join 的使用案例和部署考量
## <a name="usage-scenarios-for-azure-ad-join"></a>適用於 Azure AD Join 的使用案例
### <a name="scenario-1-businesses-largely-in-the-cloud"></a>案例 1：主要位於雲端的企業
如果您目前是在雲端為貴公司操作和管理身分識別，或者即將轉向雲端，則可從 Azure Active Directory Join (Azure AD Join) 獲益。 您可以使用在 Azure AD 中建立的帳戶來登入 Windows 10。 透過[初次執行體驗 (FRX) 程序](azuread-joined-devices-frx.md)或從[設定功能表](../user-help/device-management-azuread-joined-devices-setup.md)加入 Azure AD，您的使用者可將其電腦加入 Azure AD。  您的使用者也可以在瀏覽器或 Office 應用程式中，享受經由單一登入 (SSO) 存取雲端資源，例如 Office 365。

### <a name="scenario-2-educational-institutions"></a>案例 2：教育機構
教育機構通常有兩種使用者類型：教職員和學生。 教職成員會被視為組織中的長期成員。 為他們建立內部部署帳戶是比較好的做法。 但學生是組織中較短期的成員，因而可將其帳戶放在 Azure AD 中管理。 這表示可以將目錄範圍推送至雲端，而不是儲存在內部部署 這也表示學生可以使用其 Azure AD 帳戶登入 Windows，並在 Office 應用程式中存取 Office 365 資源。

### <a name="scenario-3-retail-businesses"></a>案例 3：零售業
零售業擁有季節性工作者和長期員工。 您通常會為較長期的全職員工建立內部部署帳戶，讓他們能夠使用已加入網域的電腦。 但季節性工作者是組織中較短期的成員，因而適合使用可更輕易四處移動使用者授權的帳戶來管理。 當您在具有 Office 365 授權的雲端中建立這些使用者帳戶時，這些使用者能夠因為使用 Azure AD 帳戶登入 Windows 和 Office 應用程式而獲益，同時在他們離開公司之後對於他們的授權保有更多彈性。

### <a name="scenario-4-additional-scenarios"></a>案例 4：其他案例
除了先前討論的案例，您還可以藉由讓使用者將其裝置加入 Azure AD，因為簡化的加入體驗、有效率的裝置管理、自動的行動裝置管理註冊，以及單一登入 Azure AD 和內部部署資源而獲益。  

## <a name="deployment-considerations-for-azure-ad-join"></a>適用於 Azure AD Join 的部署考量
### <a name="enable-your-users-to-join-a-company-owned-device-directly-to-azure-ad"></a>讓使用者可將公司擁有的裝置直接加入 Azure AD
企業可以為合作夥伴公司和組織提供僅限雲端的帳戶。 這些合作夥伴之後就能使用單一登入，輕易地存取公司的應用程式和資源。 此案例適用於主要存取雲端資源的使用者，例如 Office 365 或 SaaS 應用程式，而這類資源會依賴 Azure AD 進行驗證。

### <a name="prerequisites"></a>必要條件
**在企業層級 (系統管理員)**

* Azure 訂用帳戶搭配 Azure Active Directory  

**在使用者層級**

* Windows 10 (專業版和企業版)

### <a name="administrator-tasks"></a>系統管理員工作
* [設定裝置註冊](device-management-azure-portal.md)

### <a name="user-tasks"></a>使用者工作
* [設定期間使用 Azure AD 設定新的 Windows 10 裝置](azuread-joined-devices-frx.md)
* [從設定功能表使用 Azure AD 設定 Windows 10 裝置](../user-help/device-management-azuread-registered-devices-windows10-setup.md)
* [將個人的 Windows 10 裝置加入您的組織](../user-help/device-management-azuread-joined-devices-setup.md)

## <a name="enable-byod-in-your-organization-for-windows-10"></a>在組織中針對 Windows 10 啟用 BYOD
您可以設定使用者和員工，使用其個人的 Windows 裝置 (BYOD) 來存取公司應用程式和資源。 您的使用者可將其 Azure AD 帳戶 (工作或學校帳戶) 新增到個人的 Windows 裝置，以安全且相容的方式來存取資源。

### <a name="prerequisites"></a>必要條件
**在企業層級 (系統管理員)**

* Azure AD 訂用帳戶

**在使用者層級**

* Windows 10 (專業版和企業版)

### <a name="administrator-tasks"></a>系統管理員工作
* [設定裝置註冊](device-management-azure-portal.md)

### <a name="user-tasks"></a>使用者工作
* [將個人的 Windows 10 裝置加入您的組織](../user-help/device-management-azuread-joined-devices-setup.md)

## <a name="next-steps"></a>後續步驟

- [裝置管理](overview.md)

