---
title: 控制 Azure 虛擬網路中的路由 - CLI - 傳統 | Microsoft Docs
description: 了解如何在傳統部署模型中使用 Azure CLI 來控制 VNet 中的路由
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: ca2b4638-8777-4d30-b972-eb790a7c804f
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: genli
ms.openlocfilehash: 0b6c8da03c4a67aadb38280ba958a9b0feb88d1f
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38678573"
---
# <a name="control-routing-and-use-virtual-appliances-classic-using-the-azure-cli"></a>使用 Azure CLI 控制路由和使用虛擬應用裝置 (傳統)

> [!div class="op_single_selector"]
> * [PowerShell](tutorial-create-route-table-powershell.md)
> * [Azure CLI](tutorial-create-route-table-cli.md)
> * [PowerShell (傳統)](virtual-network-create-udr-classic-ps.md)
> * [CLI (傳統)](virtual-network-create-udr-classic-cli.md)

[!INCLUDE [virtual-network-create-udr-intro-include.md](../../includes/virtual-network-create-udr-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括傳統部署模型。 您也可以 [在資源管理員部署模型中控制路由和使用虛擬應用裝置](tutorial-create-route-table-cli.md)。

[!INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

以下的範例 Azure CLI 命令是假設您已根據上述案例建立簡單的環境。 如果您想要執行如本文件中所示的命令，請建立 [使用 Azure CLI 建立 VNet (傳統)](virtual-networks-create-vnet-classic-cli.md)中所示的環境。

[!INCLUDE [azure-cli-prerequisites-include.md](../../includes/azure-cli-prerequisites-include.md)]

## <a name="create-the-udr-for-the-front-end-subnet"></a>建立前端子網路的 UDR
若要根據上述案例建立前端子網路所需的路由表和路由，請依照下列步驟執行。

1. 執行下列命令切換至傳統模式：

    ```azurecli
    azure config mode asm
    ```

    輸出：

        info:    New mode is asm

2. 執行下列命令，建立前端子網路的路由表：

    ```azurecli
    azure network route-table create -n UDR-FrontEnd -l uswest
    ```
   
    輸出：
   
        info:    Executing command network route-table create
        info:    Creating route table "UDR-FrontEnd"
        info:    Getting route table "UDR-FrontEnd"
        data:    Name                            : UDR-FrontEnd
        data:    Location                        : West US
        info:    network route-table create command OK
   
    參數：
   
   * **-l (或 --location)**。 將要建立新 NSG 的 Azure 區域。 在本文案例中為 *westus*。
   * **-n (or --name)**。 新 NSG 的名稱。 在本文案例中為 *NSG-FrontEnd*。
3. 執行下列命令，在上方建立的路由表中建立路由，將目的地為後端子網路 (192.168.2.0/24) 的所有流量傳送到 **FW1** VM (192.168.0.4)：

    ```azurecli
    azure network route-table route set -r UDR-FrontEnd -n RouteToBackEnd -a 192.168.2.0/24 -t VirtualAppliance -p 192.168.0.4
    ```

    輸出：
   
        info:    Executing command network route-table route set
        info:    Getting route table "UDR-FrontEnd"
        info:    Setting route "RouteToBackEnd" in a route table "UDR-FrontEnd"
        info:    network route-table route set command OK
   
    參數：
   
   * **-r (或 --route-table-name)**。 將會加入路由的路由表的名稱。 在本文案例中為 *UDR-FrontEnd*。
   * **-a (或 --address-prefix)**。 封包所指向位置的子網路的位址首碼。 在本文案例中為 *192.168.2.0/24*。
   * **-t (或 --next-hop-type)**。 將傳送流量的目標物件類型。 可能的值為 VirtualAppliance、VirtualNetworkGateway、VNETLocal、Internet 或 None。
   * **-p (或 --next-hop-ip-address)**。 下個躍點的 IP 位址。 在本文案例中為 *192.168.0.4*。
4. 執行下列命令，將建立的路由表關聯至 **FrontEnd** 子網路：

    ```azurecli
    azure network vnet subnet route-table add -t TestVNet -n FrontEnd -r UDR-FrontEnd
    ```
   
    輸出：
   
        info:    Executing command network vnet subnet route-table add
        info:    Looking up the subnet "FrontEnd"
        info:    Looking up network configuration
        info:    Looking up network gateway route tables in virtual network "TestVNet" subnet "FrontEnd"
        info:    Associating route table "UDR-FrontEnd" and subnet "FrontEnd"
        info:    Looking up network gateway route tables in virtual network "TestVNet" subnet "FrontEnd"
        data:    Route table name                : UDR-FrontEnd
        data:      Location                      : West US
        data:      Routes:
        info:    network vnet subnet route-table add command OK    
   
    參數：
   
   * **-t (或 --vnet-name)**。 子網路所在的 VNet 名稱。 在本文案例中為 *TestVNet*。
   * **-n (或 --subnet-name)**。 路由表將加入的子網路的名稱。 在本文案例中為 *FrontEnd*。

## <a name="create-the-udr-for-the-back-end-subnet"></a>建立後端子網路的 UDR
若要根據案例建立後端子網路所需的路由表和路徑，完成下列步驟：

1. 執行下列命令，建立後端子網路的路由表：

    ```azurecli
    azure network route-table create -n UDR-BackEnd -l uswest
    ```

2. 執行下列命令，在上方建立的路由表中建立路由，將目的地為前端子網路 (192.168.1.0/24) 的所有流量傳送到 **FW1** VM (192.168.0.4)：

    ```azurecli
    azure network route-table route set -r UDR-BackEnd -n RouteToFrontEnd -a 192.168.1.0/24 -t VirtualAppliance -p 192.168.0.4
    ```

3. 執行下列命令，建立與 **BackEnd** 子網路關聯的路由表：

    ```azurecli
    azure network vnet subnet route-table add -t TestVNet -n BackEnd -r UDR-BackEnd
    ```

