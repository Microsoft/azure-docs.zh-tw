---
title: Azure Stack 的身分識別架構 | Microsoft Docs
description: 深入了解您可以搭配 Azure Stack 使用的身分識別架構。
services: azure-stack
documentationcenter: ''
author: PatAltimore
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/01/2018
ms.author: patricka
ms.reviewer: ''
ms.openlocfilehash: f2d843b343fa830adf41352ae692ea98bed52ecc
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44713641"
---
# <a name="identity-architecture-for-azure-stack"></a>Azure Stack 的身分識別架構
在您選擇要搭配 Azure Stack 使用的身分識別提供者之前，請先了解 Azure Active Directory (Azure AD) 與 Active Directory Federation Services (AD FS) 之間選項的重要差異。 

## <a name="capabilities-and-limitations"></a>功能和限制 
您選擇的身分識別提供者可能會限制您的選項，包括支援多租用戶。 

  

|功能或案例        |Azure AD  |AD FS  |
|------------------------------|----------|-------|
|已連線至網際網路     |是       |選用|
|支援多租用戶     |是       |否      |
|Marketplace 中的供應項目 |是       |是。 要求使用[離線 Marketplace 摘要整合](azure-stack-download-azure-marketplace-item.md#disconnected-or-a-partially-connected-scenario)工具。|
|支援 Active Directory 驗證程式庫 (ADAL) |是 |是|
|支援 Azure CLI、Visual Studio 和 PowerShell 等工具  |是 |是|
|透過 Azure 入口網站建立服務主體     |是 |否|
|建立包含憑證的服務主體      |是 |是|
|建立包含祕密 (金鑰) 的服務主體    |是 |否|
|應用程式可以使用 Graph 服務           |是 |否|
|應用程式可以使用身分識別提供者進行登入 |是 |是。 要求應用程式與內部部署 AD FS 執行個體同盟。 |

## <a name="topologies"></a>拓撲
下列各節討論您可使用的各種身分識別拓撲。

### <a name="azure-ad-single-tenant-topology"></a>Azure AD：單一租用戶拓撲 
根據預設，當您安裝 Azure Stack 並使用 Azure AD 時，Azure Stack 會使用單一租用戶拓撲。 

在下列情況下，單一租用戶拓撲很有用：
- 所有使用者都屬於相同的租用戶。
- 服務提供者主控組織的 Azure Stack 執行個體。 

![Azure Stack 單一租用戶拓撲搭配 Azure AD](media/azure-stack-identity-architecture/single-tenant.png)

此拓撲具有下列特性︰
- Azure Stack 會將所有的應用程式和服務註冊到相同的 Azure AD 租用戶目錄。 
- Azure Stack 只會驗證該目錄中的使用者和應用程式，包括權杖。 
- 系統管理員 (雲端操作員) 和租用戶使用者的身分識別位於相同的目錄租用戶中。 
- 若要讓其他目錄的使用者能夠存取此 Azure Stack 環境，您必須[邀請使用者](azure-stack-identity-overview.md#guest-users)成為租用戶目錄的來賓。 

### <a name="azure-ad-multi-tenant-topology"></a>Azure AD：多租用戶拓撲
雲端操作員可將 Azure Stack 設定為允許一或多個組織的租用戶存取應用程式。 使用者可透過使用者入口網站存取應用程式。 在此組態中，管理員入口網站 (由雲端操作員使用) 受限於單一目錄中的使用者。 

在下列情況下，多租用戶拓撲很有用：
- 服務提供者想要允許多個組織中的使用者存取 Azure Stack。

![Azure Stack 多租用戶拓撲拓樸搭配 Azure AD](media/azure-stack-identity-architecture/multi-tenant.png)

此拓撲具有下列特性︰
- 資源的存取權應該以每個組織為基礎。 
- 某個組織中的使用者不能將資源的存取權授與給其組織外部的使用者。 
- 系統管理員 (雲端操作員) 的身分識別可以位於與使用者身分識別不同的目錄租用戶中。 此分隔可提供識別提供者層級的帳戶隔離。 
 
### <a name="ad-fs"></a>AD FS  
下列其中一個條件成立時需要 AD FS 拓撲：
- Azure Stack 未連線到網際網路。
- Azure Stack 可以連線到網際網路，但您選擇使用 AD FS 作為您的識別提供者。
  
![使用 AD FS 的 Azure Stack 拓樸](media/azure-stack-identity-architecture/adfs.png)

此拓撲具有下列特性︰
- 若要支援在生產環境中使用此拓撲，您必須透過同盟信任來整合內建 Azure Stack AD FS 執行個體與 Active Directory 所支援的現有 AD FS 執行個體。 
- 您可以整合 Azure Stack 中的 Graph 服務與現有的 Active Directory 執行個體。 您也可以使用以 OData 為基礎的圖形 API 服務，該服務支援與 Azure AD Graph API 一致的 API。 

  若要與您的 Active Directory 執行個體互動，圖形 API 需要您的 Active Directory 中具有唯讀權限的使用者認證。 
  - 內建 AD FS 是以 Server 2016 為基礎。 
  - AD FS 與 Active Directory 執行個體必須以 Windows Server 2012 或更新版本為基礎。 
  
  Active Directory 執行個體與內建 AD FS 執行個體之間的互動不限於 OpenID Connect，並可使用任何相互支援的通訊協定。 
  - 使用者帳戶是在內部部署 Active Directory 執行個體中進行建立和管理。
  - 應用程式的服務主體和註冊是在內建 Active Directory 執行個體中進行管理。



## <a name="next-steps"></a>後續步驟
- [身分識別概觀](azure-stack-identity-overview.md)   
- [資料中心整合 - 身分識別](azure-stack-integrate-identity.md)
