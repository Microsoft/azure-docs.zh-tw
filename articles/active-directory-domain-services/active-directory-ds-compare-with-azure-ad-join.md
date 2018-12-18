---
title: 比較 Azure AD Join 與 Azure Active Directory Domain Services | Microsoft Docs
description: 在 Azure AD Join 與 Azure AD Domain Services 之間做決定
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 31a71d36-58c1-4839-b958-80da0c6a77eb
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/26/2017
ms.author: maheshu
ms.openlocfilehash: e4357738417a202d5dd5a97907b5240cf855f395
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39504109"
---
# <a name="choose-between-azure-active-directory-join-and-azure-active-directory-domain-services"></a>Azure Active Directory 聯結和 Azure Active Directory Domain Services 之間的選擇
本文將說明 Azure Active Directory (AD) 聯結和 Azure AD Domain Services 之間的差異，並協助您根據您的使用案例進行選擇。

## <a name="azure-ad-registered-and-azure-ad-joined-devices"></a>註冊的 Azure AD 和 Azure AD 的聯結裝置
Azure AD 可讓您管理組織所用裝置的身分識別，並控制這些裝置對公司資源的存取。 使用者可以向 Azure AD 註冊他們個人 (自備) 的裝置，以取得裝置的身分識別。 接著，當使用者登入 Azure AD 並使用裝置來存取受保護的資源時，Azure AD 就可以驗證裝置。 此外，您可以使用 Microsoft Intune 等行動裝置管理 (MDM) 軟體來管理裝置。 這項功能可讓您限制受控和符合原則的裝置對敏感資源的存取權。

您也可以將組織所擁有的裝置新增至 Azure AD。 這項機制提供的權益與在 Azure AD 中註冊個人裝置相同。 此外，使用者可以使用他們公司的認證登入裝置。 Azure AD 聯結裝置提供下列優點：
* 單一登入 (SSO) 至 Azure AD 所保護的應用程式
* 在跨裝置之間進行使用者設定的企業符合原則漫遊。
* 使用您公司的認證存取商務 Windows 市集。
* Windows Hello 企業版
* 對與公司原則相容的裝置限制應用程式和資源存取權。

| **裝置類型** | **裝置平台** | **機制** |
|:---| --- | --- |
| 個人裝置 | Windows 10、iOS、Android、Mac OS | 已註冊的 Azure AD |
| 公司所擁有但未聯結內部部署 AD 的裝置 | Windows 10 | 已聯結的 Azure AD |
| 公司所擁有且已聯結內部部署 AD 的裝置 | Windows 10 | 已聯結的混合式 Azure AD |

在 Azure AD 已聯結或已註冊的裝置上，使用者驗證會使用以最新 OAuth/OpenID Connect 為基礎的通訊協定。 這些通訊協定皆設計為可透過網際網路運作，且非常適合行動裝置使用，讓使用者可以從任何位置存取公司資源。


## <a name="domain-join-to-azure-ad-domain-services-managed-domains"></a>對 Azure AD Domain Services 受控網域進行網域加入
Azure AD Domain Services 在 Azure 虛擬網路中提供受控 AD 網域。 您可以使用傳統的網域加入機制，將機器加入此受控網域。 Windows 用戶端 (Windows 7、Windows 10) 和 Windows Server 機器都可加入受控網域。 此外，您也可以將 Linux 和 Mac OS 機器加入受控網域。 當您將機器加入 AD 網域時，使用者就可以使用他們的公司認證來登入機器。 這些機器可以使用群組原則來管理，因此可強制與您組織的安全性原則相容。

在已加入網域的機器上，使用者驗證會使用 NTLM 或 Kerberos 驗證通訊協定。 已加入網域的機器需讓受控網域中的網域控制站看見，才能執行使用者驗證。 因此，加入網域的機器必須在與受控網域相同的虛擬網路上。 或者，加入網域的機器必須透過對等互連的虛擬網路或站對站 VPN/ExpressRoute 連線來連線到受控網域。 因此，這項機制不適合行動裝置，或從公司網路外部連線到資源的裝置。

> [!NOTE]
> 技術上來說，透過站對站 VPN 或 ExpressRoute 連線，來將內部部署用戶端工作站連線到受控網域是可行的。 不過，對於使用者裝置，我們強烈建議您在 Azure AD 註冊裝置 (個人裝置)，或將裝置聯結到 Azure AD (公司裝置)。 這項機制在網際網路上的效果更好，並可讓使用者在任何地方使用。 Azure AD Domain Services 適用於部署在 Azure 虛擬網路 (部署您應用程式的地方) 上的 Windows 或 Linux 伺服器虛擬機器。


## <a name="summary---key-differences"></a>摘要 - 主要差異
| **層面** | **Azure AD Join** | **Azure AD 網域服務** |
|:---| --- | --- |
| 控制裝置的機制 | Azure AD | Azure AD Domain Services 的受控網域 |
| 在目錄中的表示法 | Azure AD 目錄中的裝置物件。 | AAD DS 受控網域中的電腦物件。 |
| 驗證 | 以 OAuth/OpenID Connect 為基礎的通訊協定 | Kerberos、NTLM 通訊協定 |
| 管理性 | Intune 等行動裝置管理 (MDM) 軟體 | 群組原則 |
| 網路 | 透過網際網路運作 | 機器需要在與受控網域相同的虛擬網路上。|
| 適用於... | 使用者行動裝置或桌面裝置 | 已部署在 Azure 中的伺服器虛擬機器 |


## <a name="next-steps"></a>後續步驟
### <a name="learn-more-about-azure-ad-domain-services"></a>深入了解 Azure AD Domain Services
* [Azure AD Domain Services 概觀](active-directory-ds-overview.md)
* [特性](active-directory-ds-features.md)
* [部署案例](active-directory-ds-scenarios.md)
* [了解 Azure AD Domain Services 是否適合您的使用案例](active-directory-ds-comparison.md)
* [了解 Azure AD Domain Services 如何與 Azure AD 目錄同步處理](active-directory-ds-synchronization.md)

### <a name="learn-more-about-azure-ad-join"></a>深入了解 Azure AD Join
* [Azure Active Directory 中的裝置管理簡介](../active-directory/device-management-introduction.md)

### <a name="get-started-with-azure-ad-domain-services"></a>開始使用 Azure AD 網域服務
* [使用 Azure 入口網站來啟用 Azure AD Domain Services](active-directory-ds-getting-started.md)
