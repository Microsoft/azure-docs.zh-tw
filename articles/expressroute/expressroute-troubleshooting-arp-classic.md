---
title: 取得 ARP 表格：傳統：Azure ExpressRoute 疑難排解 | Microsoft Docs
description: 此頁面提供取得適用於 ExpressRoute 線路之 ARP 表格的相關指示。
documentationcenter: na
services: expressroute
author: ganesr
manager: carolz
editor: tysonn
ms.assetid: b5856acf-03c2-4933-8111-6ce12998d92a
ms.service: expressroute
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/30/2017
ms.author: ganesr
ms.openlocfilehash: fcc847b7e30fd55ca759830e0254ab7542e7663e
ms.sourcegitcommit: b5c6197f997aa6858f420302d375896360dd7ceb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/21/2017
ms.locfileid: "23013163"
---
# <a name="getting-arp-tables-in-the-classic-deployment-model"></a>在傳統部署模型中取得 ARP 表格
> [!div class="op_single_selector"]
> * [PowerShell - Resource Manager](expressroute-troubleshooting-arp-resource-manager.md)
> * [PowerShell - 傳統](expressroute-troubleshooting-arp-classic.md)
> 
> 

本文章會引導您逐步完成取得 Azure ExpressRoute 電路之位址解析通訊協定 (ARP) 表格的步驟。

> [!IMPORTANT]
> 本文件旨在協助您診斷並修正簡單的問題。 它無法取代 Microsoft 支援服務。 如果以下指導方針無法解決問題，請向 [Microsoft Azure 說明 + 支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)開啟支援要求。
> 
> 

