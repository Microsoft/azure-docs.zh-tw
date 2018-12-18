---
title: 使用 PowerShell 建立網路安全性群組 (傳統) | Microsoft Docs
description: 了解如何使用 PowerShell 建立和部署網路安全性群組 (傳統)
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: 86810b0d-0240-46a2-8548-fca22daa56f3
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/02/2016
ms.author: genli
ms.openlocfilehash: ecb977660ed99a3cea2a71a867f50822b23e568c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38634191"
---
# <a name="create-a-network-security-group-classic-using-powershell"></a>使用 PowerShell 建立網路安全性群組 (傳統)
[!INCLUDE [virtual-networks-create-nsg-selectors-classic-include](../../includes/virtual-networks-create-nsg-selectors-classic-include.md)]

[!INCLUDE [virtual-networks-create-nsg-intro-include](../../includes/virtual-networks-create-nsg-intro-include.md)]

[!INCLUDE [azure-arm-classic-important-include](../../includes/azure-arm-classic-important-include.md)]

本文涵蓋之內容包括傳統部署模型。 您也可以 [在資源管理員部署模型中建立 NSG](tutorial-filter-network-traffic.md)。

[!INCLUDE [virtual-networks-create-nsg-scenario-include](../../includes/virtual-networks-create-nsg-scenario-include.md)]

以下的範例 PowerShell 命令會預期已根據上述案例建立簡單的環境。 如果您想要執行如本文件中所顯示的命令，請先建置 [建立 VNet](virtual-networks-create-vnet-classic-netcfg-ps.md)以建置測試環境。

## <a name="create-an-nsg-for-the-front-end-subnet"></a>建立前端子網路的 NSG

1. 如果您未曾用過 Azure PowerShell，請參閱[如何安裝和設定 Azure PowerShell](/powershell/azure/overview)。

2. 建立名為 *NSG-FrontEnd* 的網路安全性群組：

    ```powershell   
    New-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" -Location uswest `
      -Label "Front end subnet NSG"
   ```

3. 建立允許從網際網路存取連接埠 3389 的安全性規則：

    ```powershell   
    Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
      | Set-AzureNetworkSecurityRule -Name rdp-rule `
      -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
      -SourceAddressPrefix Internet  -SourcePortRange '*' `
      -DestinationAddressPrefix '*' -DestinationPortRange '3389'
   ```

4. 建立允許從網際網路存取連接埠 80 的安全性規則：

    ```powershell   
    Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
      | Set-AzureNetworkSecurityRule -Name web-rule `
      -Action Allow -Protocol TCP -Type Inbound -Priority 200 `
      -SourceAddressPrefix Internet  -SourcePortRange '*' `
      -DestinationAddressPrefix '*' -DestinationPortRange '80'
    ```

## <a name="create-an-nsg-for-the-back-end-subnet"></a>建立後端子網路的 NSG

1. 建立名為 *NSG-BackEnd* 的網路安全性群組：
   
    ```powershell
    New-AzureNetworkSecurityGroup -Name "NSG-BackEnd" -Location uswest `
      -Label "Back end subnet NSG"
    ```

2. 建立允許從前端子網路存取連接埠 1433 (SQL Server 所使用的預設連接埠) 的安全性規則：
   
    ```powershell
    Get-AzureNetworkSecurityGroup -Name "NSG-FrontEnd" `
      | Set-AzureNetworkSecurityRule -Name rdp-rule `
      -Action Allow -Protocol TCP -Type Inbound -Priority 100 `
      -SourceAddressPrefix 192.168.1.0/24  -SourcePortRange '*' `
      -DestinationAddressPrefix '*' -DestinationPortRange '1433'
    ```

3. 建立封鎖從子網路存取網際網路的安全性規則：
   
    ```powershell
    Get-AzureNetworkSecurityGroup -Name "NSG-BackEnd" `
      | Set-AzureNetworkSecurityRule -Name block-internet `
      -Action Deny -Protocol '*' -Type Outbound -Priority 200 `
      -SourceAddressPrefix '*'  -SourcePortRange '*' `
      -DestinationAddressPrefix Internet -DestinationPortRange '*'
   ```