---
title: 確認 VPN 閘道連線 | Microsoft Docs
description: 本文說明如何確認虛擬網路「VPN 閘道」連線。
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: 7e3d1043-caa9-4472-96d3-832f4e2c91ee
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/16/2017
ms.author: cherylmc
ms.openlocfilehash: 9dfea33b558b4b568da1fd9ed90402e6c5ce54bf
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45573190"
---
# <a name="verify-a-vpn-gateway-connection"></a>確認 VPN 閘道連線

本文說明如何驗證傳統和 Resource Manager 部署模型的 VPN 閘道連線。

## <a name="azure-portal"></a>Azure 入口網站

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="powershell"></a>PowerShell

若要使用 PowerShell 驗證 Resource Manager 部署模型的 VPN 閘道連線，請安裝最新版的 [Azure Resource Manager PowerShell Cmdlet](/powershell/azure/overview)。

[!INCLUDE [PowerShell](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

## <a name="azure-cli"></a>Azure CLI

若要使用 Azure CLI 驗證 Resource Manager 部署模型的 VPN 閘道連線，請安裝最新版的 [CLI 命令](https://docs.microsoft.com/cli/azure/install-azure-cli) (2.0 或更新版本)。

[!INCLUDE [CLI](../../includes/vpn-gateway-verify-connection-cli-rm-include.md)]


## <a name="azure-portal-classic"></a>Azure 入口網站 (傳統)

[!INCLUDE [Azure portal](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

## <a name="powershell-classic"></a>PowerShell (傳統)

若要使用 PowerShell 驗證傳統部署模型的 VPN 閘道連線，請安裝最新版的 Azure Resource Manager PowerShell Cmdlet。 請務必下載並安裝 [Service Management](https://docs.microsoft.com/powershell/azure/servicemanagement/install-azure-ps?view=azuresmps-4.0.0#azure-service-management-cmdlets) 模組。 使用 'Add-AzureAccount' 登入傳統部署模型。

[!INCLUDE [Classic PowerShell](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]

## <a name="next-steps"></a>後續步驟

* 您可以將虛擬機器加入您的虛擬網路。 請參閱 [建立網站的虛擬機器](../virtual-machines/virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) 以取得相關步驟。