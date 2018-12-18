---
title: Azure PowerShell 指令碼範例 - 限制網路流量 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 建立具有 Web 應用程式防火牆和虛擬機器擴展集的應用程式閘道，它會使用 OWASP 規則來限制流量。
services: application-gateway
documentationcenter: networking
author: vhorne
manager: jpconnock
editor: tysonn
tags: azure-resource-manager
ms.service: application-gateway
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 01/29/2018
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: d6f8083e0070435e6ad8bb0dc0ff520b58c7eead
ms.sourcegitcommit: c47ef7899572bf6441627f76eb4c4ac15e487aec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/04/2018
ms.locfileid: "33202281"
---
# <a name="restrict-web-traffic-using-azure-powershell"></a>使用 Azure PowerShell 限制網路流量

這個指令碼會建立具有 Web 應用程式防火牆的應用程式閘道，它使用虛擬機器擴展集作為後端伺服器。 Web 應用程式防火牆會根據 OWASP 規則限制網路流量。 執行指令碼之後，您可以使用應用程式閘道的公用 IP 位址來進行測試。

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/application-gateway/create-vmss/create-vmss-waf.ps1 "Create application gateway with WAF")]

## <a name="clean-up-deployment"></a>清除部署 

執行下列命令來移除資源群組、應用程式閘道和所有相關資源。

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroupAG
```

## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令來建立部署。 下表中的每個項目都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) | 建立用來存放所有資源的資源群組。 |
| [New-AzureRmVirtualNetworkSubnetConfig](/powershell/module/azurerm.network/new-azurermvirtualnetworksubnetconfig) | 建立子網路組態。 |
| [New-AzureRmVirtualNetwork](/powershell/module/azurerm.network/new-azurermvirtualnetwork) | 使用子網路組態建立虛擬網路。 |
| [New-AzureRmPublicIpAddress](/powershell/module/azurerm.network/new-azurermpublicipaddress) | 建立應用程式閘道的公用 IP 位址。 |
| [New-AzureRmApplicationGatewayIPConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewayipconfiguration) | 建立組態，該組態讓子網路與應用程式閘道產生關聯。 |
| [New-AzureRmApplicationGatewayFrontendIPConfig](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendipconfig) | 建立組態，該組態將公用 IP 位址指派給應用程式閘道。 |
| [New-AzureRmApplicationGatewayFrontendPort](/powershell/module/azurerm.network/new-azurermapplicationgatewayfrontendport) | 指派要用來存取應用程式閘道的連接埠。 |
| [New-AzureRmApplicationGatewayBackendAddressPool](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendaddresspool) | 為應用程式閘道建立後端集區。 |
| [New-AzureRmApplicationGatewayBackendHttpSettings](/powershell/module/azurerm.network/new-azurermapplicationgatewaybackendhttpsettings) | 設定後端集區的設定。 |
| [New-AzureRmApplicationGatewayHttpListener](/powershell/module/azurerm.network/new-azurermapplicationgatewayhttplistener) | 建立接聽程式。 |
| [New-AzureRmApplicationGatewayRequestRoutingRule](/powershell/module/azurerm.network/new-azurermapplicationgatewayrequestroutingrule) | 建立路由規則。 |
| [New-AzureRmApplicationGatewaySku](/powershell/module/azurerm.network/new-azurermapplicationgatewaysku) | 指定應用程式閘道的層級和容量。 |
| [New-AzureRmApplicationGatewayWebApplicationFirewallConfiguration](/powershell/module/azurerm.network/new-azurermapplicationgatewaywebapplicationfirewallconfiguration) | 建立 Web 應用程式防火牆組態。 |
| [New-AzureRmApplicationGateway](/powershell/module/azurerm.network/new-azurermapplicationgateway) | 建立應用程式閘道。 |
| [Set-AzureRmVmssStorageProfile](/powershell/module/azurerm.compute/set-azurermvmssstorageprofile) | 建立擴展集的儲存體設定檔。 |
| [Set-AzureRmVmssOsProfile](/powershell/module/azurerm.compute/set-azurermvmssosprofile) | 定義擴展集的作業系統。 |
| [Add-AzureRmVmssNetworkInterfaceConfiguration](/powershell/module/azurerm.compute/add-azurermvmssnetworkinterfaceconfiguration) | 定義擴展集的網路介面。 |
| [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvm) | 建立虛擬機器擴展集。 |
| [New-AzureRmStorageAccount](/powershell/module/azurerm.storage/new-azurermstorageaccount) | 建立儲存體帳戶。 |
| [Set-AzureRmDiagnosticSetting](/powershell/module/azurerm.insights/set-azurermdiagnosticsetting) | 設定記錄資料的診斷。 |
| [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress) | 取得應用程式閘道的公用 IP 位址。 |
|[Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup) | 移除資源群組及其內含的所有資源。 | 
## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

您可以在 [Azure 應用程式閘道文件](../powershell-samples.md)中找到其他應用程式閘道的 PowerShell 指令碼範例。
