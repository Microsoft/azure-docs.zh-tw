---
title: Azure 詞彙 - Azure 字典 | Microsoft Docs
description: 使用 Azure 詞彙，來了解 Azure 平台上的雲端術語。 這個簡短的 Azure 字典提供 Azure 的常見雲端術語定義。
keywords: Azure 字典、雲端術語、Azure 詞彙、詞彙定義、雲端詞彙
services: na
documentationcenter: na
author: MonicaRush
manager: jhubbard
editor: ''
ms.assetid: d7ac12f7-24b5-4bcd-9e4d-3d76fbd8d297
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2017
ms.author: monicar
ms.openlocfilehash: e62910cc760a200abba6d56e4fa1eb87973285ec
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46963589"
---
# <a name="microsoft-azure-glossary-a-dictionary-of-cloud-terminology-on-the-azure-platform"></a>Microsoft Azure 詞彙︰Azure 平台上的雲端術語字典

Microsoft Azure 詞彙是 Azure 平台上簡短的雲端術語字典。 另請參閱：

* [Microsoft Azure 和 Amazon Web Services](https://azure.microsoft.com/campaigns/azure-vs-aws/mapping/) - Azure 服務及其 AWS 對應項目的定義。<!-- I propose to link to https://azure.microsoft.com/services/ instead of this -->
* [雲端運算詞彙](https://azure.microsoft.com/overview/cloud-computing-dictionary/) - 一般產業雲端詞彙。

## <a name="account"></a>帳戶
用來存取和管理 Azure 訂用帳戶的帳戶。 通常稱為 Azure 帳戶，雖然帳戶可以是以下任何一項：現有公司、學校或個人 Microsoft 帳戶，或 Office 365 使用者名稱和密碼。 當您註冊[免費試用版](https://azure.microsoft.com)時，也可以建立帳戶來管理 Azure 訂用帳戶。  
請參閱[ Office 365 帳戶註冊 Azure 訂用帳戶](billing/billing-use-existing-office-365-account-azure-subscription.md)和[您可以用來登入的帳戶](active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)。

## <a name="api-app"></a>API 應用程式
[App Service 應用程式](#app-service-app)的另一個名稱。

## <a name="app-service-app"></a>App Service 應用程式
[Azure App Service](app-service/app-service-web-overview.md) 提供的計算資源，可供裝載網站或 Web 應用程式、Web API或[行動裝置應用程式後端](app-service-mobile/app-service-mobile-value-prop.md)。 App Service 應用程式也稱為「應用程式服務」、「Web 應用程式」、「API 應用程式」和「行動裝置應用程式」。

## <a name="availability-set"></a>可用性設定組
可一起管理的虛擬機器集合，以提供應用程式備援能力和可靠性。 可用性設定組的用法可確保在預定進行或未預定進行的維護事件期間，至少有一部虛擬機器可以使用。  
請參閱[管理 Windows 虛擬機器的可用性](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)和[管理 Linux 虛擬機器的可用性](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="classic-model"></a>Azure 傳統部署模型
您可以使用兩個 [部署模型](resource-manager-deployment-model.md) 之一來部署 Azure 中的資源 (新模型是 Azure Resource Manager)。 有些 Azure 服務僅支援 Resource Manager 部署模型、有些僅支援傳統部署模型，而有些則可支援兩個模型。 每個 Azure 服務的文件指定其支援的模型。

## <a name="cli"></a>Azure 命令列介面 (CLI)
命令列介面，可用來從 Windows、macOS 和 Linux 電腦管理 Azure 服務。  某些服務或服務功能可以只透過 PowerShell 或 CLI 進行管理。 請參閱 [Azure CLI](/cli/azure)

## <a name="powershell"></a>Azure PowerShell
命令列介面，可透過命令列從 Windows 電腦管理 Azure 服務。 某些服務或服務功能可以只透過 PowerShell 或 CLI 進行管理。
請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)

## <a name="arm-model"></a>Azure Resource Manager 部署模型
您可以使用兩個 [部署模型](resource-manager-deployment-model.md) 之一來部署 Microsoft Azure 中的資源 (另一個是傳統部署模型)。 有些 Azure 服務僅支援 Resource Manager 部署模型、有些僅支援傳統部署模型，而有些則可支援兩個模型。 每個 Azure 服務的文件指定其支援的模型。

## <a name="fault-domain"></a>容錯網域
可用性設定組中可能會同時失敗的虛擬機器集合。 範例之一是位於一個機架中的電腦群組，這組電腦會共用通用電源和網路開關。 在 Azure 中，可用性設定組中的虛擬機器會自動分散於多個容錯網域中。  
請參閱[管理 Windows 虛擬機器的可用性](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)或[管理 Linux 虛擬機器的可用性](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)  

## <a name="geo"></a>地區
針對資料常駐定義且通常包含兩個以上區域的界限。 界限可能是在國界內或超出國界，並受到稅法所影響。 每個地理區域至少擁有一個區域。 地理區域的範例為亞太地區和日本。 也稱為 *地理位置*。  
請參閱 [Azure 區域](best-practices-availability-paired-regions.md)

## <a name="geo-replication"></a>異地複寫
自動複寫內容的程序，例如區域配對中的 Blob、資料表和佇列。  
請參閱 [Azure SQL Database 的主動式異地複寫](sql-database/sql-database-geo-replication-overview.md)
<!-- The meaning of "geo" in this term seems to be different than the meaning provided in the "geo" entry -->

## <a name="image"></a>映像
包含作業系統和應用程式組態的檔案，可用來建立任意數目的虛擬機器。 在 Azure 中有兩種類型的映像：VM 映像和作業系統映像。 VM 映像包含作業系統和建立映像時所有連接至虛擬機器的磁碟。 作業系統映像只包含通用的作業系統且不含任何資料磁碟組態。  
請參閱[使用 PowerShell 或 CLI 在 Azure 中瀏覽並選取 Windows 虛擬機器映像](virtual-machines/windows/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="limits"></a>限制
您可以建立的資源數目或可達到的效能評定。 限制通常會與訂用帳戶、服務和供應項目相關聯。  
請參閱 [Azure 訂用帳戶和服務限制、配額與限制](azure-subscription-service-limits.md)

## <a name="load-balancer"></a>負載平衡器
網路上電腦之間分散連入流量的資源。 在 Azure 中，負載平衡器會將流量分散到定義於內部負載平衡器集中的虛擬機器。 [負載平衡器](load-balancer/load-balancer-overview.md) 可以是面向網際網路的，也可以是內部的。  

## <a name="mobile-app"></a>行動裝置應用程式
[App Service 應用程式](#app-service-app)的另一個名稱。

## <a name="offer"></a>供應項目
適用於 Azure 訂用帳戶的定價、信用額度及相關條款。  
請參閱 [Azure 供應項目詳細資料頁面](https://azure.microsoft.com/support/legal/offer-details/)

## <a name="portal"></a>入口網站
用來部署和管理 Azure 服務的安全 Web 入口網站。

## <a name="region"></a>region
地理區域內不會跨越國界且包含一或多個資料中心的地區。 定價、區域性服務和供應項目類型是在區域層級公開。 區域通常會與另一個區域配對，兩者可多達數百英哩遠。 區域配對可用來做為災害復原及高可用性案例的機制。 也稱為「位置」。  
請參閱 [Azure 區域](best-practices-availability-paired-regions.md)

## <a name="resource"></a>resource
Azure 方案一部分的項目。 每個 Azure 服務可讓您部署不同類型的資源，例如資料庫或虛擬機器。   
請參閱 [Azure Resource Manager 概觀](azure-resource-manager/resource-group-overview.md)

## <a name="resource-group"></a>資源群組
資源管理員中的容器，可保留應用程式的相關資源。 資源群組可以包含應用程式的所有資源，或只保留以邏輯方式分組在一起的資源。 您可以決定如何根據對組織最有利的方式，將資源配置到資源群組。  
請參閱 [Azure Resource Manager 概觀](azure-resource-manager/resource-group-overview.md)

## <a name="arm-template"></a>Resource Manager 範本
一個 JSON 檔案，可以宣告方式定義一或多個 Azure 資源，並定義所部署資源之間的相依性。 範本可用來以一致性方式重複部署資源。  
請參閱[編寫 Azure Resource Manager 範本](resource-group-authoring-templates.md)

## <a name="resource-provider"></a>資源提供者
提供資源的服務，讓您可透過資源管理員進行部署及管理。 每個資源提供者都會提供作業，以便能運用所部署的資源。 資源提供者可以透過 Azure 入口網站、Azure PowerShell 和數個程式設計的 SDK 來存取。  
請參閱 [Azure Resource Manager 概觀](azure-resource-manager/resource-group-overview.md)

## <a name="role"></a>角色
用於控制可指派給使用者、群組和服務的存取權的方式。 角色能夠在 Azure 資源上執行建立、管理及讀取之類的動作。  
請參閱 [RBAC：內建角色](role-based-access-control/built-in-roles.md)

## <a name="sla"></a>服務等級協定 (SLA)
此協定會描述 Microsoft 對執行時間與連線能力的承諾。 每個 Azure 服務都有特定的 SLA。  
請參閱[服務等級協定](https://azure.microsoft.com/support/legal/sla/)

## <a name="sas"></a>共用存取簽章 (SAS)
不需公開帳戶金鑰，即可讓您授與有限資源存取權的簽章。 例如，[Azure 儲存體使用 SAS](storage/common/storage-dotnet-shared-access-signature-part-1.md) 授與 blob 等物件的用戶端存取權。 [IoT 中樞會使用 SAS](iot-hub/iot-hub-devguide-security.md#security-tokens) 授與裝置權限以傳送遙測。

## <a name="storage-account"></a>storage account
可讓您存取 Azure 儲存體中 Azure Blob、佇列、資料表和檔案服務的帳戶。 儲存體帳戶名稱可定義 Azure 儲存體資料物件的唯一命名空間。  
請參閱[關於 Azure 儲存體帳戶](storage/common/storage-create-storage-account.md)

## <a name="subscription"></a>訂用帳戶
客戶與 Microsoft 的合約，可讓他們取得 Azure 服務。 訂用帳戶定價及相關條款是由針對訂用帳戶所選擇的供應項目來控管。
請參閱 [Microsoft 線上訂用帳戶合約](https://azure.microsoft.com/support/legal/subscription-agreement/)和 [Azure 訂用帳戶如何與 Azure Active Directory 產生關聯](active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

## <a name="tag"></a>tag
一個編製索引的詞彙，可讓您根據管理或計費需求將資源分類。 當您有複雜的資源集合時，您可使用標籤以最有利的方式將這些資產視覺化。 例如，您可以標記在組織中具有類似角色，或屬於相同部門的資源。  
請參閱[使用標籤來組織您的 Azure 資源](resource-group-using-tags.md)

## <a name="update-domain"></a>更新網域
可用性設定組中會同時更新的虛擬機器集合。 同一個更新網域中的虛擬機器會在預定進行的維護期間一起重新啟動。 Azure 永遠不會一次重新啟動多個更新網域。 也稱為升級網域。  
請參閱[管理 Windows 虛擬機器的可用性](virtual-machines/windows/manage-availability.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)和[管理 Linux 虛擬機器的可用性](virtual-machines/linux/manage-availability.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="vm"></a>虛擬機器
執行作業系統的實體電腦軟體實作。 同一個硬碟上可同時執行多部虛擬機器。 在 Azure 中，虛擬機器適用於各種大小。  
請參閱[虛擬機器文件](https://azure.microsoft.com/documentation/services/virtual-machines/)

## <a name="vm-extension"></a>虛擬機器擴充功能
一個實作行為或功能的資源，可協助其他程式運作，或提供與執行中電腦互動的能力。 例如，您可以使用 VM 存取擴充功能，來重設或修改 Azure 虛擬機器上的遠端存取值。
<!-- This definition seems obscure to me; maybe a list of examples would work better than a conceptual definition? -->
請參閱[關於虛擬機器擴充功能和功能 (Windows)](virtual-machines/windows/extensions-features.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 或[關於虛擬機器擴充功能和功能 (Linux)](virtual-machines/linux/extensions-features.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

## <a name="vnet"></a>虛擬網路
一種網路，可提供與所有其他 Azure 租用戶隔離之 Azure 資源間的連線能力。 [Azure VPN 閘道](vpn-gateway/vpn-gateway-about-vpngateways.md)可讓您建立虛擬網路之間及[虛擬網路與內部部署網路之間](vpn-gateway/vpn-gateway-plan-design.md)的連線。 您可以完全控制虛擬網路內的 IP 位址區塊、DNS 設定、安全性原則和路由表。  
請參閱[虛擬網路概觀](virtual-network/virtual-networks-overview.md)  

## <a name="web-app"></a>Web 應用程式
[App Service 應用程式](#app-service-app)的另一個名稱。

## <a name="see-also"></a>另請參閱

* [開始使用 Azure](https://azure.microsoft.com/get-started/)
* [雲端資源中心](https://azure.microsoft.com/resources/)  
* [讓 Azure 助您擴展商務應用程式](https://azure.microsoft.com/overview/business-apps-on-azure/)
* [Azure 之於您的資料中心](https://azure.microsoft.com/overview/business-apps-on-azure/)

