---
title: 控制 Azure 虛擬網路中的路由 - PowerShell - 傳統 | Microsoft Docs
description: 了解如何使用 PowerShell 來控制 VNet 中的路由 | Classic
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: d8d07c16-cbe5-4536-acd6-870269346fe3
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: genli
ms.openlocfilehash: 930676a396ae316ec761ba5d03ad1a1d0fd7a425
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38232561"
---
# <a name="control-routing-and-use-virtual-appliances-classic-using-powershell"></a>透過 PowerShell 控制路由和使用虛擬應用裝置 (傳統) 

> [!div class="op_single_selector"]
> * [PowerShell](tutorial-create-route-table-powershell.md)
> * [Azure CLI](tutorial-create-route-table-cli.md)
> * [PowerShell (傳統)](virtual-network-create-udr-classic-ps.md)
> * [CLI (傳統)](virtual-network-create-udr-classic-cli.md)

[!INCLUDE [virtual-network-create-udr-intro-include.md](../../includes/virtual-network-create-udr-intro-include.md)]

> [!IMPORTANT]
> 使用 Azure 資源之前，請務必了解 Azure 目前有 Azure Resource Manager 和「傳統」兩種部署模型。 在使用任何 Azure 資源之前，請先確認您了解 [部署模型和工具](../azure-resource-manager/resource-manager-deployment-model.md) 。 您可以選取本文章頂端的選項，檢視不同工具的文件。 本文涵蓋之內容包括傳統部署模型。
> 

[!INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

以下的 Azure PowerShell 命令範例假設您已根據上述案例建立簡單的環境。 如果您想要執行如本文件中所示的命令，請建立 [使用 PowerShell 建立 VNet (傳統)](virtual-networks-create-vnet-classic-netcfg-ps.md)中所示的環境。

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-the-udr-for-the-front-end-subnet"></a>建立前端子網路的 UDR
若要根據上述案例建立前端子網路所需的路由表和路由，請依照下列步驟執行。

1. 執行下列命令，建立前端子網路的路由表：

    ```powershell
    New-AzureRouteTable -Name UDR-FrontEnd -Location uswest `
    -Label "Route table for front end subnet"
    ```

2. 執行下列命令，在上方建立的路由表中建立路由，將目的地為後端子網路 (192.168.2.0/24) 的所有流量傳送到 **FW1** VM (192.168.0.4)：

    ```powershell
    Get-AzureRouteTable UDR-FrontEnd `
    |Set-AzureRoute -RouteName RouteToBackEnd -AddressPrefix 192.168.2.0/24 `
    -NextHopType VirtualAppliance `
    -NextHopIpAddress 192.168.0.4
    ```

3. 執行下列命令，建立與 **FrontEnd** 子網路關聯的路由表：

    ```powershell
    Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
    -SubnetName FrontEnd `
    -RouteTableName UDR-FrontEnd
    ```

## <a name="create-the-udr-for-the-back-end-subnet"></a>建立後端子網路的 UDR
若要根據案例建立後端子網路所需的路由表和路徑，完成下列步驟：

1. 執行下列命令，建立後端子網路的路由表：

    ```powershell
    New-AzureRouteTable -Name UDR-BackEnd `
    -Location uswest `
    -Label "Route table for back end subnet"
    ```

2. 執行下列命令，在上方建立的路由表中建立路由，將目的地為前端子網路 (192.168.1.0/24) 的所有流量傳送到 **FW1** VM (192.168.0.4)：

    ```powershell
    Get-AzureRouteTable UDR-BackEnd
    | Set-AzureRoute `
    -RouteName RouteToFrontEnd `
    -AddressPrefix 192.168.1.0/24 `
    -NextHopType VirtualAppliance `
    -NextHopIpAddress 192.168.0.4
    ```

3. 執行下列命令，建立與 **BackEnd** 子網路關聯的路由表：

    ```powershell
    Set-AzureSubnetRouteTable -VirtualNetworkName TestVNet `
    -SubnetName BackEnd `
    -RouteTableName UDR-BackEnd
    ```

## <a name="enable-ip-forwarding-on-the-fw1-vm"></a>啟用 FW1 VM 上的 IP 轉送

若要啟用 FW1 VM 中的 IP 轉送，完成下列步驟：

1. 執行下列命令檢查 IP 轉送的狀態：

    ```powershell
    Get-AzureVM -Name FW1 -ServiceName TestRGFW `
    | Get-AzureIPForwarding
    ```

2. 執行下列命令，以啟用適用於 *FW1* VM 的 IP 轉送：

    ```powershell
    Get-AzureVM -Name FW1 -ServiceName TestRGFW `
    | Set-AzureIPForwarding -Enable
    ```
