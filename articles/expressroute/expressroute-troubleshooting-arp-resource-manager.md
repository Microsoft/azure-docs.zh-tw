---
title: 取得 ARP 表格：Resource Manager：Azure ExpressRoute 疑難排解 | Microsoft Docs
description: 此頁面提供相關指示，協助您取得適用於 ExpressRoute 線路的 ARP 表格
documentationcenter: na
services: expressroute
author: ganesr
manager: carolz
editor: tysonn
ms.assetid: 0a6bf1d5-6baf-44dd-87d3-1ebd2fd08bdc
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/30/2017
ms.author: ganesr
ms.openlocfilehash: a65b1ba2998eae33b3e73bd2492fbbf025eb5946
ms.sourcegitcommit: b5c6197f997aa6858f420302d375896360dd7ceb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/21/2017
ms.locfileid: "23013133"
---
# <a name="getting-arp-tables-in-the-resource-manager-deployment-model"></a>在 Resource Manager 部署模型中取得 ARP 表格
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell - 傳統](expressroute-troubleshooting-arp-classic.md)
> 
> 

本文將逐步引導您了解適用於 ExpressRoute 線路的 ARP 表格。 

> [!IMPORTANT]
> 本文件旨在協助您診斷並修正簡單的問題。 它無法取代 Microsoft 支援服務。 如果無法使用如下所述的指引來解決問題，您必須向 [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 開啟支援票證。
> 
> 

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>位址解析通訊協定 (ARP) 和 ARP 表格
位址解析通訊協定 (ARP) 是 [RFC 826](https://tools.ietf.org/html/rfc826)中定義的第 2 層通訊協定。 ARP 可用於利用 IP 位址來對應乙太網路位址 (MAC 位址)。

ARP 表格針對特定的對等互連提供 IPv4 位址與 MAC 位址的對應。 適用於 ExpressRoute 線路對等互連的 ARP 表格會針對每個介面 (主要和次要) 提供下列資訊

1. 內部部署路由器介面 IP 位址到 MAC 位址的對應
2. ExpressRoute 路由器介面 IP 位址到 MAC 位址的對應
3. 對應的存留期

ARP 表格可協助您驗證第 2 層組態，並為第 2 層的基礎連線問題進行疑難排解。 

範例 ARP 表格： 

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           10.0.0.1   ffff.eeee.dddd
          0 Microsoft         10.0.0.2   aaaa.bbbb.cccc


下一節的資訊能為您說明如何檢視 ExpressRoute 邊緣路由器所看見的 ARP 表格。 

## <a name="prerequisites-for-learning-arp-tables"></a>了解 ARP 表格的必要條件
進一步了解之前，請確定您符合下列條件

* 至少使用一個對等互連設定的有效 ExpressRoute 線路。 線路必須由連線提供者完全設定。 您 (或您的連線提供者) 必須在這個線路上至少設定一個對等互連 (Azure 私用、Azure 公用和 Microsoft)。
* 用來設定對等互連 (Azure 私用、Azure 公用和 Microsoft) 的 IP 位址範圍。 檢閱 [ExpressRoute 路由需求頁面](expressroute-routing.md) 中的 IP 位址指派範例，以了解如何將 IP 位址對應到您所在處和 ExpressRoute 端上的介面。 您可以藉由檢閱 [ExpressRoute 對等互連組態頁面](expressroute-howto-routing-arm.md)來取得對等互連組態的詳細資訊。
* 在可與這些 IP 位址搭配使用的介面的 MAC 位址上，由網路小組 / 連線提供者所提供的資訊。
* 您必須擁有適用於 Azure 的最新 PowerShell 模組 (版本 1.50 或更新版本)。

## <a name="getting-the-arp-tables-for-your-expressroute-circuit"></a>取得適用於 ExpressRoute 線路的 ARP 表格
本節將指示您如何使用 PowerShell 來為每個對等互連檢視 ARP 表格。 您或您的連線提供者必須先設定對等互連，才能有進一步的進展。 每個線路都有兩個路徑 (主要和次要)。 您可以單獨檢查每個路徑的 ARP 表格。

### <a name="arp-tables-for-azure-private-peering"></a>適用於 Azure 私用對等互連的 ARP 表格
下列 Cmdlet 提供適用於 Azure 私用對等互連的 ARP 表格

        # Required Variables
        $RG = "<Your Resource Group Name Here>"
        $Name = "<Your ExpressRoute Circuit Name Here>"

        # ARP table for Azure private peering - Primary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Primary

        # ARP table for Azure private peering - Secodary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePrivatePeering -DevicePath Secondary 

針對其中一個路徑的範例輸出如下所示

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           10.0.0.1   ffff.eeee.dddd
          0 Microsoft         10.0.0.2   aaaa.bbbb.cccc


### <a name="arp-tables-for-azure-public-peering"></a>適用於 Azure 公用對等互連的 ARP 表格
下列 Cmdlet 提供適用於 Azure 公用對等互連的 ARP 表格

        # Required Variables
        $RG = "<Your Resource Group Name Here>"
        $Name = "<Your ExpressRoute Circuit Name Here>"

        # ARP table for Azure public peering - Primary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Primary

        # ARP table for Azure public peering - Secodary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType AzurePublicPeering -DevicePath Secondary 


針對其中一個路徑的範例輸出如下所示

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           64.0.0.1   ffff.eeee.dddd
          0 Microsoft         64.0.0.2   aaaa.bbbb.cccc


### <a name="arp-tables-for-microsoft-peering"></a>適用於 Microsoft 對等互連的 ARP 表格
下列 Cmdlet 提供適用於 Microsoft 對等互連的 ARP 表格

        # Required Variables
        $RG = "<Your Resource Group Name Here>"
        $Name = "<Your ExpressRoute Circuit Name Here>"

        # ARP table for Microsoft peering - Primary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Primary

        # ARP table for Microsoft peering - Secodary path
        Get-AzureRmExpressRouteCircuitARPTable -ResourceGroupName $RG -ExpressRouteCircuitName $Name -PeeringType MicrosoftPeering -DevicePath Secondary 


針對其中一個路徑的範例輸出如下所示

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           65.0.0.1   ffff.eeee.dddd
          0 Microsoft         65.0.0.2   aaaa.bbbb.cccc


## <a name="how-to-use-this-information"></a>如何使用此資訊
對等互連的 ARP 表格可用來決定驗證第 2 層組態與連線。 本節將概述 ARP 表格在不同狀況下的呈現方式。

### <a name="arp-table-when-a-circuit-is-in-operational-state-expected-state"></a>當線路處於運作狀態 (預期狀態) 時的 ARP 表格
* ARP 表格必須有一個適用於內部部署端的項目，其中具備有效的 IP 位址和 MAC 位址，以及一個適用於 Microsoft 端的類似項目。 
* 內部部署 IP 位址的最後一個八位元一定是奇數。
* Microsoft IP 位址的最後一個八位元一定是偶數。
* 同一個 MAC 位址將針對所有 3 個對等互連 (主要 / 次要) 出現在 Microsoft 端。 

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           65.0.0.1   ffff.eeee.dddd
          0 Microsoft         65.0.0.2   aaaa.bbbb.cccc

### <a name="arp-table-when-on-premises--connectivity-provider-side-has-problems"></a>當內部部署 / 連線提供者端發生問題時的 ARP 表格
如果內部部署或連線提供者發生問題，您可能會看到只有一個項目出現在 ARP 資料表中，或顯示的內部部署 MAC 位址不完整。 並為 Microsoft 端所使用的 MAC 位址與 IP 位址顯示其間的對應。 
  
       Age InterfaceProperty IpAddress  MacAddress    
       --- ----------------- ---------  ----------    
         0 Microsoft         65.0.0.2   aaaa.bbbb.cccc

或
       
       Age InterfaceProperty IpAddress  MacAddress    
       --- ----------------- ---------  ----------   
         0 On-Prem           65.0.0.1   Incomplete
         0 Microsoft         65.0.0.2   aaaa.bbbb.cccc


> [!NOTE]
> 向連線提供者開啟支援要求，對這類問題進行偵錯。 如果 ARP 表沒有對應到 MAC 位址之介面的 IP 位址，請檢閱下列條件︰
> 
> 1. 針對 MSEE-PR 和 MSEE 之間的連結所指派的 /30 子網路的第一個 IP 位址，是用在 MSEE-PR 的介面上。 Azure 一律為 MSEE 使用第二個 IP 位址。
> 2. 確認客戶 (C 標籤) 和服務 (S 標籤) VLAN 標籤是否都和「MSEE-PR 與 MSEE」對上的相符。
> 

### <a name="arp-table-when-microsoft-side-has-problems"></a>當 Microsoft 端發生問題時的 ARP 表格
* 如果 Microsoft 端發生問題，您將不會看見針對對等互連顯示的 ARP 表格。 
* 向 [Microsoft 支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)開啟支援票證。 指出您發生第 2 層連線的問題。 

## <a name="next-steps"></a>後續步驟
* 針對您的 ExpressRoute 線路驗證第 3 層組態
  * 取得路由摘要以判斷 BGP 工作階段的狀態 
  * 取得路由表以判斷哪些首碼是透過 ExpressRoute 來公告的
* 檢閱輸入 / 輸出的位元組來驗證資料傳輸
* 如果仍會發生問題，請向 [Microsoft 支援服務](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 開啟支援票證。

