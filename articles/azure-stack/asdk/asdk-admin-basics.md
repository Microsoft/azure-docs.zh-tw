---
title: Azure Stack 開發套件基本概念 | Microsoft Docs
description: 說明如何執行「Azure Stack 開發套件」(ASDK) 的基本管理工作。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 555bf95367ab0d3e6c4afe09edb6777899400dd8
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49344663"
---
# <a name="asdk-administration-basics"></a>ASDK 管理基本概念 
若您還不熟悉 Azure Stack 開發套件(ASDK) 管理，有幾件事需要知道。 本指南提供您在評估環境中擔任 Azure Stack 操作員之角色的概觀，以及如何確定測試使用者可以快速上手。

首先，您應檢閱[什麼是 Azure Stack 開發套件？](asdk-what-is.md)文件，以確定您了解 ASDK 的目的及其限制。 您應該將開發套件當作「沙箱」使用，在其中評估 Azure Stack 以在非生產環境中開發和測試您的應用程式。 

與 Azure 類似，Azure Stack 會快速創新，因此我們會定期發行 ASDK 的新組建。 不過，您無法像升級 Azure Stack 整合系統部署一樣地升級 ASDK。 因此，若您想要移至最新的組建，就必須徹底[重新部署 ASDK](asdk-redeploy.md)。 您無法套用更新套件。 此程序需要時間，但好處是您可以在最新功能可供使用時立即試用。 

## <a name="what-account-should-i-use"></a>我該使用哪個帳戶?
管理 Azure Stack 時，有幾個您應該注意的帳戶考量。 尤其是在使用 Windows Server「Active Directory 同盟服務」(AD FS) 而不是 Azure Active Directory (Azure AD) 作為身分識別提供者的部署中。 下列帳戶考量同時適用於 Azure Stack 整合式系統和 ASDK 部署：

|帳戶|Azure AD|AD FS|
|-----|-----|-----|
|本機系統管理員 (.\Administrator)|ASDK 主機管理員|ASDK 主機管理員|
|AzureStack\AzureStackAdmin|ASDK 主機管理員<br><br>可用來登入 Azure Stack 管理入口網站<br><br>具備檢視及管理 Service Fabric 環的存取權|ASDK 主機管理員<br><br>無法存取 Azure Stack 管理入口網站<br><br>具備檢視及管理 Service Fabric 環的存取權<br><br>已不再是「預設提供者訂用帳戶」(DPS) 的擁有者|
|AzureStack\CloudAdmin|可在「具有特殊權限的端點」內存取及執行允許的命令|可在「具有特殊權限的端點」內存取及執行允許的命令<br><br>無法登入 ASDK 主機<br><br>「預設提供者訂用帳戶」(DPS) 的擁有者|
|Azure AD 全域管理員|安裝時所使用<br><br>「預設提供者訂用帳戶」(DPS) 的擁有者|不適用|
|

## <a name="what-tools-do-i-use-to-manage"></a>我可以使用哪些工具來管理？
您可以使用 [Azure Stack 系統管理員入口網站](https://adminportal.local.azurestack.external)或 PowerShell 來管理 Azure Stack。 了解基本概念最簡單的方式是透過入口網站。 如果您想要使用 PowerShell，您需要安裝[適用於 Azure Stack 的 PowerShell](asdk-post-deploy.md#install-azure-stack-powershell) 並[從 GitHub 下載 Azure Stack 工具](asdk-post-deploy.md#download-the-azure-stack-tools)。

Azure Stack 使用 Azure Resource Manager 作為其基礎的部署、管理及組織機制。 如果您要管理 Azure Stack 並協助支援使用者，您應該了解 Azure Resource Manager。 您可以透過閱讀[開始使用 Azure Resource Manager 白皮書](http://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf)來深入了解。

## <a name="your-typical-responsibilities"></a>您的一般責任
您的使用者想要使用服務。 從其觀點看來，您的主要角色是將這些服務提供給他們。 您可以使用 ASDK 來了解該提供什麼服務，以及如何藉由[建立方案、供應項目及配額](asdk-offer-services.md)來提供這些服務。 您還需要將項目 (例如虛擬機器映像) 新增到 Marketplace 中。 最簡單的方式就是從 Azure [下載市集項目](asdk-marketplace-item.md)到 Azure Stack。

> [!NOTE]
> 如果您想要測試方案、供應項目和服務，您應該使用[使用者入口網站](https://portal.local.azurestack.external)，而非[系統管理員入口網站](https://adminportal.local.azurestack.external)。

除了提供服務外，您還必須執行 Azure Stack 操作員的所有一般職務，以保持 ASDK 正常運作。 這些職務包括以下項目：
- 新增 Azure Active Directory (Azure AD) 或 Active Directory 同盟服務 (AD FS) 部署的使用者帳戶。
- 指派角色型存取控制 (RBAC) 角色 (這不僅限於系統管理員)
- 監視基礎結構健康情況
- 管理網路和儲存體資源
- 更換故障的開發套件主機電腦硬體 

## <a name="where-to-get-support"></a>從哪裡取得支援
若是開發套件，您唯一的支援選項是在 [Microsoft Azure Stack 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)詢問與支援相關的問題。 如果您按一下系統管理員入口網站右上角的 [說明及支援] 圖示 (問號)，然後按一下 [新增支援要求]，這會直接開啟論壇網站。 我們會定期留意這些論壇。 

> [!IMPORTANT]
> 因為 ASDK 屬於評估環境，因此 Microsoft 客戶支援服務 (CSS) 不會提供官方支援。

## <a name="next-steps"></a>後續步驟
[部署 ASDK](asdk-install.md)

