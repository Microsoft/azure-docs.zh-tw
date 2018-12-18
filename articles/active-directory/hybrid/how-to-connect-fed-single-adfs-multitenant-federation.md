---
title: 將多個 Azure AD 和單一 AD FS 建立同盟關係 | Microsoft Docs
description: 在本文件中，您將學習如何使用多個 Azure AD 和單一 AD FS 建立同盟。
keywords: 聯盟, ADFS, AD FS, 多個租用戶, 單一 AD FS, 一個 ADFS, 多個租用戶同盟, 多樹系 adfs, aad 連線, 同盟, 跨租用戶同盟
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 07/17/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: a607591dd6b60b80c670bdd08b6f7c320aadc17f
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46301406"
---
# <a name="federate-multiple-instances-of-azure-ad-with-single-instance-of-ad-fs"></a>將多個 Azure AD 執行個體和單一 AD FS 執行個體建立同盟

如果樹系之間有雙向信任，單一高可用 AD FS 伺服器陣列就可以聯合多個樹系。 這些樹系可能會也可能不會對應到相同的 Azure Active Directory。 本文提供有關如何在單一 AD FS 部署和同步至不同 Azure AD 的多個樹系之間建立同盟的指示。

![多租用戶與單一 AD FS 的同盟關係](./media/how-to-connect-fed-single-adfs-multitenant-federation/concept.png)
 
> [!NOTE]
> 在此案例中不支援裝置回寫和自動加入裝置。

> [!NOTE]
> Azure AD Connect 無法用來設定此案例中的同盟關係，因為 Azure AD Connect 可以在單一 Azure AD 中設定網域的同盟關係。

## <a name="steps-for-federating-ad-fs-with-multiple-azure-ad"></a>將 AD FS 和多個 Azure AD 建立同盟的步驟

請將 Azure Active Directory contoso.onmicrosoft.com 中的網域 contoso.com 已經與 contoso.com 內部部署 Active Directory 環境中安裝的 AD FS 內部部署建立同盟這點列入考慮。 Fabrikam.com 是 fabrikam.onmicrosoft.com Azure Active Directory 中的網域。

## <a name="step-1-establish-a-two-way-trust"></a>步驟 1︰建立雙向信任
 
若要 contoso.com 中的 AD FS 能夠驗證 fabrikam.com 中的使用者，需要 contoso.com 和 fabrikam.com 之間的雙向信任。 請依照本[文章](https://technet.microsoft.com/library/cc816590.aspx)的指導方針建立雙向信任。
 
## <a name="step-2-modify-contosocom-federation-settings"></a>步驟 2︰修改 contoso.com 同盟設定 
 
與 AD FS 建立同盟的單一網域的預設簽發者為 "http://ADFSServiceFQDN/adfs/services/trust"，例如 “http://fs.contoso.com/adfs/services/trust”。 Azure Active Directory 要求每個同盟網域都需要擁有唯一簽發者。 由於相同的 AD FS 會建立兩個網域的同盟，因此必須修改簽發者值，所以該值對於 AD FS 與 Azure Active Directory 同盟的每個網域而言是唯一的值。 
 
在 AD FS 伺服器上，開啟 Azure AD PowerShell 並執行下列步驟︰
 
連線至 Azure Active Directory，其中包含網域 contoso.com Connect-MsolService Update contoso.com 的同盟設定 Update-MsolFederatedDomain -DomainName contoso.com –SupportMultipleDomain
 
在網域同盟設定中的簽發者會變更為 "http://contoso.com/adfs/services/trust"，並會針對 Azure AD 信賴憑證者信任新增發行宣告規則，以根據 UPN 尾碼發出正確的 issuerId 值。
 
## <a name="step-3-federate-fabrikamcom-with-ad-fs"></a>步驟 3︰建立 fabrikam.com 與 AD FS 的同盟
 
在 Azure AD powershell 工作階段中執行下列步驟︰連線至包含網域 fabrikam.com 的 Azure Active Directory

    Connect-MsolService
將 fabrikam.com 受控網域轉換為同盟︰

    Convert-MsolDomainToFederated -DomainName fabrikam.com -Verbose -SupportMultipleDomain
 
上述作業會建立網域 fabrikam.com 與相同 AD FS 的同盟。 您可以使用兩個網域的 Get-msoldomainfederationsettings 確認網域設定。

## <a name="next-steps"></a>後續步驟
[使用 Azure Active Directory 與 Active Directory 連線](whatis-hybrid-identity.md)