## <a name="address-resolution-protocol-arp-and-arp-tables"></a>位址解析通訊協定 (ARP) 和 ARP 表格
ARP 是在 [RFC 826](https://tools.ietf.org/html/rfc826)中定義的第 2 層通訊協定。 ARP 可用於對應乙太網路位址 (MAC 位址) 與 IP 位址。

ARP 表格能針對特定的對等互連提供 IPv4 位址與 MAC 位址對應。 適用於 ExpressRoute 線路對等互連的 ARP 表格能提供各個介面 (主要和次要) 的下列資訊：

1. 內部部署路由器介面 IP 位址與 MAC 位址的對應
2. ExpressRoute 路由器介面 IP 位址與 MAC 位址的對應
3. 對應的存留期

ARP 表格可協助您驗證第 2 層組態，並針對基本的第 2 層連線問題進行疑難排解。

以下是 ARP 表格的範例：

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           10.0.0.1 ffff.eeee.dddd
          0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


下節提供的資訊有關如何檢視 ExpressRoute 邊緣路由器所見的 ARP 表格。

## <a name="prerequisites-for-using-arp-tables"></a>使用 ARP 表格的必要條件
繼續操作之前，請確定您已備妥以下必要條件：

* 至少使用一個對等互連設定的有效 ExpressRoute 線路。 線路必須由連線提供者完全設定。 您 (或您的連線提供者) 必須在這個線路上至少設定一個對等互連 (Azure 私用、Azure 公用或 Microsoft)。
* 用來設定對等互連 (Azure 私用、Azure 公用和 Microsoft) 的 IP 位址範圍。 檢閱 [ExpressRoute 路由需求頁面](expressroute-routing.md) 中的 IP 位址指派範例，以了解如何將 IP 位址對應到您的 aise 和 ExpressRoute 端上的介面。 您可以藉由檢閱 [ExpressRoute 對等互連組態頁面](expressroute-howto-routing-classic.md)來取得對等互連組態的相關資訊。
* 由網路團隊或連線提供者所提供的資訊，其與這些 IP 位址搭配使用之介面的 MAC 位址相關。
* 適用於 Azure 的最新 Windows PowerShell 模組 (1.50 或更新版本)。

## <a name="arp-tables-for-your-expressroute-circuit"></a>適用於 ExpressRoute 線路的 ARP 表格
本節提供如何使用 PowerShell 檢視各種對等互連之 ARP 表格的指示。 在繼續之前，您或您的連線提供者需要設定對等互連。 每個線路都有兩個路徑 (主要和次要)。 您可以單獨檢查每個路徑的 ARP 表格。

### <a name="arp-tables-for-azure-private-peering"></a>適用於 Azure 私用對等互連的 ARP 表格
下列 Cmdlet 提供適用於 Azure 私用對等互連的 ARP 表格：

        # Required variables
        $ckt = "<your Service Key here>

        # ARP table for Azure private peering--primary path
        Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Primary

        # ARP table for Azure private peering--secondary path
        Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Private -Path Secondary

以下是其中一個路徑的範例輸出：

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           10.0.0.1 ffff.eeee.dddd
          0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


### <a name="arp-tables-for-azure-public-peering"></a>適用於 Azure 公用對等互連的 ARP 表格：
下列 Cmdlet 提供適用於 Azure 公用對等互連的 ARP 表格：

        # Required variables
        $ckt = "<your Service Key here>

        # ARP table for Azure public peering--primary path
        Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Primary

        # ARP table for Azure public peering--secondary path
        Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Public -Path Secondary

以下是其中一個路徑的範例輸出：

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           10.0.0.1 ffff.eeee.dddd
          0 Microsoft         10.0.0.2 aaaa.bbbb.cccc


以下是其中一個路徑的範例輸出：

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           64.0.0.1 ffff.eeee.dddd
          0 Microsoft         64.0.0.2 aaaa.bbbb.cccc


### <a name="arp-tables-for-microsoft-peering"></a>適用於 Microsoft 對等互連的 ARP 表格
下列 Cmdlet 提供適用於 Microsoft 對等互連的 ARP 表格：

    # ARP table for Microsoft peering--primary path
    Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Primary

    # ARP table for Microsoft peering--secondary path
    Get-AzureDedicatedCircuitPeeringArpInfo -ServiceKey $ckt -AccessType Microsoft -Path Secondary


其中一個路徑的範例輸出如下所示：

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           65.0.0.1 ffff.eeee.dddd
          0 Microsoft         65.0.0.2 aaaa.bbbb.cccc


## <a name="how-to-use-this-information"></a>如何使用此資訊
對等互連的 ARP 表格可用來驗證第 2 層組態與連線。 本節提供在不同狀況下 ARP 表格呈現方式的概觀。

### <a name="arp-table-when-a-circuit-is-in-an-operational-expected-state"></a>當線路處於運作 (預期) 狀態時的 ARP 表格
* ARP 表格有一個適用於內部部署端的項目，其中具備有效的 IP 和 MAC 位址，以及一個適用於 Microsoft 端的類似項目。
* 內部部署 IP 位址的最後一個八位元一定是奇數。
* Microsoft IP 位址的最後一個八位元一定是偶數。
* 同一個 MAC 位址會出現這三個對等互連 (主要 / 次要) 的 Microsoft 端。

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
         10 On-Prem           65.0.0.1 ffff.eeee.dddd
          0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

### <a name="arp-table-when-its-on-premises-or-when-the-connectivity-provider-side-has-problems"></a>當內部部署 / 連線提供者端發生問題時的 ARP 表格
 ARP 表格中只會出現一個項目。 它會顯示 Microsoft 端使用之 MAC 位址與 IP 位址的對應。

        Age InterfaceProperty IpAddress  MacAddress    
        --- ----------------- ---------  ----------    
          0 Microsoft         65.0.0.2 aaaa.bbbb.cccc

> [!NOTE]
> 如果您遇到像這樣的問題，請向連線提供者開啟支援要求以尋求解決。
> 
> 

### <a name="arp-table-when-the-microsoft-side-has-problems"></a>當 Microsoft 端發生問題時的 ARP 表格
* 如果 Microsoft 端發生問題，您將不會看見針對對等互連顯示的 ARP 表格。
* 向 [Microsoft Azure 說明 + 支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)開啟支援要求。 指出您發生第 2 層連線的問題。

## <a name="next-steps"></a>後續步驟
* 驗證 ExpressRoute 線路的第 3 層組態：
  * 取得路由摘要以判斷 BGP 工作階段的狀態。
  * 取得路由表以判斷哪些首碼是透過 ExpressRoute 來公告。
* 檢閱輸入和輸出的位元組來驗證資料傳輸。
* 如果問題持續發生，請向 [Microsoft Azure 說明 + 支援](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) 開啟支援要求。

