---
title: Azure Stack 管理基本知識 | Microsoft Docs
description: 了解要管理 Azure Stack 需要知道哪些事項。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 856738a7-1510-442a-88a8-d316c67c757c
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.openlocfilehash: 37b8eff2d4ed89c90f1fa6f128673ed5bacaaa90
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339945"
---
# <a name="azure-stack-administration-basics"></a>Azure Stack 管理基本知識
若您還不熟悉 Azure Stack 管理，有幾件事需要知道。 本指南提供您身為 Azure Stack 操作員的角色概觀，以及您需要告知使用者哪些事項，使他們快速上手。

## <a name="understand-the-builds"></a>了解組建

### <a name="integrated-systems"></a>整合系統

如果您使用 Azure Stack 整合系統，將會透過更新套件散發 Azure Stack 更新版本給您。 您可以使用系統管理員入口網站中的 [更新 ] 圖格來匯入及套用這些套件。
 
### <a name="development-kit"></a>開發套件

如果您使用的是「Azure Stack 開發套件」，請檢閱[什麼是 Azure Stack？](.\asdk\asdk-what-is.md)一文，以確保您了解開發套件的目的及其限制。 您應該將開發套件當作「沙箱」使用，在其中評估 Azure Stack，並在非生產環境中開發和測試您的應用程式。 (如需部署資訊，請參閱 [Azure Stack 開發套件部署](.\asdk\asdk-install.md)一文)。

就像 Azure 一樣，我們迅速地進行創新。 我們會定期發行新組建。 如果您執行的是開發套件並且想要移至最新的組建，就必須[重新部署 Azure Stack](.\asdk\asdk-redeploy.md)。 您無法套用更新套件。 此流程需要時間，但好處是您可以嘗試最新的功能。 我們網站上的開發套件文件會反映最新發行的組建。

## <a name="learn-about-available-services"></a>了解可用的服務

您需要意識到有哪些服務您可以提供給您的使用者。 Azure Stack 支援 Azure 服務的子集。 支援的服務清單將持續改進。

**基礎服務**

依預設，當您部署 Azure Stack 時，Azure Stack 會包含下列的「基礎服務」：

- 計算
- 儲存體
- 網路功能
- Key Vault

利用這些基礎服務，您可用最少的設定，將基礎結構即服務 (IaaS) 提供給您的使用者。

**其他服務**

目前，我們支援下列的其他平台即服務 (PaaS) 服務：

- App Service 方案
- Azure Functions
- SQL 和 MySQL 資料庫

在可讓使用您的使用者使用之前，這些服務需要其他設定。 如需詳細資訊，請參閱 Azure Stack 操作員文件的＜教學課程＞和＜做法指南\提供服務＞小節。

**服務藍圖**

