---
title: 檢視 Azure Stack 中的公用 IP 位址使用 |Microsoft Docs
description: 系統管理員可以檢視區域中的公用 IP 位址使用
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/10/2018
ms.author: mabrigg
ms.openlocfilehash: 6207a77555d0d4656c0728dddec769e483f4d762
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44378239"
---
# <a name="view-public-ip-address-consumption-in-azure-stack"></a>檢視 Azure Stack 中的公用 IP 位址使用

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

如果您是雲端系統管理員，您可以檢視：
 - 已配置給租用戶的公用 IP 位址數目。
 - 仍可供配置的公用 IP 位址數目。
 - 該位置中已配置的公用 IP 位址百分比。

[公用 IP 集區使用量] 圖格會顯示所有公用 IP 位址集區使用的公用 IP 位址數目。 針對每個 IP 位址，此圖格會針對租用戶 IaaS VM 執行個體、網狀架構基礎結構服務，以及租用戶明確建立的公用 IP 位址資源，顯示其使用量。

此圖格的目的在於讓 Azure Stack 操作員了解此位置中已使用的公用 IP 位址數目。 此數字可協助系統管理員判斷此資源是否即將耗盡。

[租用戶資源] 下的 [公用 IP 位址] 功能表項目只會列出「租用戶明確建立的」公用 IP 位址。 您可以在 [資源提供者]、[網路] 窗格上找到此功能表項目。 [公用 IP 集區使用方式] 圖格上的 [已使用] 公用 IP 位址數目，永遠不同於 (大於) 在 [租用戶資源] 下 [公用 IP 位址] 圖格上的數目。

## <a name="view-the-public-ip-address-usage-information"></a>檢視公用 IP 位址使用方式資訊
若要檢視區域中已使用的公用 IP 位址總數：

1. 在 Azure Stack 系統管理員入口網站中，選取 [所有服務]。 接著，在 [系統管理] 類別底下，選取 [網路]。
1. [網路] 視格會在 [概觀] 區段中顯示 [公用 IP 集區使用方式] 圖格。

![網路資源提供者窗格](media/azure-stack-viewing-public-ip-address-consumption/image01.png)

[已使用] 數字代表從公用 IP 位址集區指派的公用 IP 位址數目。 [可用] 數字代表來自尚未指派，且仍可供使用之公用 IP 位址集區的公用 IP 數目。 [% 已使用] 數字代表該位置內公用 IP 位址集區中已使用或已指派公用 IP 位址總數的百分比數目。

## <a name="view-the-public-ip-addresses-that-were-created-by-tenant-subscriptions"></a>檢視租用戶訂用帳戶所建立的公用 IP 位址
選取 [租用戶資源] 下的 [公用 IP 位址]。 檢閱特定區域中租用戶訂用帳戶所明確建立的公用 IP 位址清單。

![租用戶公用 IP 位址](media/azure-stack-viewing-public-ip-address-consumption/image02.png)

您可能會注意到某些已動態配置的公用 IP 位址會顯示在清單中。 不過，尚未具有與其相關聯的位址。 位址資源已在網路資源提供者中建立，但尚未在網路控制卡中建立。

網路控制卡在位址繫結至介面、網路介面卡 (NIC)、負載平衡器或虛擬網路閘道之前，不會將該位址指派給此資源。 當公用 IP 位址繫結至介面時，網路控制卡會配置 IP 位址。 此位址會顯示在 [位址] 欄位中。

## <a name="view-the-public-ip-address-information-summary-table"></a>檢視公用 IP 位址資訊摘要資料表
在不同的情況下，指派公用 IP 位址可判斷位址是否顯示在一個 (或其他) 清單中。

| **公用 IP 位址指派案例** | **顯示於使用方式摘要** | **顯示於租用戶公用 IP 位址清單** |
| --- | --- | --- |
| 尚未指派給 NIC 或負載平衡器 (暫時) 的動態公用 IP 位址 |否 |是 |
| 已指派給 NIC 或負載平衡器的動態公用 IP 位址。 |是 |是 |
| 已指派給租用戶 NIC 或負載平衡器的靜態公用 IP 位址。 |是 |是 |
| 已指派給網狀架構基礎結構服務端點的靜態公用 IP 位址。 |是 |否 |
| 公用 IP 位址會以隱含方式針對 IaaS VM 執行個體而建立，並用於虛擬網路上的輸出 NAT。 每當租用戶建立 VM 執行個體，讓 VM 可以將資訊送出到網際網路時，這些位址便會在幕後建立。 |是 |否 |

## <a name="next-steps"></a>後續步驟
[在 Azure Stack 中管理儲存體帳戶](azure-stack-manage-storage-accounts.md)