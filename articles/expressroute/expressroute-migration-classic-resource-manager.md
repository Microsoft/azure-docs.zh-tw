---
title: 將 ExpressRoute 關聯的虛擬網路從傳統移轉至 Resource Manager：Azure：PowerShell | Microsoft Docs
description: 此頁面描述如何在移動線路之後將關聯的虛擬網路移轉至 Resource Manager。
documentationcenter: na
services: expressroute
author: ganesr
manager: timlt
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/06/2017
ms.author: ganesr;cherylmc
ms.openlocfilehash: 336f68308f7d4b4dd3c7476a4fabd793939e9e85
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23013203"
---
# <a name="migrate-expressroute-associated-virtual-networks-from-classic-to-resource-manager"></a>將 ExpressRoute 關聯的虛擬網路從傳統移至 Resource Manager

此文章說明如何在移動您的 ExpressRoute 線路之後將 Azure ExpressRoute 關聯的虛擬網路從傳統部署模型移轉至 Azure Resource Manager 部署模型。 


## <a name="before-you-begin"></a>開始之前
* 請確認您有最新版的 Azure PowerShell 模組。 如需詳細資訊，請參閱 [如何安裝及設定 Azure PowerShell](/powershell/azure/overview)。
* 開始設定之前，請確定您已經檢閱過[必要條件](expressroute-prerequisites.md)、[路由需求](expressroute-routing.md)和[工作流程](expressroute-workflows.md)。
* 請檢閱[將 ExpressRoute 電路從傳統移至 Resource Manager](expressroute-move.md) 下提供的資訊。 請確定您已完整了解各項限制。
* 請確認電路在傳統部署模型中的運作完全正常。
* 請確定您擁有建立在 Resource Manager 部署模型中建立的資源群組。
* 檢閱下列資源移轉文件︰

    * [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager.md)
    * [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager-deep-dive.md)
    * [常見問題集：平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager.md)
    * [檢閱最常見的移轉錯誤和避免方式](../virtual-machines/windows/migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

## <a name="supported-and-unsupported-scenarios"></a>支援和不支援的案例

* ExpressRoute 線路可以從傳統移轉至 Resource Manager 環境，而不需要停機。 您可以將任何 ExpressRoute 線路從傳統移轉至 Resource Manager 環境，而不需要停機。 請依照[使用 PowerShell 將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型](expressroute-howto-move-arm.md)中的指示執行。 這是將所連接資源移至虛擬網路的必要條件。
* 虛擬網路、閘道，以及虛擬網路中連結至相同訂用帳戶中 ExpressRoute 線路的相關聯部署，都可以移轉至 Resource Manager 環境，而不需要停機。 您可以依照稍後描述的步驟來移轉資源，例如虛擬網路、閘道，以及虛擬網路中部署的虛擬機器。 您必須確保虛擬網路在移轉之前都已正確設定。 
* 虛擬網路、閘道，以及虛擬網路內與 ExpressRoute 線路位於不同訂用帳戶的相關聯部署，都需要一些停機時間，才能完成移轉。 本文件的最後一節描述移轉資源所需遵循的步驟。
* 無法移轉同時具有「ExpressRoute 閘道」和「VPN 閘道」的虛擬網路。

## <a name="move-an-expressroute-circuit-from-classic-to-resource-manager"></a>將 ExpressRoute 線路從傳統移到 Resource Manager
嘗試移轉已連結至 ExpressRoute 線路的資源之前，您必須先將 ExpressRoute 線路從傳統移至 Resource Manager 環境。 若要完成此工作，請參閱下列文章：

* 請檢閱[將 ExpressRoute 電路從傳統移至 Resource Manager](expressroute-move.md) 下提供的資訊。
* [使用 Azure PowerShell 將戲路從傳統移至 Resource Manager](expressroute-howto-move-arm.md)。
* 使用 Azure 服務管理入口網站。 您可以依照工作流程來[建立新的 ExpressRoute 線路](expressroute-howto-circuit-portal-resource-manager.md)並選取匯入選項。 

這項作業不需要停機。 進行移轉時，您可以繼續在內部部署與 Microsoft 之間傳輸資料。

## <a name="migrate-virtual-networks-gateways-and-associated-deployments"></a>移轉虛擬網路、閘道與關聯的部署

移轉步驟取決於您的資源位於相同訂用帳戶、位於不同訂用帳戶，或混合兩者的情況。

### <a name="migrate-virtual-networks-gateways-and-associated-deployments-in-the-same-subscription-as-the-expressroute-circuit"></a>移轉虛擬網路、閘道，以及與 ExpressRoute 線路位於相同訂用帳戶的相關聯部署。
這一節說明移轉虛擬網路、閘道，以及與 ExpressRoute 線路位於相同訂用帳戶的相關聯部署所需遵循的步驟。 這項移轉不需要停機。 您可以透過移轉程序繼續使用所有資源。 管理平面會在移轉進行時遭到封鎖。 

1. 確保 ExpressRoute 線路已從傳統移至 Resource Manager 環境。
2. 確保已為了移轉適當地準備虛擬網路。
3. 註冊您的訂用帳戶以便移轉資源。 若要註冊您的訂用帳戶以便移轉資源，請使用下列 PowerShell 程式碼片段︰

  ```powershell 
  Select-AzureRmSubscription -SubscriptionName <Your Subscription Name>
  Register-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
  Get-AzureRmResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
  ```
4. 驗證、準備和移轉。 若要移轉虛擬網路，請使用下列 PowerShell 程式碼片段︰

  ```powershell
  Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
  Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
  Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
  ```

  您也可以執行下列 PowerShell Cmdlet 來中止移轉：

  ```powershell
  Move-AzureVirtualNetwork -Abort $vnetName
  ```

## <a name="next-steps"></a>後續步驟
* [平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager.md)
* [平台支援的從傳統移轉至 Azure Resource Manager 的技術深入探討](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager-deep-dive.md)
* [常見問題集：平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)](../virtual-machines/virtual-machines-windows-migration-classic-resource-manager.md)
* [檢閱最常見的移轉錯誤和避免方式](../virtual-machines/windows/migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