Azure Stack 會持續新增對 Azure 服務的支援。 如需了解所規劃的藍圖，請參閱 [Azure Stack：Azure 的延伸](https://go.microsoft.com/fwlink/?LinkId=842846&clcid=0x409) \(英文\) 白皮書。 您也可以留意 [Azure Stack 部落格文章](https://azure.microsoft.com/blog/tag/azure-stack-technical-preview)的新宣告。

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
 
您可以使用[系統管理員入口網站](azure-stack-manage-portals.md)或 PowerShell 來管理 Azure Stack。 了解基本概念最簡單的方式是透過入口網站。 如果您想要使用 PowerShell，則有些準備步驟。 您必須[安裝](azure-stack-powershell-install.md) PowerShell、[下載](azure-stack-powershell-download.md)其他模組，並[設定](azure-stack-powershell-configure-admin.md) PowerShell。

Azure Stack 使用 Azure Resource Manager 作為其基礎的部署、管理及組織機制。 如果您要管理 Azure Stack 並協助支援使用者，您應該了解 Resource Manager。 請參閱[開始使用 Azure Resource Manager](http://download.microsoft.com/download/E/A/4/EA4017B5-F2ED-449A-897E-BD92E42479CE/Getting_Started_With_Azure_Resource_Manager_white_paper_EN_US.pdf) 白皮書。

## <a name="your-typical-responsibilities"></a>您的一般責任

您的使用者想要使用服務。 從其觀點看來，您的主要角色是將這些服務提供給他們。 您必須決定所要提供的服務，並藉由建立方案、供應項目及配來提供這些服務。 如需詳細資訊，請參閱 [Azure Stack 中的服務提供概觀](azure-stack-offer-services-overview.md)。 

您也需要將項目 (例如虛擬機器映像) 新增到[市集](azure-stack-marketplace.md)。 最簡單方式是[從 Azure 下載 Marketplace 項目到 Azure Stack 中](azure-stack-download-azure-marketplace-item.md)。

> [!NOTE]
> 如果您想要測試您的方案、供應項目和服務，您應該使用[使用者入口網站](azure-stack-manage-portals.md)，而非系統管理員入口網站。

除了提供服務之外，您還必須執行操作員的所有一般職務，以保持 Azure Stack 正常運作。 這些工作包括：

- 新增使用者帳戶 (如 [Azure Active Directory](azure-stack-add-new-user-aad.md) 部署或 [Active Directory 同盟服務](azure-stack-add-users-adfs.md)部署)
- [指派角色型存取控制 (RBAC) 角色](azure-stack-manage-permissions.md) (這不限於系統管理員)。
- [監視基礎結構健康情況](azure-stack-monitor-health.md)
- 管理[網路](azure-stack-viewing-public-ip-address-consumption.md)和[儲存體](azure-stack-manage-storage-accounts.md)資源
- 更換故障的硬體，例如[更換故障的磁碟](azure-stack-replace-disk.md)。

## <a name="what-to-tell-your-users"></a>要告訴使用者的內容

您將需要讓使用者知道如何使用 Azure Stack 中的服務、如何連線到環境，以及如何訂閱供應項目。 除了任何您想要提供給使用者的自訂文件之外，您還可以將使用者導向「Azure Stack 使用者文件」網站。

**了解如何使用 Azure Stack 中的服務**

在使用服務和 Azure Stack 中的應用程式之前，有些資訊您的使用者必須先了解。 例如，有特定的 PowerShell 和 API 版本需求。 此外，在 Azure 中的服務與 Azure Stack 中的對等服務之間有一些功能差異。 請確保您的使用者會檢閱下列文章：

- [關鍵考量：使用 Azure Stack 的服務或組建 Azure Stack 應用程式](user/azure-stack-considerations.md)
- [Azure Stack 中虛擬機器的考量](user/azure-stack-vm-considerations.md)
- [儲存體：差異和考量](user/azure-stack-acs-differences.md)

這些文件中的資訊摘要了說明在 Azure 中與 Azure Stack 中服務之間的差異。 它補充了全域 Azure 文件中所提供的 Azure 服務相關資訊。

**以使用者的身分連線到 Azure Stack**

在開發套件環境中，如果使用者沒有遠端桌面對開發套件主機的存取權，就必須設定虛擬私人網路 (VPN) 連線，才能存取 Azure Stack。 請參閱[連線到 Azure Stack](azure-stack-connect-azure-stack.md)。 

您的使用者會想要知道如何[存取使用者入口網站](user/azure-stack-use-portal.md)或如何透過 PowerShell 連線。 在整合系統環境中，使用者入口網站位址會因部署而異。 您將必須提供正確的 URL 給使用者。

如果使用 PowerShell，使用者可能需要先註冊資源提供者，才能使用服務。 (資源提供者負責管理服務。 例如，網路資源提供者負責管理如虛擬網路、網路介面和負載平衡器等資源。)他們必須[安裝](user/azure-stack-powershell-install.md) PowerShell、[下載](user/azure-stack-powershell-download.md) 其他模組，並[設定](user/azure-stack-powershell-configure-user.md) PowerShell (其中包含資源提供者註冊)。

**訂閱供應項目**

使用者必須先[訂閱供應項目](azure-stack-subscribe-plan-provision-vm.md) (您以操作員身分所建立)，才能存取服務。

## <a name="where-to-get-support"></a>從哪裡取得支援

### <a name="integrated-systems"></a>整合系統

就整合系統而言，在 Microsoft 與原始設備製造商 (OEM) 硬體合作夥伴之間有一個協調的呈報和解決程序。

如果雲端服務發生問題，將會透過「Microsoft 客戶支援服務」(CSS).提供支援。 如果您按一下系統管理員入口網站右上角的 [說明及支援] 圖示 (問號)，然後按一下 [新增支援要求]，就會開啟一個可供您直接建立支援要求的網站。

如果部署、修補程式和更新、硬體 (包括現場可更換單元) 及任何硬體品牌化軟體 (例如在硬體生命週期主機上執行的軟體) 發生問題，請先連絡您的 OEM 硬體廠商。

針對任何其他情況，則請連絡 Microsoft CSS。

### <a name="development-kit"></a>開發套件

針對開發套件，您可以在 [Microsoft 論壇](https://social.msdn.microsoft.com/Forums/azure/home?forum=azurestack)中詢問與支援相關的問題。 如果您按一下系統管理員入口網站右上角的 [說明及支援] 圖示 (問號)，然後按一下 [新增支援要求]，這會直接開啟論壇網站。 我們會定期留意這些論壇。 由於開發套件是一個評估環境，因此並未透過 Microsoft CSS 提供官方支援。

## <a name="next-steps"></a>後續步驟

[Azure Stack 中的區域管理](azure-stack-region-management.md)


